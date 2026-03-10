# Reporting: Iteration 3 — Implementation Report

**Project:** Exercise/Workout Tracker  
**Iteration:** 3 of 4  
**Related assignment:** [ASSIGNMENT_ITERATION_3.md](./ASSIGNMENT_ITERATION_3.md) (PostgreSQL Database and Exercise Create/Update)

---

## 1. Purpose

Create an **implementation report** that documents your completed Iteration 3 solution. The report should describe what you built (PostgreSQL persistence via SQLAlchemy 2, database-backed repository, create and update exercises, create and edit forms), how it meets the assignment requirements, how you tested the solution, and include **screenshots** of the database setup, API docs, and the working full-stack application including the create and edit forms.

---

## 2. Task

1. Complete the assignment described in **ASSIGNMENT_ITERATION_3.md** (PostgreSQL Database and Exercise Create/Update).
2. Write an **implementation report** (see structure below).
3. Include **screenshots** of the database schema, backend API documentation, create form, edit form, and the frontend loading and persisting data (see required screenshots below).
4. Submit the report in **PDF format** (see section 5). Screenshots must be included in the report (embedded in the PDF). Submit as specified by your course (e.g. in Learn).

---

## 3. Report structure

Your report must be submitted in **PDF format**. You may draft it in Markdown or a word processor and then export or print to PDF. It should include the following sections.

1. **Title and metadata**

   - Title: e.g. "Implementation Report — Iteration 3: PostgreSQL Database and Exercise Create/Update"
   - Project name, your name(s), date.
   - Link(s) to your classroom GitHub repository.

2. **Database setup**

   - Briefly describe how you installed and ran PostgreSQL (locally or via Docker). Do **not** include passwords.
   - State the database name (e.g. `exercise_tracker`) and how you created it.
   - Explain how the backend reads the connection URL (e.g. `DATABASE_URL` environment variable, `.env` file).

3. **Backend implementation overview**

   - **ORM models and schema:** Describe the SQLAlchemy 2 declarative models (e.g. `models/orm_models.py`): `Category`, `Exercise`, `ExerciseField`, and the `exercise_categories` association table. Mention the relationships (many-to-many for exercise–category, one-to-many for exercise–fields).
   - **Repository:** Explain that `repositories/store.py` was replaced with a database-backed implementation using SQLAlchemy 2 (e.g. `select()`, `session.execute()`, `session.add()`, `session.get()`). The service layer continues to call the same logical operations.
   - **Schema creation and seeding:** State how you create tables (e.g. `create_tables()` at startup, Alembic migrations, or SQL script) and how you seed initial data (e.g. `python -m scripts.seed_db`).
   - **API contract:** List all six endpoints: `GET /api/categories`, `GET /api/categories/{id}`, `GET /api/exercises`, `GET /api/exercises/{id}`, `POST /api/exercises`, `PUT /api/exercises/{id}`. Describe the request body for POST and PUT (name, notes, date, categoryIds, fields).

4. **Frontend implementation overview**

   - **Create and edit forms:** Describe the form page(s) at `/exercises/new` and `/exercises/:id/edit`. Mention the fields: name (required), notes (optional), date (required), categories (checkboxes), fields (dynamic list with add/remove).
   - **API client and api service:** Note the added `createExercise` and `updateExercise` in `api/exercisesApi.ts` and `apiService.ts`.
   - **Navigation:** Explain that the list page has a "New exercise" button and the detail page has an "Edit" button.

5. **Testing**

   - Describe how you tested the backend API (e.g. Swagger UI at /docs for GET, POST, PUT).
   - Describe how you verified that data persists (e.g. create an exercise, restart the backend, reload the frontend—the new exercise still appears).
   - Optionally describe how you tested the create and edit forms, validation, and handling of invalid ids.

6. **Screenshots of the working solution**

   - For each screenshot: a **figure number**, a **short caption** describing what is shown, and the **image** (see section 4 below for required screenshots).
   - Ensure screenshots are clear and show the relevant content.

7. **Optional: Challenges and decisions**

   - Any difficulties you ran into and how you solved them (e.g. SQLAlchemy 2 query style, datetime vs ISO string conversion, session handling).
   - Notable design or implementation choices (e.g. use of `selectinload` for eager loading, reuse of one form component for create and edit).

8. **Optional: Reflection**

   - What you learned or would do differently next time (e.g. using Alembic for migrations, adding more validation, preparing for Iteration 4).

---

## 4. Required screenshots

Include at least the following screenshots. Name the images clearly and reference them in the report with captions.

| #   | Screenshot                           | Description / caption                                                                                                                                                                                                                                                                                        |
| --- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | **Backend API docs (Swagger)**       | Full view of the FastAPI docs (e.g. http://127.0.0.1:8000/docs) showing all six endpoints: GET /api/categories, GET /api/categories/{id}, GET /api/exercises, GET /api/exercises/{id}, POST /api/exercises, PUT /api/exercises/{id}. Optionally expand POST or PUT to show the request body schema.          |
| 2   | **Create exercise form**             | The create form at `/exercises/new` with name, notes, date, category checkboxes, and at least one field row (name, value, unit). Caption should state that the form submits to POST /api/exercises. The "New exercise" navigation or the form heading should be visible.                                     |
| 3   | **Edit exercise form**               | The edit form at `/exercises/:id/edit` pre-filled with an existing exercise (e.g. name, notes, date, selected categories, and fields). Caption should state that the form loads data via GET /api/exercises/{id} and submits to PUT /api/exercises/{id}.                                                     |
| 4   | **Exercises list (with new item)**   | The exercises list page showing at least one exercise that was created via the create form. Caption should state that data is loaded from the database and persists across restarts.                                                                                                                          |
| 5   | **Exercise detail (created/edited)** | The detail page of an exercise that was created or edited via the forms, showing name, date, notes, categories, and fields. Confirms that create and update work end-to-end.                                                                                                                                  |
| 6   | **Database view (optional)**         | A database client (e.g. pgAdmin, DBeaver, psql output, or TablePlus) showing one or more tables with sample rows (e.g. `exercises`, `exercise_fields`, or `exercise_categories`). Confirms that data is stored in PostgreSQL.                                                                                 |

Screenshot 6 is optional but recommended. If you include it, do **not** show passwords or sensitive connection details.

---

## 5. Format and delivery

- **Report format:** The report **must** be returned in **PDF format**. No other formats (e.g. Markdown only or Word) are accepted.
- **Screenshots:** Embed all required screenshots in the PDF (e.g. paste or insert images into your document before exporting to PDF). Each screenshot should have a figure number and caption as described in section 3.
- **Delivery:** Submit the single PDF file as specified by your course (e.g. in Learn by the given deadline).

---

## 6. Checklist before submission

- [ ] Report is in **PDF format** and includes all sections in section 3 (at least title/metadata, database setup, backend overview, frontend overview, testing, screenshots with captions).
- [ ] All required screenshots (API docs, create form, edit form, list, detail, and optionally database view) are **embedded in the PDF** and captioned.
- [ ] Screenshots clearly show the working solution (readable text, relevant UI or API docs visible).
- [ ] The backend runs with `DATABASE_URL` set and `uvicorn main:app --reload`; the frontend runs with `npm run dev`; data persists in PostgreSQL and the create/edit forms work as shown in the screenshots.
- [ ] You have described how you tested the API (e.g. Swagger UI) and verified data persistence across restarts.
