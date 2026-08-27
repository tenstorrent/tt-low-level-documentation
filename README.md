# Low Level Software Documentation

This repository will contain documentation for both Compute Low-level kernel APIs, and Data Movement APIs in [tt-metal](https://github.com/tenstorrent/tt-metal) repository. 

Data Movement APIs documentation will cover information about the NOC (Network on chip) & overlay for Wormhole and Blackhole devices, and also information about testing and performance results from the data_movement isolated test suites found [here](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement)

Compute Low-level kernel APIs will cover information about the Tensix Hardware for Wormhole and Blackhole devices, and expected usage for LLK APIs found [here](https://github.com/tenstorrent/tt-llk)

This repository will only contain documentation, and it is recommended for OP & Model writers of tt-metal, since it will cover both the usage of the APIs, and performance. For more in depth knowledge about HW instructions and features, isa documentation can be found [here](https://github.com/tenstorrent/tt-isa-documentation)

## Data Movement Web Viewer

The [Data Movement Web Viewer](https://docs.tenstorrent.com/tt-low-level-documentation/) is an interactive tool which compiles performance results from the data movement test suite, in particular from sweep tests, into a user-friendly interface. It aims to make our test data more easily interpreted and accessible.

More details can be found [here](https://github.com/tenstorrent/tt-metal/tree/main/tests/tt_metal/tt_metal/data_movement#data-movement-web-viewer)
