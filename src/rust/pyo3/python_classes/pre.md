# Python classes

PyO3 提供许多 Rust 宏支持的属性来定义 Rust 结构的 Python 类。

主要的属性是 `#[pyclass]`，放在一个 Rust 结构或者一个 fieldless 的枚举上面来为其生成一个 Python 类型。通常它们会有一个 `#[pymethods]`：用 `impl` 块注释的结构，用来定义 Python 方法和生成的 Python 类型的约束。（如果开启了 `multiple-pymethods`）特性，那么每个 `#[pyclass]`可以有多个 `#[pymethods]` 块。`#[pymethods]` 可以有类似 `__str__` 的 Python 魔术方法的实现。

## 定义一个新类

定义一个 Python 类，需要在 Rust 结构或者 fieldless 枚举前加上 `#[pyclass]`

```rust
use pyo3::prelude::*;

#[pyclass]
struct Integer {
    inner: i32,
}

// A "tuple" struct
#[pyclass]
struct Number(i32);

// PyO3 supports custom discriminants in enums
#[pyclass]
enum HttpResponse {
    Ok = 200,
    NotFound = 404,
    Teapot = 418,
    // ...
}

#[pyclass]
enum MyEnum {
    Variant,
    OtherVariant = 30, // PyO3 supports custom discriminants.
}
```

### 限制

