Let's proceed to **Iteration 2: User Account Management - Registration & Login Foundation**.

---

### Iteration 2: User Account Management - Registration & Login Foundation

This iteration focuses on user registration, password management, and JWT-based login.

---
**Prompt 6: User Models and Enums**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Create Pydantic models for User entity and define User Roles.

**Files to Create/Modify:**
*   `app/models/enums.py` (New file)
*   `app/models/user.py` (New file)
*   `tests/models/test_user.py` (New file, in a new `tests/models/` directory)

**Instructions for `app/models/enums.py`:**

1.  **Import `Enum` from `enum`.**
2.  **Define `UserRole(str, Enum)`:**
    *   Add roles: `VIEWER = "viewer"`, `CREATOR = "creator"`, `MODERATOR = "moderator"`.

**Instructions for `app/models/user.py`:**

1.  **Import necessary modules:** `uuid` from `uuid`, `List`, `Optional` from `typing`, `BaseModel`, `EmailStr`, `Field` from `pydantic`, `UserRole` from `app.models.enums`.
2.  **Define `UserBase(BaseModel)`:**
    *   `email: EmailStr`
    *   `first_name: Optional[str] = None`
    *   `last_name: Optional[str] = None`
3.  **Define `UserCreate(UserBase)`:**
    *   `password: str = Field(min_length=8)`
4.  **Define `UserUpdate(BaseModel)`:**
    *   `first_name: Optional[str] = None`
    *   `last_name: Optional[str] = None`
5.  **Define `UserInDBBase(UserBase)`:**
    *   `user_id: str = Field(default_factory=lambda: str(uuid.uuid4()))`
    *   `roles: List[UserRole] = Field(default_factory=lambda: [UserRole.VIEWER])`
    *   `is_active: bool = True` (Good to have for soft disables)
    *   Make sure `user_id` is properly aliased or handled for AstraDB if it uses `_id` by default for documents. For now, we'll assume `user_id` is the field name in the DB.
6.  **Define `UserInDB(UserInDBBase)`:**
    *   `hashed_password: str`
