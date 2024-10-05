[<img alt="foamlib" src="https://github.com/gerlero/foamlib/raw/main/logo.png" height="50">](https://github.com/gerlero/foamlib)

[![Documentation](https://img.shields.io/readthedocs/foamlib)](https://foamlib.readthedocs.io/)
[![CI](https://github.com/gerlero/foamlib/actions/workflows/ci.yml/badge.svg)](https://github.com/gerlero/foamlib/actions/workflows/ci.yml)
[![Codecov](https://codecov.io/gh/gerlero/foamlib/branch/main/graph/badge.svg)](https://codecov.io/gh/gerlero/foamlib)
[![Checked with mypy](http://www.mypy-lang.org/static/mypy_badge.svg)](http://mypy-lang.org/)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
[![PyPI](https://img.shields.io/pypi/v/foamlib)](https://pypi.org/project/foamlib/)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/foamlib)](https://pypi.org/project/foamlib/)
![OpenFOAM](https://img.shields.io/badge/openfoam-.com%20|%20.org-informational)
[![Docker image](https://img.shields.io/badge/docker%20image-microfluidica%2Ffoamlib-0085a0)](https://hub.docker.com/r/microfluidica/foamlib/)

**foamlib** provides a simple, modern and ergonomic Python interface for interacting with [OpenFOAM](https://www.openfoam.com).

It offers the following classes:

* [`FoamFile`](https://foamlib.readthedocs.io/en/stable/files.html#foamlib.FoamFile) (and [`FoamFieldFile`](https://foamlib.readthedocs.io/en/stable/files.html#foamlib.FoamFieldFile)): read-write access to OpenFOAM configuration and field files as if they were Python `dict`s, using `foamlib`'s own parser. Supports both ASCII and binary field formats.
* [`FoamCase`](https://foamlib.readthedocs.io/en/stable/cases.html#foamlib.FoamCase): a class for configuring, running, and accessing the results of OpenFOAM cases.
* [`AsyncFoamCase`](https://foamlib.readthedocs.io/en/stable/cases.html#foamlib.AsyncFoamCase): variant of `FoamCase` with asynchronous methods for running multiple cases at once.

## Get started

### Install

Install with [pip](https://pypi.org/project/pip/):

```bash
pip install foamlib
```

### Clone a case

```python
import os
from pathlib import Path
from foamlib import FoamCase

pitz_tutorial = FoamCase(Path(os.environ["FOAM_TUTORIALS"]) / "incompressible/simpleFoam/pitzDaily")

my_pitz = pitz_tutorial.clone("myPitz")
```

### Run the case

```python
my_pitz.run()
```

### Access the results

```python
latest_time = my_pitz[-1]

p = latest_time["p"]
U = latest_time["U"]

print(p.internal_field)
print(U.internal_field)
```

### Clean the case

```python
my_pitz.clean()
```

### Edit the `controlDict` file

```python
my_pitz.control_dict["writeInterval"] = 10
```

### Make multiple file reads and writes in a single go

```python
with my_pitz.fv_schemes as f:
    f["gradSchemes"]["default"] = f["divSchemes"]["default"]
    f["snGradSchemes"]["default"] = "uncorrected"
```

### Run a case asynchronously

```python
import asyncio
from foamlib import AsyncFoamCase

async def run_case():
    my_pitz_async = AsyncFoamCase(my_pitz)
    await my_pitz_async.run()

asyncio.run(run_case())
```

### Parse a field using the [`FoamFieldFile`](https://foamlib.readthedocs.io/en/stable/#foamlib.FoamFieldFile) class directly

```python
from foamlib import FoamFieldFile

U = FoamFieldFile(Path(my_pitz) / "0/U")

print(U.internal_field)
```

### Run an optimization loop in parallel

```python
import os
from pathlib import Path
from foamlib import AsyncFoamCase
from scipy.optimize import differential_evolution

base = AsyncFoamCase(Path(os.environ["FOAM_TUTORIALS"]) / "incompressible/simpleFoam/pitzDaily")

async def cost(x):
    async with base.clone() as clone:
        clone[0]["U"].boundary_field["inlet"].value = [x[0], 0, 0]
        await clone.run()
        return abs(clone[-1]["U"].internal_field[0][0])

result = differential_evolution(cost, bounds=[(-1, 1)], workers=AsyncFoamCase.map, polish=False)
```

### Use it to create a `run` (or `clean`) script
    
```python
#!/usr/bin/env python3
from pathlib import Path
from foamlib import FoamCase

case = FoamCase(Path(__file__).parent)
# Any additional configuration here
case.run()
```

## Documentation

For more information, check out the [documentation](https://foamlib.readthedocs.io/).
