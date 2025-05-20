Okay, let's move to **Iteration 3: Video Management - Core CRUD & Async Foundation**.

This iteration focuses on submitting videos, retrieving them, and setting up the foundation for asynchronous processing (like thumbnail generation, AI tagging, etc.).

---

### Iteration 3: Video Management - Core CRUD & Async Foundation

---
**Prompt 16: Video Models**
```text
Continuing with the KillrVideo 2025 project (Iteration 3):

**Task:** Create Pydantic models for Video entities.

**Files to Create/Modify:**
*   `app/models/video.py` (New file)
*   `tests/models/test_video.py` (New file)
*   `app/models/enums.py` (Add `VideoProcessingStatusEnum`)

**Instructions for `app/models/enums.py`:**

1.  **Add `VideoProcessingStatusEnum(str, Enum)`:**
    *   Values: `PROCESSING = "PROCESSING"`, `COMPLETED = "COMPLETED"`, `FAILED = "FAILED"`.

**Instructions for `app/models/video.py`:**

1.  **Import necessary modules:** `uuid` from `uuid`, `List`, `Optional`, `Dict` from `typing`, `HttpUrl` (for YouTube URL), `BaseModel`, `Field` from `pydantic`, `datetime` from `datetime`, `UserResponse` from `app.models.user` (for uploader info), `VideoProcessingStatusEnum` from `app.models.enums`.
2.  **Define `VideoBase(BaseModel)`:**
    *   `title: Optional[str] = Field(None, max_length=255)`
    *   `description: Optional[str] = Field(None, max_length=5000)`
    *   `tags: Optional[List[str]] = Field(None, max_items=50)` # Max 50 tags, each tag some max length?
3.  **Define `VideoCreate(VideoBase)`:**
    *   `you_tube_url: HttpUrl` # Pydantic will validate if it's a valid HTTP/HTTPS URL
    *   *User can optionally provide title, description, tags here. If not, they'll be fetched/generated.*
4.  **Define `VideoProcessingDetails(BaseModel)`:** (Fields populated after async processing)
    *   `youtube_video_id: Optional[str] = None`
    *   `fetched_title: Optional[str] = None`
    *   `fetched_description: Optional[str] = None`
    *   `thumbnail_url: Optional[HttpUrl] = None`
    *   `embed_html: Optional[str] = None` # Or just store YouTube ID and construct on frontend
    *   `duration_seconds: Optional[int] = None`
    *   `suggested_tags: Optional[List[str]] = None`
    *   `vector_embedding: Optional[List[float]] = None` # For AI features
    *   `failure_reason: Optional[str] = None`
    *   `processed_at: Optional[datetime] = None`
5.  **Define `VideoInDBBase(VideoBase)`:**
    *   `video_id: str = Field(default_factory=lambda: str(uuid.uuid4()))`
    *   `uploader_id: str` # Will be linked to User.user_id
    *   `you_tube_url: HttpUrl` # Store the original URL
    *   `status: VideoProcessingStatusEnum = VideoProcessingStatusEnum.PROCESSING`
    *   `submitted_at: datetime = Field(default_factory=datetime.utcnow)`
    *   `view_count: int = 0`
    *   `# Fields from VideoProcessingDetails will be added here after processing`
    *   `processing_details: Optional[VideoProcessingDetails] = None`
    *   `# Effective fields (merged from user input and processing)`
    *   `effective_title: Optional[str] = None` # Merged from user_provided and fetched
    *   `effective_description: Optional[str] = None`
    *   `effective_tags: Optional[List[str]] = None`
    *   `model_config = {"from_attributes": True}`
6.  **Define `VideoInDB(VideoInDBBase)`:**
    *   This can be an alias or inherit if more fields are needed only in DB representation that are not in Base. For now, `VideoInDB = VideoInDBBase` might suffice or `pass`.
7.  **Define `VideoResponse(VideoInDBBase)`:**
    *   `uploader: Optional[UserResponse] = None` # To embed uploader info.
    *   *This will show the current state of the video, including its processing status and details if completed.*
    *   Exclude fields like `vector_embedding` from general response unless specifically needed.
    *   Fields to show: `video_id`, `effective_title`, `effective_description`, `effective_tags`, `thumbnail_url` (from processing\_details), `uploader`, `submitted_at`, `view_count`, `status`, `duration_seconds` (from processing\_details), `youtube_video_id` (from processing\_details).
    *   You might need to use Pydantic's `computed_field` or `model_validator` to present these fields neatly from `processing_details`.
    ```python
    # Example for VideoResponse to make fields from processing_details top-level
    class VideoResponse(VideoInDBBase):
        uploader: Optional[UserResponse] = None # Populated by service

        # Expose processed details more directly
        youtube_video_id: Optional[str] = Field(None, validation_alias=AliasPath("processing_details", "youtube_video_id"))
        thumbnail_url: Optional[HttpUrl] = Field(None, validation_alias=AliasPath("processing_details", "thumbnail_url"))
        # ... other fields like duration_seconds, etc.

        class Config:
            populate_by_name = True # Allows using AliasPath effectively

    # For VideoSummary, pick fewer fields
    class VideoSummary(BaseModel):
        video_id: str
        effective_title: Optional[str] = None
        thumbnail_url: Optional[HttpUrl] = None # Will need careful population
        uploader_name: Optional[str] = None # Simplified uploader info
        view_count: int = 0
        submitted_at: datetime
        duration_seconds: Optional[int] = None
        status: VideoProcessingStatusEnum

        model_config = {"from_attributes": True}
    ```
