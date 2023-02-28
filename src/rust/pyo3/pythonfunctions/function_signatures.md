# 函数签名（Function signatures）

`#[pyfunction]`同样也接受参数来控制生成的 Python 函数如何来接受参数。就像在 Python中一样，参数可以是 positional-only，keyword-only 或者同时是都有。`*args` 列表和 `**kwargs`字典也可以被接收。这些参数对下一章 Python Classes 中介绍的 `[pyclassmethods]` 也同样有效。

默认下，和 Python 一样，PyO3 接受所有参数作为 positional 或者 keyword 参数。大多数参数默认是需要的，除非结尾 `Option<_>` 参数，它会隐式的被给予一个 `None`。有两种方法改变这种模式：
- `#[pyo3(signature = (...))]` 允许用 Python 语法写一个签名
- 额外的参数直接使用`#[pyfunction]`，见deprecated form（已弃用？）

## 使用`#[pyo3(signature = (...))]`

下述例子是一个函数可以接受任意的关键字参数（Python 语法中的`**kwargs`）并返回传入的参数量：
```rust
use pyo3::prelude::*;
use pyo3::types::PyDict;

#[pyfunction]
#[pyo3(signature = (**kwds))]
fn num_kwds(kwds: Option<&PyDict>) -> usize {
    kwds.map_or(0, |dict| dict.len())
}

#[pymodule]
fn module_with_functions(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(num_kwds, m)?).unwrap();
    Ok(())
}

```

下述结构可以可以作为签名的一部分

- `/`: 位置变量分隔符，每个定义在`/`前的变量都是一个位置（positionla-only）变量
- `*`: 每个定义在`*`后的变量是关键字（keyword-only）变量
- `*args`: var args，args parameter 的类型必须为 `&PyTuple`
- `**kwargs`: 接受关键字变量，关键自变量的类型必须为 `Option<&PyDict>`
- `arg=Value`: 有默认值的变量，如果 `arg` 在 var argments 后被定义，则被视为  keyword-only 变量

例子

```rust
use pyo3::types::{PyDict, PyTuple};
#[pymethods]
impl MyClass {
    #[new]
    #[pyo3(signature = (num=-1))]
    fn new(num: i32) -> Self {
        MyClass { num }
    }

    #[pyo3(signature = (num=10, *py_args, name="Hello", **py_kwargs))]
    fn method(
        &mut self,
        num: i32,
        py_args: &PyTuple,
        name: &str,
        py_kwargs: Option<&PyDict>,
    ) -> String {
        let num_before = self.num;
        self.num = num;
        format!(
            "num={} (was previously={}), py_args={:?}, name={}, py_kwargs={:?} ",
            num, num_before, py_args, name, py_kwargs,
        )
    }

    fn make_change(&mut self, num: i32) -> PyResult<String> {
        self.num = num;
        Ok(format!("num={}", self.num))
    }
}
```

`Python`类型的变量不能是签名的一部分

```rust
#[pyfunction]
#[pyo3(signature = (lambda))]
pub fn simple_python_bound_function(
    py: Python<'_>,
    lambda: PyObject,
) -> PyResult<()> {
    Ok(())
}

```

`/`和`*`变量的位置决定了系统如何处理位置和关键自变量，在 Python 中

```python
import mymodule

mc = mymodule.MyClass()
print(mc.method(44, False, "World", 666, x=44, y=55))
print(mc.method(num=-1, name="World"))
print(mc.make_change(44, False))

```

输出如下

```console
py_args=('World', 666), py_kwargs=Some({'x': 44, 'y': 55}), name=Hello, num=44
py_args=(), py_kwargs=None, name=World, num=-1
num=44
num=-1

```

> 使用关键字如`struct`作为函数的变量，同时在签名和函数定义中使用 'raw identifier' 语法 `r#struct`
>```rust
>#[pyfunction(signature = (r#struct = "foo"))]
>fn function_with_keyword(r#struct: &str) {
>    /* ... */
>}
>```

## 尾随可选参数（Trailing optional arguments）

方便起见，没有`#[pyo3(signature = (...))]`的函数将末尾的`Option<T>`参数默认视为`None`，下述例子中，PyO3会创建 `increment` 函数带有签名 `increment(x, amount=None)`

```rust
use pyo3::prelude::*;

/// Returns a copy of `x` increased by `amount`.
///
/// If `amount` is unspecified or `None`, equivalent to `x + 1`.
#[pyfunction]
fn increment(x: u64, amount: Option<u64>) -> u64 {
    x + amount.unwrap_or(1)
}

```

要让末尾参数 `Option<T>` required，但仍然允许 `None`，添加一个`#[pyo3(signature = (...))]` 声明，在上述例子中，就是`#[pyo3(signature = (x, amount))]`：
```rust
#[pyfunction]
#[pyo3(signature = (x, amount))]
fn increment(x: u64, amount: Option<u64>) -> u64 {
    x + amount.unwrap_or(1)
}
```

为避免混淆，当 `Option<T>` 变量被不是 `Option<T>`的变量包围时，PyO3 需要 `#[pyo3(signature = (...))]`。

## 使函数签名对Python可用

函数签名对 Python 是可见的，通过`__text_signature__`属性。PyO3会自动为每个`#[pyfunction]`和`#[pymethods]`生成属性，会被`#[pyo3(signature = (...))]`覆盖。

这个仍有些缺点，未来有待改进
- 不包含变量的默认值，显示 `...`
- 对`#[pyclass]`中的`#[new]`方法不会生成

例子

```rust
use pyo3::prelude::*;

/// This function adds two unsigned 64-bit integers.
#[pyfunction]
#[pyo3(signature = (a, b=0, /))]
fn add(a: u64, b: u64) -> u64 {
    a + b
}
```

其签名在 Python 中是这样的

```console
>>> pyo3_test.add.__text_signature__
'(a, b=..., /)'
>>> pyo3_test.add?
Signature: pyo3_test.add(a, b=Ellipsis, /)
Docstring: This function adds two unsigned 64-bit integers.
Type:      builtin_function_or_method
```

### 替换生成的签名

`#[pyo3(text_signature = "(<some signature>)")]`可以用覆盖默认生成的签名，

```rust
use pyo3::prelude::*;

/// This function adds two unsigned 64-bit integers.
#[pyfunction]
#[pyo3(signature = (a, b=0, /), text_signature = "(a, b=0, /)")]
fn add(a: u64, b: u64) -> u64 {
    a + b
}

```

Python 中显示

```console
>>> pyo3_test.add.__text_signature__
'(a, b=0, /)'
>>> pyo3_test.add?
Signature: pyo3_test.add(a, b=0, /)
Docstring: This function adds two unsigned 64-bit integers.
Type:      builtin_function_or_method

```