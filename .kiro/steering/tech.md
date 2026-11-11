# Tech Stack

- Runtime: Node.js (CommonJS, `require`/`module.exports`), npm
- Web: Express 4, body-parser, cors, serve-favicon
- Views: mustache-express with `.html` templates; Bootstrap 4 and jQuery served from `public/`
- Validation: express-validator (`body(...).trim().escape()`)
- DB: MySQL via `mysql2` (callback API, no ORM)
- AWS: `aws-sdk` (v2) for Secrets Manager (secret `Mydbsecret`, region `us-east-1`)
- Infra: EC2 Ubuntu bootstrapped by `UserdataScript-phase-2.sh` (installs Node and MySQL, creates DB and table, starts app on port 80)

## Configuration

Env vars: `APP_DB_HOST`, `APP_DB_USER`, `APP_DB_PASSWORD`, `APP_DB_NAME`, `APP_PORT` (default 3000).
`app/config/config.js` loads from Secrets Manager first, then falls back to local defaults and env vars.

## Commands

Run from `code/resources/codebase_partner`:

- Install: `npm install` (also needs `aws-sdk`, which is not in package.json)
- Start: `npm start` (script uses POSIX env syntax, so on Windows run `node index.js` with env vars set)
- No test, lint, or build scripts are defined.

## Conventions

- Callback style: models call `result(err, data)`. Not-found errors use `{kind: "not_found"}`.
- Use parameterized queries (`?` placeholders) for all SQL. Do not interpolate user input. `findById` currently does and should be fixed if touched.
- Never commit or echo secrets. `aws-credentials/` holds keys and a `.pem` file and must not be read, edited, or committed.
- Known issues: the stray `supplier.i` line in the update controller, and the hardcoded fallback DB credentials in `config.js`.
