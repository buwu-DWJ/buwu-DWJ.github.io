# 模拟数值类型

目前为止我们有了一个`Number`类但我们实际上不能做任何数学运算！

在继续前，我们先要考虑如何来处理溢出（overflows），有三种解决方式
- 可以像Python中的`int`一样有无限的精确度，但这会非常无聊，我们在重复造轮子
- 我们可以在`Nmuber`溢出时抛出异常，但这会使API很难用
- 我们可以按照`i32`的边界进行包装，这也是下面将要做的。我们要沿着`i32`的`wrapping`方法

## 改造我们的构造器

现在来对付第一个溢出，在`Number`的构造器中

```python
from my_module import Number

n = Number(1 << 1337)
```

```text
Traceback (most recent call last):
  File "example.py", line 3, in <module>
    n = Number(1 << 1337)
OverflowError: Python int too large to convert to C long
```

除了靠默认的`FromPyObject`异常来语法分析变量，我们可以使用`#[pyo3(from_py_with = "...")]`属性来特别指定我们自己的异常函数。不幸的是，PyO3不能在 box 外包装 Python的整数，但我们可以做一个 Python 调用来掩饰它并把它传成一个`i32`

```rust
# #![allow(dead_code)]
use pyo3::prelude::*;

fn wrap(obj: &PyAny) -> Result<i32, PyErr> {
    let val = obj.call_method1("__and__", (0xFFFFFFFF_u32,))?;
    let val: u32 = val.extract()?;
    //     👇 This intentionally overflows!
    Ok(val as i32)
}
```

通过`///`和`#[pyo3(text_signature = "...")]`属性加上文档，都可以被Python用户看到

```rust
# #![allow(dead_code)]
use pyo3::prelude::*;

fn wrap(obj: &PyAny) -> Result<i32, PyErr> {
    let val = obj.call_method1("__and__", (0xFFFFFFFF_u32,))?;
    let val: u32 = val.extract()?;
    Ok(val as i32)
}

/// Did you ever hear the tragedy of Darth Signed The Overfloweth? I thought not.
/// It's not a story C would tell you. It's a Rust legend.
#[pyclass(module = "my_module")]
#[pyo3(text_signature = "(int)")]
struct Number(i32);

#[pymethods]
impl Number {
    #[new]
    fn new(#[pyo3(from_py_with = "wrap")] value: i32) -> Self {
        Self(value)
    }
}
```

有了这些，来实现几个算子

```rust
use std::convert::TryInto;
use pyo3::exceptions::{PyZeroDivisionError, PyValueError};

# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __add__(&self, other: &Self) -> Self {
        Self(self.0.wrapping_add(other.0))
    }

    fn __sub__(&self, other: &Self) -> Self {
        Self(self.0.wrapping_sub(other.0))
    }

    fn __mul__(&self, other: &Self) -> Self {
        Self(self.0.wrapping_mul(other.0))
    }

    fn __truediv__(&self, other: &Self) -> PyResult<Self> {
        match self.0.checked_div(other.0) {
            Some(i) => Ok(Self(i)),
            None => Err(PyZeroDivisionError::new_err("division by zero")),
        }
    }

    fn __floordiv__(&self, other: &Self) -> PyResult<Self> {
        match self.0.checked_div(other.0) {
            Some(i) => Ok(Self(i)),
            None => Err(PyZeroDivisionError::new_err("division by zero")),
        }
    }

    fn __rshift__(&self, other: &Self) -> PyResult<Self> {
        match other.0.try_into() {
            Ok(rhs) => Ok(Self(self.0.wrapping_shr(rhs))),
            Err(_) => Err(PyValueError::new_err("negative shift count")),
        }
    }

    fn __lshift__(&self, other: &Self) -> PyResult<Self> {
        match other.0.try_into() {
            Ok(rhs) => Ok(Self(self.0.wrapping_shl(rhs))),
            Err(_) => Err(PyValueError::new_err("negative shift count")),
        }
    }
}
```

## 一元数值算子

