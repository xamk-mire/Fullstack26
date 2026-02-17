# Assignment: Iteration 2 — FastAPI Backend and Frontend Integration

**Project:** Exercise/Workout Tracker  
**Iteration:** 2 of 4  
**Focus:** Implement a Python/FastAPI backend with layered architecture and connect the existing frontend to it.

**Prerequisite:** Completed [ASSIGNMENT_ITERATION_1.md](Assignments/Assignment-2/ASSIGNMENT_ITERATION_1.md) (Basic Frontend). You will use the existing frontend (e.g. the example solution in `frontend/`) and add a new `backend/` folder in the same project.

**References:** Course materials **08 — Backend Introduction** (layered architecture, request–response, API as contract) and **09 — Python** (Python basics, FastAPI, JSON). This assignment follows the same structure as **Exercise-3** (Task Tracker backend and frontend integration).

---

## Introduction

In Iteration 1 you built a React frontend that displays exercises and categories using **mock data** stored in the frontend (e.g. in `frontend/src/data/`). In this iteration you will:

1. **Build a Python/FastAPI backend** that exposes a REST API for categories and exercises. The API will return the same data shapes (DTOs) that your frontend already uses: a list of categories, a list of exercises (each with nested categories), and a single exercise detail (with categories and fields). By matching the frontend’s types exactly, you avoid changing the UI code when you switch from mocks to the API.
2. **Use a layered architecture** (as in Materials 08): **Routes (API layer)** → **Service layer** → **Repository (data access layer)**. The repository holds data in memory for now (no database); you can replace it with PostgreSQL or another store in a later iteration without changing the service or route code.
3. **Connect the frontend** to the backend by adding an environment variable for the API base URL, a small config module, a dedicated API client (`exercisesApi.ts`) that performs `fetch` calls, and an **api service** (`apiService.ts`) that delegates to the API client. In this iteration you **rename** the existing data service file from `dataService.ts` to `apiService.ts`. The pages (Exercise list, Exercise detail) keep calling the same functions (`getCategories`, `getExercises`, `getExerciseById`) from the api service; only the implementation changes from “return mock data” to “call the API client and return the response”.

By the end, the Exercise Tracker will be **full-stack**: the browser will load exercises and categories from your backend over HTTP, and the backend will hold the data in memory. The frontend will look and behave as before, but the data will come from the API instead of mock arrays.

---

## Overall goal

1. **Set up a FastAPI backend** with **layered architecture**:
   - **Routes (API layer)** — Receive HTTP requests, call services, return JSON and status codes. Use a **route file per resource** (e.g. `routes/categories.py`, `routes/exercises.py`). `main.py` creates the app, adds CORS, and includes the routers.
   - **Service layer** — Use cases (list categories, get exercise by id, etc.). Calls the repository; no HTTP or FastAPI imports.
   - **Repository (data access layer)** — In-memory storage in `repositories/store.py` with the same structure as your Iteration 1 mock data. Exposes functions to get categories, exercises, exercise–category links, and exercise fields. Later you can replace this with a database without changing the service or routes.

2. **API contract** — The backend must expose endpoints that match what the frontend api service (and API client) expect:
   - `GET /api/categories` → list of categories (same shape as `CategoryDto`).
   - `GET /api/categories/{id}` → one category or 404.
   - `GET /api/exercises` → list of exercises with nested `categories` (same shape as `ExerciseListItemDto`).
   - `GET /api/exercises/{id}` → one exercise with `categories` and `fields` (same shape as `ExerciseDetailDto`) or 404.

   All ids are **integers**. Use the same field names as the frontend (e.g. `id`, `name`, `date`, `notes`, `categories`, `fields`).

3. **Frontend integration** — Add an environment variable for the backend URL, an API config module (`api/config.ts`), a dedicated API client (`api/exercisesApi.ts`) that performs `fetch` calls, and **rename** `dataService.ts` to `apiService.ts` so the api service delegates to the client. The existing pages (ExerciseListPage, ExerciseDetailPage) keep importing from the api service and using `getExercises()`, `getCategories()`, and `getExerciseById()`; only the implementation of those functions changes.

---

## What you need before starting

- **Iteration 1 frontend** (e.g. the example in `frontend/`). It uses:
  - **Data service** (`frontend/src/services/dataService.ts`): `getCategories()`, `getCategoryById(id)`, `getExercises()`, `getExerciseById(id)`. In this iteration you will **rename** this file to `apiService.ts` and reimplement these four functions so they call the backend (via a dedicated API client) instead of returning mock data. The example solution uses `apiService.ts` and `api/exercisesApi.ts`.
  - **Types** (`frontend/src/types/index.ts`): `CategoryDto`, `ExerciseListItemDto`, `ExerciseDetailDto`, `ExerciseFieldDto`. Your backend response shapes must match these exactly (same field names and types, integer ids) so the frontend can consume the API without changes. Keep that file open when defining your backend DTOs and repository data.
