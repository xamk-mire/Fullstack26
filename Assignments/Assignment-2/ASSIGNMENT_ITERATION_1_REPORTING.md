# Reporting: Iteration 1 — Implementation Report

**Project:** Exercise/Workout Tracker  
**Iteration:** 1 of 4  
**Related assignment:** [ASSIGNMENT_ITERATION_1.md](./ASSIGNMENT_ITERATION_1.md) (Basic Frontend)

---

## 1. Purpose

Create an **implementation report** that documents your completed Iteration 1 solution. The report should describe what you built, how it meets the assignment requirements, and include **screenshots** of the working application so that the reader can see the solution in action.

---

## 2. Task

1. Complete the assignment described in **ASSIGNMENT_ITERATION_1.md** (Basic Frontend).
2. Write an **implementation report** (see structure below).
3. Include **screenshots** of the running application for each main view and key behaviour (see required screenshots below).
4. Submit the report in **PDF format** (see section 5). Screenshots must be included in the report (embedded in the PDF). Submit as specified by your course (e.g. in Learn).

---

## 3. Report structure

Your report must be submitted in **PDF format**. You may draft it in Markdown or a word processor and then export or print to PDF. It should include the following sections.

1. **Title and metadata**

   - Title: e.g. "Implementation Report — Iteration 1: Basic Frontend"
   - Project name, your name(s), date.
   - Link(s) to your classroom gihub repository

2. **Implementation overview**

   - **Structure:** Brief description of folders/files (e.g. `src/types`, `src/data`, `src/services`, `src/components`, `src/pages`, routing in `App.tsx`).
   - **Main features:** Exercises list with date and category filters; exercise detail page with categories and fields; layout (Header, Nav); routing (`/`, `/exercises`, `/exercises/:id`).

3. **Screenshots of the working solution**

   - For each screenshot: a **figure number**, a **short caption** describing what is shown, and the **image** (see section 4 below for required screenshots).
   - Ensure screenshots are clear and show the relevant UI (e.g. full view of the list or detail, filters visible where applicable).

4. **Optional: Challenges and decisions**

   - Any difficulties you ran into and how you solved them.
   - Notable design or implementation choices (e.g. how you structured the data service, how you implemented filters).

5. **Optional: Reflection**
   - What you learned or would do differently next time.

---

## 4. Required screenshots

Include at least the following screenshots. Name the image clearly and reference them in the report with captions.

| #   | Screenshot                               | Description / caption                                                                                                                                                                                                           |
| --- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Exercises list (default)**             | Full view of the exercises list page at `/exercises`: header, navigation, heading "Exercises", date and category filters, and at least two exercise cards visible. The URL bar should show `/exercises` (optional but helpful). |
| 2   | **Exercises list (filtered)**            | The same list after applying at least one filter (e.g. a date range such as "This week" or "This month", or one or more categories). Caption should state which filter(s) were applied and that the list updates correctly.     |
| 3   | **Exercise detail**                      | Full view of one exercise detail page (e.g. `/exercises/1`): exercise name, date, notes, category badges, and the "Details" section with exercise fields (e.g. weight, sets, reps). Include the "Back to Exercises" link.       |
| 4   | **Exercise detail (different exercise)** | Detail page of another exercise (e.g. a cardio/running exercise with different fields such as distance and duration) to show that fields and categories vary per exercise.                                                      |
| 5   | **Not found / invalid id**               | The exercise detail page when the id is invalid (e.g. `/exercises/999` or `/exercises/abc`): the "Not found" message and the link back to the list.                                                                             |

---

## 5. Format and delivery

- **Report format:** The report **must** be returned in **PDF format**. No other formats (e.g. Markdown only or Word) are accepted.
- **Screenshots:** Embed all required screenshots in the PDF (e.g. paste or insert images into your document before exporting to PDF). Each screenshot should have a figure number and caption as described in section 3.
- **Delivery:** Submit the single PDF file as specified by your course (e.g. in Learn by the given deadline).

---

## 6. Checklist before submission

- [ ] Report is in **PDF format** and includes all sections in section 3 (at least title/metadata, implementation overview, screenshots with captions).
- [ ] All five required screenshots (list default, list filtered, detail 1, detail 2, not found) are **embedded in the PDF** and captioned.
- [ ] Screenshots clearly show the working solution (readable text, relevant UI visible).
- [ ] The application runs with `npm run dev` and matches the behaviour shown in the screenshots.

---