为了整合 Rust 类型与 Python，PyO3 需要对可以用 `#[pyclass]` 注释的类型加以限制。特别地，它们**不能有生命周期**的参数，**不能有泛型**参数，**必须实现了 [`send`](https://doc.rust-lang.org/std/marker/trait.Send.html)**。理由如下

#### 没有生命周期

Rust 编译器为了内存安全使用生命周期，它们只在编译中使用，无法传输到 Python 这样的动态语言中。

#### 没有泛型参数

一个带有泛型参数 `T` 的 Rust 结构 `Foo<T>` ，每次传入一个不同的具体类型的`T`时，会生成新的编译实现。这在 Python 中并不现实，因为 Python 中编译器只需要 `Foo` 的一个单独的实现。

#### 必须要有[`Send`](https://doc.rust-lang.org/std/marker/trait.Send.html)

因为 Python 解释器中，Python 对象在线程中是免费共享的，不能保证哪个线程会最终丢弃这个对象。因此，每个用 `#[pyclass]` 注释的类型必须实现 [`Send`](https://doc.rust-lang.org/std/marker/trait.Send.html)（除非注释 `#[pyclass(unsendable)]`）。

## 构造器（Constructor）

默认下是不可能通过 Python 代码创建一个自定义类的实例啊。为了声明一个构造器，需要定义一个方法并用 `#[new]` 属性注释它。只有 Python 的`__new__` 方法可以，而`__init__`不可用。

```rust
#[pymethods]
impl Number {
    #[new]
    fn new(value: i32) -> Self {
        Number(value)
    }
}
```
可选，如果你的 `#[new]` 可能会失败你可以返回 `PyResult<Self>`

```rust
#[pymethods]
impl Nonzero {
    #[new]
    fn py_new(value: i32) -> PyResult<Self> {
        if value == 0 {
            Err(PyValueError::new_err("cannot be zero"))
        } else {
            Ok(Nonzero(value))
        }
    }
}
```

如果没有声明含有 `#[new]` 的方法，那么只能由 Rust 创建实例，Python 无法创建。

## 将类添加到模组上

```rust
#[pymodule]
fn my_module(_py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_class::<Number>()?;
    Ok(())
}
```

## PyCell and interior mutability

有时候需要将 `pyclass` 转换成一个 Python 对象然后在 Rust 代码中使用它，`PyCell`是主要的接口。

`PyCell<T: PyClass>`永远出现在 Python 语块中，所以 Rust 没有它的所有权。也就是说，Rust只能提取`&PyCell<T>`, not a 而不是`PyCell<T>`。

因此，想要安全地改变`&PyCell`后的数据，PyO3 采用例如 `RefCell` 这样的 Interior Mutability Pattern。

回忆Rust中借用的规则
- 在任意时刻，只能同时存在一个可变引用或者数个不可变引用
- 引用必须永远是有效的

`PyCell`，类似`RefCell`，通过在运行中跟踪引用来确保这些借用规则

```rust
#[pyclass]
struct MyClass {
    #[pyo3(get)]
    num: i32,
}
Python::with_gil(|py| {
    let obj = PyCell::new(py, MyClass { num: 3 }).unwrap();
    {
        let obj_ref = obj.borrow(); // Get PyRef
        assert_eq!(obj_ref.num, 3);
        // You cannot get PyRefMut unless all PyRefs are dropped
        assert!(obj.try_borrow_mut().is_err());
    }
    {
        let mut obj_mut = obj.borrow_mut(); // Get PyRefMut
        obj_mut.num = 5;
        // You cannot get any other refs until the PyRefMut is dropped
        assert!(obj.try_borrow().is_err());
        assert!(obj.try_borrow_mut().is_err());
    }

    // You can convert `&PyCell` to a Python object
    pyo3::py_run!(py, obj, "assert obj.num == 5");
});
```

`&PyCell<T>`被限制了与 `GILGuard` 同样的生命周期。为了使对象存活更长（例如，在Rust中将它存在一个结构中），可以使用`Py<T>`，它可以将一个对象存储得比 GIL 更久，然后需要 `Python<'_>`来获取。

```rust
#[pyclass]
struct MyClass {
    num: i32,
}

fn return_myclass() -> Py<MyClass> {
    Python::with_gil(|py| Py::new(py, MyClass { num: 1 }).unwrap())
}

let obj = return_myclass();

Python::with_gil(|py| {
    let cell = obj.as_ref(py); // Py<MyClass>::as_ref returns &PyCell<MyClass>
    let obj_ref = cell.borrow(); // Get PyRef<T>
    assert_eq!(obj_ref.num, 1);
});
```

## 自定义类

`#[pyclass]`可以使用如下参数

| 参数                                   | 描述                                                                                                                                            |
| :------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| `crate = "some::path"`                 | 导入`pyo3`包的路径，如果`::pyo3`不可用                                                                                                          |
| `dict`                                 | 给予这个类的实例一个空的`__dict__`来存储自定义属性                                                                                              |
| `extends = BaseType`                   | 使用一个自定义的基类，默认为`PyAny`                                                                                                             |
| `freelist = N`                         | 实现一个大小为 N 的 [free list](https://en.wikipedia.org/wiki/Free_list)，对那些经常要创建和删除的类型可以提升性能                              |
| `frozen`                               | 声明你的类是不可变的。会移除当检索到Rust结构的共享引用时的顶层借用检查，但同时无法再获取到一个可变的引用。                                      |
| `get_all`                              | 为类的所有域生成 getter                                                                                                                         |
| `mapping`                              | 告诉PyO3这个类是一个 `mapping`，                                                                                                                |
| `module = "module_name"`               | Python中看到的类定义所在的模组名，默认为`builtins`                                                                                              |
| `name = "python_name"`                 | Python看到的类名                                                                                                                                |
| `sequence`                             | 告诉PyO3这个类是一个 `Sequence`                                                                                                                 |
| `set_all`                              | 为这类的所有域生成 setter                                                                                                                       |
| `subclass`                             | 允许其他Python类或者`#[pyclass]`从这个类中继承，枚举无法被继承                                                                                  |
| `text_signature = "(arg1, arg2, ...)"` | 为Python类的 `__new__` 方法设置文字签名                                                                                                         |
| `unsendable`                           | 如果你的结构不是 [`Send`](https://doc.rust-lang.org/std/marker/trait.Send.html)的，如果使用了 `unsendable`，你的类会 panic 如果被另一个线程调用 |
| `weakref`                              | 允许这个类为[弱应用](https://docs.python.org/3/library/weakref.html)                                                                            |

所有这些参数可以直接通过`#[pyclass(...)]`注释传递，或者通过`#[pyo3(...)]`传递，e.g.:

```rust
// Argument supplied directly to the `#[pyclass]` annotation.
#[pyclass(name = "SomeName", subclass)]
struct MyClass {}

// Argument supplied as a separate annotation.
#[pyclass]
#[pyo3(name = "SomeName", subclass)]
struct MyClass {}
```

### 返回类型

一般来说，`#[new]`方法必须返回`T: Into<PyClassInitializer<Self>>`或者`PyResult<T>`，其中`T: Into<PyClassInitializer<Self>>`。

对于可能失败的构造器，同样可以把返回类型包装进一个`PyResult`。

|                | 不可能失败                                                                                      | 可能失败                          |
| :------------- | :---------------------------------------------------------------------------------------------- | :-------------------------------- |
| 非继承         | `T`                                                                                             | `PyResult<T>`                     |
| 继承(T继承U)   | `(T, U)`                                                                                        | `PyResult<(T, U)>`                |
| 继承(一般情形) | [`PyClassInitializer<T>`](https://pyo3.rs/main/doc/pyo3/pyclass_init/struct.pyclassinitializer) | `PyResult<PyClassInitializer<T>>` |

## 继承 Inheritance


默认情况下，`object` i.e. `PyAny` 被用作基本类。要覆盖这种默认，对`pyclass`使用 `extends` 参数，参数值为基类的完整路径。

方便起见，`(T, U)` 实现了`Into<PyClassInitializer<T>>`，其中`U`是`T`的基类。但是对更深的嵌套循环，还是必须明确地使用`PyClassInitializer<T>`。

要从一个子类获得其父类，对方法使用`PyRef`（而不是`&self`）或者`PyRefMut`（而不是`&mut self`）。这样就可以通过`&Self::BaseClass`的`self_.as_ref()`或者`PyRef<Self::BaseClass>`的`self_.into_super()`获取父类。

```rust
use pyo3::prelude::*;

#[pyclass(subclass)]
struct BaseClass {
    val1: usize,
}

#[pymethods]
impl BaseClass {
    #[new]
    fn new() -> Self {
        BaseClass { val1: 10 }
    }

    pub fn method(&self) -> PyResult<usize> {
        Ok(self.val1)
    }
}

#[pyclass(extends=BaseClass, subclass)]
struct SubClass {
    val2: usize,
}

#[pymethods]
impl SubClass {
    #[new]
    fn new() -> (Self, BaseClass) {
        (SubClass { val2: 15 }, BaseClass::new())
    }

    fn method2(self_: PyRef<'_, Self>) -> PyResult<usize> {
        let super_ = self_.as_ref(); // Get &BaseClass
        super_.method().map(|x| x * self_.val2)
    }
}

#[pyclass(extends=SubClass)]
struct SubSubClass {
    val3: usize,
}

#[pymethods]
impl SubSubClass {
    #[new]
    fn new() -> PyClassInitializer<Self> {
        PyClassInitializer::from(SubClass::new()).add_subclass(SubSubClass { val3: 20 })
    }

    fn method3(self_: PyRef<'_, Self>) -> PyResult<usize> {
        let v = self_.val3;
        let super_ = self_.into_super(); // Get PyRef<'_, SubClass>
        SubClass::method2(super_).map(|x| x * v)
    }
}
Python::with_gil(|py| {
    let subsub = pyo3::PyCell::new(py, SubSubClass::new()).unwrap();
    pyo3::py_run!(py, subsub, "assert subsub.method3() == 3000")
});
```

也可以继承类似`PyDict`的原生类型，只要它们实现了`PySizedLayout`。但是由于技术问题，现在还未向继承了原生类型的类型提供安全的 upcasting 方法。即使在这样的情况下，可以 _unsafely get a base class by raw pointer conversion_


```rust
#[cfg(not(Py_LIMITED_API))] {
use pyo3::prelude::*;
use pyo3::types::PyDict;
use pyo3::AsPyPointer;
use std::collections::HashMap;

#[pyclass(extends=PyDict)]
#[derive(Default)]
struct DictWithCounter {
    counter: HashMap<String, usize>,
}

#[pymethods]
impl DictWithCounter {
    #[new]
    fn new() -> Self {
        Self::default()
    }

    fn set(mut self_: PyRefMut<'_, Self>, key: String, value: &PyAny) -> PyResult<()> {
        self_.counter.entry(key.clone()).or_insert(0);
        let py = self_.py();
        let dict: &PyDict = unsafe { py.from_borrowed_ptr_or_err(self_.as_ptr())? };
        dict.set_item(key, value)
    }
}
Python::with_gil(|py| {
    let cnt = pyo3::PyCell::new(py, DictWithCounter::new()).unwrap();
    pyo3::py_run!(py, cnt, "cnt.set('abc', 10); assert cnt['abc'] == 10")
});
}
```

如果 `SubClass` 没有提供基类的继承，编译会失败。

```rust
# use pyo3::prelude::*;

#[pyclass]
struct BaseClass {
    val1: usize,
}

#[pyclass(extends=BaseClass)]
struct SubClass {
    val2: usize,
}

#[pymethods]
impl SubClass {
    #[new]
    fn new() -> Self {
        SubClass { val2: 15 }
    }
}
```

当创建了一个Python实例时，原生基类的`__new__`构造器会隐式地被调用。确保在（希望基类获得的）`#[new]`方法中接受参数，即使它们没有在那个`fn`中被使用：

```rust
#[allow(dead_code)]
#[cfg(not(Py_LIMITED_API))] {
# use pyo3::prelude::*;
use pyo3::types::PyDict;

#[pyclass(extends=PyDict)]
struct MyDict {
    private: i32,
}

#[pymethods]
impl MyDict {
    #[new]
    #[pyo3(signature = (*args, **kwargs))]
    fn new(args: &PyAny, kwargs: Option<&PyAny>) -> Self {
        Self { private: 0 }
    }

    // some custom methods that use `private` here...
}
Python::with_gil(|py| {
    let cls = py.get_type::<MyDict>();
    pyo3::py_run!(py, cls, "cls(a=1, b=2)")
});
}
```

这里，`args`和`kwargs`允许创建传递了初始 item 的实例，例如`MyDict(item_sequence)` 或 `MyDict(a=1, b=2)`。

## 对象性质

PyO3 支持两种方式来对`#[pyclass]`添加性质：
- 对简单的结构域，没有副作用，可以直接在`#[pyclass]`的域定义中添加`#[pyo3(get, set)]`属性
- 对于需要计算（computation）的属性，可以在[`#[pymethods]`](#instance-methods)块中定义`#[getter]` 和 `#[setter]` 函数


### 使用`#[pyo3(get, set)]`的对象性质

对于成员变量只是读取和填写（be read and written）的简单情形，可以用`pyo3`属性在`#[pyclass]`域中声明 getters 和 setters，下面是例子：

```rust
# use pyo3::prelude::*;
#[pyclass]
struct MyClass {
    #[pyo3(get, set)]
    num: i32,
}
```

上述代码使得 `num` 域可以作为一个`self.num` Python性质来读取和覆写。要将这个性质在另一个域可见，和其他选项（options）一起指明这个标注，e.g. `#[pyo3(get, set, name = "custom_name")]`。

通过单独使用 `#[pyo3(get)]` 或 `#[pyo3(set)]`可以使性质变为只读或者只写。

要使用这些标注，域类型必须实现一些转换特征：
- 对 `get`，域类型必须实现 `IntoPy<PyObject>` 和 `Clone`
- 对 `set` ，域类型必须实现`FromPyObject`

### 使用`#[getter]`和`#[setter]`的对象属性

对于没有满足 `#[pyo3(get, set)]` 特征需求，或者需要副作用的情形，可以在一个`#[pymethods]` `impl`块中定义描述符方法（descriptor method）

通过使用 `#[getter]` 和 `#[setter]` 属性，下面是一个例子

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {
    num: i32,
}

#[pymethods]
impl MyClass {
    #[getter]
    fn num(&self) -> PyResult<i32> {
        Ok(self.num)
    }
}
```

一个 getter 或 setter 的函数名默认用作属性名。有几种方法覆写这个名字。

如果 getter 和 setter 函数名分别以 `get_` 和 `set_` 开头，描述符的名字会变成这个前缀去掉后的名字。这对于像 `type` 的 Rust 关键字也有用（[raw identifiers](https://doc.rust-lang.org/edition-guide/rust-2018/module-system/raw-identifiers.html)）。

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {
    num: i32,
}
#[pymethods]
impl MyClass {
    #[getter]
    fn get_num(&self) -> PyResult<i32> {
        Ok(self.num)
    }

    #[setter]
    fn set_num(&mut self, value: i32) -> PyResult<()> {
        self.num = value;
        Ok(())
    }
}
```

这里，定义了性质 `num`，可以在 Python 中用 `self.num` 获取它。

`#[getter]` 和 `#[setter]` 都接收一个参数。如果参数被指定了，它将被用作性质的名字，i.e.

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {
   num: i32,
}
#[pymethods]
impl MyClass {
    #[getter(number)]
    fn num(&self) -> PyResult<i32> {
        Ok(self.num)
    }

    #[setter(number)]
    fn set_num(&mut self, value: i32) -> PyResult<()> {
        self.num = value;
        Ok(())
    }
}
```

这里，定义了性质 `number` 并且在 Python 中可以用 `self.number` 获取它。

通过 `#[setter]` 或 `#[pyo3(set)]` 定义的属性永远会对 `del` 算子抛出 `AttributeError`，要自定义 `del` 参见[#1778](https://github.com/PyO3/pyo3/issues/1778).

## 实例方法

要定义一个 Python 相容的方法，必须用 `#[pymethods]` 方法标注结构的 `impl` 块。PyO3 为所有在这个块里的函数生成与 Python 相容的包装器，像描述符，类方法，静态方法等等。

既然 Rust 允许任意数量的 `impl` 块，可以任意切分方法。但是要对同一结构同时标注多个 `#[pymethods]` 的 `impl` 块，必须使用PyO3的 `multiple-pymethods` 特性。


```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {
    num: i32,
}
#[pymethods]
impl MyClass {
    fn method1(&self) -> PyResult<i32> {
        Ok(10)
    }

    fn set_method(&mut self, value: i32) -> PyResult<()> {
        self.num = value;
        Ok(())
    }
}
```

对这些方法的调用受 GIL保护，所以 `&self` 和 `&mut self` 都可以使用。返回类型必须为 `PyResult<T>` 或者某个实现了 `IntoPy<PyObject>` 的 `T`，后者在方法不会抛出Python异常时是允许的。

一个 `Python` 参数可以作为方法签名的一部分被指定，此时 `py` 变量被方法包装器注入，e.g.

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {
#[allow(dead_code)]
    num: i32,
}
#[pymethods]
impl MyClass {
    fn method2(&self, py: Python<'_>) -> PyResult<i32> {
        Ok(10)
    }
}
```

从 Python 看来，这个例子中的 `method2` 不接受任何变量。

## 类方法

为一个自定义类创建一个类方法，需要对方法标注 `#[classmethod]` 属性。这和Python中的`@classmethod`是等价的。

```rust
use pyo3::prelude::*;
use pyo3::types::PyType;
#[pyclass]
struct MyClass {
    #[allow(dead_code)]
    num: i32,
}
#[pymethods]
impl MyClass {
    #[classmethod]
    fn cls_method(cls: &PyType) -> PyResult<i32> {
        Ok(10)
    }
}
```

声明一个可被Python调用的类方法
- 第一个参数是方法被调用的类的类型对象
- 第一个参数隐式的是 `&PyType` 类型
- 对于 `parameter-list` 的细节，参见 `Method arguments` 章节
- 返回类型必须为 `PyResult<T>` 或者某个实现了 `IntoPy<PyObject>` 的类型 `T`

## 静态方法

要为自定义类创建一个静态方法，需要用 `#[staticmethod]` 属性标注该方法，返回类型必须为`PyResult<T>` 或者某个实现了 `IntoPy<PyObject>` 的类型 `T`

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {
    #[allow(dead_code)]
    num: i32,
}
#[pymethods]
impl MyClass {
    #[staticmethod]
    fn static_method(param1: i32, param2: &str) -> PyResult<i32> {
        Ok(10)
    }
}
```

## 类属性

要创建一个类属性，也称为（`class variable`, `classattr`），可以用 `#[classattr]` 标注一个没有任何变量的方法。

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {}
#[pymethods]
impl MyClass {
    #[classattr]
    fn my_attribute() -> String {
        "hello".to_string()
    }
}

Python::with_gil(|py| {
    let my_class = py.get_type::<MyClass>();
    pyo3::py_run!(py, my_class, "assert my_class.my_attribute == 'hello'")
});
```
> Note: 如果方法有一个 `Result` 返回类型并且返回了 `Err`，PyO3会在类创建过程中 panic


如果只用了 `const` 来定义类属性，也可以标注对应的常量：

```rust
use pyo3::prelude::*;
#[pyclass]
struct MyClass {}
#[pymethods]
impl MyClass {
    #[classattr]
    const MY_CONST_ATTRIBUTE: &'static str = "foobar";
}
```

## 方法变量

类似`#[pyfunction]`，可以用 `#[pyo3(signature = (...))]` 属性来指定 `#[pymethods]` 接收变量的方式，参见方法签名章节。

下述例子定义了一个具有`method`方法的类 `MyClass`。这个方法有一个签名，它为 `num` 和 `name` 设置了默认值，并且表明 `py_args` 会接收所有的位置变量，而 `py_kwargs` 会接收所有的关键字变量：

```rust
use pyo3::prelude::*;
use pyo3::types::{PyDict, PyTuple};

#[pyclass]
struct MyClass {
    num: i32,
}
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
}
```

而在Python中是这样子的

```python
>>> import mymodule
>>> mc = mymodule.MyClass()
>>> print(mc.method(44, False, "World", 666, x=44, y=55))
py_args=('World', 666), py_kwargs=Some({'x': 44, 'y': 55}), name=Hello, num=44, num_before=-1
>>> print(mc.method(num=-1, name="World"))
py_args=(), py_kwargs=None, name=World, num=-1, num_before=44
```

## 使得类方法签名对Python可用

`#[pyfunction]` 的 `text_signature = "..."` 选项对类和方法也是可用的：

```rust
#![allow(dead_code)]
use pyo3::prelude::*;
use pyo3::types::PyType;

// it works even if the item is not documented:
#[pyclass(text_signature = "(c, d, /)")]
struct MyClass {}

#[pymethods]
impl MyClass {
    // the signature for the constructor is attached
    // to the struct definition instead.
    #[new]
    fn new(c: i32, d: &str) -> Self {
        Self {}
    }
    // the self argument should be written $self
    #[pyo3(text_signature = "($self, e, f)")]
    fn my_method(&self, e: i32, f: i32) -> i32 {
        e + f
    }
    #[classmethod]
    #[pyo3(text_signature = "(cls, e, f)")]
    fn my_class_method(cls: &PyType, e: i32, f: i32) -> i32 {
        e + f
    }
    #[staticmethod]
    #[pyo3(text_signature = "(e, f)")]
    fn my_static_method(e: i32, f: i32) -> i32 {
        e + f
    }
}

fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        let inspect = PyModule::import(py, "inspect")?.getattr("signature")?;
        let module = PyModule::new(py, "my_module")?;
        module.add_class::<MyClass>()?;
        let class = module.getattr("MyClass")?;

        if cfg!(not(Py_LIMITED_API)) || py.version_info() >= (3, 10)  {
            let doc: String = class.getattr("__doc__")?.extract()?;
            assert_eq!(doc, "");

            let sig: String = inspect
                .call1((class,))?
                .call_method0("__str__")?
                .extract()?;
            assert_eq!(sig, "(c, d, /)");
       } else {
            let doc: String = class.getattr("__doc__")?.extract()?;
            assert_eq!(doc, "");

            inspect.call1((class,)).expect_err("`text_signature` on classes is not compatible with compilation in `abi3` mode until Python 3.10 or greater");
         }

        {
            let method = class.getattr("my_method")?;

            assert!(method.getattr("__doc__")?.is_none());

            let sig: String = inspect
                .call1((method,))?
                .call_method0("__str__")?
                .extract()?;
            assert_eq!(sig, "(self, /, e, f)");
        }

        {
            let method = class.getattr("my_class_method")?;

            assert!(method.getattr("__doc__")?.is_none());

            let sig: String = inspect
                .call1((method,))?
                .call_method0("__str__")?
                .extract()?;
            assert_eq!(sig, "(cls, e, f)");
        }

        {
            let method = class.getattr("my_static_method")?;

            assert!(method.getattr("__doc__")?.is_none());

            let sig: String = inspect
                .call1((method,))?
                .call_method0("__str__")?
                .extract()?;
            assert_eq!(sig, "(e, f)");
        }

        Ok(())
    })
}
```
注意到在`abi3`模式下编译时，类的 `text_signature` 只在 Python 3.10 或更高的版本相容。

## #[pyclass]枚举

目前PyO3只支持 fieldless 枚举。PyO3对每个变量添加了类属性，所以你不用定义 `#[new]` 就可以在Python中获取它们。PyO3同样提供了 `__richcmp__` 和 `__int__` 的默认实现，所以它们可以用 `==` 来进行比较：

```rust
use pyo3::prelude::*;
#[pyclass]
enum MyEnum {
    Variant,
    OtherVariant,
}

Python::with_gil(|py| {
    let x = Py::new(py, MyEnum::Variant).unwrap();
    let y = Py::new(py, MyEnum::OtherVariant).unwrap();
    let cls = py.get_type::<MyEnum>();
    pyo3::py_run!(py, x y cls, r#"
        assert x == cls.Variant
        assert y == cls.OtherVariant
        assert x != y
    "#)
})
```

也可以将枚举转换为 `int`：

```rust
use pyo3::prelude::*;
#[pyclass]
enum MyEnum {
    Variant,
    OtherVariant = 10,
}

Python::with_gil(|py| {
    let cls = py.get_type::<MyEnum>();
    let x = MyEnum::Variant as i32; // The exact value is assigned by the compiler.
    pyo3::py_run!(py, cls x, r#"
        assert int(cls.Variant) == x
        assert int(cls.OtherVariant) == 10
        assert cls.OtherVariant == 10  # You can also compare against int.
        assert 10 == cls.OtherVariant
    "#)
})
```

PyO3 也为枚举提供了 `__repr__`

```rust
use pyo3::prelude::*;
#[pyclass]
enum MyEnum{
    Variant,
    OtherVariant,
}

Python::with_gil(|py| {
    let cls = py.get_type::<MyEnum>();
    let x = Py::new(py, MyEnum::Variant).unwrap();
    pyo3::py_run!(py, cls x, r#"
        assert repr(x) == 'MyEnum.Variant'
        assert repr(cls.OtherVariant) == 'MyEnum.OtherVariant'
    "#)
})
```

PyO3定义的所有方法可以被覆写，例如想要覆写 `__repr__`：

```rust
use pyo3::prelude::*;
#[pyclass]
enum MyEnum {
    Answer = 42,
}

#[pymethods]
impl MyEnum {
    fn __repr__(&self) -> &'static str {
        "42"
    }
}

Python::with_gil(|py| {
    let cls = py.get_type::<MyEnum>();
    pyo3::py_run!(py, cls, "assert repr(cls.Answer) == '42'")
})
```

枚举以及其变量也能使用 `#[pyo3(name)]` 来重命名：

```rust
use pyo3::prelude::*;
#[pyclass(name = "RenamedEnum")]
enum MyEnum {
    #[pyo3(name = "UPPERCASE")]
    Variant,
}

Python::with_gil(|py| {
    let x = Py::new(py, MyEnum::Variant).unwrap();
    let cls = py.get_type::<MyEnum>();
    pyo3::py_run!(py, x cls, r#"
        assert repr(x) == 'RenamedEnum.UPPERCASE'
        assert x == cls.UPPERCASE
    "#)
})
```

不能使用枚举作为基类或者从其他类进行继承：

```rust
use pyo3::prelude::*;
#[pyclass(subclass)]
enum BadBase {
    Var1,
}
```

```rust
use pyo3::prelude::*;

#[pyclass(subclass)]
struct Base;

#[pyclass(extends=Base)]
enum BadSubclass {
    Var1,
}
```

在Python中，`#[pyclass]` 枚举目前还不能与`IntEnum` 一起用。

## 实现的细节

`#[pyclass]`宏依赖许多 conditional code generation，每个 `#[pyclass]` 可以选择性地拥有一个 `#[pymethods]` 块。

>To support this flexibility the `#[pyclass]` macro expands to a blob of boilerplate code which sets up the structure for ["dtolnay specialization"](https://github.com/dtolnay/case-studies/blob/master/autoref-specialization/README.md). This implementation pattern enables the Rust compiler to use `#[pymethods]` implementations when they are present, and fall back to default (empty) definitions when they are not.
>
>This simple technique works for the case when there is zero or one implementations. To support multiple `#[pymethods]` for a `#[pyclass]` (in the [`multiple-pymethods`] feature), a registry mechanism provided by the [`inventory`](https://github.com/dtolnay/inventory) crate is used instead. This collects `impl`s at library load time, but isn't supported on all platforms. See [inventory: how it works](https://github.com/dtolnay/inventory#how-it-works) for more details.
>
>The `#[pyclass]` macro expands to roughly the code seen below. The `PyClassImplCollector` is the type used internally by PyO3 for dtolnay specialization:

```rust
#[cfg(not(feature = "multiple-pymethods"))] {
# use pyo3::prelude::*;
// Note: the implementation differs slightly with the `multiple-pymethods` feature enabled.
struct MyClass {
    # #[allow(dead_code)]
    num: i32,
}
unsafe impl pyo3::type_object::PyTypeInfo for MyClass {
    type AsRefTarget = pyo3::PyCell<Self>;
    const NAME: &'static str = "MyClass";
    const MODULE: ::std::option::Option<&'static str> = ::std::option::Option::None;
    #[inline]
    fn type_object_raw(py: pyo3::Python<'_>) -> *mut pyo3::ffi::PyTypeObject {
        <Self as pyo3::impl_::pyclass::PyClassImpl>::lazy_type_object()
            .get_or_init(py)
            .as_type_ptr()
    }
}

impl pyo3::PyClass for MyClass {
    type Frozen = pyo3::pyclass::boolean_struct::False;
}

impl<'a, 'py> pyo3::impl_::extract_argument::PyFunctionArgument<'a, 'py> for &'a MyClass
{
    type Holder = ::std::option::Option<pyo3::PyRef<'py, MyClass>>;

    #[inline]
    fn extract(obj: &'py pyo3::PyAny, holder: &'a mut Self::Holder) -> pyo3::PyResult<Self> {
        pyo3::impl_::extract_argument::extract_pyclass_ref(obj, holder)
    }
}

impl<'a, 'py> pyo3::impl_::extract_argument::PyFunctionArgument<'a, 'py> for &'a mut MyClass
{
    type Holder = ::std::option::Option<pyo3::PyRefMut<'py, MyClass>>;

    #[inline]
    fn extract(obj: &'py pyo3::PyAny, holder: &'a mut Self::Holder) -> pyo3::PyResult<Self> {
        pyo3::impl_::extract_argument::extract_pyclass_ref_mut(obj, holder)
    }
}

impl pyo3::IntoPy<PyObject> for MyClass {
    fn into_py(self, py: pyo3::Python<'_>) -> pyo3::PyObject {
        pyo3::IntoPy::into_py(pyo3::Py::new(py, self).unwrap(), py)
    }
}

impl pyo3::impl_::pyclass::PyClassImpl for MyClass {
    const DOC: &'static str = "Class for demonstration\u{0}";
    const IS_BASETYPE: bool = false;
    const IS_SUBCLASS: bool = false;
    type Layout = PyCell<MyClass>;
    type BaseType = PyAny;
    type ThreadChecker = pyo3::impl_::pyclass::ThreadCheckerStub<MyClass>;
    type PyClassMutability = <<pyo3::PyAny as pyo3::impl_::pyclass::PyClassBaseType>::PyClassMutability as pyo3::impl_::pycell::PyClassMutability>::MutableChild;
    type Dict = pyo3::impl_::pyclass::PyClassDummySlot;
    type WeakRef = pyo3::impl_::pyclass::PyClassDummySlot;
    type BaseNativeType = pyo3::PyAny;

    fn items_iter() -> pyo3::impl_::pyclass::PyClassItemsIter {
        use pyo3::impl_::pyclass::*;
        let collector = PyClassImplCollector::<MyClass>::new();
        static INTRINSIC_ITEMS: PyClassItems = PyClassItems { slots: &[], methods: &[] };
        PyClassItemsIter::new(&INTRINSIC_ITEMS, collector.py_methods())
    }

    fn lazy_type_object() -> &'static pyo3::impl_::pyclass::LazyTypeObject<MyClass> {
        use pyo3::impl_::pyclass::LazyTypeObject;
        static TYPE_OBJECT: LazyTypeObject<MyClass> = LazyTypeObject::new();
        &TYPE_OBJECT
    }
}

Python::with_gil(|py| {
    let cls = py.get_type::<MyClass>();
    pyo3::py_run!(py, cls, "assert cls.__name__ == 'MyClass'")
});
}
```
