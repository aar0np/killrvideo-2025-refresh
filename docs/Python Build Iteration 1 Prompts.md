## KillrVideo 2025 Python Backend: Development Blueprint

**Overall Guiding Principles:**

*   **Test-Driven Development (TDD):** Write tests before or alongside implementation.
*   **Incremental Progress:** Each step should result in a small, working, and testable addition.
*   **Clean Code & Best Practices:** Adhere to Python typing, Pydantic validation, FastAPI conventions, and SOLID principles where applicable.
*   **Modularity:** Keep components decoupled (APIs, services, DB interactions).
*   **Configuration Management:** Use Pydantic settings for environment-driven configuration.
*   **Security First:** Implement security features (hashing, JWTs, RBAC) early and correctly.
*   **AstraPy Integration:** Gradually integrate AstraPy for Data API interactions.

---

### Iteration 1: Project Foundation & Core Setup

This iteration focuses on setting up the project structure, basic FastAPI application, configuration, and initial database client.

**Steps:**

1.  **Project Initialization & Core Dependencies:**
    *   Initialize Poetry.
    *   Add FastAPI, Uvicorn, Pydantic, Pydantic-Settings, python-dotenv.
    *   Create the basic directory structure (`app/`, `app/core/`, `app/main.py`, `tests/`).
2.  **Configuration Management:**
    *   Implement `app/core/config.py` with `Settings` (APP\_NAME, API\_V1\_STR, ENVIRONMENT).
    *   Create `.env.example` and `.env`.
3.  **Basic FastAPI Application:**
    *   Create FastAPI app instance in `app/main.py`.
    *   Add a root `GET /` health check endpoint.
    *   Write a test for the root endpoint.
4.  **Logging Setup:**
    *   Configure basic logging in `app/main.py` or a separate `app/core/logging_config.py`.
5.  **AstraDB Configuration & Client Shell:**
    *   Add AstraDB related variables to `app/core/config.py` and `.env*` files.
    *   Create `app/db/astra_client.py` with `get_astra_db()` to initialize `AstraDB` and functions like `get_collection(name: str)` returning `AstraDBCollection`.
    *   Add a startup event in `app/main.py` to initialize/test the DB connection (log success/failure).

---

### Iteration 2: User Account Management - Registration & Login Foundation

Focuses on user registration, password management, and JWT-based login.

**Steps:**

1.  **User Models & Enums:**
    *   Create `app/models/enums.py` (e.g., `UserRole`).
    *   Create `app/models/user.py` with Pydantic models: `UserBase`, `UserCreate`, `UserInDBBase`, `UserInDB` (with `hashed_password`), `UserResponse`.
    *   Write unit tests for model validation.
2.  **Password Hashing Utilities:**
    *   Implement `app/core/security.py` with `get_password_hash()` and `verify_password()` using `passlib`.
    *   Write unit tests for these functions.
3.  **User Service - Registration Logic (In-Memory/Mocked DB):**
    *   Create `app/services/user_service.py` with `UserService`.
    *   Implement `create_user(user_data: UserCreate)`:
        *   Hashes password.
        *   *Initially mocks DB interaction* (e.g., stores in a class-level list or uses a MagicMock).
        *   Returns `UserResponse`.
    *   Write unit tests for `UserService.create_user` with a mocked DB.
4.  **Authentication API Router & Registration Endpoint:**
    *   Create `app/api/v1/endpoints/auth.py` with an `APIRouter`.
    *   Implement `POST /register` endpoint using `UserService`.
    *   Write integration tests using `TestClient` (with service layer mocked initially if complex, or allow passthrough to in-memory service).
5.  **API Router Integration:**
    *   Create `app/api/v1/api_router.py` to include the `auth_router`.
    *   Include `v1_api_router` in `app/main.py` prefixed with `settings.API_V1_STR`.
    *   Test the `/api/v1/auth/register` endpoint.
6.  **User Service - AstraDB Integration for Registration:**
    *   Modify `UserService.create_user` to use `get_user_collection()` from `astra_client.py` for `find_one` (check existing) and `insert_one`.
    *   Handle potential duplicate email errors (HTTP 409 Conflict).
    *   Update/add integration tests focusing on DB interaction (mocking `AstraDBCollection` methods).
