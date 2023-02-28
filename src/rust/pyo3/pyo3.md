# 官方readme

[官方入门教程](https://pyo3.rs/main/)
[api文档](https://pyo3.rs/main/doc/pyo3/)

## 在Python中使用Rust
PyO3用来生成原生 Python 模块，最简单的方法是使用`maturin`，这是一个用最少配置发布基于 Rust 的 Python 包的工具。新建一个测试用的文件夹

```console
# (replace string_sum with the desired package name)
$ mkdir string_sum
$ cd string_sum
$ pip install maturin
```

在上述`string_sum`文件夹中初始化`maturin`，

```console
$ maturin init
✔ 🤷 What kind of bindings to use? · pyo3
  ✨ Done! New project created string_sum
```

可以看到生成了新包的source，最重要的文件时`Cargo.toml`和`lib.rs`，大约文件中看起来是这样的

`Cargo.toml`
```toml
[package]
name = "string_sum"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
[lib]
# The name of the native library. This is the name which will be used in Python to import the
# library (i.e. `import string_sum`). If you change this, you must also change the name of the
# `#[pymodule]` in `src/lib.rs`.
name = "string_sum"
# "cdylib" is necessary to produce a shared library for Python to import from.
#
# Downstream Rust code (including code in `bin/`, `examples/`, and `tests/`) will not be able
# to `use string_sum;` unless the "rlib" or "lib" crate type is also included, e.g.:
# crate-type = ["cdylib", "rlib"]
crate-type = ["cdylib"]

[dependencies]
pyo3 = "0.18.1"
```

`src/lib.rs`

```rust
use pyo3::prelude::*;

/// Formats the sum of two numbers as string.
#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}

/// A Python module implemented in Rust. The name of this function must match
/// the `lib.name` setting in the `Cargo.toml`, else Python will not be able to
/// import the module.
#[pymodule]
fn string_sum(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}
```

最后，终端输入`maturin develop`。这会创建一个 package 然后安装进当前的 Python 环境，然后这个 package 就能在 Python 里用了

```console
$ maturin develop
# lots of progress output as maturin runs the compilation...
$ python
>>> import string_sum
>>> string_sum.sum_as_string(5, 20)
'25'
```

## 在Rust中使用Python

将 Python 嵌入为 Rust 二进制，你要确保你的 Python 安装包含了一个共享库(ensure that your Python installation contains a shared library?啥意思)。

用 `cargo new` 新建一个项目并像这样将 `pyo3` 加入到 `Cargo.toml` 中，

```toml
[dependencies.pyo3]
version = "0.18.1"
features = ["auto-initialize"]
```

下面的例子会输出 `sys.version` 的值以及当前用户名

```rust
use pyo3::prelude::*;
use pyo3::types::IntoPyDict;

fn main() -> PyResult<()> {
    Python::with_gil(|py| {
        let sys = py.import("sys")?;
        let version: String = sys.getattr("version")?.extract()?;

        let locals = [("os", py.import("os")?)].into_py_dict(py);
        let code = "os.getenv('USER') or os.getenv('USERNAME') or 'Unknown'";
        let user: String = py.eval(code, None, Some(&locals))?.extract()?;

        println!("Hello {}, I'm Python {}", user, version);
        Ok(())
    })
}
```

> 如果 `cargo run` 后出现`Fatal Python error: init_fs_encoding: failed to get the Python codec of the filesystem encoding`，上面信息中应该还有`PYTHONHOME: not set, PYTHONPATH: not set`的信息，应该将使用的 Python 解释器路径加入到系统环境中，比如 Annconda 则将 Annaconda 文件夹路径创建为 `PYTHONHOME`，其下的 python.exe 路径创建为 `PYTHONPATH`。

## 工具和库

- [maturin](https://github.com/PyO3/maturin) _Build and publish crates with pyo3, rust-cpython or cffi bindings as well as rust binaries as python packages_
- [setuptools-rust](https://github.com/PyO3/setuptools-rust) _Setuptools plugin for Rust support_.
- [pyo3-built](https://github.com/PyO3/pyo3-built) _Simple macro to expose metadata obtained with the [`built`](https://crates.io/crates/built) crate as a [`PyDict`](https://docs.rs/pyo3/*/pyo3/types/struct.PyDict.html)_
- [rust-numpy](https://github.com/PyO3/rust-numpy) _Rust binding of NumPy C-API_
- [dict-derive](https://github.com/gperinazzo/dict-derive) _Derive FromPyObject to automatically transform Python dicts into Rust structs_
- [pyo3-log](https://github.com/vorner/pyo3-log) _Bridge from Rust to Python logging_
- [pythonize](https://github.com/davidhewitt/pythonize) _Serde serializer for converting Rust objects to JSON-compatible Python objects_
- [pyo3-asyncio](https://github.com/awestlake87/pyo3-asyncio) _Utilities for working with Python's Asyncio library and async functions_
- [rustimport](https://github.com/mityax/rustimport) _Directly import Rust files or crates from Python, without manual compilation step. Provides pyo3 integration by default and generates pyo3 binding code automatically._

## 例子

- [autopy](https://github.com/autopilot-rs/autopy) _A simple, cross-platform GUI automation library for Python and Rust._
  - Contains an example of building wheels on TravisCI and appveyor using [cibuildwheel](https://github.com/pypa/cibuildwheel)
- [ballista-python](https://github.com/apache/arrow-ballista-python) _A Python library that binds to Apache Arrow distributed query engine Ballista._
- [bed-reader](https://github.com/fastlmm/bed-reader) _Read and write the PLINK BED format, simply and efficiently._
    - Shows Rayon/ndarray::parallel (including capturing errors, controlling thread num), Python types to Rust generics, Github Actions
- [cryptography](https://github.com/pyca/cryptography/tree/main/src/rust) _Python cryptography library with some functionality in Rust._
- [css-inline](https://github.com/Stranger6667/css-inline/tree/master/bindings/python) _CSS inlining for Python implemented in Rust._
- [datafusion-python](https://github.com/apache/arrow-datafusion-python) _A Python library that binds to Apache Arrow in-memory query engine DataFusion._
- [deltalake-python](https://github.com/delta-io/delta-rs/tree/main/python) _Native Delta Lake Python binding based on delta-rs with Pandas integration._
- [fastbloom](https://github.com/yankun1992/fastbloom) _A fast [bloom filter](https://github.com/yankun1992/fastbloom#BloomFilter) | [counting bloom filter](https://github.com/yankun1992/fastbloom#countingbloomfilter) implemented by Rust for Rust and Python!_
- [fastuuid](https://github.com/thedrow/fastuuid/) _Python bindings to Rust's UUID library._
- [feos](https://github.com/feos-org/feos) _Lightning fast thermodynamic modeling in Rust with fully developed Python interface._
- [forust](https://github.com/jinlow/forust) _A lightweight gradient boosted decision tree library written in Rust._
- [html-py-ever](https://github.com/PyO3/setuptools-rust/tree/main/examples/html-py-ever) _Using [html5ever](https://github.com/servo/html5ever) through [kuchiki](https://github.com/kuchiki-rs/kuchiki) to speed up html parsing and css-selecting._
- [hyperjson](https://github.com/mre/hyperjson) _A hyper-fast Python module for reading/writing JSON data using Rust's serde-json._
- [inline-python](https://github.com/fusion-engineering/inline-python) _Inline Python code directly in your Rust code._
- [jsonschema-rs](https://github.com/Stranger6667/jsonschema-rs/tree/master/bindings/python) _Fast JSON Schema validation library._
- [mocpy](https://github.com/cds-astro/mocpy) _Astronomical Python library offering data structures for describing any arbitrary coverage regions on the unit sphere._
- [orjson](https://github.com/ijl/orjson) _Fast Python JSON library._
- [ormsgpack](https://github.com/aviramha/ormsgpack) _Fast Python msgpack library._
- [point-process](https://github.com/ManifoldFR/point-process-rust/tree/master/pylib) _High level API for pointprocesses as a Python library._
- [polaroid](https://github.com/daggy1234/polaroid) _Hyper Fast and safe image manipulation library for Python written in Rust._
- [polars](https://github.com/pola-rs/polars) _Fast multi-threaded DataFrame library in Rust | Python | Node.js._
- [pydantic-core](https://github.com/pydantic/pydantic-core) _Core validation logic for pydantic written in Rust._
- [pyheck](https://github.com/kevinheavey/pyheck) _Fast case conversion library, built by wrapping [heck](https://github.com/withoutboats/heck)._
    - Quite easy to follow as there's not much code.
- [pyre](https://github.com/Project-Dream-Weaver/pyre-http) _Fast Python HTTP server written in Rust._
- [ril-py](https://github.com/Cryptex-github/ril-py) _A performant and high-level image processing library for Python written in Rust._
- [river](https://github.com/online-ml/river) _Online machine learning in python, the computationally heavy statistics algorithms are implemented in Rust._
- [rust-python-coverage](https://github.com/cjermain/rust-python-coverage) _Example PyO3 project with automated test coverage for Rust and Python._
- [tiktoken](https://github.com/openai/tiktoken) _A fast BPE tokeniser for use with OpenAI's models._
- [tokenizers](https://github.com/huggingface/tokenizers/tree/main/bindings/python) _Python bindings to the Hugging Face tokenizers (NLP) written in Rust._
- [wasmer-python](https://github.com/wasmerio/wasmer-python) _Python library to run WebAssembly binaries._

## Articles and other media

- [How Pydantic V2 leverages Rust's Superpowers](https://fosdem.org/2023/schedule/event/rust_how_pydantic_v2_leverages_rusts_superpowers/) - Feb 4, 2023
- [Nine Rules for Writing Python Extensions in Rust](https://towardsdatascience.com/nine-rules-for-writing-python-extensions-in-rust-d35ea3a4ec29?sk=f8d808d5f414154fdb811e4137011437) - Dec 31, 2021
- [Calling Rust from Python using PyO3](https://saidvandeklundert.net/learn/2021-11-18-calling-rust-from-python-using-pyo3/) - Nov 18, 2021
- [davidhewitt's 2021 talk at Rust Manchester meetup](https://www.youtube.com/watch?v=-XyWG_klSAw&t=320s) - Aug 19, 2021
- [Incrementally porting a small Python project to Rust](https://blog.waleedkhan.name/port-python-to-rust/) - Apr 29, 2021
- [Vortexa - Integrating Rust into Python](https://www.vortexa.com/insight/integrating-rust-into-python) - Apr 12, 2021
- [Writing and publishing a Python module in Rust](https://blog.yossarian.net/2020/08/02/Writing-and-publishing-a-python-module-in-rust) - Aug 2, 2020