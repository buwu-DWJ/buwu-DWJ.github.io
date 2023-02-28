# Python functions

`#[pyfunction]`用来通过一个 Rust 函数定义 Python 函数。一旦定义了，需要通过 `wrap_pyfunction!` 宏来添加到一个 module 中。

下述例子定义了一个在 Python 模组 `my_extension` 中名为 `double` 的函数

```rust
use pyo3::prelude::*;

#[pyfunction]
fn double(x: usize) -> usize {
    x * 2
}

#[pymodule]
fn my_extension(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(double, m)?)?;
    Ok(())
}
```

## 函数选项（Function options）

`#[pyo3]`属性可以用来改造生成的 Python 函数的性质。可以采用如下选项的任意组合

- `#[pyo3(name = "...")]`：覆盖传递给 Python 的函数名，

```rust
use pyo3::prelude::*;

#[pyfunction]
#[pyo3(name = "no_args")]
fn no_args_py() -> usize {
    42
}

#[pymodule]
fn module_with_functions(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(no_args_py, m)?)?;
    Ok(())
}
```

- `#[pyo3(signature = (...))]`：定义 Python 中函数的签名，见本页 Function Signatures
- `#[pyo3(text_signature = "...")]`：覆盖在 Python 工具中可见的由 PyO3 生成的函数签名（例如通过`inspect.signature`）
- `#[pyo3(pass_module)]`：让 PyO3 将 "pass_module" 作为第一个变量传递给函数，那么就能够在函数体中使用该模组，该变量类型必须为 `&PyModule`。下述例子将创建一个函数`pyfunction_with_module`，它会返回它包含的模组的名称（i.e. `module_with_fn`）：
```rust
use pyo3::prelude::*;

#[pyfunction]
#[pyo3(pass_module)]
fn pyfunction_with_module(module: &PyModule) -> PyResult<&str> {
    module.name()
}

#[pymodule]
fn module_with_fn(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(pyfunction_with_module, m)?)
}

```

## 变量单独选项（Per-argument options）

`#[pyo3]`也可以用在单独的变量中来改变其性质，它可以是下述选项的任意组合：

- `#[pyo3(from_py_with = "...")]`：将一个本地函数从 Python 形式转化为 Rust 形式的函数参数，取代使用默认的 `FromPyObject`。函数签名必须为`fn(&PyAny) -> PyResult<T>`，其中 `T` 必须为 Rust 类型的变量。下述例子使用`from_py_with`将输入的 Python 对象转换为它的长度
```rust
use pyo3::prelude::*;

fn get_length(obj: &PyAny) -> PyResult<usize> {
    let length = obj.len()?;
    Ok(length)
}

#[pyfunction]
fn object_length(
    #[pyo3(from_py_with = "get_length")] argument: usize
) -> usize {
    argument
}
```

## 高级函数模式（Advanced function patterns）

### 在 Rust 中调用 Python 函数

可以将 Python 中定义的函数或者内置函数传递为 Rust 函数 （`PyFunction`-常规Python函数，`PyCFunction`-内置函数）`repr()`。

也可以使用 `PyAny::is_callable` 来检查是否有一个可调用的（callable） 对象。 `is_callable`会返回`true`若函数（包括 lambdas），方法和 objects 带有 `__call__` 方法。可以用 `PyAny::call` 调用对象，args作为第一个参数，kwargs作为第二个参数。另外没参数时使用 `PyAny::call0`，只有 positional 参数时使用`PyAny::call1`。

### 在 Python 中调用 Rust 函数

将 Rust 函数转换为 Python 对象的方法取决于函数：
- Named functions，e.g. `fn foo():`添加`#[pyfunction]` 然后使用 `wrap_pyfunction!`得到对应的`PyCFunction`
- 匿名函数（Anonymous functions）或者闭包（closures）e.g. `foo: fn()`：
  - 使用 `#[pyclass]` 结构将函数保存为一个域（field）并用`__call__`来调用保存的函数
  - 使用 `PyCFunction::new_closure` 来从函数直接创建

### 获得FFI函数

为了使 Rust 函数在 Python 中可调用，PyO3生成一个外部的 "C" 函数，具体签名依赖于 Rust的签名。它将对于 Rust 的调用嵌入在这个 FFI-包装器（wrapper）函数中，这个包装器负责从输入`PyObject`中提取一般参数（regular argument）和关键字参数（keyword argument）。

`wrap_pyfunction`宏能用来根据给定的`#[pyfunction]`和一个`PyModule: wrap_pyfunction!(rust_fun, module)`直接得到一个`PyCFunction`。

## #[pyfn]简写

有一种对`#[pyfunction]`和`wrap_pymodule`的简写：函数可以放置在模组的定义中并用`#[pyfn]`声明，但未来可能被移除
```rust
use pyo3::prelude::*;

#[pymodule]
fn my_extension(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    #[pyfn(m)]
    fn double(x: usize) -> usize {
        x * 2
    }

    Ok(())
}

```
`#[pyfn(m)]` 只是 `#[pyfunction]`的语法糖

```rust
use pyo3::prelude::*;

#[pymodule]
fn my_extension(py: Python<'_>, m: &PyModule) -> PyResult<()> {
    #[pyfunction]
    fn double(x: usize) -> usize {
        x * 2
    }

    m.add_function(wrap_pyfunction!(double, m)?)?;
    Ok(())
}

```