7.  **JWT Utilities & Token Model:**
    *   Add JWT settings to `app/core/config.py`.
    *   Create `app/models/token.py` with `Token` and `TokenData` Pydantic models.
    *   Implement `create_access_token()` and `decode_access_token()` in `app/core/security.py` using `python-jose`.
    *   Write unit tests for JWT functions.
8.  **User Login Endpoint:**
    *   Implement `POST /login` in `app/api/v1/endpoints/auth.py`.
        *   It should take `OAuth2PasswordRequestForm` (from `fastapi.security`).
        *   Authenticate user (fetch from DB via `UserService`, verify password).
        *   Generate and return JWT (`Token` model).
    *   Write integration tests for login.
9.  **Authenticated User Dependency:**
    *   Create `app/api/v1/dependencies.py`.
    *   Implement `get_current_user(token: str = Depends(oauth2_scheme))` using `decode_access_token` and fetching user from DB.
    *   Write tests for this dependency (can be tricky, often tested implicitly via protected endpoints).
10. **Protected Endpoint Example & User Profile:**
    *   Add a `GET /users/me` endpoint in a new `app/api/v1/endpoints/users.py` router that uses `get_current_user` and returns `UserResponse`.
    *   Integrate `users_router` into `api_router.py`.
    *   Write integration tests for `/users/me`.
11. **User Profile Update:**
    *   Add `UserUpdate` Pydantic model in `app/models/user.py`.
    *   Implement `update_user` method in `UserService`.
    *   Implement `PUT /users/me` endpoint in `users.py` to update first/last name.
    *   Write tests for profile update.

---

### Iteration 3: Video Management - Core CRUD & Async Foundation

Focuses on submitting videos, retrieving them, and setting up for asynchronous processing.

**Steps:**

1.  **Video Models:**
    *   Create `app/models/video.py` with `VideoBase`, `VideoCreate` (YouTube URL, optional title/desc/tags), `VideoInDB`, `VideoResponse`, `VideoSummary` (for listings), `VideoProcessingStatus`.
    *   Write unit tests for model validation.
2.  **Video Service - Submission & Async Placeholder:**
    *   Create `app/services/video_service.py` with `VideoService`.
    *   Implement `submit_video(video_data: VideoCreate, current_user: UserInDB, background_tasks: BackgroundTasks)`:
        *   Stores initial video data (video\_id, uploader\_id, YouTube URL, status="PROCESSING", submitted\_at) in AstraDB (`videos` collection).
        *   Adds a *placeholder* background task `process_new_video(video_id: str, youtube_url: str)` (this task will initially just log or simulate work).
        *   Returns `VideoProcessingStatus`.
    *   Write unit tests for `submit_video` (mocking DB and `BackgroundTasks.add_task`).
3.  **Video Endpoints - Submission & Status:**
    *   Create `app/api/v1/endpoints/videos.py` router.
    *   Implement `POST /videos` using `VideoService.submit_video` and `get_current_active_creator` dependency (to be created).
    *   Implement `GET /videos/{video_id}/status` that fetches the video document from DB and returns its status and details (as `VideoProcessingStatus` or similar).
    *   Write integration tests.
4.  **Role-Based Access Control (RBAC) Dependencies:**
    *   Add `get_current_active_creator` and `get_current_active_moderator` to `app/api/v1/dependencies.py`. (Initially, "creator" might just be any authenticated user, or you can add a "creator" role).
    *   Update `POST /videos` to use `get_current_active_creator`.
5.  **Video Service & Endpoint - Get Video Details:**
    *   Implement `get_video_by_id(video_id: str)` in `VideoService`.
    *   Implement `GET /videos/{video_id}` endpoint returning `VideoResponse`.
    *   Write tests.
6.  **Video Service & Endpoint - List Latest Videos:**
    *   Implement `list_latest_videos(page: int, page_size: int)` in `VideoService` (query AstraDB, sort by `submitted_at`).
    *   Create `app/models/common.py` with `PaginatedResponse[T]`.
    *   Implement `GET /videos/latest` endpoint returning `PaginatedResponse[VideoSummary]`.
    *   Write tests.