- **Python 3.10+** installed. Check with `python --version` or `python3 --version` from a terminal.
- Course materials **08 — Backend Introduction** (layered architecture, API as contract) and **09 — Python** (FastAPI, JSON). Optionally **Exercise-3** for a similar step-by-step pattern with a Task Tracker.

---

## Backend folder structure (target)

When you are done, the backend should look like this. You do **not** need `__init__.py` files in any of these folders; Python 3.3+ will treat the directories as namespace packages.

```
LearningProject/
  backend/
    main.py                         # FastAPI app, CORS, include routers; root route
    models/
      dtoModels.py                  # Pydantic DTOs for API responses
    repositories/
      store.py                      # In-memory data + get_all_categories, get_exercise_*, etc.
    routes/
      categories.py                 # GET /api/categories, GET /api/categories/{id}
      exercises.py                  # GET /api/exercises, GET /api/exercises/{id}
    services/
      category_service.py          # list_categories, get_category_by_id
      exercise_service.py          # list_exercises, get_exercise_by_id
    venv/                           # (optional) virtual environment
  frontend/                         # existing; you will add api/, rename dataService.ts → apiService.ts
```

Request flow: **Route → Service → Repository (store)**. Response flows back up. Routes must not import the store directly; they call services. Services must not import FastAPI or handle HTTP; they call the repository and build DTOs.

---

# Step 1 — Create the FastAPI project (backend folder and dependencies)

## Goal

Create a `backend` folder in your project root, install FastAPI and uvicorn, and run a minimal app that returns JSON. This gives you a working server and confirms your environment is set up correctly before you add layers.

## 1.1 Create the backend folder

In your **LearningProject** root (same level as `frontend/`), create a folder named `backend`. All Python backend code will live under `backend/`. From now on, whenever the assignment says “in the backend” or “under backend”, it means inside this folder. **Important:** All commands that run the app (e.g. uvicorn) must be run from the **backend** directory so that Python can find your modules (`main`, `models`, `routes`, etc.).

## 1.2 Virtual environment (recommended but optional)

Using a virtual environment keeps backend dependencies separate from other Python projects and avoids version conflicts.

1. Open a terminal and change into the backend folder (e.g. `cd backend`).
2. Create a virtual environment with the standard library’s `venv` module (e.g. `python -m venv venv`). This creates a `venv` folder inside `backend/`.
3. Activate the environment:
   - **Windows (PowerShell):** run the script under `venv\Scripts` (e.g. `.\venv\Scripts\Activate.ps1`). If execution is restricted, you may need to allow scripts for the current user.
   - **Windows (cmd):** run the `.bat` under `venv\Scripts` (e.g. `venv\Scripts\activate.bat`).
   - **macOS/Linux:** run `source venv/bin/activate`.
4. Confirm activation: your shell prompt should show something like `(venv)` at the start, or `which python` (macOS/Linux) / `where python` (Windows) should point inside the `venv` folder. You need to activate the venv in every new terminal where you run the backend.

## 1.3 Install dependencies

With the virtual environment activated and your current directory set to the backend folder:

1. Install **FastAPI** and **uvicorn**. FastAPI provides the web framework (routes, request/response, validation); uvicorn is the ASGI server that runs the FastAPI app.
2. You can install with plain pip (e.g. `pip install fastapi uvicorn`) or use a `requirements.txt` for repeatable installs. If you use `requirements.txt`, create it in the backend folder with at least two lines (e.g. `fastapi>=0.109.0` and `uvicorn[standard]>=0.27.0`), then run `pip install -r requirements.txt`. The `[standard]` extra for uvicorn adds useful dependencies for development.
3. Confirm installation: from the backend directory, `python -c "import fastapi; import uvicorn; print('OK')"` should run without errors.

## 1.4 Minimal app

