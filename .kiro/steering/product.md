# Product

AWS Academy capstone lab project: a "coffee partners" web app (package name `coffee_api`) for managing records through a server-rendered UI.

- Users list, add, update, and delete entries.
- Code and UI use "supplier" naming, while the DB table and user-facing text use "students" (`students` table, `STUDENTS` database). Both terms refer to the same entity; keep the existing naming when editing.
- Intended to be deployed on an AWS EC2 Ubuntu instance with MySQL on the same host. It can later move to RDS, with credentials from AWS Secrets Manager.
- This is a lab/teaching codebase. Prefer small, simple changes over heavy abstractions.
