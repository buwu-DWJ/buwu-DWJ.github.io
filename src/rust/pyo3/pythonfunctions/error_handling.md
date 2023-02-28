# 错误处理（Error handling）

## 表示Python中错误

Rust 代码使用 `Result<T, E>`枚举来表示异常。错误类型 `E` 由代码作者指定。

PyO3 由 `PyErr` 类型来表示 Python 中的异常。如果 PyO3 API 可以抛出一个 Python 异常，那么那个 API 的返回类型会是 `PyResult<T>`，这是类型 `Result<T, PyErr>`的别名。

总的来说
- 当 Python 异常被抛出并被 PtO3 获取到，这个异常信息会被保存在 `PyResult` 的 `Err` 变量中
- 通过 Rust 代码传递 Python 异常，然后使用通常的技术来处理（例如`?`算子，`PyErr`作为错误类型）
- 最后，当一个 `PyResult`从 Rust 通过 PyO3 返回到 Python 中时，如果结果是`Err`类型变量，那么其包含的异常会被抛出

## 在函数中抛出异常

如上所说，当一个 `PyResult`从 Rust 通过 PyO3 返回到 Python 中时，如果结果是`Err`类型变量，那么其包含的异常会被抛出。

那么，为了在一个`#[pyfunction]`中抛出异常，要把返回类型从`T`转为`PyResult<T>`。当函数返回一个`Err`，它会抛出一个 Python 异常。这对于`#[pymethods]`中的函数也有效。

例如，下述的 `check_positive` 函数当输入是负值时抛出一个 `ValueError`：

```rust
use pyo3::exceptions::PyValueError;
use pyo3::prelude::*;

#[pyfunction]
fn check_positive(x: i32) -> PyResult<()> {
    if x < 0 {
        Err(PyValueError::new_err("x is negative"))
    } else {
        Ok(())
    }
}
```

所有 Python 内置的异常类型都定义在 `pyo3::exceptions` 模块中，它们有一个 `new_err` 构造器来直接构造一个 `PyErr`。

## Rust 错误类型

PyO3 自动将 `#[pyfunction]` 返回的 `Result<T, E>` 转换为 `PyResult<T>`，只要在 `std::from::From<E> for PyErr` 中有其实现。在 Rust 标准库中有许多这样定义的 `From` 转换的错误类型。

如果你处理的错误类型`E`来自第三方包，看本章的 foreign rust error types。

下述例子使用了 `PyErr` 的 `From<ParseIntError>`实现来抛出异常：parsing strings as integers
```rust
use std::num::ParseIntError;

#[pyfunction]
fn parse_int(x: &str) -> Result<usize, ParseIntError> {
    x.parse()
}
```

当传入一个不包含 floating-point number 的字符串，异常如下

```console
>>> parse_int("bar")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid digit found in string
```

用一个更复杂的例子，下述代码定义了一个 Rust 错误名为 `CustomIOError`。然后定义了一个 `PyErr` 下的 `From<CustomIOError>`，它会返回一个 `PyErr` 代表 Python 的 `OSError`。因此，可以直接在 `#[pyfunction]` 中使用这个错误。

```rust
use pyo3::exceptions::PyOSError;
use pyo3::prelude::*;
use std::fmt;

#[derive(Debug)]
struct CustomIOError;

impl std::error::Error for CustomIOError {}

impl fmt::Display for CustomIOError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Oh no!")
    }
}

impl std::convert::From<CustomIOError> for PyErr {
    fn from(err: CustomIOError) -> PyErr {
        PyOSError::new_err(err.to_string())
    }
}

pub struct Connection {/* ... */}

fn bind(addr: String) -> Result<Connection, CustomIOError> {
    if &addr == "0.0.0.0" {
        Err(CustomIOError)
    } else {
        Ok(Connection{ /* ... */})
    }
}

#[pyfunction]
fn connect(s: String) -> Result<(), CustomIOError> {
    bind(s)?;
    // etc.
    Ok(())
}

fn main() {
    Python::with_gil(|py| {
        let fun = pyo3::wrap_pyfunction!(connect, py).unwrap();
        let err = fun.call1(("0.0.0.0",)).unwrap_err();
        assert!(err.is_instance_of::<PyOSError>(py));
    });
}
```

任何有 `From` 转换的错误 `E` 可以用在 `?` 算子中。上述代码中 `parse_int` 的一个会返回 `PyResult` 的可选实现如下：
```rust
use pyo3::prelude::*;

fn parse_int(s: String) -> PyResult<usize> {
    let x = s.parse()?;
    Ok(x)
}
```

## 第三方 Rust 错误类型（Foreign Rust error types）

The Rust compiler will not permit implementation of traits for types outside of the crate where the type is defined. (This is known as the "orphan rule".)

Given a type OtherError which is defined in third-party code, there are two main strategies available to integrate it with PyO3:

- Create a newtype wrapper, e.g. `MyOtherError`. Then implement `From<MyOtherError>` for `PyErr` (or `PyErrArguments`), as well as `From<OtherError>` for `MyOtherError`.
- Use Rust's Result combinators such as `map_err` to write code freely to convert `OtherError` into whatever is needed. This requires boilerplate at every usage however gives unlimited flexibility.

To detail the newtype strategy a little further, the key trick is to return `Result<T, MyOtherError>` from the `#[pyfunction]`. This means that PyO3 will make use of `From<MyOtherError>` for `PyErr` to create Python exceptions while the `#[pyfunction]` implementation can use `?` to convert `OtherError` to `MyOtherError` automatically.

The following example demonstrates this for some imaginary third-party crate `some_crate` with a function `get_x` returning `Result<i32, OtherError>`:

```rust

use pyo3::prelude::*;
use pyo3::exceptions::PyValueError;
use some_crate::{OtherError, get_x};

struct MyOtherError(OtherError);

impl From<MyOtherError> for PyErr {
    fn from(error: MyOtherError) -> Self {
        PyValueError::new_err(error.0.message())
    }
}

impl From<OtherError> for MyOtherError {
    fn from(other: OtherError) -> Self {
        Self(other)
    }
}

#[pyfunction]
fn wrapped_get_x() -> Result<i32, MyOtherError> {
    // get_x is a function returning Result<i32, OtherError>
    let x: i32 = get_x()?;
    Ok(x)
}
```