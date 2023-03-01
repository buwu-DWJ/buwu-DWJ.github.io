# Calling Python in Rust code

本章包含在 Rust 中与 Python 代码互动的几种方法
- 怎么调用 Python 函数
- 怎么执行现有的 Python 代码

## Calling Python functions

任意 Python 原生的对象引用（例如`&PyAny`, `&PyList` 或 `&PyCell<MyClass>`）可以被用来调用 Python 函数。

PyO3 提供了两个 API 来进行函数调用

- `call`：调用任何可调用的 Python 对象
- `call_method`：调用 Python 对象上的一个方法

这两个 API 接受`args` 和 `kwargs` 变量，有更简单形式的调用

- `call1`和`call_method1`仅用位置变量`args`来进行调用
- `call0`和`call_method0`不需要变量进行调用

Both of these APIs take `args` and `kwargs` arguments (for positional and keyword arguments respectively). There are variants for less complex calls:

方便起见 `Py<T>` 智能指针也会 expose 六个 API 方法上，但是需要 `Python` 记号作为额外的第一个变量来显示 GIL 被占用。

下述例子调用了`PyObject`（aka `Py<PyAny>`）引用后的一个 Python 函数：

```rust
use pyo3::prelude::*;
use pyo3::types::PyTuple;

fn main() -> PyResult<()> {
    let arg1 = "arg1";
    let arg2 = "arg2";
    let arg3 = "arg3";

    Python::with_gil(|py| {
        let fun: Py<PyAny> = PyModule::from_code(
            py,
            "def example(*args, **kwargs):
                if args != ():
                    print('called with args', args)
                if kwargs != {}:
                    print('called with kwargs', kwargs)
                if args == () and kwargs == {}:
                    print('called with no arguments')",
            "",
            "",
        )?
        .getattr("example")?
        .into();

        // call object without any arguments
        fun.call0(py)?;

        // call object with PyTuple
        let args = PyTuple::new(py, &[arg1, arg2, arg3]);
        fun.call1(py, args)?;

        // pass arguments as rust tuple
        let args = (arg1, arg2, arg3);
        fun.call1(py, args)?;
        Ok(())
    })
}
```

### 创建关键字变量

对于 `call` 和 `call_method` API，`kwargs`可以是`None`或者 `Some(&PyDict)`。可以使用 `IntoPyDict` 特征来转换其他的 dict-like 容器，e.g. `HashMap` 或者 `BTreeMap`， 至多十个元素的元组和每个元素是一个二元元组的`Vec`。

```rust
use pyo3::prelude::*;
use pyo3::types::IntoPyDict;
use std::collections::HashMap;

fn main() -> PyResult<()> {
    let key1 = "key1";
    let val1 = 1;
    let key2 = "key2";
    let val2 = 2;

    Python::with_gil(|py| {
        let fun: Py<PyAny> = PyModule::from_code(
            py,
            "def example(*args, **kwargs):
                if args != ():
                    print('called with args', args)
                if kwargs != {}:
                    print('called with kwargs', kwargs)
                if args == () and kwargs == {}:
                    print('called with no arguments')",
            "",
            "",
        )?
        .getattr("example")?
        .into();

        // call object with PyDict
        let kwargs = [(key1, val1)].into_py_dict(py);
        fun.call(py, (), Some(kwargs))?;

        // pass arguments as Vec
        let kwargs = vec![(key1, val1), (key2, val2)];
        fun.call(py, (), Some(kwargs.into_py_dict(py)))?;

        // pass arguments as HashMap
        let mut kwargs = HashMap::<&str, i32>::new();
        kwargs.insert(key1, 1);
        fun.call(py, (), Some(kwargs.into_py_dict(py)))?;

        Ok(())
    })
}
```

## 执行已有的Python代码

如果有想在 Rust 中执行的 Python 代码，下面的 FAQS 可以帮助选择正确的 PyO3 功能

### 想要接入 Python API？`PyModule::import`

`Pymodule::import` 可以用来在 Rust 中处理 Python 模组。

```rust
use pyo3::prelude::*;

fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        let builtins = PyModule::import(py, "builtins")?;
        let total: i32 = builtins
            .getattr("sum")?
            .call1((vec![1, 2, 3],))?
            .extract()?;
        assert_eq!(total, 6);
        Ok(())
    })
}
```

### 仅仅运行一个表达式（expressing）？使用`eval`

