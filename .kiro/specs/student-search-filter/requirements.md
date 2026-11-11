# Requirements Document

## Introduction

This feature adds server-side search and filtering to the student list page of the coffee partners web app. Users can search students by name and filter by city or state. Results render on the same list page (`GET /students/`) using the existing mustache view. All SQL uses parameterized queries. The design stays simple, as befits a lab codebase.

## Glossary

- **Student_List_Page**: The server-rendered page at `GET /students/` that displays student records (view `supplier-list-all.html`).
- **Search_Form**: The HTML form on the Student_List_Page that submits search and filter values using the GET method.
- **Student_Service**: The server-side controller and model code that reads query parameters and retrieves student records from the database.
- **Name_Query**: The text value of the `name` query parameter, trimmed of leading and trailing whitespace.
- **City_Filter**: The value of the `city` query parameter, trimmed of leading and trailing whitespace.
- **State_Filter**: The value of the `state` query parameter, trimmed of leading and trailing whitespace.
- **Student_Record**: A row in the `students` table.
- **Active_Criteria**: The set of Name_Query, City_Filter, and State_Filter values that are non-empty.

## Requirements

### Requirement 1: Search by Name

**User Story:** As a user, I want to search students by name, so that I can find a specific student quickly.

#### Acceptance Criteria

1. WHEN a request to the Student_List_Page includes a Name_Query that is non-empty after leading and trailing whitespace is removed, THE Student_Service SHALL return only Student_Records whose name contains the trimmed Name_Query as a case-insensitive substring.
2. WHEN a request to the Student_List_Page includes a non-empty Name_Query, THE Student_Service SHALL pass the Name_Query to the database as a parameterized query value.
3. IF the Name_Query contains the SQL wildcard characters `%` or `_`, THEN THE Student_Service SHALL match those characters literally.
4. IF a request to the Student_List_Page has no Name_Query or a Name_Query that is empty after leading and trailing whitespace is removed, THEN THE Student_Service SHALL return all Student_Records.
5. IF the Name_Query exceeds 100 characters, THEN THE Student_Service SHALL truncate it to 100 characters before building the query and SHALL complete the request without returning an error.
6. IF no Student_Record matches a non-empty Name_Query, THEN THE Student_List_Page SHALL display the message "No students match your search." as specified in Requirement 5.

### Requirement 2: Filter by City or State

**User Story:** As a user, I want to filter students by city or state, so that I can narrow the list to a location.

#### Acceptance Criteria

1. WHEN a request to the Student_List_Page includes a non-empty City_Filter and no State_Filter, THE Student_Service SHALL return only Student_Records whose city equals the City_Filter, ignoring letter case, in the same order as the unfiltered list.
2. WHEN a request to the Student_List_Page includes a non-empty State_Filter and no City_Filter, THE Student_Service SHALL return only Student_Records whose state equals the State_Filter, ignoring letter case, in the same order as the unfiltered list.
3. WHEN a request to the Student_List_Page includes both a non-empty City_Filter and a non-empty State_Filter, THE Student_Service SHALL return only Student_Records whose city equals the City_Filter and whose state equals the State_Filter, both ignoring letter case.
4. WHEN a request to the Student_List_Page includes a non-empty City_Filter or State_Filter, THE Student_Service SHALL pass each supplied filter value to the database as a parameterized query value and SHALL NOT concatenate the value into the query text.
5. IF a City_Filter or State_Filter is empty or contains only whitespace, THEN THE Student_Service SHALL ignore that filter and apply no restriction for it.
6. IF a City_Filter or State_Filter exceeds 50 characters after trimming, THEN THE Student_Service SHALL truncate it to 50 characters before building the query and SHALL complete the request without returning an error.
7. IF no Student_Record matches the supplied filters, THEN THE Student_List_Page SHALL display the message "No students match your search." as specified in Requirement 5.

### Requirement 3: Combine Search and Filters

**User Story:** As a user, I want to combine name search with city and state filters, so that I can refine results in one step.

#### Acceptance Criteria

1. WHEN a request to the Student_List_Page includes two or three Active_Criteria, THE Student_Service SHALL return only Student_Records that satisfy every Active_Criteria simultaneously.
2. WHEN a request to the Student_List_Page includes no Active_Criteria, THE Student_Service SHALL return all Student_Records.
3. WHEN the Student_List_Page is requested without any query parameters, THE Student_Service SHALL return the same set of Student_Records, in the same order, as the list page returned before this feature was added.
4. IF a request to the Student_List_Page includes two or more Active_Criteria and no Student_Record satisfies all of them, THEN THE Student_Service SHALL return an empty result set and THE Student_List_Page SHALL display the message "No students match your search."
5. WHEN a request to the Student_List_Page includes a query parameter whose value is empty or contains only whitespace, THE Student_Service SHALL exclude that parameter from the Active_Criteria and SHALL apply only the remaining Active_Criteria.