7.  **Define `UserResponse(UserInDBBase)`:**
    *   This model will represent the user data returned by API endpoints.
    *   For now, it can inherit all fields from `UserInDBBase`.
    *   Add `model_config = {"from_attributes": True}` to allow ORM mode (though we'll be using dicts from AstraPy).

**Instructions for `tests/models/test_user.py`:**

1.  **Import Pydantic models from `app.models.user` and `pytest`.**
2.  **Test `UserCreate`:**
    *   Valid data: `UserCreate(email="test@example.com", password="password123")`.
    *   Invalid data: Short password (should raise `ValidationError`).
3.  **Test `UserInDBBase`:**
    *   Ensure `user_id` is generated.
    *   Ensure default `roles` are `[UserRole.VIEWER]`.
    *   Ensure default `is_active` is `True`.
4.  **Test `UserResponse`:**
    *   Test creating an instance from a dictionary that mimics `UserInDBBase` structure.

```

---
**Prompt 7: Password Hashing Utilities**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Implement password hashing and verification utilities.

**Files to Create/Modify:**
*   `app/core/security.py` (New file)
*   `tests/core/test_security.py` (New file, in a new `tests/core/` directory)

**Instructions for `app/core/security.py`:**

1.  **Import `CryptContext` from `passlib.context`.**
2.  **Create `pwd_context` instance:**
    *   `pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")`
3.  **Define `verify_password(plain_password: str, hashed_password: str) -> bool`:**
    *   Use `pwd_context.verify(plain_password, hashed_password)`.
4.  **Define `get_password_hash(password: str) -> str`:**
    *   Use `pwd_context.hash(password)`.

**Instructions for `tests/core/test_security.py`:**

1.  **Import `verify_password`, `get_password_hash` from `app.core.security`.**
2.  **Write `test_password_hashing()`:**
    *   Generate a hash for a sample password using `get_password_hash`.
    *   Assert that the generated hash is not the same as the plain password.
3.  **Write `test_verify_password()`:**
    *   Generate a hash for a sample password.
    *   Use `verify_password` to check the correct plain password against the hash (should be `True`).
    *   Use `verify_password` to check an incorrect plain password against the hash (should be `False`).
```

---
**Prompt 8: User Service - Registration Logic (Mocked DB)**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Create the `UserService` with initial user creation logic, mocking the database interaction for now.

**Files to Create/Modify:**
*   `app/services/user_service.py` (New file)
*   `tests/services/test_user_service.py` (New file, in a new `tests/services/` directory)

**Instructions for `app/services/user_service.py`:**

1.  **Import necessary modules:** `UserCreate`, `UserInDB`, `UserResponse` from `app.models.user`, `get_password_hash` from `app.core.security`, `Optional`, `List`, `Dict` from `typing`, `HTTPException`, `status` from `fastapi`.
2.  **Define `UserService` class:**
    *   **Mocked DB Storage (temporary):** Add a class-level dictionary `_mock_db_users: Dict[str, UserInDB] = {}` to simulate database storage. This will be replaced later.
    *   **`async def get_user_by_email(self, email: str) -> Optional[UserInDB]:`**
        *   Iterate through `self._mock_db_users.values()` to find a user by email.
        *   Return the `UserInDB` object if found, else `None`.
    *   **`async def create_user(self, user_create: UserCreate) -> UserResponse:`**
        *   Check if a user with the given email already exists using `await self.get_user_by_email(user_create.email)`. If so, raise `HTTPException(status_code=status.HTTP_409_CONFLICT, detail="Email already registered")`.
        *   Hash the password using `get_password_hash(user_create.password)`.
        *   Create a `UserInDB` instance using data from `user_create` (email, first\_name, last\_name) and the `hashed_password`. The `user_id` and default `roles` will be generated by the model.
        *   Store this `UserInDB` object in `self._mock_db_users` keyed by its `user_id`.
        *   Return a `UserResponse` created from the `UserInDB` object.
    *   **Add a method to clear the mock DB for testing:** `def clear_mock_db(self): self._mock_db_users.clear()`

**Instructions for `tests/services/test_user_service.py`:**

1.  **Import necessary modules:** `pytest`, `HTTPException` from `fastapi`, `UserService` from `app.services.user_service`, `UserCreate`, `UserResponse`, `UserRole` from `app.models.user`.
2.  **Setup Pytest Fixture for `UserService`:**
    ```python
    @pytest.fixture
    def user_service():
        service = UserService()
        service.clear_mock_db() # Ensure clean state for each test
        return service
    ```
3.  **Write `test_create_new_user(user_service: UserService)`:**
    *   Create `UserCreate` data.
    *   Call `await user_service.create_user()` with the data.
    *   Assert the returned object is an instance of `UserResponse`.
    *   Assert the email matches.
    *   Assert default roles are `[UserRole.VIEWER]`.
    *   Try to retrieve the user using `await user_service.get_user_by_email()` and assert it's found and its `hashed_password` is not the plain password.
4.  **Write `test_create_duplicate_user(user_service: UserService)`:**
    *   Create a user.
    *   Attempt to create another user with the same email.
    *   Use `pytest.raises(HTTPException)` to assert that a 409 conflict occurs. Check the `status_code` of the exception.
5.  **Write `test_get_user_by_email_not_found(user_service: UserService)`:**
    *   Call `await user_service.get_user_by_email()` with a non-existent email.
    *   Assert the result is `None`.

Ensure tests are `async def` and use `await` where necessary.
```
---
**Prompt 9: Authentication API Router & Registration Endpoint**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Create the API router for authentication and implement the user registration endpoint.

**Files to Create/Modify:**
*   `app/api/v1/endpoints/auth.py` (New file)
*   `tests/api/v1/test_auth_endpoints.py` (New file, in new `tests/api/v1/` directories)

**Instructions for `app/api/v1/endpoints/auth.py`:**

1.  **Import necessary modules:** `APIRouter`, `Depends`, `HTTPException`, `status` from `fastapi`, `UserCreate`, `UserResponse` from `app.models.user`, `UserService` from `app.services.user_service`.
2.  **Create `router = APIRouter()`**.
3.  **Define `get_user_service() -> UserService` dependency:**
    *   This simple dependency will just instantiate and return `UserService()`. This allows for easier mocking in tests if needed later, or for more complex service initialization.
    ```python
    def get_user_service() -> UserService:
        return UserService()
    ```
4.  **Implement `POST /register` endpoint:**
    *   Path: `/register` (will be prefixed by `/auth` later from the main router).
    *   `response_model=UserResponse`
    *   `status_code=status.HTTP_201_CREATED`
    *   Parameters:
        *   `user_in: UserCreate` (request body)
        *   `user_service: UserService = Depends(get_user_service)`
    *   Logic: Call `await user_service.create_user(user_create=user_in)`.
    *   FastAPI will automatically handle the `HTTPException` (e.g., 409) raised by the service.

**Instructions for `tests/api/v1/test_auth_endpoints.py`:**

1.  **Import necessary modules:** `TestClient` from `fastapi.testing`, `status` from `fastapi`, `app` from `app.main` (main FastAPI app instance), `UserCreate` from `app.models.user`, `UserService` from `app.services.user_service`.
    *   **Important:** To test this endpoint effectively with the mocked `UserService` and its in-memory store, we need to ensure the `TestClient` uses a fresh `UserService` instance or that the `UserService`'s mock DB is cleared. For now, let's rely on the mock DB clearing in the `UserService` constructor or a fixture if it was global.
    *   A better approach for tests might be to override the `get_user_service` dependency in tests if complex setup is needed. For now, let's assume the default `UserService` with its in-memory store will work for isolated endpoint tests.
2.  **Create `client = TestClient(app)`**.
3.  **Write `test_register_new_user()`:**
    *   Prepare `UserCreate` data.
    *   Make a POST request to `/api/v1/auth/register` (the full path once router is included).
        *   *Note: The `/api/v1/auth` prefix is not yet defined. For this isolated test, we might need to mount the `auth.router` directly on a test app or adjust the path. Let's assume the test will use the full path that will eventually exist.*
    *   Assert status code is `201 CREATED`.
    *   Assert response JSON matches `UserResponse` structure (email, user\_id, roles).
    *   **Cleanup/Side-effect Consideration:** Since `UserService` uses a class-level mock DB, subsequent tests might be affected. Add a step to clear the mock DB *after* this test or ensure the `user_service` fixture from the previous step is used/adapted here. Perhaps by overriding dependency in `app.dependency_overrides`.
    ```python
    # Example of clearing for this specific test (if not using a fixture that handles it)
    from app.services.user_service import UserService
    UserService._mock_db_users.clear() # Direct access for testing, not ideal for prod code
    
    response = client.post("/api/v1/auth/register", json={"email": "test@example.com", "password": "password123", "first_name": "Test"})
    # assertions ...
    UserService._mock_db_users.clear() # Cleanup
    ```
4.  **Write `test_register_duplicate_user()`:**
    *   Register a user successfully first.
    *   Attempt to register the same user again.
    *   Assert status code is `409 CONFLICT`.
    *   Assert the detail message.
    *   **Cleanup:** Clear mock DB.

*Self-correction on testing path:* The endpoint path in the test should be what it will be once the router is fully integrated. For now, tests might need to adapt or we can proceed to router integration first. Let's assume router integration is next, and tests use the final path.
```
---

**Prompt 10: API Router Integration**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Integrate the authentication API router into the main v1 API router and then into the FastAPI application.

**Files to Create/Modify:**
*   `app/api/v1/api_router.py` (New file)
*   `app/main.py`
*   Update `tests/api/v1/test_auth_endpoints.py` if paths were placeholders.

**Instructions for `app/api/v1/api_router.py`:**

1.  **Import `APIRouter` from `fastapi`.**
2.  **Import the auth router:** `from app.api.v1.endpoints import auth as auth_router`
3.  **Create `api_router = APIRouter()`**.
4.  **Include the auth router:**
    *   `api_router.include_router(auth_router.router, prefix="/auth", tags=["Authentication"])`

**Instructions for `app/main.py`:**

1.  **Import `api_router as api_v1_router` from `app.api.v1.api_router`.**
2.  **Import `settings` from `app.core.config`.**
3.  **Include the v1 API router in the main `app` instance:**
    *   `app.include_router(api_v1_router, prefix=settings.API_V1_STR)`
4.  **Update OpenAPI URLs in `FastAPI` instantiation in `app/main.py`:**
    *   `openapi_url=f"{settings.API_V1_STR}/openapi.json"`
    *   `docs_url=f"{settings.API_V1_STR}/docs"` (Moves docs to be versioned)
    *   `redoc_url=f"{settings.API_V1_STR}/redoc"` (Moves redoc to be versioned)

**Instructions for `tests/api/v1/test_auth_endpoints.py`:**

1.  **Verify/Update API paths:** Ensure that the paths used in tests for the registration endpoint now correctly reflect the full path, e.g., `client.post(f"{settings.API_V1_STR}/auth/register", ...)` by importing `settings`.

**Testing:**
Advise the user to:
1.  Run the application.
2.  Navigate to `/api/v1/docs` in their browser to see the new "Authentication" tag and the `/auth/register` endpoint.
3.  Re-run the tests in `test_auth_endpoints.py` to ensure they pass with the updated paths.
```
---

**Prompt 11: User Service - AstraDB Integration for Registration**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Modify `UserService.create_user` and `get_user_by_email` to use AstraDB (`videos` collection) instead of the mock in-memory store.

**Files to Create/Modify:**
*   `app/services/user_service.py`
*   `app/db/astra_client.py` (Add `get_user_collection`)
*   `tests/services/test_user_service.py` (Update tests to mock AstraDBCollection)

**Instructions for `app/db/astra_client.py`:**

1.  **Add `get_user_collection() -> AstraDBCollection` function:**
    *   Similar to the example `get_collection` or other specific collection getters.
    *   It should call `get_collection("users", namespace=settings.ASTRA_DB_KEYSPACE)`.

**Instructions for `app/services/user_service.py`:**

1.  **Import `AstraDBCollection` from `astrapy.db` and `Depends` from `fastapi`.**
2.  **Import `get_user_collection` from `app.db.astra_client`.**
3.  **Modify `UserService.__init__`:**
    *   Remove the `_mock_db_users` class variable and `clear_mock_db` method.
    *   Add a constructor parameter `user_collection: AstraDBCollection = Depends(get_user_collection)`.
    *   Store `self.user_collection = user_collection`.
4.  **Update `async def get_user_by_email(self, email: str) -> Optional[UserInDB]:`**
    *   Use `self.user_collection.find_one({"email": email})`.
    *   If a document is found, instantiate and return `UserInDB(**document_data)`. Otherwise, return `None`.
5.  **Update `async def create_user(self, user_create: UserCreate) -> UserResponse:`**
    *   Existing logic to check for duplicates using `await self.get_user_by_email()` remains.
    *   Hashing password remains.
    *   Create the `UserInDB` instance as before.
    *   Instead of storing in `_mock_db_users`, use `self.user_collection.insert_one(db_user.model_dump(by_alias=True))`. (`model_dump` converts Pydantic model to dict. `by_alias=True` is important if you use field aliases for `_id` etc.).
    *   Return `UserResponse.model_validate(db_user)` (or `UserResponse(**db_user.model_dump())`).

**Instructions for `tests/services/test_user_service.py`:**

1.  **Import `MagicMock` from `unittest.mock`.**
2.  **Update `user_service` fixture:**
    *   It now needs to mock the `AstraDBCollection`.
    ```python
    @pytest.fixture
    def mock_user_collection():
        collection = MagicMock(spec=AstraDBCollection)
        # Pre-configure common mock return values for find_one, insert_one if tests need them
        collection.find_one.return_value = None # Default: user not found
        collection.insert_one.return_value = MagicMock(inserted_id="mock_inserted_id") # Simulate successful insert
        return collection

    @pytest.fixture
    def user_service(mock_user_collection: MagicMock):
        return UserService(user_collection=mock_user_collection)
    ```
3.  **Update `test_create_new_user`:**
    *   The `user_service` fixture now provides a service with a mocked collection.
    *   Verify `mock_user_collection.find_one` was called to check for existing user.
    *   Verify `mock_user_collection.insert_one` was called with the correct user data (hashed password, etc.).
    *   To test retrieval after creation, you might need to configure `mock_user_collection.find_one` to return the created user data on a subsequent call.
4.  **Update `test_create_duplicate_user`:**
    *   Configure `mock_user_collection.find_one` to return a dummy user document on the first call (to simulate existing user).
    *   Assert `HTTPException` (409) is raised.
    *   Ensure `mock_user_collection.insert_one` was NOT called.
5.  **Update `test_get_user_by_email_not_found`:**
    *   Ensure `mock_user_collection.find_one` is configured to return `None` (default in fixture).
    *   Verify the call and `None` result.
6.  **Add `test_get_user_by_email_found(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Create sample user data (as a dict, like it would come from DB).
    *   Configure `mock_user_collection.find_one.return_value = sample_user_data`.
    *   Call `await user_service.get_user_by_email()`.
    *   Assert the returned object is a `UserInDB` instance and matches `sample_user_data`.

**Note on AstraPy Async:** AstraPy v0.7.0+ methods like `find_one`, `insert_one` are typically synchronous. If they become async in future versions or if you use an async wrapper, ensure your service methods and tests use `await` accordingly. For now, assume synchronous DB calls from the service layer. If AstraPy methods are async, service methods must be `async` and calls `await`ed. The `MagicMock` for async methods would need to be `AsyncMock`. *Correction: AstraPy's Data API methods `find_one`, `insert_one` etc., are synchronous. So `UserService` methods making these calls do not need to be `async` purely for the DB interaction itself, unless other async operations are involved.* Let's keep them `async` for consistency with FastAPI endpoint structure and potential future async operations. Tests using `MagicMock` for synchronous methods are fine.
```
---
**Prompt 12: JWT Utilities & Token Model**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Implement JWT creation/decoding utilities and Pydantic models for tokens.

**Files to Create/Modify:**
*   `app/models/token.py` (New file)
*   `app/core/security.py` (Modify)
*   `app/core/config.py` (Verify/Ensure JWT settings)
*   `tests/core/test_security.py` (Modify)

**Instructions for `app/models/token.py`:**

1.  **Import `BaseModel` from `pydantic` and `List`, `Optional` from `typing`.**
2.  **Define `Token(BaseModel)`:**
    *   `access_token: str`
    *   `token_type: str` (default to "bearer")
3.  **Define `TokenData(BaseModel)`:**
    *   `user_id: Optional[str] = None`
    *   `roles: Optional[List[str]] = None` (To store roles in the token payload)

**Instructions for `app/core/config.py`:**
1.  Ensure these settings are present (they should be from Prompt 2):
    *   `SECRET_KEY: str`
    *   `ALGORITHM: str = "HS256"`
    *   `ACCESS_TOKEN_EXPIRE_MINUTES: int = 60 * 24 * 7`

**Instructions for `app/core/security.py`:**

1.  **Import necessary modules:** `datetime`, `timedelta`, `timezone` from `datetime`, `JWTError`, `jwt` from `jose`, `settings` from `app.core.config`, `TokenData` from `app.models.token`.
2.  **Define `create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str`:**
    *   `to_encode = data.copy()`
    *   Calculate expiry time: `datetime.now(timezone.utc) + expires_delta` or use `settings.ACCESS_TOKEN_EXPIRE_MINUTES`.
    *   `to_encode.update({"exp": expire})`
    *   `encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)`
    *   Return `encoded_jwt`.
3.  **Define `decode_access_token(token: str) -> Optional[TokenData]:`**
    *   Try to decode: `payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])`.
    *   Extract `user_id = payload.get("sub")` (this will be the convention for user ID in token).
    *   Extract `roles = payload.get("roles", [])`.
    *   If `user_id` is `None`, return `None`.
    *   Return `TokenData(user_id=user_id, roles=roles)`.
    *   Catch `JWTError` and return `None`.

**Instructions for `tests/core/test_security.py`:**

1.  **Import `create_access_token`, `decode_access_token`, `settings`, `TokenData`.**
2.  **Write `test_create_and_decode_access_token()`:**
    *   Prepare sample data for token: `{"sub": "test_user_id", "roles": ["viewer", "creator"]}`.
    *   Create a token using `create_access_token(data=sample_data)`.
    *   Decode the token using `decode_access_token(token)`.
    *   Assert that the decoded `TokenData` is not `None`.
    *   Assert `decoded_token.user_id` matches "test\_user\_id".
    *   Assert `decoded_token.roles` matches `["viewer", "creator"]`.
3.  **Write `test_decode_expired_token()`:**
    *   Create a token with a very short expiry (e.g., `expires_delta=timedelta(seconds=-1)` or `timedelta(milliseconds=1)` and then sleep).
    *   Attempt to decode it.
    *   Assert the result is `None` or raises an appropriate exception handled by `decode_access_token`.
4.  **Write `test_decode_invalid_token()`:**
    *   Attempt to decode a garbage string.
    *   Assert the result is `None`.
```
---
**Prompt 13: User Login Endpoint**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Implement the user login endpoint, which authenticates users and returns a JWT.

**Files to Create/Modify:**
*   `app/api/v1/endpoints/auth.py` (Modify)
*   `app/services/user_service.py` (Add authentication method)
*   `tests/api/v1/test_auth_endpoints.py` (Add login tests)
*   `tests/services/test_user_service.py` (Add tests for authentication method)

**Instructions for `app/services/user_service.py`:**

1.  **Import `verify_password` from `app.core.security`, `UserInDB` from `app.models.user`.**
2.  **Add `async def authenticate_user(self, email: str, password: str) -> Optional[UserInDB]:` method to `UserService`:**
    *   Get user by email using `await self.get_user_by_email(email)`.
    *   If user not found or `user.is_active` is False (important check!), return `None`.
    *   Verify password using `verify_password(password, user.hashed_password)`.
    *   If password matches, return the `user` object (`UserInDB`). Otherwise, return `None`.

**Instructions for `app/api/v1/endpoints/auth.py`:**

1.  **Import `OAuth2PasswordRequestForm` from `fastapi.security`, `Token` from `app.models.token`, `create_access_token` from `app.core.security`, `timedelta` from `datetime`, `settings` from `app.core.config`.**
2.  **Implement `POST /login` endpoint:**
    *   Path: `/login` (will be prefixed by `/auth`).
    *   `response_model=Token`.
    *   Parameters:
        *   `form_data: OAuth2PasswordRequestForm = Depends()`
        *   `user_service: UserService = Depends(get_user_service)`
    *   Logic:
        *   Call `db_user = await user_service.authenticate_user(email=form_data.username, password=form_data.password)`. (Note: `form_data.username` is used for email).
        *   If `db_user` is `None`, raise `HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Incorrect email or password", headers={"WWW-Authenticate": "Bearer"})`.
        *   Prepare token data: `token_payload = {"sub": db_user.user_id, "roles": [role.value for role in db_user.roles]}`.
        *   Create access token: `access_token = create_access_token(data=token_payload)`.
        *   Return `Token(access_token=access_token, token_type="bearer")`.

**Instructions for `tests/services/test_user_service.py`:**

1.  **Import `get_password_hash` from `app.core.security`.**
2.  **Write `test_authenticate_user_success(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Sample plain password. Hash it.
    *   Prepare mock user data (dict) including the hashed password, email, and `is_active=True`.
    *   Configure `mock_user_collection.find_one.return_value = mock_user_data`.
    *   Call `await user_service.authenticate_user(email=mock_user_data['email'], password=plain_password)`.
    *   Assert a `UserInDB` object is returned and its details match.
3.  **Write `test_authenticate_user_wrong_password(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Prepare mock user data with a hashed password.
    *   Configure `mock_user_collection.find_one.return_value = mock_user_data`.
    *   Call `await user_service.authenticate_user()` with the correct email but wrong password.
    *   Assert the result is `None`.
4.  **Write `test_authenticate_user_not_found(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Configure `mock_user_collection.find_one.return_value = None`.
    *   Call `await user_service.authenticate_user()`.
    *   Assert the result is `None`.
5.  **Write `test_authenticate_user_inactive(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Prepare mock user data with `is_active=False`.
    *   Configure `mock_user_collection.find_one.return_value = mock_user_data`.
    *   Call `await user_service.authenticate_user()`.
    *   Assert the result is `None`.

**Instructions for `tests/api/v1/test_auth_endpoints.py`:**

1.  **Import `OAuth2PasswordRequestForm` (though `TestClient` sends form data differently).**
2.  **Add `test_login_success()`:**
    *   First, register a user via the API or directly mock the `UserService.authenticate_user` method for this test. For a true integration test, let's register a user.
        *   Use `client.post(f"{settings.API_V1_STR}/auth/register", ...)` to create a user. Store the plain password.
    *   Make a POST request to `f"{settings.API_V1_STR}/auth/login"` with `data={"username": "user_email", "password": "user_plain_password"}`.
    *   Assert status code is `200 OK`.
    *   Assert response JSON contains `access_token` and `token_type="bearer"`.
    *   Optionally, decode the token to verify its contents (user\_id, roles).
    *   Cleanup: `UserService._mock_db_users.clear()` if still using any shared mock state from registration for simplicity, or ensure proper DB mocking if `UserService` is fully DB integrated in tests. Since `UserService` is now using a mocked AstraDB collection via fixture, this direct clear won't work. We'd need to mock the `authenticate_user` method of the *instance* of `UserService` that the endpoint receives, or ensure the mocked DB for `UserService` is set up correctly by the test.
        *   For this endpoint test, it's cleaner to mock the `user_service.authenticate_user` method if the test is focused *only* on the login endpoint logic (token creation) and not the full auth flow. Or, ensure the mocked `get_user_by_email` in the `user_service`'s mocked collection returns a user compatible with login.
3.  **Add `test_login_failure_wrong_password()`:**
    *   Register a user.
    *   Attempt login with correct email, wrong password.
    *   Assert status code is `401 UNAUTHORIZED`.
    *   Assert detail message.
4.  **Add `test_login_failure_user_not_found()`:**
    *   Attempt login with non-existent user email.
    *   Assert status code is `401 UNAUTHORIZED`.

*Testing Strategy for Login Endpoint:* Given `UserService` now uses a `mock_user_collection` via fixture, the `test_login_success` should:
    1. Define what `mock_user_collection.find_one` should return when `authenticate_user` (which calls `get_user_by_email`) is invoked. This mock user should have a known hashed password.
    2. Then call the login endpoint with the corresponding plain password.
```

---
**Prompt 14: Authenticated User Dependency**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Implement a FastAPI dependency to get the current authenticated user from a JWT.

**Files to Create/Modify:**
*   `app/api/v1/dependencies.py` (New file)
*   `app/core/security.py` (Ensure `decode_access_token` is robust)
*   `app/services/user_service.py` (Add `get_user_by_id`)

**Instructions for `app/services/user_service.py`:**

1.  **Add `async def get_user_by_id(self, user_id: str) -> Optional[UserInDB]:` method to `UserService`:**
    *   Use `self.user_collection.find_one({"user_id": user_id})`.
    *   If found, return `UserInDB(**document_data)`. Else, `None`.

**Instructions for `app/api/v1/dependencies.py`:**

1.  **Import necessary modules:** `Depends`, `HTTPException`, `status` from `fastapi`, `OAuth2PasswordBearer` from `fastapi.security`, `settings` from `app.core.config`, `decode_access_token` from `app.core.security`, `TokenData` from `app.models.token`, `UserInDB` from `app.models.user`, `UserService` from `app.services.user_service`.
    *   Import `get_user_service` from `app.api.v1.endpoints.auth` (or move `get_user_service` to `dependencies.py` if it's general). Let's assume for now it's fine in `auth.py` and we import it.
2.  **Define `oauth2_scheme`:**
    *   `oauth2_scheme = OAuth2PasswordBearer(tokenUrl=f"{settings.API_V1_STR}/auth/login")` (Matches the login endpoint path).
3.  **Define `async def get_current_user(token: str = Depends(oauth2_scheme), user_service: UserService = Depends(get_user_service)) -> UserInDB:`:**
    *   Define `credentials_exception = HTTPException(...)` for 401 Unauthorized.
    *   Decode the token: `token_data: Optional[TokenData] = decode_access_token(token)`.
    *   If `token_data` is `None` or `token_data.user_id` is `None`, raise `credentials_exception`.
    *   Fetch the user from DB: `user = await user_service.get_user_by_id(user_id=token_data.user_id)`.
    *   If `user` is `None` or `not user.is_active`, raise `credentials_exception`.
    *   Important: The roles from `token_data.roles` should be considered authoritative for the session, as they were set at login. You might want to assign `user.roles = [UserRole(role_str) for role_str in token_data.roles]` if the `UserInDB` model's roles are from the DB and could be stale relative to the token.
    *   Return the `user` (`UserInDB` instance).
4.  **Define `async def get_current_active_user(current_user: UserInDB = Depends(get_current_user)) -> UserInDB:`:**
    *   This is a convenience re-export or can add more checks. For now, it just ensures the user fetched by `get_current_user` is implicitly active due to checks within `get_current_user`. (The `is_active` check is already in `get_current_user`). So, this can simply be:
    ```python
    async def get_current_active_user(current_user: UserInDB = Depends(get_current_user)) -> UserInDB:
        # The get_current_user dependency already checks for active status.
        # Add more checks here if needed, e.g., email verified, etc.
        return current_user
    ```

**Testing `get_current_user`:**
*   This dependency is typically tested implicitly through protected endpoints. We will create a protected endpoint next.
*   Unit testing it directly would involve mocking `oauth2_scheme` (to provide a token), `decode_access_token`, and `user_service.get_user_by_id`.
```
---
**Prompt 15: Protected Endpoint Example & User Profile (`/users/me`)**
```text
Continuing with the KillrVideo 2025 project (Iteration 2):

**Task:** Create a protected endpoint (`/users/me`) to retrieve the current user's profile, demonstrating the `get_current_user` dependency. Also, implement user profile update.

**Files to Create/Modify:**
*   `app/api/v1/endpoints/users.py` (New file)
*   `app/api/v1/api_router.py` (Modify to include users router)
*   `tests/api/v1/test_users_endpoints.py` (New file)
*   `app/services/user_service.py` (Add update_user method)
*   `tests/services/test_user_service.py` (Add tests for update_user)

**Instructions for `app/services/user_service.py`:**
1.  **Import `UserUpdate` from `app.models.user`.**
2.  **Add `async def update_user(self, user_id: str, user_update_data: UserUpdate) -> Optional[UserResponse]:`**
    *   Fetch the user by `user_id` using `self.user_collection.find_one({"user_id": user_id})`. If not found, return `None`.
    *   Create a dictionary `update_data = user_update_data.model_dump(exclude_unset=True)` to get only fields that were provided.
    *   If `update_data` is empty, return `UserResponse.model_validate(db_user_data)`.
    *   Perform the update in DB: `updated_doc = self.user_collection.find_one_and_update({"user_id": user_id}, {"$set": update_data}, return_document=True)`. AstraPy's `find_one_and_update` might need `ReturnDocument.AFTER` from `pymongo.collection` if that's its underlying mechanism, or check AstraPy's specific way to return the updated document. If it doesn't return the full doc, you might need to fetch it again or merge.
        *   *AstraPy v0.7.1 `find_one_and_update` does not have `return_document`. You'll need to `update_one` then `find_one` or manage this carefully. Let's simplify: use `update_one` and then fetch the user again for the response.*
        ```python
        # In UserService.update_user
        result = self.user_collection.update_one({"user_id": user_id}, {"$set": update_data})
        if result.modified_count == 0 and not update_data: # No change if no data or no actual modification
             # If no actual fields were to be updated, and result.matched_count > 0
             # we can assume it's "updated" to its current state.
             # Or if update_data was empty.
             pass # or some specific logic if needed.
        elif result.modified_count == 0 and result.matched_count == 0: # User not found by update
            return None
            
        updated_db_user = await self.get_user_by_id(user_id)
        if not updated_db_user: return None # Should not happen if update was successful
        return UserResponse.model_validate(updated_db_user)
        ```
    *   If update successful, return `UserResponse.model_validate(updated_user_data_from_db)`.

**Instructions for `app/api/v1/endpoints/users.py`:**

1.  **Import `APIRouter`, `Depends` from `fastapi`, `UserResponse`, `UserUpdate` from `app.models.user`, `UserInDB` from `app.models.user`, `get_current_active_user` from `app.api.v1.dependencies`, `UserService` from `app.services.user_service`, and `get_user_service` from `app.api.v1.endpoints.auth` (or its new location if moved).**
2.  **Create `router = APIRouter()`**.
3.  **Implement `GET /me` endpoint:**
    *   Path: `/me`.
    *   `response_model=UserResponse`.
    *   Parameter: `current_user: UserInDB = Depends(get_current_active_user)`.
    *   Logic: Simply return `current_user`. FastAPI will convert `UserInDB` to `UserResponse` based on the `response_model`.
4.  **Implement `PUT /me` endpoint:**
    *   Path: `/me`.
    *   `response_model=UserResponse`.
    *   Parameters:
        *   `user_update: UserUpdate` (request body)
        *   `current_user: UserInDB = Depends(get_current_active_user)`
        *   `user_service: UserService = Depends(get_user_service)`
    *   Logic:
        *   Call `updated_user = await user_service.update_user(user_id=current_user.user_id, user_update_data=user_update)`.
        *   If `updated_user` is `None` (should not happen if user exists), raise 404.
        *   Return `updated_user`.

**Instructions for `app/api/v1/api_router.py`:**

1.  **Import the users router:** `from app.api.v1.endpoints import users as users_router`.
2.  **Include the users router:**
    *   `api_router.include_router(users_router.router, prefix="/users", tags=["Users"])`.

**Instructions for `tests/services/test_user_service.py`:**
1.  **Write `test_update_user_success(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Mock `mock_user_collection.find_one` to return initial user data.
    *   Mock `mock_user_collection.update_one` to simulate success (`MagicMock(modified_count=1, matched_count=1)`).
    *   Mock `mock_user_collection.find_one` again (for the re-fetch after update) to return the *updated* user data.
    *   Call `await user_service.update_user()`.
    *   Assert the returned `UserResponse` has updated fields. Verify `update_one` was called with correct `$set` payload.
2.  **Write `test_update_user_no_changes(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Pass empty `UserUpdate` data.
    *   Mock `find_one` to return user data.
    *   `update_one` should not be called if data is empty, or if it is, `modified_count` might be 0.
    *   Assert the original user data is returned.
3.  **Write `test_update_user_not_found(user_service: UserService, mock_user_collection: MagicMock)`:**
    *   Mock `mock_user_collection.find_one` (initial fetch for `update_one`) to return `None`.
    *   Call `await user_service.update_user()`.
    *   Assert result is `None`.

**Instructions for `tests/api/v1/test_users_endpoints.py`:**

1.  **Import `TestClient`, `settings`, `status`, `UserResponse`, `UserUpdate`.**
2.  **Helper function to get a valid token:**
    ```python
    def get_valid_token(client: TestClient, email: str = "testuser@example.com", password: str = "password123") -> str:
        # Assumes user is registered or mocks allow login
        # For a real test, you might register the user first via API
        # For simplicity here, we might need to mock the user service for login if registration is complex in test setup
        login_data = {"username": email, "password": password}
        response = client.post(f"{settings.API_V1_STR}/auth/login", data=login_data)
        assert response.status_code == 200
        return response.json()["access_token"]
    ```
    *This helper needs a way to ensure a user exists for login. For true endpoint tests, register a user first.*
3.  **Write `test_get_current_user_me_success(client: TestClient)`:**
    *   Register a test user (e.g., `testgetme@example.com`) via `POST /auth/register`.
    *   Log in as that user to get a token using the helper or directly.
    *   Make a GET request to `f"{settings.API_V1_STR}/users/me"` with the `Authorization: Bearer <token>` header.
    *   Assert status code is `200 OK`.
    *   Assert response JSON matches `UserResponse` and contains the correct user email.
4.  **Write `test_get_current_user_me_no_token(client: TestClient)`:**
    *   Make a GET request to `/users/me` without the Authorization header.
    *   Assert status code is `401 UNAUTHORIZED`.
5.  **Write `test_get_current_user_me_invalid_token(client: TestClient)`:**
    *   Make a GET request to `/users/me` with an invalid token.
    *   Assert status code is `401 UNAUTHORIZED`.
6.  **Write `test_update_current_user_me_success(client: TestClient)`:**
    *   Register and log in a user to get a token.
    *   Prepare `UserUpdate` data (e.g., new `first_name`).
    *   Make a PUT request to `/users/me` with the token and JSON payload.
    *   Assert status code is `200 OK`.
    *   Assert response contains the updated `first_name`.
    *   Optionally, call `GET /users/me` again to verify persisted change.
7.  **Write `test_update_current_user_me_no_token(client: TestClient)`:**
    *   Make PUT request without token. Assert 401.

**Final Testing for Iteration 2:**
Advise the user to:
1. Run the application.
2. Use `/api/v1/docs` to:
    a. Register a new user.
    b. Log in with that user (copy the access token).
    c. Authorize in Swagger UI using the token.
    d. Test `GET /users/me`.
    e. Test `PUT /users/me`.
3. Re-run all tests: `poetry run pytest`.
```

This completes the prompts for Iteration 2. Each step builds carefully on the last, ensuring that services are tested with mocks before being integrated into API endpoints, and API endpoints are tested for both success and failure cases, including authentication. The database interaction is mocked at the service test level, allowing focus on the service logic itself.