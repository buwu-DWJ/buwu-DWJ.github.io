# 魔术方法和值槽slots

Python的对象模型为不同的对象行为定义了数种协议（protocol），例如 sequence，mapping和 number。在Python中通过魔术方法实现这些协议，例如`__str__`，`__repr__`，也被称为双下划线方法（dunder methods）。

在基于 Python C-接口实现PyO3时，许多魔术方法必须放进类对象的值槽（slot）中。如果 `#[pymethods]` 中的一个函数被识别为魔术方法，它会被自动地放置进Python对象的正确值槽中。

PyO3处理的魔术方法和[标准Python](https://docs.python.org/3/reference/datamodel.html#special-method-names)非常相像，它们是[这里的值槽](https://docs.python.org/3/c-api/typeobj.html)的子集。一些值槽没有Python中对应的魔术方法，它们需要一些额外的处理：
- 垃圾处理的魔术方法
- 缓存协议的魔术方法

当PyO3处理一个魔术方法时，和其他`#[pymethods]`有一些区别
- Rust的函数签名必须和魔术方法相匹配
- **不允许**`#[pyo3(signature = (...)]` 和 `#[pyo3(text_signature = "...")]`属性

下面列出所有PyO3处理的魔术方法

- 所有方法接受的第一个参数，显示为`<self>`。可以是`&self`，`&mut self`或者是形如`self_: PyRef<'_, Self>` 或 `self_: PyRefMut<'_, Self>` 的`PyCell`引用
- 永远允许可选参数`Python<'py>`作为第一个参数
- 返回值可以选择被包装进`PyResult`
- `object`意为任何类型都可以被一个Python对象提取（变量）或者被转换为一个Python对象（返回值）
- 其他类型（若有）时必须匹配，例如`pyo3::basic::CompareOp` 作为 `__richcmp__`的第二个参数
- 对于比较方法（comparison method）或者算数方法（arithmetic method），提取时的错误不表现为一个异常，而是返回一个`NotImplemented`
- 对一些魔术方法，返回值不受PyO3约束，但是会被Python解释器检查。例如，`__str__`需要返回一个字符串对象。这由`object(Python type)`指明。

## 基本对象的自定义

- `__str__(<self>)` -> `object (str)`
- `__repr__(<self>)` -> `object (str)`
- `__hash__(<self>)` -> `isize`

objects that compare equal 必须有相同的哈希值。`isize` 处可能是返回任意到 64 bits 的类型，PyO3会自动地将其转换为一个 `isize`，
<details>
<summary>使Python默认的哈希失效</summary>
默认情况下，任何 `#[pyclass]` 类型在Python下有一个默认的哈希实现，不想要被哈希的类型需要替换`__hash__`为`None`。例如

```rust
# use pyo3::prelude::*;
#
#[pyclass]
struct NotHashable {}

#[pymethods]
impl NotHashable {
    #[classattr]
    const __hash__: Option<PyObject> = None;
}
```
</details>

- `__richcmp__(<self>, object, pyo3::basic::CompareOp)` -> `object`

Overloads Python 的比较算子 (`==`, `!=`, `<`, `<=`, `>` 和 `>=`)。变量 `CompareOp` 表示这是一个比较算子。

_注意到实现 `__richcmp__` 会让Python不生成一个默认的 `__hash__` 实现, 所以当实现 `__richcmp__`时考虑实现 `__hash__` ._

<details>
<summary>返回类型</summary>

返回类型通常是`PyResult<bool>`，但是其实可以返回任意的Python对象。如果第二个变量`object`不是在签名中指定的类型，生成的代码会自动返回`NotImplemented`。

可以使用`CompareOp::matches`来讲一个Rust的`std::cmp::Ordering`调整为需要的比较类型。
</details>

- `__getattr__(<self>, object)` -> `object`
- `__getattribute__(<self>, object)` -> `object`

<details>
<summary>`__getattr__` 和 `__getattribute__`的区别</summary>

在Python中，只有当无法通过常规查找调用属性时会使用`__getattr__`。而`__getattribute__`可以通过任意属性方式调用。如果想要获取`self`中存在的属性，要特别小心不要引入无穷递归，并使用`baseclass.__getattribute__()`
</details>

- `__setattr__(<self>, value: object)` -> `()`
- `__delattr__(<self>, object)` -> `()`

覆盖attribute access

- `__bool__(<self>)` -> `bool`

决定对象的真伪值

- `__call__(<self>, ...)` -> `object` - 可以对一般的`pymethods`定义任意变量列表

## 可迭代对象

可以用下述方法定义迭代器

- `__iter__(<self>) -> object`
- `__next__(<self>) -> Option<object> or IterNextOutput`

在`__next__`中返回`None`则表示后面没有元素了。

例子：

```rust
use pyo3::prelude::*;

#[pyclass]
struct MyIterator {
    iter: Box<dyn Iterator<Item = PyObject> + Send>,
}

#[pymethods]
impl MyIterator {
    fn __iter__(slf: PyRef<'_, Self>) -> PyRef<'_, Self> {
        slf
    }
    fn __next__(mut slf: PyRefMut<'_, Self>) -> Option<PyObject> {
        slf.iter.next()
    }
}
```

许多时候要区分迭代器和可迭代类型。事实上，可迭代对象只要实现`__iter__()`，而迭代器必须同时实现`__iter__()` 和 `__next__()`。例如：

```rust
# use pyo3::prelude::*;

#[pyclass]
struct Iter {
    inner: std::vec::IntoIter<usize>,
}

#[pymethods]
impl Iter {
    fn __iter__(slf: PyRef<'_, Self>) -> PyRef<'_, Self> {
        slf
    }

    fn __next__(mut slf: PyRefMut<'_, Self>) -> Option<usize> {
        slf.inner.next()
    }
}

#[pyclass]
struct Container {
    iter: Vec<usize>,
}

#[pymethods]
impl Container {
    fn __iter__(slf: PyRef<'_, Self>) -> PyResult<Py<Iter>> {
        let iter = Iter {
            inner: slf.iter.clone().into_iter(),
        };
        Py::new(slf.py(), iter)
    }
}

# Python::with_gil(|py| {
#     let container = Container { iter: vec![1, 2, 3, 4] };
#     let inst = pyo3::PyCell::new(py, container).unwrap();
#     pyo3::py_run!(py, inst, "assert list(inst) == [1, 2, 3, 4]");
#     pyo3::py_run!(py, inst, "assert list(iter(iter(inst))) == [1, 2, 3, 4]");
# });
```

更多信息见Python官方文档[迭代器-类型](https://docs.python.org/library/stdtypes.html#iterator-types)。

### 在迭代中返回值

刚才讲了如何使用`Option<T>`在迭代中yield值。在Python中，生成器也可以返回值。PyO3提供了`IterNextOutput`枚举来同时`Yield`值和`Return`一个最终值。

## Awaitable 对象

- `__await__(<self>)` -> `object`
- `__aiter__(<self>)` -> `object`
- `__anext__(<self>)` -> `Option<object> or IterANextOutput`

### Mapping & Sequence类型

这一章中的魔术方法可以用来实现Python的容器（container）类型。Python中有两种主要的容器："mappings"（例如字典）和 "sequences"（例如列表和元祖）。

PyO3所用的Python C-API 对于 sequence 和 mapping 有不同的值槽。在纯Python中书写时，实现的时候并没有这种区别，例如`__getitem__`的实现对这两种容器都会填充值槽。

mapping 类型不希望 sequence 值槽被填充，因为这会带来一些不想要结果，例如：

- mapping 类型会成功 cast to `PySequence`].
- Python 会对 sequence 提供默认的`__iter__`实现，它可以连续地用正整数调用`__getitem__`直到返回`IndexError`。

使用`#[pyclass(mapping)]`标注来指示PyO3只填充 mapping 值槽，保持 sequence值槽为空。这会应用到`__getitem__`, `__setitem__` 和 `__delitem__`。

使用`#[pyclass(sequence)]`标注来指示PyO3用`sq_length`而不是`mp_length`来填充`__len__`。这可以帮助例如numpy的库将类识别为一个 sequence。

- `__len__(<self>)` -> `usize`

内置函数`len()`的实现

- `__contains__(<self>, object)` -> `bool`

Implements membership测试算子。会返回 true 若`item` 在 `self`中。对于没有定义`__contains__()`的对象，该测试对简单地遍历sequence直到找到匹配。

<details>
<summary>使Python默认的contain失效</summary>

默认情况下，所有带有`__iter__`方法的`#[pyclass]`类型支持默认的`in`算子的实现。不想要这个的类型可以将`__contains__` 设置为 `None`。

```rust
# use pyo3::prelude::*;
#
#[pyclass]
struct NoContains {}

#[pymethods]
impl NoContains {
    #[classattr]
    const __contains__: Option<PyObject> = None;
}
```
</details>

- `__getitem__(<self>, object)` -> `object`

实现取得`self[a]`元素

- `__setitem__(<self>, object, object)` -> `()`

实现分配（assign）`self[a]`元素，仅当元素可以被替换时进行实现

- `__delitem__(<self>, object)` -> `()`

实现删除元素

- `fn __concat__(&self, other: impl FromPyObject)` -> `PyResult<impl ToPyObject>`

通常使用`+`号，在尝试数值加法`__add__` 和 `__radd__`后拼接两个sequence

- `fn __repeat__(&self, count: isize)` -> `PyResult<impl ToPyObject>`

通常使用`*`号，在尝试数值乘法`__mul__` and `__rmul__`后重复sequence `count`次

- `fn __inplace_concat__(&self, other: impl FromPyObject)` -> `PyResult<impl ToPyObject>`

通常使用`+=`算子，在尝试数值加法`__iadd__`后相加两个sequence

- `fn __inplace_repeat__(&self, count: isize)` -> `PyResult<impl ToPyObject>`

通常使用`*=`算子，在尝试数值乘法`__imul__`后相乘两个sequence

## 描述符 Descriptors

  - `__get__(<self>, object, object)` -> `object`
  - `__set__(<self>, object, object)` -> `()`
  - `__delete__(<self>, object)` -> `()`

### 数值类型

二元数值算子 (`+`, `-`, `*`, `@`, `/`, `//`, `%`, `divmod()`,
`pow()`, `**`, `<<`, `>>`, `&`, `^` 和 `|`)

(如果对象 `object` 不是在签名中指明的类型, 生成的代码会自动`return NotImplemented`.)

- `__add__(<self>, object)` -> `object`
- `__radd__(<self>, object)` -> `object`
- `__sub__(<self>, object)` -> `object`
- `__rsub__(<self>, object)` -> `object`
- `__mul__(<self>, object)` -> `object`
- `__rmul__(<self>, object)` -> `object`
- `__matmul__(<self>, object)` -> `object`
- `__rmatmul__(<self>, object)` -> `object`
- `__floordiv__(<self>, object)` -> `object`
- `__rfloordiv__(<self>, object)` -> `object`
- `__truediv__(<self>, object)` -> `object`
- `__rtruediv__(<self>, object)` -> `object`
- `__divmod__(<self>, object)` -> `object`
- `__rdivmod__(<self>, object)` -> `object`
- `__mod__(<self>, object)` -> `object`
- `__rmod__(<self>, object)` -> `object`
- `__lshift__(<self>, object)` -> `object`
- `__rlshift__(<self>, object)` -> `object`
- `__rshift__(<self>, object)` -> `object`
- `__rrshift__(<self>, object)` -> `object`
- `__and__(<self>, object)` -> `object`
- `__rand__(<self>, object)` -> `object`
- `__xor__(<self>, object)` -> `object`
- `__rxor__(<self>, object)` -> `object`
- `__or__(<self>, object)` -> `object`
- `__ror__(<self>, object)` -> `object`
- `__pow__(<self>, object, object)` -> `object`
- `__rpow__(<self>, object, object)` -> `object`

占位分配算子(in-place assignment operator)：(`+=`, `-=`, `*=`, `@=`, `/=`, `//=`, `%=`, `**=`, `<<=`, `>>=`, `&=`, `^=`, `|=`):

- `__iadd__(<self>, object)` -> `()`
- `__isub__(<self>, object)` -> `()`
- `__imul__(<self>, object)` -> `()`
- `__imatmul__(<self>, object)` -> `()`
- `__itruediv__(<self>, object)` -> `()`
- `__ifloordiv__(<self>, object)` -> `()`
- `__imod__(<self>, object)` -> `()`
- `__ipow__(<self>, object, object)` -> `()`
- `__ilshift__(<self>, object)` -> `()`
- `__irshift__(<self>, object)` -> `()`
- `__iand__(<self>, object)` -> `()`
- `__ixor__(<self>, object)` -> `()`
- `__ior__(<self>, object)` -> `()`

一元算子(Unary operations)：(`-`, `+`, `abs()` 和 `~`):

- `__pos__(<self>)` -> `object`
- `__neg__(<self>)` -> `object`
- `__abs__(<self>)` -> `object`
- `__invert__(<self>)` -> `object`

强制转换？Coercions:

- `__index__(<self>)` -> `object (int)`
- `__int__(<self>)` -> `object (int)`
- `__float__(<self>)` -> `object (float)`

## 缓存对象

- `__getbuffer__(<self>, *mut ffi::Py_buffer, flags)` -> `()`
- `__releasebuffer__(<self>, *mut ffi::Py_buffer)` -> `()`

`__releasebuffer__`返回的错误会被传送到`sys.unraiseablehook`中。强烈建议永远不要从`__releasebuffer__`返回错误，`__releasebuffer__`不会被调用第二次，任何没有释放的东西会被泄露。

### 垃圾收集Garbage Collector Integration

如果类型有对其他Python对象的引用，需要使用Python的垃圾收集器来让GC识别这些引用。这需要实现`__traverse__` 和 `__clear__`，对应于Python C API 中的值槽`tp_traverse` 和 `tp_clear`。每次引用另一个Python对象时，`__traverse__` 必须调用 `visit.call()` 。`__clear__` 必须清理其他Python对象的任意可变引用（来打破引用cycle）。不可变引用不是必须被清理，因为每个cycle必须包含至少一个可变引用。

- `__traverse__(<self>, pyo3::class::gc::PyVisit<'_>)` -> `Result<()`, `pyo3::class::gc::PyTraverseError>`
- `__clear__(<self>)` -> `()`

例子

```rust
use pyo3::prelude::*;
use pyo3::PyTraverseError;
use pyo3::gc::PyVisit;

#[pyclass]
struct ClassWithGCSupport {
    obj: Option<PyObject>,
}

#[pymethods]
impl ClassWithGCSupport {
    fn __traverse__(&self, visit: PyVisit<'_>) -> Result<(), PyTraverseError> {
        if let Some(obj) = &self.obj {
            visit.call(obj)?
        }
        Ok(())
    }

    fn __clear__(&mut self) {
        // Clear reference, this decrements ref counter.
        self.obj = None;
    }
}
```