### Requirement 4: Render Results on the Same Page

**User Story:** As a user, I want results to appear on the same list page, so that I can search without leaving the page.

#### Acceptance Criteria

1. THE Student_List_Page SHALL display a Search_Form containing exactly three text inputs: one for name, one for city, and one for state.
2. WHEN the user submits the Search_Form, THE Search_Form SHALL send the values to `GET /students/` as the query parameters `name`, `city`, and `state`, and THE Student_List_Page SHALL reload at `GET /students/` in the same browser tab.
3. WHEN the Student_Service returns one or more Student_Records for a request, THE Student_List_Page SHALL render each returned Student_Record as one row in the existing results table, with no rows omitted and no rows duplicated.
4. WHEN the Student_List_Page renders after a search, THE Search_Form SHALL display the submitted Name_Query, City_Filter, and State_Filter values in their respective inputs, escaped as text so that markup characters are not interpreted as HTML.
5. WHILE at least one Active_Criteria is present, THE Student_List_Page SHALL display a link that navigates to `GET /students/` with no query parameters, and WHEN the user activates the link, THE Student_List_Page SHALL render all Student_Records with empty Search_Form inputs.
6. IF the Student_Service returns an error for a request, THEN THE Student_List_Page SHALL render `views/500.html` and SHALL NOT render a partial results table.

### Requirement 5: Empty Result Feedback

**User Story:** As a user, I want clear feedback when nothing matches, so that I know the search worked.

#### Acceptance Criteria

1. IF the Student_Service returns zero Student_Records for a request with at least one Active_Criteria, THEN THE Student_List_Page SHALL display the message "No students match your search." exactly once, in place of the results table rows, and SHALL retain the submitted values in the search inputs.
2. IF the Student_Service returns zero Student_Records for a request with no Active_Criteria, THEN THE Student_List_Page SHALL display an empty results table containing zero data rows and SHALL NOT display the message "No students match your search."
3. IF the Student_Service returns one or more Student_Records, THEN THE Student_List_Page SHALL NOT display the message "No students match your search."
4. IF the Student_Service fails to return a result for a request, THEN THE Student_List_Page SHALL NOT display the message "No students match your search."

### Requirement 6: Safe Handling of Search Input

**User Story:** As a maintainer, I want search input handled safely, so that the feature does not introduce injection or display vulnerabilities.

#### Acceptance Criteria

1. THE Student_Service SHALL pass every user-supplied value of the search and filter feature (Name_Query, City_Filter, State_Filter) to the database as a bound parameter using `?` placeholders, and SHALL NOT concatenate or interpolate any user-supplied value into the SQL text.
2. WHEN the Student_List_Page renders the submitted Name_Query, City_Filter, or State_Filter, THE Student_List_Page SHALL HTML-escape each value so that the characters `<`, `>`, `&`, `"`, and `'` are displayed as literal text and are not interpreted as markup or script.
3. IF a query parameter for Name_Query, City_Filter, or State_Filter is provided more than once (array) or in nested form (object) instead of as a single string, THEN THE Student_Service SHALL treat that parameter as an empty string, apply no filtering for it, and complete the request without returning an error.
4. IF the database returns an error while processing a search, THEN THE Student_List_Page SHALL render `views/500.html` with a generic error message, SHALL NOT display SQL text or database error details to the user, and SHALL leave stored student records unchanged.

### Requirement 7: Preserve Existing Behavior

**User Story:** As a maintainer, I want existing behavior preserved, so that the feature does not break other pages.

#### Acceptance Criteria

1. THE Student_Service SHALL invoke every model function's completion callback as `result(err, data)`, where `err` is `null` on success and `data` is `null` on failure.
2. IF a model function does not find the requested Student_Record, THEN THE Student_Service SHALL call `result` with `err` set to an object whose `kind` property equals `"not_found"` and `data` set to `null`.
3. WHEN a user successfully adds, updates, or removes a Student_Record, THE System SHALL redirect the browser to `/students`.
4. THE Student_List_Page SHALL display, for each Student_Record, exactly the columns Name, Address, City, State, Email, and Phone, in that order, followed by one edit link that navigates to the update page for that Student_Record.
5. WHEN the Student_List_Page contains zero Student_Records, THE Student_List_Page SHALL display the column headers Name, Address, City, State, Email, and Phone with no record rows.