`Python::eval` 是执行[Python 表达式](https://docs.python.org/3.7/reference/expressions.html)的方法，返回一个 `&PyAny` 对象。

```rust
use pyo3::prelude::*;

# fn main() -> Result<(), ()> {
Python::with_gil(|py| {
    let result = py
        .eval("[i * 10 for i in range(5)]", None, None)
        .map_err(|e| {
            e.print_and_set_sys_last_vars(py);
        })?;
    let res: Vec<i64> = result.extract().unwrap();
    assert_eq!(res, vec![0, 10, 20, 30, 40]);
    Ok(())
})
# }
```

### 想要运行语句（statement）？使用`run`

`Python::run` 是执行一个或多个[Python语句](https://docs.python.org/3.7/reference/simple_stmts.html)的方法。这个方法不返回东西，但是可以通过`locals`字典去获取对象。

也可以使用其简写：`py_run!`宏。因为`py_run!`在异常时会panic，所以只推荐在测试异常时使用宏

```rust
use pyo3::prelude::*;
use pyo3::{PyCell, py_run};

fn main() {
#[pyclass]
struct UserData {
    id: u32,
    name: String,
}

#[pymethods]
impl UserData {
    fn as_tuple(&self) -> (u32, String) {
        (self.id, self.name.clone())
    }

    fn __repr__(&self) -> PyResult<String> {
        Ok(format!("User {}(id: {})", self.name, self.id))
    }
}

Python::with_gil(|py| {
    let userdata = UserData {
        id: 34,
        name: "Yu".to_string(),
    };
    let userdata = PyCell::new(py, userdata).unwrap();
    let userdata_as_tuple = (34, "Yu");
    py_run!(py, userdata userdata_as_tuple, r#"
assert repr(userdata) == "User Yu(id: 34)"
assert userdata.as_tuple() == userdata_as_tuple
    "#);
})
}
```

## 有一个Python文件或者代码片段？使用`PyModule::from_code`

`PyModule::from_code`可以用来生成能使用的Python模组，就像通过`PyModule::import`导入的。

**Warning**: 这会编译并执行代码，**永远不要** 传递不可信任的代码给这个函数

```rust
use pyo3::{prelude::*, types::{IntoPyDict, PyModule}};

fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        let activators = PyModule::from_code(py, r#"
    def relu(x):
        """see https://en.wikipedia.org/wiki/Rectifier_(neural_networks)"""
        return max(0.0, x)

    def leaky_relu(x, slope=0.01):
        return x if x >= 0 else x * slope
        "#, "activators.py", "activators")?;

        let relu_result: f64 = activators.getattr("relu")?.call1((-1.0,))?.extract()?;
        assert_eq!(relu_result, 0.0);

        let kwargs = [("slope", 0.2)].into_py_dict(py);
        let lrelu_result: f64 = activators
            .getattr("leaky_relu")?
            .call((-1.0,), Some(kwargs))?
            .extract()?;
        assert_eq!(lrelu_result, -0.2);
        Ok(())
    })
}
```

### 想在Rust中嵌入Python一个额外的模组？

Python 为所有导入的包保存了一个`sys.modules`字典。在Python中的导入包会首先在这个字典中进行查找，如果没有找到会尝试一些其他的办法。

`append_to_inittab`宏可以用来为嵌入的 Python 解释器添加额外的`#[pymodule]`，这个宏**必须**在初始化Python前唤出。

例子，为嵌入的解释器添加`foo`模组

```rust
use pyo3::prelude::*;

#[pyfunction]
fn add_one(x: i64) -> i64 {
    x + 1
}

#[pymodule]
fn foo(_py: Python<'_>, foo_module: &PyModule) -> PyResult<()> {
    foo_module.add_function(wrap_pyfunction!(add_one, foo_module)?)?;
    Ok(())
}

fn main() -> PyResult<()> {
    pyo3::append_to_inittab!(foo);
    Python::with_gil(|py| Python::run(py, "import foo; foo.add_one(6)", None, None))
}
```

如果`append_to_inittab`因为程序员因无法使用，可以使用`PyModule::new`创建一个模组并手动添加进`sys.modules`

```rust
use pyo3::prelude::*;
use pyo3::types::PyDict;

#[pyfunction]
pub fn add_one(x: i64) -> i64 {
    x + 1
}

fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        // Create new module
        let foo_module = PyModule::new(py, "foo")?;
        foo_module.add_function(wrap_pyfunction!(add_one, foo_module)?)?;

        // Import and get sys.modules
        let sys = PyModule::import(py, "sys")?;
        let py_modules: &PyDict = sys.getattr("modules")?.downcast()?;

        // Insert foo into sys.modules
        py_modules.set_item("foo", foo_module)?;

        // Now we can import + run our python code
        Python::run(py, "import foo; foo.add_one(6)", None, None)
    })
}
```

### 包含许多Python文件

可以使用[`std::include_str`](https://doc.rust-lang.org/std/macro.include_str.html)宏在编译时包含一个文件.

或者在运行时通过[`std::fs::read_to_string`](https://doc.rust-lang.org/std/fs/fn.read_to_string.html)函数加载一个文件。

许多Python文件可以作为模组被包含并加载。如果一个文件依赖于另一个文件，必须在声明`PyModule`时保持正确的顺序，

目录结构例子：

```text
.
├── Cargo.lock
├── Cargo.toml
├── python_app
│   ├── app.py
│   └── utils
│       └── foo.py
└── src
    └── main.rs
```

`python_app/app.py`:
```python
from utils.foo import bar


def run():
    return bar()
```

`python_app/utils/foo.py`:
```python
def bar():
    return "baz"
```

下面的例子说明了
- 怎样将`app.py` 和 `utils/foo.py` 的内容包含进你的Rust二进制中
- 怎么调用在`app.py`中声明的函数`run()`（它需要从`utils/foo.py`导入的函数）

`src/main.rs`:
```rust
use pyo3::prelude::*;

fn main() -> PyResult<()> {
    let py_foo = include_str!(concat!(env!("CARGO_MANIFEST_DIR"), "/python_app/utils/foo.py"));
    let py_app = include_str!(concat!(env!("CARGO_MANIFEST_DIR"), "/python_app/app.py"));
    let from_python = Python::with_gil(|py| -> PyResult<Py<PyAny>> {
        PyModule::from_code(py, py_foo, "utils.foo", "utils.foo")?;
        let app: Py<PyAny> = PyModule::from_code(py, py_app, "", "")?
            .getattr("run")?
            .into();
        app.call0(py)
    });

    println!("py: {}", from_python?);
    Ok(())
}
```

下面的例子说明了：

- 怎样在运行中导入`app.py`的内容，让它自动解决其依赖
- 怎么调用在`app.py`中声明的函数`run()`（它需要从`utils/foo.py`导入的函数）

建议使用绝对路径，从而你的二进制可以在任意位置运行只要你的 `app.py` 在预期的目录下（这个例子中，目录为`/usr/share/python_app`）

`src/main.rs`:
```rust
use pyo3::prelude::*;
use pyo3::types::PyList;
use std::fs;
use std::path::Path;

fn main() -> PyResult<()> {
    let path = Path::new("/usr/share/python_app");
    let py_app = fs::read_to_string(path.join("app.py"))?;
    let from_python = Python::with_gil(|py| -> PyResult<Py<PyAny>> {
        let syspath: &PyList = py.import("sys")?.getattr("path")?.downcast()?;
        syspath.insert(0, &path)?;
        let app: Py<PyAny> = PyModule::from_code(py, &py_app, "", "")?
            .getattr("run")?
            .into();
        app.call0(py)
    });

    println!("py: {}", from_python?);
    Ok(())
}
```

## 需要在Rust中使用上下文管理器？

直接通过 `__enter__` 和 `__exit__` 使用上下文管理器

```rust
use pyo3::prelude::*;
use pyo3::types::PyModule;

fn main() {
    Python::with_gil(|py| {
        let custom_manager = PyModule::from_code(py, r#"
class House(object):
    def __init__(self, address):
        self.address = address
    def __enter__(self):
        print(f"Welcome to {self.address}!")
    def __exit__(self, type, value, traceback):
        if type:
            print(f"Sorry you had {type} trouble at {self.address}")
        else:
            print(f"Thank you for visiting {self.address}, come again soon!")

        "#, "house.py", "house").unwrap();

        let house_class = custom_manager.getattr("House").unwrap();
        let house = house_class.call1(("123 Main Street",)).unwrap();

        house.call_method0("__enter__").unwrap();

        let result = py.eval("undefined_variable + 1", None, None);

        // If the eval threw an exception we'll pass it through to the context manager.
        // Otherwise, __exit__  is called with empty arguments (Python "None").
        match result {
            Ok(_) => {
                let none = py.None();
                house.call_method1("__exit__", (&none, &none, &none)).unwrap();
            },
            Err(e) => {
                house.call_method1(
                    "__exit__",
                    (e.get_type(py), e.value(py), e.traceback(py))
                ).unwrap();
            }
        }
    })
}
```
