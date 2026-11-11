# Implementation Plan: Student Search and Filter

## Overview

Add server-side search and filtering to `GET /students/` in `code/resources/codebase_partner`. The work is done in JavaScript (CommonJS), matching the design. The model gets pure helpers and a new `Supplier.search` function, the controller's `findAll` normalizes query parameters and builds the view model, and the list view gets a GET form, a clear link, and an empty-result row. `getAll` and the route in `index.js` stay unchanged. Do not read or modify `aws-credentials/`.

## Tasks

- [ ] 1. Set up the test framework
  - [-] 1.1 Add jest and fast-check as devDependencies
    - Edit `code/resources/codebase_partner/package.json`: add `jest` and `fast-check` to `devDependencies` with exact pinned versions, and add the script `"test": "jest"`
    - Tests live in `code/resources/codebase_partner/test/`
    - Run `npm install` from `code/resources/codebase_partner`
    - _Requirements: 6.1_

- [ ] 2. Implement model search helpers and query
  - [x] 2.1 Add `normalizeParam`, `escapeLike`, `buildSearchQuery`, and `search` to `app/models/supplier.model.js`
    - `normalizeParam(value, max)`: return `""` for non-strings, otherwise trim and truncate to `max`
    - `escapeLike(s)`: escape `\`, `%`, `_` with a backslash
    - `buildSearchQuery(criteria)`: return `{sql, params}` using `LOWER(name) LIKE LOWER(?) ESCAPE '\\\\'`, `LOWER(city) = LOWER(?)`, `LOWER(state) = LOWER(?)` joined by `AND`; with no active criteria return exactly `SELECT * FROM students` and `[]`; no `ORDER BY`
    - `search(criteria, result)`: use `Supplier.dbConnect()`, `dbConn.query(sql, params, cb)`, log errors, call `result(err, null)` or `result(null, rows)`
    - Leave `getAll`, `findById`, and the other model functions unchanged
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 3.1, 3.2, 3.3, 3.5, 6.1, 6.3, 7.1_

  - [~] 2.2 Write property test for `normalizeParam`
    - File: `test/normalizeParam.property.test.js`
    - **Property 1: Normalization yields a bounded, trimmed string**
    - Tag with `Feature: student-search-filter, Property 1: Normalization yields a bounded, trimmed string`; at least 100 runs
    - **Validates: Requirements 1.4, 1.5, 2.5, 2.6, 3.5, 6.3**

  - [~] 2.3 Write property test for `escapeLike`
    - File: `test/escapeLike.property.test.js`
    - **Property 2: LIKE escaping round trip**
    - Tag with `Feature: student-search-filter, Property 2: LIKE escaping round trip`; at least 100 runs
    - **Validates: Requirements 1.3**

  - [~] 2.4 Write property test for query text independence
    - File: `test/buildSearchQuery.sql.property.test.js`
    - **Property 3: Query text is independent of user values**
    - Include SQL injection style strings in the generator; assert placeholder count equals `params.length`
    - Tag with `Feature: student-search-filter, Property 3: Query text is independent of user values`; at least 100 runs
    - **Validates: Requirements 1.2, 2.4, 6.1**

  - [~] 2.5 Write property test for params and unit test for the no-criteria query
    - File: `test/buildSearchQuery.params.property.test.js`
    - **Property 4: Params reflect exactly the active criteria**
    - Tag with `Feature: student-search-filter, Property 4: Params reflect exactly the active criteria`; at least 100 runs
    - Add a unit example asserting no criteria yields `{sql: "SELECT * FROM students", params: []}`
    - **Validates: Requirements 1.4, 2.5, 3.1, 3.2, 3.3, 3.5**

- [ ] 3. Update the controller
  - [~] 3.1 Change `findAll` in `app/controller/supplier.controller.js`
    - Build `criteria` with `Supplier.normalizeParam(req.query.name, 100)` and `(req.query.city, 50)` / `(req.query.state, 50)`
    - Call `Supplier.search(criteria, cb)`
    - On error render `500` with the generic message "There was a problem retrieving the list of students"
    - On success render `supplier-list-all` with `students`, `search`, `hasCriteria`, and `noResults` (`hasCriteria && data.length === 0`)
    - _Requirements: 1.6, 2.7, 3.4, 4.6, 5.1, 5.2, 5.3, 5.4, 6.3, 6.4_

  - [~] 3.2 Write property test and unit tests for `findAll`
    - File: `test/findAll.property.test.js`
    - **Property 5: Empty-result message flag**
    - Mock `Supplier.search` and `res.render`; assert `noResults` and `hasCriteria` flags for generated query params and result lists; assert `500` is rendered and `supplier-list-all` is not on model error
    - Unit example: the 500 message contains no SQL text
    - Tag with `Feature: student-search-filter, Property 5: Empty-result message flag`; at least 100 runs
    - **Validates: Requirements 1.6, 2.7, 3.4, 4.6, 5.1, 5.2, 5.3, 5.4, 6.4**

- [ ] 4. Update the list view
  - [x] 4.1 Edit `views/supplier-list-all.html`
    - Add `<form method="get" action="/students/">` above the table with three text inputs named `name`, `city`, `state`, bound to `{{search.name}}`, `{{search.city}}`, `{{search.state}}` (double braces only), `maxlength` 100/50/50, and a submit button
    - Add `{{#hasCriteria}}<a href="/students/">Clear search</a>{{/hasCriteria}}`
    - In `<tbody>`, after the `{{#students}}` section, add `{{#noResults}}<tr><td colspan="7">No students match your search.</td></tr>{{/noResults}}`
    - Keep the existing columns (Name, Address, City, State, Email, Phone) and the edit link unchanged
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 5.1, 5.2, 6.2, 7.4, 7.5_

  - [~] 4.2 Write view rendering tests
    - File: `test/supplier-list-all.view.test.js`, rendering the template with mustache
    - Assert: form `method="get"` and `action="/students/"`, exactly three text inputs, existing headers and edit link, one row per record, zero rows with headers only
    - Assert values containing `<script>"'&` are escaped in the inputs
    - Assert the clear link appears only when `hasCriteria` is true, and the no-match message appears exactly once only when `noResults` is true
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 5.1, 5.2, 5.3, 6.2, 7.4, 7.5_

- [~] 5. Checkpoint - Ensure all tests pass
  - Run `npx jest --runInBand` from `code/resources/codebase_partner`. Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties; unit tests cover specific examples
- MySQL matching semantics (the `ESCAPE` clause, case-insensitivity) are not covered by automated tests here; check them manually against a local MySQL instance
- Requirements 7.2 and 7.3 cover existing behavior that this feature does not touch

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "2.1", "4.1"] },
    { "id": 1, "tasks": ["2.2", "2.3", "2.4", "2.5", "3.1", "4.2"] },
    { "id": 2, "tasks": ["3.2"] }
  ]
}
```
