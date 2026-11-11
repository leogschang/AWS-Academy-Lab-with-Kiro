# Structure

```
.kiro/steering/            Steering docs
aws-credentials/           Sensitive keys (do not read or modify)
UserdataScript-phase-2.sh  EC2 user-data bootstrap script
code/resources/codebase_partner/   Node app root
  index.js                 Express setup, middleware, routes, server start
  app/
    config/config.js       DB config (Secrets Manager, env, defaults)
    controller/            Request handlers and validation (supplier.controller.js)
    models/                MySQL data access (supplier.model.js)
  views/                   Mustache HTML templates (home, supplier-*, 404, 500, header/nav/footer partials)
  public/                  Static assets: css, js, img
```

## Patterns

- MVC-style: routes in `index.js` -> controller -> model -> MySQL.
- Routes: `GET /`, `GET /students/`, `GET|POST /supplier-add`, `GET /supplier-update/:id`, `POST /supplier-update`, `POST /supplier-remove/:id`. Mutations use form POSTs and redirect to `/students`.
- New entity logic goes in a new `*.controller.js` and `*.model.js` pair, and routes get registered in `index.js`.
- Models open a new DB connection per request (`Supplier.dbConnect()`), by design.
- Errors render `views/500.html` with a `message`. Missing records return 404.
- Shared form markup lives in `supplier-form-fields.html`. Reuse it for add and update views.
- Style: 4-space indentation, double-quoted strings in most files, semicolons.
