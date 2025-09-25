# LLK Testing and Performance measurement infrastructure

## Test infrastructure
LLK testing infra provides a framework for compiling and running tests on device. It is supported
by python code that runs on the host side, and C++ code that runs on the device.


When LLKs break, the process of diagnosing the breakage is tedious and complicated, especially when
the bug manifests in higher levels of the software stack. We want to minimize the amount of escapes
of LLK bugs to consumers of the API. For this reason the LLK testing infrastructure was created.
The goals of the infrastructure are the following:
- Zero escapes to higher levels of the stack
- Simple tests that can be used to easily diagnose which APIs are breaking
- Fast turnaround for code updates

## Performance infrastructure
Performance infrastructure is an extension of test infra that seeks to evaluate the performance of 
LLKs. This data can then be used to diagnose bottlenecks, as well as detect performance regressions
before they are deployed to consumers.

## Running tests

### Requirements

To run the test infrastructure, ensure your system has one of the following devices installed:

- **Blackhole** (or any derivative)
- **Wormhole** (or any derivative)

> ⚠️ The device must be flashed with the original firmware.

When the testing environment is correctly initialized, it will auto-detect the underlying hardware.

### 1. Set Up the Environment
- **If using the `tt-llk` Docker image**, run:

```bash
./setup_testing_env.sh
```

- **If you are an external developer or not using the Docker image**, run:

```bash
source ./setup_external_testing_env.sh
```

> 🔄 **Note**: Always use `source` to ensure the environment is activated in your current shell session.

### 2. Navigate to the Python Test Directory
```bash
cd python_tests
```

### 3. Run any test using `pytest`
The directory you are currently in contains the functional tests and performance benchmarks.
The files containing the functional tests start with the prefix: `test_` and the performance
benchmarks start with the prefix `perf_`

You can run any of these tests using the following command:

```bash
pytest <test_file_name>
```

If you want to run all of the functional tests:
```bash
pytest -m "not perf"
```

And if you want to run the performance benchmarks you can run:
```bash
pytest -m perf
```
