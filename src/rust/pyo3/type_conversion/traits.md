# 转换特征

PyO3提供了一些便利的特征来进行 Python 类型和 Rust 类型之间的转换。

## `.extract()` 和 `FromPyObject` 特征

将 Python 对象转成一个 Rust 值最简单的方法是使用`.extract()`。如果转换失败了，它会返回一个类型错误`PyResult`，所以通常可以这样使用

```rust
use pyo3::prelude::*;
use pyo3::types::PyList;
fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        let list = PyList::new(py, b"foo");
let v: Vec<i32> = list.extract()?;
        assert_eq!(&v, &[102, 111, 111]);
        Ok(())
    })
}
```

这个方法对许多 Python 类型是可用的，并且可以产生许多 Rust 类型，可以在`FromPyObject`的实现列表中查看。对包装成 Python 对象的自己的 Rust 类型也有`FromPyObject`实现（见class章节）。为了同时操作可变引用并且满足 Rust 的无别名（non-aliasing）可变引用的规则，必须提取（extract）的 PyO3 引用包装器`PyRef`和`PyRefMut`。它们类似引用包装器`std::cell::RefCell`，确保了在运行时允许 Rust 借用。


### 提取（deriving）`FromPyObject`

`FromPyObject` 可以自动地由许多种的结构和枚举派生（derived），只要成员的类型实现了 `FromPyObject`。这甚至包含了泛型类型`T: FromPyObject`。

### 结构提取 `FromPyObject`

The derivation generates code that will attempt to access the attribute  `my_string` on
the Python object, i.e. `obj.getattr("my_string")`, and call `extract()` on the attribute.

```rust
use pyo3::prelude::*;

[derive(FromPyObject)]
struct RustyStruct {
    my_string: String,
}

fn main() -> PyResult<()> {
    Python::with_gil(|py| -> PyResult<()> {
        let module = PyModule::from_code(
            py,
            "class Foo:
            def __init__(self):
                self.my_string = 'test'",
            "",
            "",
        )?;

        let class = module.getattr("Foo")?;
        let instance = class.call0()?;
        let rustystruct: RustyStruct = instance.extract()?;
        assert_eq!(rustystruct.my_string, "test");
    Ok(())
    })
}
```

By setting the `#[pyo3(item)]` attribute on the field, PyO3 will attempt to extract the value by calling the `get_item` method on the Python object.

```rust
use pyo3::prelude::*;

#[derive(FromPyObject)]
struct RustyStruct {
    #[pyo3(item)]
    my_string: String,
}
#
# use pyo3::types::PyDict;
# fn main() -> PyResult<()> {
#     Python::with_gil(|py| -> PyResult<()> {
#         let dict = PyDict::new(py);
#         dict.set_item("my_string", "test")?;
#
#         let rustystruct: RustyStruct = dict.extract()?;
#         assert_eq!(rustystruct.my_string, "test");
#         Ok(())
#     })
# }
```

传递给 `getattr` 和 `get_item`的变量也可以被配置：

```rust
use pyo3::prelude::*;

[derive(FromPyObject)]
struct RustyStruct {
    #[pyo3(item("key"))]
    string_in_mapping: String,
    #[pyo3(attribute("name"))]
    string_attr: String,
}

fn main() -> PyResult<()> {
    Python::with_gil(|py| -> PyResult<()> {
        let module = PyModule::from_code(
            py,
            "class Foo(dict):
            def __init__(self):
                self.name = 'test'
                self['key'] = 'test2'",
            "",
            "",
        )?;
        let class = module.getattr("Foo")?;
        let instance = class.call0()?;
        let rustystruct: RustyStruct = instance.extract()?;
		assert_eq!(rustystruct.string_attr, "test");
        assert_eq!(rustystruct.string_in_mapping, "test2");
        Ok(())
    })
}
```

这尝试从一个带有键值`key`的mapping的属性`name`和`string_in_mapping`中提取`string_attr`。只要 `item` 能得到任意正当的实现了`ToBorrowedObject`的字面值，`attribute`的变量就被限制为非空字符串字面值。


### 元组结构提取 `FromPyObject`

元组结构支持但无法自定义提取（extraction）。永远假定输入是与 Rust 类型具有同样长度的 Python 元组。

```rust
use pyo3::prelude::*;

[derive(FromPyObject)]
struct RustyTuple(String, String);

use pyo3::types::PyTuple;
fn main() -> PyResult<()> {
    Python::with_gil(|py| -> PyResult<()> {
        let tuple = PyTuple::new(py, vec!["test", "test2"]);
        let rustytuple: RustyTuple = tuple.extract()?;
        assert_eq!(rustytuple.0, "test");
        assert_eq!(rustytuple.1, "test2");

        Ok(())
    })
}
```

