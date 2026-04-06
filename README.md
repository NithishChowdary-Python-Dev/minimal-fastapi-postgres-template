# Minimal FastAPI PostgreSQL template

[![Live example](https://img.shields.io/badge/live%20example-https%3A%2F%2Fminimal--fastapi--postgres--template.rafsaf.pl-blueviolet)](https://minimal-fastapi-postgres-template.rafsaf.pl/)
[![License](https://img.shields.io/github/license/NithishChowdary-Python-Dev/minimal-fastapi-postgres-template)](https://github.com/NithishChowdary-Python-Dev/minimal-fastapi-postgres-template/blob/main/LICENSE)
[![Python 3.14](https://img.shields.io/badge/python-3.14-blue)](https://docs.python.org/3/whatsnew/3.14.html)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
[![Tests](https://github.com/NithishChowdary-Python-Dev/minimal-fastapi-postgres-template/actions/workflows/tests.yml/badge.svg)](https://github.com/NithishChowdary-Python-Dev/minimal-fastapi-postgres-template/actions/workflows/tests.yml)

_Check out the online example: [https://minimal-fastapi-postgres-template.rafsaf.pl](https://minimal-fastapi-postgres-template.rafsaf.pl). This demonstration uses the exact code provided in this template, configured with a custom domain and HTTPS._

- [Minimal FastAPI PostgreSQL template](#minimal-fastapi-postgresql-template)
  - [About](#about)
  - [Features](#features)
  - [Quickstart](#quickstart)
    - [1. Create repository from a template](#1-create-repository-from-a-template)
    - [2. Install dependencies with uv](#2-install-dependencies-with-uv)
    - [3. Run app](#3-run-app)
    - [4. Activate pre-commit](#4-activate-pre-commit)
    - [5. Running tests](#5-running-tests)
  - [Step by step example - POST and GET endpoints](#step-by-step-example---post-and-get-endpoints)
    - [1. Create new app](#1-create-new-app)
    - [2. Create SQLAlchemy model](#2-create-sqlalchemy-model)
    - [3. Import new models.py file in alembic env.py](#3-import-new-modelspy-file-in-alembic-envpy)
    - [4. Create and apply alembic migration](#4-create-and-apply-alembic-migration)
    - [5. Create request and response schemas](#5-create-request-and-response-schemas)
    - [6. Create endpoints](#6-create-endpoints)
    - [7. Add Pet model to tests factories](#7-add-pet-model-to-tests-factories)
    - [8. Create new test file](#8-create-new-test-file)
    - [9. Write tests](#9-write-tests)
  - [Design choices](#design-choices)
    - [Dockerfile](#dockerfile)
    - [Registration](#registration)
    - [Delete user endpoint](#delete-user-endpoint)
    - [JWT and refresh tokens](#jwt-and-refresh-tokens)
    - [Writing scripts / cron](#writing-scripts--cron)
    - [Docs URL](#docs-url)
    - [CORS](#cors)
    - [Allowed Hosts](#allowed-hosts)
  - [License](#license)
  - [Maintainer](#maintainer)

## About

For technical rationale and the latest architectural changes, refer to the 2026 update documentation: [Update of minimal-fastapi-postgres-template to version 7.0.0](https://rafsaf.pl/blog/2026/02/07/update-of-minimal-fastapi-postgres-template-to-version-7.0.0/).

## Features

- [x] Template repository.
- [x] [SQLAlchemy](https://github.com/sqlalchemy/sqlalchemy) 2.0, async queries, best possible autocompletion support.
- [x] PostgreSQL 18 database under [asyncpg](https://github.com/MagicStack/asyncpg) interface.
- [x] Full [Alembic](https://github.com/alembic/alembic) migrations setup (also in unit tests).
- [x] Secure and tested setup for [PyJWT](https://github.com/jpadilla/pyjwt) and [bcrypt](https://github.com/pyca/bcrypt).
- [x] Ready to go Dockerfile with [uvicorn](https://www.uvicorn.org/) webserver.
- [x] [uv](https://docs.astral.sh/uv/getting-started/installation/), [mypy](https://github.com/python/mypy), [pre-commit](https://github.com/pre-commit/pre-commit) hooks with [ruff](https://github.com/astral-sh/ruff).
- [x] Perfect pytest asynchronous test setup and full coverage.

![template-fastapi-minimal-openapi-example](https://rafsaf.pl/blog/2026/02/07/update-of-minimal-fastapi-postgres-template-to-version-7.0.0/minimal-fastapi-postgres-template-2026-02-07-version-7.0.0.png)

## Quickstart

### 1. Create repository from a template

See the official documentation or use git clone to get started with this repository.

### 2. Install dependencies with [uv](https://docs.astral.sh/uv/getting-started/installation/)

```bash
cd your_project_name

uv sync
```

Uv should automatically install the Python version currently required by the template (>=3.14) or use an existing Python installation if available.

### 3. Run app

```bash
make up
```

Refer to the `Makefile` to see shortcuts (requires `build-essential` on Linux).

If you prefer to work without the Makefile:

```bash
docker compose up -d

alembic upgrade head

uvicorn app.main:app --reload
```

Initialize the git repository if needed and access the OpenAPI spec at [http://localhost:8000/](http://localhost:8000/) by default.

### 4. Activate pre-commit

[pre-commit](https://pre-commit.com/) is the standard for pre-push activities. Refer to the `.pre-commit-config.yaml` file to see the current opinionated choices for linting and formatting.

```bash
# Shortcut
make lint
```

Full commands:

```bash
# Install pre-commit
pre-commit install --install-hooks

# Run on all files
pre-commit run --all-files
```

### 5. Running tests

The test suite creates databases for the session and runs tests in parallel by default (using pytest-xdist) to optimize execution based on available CPU resources.

For details about the initial database setup, see the logic in the `app/conftest.py` file, specifically the `fixture_setup_new_test_database` function. Pytest configuration is located in `[tool.pytest.ini_options]` within `pyproject.toml`.

The project includes a coverage plugin with a required code coverage level of 100%.

```bash
# See all pytest configuration flags in pyproject.toml
pytest

# or 
make test
```

## Step by step example - POST and GET endpoints

This template includes a practical example to demonstrate the workflow and save development time. Let's create two example endpoints:

- `POST` endpoint `/pets/create` for creating `Pets` with a relation to the currently logged `User`.
- `GET` endpoint `/pets/me` for fetching all user's pets.

### 1. Create new app

Add the `app/pets` folder and `app/pets/__init__.py`.

### 2. Create SQLAlchemy model

Add the `Pet` model to `app/pets/models.py`.

```python
# app/pets/models.py

import sqlalchemy as sa
from sqlalchemy.orm import Mapped, mapped_column

from app.core.models import Base


class Pet(Base):
    __tablename__ = "pets_pet"

    id: Mapped[int] = mapped_column(sa.BigInteger, primary_key=True)
    user_id: Mapped[str] = mapped_column(
        sa.ForeignKey("auth_user.user_id", ondelete="CASCADE"),
    )
    pet_name: Mapped[str] = mapped_column(sa.String(50), nullable=False)
```

This uses SQLAlchemy 2.0 syntax with `Mapped` and `mapped_column`.

### 3. Import new models.py file in alembic env.py

In `alembic/env.py`, import the new file so Alembic can track changes:

```python
# alembic/env.py

(...) 
# import other models here
import app.pets.models  # noqa

(...)
```

### 4. Create and apply alembic migration

```bash
# Create migration with alembic revision
alembic revision --autogenerate -m "create_pet_model"

# Apply migration using alembic upgrade
alembic upgrade head
```

### 5. Create request and response schemas

```python
# app/pets/schemas.py

from pydantic import BaseModel, ConfigDict


class PetCreateRequest(BaseModel):
    pet_name: str


class PetResponse(BaseModel):
    id: int
    pet_name: str
    user_id: str

    model_config = ConfigDict(from_attributes=True)
```

### 6. Create endpoints

```python
# app/pets/views.py

from fastapi import APIRouter, Depends, status
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.auth.dependencies import get_current_user
from app.auth.models import User
from app.core import database_session
from app.pets.models import Pet
from app.pets.schemas import PetCreateRequest, PetResponse

router = APIRouter()


@router.post(
    "/create",
    response_model=PetResponse,
    status_code=status.HTTP_201_CREATED,
    description="Creates new pet. Only for logged users.",
)
async def create_new_pet(
    data: PetCreateRequest,
    session: AsyncSession = Depends(database_session.new_async_session),
    current_user: User = Depends(get_current_user),
) -> Pet:
    new_pet = Pet(user_id=current_user.user_id, pet_name=data.pet_name)

    session.add(new_pet)
    await session.commit()

    return new_pet


@router.get(
    "/me",
    response_model=list[PetResponse],
    status_code=status.HTTP_200_OK,
    description="Get list of pets for currently logged user.",
)
async def get_all_my_pets(
    session: AsyncSession = Depends(database_session.new_async_session),
    current_user: User = Depends(get_current_user),
) -> list[Pet]:
    pets = await session.scalars(
        select(Pet).where(Pet.user_id == current_user.user_id).order_by(Pet.pet_name)
    )

    return list(pets.all())
```

Add the router to `main.py`:

```python
# main.py
from app.pets.views import router as pets_router
app.include_router(pets_router, prefix="/pets", tags=["pets"])
```

### 7. Add Pet model to tests factories

```python
# app/tests/factories.py
from app.pets.models import Pet

class PetFactory(SQLAlchemyFactory[Pet]):
    pet_name = Use(Faker().first_name)
```

### 8. Create new test file

Create `app/pet/tests/test_pets_views.py`.

### 9. Write tests

```python
# app/pet/tests/test_pets_views.py

from fastapi import status
from httpx import AsyncClient

from app.auth.models import User
from app.main import app
from app.tests.factories import PetFactory


async def test_create_new_pet(
    client: AsyncClient, default_user_headers: dict[str, str], default_user: User
) -> None:
    response = await client.post(
        app.url_path_for("create_new_pet"),
        headers=default_user_headers,
        json={"pet_name": "Tadeusz"},
    )
    assert response.status_code == status.HTTP_201_CREATED

    result = response.json()
    assert result["user_id"] == default_user.user_id
    assert result["pet_name"] == "Tadeusz"
```

## Design choices

The following sections describe the core design decisions and architectural choices maintained in this project.

### Dockerfile

The template includes a `Dockerfile` with [Uvicorn](https://www.uvicorn.org/). This setup is secure for production use when placed behind a properly configured load balancer or proxy with HTTPS enabled.

### Registration

User registration is open by default. This can be restricted or removed based on specific project requirements.

### Delete user endpoint

The `delete_current_user` endpoint is provided for completeness; evaluate if this is necessary for your specific application.

### JWT and refresh tokens

Users exchange credentials for a JWT via `/auth/access-token`. Refresh tokens are stored in a database table, allowing for token revocation. This implementation focuses on a secure and maintainable setup compared to common alternatives.

### Writing scripts / cron

For background tasks or scripts requiring database access outside of the FastAPI request cycle, use `new_script_async_session`.

### Docs URL

The documentation page is served at the root `/` by default.

```python
app = FastAPI(
    title="minimal fastapi postgres template",
    version="7.0.0",
    description="https://github.com/NithishChowdary-Python-Dev/minimal-fastapi-postgres-template",
    openapi_url="/openapi.json",
    docs_url="/",
)
```

### CORS

Configure `BACKEND_CORS_ORIGINS` in the `.env` file to include your frontend domain.

### Allowed Hosts

The `TrustedHostMiddleware` is used to prevent HTTP Host Header attacks. Configure `ALLOWED_HOSTS` with your server IP or domain.

## License

The code is available for use and modification under the terms of the MIT License.

## Maintainer

**Nithish Chowdary**
Senior Python Developer / AI/ML Engineer

Senior Python Developer with over 10 years of experience in building scalable backend services and microservices using FastAPI, Django, and Flask. Specialized in deploying high-performance systems and advanced analytics solutions.

- **GitHub**: [https://github.com/NithishChowdary-Python-Dev](https://github.com/NithishChowdary-Python-Dev)
- **LinkedIn**: [http://www.linkedin.com/in/nithishchowdary-python](http://www.linkedin.com/in/nithishchowdary-python)
- **Email**: nithishmc1@gmail.com