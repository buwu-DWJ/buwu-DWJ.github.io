# Rust类型到Python类型的映射

当编写在Python中可调用的函数时（例如`#[pyfunction]`或者`#[pymethods]`），函数的变量需要是特征`FromPyObject`，函数的返回值需要是`IntoPy<PyObject>`

## 变量类型

当接受一个函数变量时，可以使用 Rust 库的类型或者 PyO3 的 Python-原生类型



The table below contains the Python type and the corresponding function argument types that will accept them:

| Python                 |                                               Rust                                                | Rust (Python-native) |
| ---------------------- | :-----------------------------------------------------------------------------------------------: | :------------------: |
| `object`               |                                                 -                                                 |       `&PyAny`       |
| `str`                  |                        `String`, `Cow<str>`, `&str`, `OsString`, `PathBuf`                        |     `&PyUnicode`     |
| `bytes`                |                                  `Vec<u8>`, `&[u8]`, `Cow<[u8]>`                                  |      `&PyBytes`      |
| `bool`                 |                                              `bool`                                               |      `&PyBool`       |
| `int`                  |                           Any integer type (`i32`, `u32`, `usize`, etc)                           |      `&PyLong`       |
| `float`                |                                           `f32`, `f64`                                            |      `&PyFloat`      |
| `complex`              |                                    `num_complex::Complex`[^1]                                     |     `&PyComplex`     |
| `list[T]`              |                                             `Vec<T>`                                              |      `&PyList`       |
| `dict[K, V]`           | `HashMap<K, V>`, `BTreeMap<K, V>`, `hashbrown::HashMap<K, V>`[^2], `indexmap::IndexMap<K, V>`[^3] |      `&PyDict`       |
| `tuple[T, U]`          |                                        `(T, U)`, `Vec<T>`                                         |      `&PyTuple`      |
| `set[T]`               |                     `HashSet<T>`, `BTreeSet<T>`, `hashbrown::HashSet<T>`[^2]                      |       `&PySet`       |
| `frozenset[T]`         |                     `HashSet<T>`, `BTreeSet<T>`, `hashbrown::HashSet<T>`[^2]                      |    `&PyFrozenSet`    |
| `bytearray`            |                                      `Vec<u8>`, `Cow<[u8]>`                                       |    `&PyByteArray`    |
| `slice`                |                                                 -                                                 |      `&PySlice`      |
| `type`                 |                                                 -                                                 |      `&PyType`       |
| `module`               |                                                 -                                                 |     `&PyModule`      |
| `datetime.datetime`    |                                                 -                                                 |    `&PyDateTime`     |
| `datetime.date`        |                                                 -                                                 |      `&PyDate`       |
| `datetime.time`        |                                                 -                                                 |      `&PyTime`       |
| `datetime.tzinfo`      |                                                 -                                                 |     `&PyTzInfo`      |
| `datetime.timedelta`   |                                                 -                                                 |      `&PyDelta`      |
| `typing.Optional[T]`   |                                            `Option<T>`                                            |          -           |
| `typing.Sequence[T]`   |                                             `Vec<T>`                                              |    `&PySequence`     |
| `typing.Mapping[K, V]` | `HashMap<K, V>`, `BTreeMap<K, V>`, `hashbrown::HashMap<K, V>`[^2], `indexmap::IndexMap<K, V>`[^3] |     `&PyMapping`     |
| `typing.Iterator[Any]` |                                                 -                                                 |    `&PyIterator`     |
| `typing.Union[...]`    |                                     `#[derive(FromPyObject)]`                                     |          -           |

也有一些与 GIL 以及 Rust-defined `#[pyclass]`有关的特殊类型

| What          | Description                                                                        |
| ------------- | ---------------------------------------------------------------------------------- |
| `Python`      | A GIL token, used to pass to PyO3 constructors to prove ownership of the GIL       |
| `Py<T>`       | A Python object isolated from the GIL lifetime. This can be sent to other threads. |
| `PyObject`    | `Py<PyAny>`的别名                                                                  |
| `&PyCell<T>`  | A `#[pyclass]` value owned by Python.                                              |
| `PyRef<T>`    | A `#[pyclass]` borrowed immutably.                                                 |
| `PyRefMut<T>` | A `#[pyclass]` borrowed mutably.                                                   |


### Using Rust library types vs Python-native types

相比于 Python 原生类型，使用 Rust 库类型作为函数变量会导致转换成本。使用 Python 原生类型几乎是零成本的（它们只需要一个类似 Python 内建函数`isinstance()`的类型检查）。

然而，一旦支付了转换成本，Rust 标准库类型有下述的优点
- 可以编写原生 Rust 速度的代码
- 与 Rust 生态有更好的互动
- 可以使用`Python::allow_threads`来解放 Python GIL，当你的 Rust 代码运行时使其他 Python 线程有更好的表现
- 可以有更严格的类型检查，比如指定`Vec<i32>`后将只能接收一个包含整数的 Python 列表，而 Python 原生的等价物`&PyList`会接受包含任意类型的 Python 列表

绝大多数时间考虑到获得的收益，转换成本都是值得的！

## 返回 Rust 值到 Python 中

当从 Python 中调用函数返回值时，Python原生类型（`&PyAny`, `&PyDict` etc.）可以零成本使用。

因为这些类型是引用，在一些情况下 Rust 编译器会要求生命周期的标注。如果是这样，则需要`Py<PyAny>`, `Py<PyDict>` etc. 来代替，这同样是零成本的。对所有这些 Python原生类型`T`，`Py<T>`可以通过在`T`上用`.into()`转换来创建。


如果函数是易犯错的，则需要返回`PyResult<T>` 或 `Result<T, E>`，其中 `E` 是`From<E> for PyErr`的实现。如果返回了 `Err` 变量，这会抛出一个 `Python` 异常。

| Rust type                                     | Resulting Python Type |
| --------------------------------------------- | :-------------------: |
| `String`                                      |         `str`         |
| `&str`                                        |         `str`         |
| `bool`                                        |        `bool`         |
| Any integer type (`i32`, `u32`, `usize`, etc) |         `int`         |
| `f32`, `f64`                                  |        `float`        |
| `Option<T>`                                   |     `Optional[T]`     |
| `(T, U)`                                      |     `Tuple[T, U]`     |
| `Vec<T>`                                      |       `List[T]`       |
| `Cow<[u8]>`                                   |        `bytes`        |
| `HashMap<K, V>`                               |     `Dict[K, V]`      |
| `BTreeMap<K, V>`                              |     `Dict[K, V]`      |
| `HashSet<T>`                                  |       `Set[T]`        |
| `BTreeSet<T>`                                 |       `Set[T]`        |
| `&PyCell<T: PyClass>`                         |          `T`          |
| `PyRef<T: PyClass>`                           |          `T`          |
| `PyRefMut<T: PyClass>`                        |          `T`          |

[^1]: 需要 `num-complex` optional feature.

[^2]: 需要 `hashbrown` optional feature.

[^3]: 需要 `indexmap` optional feature.