只有一个域的元组结构被处理为包装类型（下节讲），为了覆写这个特性以保证输入的是一个元组，需要指定结构为：
```rust
use pyo3::prelude::*;

[derive(FromPyObject)]
struct RustyTuple((String,));

use pyo3::types::PyTuple;
fn main() -> PyResult<()> {
    Python::with_gil(|py| -> PyResult<()> {
        let tuple = PyTuple::new(py, vec!["test"]);
        let rustytuple: RustyTuple = tuple.extract()?;
        assert_eq!((rustytuple.0).0, "test");

        Ok(())
    })
}
```

### 包装类型（wrapper types）提取`FromPyObject`

`pyo3(transparent)`可以用在只有一个域的结构上，这导致了直接提取输入的对象，i.e. `obj.extract()`，而不是尝试获取一个 item 或者属性。

```rust
use pyo3::prelude::*;

[derive(FromPyObject)]
struct RustyTransparentTupleStruct(String);

[derive(FromPyObject)]
[pyo3(transparent)]
struct RustyTransparentStruct {
    inner: String,
}

use pyo3::types::PyString;
fn main() -> PyResult<()> {
    Python::with_gil(|py| -> PyResult<()> {
        let s = PyString::new(py, "test");
        let tup: RustyTransparentTupleStruct = s.extract()?;
        assert_eq!(tup.0, "test");

        let stru: RustyTransparentStruct = s.extract()?;
        assert_eq!(stru.inner, "test");

        Ok(())
    })
}
```

### 枚举提取`FromPyObject`

枚举的`FromPyObject`提取会生成尝试按照域的顺序提取变量的代码，只要变量成功被提取，该变量会被返回。这使得能够从 Python 中提取如`str|int`的聚合类型（union type）。

结构提取中的自定义和限制对于枚举变量也是同样的，i.e. 一个元组变量假定输入是一个 Python 元组。`transparent`属性也能被应用到单域变量（single-field-variants）

代码有隐藏

```rust
use pyo3::prelude::*;

#[derive(FromPyObject)]
# #[derive(Debug)]
enum RustyEnum<'a> {
    Int(usize), // input is a positive int
    String(String), // input is a string
    IntTuple(usize, usize), // input is a 2-tuple with positive ints
    StringIntTuple(String, usize), // input is a 2-tuple with String and int
    Coordinates3d { // needs to be in front of 2d
        x: usize,
        y: usize,
        z: usize,
    },
    Coordinates2d { // only gets checked if the input did not have `z`
        #[pyo3(attribute("x"))]
        a: usize,
        #[pyo3(attribute("y"))]
        b: usize,
    },
    #[pyo3(transparent)]
    CatchAll(&'a PyAny), // This extraction never fails
}
#
# use pyo3::types::{PyBytes, PyString};
# fn main() -> PyResult<()> {
#     Python::with_gil(|py| -> PyResult<()> {
#         {
#             let thing = 42_u8.to_object(py);
#             let rust_thing: RustyEnum<'_> = thing.extract(py)?;
#
#             assert_eq!(
#                 42,
#                 match rust_thing {
#                     RustyEnum::Int(i) => i,
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#         {
#             let thing = PyString::new(py, "text");
#             let rust_thing: RustyEnum<'_> = thing.extract()?;
#
#             assert_eq!(
#                 "text",
#                 match rust_thing {
#                     RustyEnum::String(i) => i,
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#         {
#             let thing = (32_u8, 73_u8).to_object(py);
#             let rust_thing: RustyEnum<'_> = thing.extract(py)?;
#
#             assert_eq!(
#                 (32, 73),
#                 match rust_thing {
#                     RustyEnum::IntTuple(i, j) => (i, j),
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#         {
#             let thing = ("foo", 73_u8).to_object(py);
#             let rust_thing: RustyEnum<'_> = thing.extract(py)?;
#
#             assert_eq!(
#                 (String::from("foo"), 73),
#                 match rust_thing {
#                     RustyEnum::StringIntTuple(i, j) => (i, j),
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#         {
#             let module = PyModule::from_code(
#                 py,
#                 "class Foo(dict):
#             def __init__(self):
#                 self.x = 0
#                 self.y = 1
#                 self.z = 2",
#                 "",
#                 "",
#             )?;
#
#             let class = module.getattr("Foo")?;
#             let instance = class.call0()?;
#             let rust_thing: RustyEnum<'_> = instance.extract()?;
#
#             assert_eq!(
#                 (0, 1, 2),
#                 match rust_thing {
#                     RustyEnum::Coordinates3d { x, y, z } => (x, y, z),
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#
#         {
#             let module = PyModule::from_code(
#                 py,
#                 "class Foo(dict):
#             def __init__(self):
#                 self.x = 3
#                 self.y = 4",
#                 "",
#                 "",
#             )?;
#
#             let class = module.getattr("Foo")?;
#             let instance = class.call0()?;
#             let rust_thing: RustyEnum<'_> = instance.extract()?;
#
#             assert_eq!(
#                 (3, 4),
#                 match rust_thing {
#                     RustyEnum::Coordinates2d { a, b } => (a, b),
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#
#         {
#             let thing = PyBytes::new(py, b"text");
#             let rust_thing: RustyEnum<'_> = thing.extract()?;
#
#             assert_eq!(
#                 b"text",
#                 match rust_thing {
#                     RustyEnum::CatchAll(i) => i.downcast::<PyBytes>()?.as_bytes(),
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#         Ok(())
#     })
# }
```

