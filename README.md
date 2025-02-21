# TODO LIST
feel free to raise PRs if you have interest and just enough knowledge to solve any of the following issues.

[ ] run test is not success by `python test_extension.py` but not `python test_extension.py`

[ ] cuda compile options are not set. please approch to the `CMakeLists.txt` file

[ ] the compiled _C.so file will show in the root of project

[ ] remove redundent files

[ ] add pybind11 example

# C++/CUDA Extensions for PyTorch by scikit-build-core

An example of writing a C++/CUDA extension for PyTorch. See
[here](https://pytorch.org/tutorials/advanced/cpp_custom_ops.html) for the accompanying tutorial.
This repo demonstrates how to write an example `extension_cpp.ops.mymuladd`
custom op that has both custom CPU and CUDA kernels.

The examples in this repo work with PyTorch 2.4+.

To build:
```
uv sync
```

To test:
1. move the test_extension.py to the root directory
2. run the following command
```
python test_extension.py
```

## Authors
[Liu Zhen](https://github.com/Hernandor)

# Thanks to the original authors
[Peter Goldsborough](https://github.com/goldsborough), [Richard Zou](https://github.com/zou3519)
