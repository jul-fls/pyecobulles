# pyecobulles

[![CI](https://github.com/jul-fls/pyecobulles/actions/workflows/ci.yml/badge.svg)](https://github.com/jul-fls/pyecobulles/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/jul-fls/pyecobulles/branch/master/graph/badge.svg)](https://codecov.io/gh/jul-fls/pyecobulles)

Async Python client for the Ecobulles cloud API.

This repository started as reverse-engineering notes for the Ecobulles mobile API and now hosts the reusable Python package needed by the Home Assistant integration.

## Install

```bash
pip install pyecobulles
```

## Usage

```python
import asyncio
from aiohttp import ClientSession
from pyecobulles import EcobullesClient

async def main():
    async with ClientSession() as session:
        client = EcobullesClient(session=session)
        ok, user_id, eco_ref, name = await client.authenticate("email", "password")
        if not ok or eco_ref is None:
            raise RuntimeError("Authentication failed")
        usage = await client.get_total_water_and_co2_usage(eco_ref)
        device = await client.get_device_info(eco_ref)
        print(name, usage, device)

asyncio.run(main())
```

## Development

```bash
python -m pip install -r requirements_dev.txt
pytest -q --cov
mypy pyecobulles
python -m build
```

## Publishing

Publishing is automated with PyPI Trusted Publishing through `.github/workflows/publish.yml`.

Manual setup required once:

1. Create a PyPI account.
2. Create a GitHub environment named `pypi`.
3. In PyPI, configure Trusted Publishing for:
   - owner: `jul-fls`
   - repository: `ecobulles_api`
   - workflow: `publish.yml`
   - environment: `pypi`
4. Create a GitHub Release to publish the package.

No long-lived PyPI API token is needed when Trusted Publishing is configured.