7.  **Video Playback Statistics - Record View:**
    *   Add `view_count: int = 0` to `VideoInDB` and `VideoResponse`.
    *   Implement `record_video_view(video_id: str)` in `VideoService` using AstraDB's `$inc` operator to increment `view_count`.
    *   Implement `POST /videos/{video_id}/views` endpoint (204 No Content).
    *   Write tests.

---

This covers the first few iterations. Subsequent iterations (Comments & Ratings, Search, Moderation, Actual AI Integration, etc.) would follow a similar pattern: Models -> Service Logic (mocked external parts) -> Endpoints -> DB Integration -> External Service Integration -> Testing at each sub-step.

## LLM Prompts for Code Generation (Iterative & TDD-focused)

Here are the prompts, designed to be fed to a code-generation LLM one by one. Each prompt builds on the output of the previous ones.

---

**Prompt 1: Project Initialization and Core Dependencies**
```text
You are an expert Python developer specializing in FastAPI applications. Your task is to initialize a new Python project for "KillrVideo 2025" using Poetry and set up the basic application structure and core dependencies.

**Instructions:**

1.  **Create `pyproject.toml`:**
    *   Initialize a Poetry project.
    *   Set project name to `killrvideo-2025-python`, version `1.0.0`.
    *   Add the following dependencies:
        *   `python = "^3.10"`
        *   `fastapi = "^0.110.0"` (or latest stable)
        *   `uvicorn = {extras = ["standard"], version = "^0.29.0"}`
        *   `pydantic = "^2.7.0"`
        *   `pydantic-settings = "^2.2.0"`
        *   `python-dotenv = "^1.0.0"`
        *   `astrapy = "^0.7.1"`
        *   `passlib = {extras = ["bcrypt"], version = "^1.7.4"}`
        *   `python-jose = {extras = ["cryptography"], version = "^3.3.0"}`
        *   `httpx = "^0.27.0"`
    *   Add the following development dependencies (`--group dev`):
        *   `pytest = "^8.0.0"`
        *   `pytest-asyncio = "^0.23.0"`
        *   `mypy = "^1.9.0"`
        *   `ruff = "^0.4.0"`
    *   Configure `ruff` for linting (line-length 88, select E, W, F, I, C, B, ignore E501) and formatting (double quotes).
    *   Configure `mypy` (python_version 3.10, warn_return_any, ignore_missing_imports=true, pydantic.mypy plugin).

2.  **Create Directory Structure:**
    Create the following initial directory structure. Placeholders for `__init__.py` should be created where appropriate to make them Python packages.
    ```
    killrvideo-2025-python/
    ├── app/
    │   ├── __init__.py
    │   ├── core/
    │   │   └── __init__.py
    │   ├── db/
    │   │   └── __init__.py
    │   ├── models/
    │   │   └── __init__.py
    │   ├── services/
    │   │   └── __init__.py
    │   ├── api/
    │   │   ├── __init__.py
    │   │   └── v1/
    │   │       ├── __init__.py
    │   │       ├── endpoints/
    │   │       │   └── __init__.py
    │   │       └── __init__.py
    │   ├── main.py
    ├── tests/
    │   ├── __init__.py
    ├── .gitignore  # Add typical Python .gitignore contents (.env, __pycache__, .pytest_cache, .mypy_cache, .ruff_cache, .venv etc.)
    ├── README.md   # Basic placeholder
    └── pyproject.toml # Generated by poetry, you'll fill/verify it
    ```

3.  **Provide `poetry lock --no-update` and `poetry install --all-extras` commands** as the next steps for the user.

Ensure all Python files have basic imports (like `from typing import ...`) if needed, even if empty.
The `app/main.py` file can be empty for now.
```

---