```rust
# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
#[pymethods]
impl Number {
    fn __pos__(slf: PyRef<'_, Self>) -> PyRef<'_, Self> {
        slf
    }

    fn __neg__(&self) -> Self {
        Self(-self.0)
    }

    fn __abs__(&self) -> Self {
        Self(self.0.abs())
    }

    fn __invert__(&self) -> Self {
        Self(!self.0)
    }
}
```

## 对内置函数 `complex()`, `int()` 和 `float()`的支持.

```rust
# use pyo3::prelude::*;
#
# #[pyclass]
# struct Number(i32);
#
use pyo3::types::PyComplex;

#[pymethods]
impl Number {
    fn __int__(&self) -> i32 {
        self.0
    }

    fn __float__(&self) -> f64 {
        self.0 as f64
    }

    fn __complex__<'py>(&self, py: Python<'py>) -> &'py PyComplex {
        PyComplex::from_doubles(py, self.0 as f64, 0.0)
    }
}
```

我们没有实现类似`__iadd__`的占位算子，因为我们不想让`Number`可变，同样的像`__radd__`的反射算子也没有实现

现在Python可以使用`Number`类了

```python
from my_module import Number

def hash_djb2(s: str):
	'''
	A version of Daniel J. Bernstein's djb2 string hashing algorithm
	Like many hashing algorithms, it relies on integer wrapping.
	'''

	n = Number(0)
	five = Number(5)

	for x in s:
		n = Number(ord(x)) + ((n << five) - n)
	return n

assert hash_djb2('l50_50') == Number(-1152549421)
```

## 最终代码

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};
use std::convert::TryInto;

use pyo3::exceptions::{PyValueError, PyZeroDivisionError};
use pyo3::prelude::*;
use pyo3::class::basic::CompareOp;
use pyo3::types::PyComplex;

fn wrap(obj: &PyAny) -> Result<i32, PyErr> {
    let val = obj.call_method1("__and__", (0xFFFFFFFF_u32,))?;
    let val: u32 = val.extract()?;
    Ok(val as i32)
}
/// Did you ever hear the tragedy of Darth Signed The Overfloweth? I thought not.
/// It's not a story C would tell you. It's a Rust legend.
#[pyclass(module = "my_module")]
#[pyo3(text_signature = "(int)")]
struct Number(i32);

#[pymethods]
impl Number {
    #[new]
    fn new(#[pyo3(from_py_with = "wrap")] value: i32) -> Self {
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

    fn __add__(&self, other: &Self) -> Self {
        Self(self.0.wrapping_add(other.0))
    }

    fn __sub__(&self, other: &Self) -> Self {
        Self(self.0.wrapping_sub(other.0))
    }

    fn __mul__(&self, other: &Self) -> Self {
        Self(self.0.wrapping_mul(other.0))
    }

    fn __truediv__(&self, other: &Self) -> PyResult<Self> {
        match self.0.checked_div(other.0) {
            Some(i) => Ok(Self(i)),
            None => Err(PyZeroDivisionError::new_err("division by zero")),
        }
    }

    fn __floordiv__(&self, other: &Self) -> PyResult<Self> {
        match self.0.checked_div(other.0) {
            Some(i) => Ok(Self(i)),
            None => Err(PyZeroDivisionError::new_err("division by zero")),
        }
    }

    fn __rshift__(&self, other: &Self) -> PyResult<Self> {
        match other.0.try_into() {
            Ok(rhs) => Ok(Self(self.0.wrapping_shr(rhs))),
            Err(_) => Err(PyValueError::new_err("negative shift count")),
        }
    }

    fn __lshift__(&self, other: &Self) -> PyResult<Self> {
        match other.0.try_into() {
            Ok(rhs) => Ok(Self(self.0.wrapping_shl(rhs))),
            Err(_) => Err(PyValueError::new_err("negative shift count")),
        }
    }

    fn __xor__(&self, other: &Self) -> Self {
        Self(self.0 ^ other.0)
    }

    fn __or__(&self, other: &Self) -> Self {
        Self(self.0 | other.0)
    }

    fn __and__(&self, other: &Self) -> Self {
        Self(self.0 & other.0)
    }

    fn __int__(&self) -> i32 {
        self.0
    }

    fn __float__(&self) -> f64 {
        self.0 as f64
    }

    fn __complex__<'py>(&self, py: Python<'py>) -> &'py PyComplex {
        PyComplex::from_doubles(py, self.0 as f64, 0.0)
    }
}