8.  **Define `VideoProcessingStatusResponse(BaseModel)`:**
    *   `video_id: str`
    *   `status: VideoProcessingStatusEnum`
    *   `message: Optional[str] = None`
    *   `status_url: Optional[str] = None` # Link to GET /videos/{video_id}/status or GET /videos/{video_id}
    *   `details: Optional[VideoProcessingDetails] = None` # Show processing details if available

**Instructions for `tests/models/test_video.py`:**

1.  **Import Pydantic models from `app.models.video` and `pytest`.**
2.  **Test `VideoCreate`:**
    *   Valid data: `VideoCreate(you_tube_url="https://www.youtube.com/watch?v=dQw4w9WgXcQ", title="Test Video")`.
    *   Invalid URL (should raise `ValidationError`).
3.  **Test `VideoInDBBase`:**
    *   Ensure `video_id` is generated.
    *   Ensure default `status` is `PROCESSING`.
    *   Ensure `submitted_at` is set.
4.  **Test `VideoResponse` and `VideoSummary` (if using `AliasPath` or validators, test those behaviors):**
    *   Test creating instances from dictionaries mimicking `VideoInDBBase` structure, especially how nested `processing_details` map to top-level fields in `VideoResponse`.

```

---
**Prompt 17: Video Service - Submission & Async Placeholder**
```text
Continuing with the KillrVideo 2025 project (Iteration 3):

**Task:** Create the `VideoService` with initial video submission logic. This will include storing the video in AstraDB with a "PROCESSING" status and adding a *placeholder* background task for further processing.

**Files to Create/Modify:**
*   `app/services/video_service.py` (New file)
*   `tests/services/test_video_service.py` (New file)
*   `app/db/astra_client.py` (Add `get_video_collection`)

**Instructions for `app/db/astra_client.py`:**

1.  **Add `get_video_collection() -> AstraDBCollection` function:**
    *   Calls `get_collection("videos", namespace=settings.ASTRA_DB_KEYSPACE)`.

**Instructions for `app/services/video_service.py`:**

