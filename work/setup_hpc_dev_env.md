# Setup a development environment for Scientific Computing

## Aim

* Languages  - C++, Python
* Package manager - mamba/conda


## MPI

```bash
mamba install cxx-compiler
```

You need to specify CMAKE_CXX_COMPILE(CMAKE_C_COMPILER) to find mpi, it should be configured as:

```bash
CMAKE_CXX_COMPILE=${CONDA_PREFIX}/bin/clang++
```

Better to put this setting in CMakePresets.json