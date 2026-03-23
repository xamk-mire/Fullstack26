# Reporting: Iteration 4 — Implementation Report

**Project:** Exercise/Workout Tracker  
**Iteration:** 4 of 4  
**Related assignment:** [ASSIGNMENT_ITERATION_4.md](./ASSIGNMENT_ITERATION_4.md) (Authentication and Authorization)

---

## 1. Purpose

Create an **implementation report** that documents your completed Iteration 4 solution. The report should describe what you built (user registration, login, JWT, protected exercise routes, scoped data by user, frontend auth, Docker setup), how it meets the assignment requirements, how you tested the solution, and include **screenshots** of the login flow, protected exercise list, API with authentication, and Docker containers running.

---

## 2. Task

1. Complete the assignment described in **ASSIGNMENT_ITERATION_4.md** (Authentication and Authorization).
2. Write an **implementation report** (see structure below).
3. Include **screenshots** of the login page, protected exercise list, and API documentation showing auth and protected endpoints (see required screenshots below).
4. Submit the report in **PDF format** (see section 5). Screenshots must be included in the report (embedded in the PDF). Submit as specified by your course (e.g. in Learn).

---

## 3. Report structure

Your report must be submitted in **PDF format**. You may draft it in Markdown or a word processor and then export or print to PDF. It should include the following sections.

1. **Title and metadata**

   - Title: e.g. "Implementation Report — Iteration 4: Authentication and Authorization"
   - Project name, your name(s), date.
   - Link(s) to your classroom GitHub repository.

2. **Backend auth implementation overview**

   - **User model and migration:** Describe the `User` ORM model and the `user_id` column added to `Exercise`. Mention that `user_id` is nullable for backward compatibility with existing data.
   - **Auth routes:** State `POST /api/auth/register` and `POST /api/auth/login`. Describe the JWT flow (payload with `sub`, expiry) and `get_current_user` dependency.
   - **Protected routes:** Explain that all exercise endpoints (`GET`, `POST`, `PUT`) require authentication via `get_current_user`. Categories remain public.
   - **Authorization:** Describe how exercises are filtered and scoped by `user_id` in the repository, service, and routes. Users can only access their own exercises.

3. **Frontend auth implementation overview**

   - **Auth context:** Describe `AuthContext` (user, token, login, logout, isReady) and token persistence in `localStorage`.
   - **Login and register pages:** Paths, form fields, API calls.
   - **Protected routes:** How `ProtectedRoute` redirects unauthenticated users to `/login`.
   - **API client:** How the client adds the Bearer token and handles 401 (clear auth, redirect to login).
   - **Logout:** Where the logout button appears and what it does.

4. **Testing**

   - Describe how you tested the backend (e.g. Swagger with register, login, authorize with token, call protected endpoints).
   - Describe how you verified authorization: two users, each sees only their own exercises.
   - Describe how you tested the frontend: login flow, protected route redirect when not logged in, logout.

5. **Screenshots of the working solution**

   - For each screenshot: a **figure number**, a **short caption** describing what is shown, and the **image** (see section 4 below for required screenshots).
   - Ensure screenshots are clear and show the relevant content.

6. **Docker setup**

   - Describe the Docker configuration: `docker-compose.yml` structure (db, api services), `backend/Dockerfile`, environment variables, volume for database persistence.
   - Explain how to run the stack (e.g. `docker compose up -d --build`) and that migrations run on API startup.

7. **Optional: Challenges and decisions**

   - Any difficulties you ran into (e.g. OAuth2 form vs JSON for login, 401 handling, redirect after login, Docker networking).
   - Notable design choices (e.g. keeping categories public, nullable `user_id` for migration).

8. **Frontend design update (Part C — optional, graded)**

   - If you completed Part C, describe your design changes (e.g. layout, colors, typography, component styling) and include before/after or updated screenshots. This section is **required to receive points** for Part C.

---

## 4. Required screenshots

Include at least the following screenshots. Name the images clearly and reference them in the report with captions.

| #   | Screenshot                           | Description / caption                                                                                                                                                                                                                                                                                    |
| --- | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Login page**                       | Full view of the login page at `/login`. Caption should state that unauthenticated users are redirected here when accessing protected routes.                                                                                                                                                             |
| 2   | **Exercise list after login**        | The exercises list page after logging in, showing exercises belonging to the logged-in user. Caption should state that data is scoped by user and loaded via protected endpoints.                                                                                                                         |
| 3   | **API docs with auth**               | Swagger or Scalar showing the auth endpoints (`POST /api/auth/register`, `POST /api/auth/login`) and/or a protected endpoint (e.g. `GET /api/exercises`) called with a valid token (e.g. "Authorize" with Bearer token, then successful response). The URL bar should show the docs address.               |
| 4   | **401 without token (optional)**     | A request to a protected endpoint (e.g. `GET /api/exercises`) without a token, showing 401 response. Confirms that unauthenticated requests are rejected.                                                                                                                                                 |
| 5   | **Docker containers**                | `docker compose ps` or `docker compose up` showing the running containers (db, api, optionally frontend). Caption should state that the stack runs with Docker Compose.                                                                                                                                  |
| 6   | **Frontend design (Part C)**         | If you completed Part C: before/after or updated screenshots showing your design improvements. **Required** to receive points for Part C.                                                                                                                                                                |

Screenshot 4 is optional but recommended. Screenshot 6 is required **only if** you completed Part C (to receive Part C points).

---

## 5. Format and delivery

- **Report format:** The report **must** be returned in **PDF format**. No other formats (e.g. Markdown only or Word) are accepted.
- **Screenshots:** Embed all required screenshots in the PDF (e.g. paste or insert images into your document before exporting to PDF). Each screenshot should have a figure number and caption as described in section 3.
- **Delivery:** Submit the single PDF file as specified by your course (e.g. in Learn by the given deadline).

---

## 6. Checklist before submission

- [ ] Report is in **PDF format** and includes all sections in section 3 (at least title/metadata, backend auth overview, frontend auth overview, testing, Docker setup, screenshots with captions).
- [ ] All required screenshots (login page, exercise list after login, API docs with auth, Docker containers) are **embedded in the PDF** and captioned.
- [ ] Screenshots clearly show the working solution (readable text, relevant UI or API docs visible).
- [ ] Register, login, protected routes, and logout work as described. Each user sees only their own exercises.
- [ ] Docker stack runs with `docker compose up`; db and api containers start successfully; data persists in a volume.
- [ ] You have described how you tested the backend (e.g. Swagger with token) and verified authorization with two users.
- [ ] If you completed Part C: you have documented your design changes and included screenshots (required for Part C points).