如果没有枚举变量匹配到，会返回一个包含着被测试变量名的`PyTypeError`。错误信息中报告的名字可以通过`#[pyo3(annotation = "name")]`属性自定义，e.g. 使用传统的 Python 类型名：

代码有折叠

```rust
use pyo3::prelude::*;

#[derive(FromPyObject)]
# #[derive(Debug)]
enum RustyEnum {
    #[pyo3(transparent, annotation = "str")]
    String(String),
    #[pyo3(transparent, annotation = "int")]
    Int(isize),
}
#
# fn main() -> PyResult<()> {
#     Python::with_gil(|py| -> PyResult<()> {
#         {
#             let thing = 42_u8.to_object(py);
#             let rust_thing: RustyEnum = thing.extract(py)?;
#
#             assert_eq!(
#                 42,
#                 match rust_thing {
#                     RustyEnum::Int(i) => i,
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#
#         {
#             let thing = "foo".to_object(py);
#             let rust_thing: RustyEnum = thing.extract(py)?;
#
#             assert_eq!(
#                 "foo",
#                 match rust_thing {
#                     RustyEnum::String(i) => i,
#                     other => unreachable!("Error extracting: {:?}", other),
#                 }
#             );
#         }
#
#         {
#             let thing = b"foo".to_object(py);
#             let error = thing.extract::<RustyEnum>(py).unwrap_err();
#             assert!(error.is_instance_of::<pyo3::exceptions::PyTypeError>(py));
#         }
#
#         Ok(())
#     })
# }
```

如果输入既不是 string 也不是 integer，则错误信息为 `"'<INPUT_TYPE>' cannot be converted to 'str | int'"`。

### `#[derive(FromPyObject)]`容器属性
- `pyo3(transparent)`
    - 直接从`obj.extract()`对象中提取域，而不是`get_item()` 或者 `getattr()`
    - 每次默认中，新类型结构和元祖变量会被视为`transparent`
    - 只对单域结构或者枚举变量有效
- `pyo3(annotation = "name")`
    - 改变在失败时生成的错误信息中失败的变量的名字
    - e.g. `pyo3("int")` 将变量类型报告为 `int`
    - 只支持枚举变量

### `#[derive(FromPyObject)]`域属性
- `pyo3(attribute)`, `pyo3(attribute("name"))`
    - 从一个属性中取回（retrieve）域，可能由变量指定一个自定义的名字
    - 变量必须为字符串字面值
- `pyo3(item)`, `pyo3(item("key"))`
    - 从一个mapping取回（retrieve）域，可能由变量指定一个自定义的名字
    - 可以是实现了`ToBorrowedObject`的字面值
- `pyo3(from_py_with = "...")`
    - 应用一个自定义的函数来 convert the field from Python the desired Rust type.
    - 变量必须是函数名的 string
    - 函数签名必须为`fn(&PyAny) -> PyResult<T>`，其中 `T` 是 Rust 类型的变量

## `IntoPy<T>`

这个特征定义了 Rust 类型的 to-python 转换，通常被实现为 `IntoPy<PyObject>` ，这是在从 `#[pyfunction]` 和 `#[pymethods]` 返回一个值时所需要的。

所有 PyO3 的类型实现了这个特征，一个没有使用 `extends` 的 `#[pyclass]` 也实现了。

偶尔在将自定义类型映射为没有独立（unique）类型的 Python 类型时会选择实现它。

```rust
use pyo3::prelude::*;

struct MyPyObjectWrapper(PyObject);

impl IntoPy<PyObject> for MyPyObjectWrapper {
    fn into_py(self, py: Python<'_>) -> PyObject {
        self.0
    }
}
```

## The `ToPyObject` trait

`ToPyObject` 是一个转变特征，它允许将各种对象转换为`PyObject`。`IntoPy<PyObject>`有同样的效果，除了它 consumes `self`。
