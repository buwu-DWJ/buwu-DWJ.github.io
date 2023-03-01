# 自定义基本对象

回忆上一章中的`Number`类

```rust
use pyo3::prelude::*;

#[pyclass]
struct Number(i32);

#[pymethods]
impl Number {
    #[new]
    fn new(value: i32) -> Self {
        Self(value)
    }
}

#[pymodule]
fn my_module(_py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_class::<Number>()?;
    Ok(())
}
```

目前Python可以导入这个模组，获取到这个类，并创建实例——没别的了。

```python
from my_module import Number

n = Number(5)
print(n)
```

```text
<builtins.Number object at 0x000002B4D185D7D0>
```

### 字符串表示String representations

它甚至不能打印一个用户可见的它自身的说明！我们可以在`#[pymethods]`块中加上`__repr__` 和 `__str__` 方法，用这个来获得`Number`中包含的值。methods inside a

```rust
# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    // For `__repr__` we want to return a string that Python code could use to recreate
    // the `Number`, like `Number(5)` for example.
    fn __repr__(&self) -> String {
        // We use the `format!` macro to create a string. Its first argument is a
        // format string, followed by any number of parameters which replace the
        // `{}`'s in the format string.
        //
        //                       👇 Tuple field access in Rust uses a dot
        format!("Number({})", self.0)
    }

    // `__str__` is generally used to create an "informal" representation, so we
    // just forward to `i32`'s `ToString` trait implementation to print a bare number.
    fn __str__(&self) -> String {
        self.0.to_string()
    }
}
```

### 哈希Hashing

接下来实现哈希。我们仅仅哈希`i32`，为此我们需要一个[`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)，`std`提供的是[`DefaultHasher`](https://doc.rust-lang.org/std/collections/hash_map/struct.DefaultHasher.html)，使用的是[SipHash](https://en.wikipedia.org/wiki/SipHash)算法。

```rust
use std::collections::hash_map::DefaultHasher;

// Required to call the `.hash` and `.finish` methods, which are defined on traits.
use std::hash::{Hash, Hasher};

# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __hash__(&self) -> u64 {
        let mut hasher = DefaultHasher::new();
        self.0.hash(&mut hasher);
        hasher.finish()
    }
}
```

> **Note**: 当实现 `__hash__` 和比较时, 下属性质需要满足
>
> ```text
> k1 == k2 -> hash(k1) == hash(k2)
> ```
>
> 也就是说两个键值相等时，它们的哈希也要相等。
> 另外必须注意类的哈希不能在生命周期中改变。在这里的教程中，我们不让Python代码改变我们的`Number`类，也就是说，它是不可变的。
> 默认情况下，所有 `#[pyclass]` 类型在Python中有一个默认的哈希实现。
> 不需要被哈希的类型需要改写`__hash__`为`None`
> ```rust
> # use pyo3::prelude::*;
> #[pyclass]
> struct NotHashable {}
>
> #[pymethods]
> impl NotHashable {
>     #[classattr]
>     const __hash__: Option<Py<PyAny>> = None;
> }
> ```

### 比较Comparisons

不像在Python中，PyO3不提供类似`__eq__`, `__lt__`的魔术比较方法。你需要通过`__richcmp__`一次性实现所有六个算子。
Unlike in Python, PyO3 does not provide the magic comparison methods you might expect like `__eq__`, 这个方法会根据算子的`CompareOp`值被调用

```rust
use pyo3::class::basic::CompareOp;

# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __richcmp__(&self, other: &Self, op: CompareOp) -> PyResult<bool> {
        match op {
            CompareOp::Lt => Ok(self.0 < other.0),
            CompareOp::Le => Ok(self.0 <= other.0),
            CompareOp::Eq => Ok(self.0 == other.0),
            CompareOp::Ne => Ok(self.0 != other.0),
            CompareOp::Gt => Ok(self.0 > other.0),
            CompareOp::Ge => Ok(self.0 >= other.0),
        }
    }
}
```

如果通过比较两个Rust值获得了结果，可以采取一个捷径`CompareOp::matches`

```rust
use pyo3::class::basic::CompareOp;

# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __richcmp__(&self, other: &Self, op: CompareOp) -> bool {
        op.matches(self.0.cmp(&other.0))
    }
}
```

它会检查Rust的 `Ord` 中得到的 `std::cmp::Ordering` 是否匹配给定的`CompareOp`。

另外，如果想要保留一些算子不被实现，可以返回`py.NotImplemented()`

```rust
use pyo3::class::basic::CompareOp;

# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __richcmp__(&self, other: &Self, op: CompareOp, py: Python<'_>) -> PyObject {
        match op {
            CompareOp::Eq => (self.0 == other.0).into_py(py),
            CompareOp::Ne => (self.0 != other.0).into_py(py),
            _ => py.NotImplemented(),
        }
    }
}
```

### 真伪Truthyness

考虑`Number`是`True`如果它不是零

```rust
# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __bool__(&self) -> bool {
        self.0 != 0
    }
}
```

### 最终的代码

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

use pyo3::prelude::*;
use pyo3::class::basic::CompareOp;

#[pyclass]
struct Number(i32);

#[pymethods]
impl Number {
    #[new]
    fn new(value: i32) -> Self {
        Self(value)
    }

    fn __repr__(&self) -> String {
        format!("Number({})", self.0)
    }

    fn __str__(&self) -> String {
        self.0.to_string()
    }

    fn __hash__(&self) -> u64 {
        let mut hasher = DefaultHasher::new();
        self.0.hash(&mut hasher);
        hasher.finish()
    }

    fn __richcmp__(&self, other: &Self, op: CompareOp) -> PyResult<bool> {
        match op {
            CompareOp::Lt => Ok(self.0 < other.0),
            CompareOp::Le => Ok(self.0 <= other.0),
            CompareOp::Eq => Ok(self.0 == other.0),
            CompareOp::Ne => Ok(self.0 != other.0),
            CompareOp::Gt => Ok(self.0 > other.0),
            CompareOp::Ge => Ok(self.0 >= other.0),
        }
    }

    fn __bool__(&self) -> bool {
        self.0 != 0
    }
}

#[pymodule]
fn my_module(_py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_class::<Number>()?;
    Ok(())
}
```

