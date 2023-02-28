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