# Python 异常

## 定义一个新异常

可以使用`create_exception!`宏来定义一个新异常类型

```rust
use pyo3::create_exception;

create_exception!(module, MyError, pyo3::exceptions::PyException);
```

- `module` 是包含它的模组名
- `MyError` 是新异常类型名

例如

```rust
use pyo3::prelude::*;
use pyo3::create_exception;
use pyo3::types::IntoPyDict;
use pyo3::exceptions::PyException;

create_exception!(mymodule, CustomError, PyException);

Python::with_gil(|py| {
    let ctx = [("CustomError", py.get_type::<CustomError>())].into_py_dict(py);
    pyo3::py_run!(
        py,
        *ctx,
        "assert str(CustomError) == \"<class 'mymodule.CustomError'>\""
    );
    pyo3::py_run!(py, *ctx, "assert CustomError('oops').args == ('oops',)");
});
```

当使用 PyO3 来创建一个额外模组时，可以像这样在该模组上添加一个新异常，从而能在 Python 中导入它

```rust
use pyo3::prelude::*;
use pyo3::types::PyModule;
use pyo3::exceptions::PyException;

pyo3::create_exception!(mymodule, CustomError, PyException);

#[pymodule]
fn mymodule(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    // ... other elements added to module ...
    m.add("CustomError", py.get_type::<CustomError>())?;

    Ok(())
}
```

## 抛出异常

像在函数/错误处理中说的那样，要从`#[pyfunction]` 或 `#[pymethods]`中抛出一个异常需要返回一个`Err(PyErr)`。PyO3 在返回这个结果给 Python 时会自动抛出这个异常。

也可以manually write and fetch errors in the Python interpreter's global state：

```rust
use pyo3::{Python, PyErr};
use pyo3::exceptions::PyTypeError;

Python::with_gil(|py| {
    PyTypeError::new_err("Error").restore(py);
    assert!(PyErr::occurred(py));
    drop(PyErr::fetch(py));
});
```

## 检查异常类型

Python 有一个[`isinstance`](https://docs.python.org/3/library/functions.html#isinstance)方法来检查一个对象的类型。在 PyO3 中，每个对象都有具有同样效果的`PyAny::is_instance` 和 `PyAny::is_instance_of`方法。

```rust
use pyo3::Python;
use pyo3::types::{PyBool, PyList};

Python::with_gil(|py| {
    assert!(PyBool::new(py, true).is_instance_of::<PyBool>().unwrap());
    let list = PyList::new(py, &[1, 2, 3, 4]);
    assert!(!list.is_instance_of::<PyBool>().unwrap());
    assert!(list.is_instance_of::<PyList>().unwrap());
});
```

要检查一个异常的类型，类似地可以

```rust
# use pyo3::exceptions::PyTypeError;
# use pyo3::prelude::*;
# Python::with_gil(|py| {
# let err = PyTypeError::new_err(());
err.is_instance_of::<PyTypeError>(py);
# });
```

## 使用 Python 中定义的异常

可以用原生 Rust 类型来使用定义在 Python 代码中的异常。`import_exception!`宏允许导入一个特定的异常类型并为其定义一个 Rust 类型

```rust
#![allow(dead_code)]
use pyo3::prelude::*;

mod io {
    pyo3::import_exception!(io, UnsupportedOperation);
}

fn tell(file: &PyAny) -> PyResult<u64> {
    match file.call_method0("tell") {
        Err(_) => Err(io::UnsupportedOperation::new_err("not supported: tell")),
        Ok(x) => x.extract::<u64>(),
    }
}
```