**Prompt 2: Configuration Management**
```text
Continuing with the KillrVideo 2025 project:

**Task:** Implement configuration management using Pydantic's `BaseSettings`.

**File to Create/Modify:** `app/core/config.py`

**Instructions:**

1.  **Define `EnvironmentEnum`:**
    *   Create an `Enum` named `EnvironmentEnum(str, Enum)` with values: `DEVELOPMENT = "development"`, `PRODUCTION = "production"`, `TESTING = "testing"`.
2.  **Define `Settings` class:**
    *   Create a class `Settings(BaseSettings)`.
    *   Add the following configuration variables with type hints and default values where appropriate:
        *   `APP_NAME: str = "KillrVideo 2025"`
        *   `ENVIRONMENT: EnvironmentEnum = EnvironmentEnum.DEVELOPMENT`
        *   `API_V1_STR: str = "/api/v1"`
        *   `# AstraDB Settings (to be filled by .env)`
        *   `ASTRA_DB_ID: str`
        *   `ASTRA_DB_REGION: str`
        *   `ASTRA_DB_APPLICATION_TOKEN: str`
        *   `ASTRA_DB_KEYSPACE: str`
        *   `# JWT Settings (to be filled by .env or generated)`
        *   `SECRET_KEY: str`  # Remind user: use `openssl rand -hex 32`
        *   `ALGORITHM: str = "HS256"`
        *   `ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24 * 7` # 7 days
    *   Configure `model_config = SettingsConfigDict(env_file=".env", extra="ignore")` to load from a `.env` file.
3.  **Instantiate Settings:**
    *   Create an instance `settings = Settings()`.
4.  **Create `.env.example`:**
    Create this file in the project root with placeholder values for all environment-specific variables defined in `Settings` (especially ASTRA\_DB\_\* and SECRET\_KEY).
    ```
    # .env.example
    ENVIRONMENT="development"

    ASTRA_DB_ID="your-astra-db-id"
    ASTRA_DB_REGION="your-astra-db-region" # e.g., us-east1
    ASTRA_DB_APPLICATION_TOKEN="AstraCS:your-token:..."
    ASTRA_DB_KEYSPACE="your_keyspace_name"

    SECRET_KEY="generate_a_strong_secret_key_here" # openssl rand -hex 32
    ACCESS_TOKEN_EXPIRE_MINUTES=10080
    ```
5.  **Instruct the user to create their own `.env` file** by copying `.env.example` and filling in their actual credentials.
Ensure necessary imports (`from pydantic_settings import BaseSettings, SettingsConfigDict`, `from enum import Enum`).
```

---

**Prompt 3: Basic FastAPI Application and Root Endpoint**
```text
Continuing with the KillrVideo 2025 project:

**Task:** Set up a basic FastAPI application instance and a root health check endpoint.

**Files to Create/Modify:**
*   `app/main.py`
*   `tests/test_main.py` (New file)

**Instructions for `app/main.py`:**

1.  **Import necessary modules:** `FastAPI` from `fastapi`, `settings` from `app.core.config`.
2.  **Create FastAPI app instance:**
    ```python
    app = FastAPI(
        title=settings.APP_NAME,
        openapi_url=f"{settings.API_V1_STR}/openapi.json", # Note: API_V1_STR will be used later for versioned router
        docs_url="/docs", # For now, docs at root, will move later
        redoc_url="/redoc" # For now, redoc at root
    )
    ```
3.  **Add Root Endpoint:**
    *   Create a `GET /` endpoint.
    *   It should return a JSON response: `{"message": f"Welcome to {settings.APP_NAME}"}`.
4.  **Add CORS Middleware (Development only for now):**
    *   Import `CORSMiddleware`.
    *   If `settings.ENVIRONMENT == "development"`, add `CORSMiddleware` allowing all origins, methods, and headers.

**Instructions for `tests/test_main.py`:**

1.  **Import necessary modules:** `TestClient` from `fastapi.testing`, `app` from `app.main`.
2.  **Create a test client instance:** `client = TestClient(app)`.
3.  **Write a test function `test_read_root()`:**
    *   It should make a GET request to `/`.
    *   Assert that the status code is 200.
    *   Assert that the JSON response is `{"message": "Welcome to KillrVideo 2025"}` (or dynamically check against `settings.APP_NAME`).

**Provide Uvicorn Run Command:**
Remind the user they can run the app using: `poetry run uvicorn app.main:app --reload`
```

---
**Prompt 4: Logging Setup**
```text
Continuing with the KillrVideo 2025 project:

