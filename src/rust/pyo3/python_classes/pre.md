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

为了整合 Rust 类型与 Python，PyO3 需要对可以用 `#[pyclass]` 注释的类型加以限制。特别地，它们**不能有生命周期**的参数，**不能有泛型**参数，**必须实现了 `send`**。理由如下

#### 没有生命周期

Rust 编译器为了内存安全使用生命周期，它们只在编译中使用，无法传输到 Python 这样的动态语言中。

#### 没有泛型参数

一个带有泛型参数 `T` 的 Rust 结构 `Foo<T>` ，每次传入一个不同的具体类型的`T`时，会生成新的编译实现。这在 Python 中并不现实，因为 Python 中编译器只需要 `Foo` 的一个单独的实现。

#### 必须要有`Send`

因为 Python 解释器中，Python objects 在线程中是免费共享的，不能保证哪个线程会最终丢弃这个 object。因此，每个用 `#[pyclass]` 注释的类型必须实现 `Send`（除非注释 `#[pyclass(unsendable)]`）。