#[pymodule]
fn my_module(_py: Python<'_>, m: &PyModule) -> PyResult<()> {
    m.add_class::<Number>()?;
    Ok(())
}
# const SCRIPT: &'static str = r#"
# def hash_djb2(s: str):
#     n = Number(0)
#     five = Number(5)
#
#     for x in s:
#         n = Number(ord(x)) + ((n << five) - n)
#     return n
#
# assert hash_djb2('l50_50') == Number(-1152549421)
# assert hash_djb2('logo') == Number(3327403)
# assert hash_djb2('horizon') == Number(1097468315)
#
#
# assert Number(2) + Number(2) == Number(4)
# assert Number(2) + Number(2) != Number(5)
#
# assert Number(13) - Number(7) == Number(6)
# assert Number(13) - Number(-7) == Number(20)
#
# assert Number(13) / Number(7) == Number(1)
# assert Number(13) // Number(7) == Number(1)
#
# assert Number(13) * Number(7) == Number(13*7)
#
# assert Number(13) > Number(7)
# assert Number(13) < Number(20)
# assert Number(13) == Number(13)
# assert Number(13) >= Number(7)
# assert Number(13) <= Number(20)
# assert Number(13) == Number(13)
#
#
# assert (True if Number(1) else False)
# assert (False if Number(0) else True)
#
#
# assert int(Number(13)) == 13
# assert float(Number(13)) == 13
# assert Number.__doc__ == "Did you ever hear the tragedy of Darth Signed The Overfloweth? I thought not.\nIt's not a story C would tell you. It's a Rust legend."
# assert Number(12345234523452) == Number(1498514748)
# try:
#     import inspect
#     assert inspect.signature(Number).__str__() == '(int)'
# except ValueError:
#     # Not supported with `abi3` before Python 3.10
#     pass
# assert Number(1337).__str__() == '1337'
# assert Number(1337).__repr__() == 'Number(1337)'
"#;

#
# use pyo3::PyTypeInfo;
#
# fn main() -> PyResult<()> {
#     Python::with_gil(|py| -> PyResult<()> {
#         let globals = PyModule::import(py, "__main__")?.dict();
#         globals.set_item("Number", Number::type_object(py))?;
#
#         py.run(SCRIPT, Some(globals), None)?;
#         Ok(())
#     })
# }
```

## 附录：写一些不安全的代码（unsafe code）

在这一章开始时我们说PyO3没有提供在box外包装Python整数的方法，其实并不完全对，虽然PyO3 API 不可以，但是有一个 Python C API 函数能做到：

```c
unsigned long PyLong_AsUnsignedLongMask(PyObject *obj)
```

我们可以通过`pyo3::ffi::PyLong_AsUnsignedLongMask`调用这个函数，这是一个`unsafe`的函数，意味着我们必须使用一个 unsafe 块来调用它并对未保证其约束负责。回忆这些约束：
- 必须保证GIL，否则调用函数会引起数据竞争
- 指针必须是正当的，必须指向一个正当的 Python 对象

现在来创建那个帮助函数，签名必须是`fn(&PyAny) -> PyResult<T>`

- `&PyAny`表示一个检查过的借用引用，所以指向它的指针是正当的
- 每当我们在作用域中借用Python对象的引用时，必须保证GIL

```rust
# #![allow(dead_code)]
use std::os::raw::c_ulong;
use pyo3::prelude::*;
use pyo3::ffi;
use pyo3::conversion::AsPyPointer;

fn wrap(obj: &PyAny) -> Result<i32, PyErr> {
    let py: Python<'_> = obj.py();

    unsafe {
        let ptr = obj.as_ptr();

        let ret: c_ulong = ffi::PyLong_AsUnsignedLongMask(ptr);
        if ret == c_ulong::MAX {
            if let Some(err) = PyErr::take(py) {
                return Err(err);
            }
        }

        Ok(ret as i32)
    }
}
```