**Task:** Implement basic logging configuration.

**Files to Create/Modify:**
*   `app/core/logging_config.py` (New file)
*   `app/main.py`

**Instructions for `app/core/logging_config.py`:**

1.  **Import `logging` and `sys`.**
2.  **Define a function `setup_logging()`:**
    *   Get the root logger: `logger = logging.getLogger()`.
    *   Set logger level to `logging.INFO`.
    *   Create a `StreamHandler` for `sys.stdout`.
    *   Create a `Formatter` with a suitable format, e.g., `"%(asctime)s - %(name)s - %(levelname)s - %(message)s"`.
    *   Set the formatter for the handler.
    *   Add the handler to the root logger.
    *   Add a log message like `logging.info("Logging configured.")` inside this function.

**Instructions for `app/main.py`:**

1.  **Import `setup_logging` from `app.core.logging_config`.**
2.  **Call `setup_logging()`** at the beginning of the file (after imports, before FastAPI app instantiation).
3.  **Add a simple log message within the `startup_event` (to be created in next step) or root endpoint** to test if logging works, e.g., `logging.info("FastAPI application starting up...")`.

**Testing:**
Advise the user to run the application and check the console output for the log messages.
```

---
**Prompt 5: AstraDB Configuration and Client Shell with Startup Event**
```text
Continuing with the KillrVideo 2025 project:

**Task:** Implement the AstraDB client setup and add a startup event to initialize/test the connection.

**Files to Create/Modify:**
*   `app/db/astra_client.py` (New file)
*   `app/main.py`

**Instructions for `app/db/astra_client.py`:**

1.  **Import necessary modules:** `AstraDB`, `AstraDBCollection` from `astrapy.db`, `settings` from `app.core.config`, `logging`.
2.  **Global DB Instance:** Define `db_instance: Optional[AstraDB] = None`.
3.  **`get_astra_db()` function:**
    *   Implement `get_astra_db() -> AstraDB`.
    *   It should use the global `db_instance`. If `db_instance` is `None`, initialize it:
        ```python
        db_instance = AstraDB(
            token=settings.ASTRA_DB_APPLICATION_TOKEN,
            api_endpoint=f"https://{settings.ASTRA_DB_ID}-{settings.ASTRA_DB_REGION}.apps.astra.datastax.com"
        )
        logging.info(f"AstraDB client initialized for DB ID: {settings.ASTRA_DB_ID}")
        ```
    *   Return `db_instance`.
4.  **`get_collection(collection_name: str, namespace: str = None) -> AstraDBCollection` function:**
    *   Get the `AstraDB` instance using `get_astra_db()`.
    *   Use the provided `namespace` or default to `settings.ASTRA_DB_KEYSPACE`.
    *   Return `AstraDBCollection(collection_name=collection_name, astra_db=db, namespace=namespace_to_use)`.
    *   Log the attempt to get a collection.
5.  **`check_db_connection()` async function:**
    *   Get the `AstraDB` instance.
    *   Attempt a simple operation, e.g., `await db.get_collections(namespace=settings.ASTRA_DB_KEYSPACE)`. This is an async call from AstraPy 0.7.0+.
    *   Return `True` if successful, `False` otherwise. Handle exceptions and log them.

**Instructions for `app/main.py`:**

1.  **Import `check_db_connection` from `app.db.astra_client` and `logging`.**
2.  **Add Startup Event `on_event("startup")`:**
    *   Make it an `async` function.
    *   Inside the event, call `connected = await check_db_connection()`.
    *   Log "Successfully connected to AstraDB." if `connected` is `True`.
    *   Log "Failed to connect to AstraDB. Please check credentials and configuration." if `False`.
    *   This will test the connection when the app starts.

**Testing:**
Advise the user to:
1.  Ensure their `.env` file has correct AstraDB credentials.
2.  Run the application and check the startup logs for the connection status message.
```
---

This sequence of prompts will guide the LLM to build the foundation step-by-step, always with a focus on what's next and how it integrates. We'll continue this pattern for User Models, Security, Services, Endpoints, and further iterations.