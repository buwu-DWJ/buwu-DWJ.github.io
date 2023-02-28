# Python modules

可以用 `#[pymodule]` 创建一个模组

```rust
use pyo3::prelude::*;

#[pyfunction]
fn double(x: usize) -> usize {
    x * 2
}

/// This module is implemented in Rust.
#[pymodule]
fn my_extension(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(double, m)?)?;
    Ok(())
}

```

模组的名字会默认为 Rust 函数的名字（`fn my_extenssion`），如果要修改可以使用 `#[pyo3(name = "custom_name")]`

```rust
use pyo3::prelude::*;

#[pyfunction]
fn double(x: usize) -> usize {
    x * 2
}

#[pymodule]
#[pyo3(name = "custom_name")]
fn my_extension(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(double, m)?)?;
    Ok(())
}
```
模组的名字必须和 `.so` 或者 `.pyd` 一致。

## 注释

模组和其中函数都是在 `#[pyfunction]` 或 `#[pymodule]` 上方通过 `///`加入。调用是通过 `module_name.__doc__` 或者编辑器自动显示。


## 子模组

可以使用 `PyModule.add_submodule()`添加子模组

```rust
use pyo3::prelude::*;

#[pymodule]
fn parent_module(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    register_child_module(py, m)?;
    Ok(())
}

fn register_child_module(py: Python<'_>, parent_module: &PyModule) -> PyResult<()> {
    let child_module = PyModule::new(py, "child_module")?;
    child_module.add_function(wrap_pyfunction!(func, child_module)?)?;
    parent_module.add_submodule(child_module)?;
    Ok(())
}

#[pyfunction]
fn func() -> String {
    "func".to_string()
}
```

这并没有定义一个包，所以不支持直接通过`from parent_module import child_module`导入子模块，更多信息参考 [#759](https://github.com/PyO3/pyo3/issues/759)和[#1517](https://github.com/PyO3/pyo3/issues/1517#issuecomment-808664021)。
