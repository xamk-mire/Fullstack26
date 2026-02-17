# Reporting: Iteration 2 — Implementation Report

**Project:** Exercise/Workout Tracker  
**Iteration:** 2 of 4  
**Related assignment:** [ASSIGNMENT_ITERATION_2.md](Assignments/Assignment-3/ASSIGNMENT_ITERATION_2.md) (FastAPI Backend and Frontend Integration)

---

## 1. Purpose

Create an **implementation report** that documents your completed Iteration 2 solution. The report should describe what you built (backend with layered architecture and frontend integration), how it meets the assignment requirements, how you tested the API, and include **screenshots** of the backend API docs and the working full-stack application so that the reader can see the solution in action.

---

## 2. Task

1. Complete the assignment described in **ASSIGNMENT_ITERATION_2.md** (FastAPI Backend and Frontend Integration).
2. Write an **implementation report** (see structure below).
3. Include **screenshots** of the backend API documentation and of the frontend loading data from the API (see required screenshots below).
4. Submit the report in **PDF format** (see section 5). Screenshots must be included in the report (embedded in the PDF). Submit as specified by your course (e.g. in Learn).

---

## 3. Report structure

Your report must be submitted in **PDF format**. You may draft it in Markdown or a word processor and then export or print to PDF. It should include the following sections.

1. **Title and metadata**

   - Title: e.g. "Implementation Report — Iteration 2: FastAPI Backend and Frontend Integration"
   - Project name, your name(s), date.
   - Link(s) to your classroom GitHub repository.

2. **Backend implementation overview**

   - **Layered architecture:** Briefly describe the three layers—routes (API layer), services (use cases), repository (data access)—and the request flow (route → service → repository). Name the main files: `main.py` (FastAPI app, CORS, router registration), `models/dtoModels.py` (Pydantic DTOs), `repositories/store.py` (in-memory data and get_* functions), `routes/categories.py` and `routes/exercises.py`, `services/category_service.py` and `services/exercise_service.py`.
   - **API contract:** State the four endpoints (GET /api/categories, GET /api/categories/{id}, GET /api/exercises, GET /api/exercises/{id}) and that response shapes match the frontend types (CategoryDto, ExerciseListItemDto, ExerciseDetailDto).
   - **CORS:** Note that CORS is configured so the frontend (e.g. http://localhost:5173) can call the API.

3. **Frontend integration overview**

   - **Structure:** Environment variable (`VITE_API_BASE_URL`), `frontend/src/api/config.ts` (getApiBaseUrl), `frontend/src/api/exercisesApi.ts` (API client with fetch), `frontend/src/services/apiService.ts` (renamed from dataService, delegates to API client). Pages (ExerciseListPage, ExerciseDetailPage) import from apiService; list and detail views load data from the backend.
   - **Main behaviour:** The exercises list and exercise detail pages work as in Iteration 1 but data is loaded from the FastAPI backend; filters still work; invalid exercise id shows "Not found".

4. **Testing**

   - Describe how you tested the backend API (e.g. Swagger UI at /docs, curl, or Postman). Mention that you confirmed response shapes match the frontend types and that CORS allows the frontend origin.
   - Optionally describe how you verified the frontend (e.g. list and detail load from API, Network tab shows successful requests and no CORS errors).

5. **Screenshots of the working solution**

   - For each screenshot: a **figure number**, a **short caption** describing what is shown, and the **image** (see section 4 below for required screenshots).
   - Ensure screenshots are clear and show the relevant content (e.g. full Swagger view, full list or detail page, Network tab with request URLs and status codes).

6. **Optional: Challenges and decisions**

   - Any difficulties you ran into and how you solved them (e.g. CORS, import paths after renaming to apiService, mapping null to undefined).
   - Notable design or implementation choices (e.g. why you used a dedicated API client, how you structured the repository data).

7. **Optional: Reflection**

   - What you learned or would do differently next time (e.g. adding error handling, loading states, or preparing for a database in the next iteration).

---

## 4. Required screenshots

Include at least the following screenshots. Name the image clearly and reference them in the report with captions.

| #   | Screenshot                          | Description / caption                                                                                                                                                                                                                                                                          |
| --- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Backend API docs (Swagger/OpenAPI)**      | Full view of the FastAPI automatic docs (e.g. http://127.0.0.1:8000/docs) showing the four endpoints: GET /api/categories, GET /api/categories/{id}, GET /api/exercises, GET /api/exercises/{id}. The URL bar should show the docs address. Optionally expand one endpoint to show the response schema. |
| 2   | **Backend API response (optional)** | A successful response from one of the endpoints (e.g. GET /api/exercises or GET /api/exercises/1) from Swagger "Try it out", showing the JSON response with categories and/or fields. This can be a second view of the docs or a separate screenshot of the response body.                          |
| 3   | **Exercises list (from API)**       | Full view of the exercises list page with data loaded from the backend: header, navigation, heading "Exercises", filters, and exercise cards. Caption should state that the list is loaded from the FastAPI API (not mock data). The URL bar can show the frontend address (e.g. localhost:5173).  |
| 4   | **Exercise detail (from API)**      | Full view of one exercise detail page (e.g. /exercises/1): exercise name, date, notes, category badges, and the Details section with fields. Caption should state that the data is loaded from the backend (GET /api/exercises/{id}).                                                              |
| 5   | **Not found / invalid id**          | The view when the exercise id is invalid (e.g. /exercises/999): the "Not found" message and the link back to the list. Confirms that the api service returns undefined for 404 and the detail page handles it.                                                                                  |
| 6   | **Network tab (API calls)**         | Browser Developer Tools → Network tab showing successful requests to the backend (e.g. `http://localhost:8000/api/categories`, `http://localhost:8000/api/exercises`, and optionally `/api/exercises/1`) with status 200. Caption should note that there are no CORS errors and data is loaded from the API. |

---

## 5. Format and delivery

- **Report format:** The report **must** be returned in **PDF format**. No other formats (e.g. Markdown only or Word) are accepted.
- **Screenshots:** Embed all required screenshots in the PDF (e.g. paste or insert images into your document before exporting to PDF). Each screenshot should have a figure number and caption as described in section 3.
- **Delivery:** Submit the single PDF file as specified by your course (e.g. in Learn by the given deadline).

---

## 6. Checklist before submission

- [ ] Report is in **PDF format** and includes all sections in section 3 (at least title/metadata, backend overview, frontend integration overview, testing, screenshots with captions).
- [ ] All six required screenshots (backend docs, optional API response, list from API, detail from API, not found, Network tab) are **embedded in the PDF** and captioned.
- [ ] Screenshots clearly show the working solution (readable text, relevant UI or API docs visible).
- [ ] The backend runs with `uvicorn main:app --reload` and the frontend runs with `npm run dev`; the frontend loads data from the backend and matches the behaviour shown in the screenshots.
- [ ] You have described how you tested the API (e.g. Swagger UI) and confirmed CORS and response shapes.

---