1.  **Import necessary modules:** `uuid`, `datetime`, `timezone` from `datetime`, `BackgroundTasks`, `Depends`, `HTTPException`, `status` from `fastapi`, `AstraDBCollection` from `astrapy.db`, `VideoCreate`, `VideoInDBBase`, `VideoProcessingStatusResponse`, `VideoProcessingDetails` from `app.models.video`, `UserInDB` from `app.models.user`, `VideoProcessingStatusEnum` from `app.models.enums`, `get_video_collection` from `app.db.astra_client`, `logging`.
2.  **Define `VideoService` class:**
    *   Constructor: `def __init__(self, video_collection: AstraDBCollection = Depends(get_video_collection)): self.video_collection = video_collection`.
    *   **Placeholder `async def _process_new_video_task(self, video_id: str, youtube_url: str):`**
        *   `logging.info(f"Background task started for video_id: {video_id}, URL: {youtube_url}")`
        *   Simulate work: `import asyncio; await asyncio.sleep(5)` (remove asyncio for real task)
        *   *This task will eventually fetch YouTube metadata, generate embeddings, etc.*
        *   For now, after simulating work, it could update the video status in DB to "COMPLETED" (mock update) or "FAILED" randomly for testing.
        ```python
        # Example mock update within _process_new_video_task
        try:
            # ... simulate fetching ...
            mock_details = VideoProcessingDetails(
                youtube_video_id="dQw4w9WgXcQ_mock",
                fetched_title="Mock Fetched Title",
                thumbnail_url="https://example.com/mock.jpg",
                processed_at=datetime.now(timezone.utc)
            )
            update_doc = {
                "status": VideoProcessingStatusEnum.COMPLETED.value,
                "processing_details": mock_details.model_dump(exclude_none=True),
                "effective_title": mock_details.fetched_title # Or merge logic
            }
            self.video_collection.update_one({"video_id": video_id}, {"$set": update_doc})
            logging.info(f"Mock processing COMPLETED for video_id: {video_id}")
        except Exception as e:
            logging.error(f"Mock processing FAILED for video_id: {video_id}: {e}")
            update_doc = {
                "status": VideoProcessingStatusEnum.FAILED.value,
                "processing_details": VideoProcessingDetails(
                    failure_reason=str(e),
                    processed_at=datetime.now(timezone.utc)
                ).model_dump(exclude_none=True)
            }
            self.video_collection.update_one({"video_id": video_id}, {"$set": update_doc})
        ```
    *   **`async def submit_video(self, video_create: VideoCreate, current_user: UserInDB, background_tasks: BackgroundTasks) -> VideoProcessingStatusResponse:`**
        *   Generate `video_id = str(uuid.uuid4())`.
        *   Create initial `VideoInDBBase` data:
            ```python
            initial_video_data = VideoInDBBase(
                video_id=video_id,
                uploader_id=current_user.user_id,
                you_tube_url=video_create.you_tube_url,
                title=video_create.title, # User provided initial title
                description=video_create.description, # User provided initial desc
                tags=video_create.tags, # User provided initial tags
                status=VideoProcessingStatusEnum.PROCESSING,
                submitted_at=datetime.now(timezone.utc),
                # Initialize effective fields with user provided data or title from URL if possible
                effective_title=video_create.title or "Video Processing...", # Placeholder
            )
            ```
        *   Store in DB: `self.video_collection.insert_one(initial_video_data.model_dump(exclude_none=True))`.
        *   Add background task: `background_tasks.add_task(self._process_new_video_task, video_id, str(video_create.you_tube_url))`.
        *   Return `VideoProcessingStatusResponse(video_id=video_id, status=VideoProcessingStatusEnum.PROCESSING, message="Video submission accepted and is being processed.", status_url=f"/api/v1/videos/{video_id}")` (status URL will point to full details, not just status sub-resource for now).
    *   **`async def get_video_processing_status(self, video_id: str) -> Optional[VideoProcessingStatusResponse]:`**
        *   Fetch video doc from `self.video_collection.find_one({"video_id": video_id})`.
        *   If not found, return `None`.
        *   Construct and return `VideoProcessingStatusResponse` using data from the document (status, video\_id, processing\_details if any).
        *   If `status == COMPLETED`, message can be "Processing completed."
        *   If `status == FAILED`, message can be `processing_details.failure_reason`.

**Instructions for `tests/services/test_video_service.py`:**

1.  **Import `MagicMock`, `AsyncMock` from `unittest.mock`, `pytest`, `BackgroundTasks`, `VideoCreate`, `UserInDB`, `VideoProcessingStatusEnum`, `VideoService`.**
2.  **Fixtures:**
    ```python
    @pytest.fixture
    def mock_video_collection():
        collection = MagicMock(spec=AstraDBCollection)
        collection.insert_one.return_value = MagicMock(inserted_id="mock_video_id")
        collection.find_one.return_value = None # Default
        return collection

    @pytest.fixture
    def mock_background_tasks():
        return MagicMock(spec=BackgroundTasks)
    
    @pytest.fixture # Assuming UserInDB model is available and can be instantiated
    def sample_user():
        return UserInDB(user_id="uploader123", email="uploader@example.com", hashed_password="xxx", roles=["creator"])


    @pytest.fixture
    def video_service(mock_video_collection: MagicMock):
        return VideoService(video_collection=mock_video_collection)
    ```
3.  **Write `test_submit_video(video_service: VideoService, sample_user: UserInDB, mock_background_tasks: MagicMock, mock_video_collection: MagicMock)`:**
    *   Prepare `VideoCreate` data.
    *   Call `await video_service.submit_video(video_data, sample_user, mock_background_tasks)`.
    *   Assert the response is `VideoProcessingStatusResponse` with status `PROCESSING`.
    *   Assert `mock_video_collection.insert_one` was called with correct initial data (status PROCESSING, uploader\_id, etc.).
    *   Assert `mock_background_tasks.add_task` was called with `video_service._process_new_video_task`, `video_id`, and URL.