1. Create a file named `main.py` in the backend folder (same level as `venv/`).
2. In `main.py`, create a FastAPI application instance. Pass a `title` argument (e.g. `"Exercise Tracker API"`) so the auto-generated docs show a clear name.
3. Add a single route for the root path `"/"`. Use the decorator that binds a GET request to a function. The function should return a JSON-serializable value (e.g. a dict with a key like `"message"` and a value like `"Exercise Tracker API is running"`). FastAPI will automatically convert the return value to JSON and set the appropriate response headers.
4. Run the app: from the **backend** directory, run uvicorn with the module and app name (e.g. `uvicorn main:app --reload`). The `--reload` flag restarts the server when you change code. Leave this terminal open.
5. In a browser, open the root URL (e.g. http://127.0.0.1:8000). You should see the JSON object you returned. Then open http://127.0.0.1:8000/docs to see the automatic Swagger UI; at this stage it will show only the root route.

**Tip:** If you get “ModuleNotFoundError” when running uvicorn, ensure you are in the backend directory and that `main.py` exists there. If you use a different app variable name in `main.py`, use that name after the colon (e.g. `uvicorn main:my_app --reload`).

## When to move on

- You have a `backend/` folder containing at least `main.py` and (optionally) `venv/` and `requirements.txt`.
- With the venv activated and the current directory set to `backend/`, the command to run uvicorn succeeds and the root URL returns JSON in the browser.

---

# Step 2 — DTO models and repository (data access layer)

## Goal

Define **Pydantic DTO models** that match your frontend’s response shapes so the API can return JSON the frontend already understands. Implement the **repository** in `repositories/store.py`: in-memory lists for categories, exercises, exercise–category links, and exercise fields, with functions to read them. Seed the store with the same data as your frontend mock data so that when you later wire the API, the list and detail pages show the same content as before. The repository does not know about DTOs or HTTP; it only stores and returns raw data. The service layer (Step 3) will use this data to build the DTOs.

## 2.1 DTO models

1. **Create the models package:** Under `backend/`, create a folder named `models`. Inside it, create a file named `dtoModels.py`.

2. **Match the frontend types:** Open your frontend’s type definitions (e.g. `frontend/src/types/index.ts`) and use them as the source of truth. Your frontend uses **integer ids** and these shapes:
   - **CategoryDto:** `id` (int), `name` (str), `description` (str).
   - **ExerciseFieldDto:** `id` (int), `name` (str), `value` (str), `unit` (str). Note: the frontend DTO does not include `exerciseId`; the API response for fields should only expose id, name, value, unit.
   - **ExerciseListItemDto:** `id`, `name`, `date`, `notes` (same types as in the frontend), and `categories` (a list of CategoryDto).
   - **ExerciseDetailDto:** same fields as the list item, plus `fields` (a list of ExerciseFieldDto).

3. **Define Pydantic models:** In `dtoModels.py`, import `BaseModel` from `pydantic` and define one class per DTO above. Each class must have the exact same attribute names as the frontend (e.g. `id`, `name`, `description` for CategoryDto). Use Python type hints: e.g. `list[CategoryDto]` for the `categories` field, and define `CategoryDto` before the classes that reference it so that the nested types resolve. Pydantic will serialize these to JSON with the same keys the frontend expects. Do not add extra fields the frontend does not use.

4. **Scope of this file:** This file defines only the **response** shapes (DTOs). The repository will work with plain dicts or simple records; the service layer will convert repository data into these DTOs. You do not need separate “domain” models in this file unless you want to share structures between store and services—the assignment expects the store to use dicts/lists and the service to build DTOs from them.

## 2.2 Repository (store)

1. **Create the repositories package:** Under `backend/`, create a folder named `repositories`. Inside it, create a file named `store.py`. This file is the **data access layer**: it is the only place that “holds” the data. Later you can replace this file (or add a different implementation) with a database without changing the service or route code.

2. **In-memory data structures:** Maintain four in-memory lists (or lists of dicts) inside `store.py`:
   - **Categories:** each item has `id` (int), `name` (str), `description` (str). Use the same number of categories and the same ids/names/descriptions as in your frontend mock categories (e.g. `frontend/src/data/mockCategories.ts`).
   - **Exercises:** each item has `id`, `name`, `notes`, `date` (same types and values as in your frontend mock exercises).
   - **Exercise–category links:** each item has `categoryId` and `exerciseId` (ints), representing which categories are assigned to which exercises. Copy the same pairs from your frontend mock (e.g. `mockExerciseCategories.ts`).
   - **Exercise fields:** each item has `id`, `exerciseId`, `name`, `value`, `unit`. Copy from your frontend mock fields (e.g. `mockExerciseFields.ts`) so that exercise detail responses match what the UI already displays.

3. **Repository functions:** Expose the following functions. They should take and return the types described; use `Optional` or `None` for “not found” where appropriate.
   - **get_all_categories()** — returns a list of all category records (each record is a dict or object with id, name, description). Return a copy or new list so callers cannot mutate the internal store.
   - **get_category_by_id(category_id: int)** — returns the single category with that id, or `None` if not found.
   - **get_all_exercises()** — returns a list of all exercise records (id, name, notes, date).
   - **get_exercise_by_id(exercise_id: int)** — returns the single exercise with that id, or `None` if not found.
   - **get_exercise_category_ids(exercise_id: int)** — returns a list of category ids (ints) that are linked to the given exercise id, by scanning the exercise–category links. Return an empty list if the exercise has no categories.
   - **get_exercise_fields(exercise_id: int)** — returns a list of field records for that exercise. Each record should at least have `id`, `name`, `value`, `unit` so the service can build ExerciseFieldDto. You may omit `exerciseId` from the returned records if the service does not need it.

4. **No HTTP or FastAPI:** The store must not import FastAPI or perform any HTTP operations. It only reads and returns data from the in-memory lists. The **service layer** will call these functions and assemble the nested DTOs (e.g. for each exercise, fetch category ids, then fetch each category, and build the list of CategoryDto).

## When to move on

- `models/dtoModels.py` exists and defines the four DTO classes with the exact field names and types listed above. You can quickly check by importing them in a small script or in the next step when the service uses them.
- `repositories/store.py` exists, contains the four data lists seeded with the same data as your frontend mocks, and implements all six get\_\* functions. The app still runs with `uvicorn main:app --reload` from the backend directory (the new modules are not yet used by `main.py`, but they should be importable without errors).

---

# Step 3 — Service layer

## Goal

Add a **service layer** between the API (routes) and the repository. The service implements the **use cases**: list categories, get category by id, list exercises (with categories), get exercise by id (with categories and fields). It calls only the repository and the DTO models; it must not import FastAPI or perform HTTP. The routes (Step 4) will call these service functions and return their results as JSON. Keeping HTTP and “business” logic separate makes the API easier to test and change (e.g. swap the repository for a database later).

## 3.1 Category service

1. **Create the services package:** Under `backend/`, create a folder named `services`. Inside it, create a file named `category_service.py`.

2. **Imports:** At the top of the file, import the repository (e.g. `from repositories import store`) and the category DTO (e.g. `from models.dtoModels import CategoryDto`). Do not import FastAPI, HTTPException, or anything from `fastapi`.

3. **list_categories:** Implement a function (e.g. `list_categories()`) that takes no arguments and returns a list of `CategoryDto`. Inside the function, call the repository function that returns all category records (e.g. `store.get_all_categories()`). Iterate over the result and convert each record into a `CategoryDto`. You can pass the record as keyword arguments to the Pydantic model if the record’s keys match the model’s field names. Return the list of DTOs.

4. **get_category_by_id:** Implement a function (e.g. `get_category_by_id(category_id: int)`) that returns a single `CategoryDto` or `None`. Call the repository function that returns one category by id (e.g. `store.get_category_by_id(category_id)`). If the result is `None`, return `None`. Otherwise construct a `CategoryDto` from the record and return it.

**Check:** The category service should have no dependency on FastAPI or on the routes. It only depends on the repository and on `models.dtoModels`.

## 3.2 Exercise service

1. **Create the file:** In the same `services` folder, create `exercise_service.py`.

2. **Imports:** Import the repository (e.g. `from repositories import store`) and all DTOs you need from `models.dtoModels`: `CategoryDto`, `ExerciseFieldDto`, `ExerciseListItemDto`, `ExerciseDetailDto`. Again, no FastAPI imports.

3. **List exercises:** Implement a function (e.g. `list_exercises()`) that returns a list of `ExerciseListItemDto`. Logic:
   - Call the repository to get all exercises (e.g. `store.get_all_exercises()`).
   - For each exercise record, you need to attach its categories. Call the repository function that returns the list of category ids for this exercise (e.g. `store.get_exercise_category_ids(exercise["id"])`). For each category id, call the repository to get the full category (e.g. `store.get_category_by_id(cid)`); if it returns a record, convert it to `CategoryDto` and add it to a list. Build one `ExerciseListItemDto` per exercise with: `id`, `name`, `date`, `notes` from the exercise record, and `categories` as the list of `CategoryDto` you just built. Append each DTO to the result list and return it.

4. **Get exercise by id:** Implement a function (e.g. `get_exercise_by_id(exercise_id: int)`) that returns an `ExerciseDetailDto` or `None`. Logic:
   - Call the repository to get the exercise by id (e.g. `store.get_exercise_by_id(exercise_id)`). If the result is `None`, return `None`.
   - Get the category ids for this exercise from the repository, then for each id fetch the full category and convert to `CategoryDto`, building a list of categories.
   - Get the exercise fields from the repository (e.g. `store.get_exercise_fields(exercise_id)`). Convert each field record to `ExerciseFieldDto` (use only the fields the DTO needs: id, name, value, unit).
   - Construct and return an `ExerciseDetailDto` with the exercise’s id, name, date, notes, the list of CategoryDto, and the list of ExerciseFieldDto.

**Check:** The exercise service does not know about HTTP or status codes. It only returns DTOs or `None`; the route layer will turn `None` into a 404 response.

## When to move on

- `services/category_service.py` and `services/exercise_service.py` exist and expose the functions described above (e.g. `list_categories`, `get_category_by_id`, `list_exercises`, `get_exercise_by_id`).
- Both services import only the repository and `models.dtoModels`; they do not import FastAPI or handle HTTP. You can run the app with uvicorn; the new service modules are not yet used by `main.py` but should import without errors.

---

# Step 4 — REST API endpoints (routes)

## Goal

Create **route modules** that handle HTTP: they receive requests, call the appropriate service function, and return JSON responses with the correct status codes. Register the routers in `main.py` under the prefix `/api` so the full paths are `/api/categories`, `/api/categories/{id}`, `/api/exercises`, and `/api/exercises/{id}`. Enable **CORS** so the React app running on a different origin (e.g. http://localhost:5173) can call the API from the browser without being blocked.

## 4.1 Categories router

1. **Create the routes package:** Under `backend/`, create a folder named `routes`. Inside it, create a file named `categories.py`.

2. **Imports:** Import from FastAPI: `APIRouter` and `HTTPException`. Import `CategoryDto` from `models.dtoModels`. Import the category service module from the services package (e.g. `from services import category_service`). Do **not** import the repository (store) here; the route only talks to the service.

3. **Create the router:** Instantiate an `APIRouter` with `prefix="/categories"` and `tags=["categories"]`. The prefix will be combined with the `/api` prefix you add in `main.py`, so the final path for this router will be `/api/categories` and `/api/categories/{id}`.

4. **GET list of categories:** Add a GET route for the empty path (i.e. the path after the prefix is empty, so the full path is `/api/categories`). The route handler should call the category service function that returns the list of categories (e.g. `category_service.list_categories()`), then return that list. Declare the response model (e.g. `response_model=list[CategoryDto]`) so FastAPI validates and serializes the response to JSON. FastAPI will return status 200 by default for a successful response.

5. **GET category by id:** Add a GET route for a path parameter `{id}`. Declare the parameter as an integer (e.g. `id: int`) so FastAPI parses and validates it. In the handler, call the category service function that returns one category by id (e.g. `category_service.get_category_by_id(id)`). If the result is `None`, raise `HTTPException(status_code=404, detail="...")` with a short message (e.g. "Category not found"). Otherwise return the CategoryDto (200). Set `response_model=CategoryDto` for this route.

## 4.2 Exercises router

1. **Create the file:** In the same `routes` folder, create `exercises.py`.

2. **Imports:** Import from FastAPI: `APIRouter` and `HTTPException`. Import `ExerciseListItemDto` and `ExerciseDetailDto` from `models.dtoModels`. Import the exercise service module (e.g. `from services import exercise_service`). Do not import the store.

3. **Create the router:** Instantiate an `APIRouter` with `prefix="/exercises"` and `tags=["exercises"]`. Combined with the `/api` prefix in `main.py`, the paths will be `/api/exercises` and `/api/exercises/{id}`.

4. **GET list of exercises:** Add a GET route for the empty path. Call the exercise service function that returns the list of exercises with nested categories (e.g. `exercise_service.list_exercises()`), and return that list. Use `response_model=list[ExerciseListItemDto]`.

5. **GET exercise by id:** Add a GET route for the path parameter `{id}` (integer). Call the exercise service function that returns one exercise by id (e.g. `exercise_service.get_exercise_by_id(id)`). If the result is `None`, raise `HTTPException(status_code=404, detail="...")` (e.g. "Exercise not found"). Otherwise return the ExerciseDetailDto. Use `response_model=ExerciseDetailDto`.

## 4.3 Update main.py

1. **Import the routers:** At the top of `main.py`, import the router objects from the route modules. The exact attribute name depends on how you defined them in `categories.py` and `exercises.py` (e.g. if you assigned the APIRouter to a variable named `router`, import `routes.categories.router` and `routes.exercises.router`, or use aliases like `categories_router` and `exercises_router`).

2. **Add CORS middleware:** Before including the routers, add FastAPI’s CORS middleware to the app. Configure it to allow requests from the frontend origin(s). At minimum, add `http://localhost:5173` and `http://127.0.0.1:5173` to the list of allowed origins (Vite’s dev server may use either). For development, you can allow all methods and all headers; for production you would restrict these. Attach the middleware to the app so that browser requests from the frontend receive the appropriate CORS headers in the response.

3. **Include the routers:** Register the categories router with the app using `app.include_router(..., prefix="/api")`. Register the exercises router the same way with `prefix="/api"`. The router’s own prefix (`/categories` or `/exercises`) is appended, so the full paths are `/api/categories`, `/api/categories/{id}`, `/api/exercises`, and `/api/exercises/{id}`. Keep your existing root route (`/`) if you want a simple “API is running” message.

4. **Run and test:** Restart uvicorn from the backend directory. Open the docs at http://127.0.0.1:8000/docs. You should see four endpoints under the tags “categories” and “exercises”. Try each one and confirm the response JSON matches the frontend types (integer ids, same field names).

## API contract summary (for frontend)

| Method | Path                 | Response / status              |
| ------ | -------------------- | ------------------------------ |
| GET    | /api/categories      | 200, `CategoryDto[]`           |
| GET    | /api/categories/{id} | 200 `CategoryDto` or 404       |
| GET    | /api/exercises       | 200, `ExerciseListItemDto[]`   |
| GET    | /api/exercises/{id}  | 200 `ExerciseDetailDto` or 404 |

## Test the API manually

1. With the backend running from the backend directory, open http://127.0.0.1:8000/docs.
2. Call **GET /api/categories** — you should get a JSON array of category objects (id, name, description).
3. Call **GET /api/categories/1** — you should get a single category or 404 if id 1 does not exist.
4. Call **GET /api/exercises** — you should get a JSON array of exercises; each exercise must have `categories` as an array of category objects.
5. Call **GET /api/exercises/1** — you should get a single exercise with `categories` and `fields` arrays.
6. Call **GET /api/exercises/999** (or any non-existent id) — you should get a 404 response.

If any response shape differs from your frontend types (e.g. missing fields, wrong types), fix the DTOs or the service layer before moving to the frontend integration.

## When to move on

- All four endpoints respond with the correct status codes and JSON shapes. Route modules call only the **service** layer; they do not import the repository (store). CORS is configured so that a request from http://localhost:5173 (or your frontend URL) would be allowed by the browser.

---

# Step 5 — Frontend: environment variable and API base URL

## Goal

Add an **environment variable** for the backend URL so you can change it per environment (e.g. local vs production) without editing code. Add a small **API config module** that reads this variable and exposes a function returning the base URL. The API client and api service (Step 6) will use this to build full request URLs (e.g. base URL + `/api/categories`). Using a config module keeps the base URL in one place and makes it easy to strip a trailing slash so paths can be concatenated safely.

## 5.1 Environment variable

1. **Location:** In the **frontend** folder (the same folder that contains `package.json` and `vite.config.ts`), create a file named `.env`. Do not put it in `frontend/src`; Vite reads `.env` from the project root (the frontend folder when you run `npm run dev` from there).

2. **Variable name and value:** Define a variable that Vite will expose to the client. In Vite, only variables that start with the `VITE_` prefix are embedded into the client bundle and available at runtime. Use a name like `VITE_API_BASE_URL` and set its value to your backend’s base URL (e.g. `http://localhost:8000`). Use no trailing slash so that when you concatenate paths (e.g. base + `/api/categories`), you do not get a double slash. Use no quotes unless the value contains spaces (usually not needed).

3. **Security note:** Do not put secrets (e.g. API keys) in `VITE_` variables—they are visible in the browser. The API base URL is fine for this assignment.

4. **Git:** Ensure `.env` is listed in `.gitignore` if the file contains environment-specific or sensitive values. Often `.env` is gitignored and `.env.example` (without real values) is committed as a template.

## 5.2 API config module

1. **Create the api folder and config file:** Under `frontend/src`, create a folder named `api`. Inside it, create a file named `config.ts` (or `config.tsx` if you use JSX in that file). This module will be the single place that reads the API base URL and exposes it to the rest of the app.

2. **Read the environment variable:** In the config file, read the variable using Vite’s environment API (e.g. `import.meta.env.VITE_API_BASE_URL`). The value may be `undefined` if the variable was not set (e.g. in an old build or a different environment). Provide a fallback (e.g. `http://localhost:8000`) so the app still works when the variable is missing—useful for local development.

3. **Export a function:** Export a function (e.g. `getApiBaseUrl()`) that returns the base URL as a string. Inside the function, use the value you read (or the fallback), and remove any trailing slash so that callers can always do `getApiBaseUrl() + "/api/categories"` without worrying about double slashes. Return the trimmed string.

4. **Restart the dev server:** After adding or changing `.env`, restart the Vite dev server (stop it and run `npm run dev` again). Vite reads `.env` at startup and embeds the values; a restart is required for changes to take effect.

## When to move on

- A `.env` file exists in the frontend folder with `VITE_API_BASE_URL` (or your chosen name) set to the backend base URL.
- `frontend/src/api/config.ts` exists, reads that variable (with a sensible default), and exports a function that returns the base URL with no trailing slash.
- If you changed `.env` after the dev server was already running, you have restarted the dev server so the new value is available.

---

# Step 6 — Frontend: API client and api service

## Goal

Introduce a **dedicated API client** (`api/exercisesApi.ts`) that performs all `fetch` calls to the backend, and **rename** the existing data service file from `dataService.ts` to `apiService.ts`. The api service will delegate to the API client and expose the same four functions (`getCategories`, `getCategoryById`, `getExercises`, `getExerciseById`) with the same signatures and return types, so **ExerciseListPage**, **ExerciseDetailPage**, and any other callers do not need to change—only their import path (from `dataService` to `apiService`). This keeps HTTP logic in one place (the API client) and the UI contract in the api service.

## 6.1 Create the API client

1. **Create the API client module:** Create a file `frontend/src/api/exercisesApi.ts`. This module will contain all `fetch` calls to the backend. Import the types (`CategoryDto`, `ExerciseListItemDto`, `ExerciseDetailDto`) from your types module and `getApiBaseUrl` from `./config`.

2. **Implement the four client functions:** Each function builds the full URL using `getApiBaseUrl()` and the appropriate path (`/api/categories`, `/api/categories/${id}`, `/api/exercises`, `/api/exercises/${id}`). Use GET requests, await the response, and:
   - For **getCategories** and **getExercises:** if `!res.ok`, throw an error; otherwise return `res.json()`.
   - For **getCategoryById(id)** and **getExerciseById(id):** if `res.status === 404`, return `null`; if `!res.ok`, throw; otherwise return `res.json()`.
     The API client can return `null` for 404 so that the api service can map it to `undefined` for the UI. Ensure the id is converted to a string when building the path.

3. **Export the functions:** Export all four functions so the api service can import them.

## 6.2 Rename dataService.ts to apiService.ts and delegate to the API client

1. **Rename the file:** Rename `frontend/src/services/dataService.ts` to `frontend/src/services/apiService.ts`. Update all imports in your app that referenced the data service so they import from `apiService` instead (e.g. in `ExerciseListPage.tsx` and `ExerciseDetailPage.tsx`, change the import path from `'../services/dataService'` to `'../services/apiService'`).

2. **Remove mock usage:** In `apiService.ts`, remove all imports of mock data (e.g. mock categories, mock exercises, mock exercise categories, mock exercise fields) and any logic that derived list or detail data from those mocks.

3. **Delegate to the API client:** Import the four functions from the API client (e.g. `from '../api/exercisesApi'`). Use aliases if needed to avoid name clashes (e.g. `getCategories as apiGetCategories`). Implement each exported function by calling the corresponding API client function. For **getCategoryById** and **getExerciseById**, map the client’s `null` return to `undefined` so the return type remains `Promise<CategoryDto | undefined>` and `Promise<ExerciseDetailDto | undefined>` (e.g. `const result = await apiGetExerciseById(id); return result ?? undefined;`).

4. **Preserve return types and signatures:** The api service must keep the same function names, parameters, and return types as before so that pages do not need to change. Only the implementation (and the file name) changes.

## When to move on

- `frontend/src/api/exercisesApi.ts` exists and implements the four fetch-based functions. `frontend/src/services/apiService.ts` exists (renamed from `dataService.ts`), delegates to the API client, and maps `null` to `undefined` where needed. All imports that previously pointed to `dataService` now point to `apiService`.
- From the user’s perspective, the Exercise list page and Exercise detail page behave the same as before, but data is loaded from the backend. You can confirm by running both backend and frontend and checking that the list and detail views show the same content as your seeded data, and that the browser’s Network tab shows requests to the backend.

---

# Step 7 — End-to-end verification

## Goal

Confirm the **full stack** works: the frontend loads data from the backend over the network, the list and detail views display correctly, filters work, and there are no CORS or missing-data issues. This step catches integration problems (e.g. wrong URL, wrong response shape, or CORS misconfiguration) before you consider the assignment complete.

## 7.1 Run both backend and frontend

1. **Start the backend:** Open a terminal, change into the **backend** directory, activate the virtual environment if you use one, and run the FastAPI app with uvicorn (e.g. `uvicorn main:app --reload`). Leave this terminal open; the server should keep running and show log lines when you make requests.
2. **Start the frontend:** Open a second terminal, change into the **frontend** directory, and run the dev server (e.g. `npm run dev`). Note the URL (usually http://localhost:5173).
3. **Open the app:** In a browser, open the frontend URL. The app should load; the initial page may be the exercise list or a home page depending on your routing.

## 7.2 Verification checklist

Work through the following in order:

- **Backend docs:** Open http://127.0.0.1:8000/docs (or your backend URL). Confirm that the four endpoints (GET /api/categories, GET /api/categories/{id}, GET /api/exercises, GET /api/exercises/{id}) are listed. Use “Try it out” for each: call GET /api/categories and GET /api/exercises and confirm the response body is a JSON array with the expected fields (id, name, etc.). Call GET /api/exercises/1 and confirm the single object has `categories` and `fields` arrays. Call GET /api/exercises/999 and confirm you get a 404 response.

- **Frontend — list page:** In the browser, go to the Exercises list page (or the main page that lists exercises). The list should load and show exercise names (and any other columns you display). The data should match what the backend returns (same ids, names, dates, etc.). If the list is empty or shows old mock data, check that the api service (and API client) are calling the backend and that the backend is running. Verify that the **category filter** (if you have one) works: selecting a category filters the list. Verify that the **date filter** (if you have one) works as before.

- **Frontend — detail page:** Click one exercise in the list. The detail page should open and show that exercise’s name, date, notes, assigned categories, and fields (e.g. weight, reps, distance). The content should match the backend response for that exercise id. If categories or fields are missing, check that the backend’s GET /api/exercises/{id} returns nested `categories` and `fields`, and that the frontend types match.

- **Frontend — not found:** Manually navigate to a URL that uses an invalid exercise id (e.g. /exercises/999). The app should show a “Not found” message (or similar) and a link or button to go back to the list, instead of crashing or showing empty data. This confirms that the api service returns `undefined` for 404 and that the detail page handles it.

- **Network tab:** Open the browser’s Developer Tools (F12 or similar), go to the Network tab, and reload the app or navigate through the list and detail pages. You should see requests to your backend (e.g. `http://localhost:8000/api/categories`, `http://localhost:8000/api/exercises`, and `http://localhost:8000/api/exercises/1` when you open a detail). Each request should complete with status 200 (or 404 for a non-existent id). There should be **no CORS errors** in the console. If you see CORS errors, double-check the backend’s CORS configuration (allowed origins must include the frontend’s origin, e.g. http://localhost:5173).

## 7.3 Optional: loading and error handling

- **Loading state:** If the list or detail page does not already show a loading indicator while data is being fetched, add one (e.g. a spinner or “Loading…” text) that is visible until the promise from `getExercises()` or `getExerciseById()` resolves. This improves the experience when the network is slow.
- **Error handling:** If the backend is stopped or returns an error (e.g. 500), the frontend should not fail silently. Show a short user-friendly message (e.g. “Failed to load exercises” or “Network error”) and optionally a way to retry. You can check `res.ok` in the API client (or api service) and throw or return an error state so the page can display it.

---

## Summary

After completing this assignment you will have:

- A **FastAPI backend** with **layered architecture**:
  - **Routes** in `routes/categories.py` and `routes/exercises.py`: handle HTTP requests, call the service layer, return JSON and appropriate status codes (200 or 404). They do not import the repository.
  - **Services** in `services/category_service.py` and `services/exercise_service.py`: implement use cases (list categories, get by id, list exercises with categories, get exercise detail with categories and fields). They call the repository and build DTOs; they do not import FastAPI or handle HTTP.
  - **Repository** in `repositories/store.py`: in-memory data (categories, exercises, exercise–category links, exercise fields) and get\_\* functions. This layer can be replaced later with a database without changing the service or route code.
  - **DTO models** in `models/dtoModels.py`: Pydantic models aligned with the frontend types (integer ids, same field names) so the API response shape matches what the frontend expects.
- **CORS** configured in `main.py` so the React app (e.g. http://localhost:5173) can call the API from the browser without being blocked.
- **Frontend** updated: environment variable and `api/config.ts` for the base URL; dedicated API client `api/exercisesApi.ts` performs `fetch` calls; `dataService.ts` is **renamed to** `apiService.ts`, which delegates to the API client and maps `null` to `undefined` for “not found”. Pages import from `apiService`; the same pages (list, detail, filters) work unchanged, with only the import path updated from `dataService` to `apiService`.

When you add a database (Iteration 3), you keep the same API and service layer and replace the repository (e.g. `repositories/store.py`) with database access.

---

## Reporting

Follow the instructions in **[ASSIGNMENT_ITERATION_2_REPORTING.md](Assignments/Assignment-3/ASSIGNMENT_ITERATION_2_REPORTING.md)**. That document specifies:

- Report structure (title/metadata, backend overview, frontend integration overview, testing, screenshots, optional challenges and reflection).
- Required screenshots (backend API docs, list and detail from API, not found, Network tab).
- Format and delivery (PDF, embedded screenshots, submission instructions).
- A checklist before submission.

---

_End of Assignment — Iteration 2_
