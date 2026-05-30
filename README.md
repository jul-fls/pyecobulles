# pyecobulles

[![CI](https://github.com/jul-fls/pyecobulles/actions/workflows/ci.yml/badge.svg)](https://github.com/jul-fls/pyecobulles/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/jul-fls/pyecobulles/branch/master/graph/badge.svg)](https://codecov.io/gh/jul-fls/pyecobulles)
[![PyPI](https://img.shields.io/pypi/v/pyecobulles.svg)](https://pypi.org/project/pyecobulles/)

`pyecobulles` is an unofficial async Python client for the Ecobulles cloud API.

Ecobulles is a French CO2-based water treatment system used as an alternative to salt water softeners. Connected Ecobulles boxes expose water consumption, CO2 injection counters, device metadata, and alert information through the same cloud API used by the mobile/application ecosystem.

This library provides a small, reusable Python interface to that cloud API. It is intentionally focused on transport and raw API access so higher-level projects, such as Home Assistant integrations, can decide how to model sensors, statistics, diagnostics, and user-facing behavior.

## Features

- Async API client built for `aiohttp`.
- Login with Ecobulles account credentials.
- Fetch Ecobulles device metadata.
- Fetch cumulative water usage and raw CO2/gas counter data.
- Fetch the full login payload for integrations that need alert details.
- No credential storage: callers own secrets, sessions, persistence, and scheduling.
- Typed package (`py.typed`) for downstream users.

## Installation

```bash
pip install pyecobulles
```

## Quick start

```python
import asyncio

from aiohttp import ClientSession
from pyecobulles import EcobullesClient


async def main() -> None:
    async with ClientSession() as session:
        client = EcobullesClient(session=session)

        authenticated, user_id, eco_ref, box_name = await client.authenticate(
            "user@example.com",
            "password",
        )
        if not authenticated or eco_ref is None:
            raise RuntimeError("Ecobulles authentication failed")

        usage = await client.get_total_water_and_co2_usage(eco_ref)
        device = await client.get_device_info(eco_ref)

        print(f"Box: {box_name}")
        print(f"User id: {user_id}")
        print(f"Eco ref: {eco_ref}")
        print(f"Usage: {usage}")
        print(f"Device: {device}")


asyncio.run(main())
```

## API overview

### `EcobullesClient(session, now_fn=None)`

Create a client using an injected `aiohttp.ClientSession`.

The caller owns the session lifecycle. This keeps the library suitable for applications that already manage connection pooling, such as Home Assistant.

`now_fn` is optional and mainly useful for tests. It controls the stop date used by `get_total_water_and_co2_usage()`.

### `authenticate(email, password)`

Authenticate against the Ecobulles cloud API.

Returns:

```python
(authenticated, user_id, eco_ref, box_name)
```

where:

- `authenticated` is a boolean;
- `user_id` is the Ecobulles account/user identifier when available;
- `eco_ref` is the Ecobulles box reference used by other endpoints;
- `box_name` is the box name returned by the API when available.

### `get_login_payload(email, password)`

Return the raw login payload.

This is useful for integrations that need fields exposed only in the login response, such as current alert information.

### `get_device_info(eco_ref)`

Return raw device metadata for an Ecobulles box.

The payload may include fields such as:

- box name;
- serial number;
- firmware version;
- installation date;
- last cloud receive date;
- activation / lock / suspension status;
- alerts, depending on the endpoint response.

### `get_total_water_and_co2_usage(eco_ref)`

Return normalized cumulative usage data:

```python
{
    "total_gas": 72996000,
    "total_eau": 193651,
    "last_updated": "2026-05-21T21:17:58",
}
```

Current understanding of the fields:

- `total_eau` is the water usage reported by the Ecobulles API, in liters.
- `total_gas` appears to be the cumulative CO2 electrovalve open time, in milliseconds, not a direct mass of CO2.
- `last_updated` is derived from the latest graph entry when available.

The library does not convert CO2 valve-open time into kilograms. That requires product-specific assumptions or calibration and should be handled by the application using the library.

## Design philosophy

`pyecobulles` deliberately keeps the API representation close to what the Ecobulles cloud returns. It does not try to turn raw payloads into Home Assistant entities, dashboards, statistics, or long-term water accounting.

That separation keeps the package useful outside Home Assistant and avoids hiding API behavior behind assumptions that may not hold for every installation.

## Known limitations

- This is an unofficial client and is not affiliated with Ecobulles.
- The Ecobulles cloud API is not publicly documented.
- Endpoint behavior may change without notice.
- Model/gamme information is not currently exposed clearly by the observed API payloads.
- CO2 mass usage is not directly exposed by the API; only the raw gas counter is available.

## Home Assistant

This package is used by the custom Home Assistant integration [`ha_ecobulles`](https://github.com/jul-fls/ha_ecobulles).

Home Assistant-specific behavior, such as sensors, diagnostics, translations, water-cycle accounting, and estimated CO2 bottle usage, belongs in that integration rather than in this library.

## Development

For contributors:

```bash
python -m pip install -r requirements_dev.txt
pytest -q --cov
mypy pyecobulles
python -m build
```

## License

Apache-2.0