4.  **Write `test_get_video_processing_status_processing(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   Mock `mock_video_collection.find_one` to return a video document with `status: "PROCESSING"`.
    *   Call `await video_service.get_video_processing_status()`.
    *   Assert response status is `PROCESSING`.
5.  **Write `test_get_video_processing_status_completed(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   Mock `find_one` to return a document with `status: "COMPLETED"` and some `processing_details`.
    *   Call service method. Assert correct response.
6.  **Write `test_get_video_processing_status_not_found(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   `mock_video_collection.find_one.return_value = None`.
    *   Call service method. Assert `None`.
7.  **Write `test_process_new_video_task_mock_success(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   Call `await video_service._process_new_video_task("test_vid_id", "http://some.url")`.
    *   Assert `mock_video_collection.update_one` was called to set status to `COMPLETED` and update `processing_details`. (Need to inspect the `$set` part of the call).
    *   *This tests the placeholder task logic. Real task testing will be more involved.*

```

---
**Prompt 18: Video Endpoints - Submission & Status**
```text
Continuing with the KillrVideo 2025 project (Iteration 3):

**Task:** Create API endpoints for submitting a new video and checking its processing status.

**Files to Create/Modify:**
*   `app/api/v1/endpoints/videos.py` (New file)
*   `app/api/v1/api_router.py` (Include video router)
*   `tests/api/v1/test_videos_endpoints.py` (New file)
*   `app/api/v1/dependencies.py` (Add `get_current_active_creator` if not already robust)

**Instructions for `app/api/v1/dependencies.py` (Ensure/Refine `get_current_active_creator`):**
1.  **Import `UserRole` from `app.models.enums`.**
2.  **Define/Refine `async def get_current_active_creator(current_user: UserInDB = Depends(get_current_active_user)) -> UserInDB:`**
    *   Check if `UserRole.CREATOR` or `UserRole.MODERATOR` is in `current_user.roles`.
    *   If not, raise `HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="User does not have Creator privileges.")`.
    *   Return `current_user`.
    *   *(Initially, we might allow any authenticated user to be a creator. This role check can be enforced now or later based on how roles are assigned. For now, let's assume a "creator" role needs to exist on the user).*

**Instructions for `app/api/v1/endpoints/videos.py`:**

1.  **Import `APIRouter`, `Depends`, `BackgroundTasks`, `HTTPException`, `status` from `fastapi`, `VideoCreate`, `VideoProcessingStatusResponse` from `app.models.video`, `UserInDB` from `app.models.user`, `VideoService` from `app.services.video_service`, `get_current_active_creator` from `app.api.v1.dependencies`.**
2.  **Create `router = APIRouter()`**.
3.  **Define `get_video_service() -> VideoService` dependency (similar to `get_user_service`).**
4.  **Implement `POST /` (prefixed with `/videos`) endpoint for submission:**
    *   `response_model=VideoProcessingStatusResponse`
    *   `status_code=status.HTTP_202_ACCEPTED`
    *   Parameters:
        *   `video_in: VideoCreate`
        *   `current_user: UserInDB = Depends(get_current_active_creator)`
        *   `background_tasks: BackgroundTasks`
        *   `video_service: VideoService = Depends(get_video_service)`
    *   Logic: Call `await video_service.submit_video(video_in, current_user, background_tasks)`.
5.  **Implement `GET /{video_id}` endpoint for full details (which implicitly includes status):**
    *   Path: `/{video_id}`.
    *   `response_model=VideoResponse` (This will be the full video details, updated by the background task).
    *   Parameters:
        *   `video_id: str`
        *   `video_service: VideoService = Depends(get_video_service)`
    *   Logic:
        *   `video = await video_service.get_video_by_id(video_id)` (This method needs to be created in VideoService).
        *   If not `video`, raise `HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Video not found")`.
        *   Return `video`.
    *   *(Self-correction: The API spec had `/videos/{videoId}/status`. Let's align. The `GET /{videoId}` will be for full details once processing is complete. For now, let's make a specific status endpoint as per spec, though it overlaps with what `GET /{videoId}` would eventually show).*
    **Corrected Endpoint: `GET /{video_id}/status`**
    *   Path: `/{video_id}/status`
    *   `response_model=VideoProcessingStatusResponse`
    *   Parameters:
        *   `video_id: str`
        *   `video_service: VideoService = Depends(get_video_service)`
    *   Logic:
        *   `status_response = await video_service.get_video_processing_status(video_id)`
        *   If not `status_response`, raise `HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Video status not found or video does not exist")`.
        *   Return `status_response`.

**Instructions for `app/api/v1/api_router.py`:**

1.  **Import videos router:** `from app.api.v1.endpoints import videos as videos_router`.
2.  **Include videos router:** `api_router.include_router(videos_router.router, prefix="/videos", tags=["Videos"])`.

**Instructions for `tests/api/v1/test_videos_endpoints.py`:**

1.  **Import `TestClient`, `status`, `settings`, `UserInDB`, `VideoCreate`, `UserRole`, `VideoProcessingStatusEnum`. Mock `VideoService` methods for focused endpoint tests.**
2.  **Helper to get a creator token (similar to user token, but ensure user has 'creator' role for tests that need it).**
    *   This will involve registering a user, then programmatically updating their roles in the (mocked) DB for the test, then logging them in. Or, simpler: mock `get_current_active_creator` to return a pre-defined creator user.
3.  **Write `test_submit_video_success(client: TestClient, creator_token: str)`:**
    *   Use `client.app.dependency_overrides` to mock `VideoService.submit_video` to return a predefined `VideoProcessingStatusResponse`.
    *   Prepare `VideoCreate` data.
    *   POST to `/videos` with token and data.
    *   Assert status 202. Assert response matches mocked return.
4.  **Write `test_submit_video_not_creator(client: TestClient, non_creator_token: str)`:**
    *   Use a token from a user without 'creator' role.
    *   POST to `/videos`. Assert 403.
5.  **Write `test_submit_video_no_token(client: TestClient)`:**
    *   POST to `/videos` without token. Assert 401.
6.  **Write `test_get_video_status_success(client: TestClient)`:**
    *   Use `client.app.dependency_overrides` to mock `VideoService.get_video_processing_status`.
    *   Mock it to return a sample `VideoProcessingStatusResponse` (e.g., status PROCESSING).
    *   GET `/videos/{video_id}/status`. Assert 200. Assert response matches.
7.  **Write `test_get_video_status_not_found(client: TestClient)`:**
    *   Mock `VideoService.get_video_processing_status` to return `None`.
    *   GET `/videos/{video_id}/status`. Assert 404.

```

---
**Prompt 19: Video Service & Endpoint - Get Full Video Details**
```text
Continuing with the KillrVideo 2025 project (Iteration 3):

**Task:** Implement the service method and endpoint to get full details of a video. This will be used once a video is processed or to see its current (potentially partial) state.

**Files to Create/Modify:**
*   `app/services/video_service.py` (Add `get_video_by_id`)
*   `app/api/v1/endpoints/videos.py` (Modify/Add `GET /{video_id}` endpoint)
*   `tests/services/test_video_service.py` (Add tests for `get_video_by_id`)
*   `tests/api/v1/test_videos_endpoints.py` (Add tests for `GET /{video_id}`)
*   `app/services/user_service.py` (Ensure `get_user_by_id` is available if not already added for uploader info).

**Instructions for `app/services/user_service.py` (Verify):**
1. Ensure `async def get_user_by_id(self, user_id: str) -> Optional[UserInDB]:` exists and works with the (mocked or real) user collection. This will be used by `VideoService` to populate uploader details.

**Instructions for `app/services/video_service.py`:**

1.  **Import `UserResponse`, `VideoResponse` from `app.models.video`, `UserService` from `app.services.user_service`.**
2.  **Modify `VideoService.__init__` to accept `UserService`:**
    ```python
    def __init__(
        self,
        video_collection: AstraDBCollection = Depends(get_video_collection),
        user_service: UserService = Depends() # Assuming get_user_service is now general or UserService can be directly depended upon
    ):
        self.video_collection = video_collection
        self.user_service = user_service
    ```
    *   (*Dependency for UserService*: `get_user_service` was defined in `auth.py`. It might be better to make `UserService` directly injectable via `Depends()` if its dependencies (`user_collection`) are also injectable, or move `get_user_service` to `dependencies.py`.) Let's assume `UserService = Depends()` works.
3.  **Implement `async def get_video_by_id(self, video_id: str) -> Optional[VideoResponse]:`**
    *   Fetch raw video document: `video_doc = self.video_collection.find_one({"video_id": video_id})`.
    *   If not `video_doc`, return `None`.
    *   Populate uploader info:
        *   `uploader_info: Optional[UserResponse] = None`
        *   `uploader_id = video_doc.get("uploader_id")`
        *   If `uploader_id`, then `uploader_db_obj = await self.user_service.get_user_by_id(uploader_id)`.
        *   If `uploader_db_obj`, then `uploader_info = UserResponse.model_validate(uploader_db_obj)`.
    *   Instantiate `VideoInDBBase` from `video_doc` to handle defaults and structure: `video_base_obj = VideoInDBBase(**video_doc)`
    *   Create `VideoResponse` by passing fields from `video_base_obj` and the `uploader_info`.
        ```python
        # Simplified instantiation, Pydantic handles mapping if fields match
        video_data_for_response = video_base_obj.model_dump()
        video_data_for_response['uploader'] = uploader_info
        return VideoResponse(**video_data_for_response)
        # Or if VideoResponse directly takes VideoInDBBase and uploader:
        # return VideoResponse.model_construct(**video_base_obj.model_dump(), uploader=uploader_info)
        # Using model_validate is safer:
        # temp_video_obj = VideoInDBBase(**video_doc) # Ensures VideoInDBBase fields are correctly typed/defaulted
        # response_data = temp_video_obj.model_dump()
        # response_data["uploader"] = uploader_info
        # return VideoResponse.model_validate(response_data)
        ```

**Instructions for `app/api/v1/endpoints/videos.py` (Modify/Ensure `GET /{video_id}`):**

1.  **Ensure the `GET /{video_id}` endpoint (from Prompt 18, which was corrected to be `/status`) is now defined as the full detail endpoint.**
    *   Path: `/{video_id}`.
    *   `response_model=VideoResponse`.
    *   Parameters: `video_id: str`, `video_service: VideoService = Depends(get_video_service)`.
    *   Logic:
        *   `video = await video_service.get_video_by_id(video_id)`.
        *   If not `video`, raise `HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Video not found")`.
        *   Return `video`.

**Instructions for `tests/services/test_video_service.py`:**

1.  **Import `UserResponse`, `VideoResponse`, `UserService`, `AsyncMock`.**
2.  **Update `video_service` fixture to mock `UserService` dependency:**
    ```python
    @pytest.fixture
    def mock_user_service():
        service = MagicMock(spec=UserService)
        # Configure default return for get_user_by_id if needed across tests
        # For example, to simulate uploader found:
        # uploader_data = UserResponse(user_id="uploader123", email="up@ex.com", roles=[UserRole.CREATOR])
        # service.get_user_by_id = AsyncMock(return_value=UserInDB(**uploader_data.model_dump()))
        service.get_user_by_id = AsyncMock(return_value=None) # Default: uploader not found or not mocked yet
        return service

    @pytest.fixture
    def video_service(mock_video_collection: MagicMock, mock_user_service: MagicMock):
        # Now VideoService takes user_service as well
        return VideoService(video_collection=mock_video_collection, user_service=mock_user_service)
    ```
3.  **Write `test_get_video_by_id_found(video_service: VideoService, mock_video_collection: MagicMock, mock_user_service: MagicMock)`:**
    *   Prepare sample raw video document data (as dict).
    *   Mock `mock_video_collection.find_one.return_value = sample_video_doc_data`.
    *   Prepare sample uploader data (`UserInDB` instance or dict for `UserResponse`).
    *   Mock `mock_user_service.get_user_by_id.return_value = sample_uploader_db_obj`.
    *   Call `video = await video_service.get_video_by_id("some_video_id")`.
    *   Assert `video` is `VideoResponse`.
    *   Assert video details match `sample_video_doc_data`.
    *   Assert `video.uploader` details match `sample_uploader_data`.
4.  **Write `test_get_video_by_id_video_not_found(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   `mock_video_collection.find_one.return_value = None`.
    *   Call `await video_service.get_video_by_id()`. Assert `None`.
5.  **Write `test_get_video_by_id_uploader_not_found(video_service: VideoService, mock_video_collection: MagicMock, mock_user_service: MagicMock)`:**
    *   Mock `mock_video_collection.find_one` to return video data.
    *   Mock `mock_user_service.get_user_by_id.return_value = None`.
    *   Call service method. Assert `video.uploader` is `None`.

**Instructions for `tests/api/v1/test_videos_endpoints.py`:**

1.  **Write `test_get_video_details_success(client: TestClient)`:**
    *   Use `client.app.dependency_overrides` to mock `VideoService.get_video_by_id`.
    *   Mock it to return a sample `VideoResponse` object.
    *   GET `/videos/{some_video_id}`. Assert 200. Assert response matches.
2.  **Write `test_get_video_details_not_found(client: TestClient)`:**
    *   Mock `VideoService.get_video_by_id` to return `None`.
    *   GET `/videos/{some_video_id}`. Assert 404.
```

---
**Prompt 20: Video Service & Endpoint - List Latest Videos**
```text
Continuing with the KillrVideo 2025 project (Iteration 3):

**Task:** Implement listing the latest videos with pagination.

**Files to Create/Modify:**
*   `app/services/video_service.py` (Add `list_latest_videos`)
*   `app/api/v1/endpoints/videos.py` (Add `GET /latest` endpoint)
*   `tests/services/test_video_service.py` (Add tests for `list_latest_videos`)
*   `tests/api/v1/test_videos_endpoints.py` (Add tests for `GET /latest`)
*   `app/models/common.py` (Ensure `PaginatedResponse` exists or create it)

**Instructions for `app/models/common.py` (Ensure/Create):**

1.  **Import `List`, `Generic`, `TypeVar` from `typing`, `BaseModel`, `Field` from `pydantic`.**
2.  **Define `T = TypeVar('T')`.**
3.  **Define `class PaginationInfo(BaseModel):`**
    *   `current_page: int`
    *   `page_size: int`
    *   `total_items: int`
    *   `total_pages: int`
4.  **Define `class PaginatedResponse(BaseModel, Generic[T]):`**
    *   `data: List[T]`
    *   `pagination: PaginationInfo`

**Instructions for `app/services/video_service.py`:**

1.  **Import `VideoSummary`, `PaginatedResponse`, `PaginationInfo` from `app.models`.**
2.  **Implement `async def list_latest_videos(self, page: int = 1, page_size: int = 10) -> PaginatedResponse[VideoSummary]:`**
    *   Calculate `skip = (page - 1) * page_size`.
    *   Fetch videos from DB: `cursor = self.video_collection.find_many(sort={"submitted_at": -1}, projection={"_id": 0}, skip=skip, limit=page_size)`.
        *   AstraPy `find_many` might be just `find`. Ensure `sort`, `skip`, `limit` are used correctly. Projection helps select fields.
        *   AstraPy's `find()` returns a generator/iterable of dicts.
    *   Fetch total count: `total_items = self.video_collection.count_documents(filter={})`. (Ensure `count_documents` is the correct AstraPy method).
    *   Process results:
        ```python
        videos_data = list(cursor.get("data", [])) # AstraPy find() often returns {"data": [...]}
        video_summaries = []
        for video_doc in videos_data:
            # For VideoSummary, we need to carefully construct it.
            # It needs uploader_name, thumbnail_url, effective_title, duration_seconds.
            # This might require fetching uploader or having these denormalized/simplified.
            # For now, let's simplify and assume some fields are directly available or can be derived.
            
            # Simplified uploader_name for now, full UserResponse population is heavy for lists
            uploader_name_str = "Unknown Uploader" # Placeholder
            uploader_id = video_doc.get("uploader_id")
            if uploader_id:
                 # In a real scenario, you might do a batch fetch for uploader names
                 # or have uploader_name denormalized on video record.
                 # For this step, let's assume we can fetch it or use a placeholder.
                uploader = await self.user_service.get_user_by_id(uploader_id) # This can be N+1, optimize later
                if uploader:
                    uploader_name_str = f"{uploader.first_name or ''} {uploader.last_name or ''}".strip() or uploader.email

            summary = VideoSummary(
                video_id=video_doc["video_id"],
                effective_title=video_doc.get("effective_title", video_doc.get("title")),
                thumbnail_url=video_doc.get("processing_details", {}).get("thumbnail_url"), # Path to thumbnail
                uploader_name=uploader_name_str,
                view_count=video_doc.get("view_count", 0),
                submitted_at=video_doc["submitted_at"],
                duration_seconds=video_doc.get("processing_details", {}).get("duration_seconds"),
                status=video_doc.get("status", VideoProcessingStatusEnum.PROCESSING)
            )
            video_summaries.append(summary)
        
        total_pages = (total_items + page_size - 1) // page_size
        pagination_info = PaginationInfo(
            current_page=page,
            page_size=page_size,
            total_items=total_items,
            total_pages=total_pages
        )
        return PaginatedResponse[VideoSummary](data=video_summaries, pagination=pagination_info)
        ```

**Instructions for `app/api/v1/endpoints/videos.py`:**

1.  **Import `PaginatedResponse`, `VideoSummary` from `app.models`.**
2.  **Implement `GET /latest` endpoint:**
    *   `response_model=PaginatedResponse[VideoSummary]`.
    *   Parameters: `page: int = Query(1, ge=1)`, `page_size: int = Query(10, ge=1, le=100)`, `video_service: VideoService = Depends(get_video_service)`.
    *   Logic: `return await video_service.list_latest_videos(page=page, page_size=page_size)`.

**Instructions for `tests/services/test_video_service.py`:**

1.  **Write `test_list_latest_videos(video_service: VideoService, mock_video_collection: MagicMock, mock_user_service: MagicMock)`:**
    *   Prepare a list of sample video document dicts.
    *   Mock `mock_video_collection.find.return_value = {"data": sample_video_docs}` (or however AstraPy `find` returns multiple docs).
    *   Mock `mock_video_collection.count_documents.return_value = len(sample_video_docs)`.
    *   Mock `mock_user_service.get_user_by_id` to return mock uploader data when called.
    *   Call `await video_service.list_latest_videos()`.
    *   Assert the response is `PaginatedResponse[VideoSummary]`.
    *   Assert `data` contains correct number of `VideoSummary` items.
    *   Assert `pagination` info is correct.
    *   Verify that `find` was called with correct sort, skip, limit.

**Instructions for `tests/api/v1/test_videos_endpoints.py`:**

1.  **Write `test_list_latest_videos_success(client: TestClient)`:**
    *   Use `client.app.dependency_overrides` to mock `VideoService.list_latest_videos`.
    *   Mock it to return a sample `PaginatedResponse[VideoSummary]`.
    *   GET `/videos/latest` (with optional page/page\_size query params).
    *   Assert 200. Assert response matches.
2.  **Write `test_list_latest_videos_pagination_params(client: TestClient)`:**
    *   Mock `VideoService.list_latest_videos` (perhaps using `unittest.mock.patch` on the service instance method if overrides are tricky for specific args).
    *   Call endpoint with `?page=2&page_size=5`.
    *   Verify that the mocked service method was called with `page=2, page_size=5`.

```

---
**Prompt 21: Video Playback Statistics - Record View**```text
Continuing with the KillrVideo 2025 project (Iteration 3):

**Task:** Implement the service method and endpoint to record a video view, incrementing its view count.

**Files to Create/Modify:**
*   `app/services/video_service.py` (Add `record_video_view`)
*   `app/api/v1/endpoints/videos.py` (Add `POST /{video_id}/views` endpoint)
*   `tests/services/test_video_service.py` (Add tests for `record_video_view`)
*   `tests/api/v1/test_videos_endpoints.py` (Add tests for `POST /{video_id}/views`)

**Instructions for `app/services/video_service.py`:**

1.  **Implement `async def record_video_view(self, video_id: str) -> bool:`**
    *   Use AstraDB's atomic increment if available through AstraPy, or a find-and-update approach.
    *   AstraPy Data API `update_one` supports `$inc`:
        ```python
        result = self.video_collection.update_one(
            filter={"video_id": video_id},
            update={"$inc": {"view_count": 1}}
        )
        if result.modified_count == 0 and result.matched_count == 0: # Check if video was found
            # Could also mean view_count field didn't exist and wasn't created by $inc,
            # depending on DB behavior. Ensure view_count is initialized to 0.
            return False # Video not found
        return True # View incremented or video found (even if view_count wasn't modified due to some reason but matched)
        ```
    *   Return `True` if successful (video found and increment attempted), `False` if video not found.

**Instructions for `app/api/v1/endpoints/videos.py`:**

1.  **Implement `POST /{video_id}/views` endpoint:**
    *   `status_code=status.HTTP_204_NO_CONTENT`.
    *   Parameters: `video_id: str`, `video_service: VideoService = Depends(get_video_service)`.
    *   Logic:
        *   `success = await video_service.record_video_view(video_id)`.
        *   If not `success`, raise `HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Video not found to record view.")`.
        *   Return `Response(status_code=status.HTTP_204_NO_CONTENT)` (FastAPI handles this if no body is returned from a 204 endpoint).

**Instructions for `tests/services/test_video_service.py`:**

1.  **Write `test_record_video_view_success(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   Mock `mock_video_collection.update_one.return_value = MagicMock(modified_count=1, matched_count=1)`.
    *   Call `success = await video_service.record_video_view("some_video_id")`.
    *   Assert `success` is `True`.
    *   Verify `update_one` was called with `{"video_id": "some_video_id"}` and `{"$inc": {"view_count": 1}}`.
2.  **Write `test_record_video_view_video_not_found(video_service: VideoService, mock_video_collection: MagicMock)`:**
    *   Mock `mock_video_collection.update_one.return_value = MagicMock(modified_count=0, matched_count=0)`.
    *   Call `success = await video_service.record_video_view("non_existent_video_id")`.
    *   Assert `success` is `False`.

**Instructions for `tests/api/v1/test_videos_endpoints.py`:**

1.  **Write `test_record_video_view_success(client: TestClient)`:**
    *   Use `client.app.dependency_overrides` to mock `VideoService.record_video_view` to return `True`.
    *   POST to `/videos/{some_video_id}/views`. Assert status 204.
2.  **Write `test_record_video_view_not_found(client: TestClient)`:**
    *   Mock `VideoService.record_video_view` to return `False`.
    *   POST to `/videos/{non_existent_video_id}/views`. Assert status 404.

**Final Testing for Iteration 3:**
Advise the user to:
1. Run the application.
2. Use `/api/v1/docs` to:
    a. (Requires Creator Token) Submit a video. Note the `video_id`.
    b. Check its status using `GET /videos/{video_id}/status`. (It will be PROCESSING).
    c. After a short delay (for the mock background task), check status again. It might be COMPLETED or FAILED based on the placeholder.
    d. Get full video details using `GET /videos/{video_id}`.
    e. Record a view using `POST /videos/{video_id}/views`.
    f. Get full video details again and check if `view_count` (if visible in `VideoResponse` directly or through `processing_details`) has changed (mock DB won't persist this unless `record_video_view` actually fetches and re-saves the view_count in its mock). For this step, testing the service logic is key.
    g. List latest videos using `GET /videos/latest`.
3. Re-run all tests: `poetry run pytest`.
```

This completes the prompts for Iteration 3. We now have basic video submission, status checking, detail retrieval, listing, and view recording. The asynchronous processing is still a placeholder, which will be a focus for a subsequent iteration where actual YouTube integration and AI service calls are made.