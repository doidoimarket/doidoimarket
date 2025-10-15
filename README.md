# Doidoi Market Monorepo

This repository hosts a complete e-commerce marketplace platform composed of a Django backend and a React frontend.  The goal of this README is to give maintainers a quick orientation of the codebase, outline the development environment, and document the recommended workflows for local development and production deployment.

## Repository layout

```
.
├── backend/                  # Django project with REST API, Celery tasks, and management commands
├── docker/                   # Dockerfiles and helper scripts for production images
├── docker-compose.yml        # Production docker stack (PostgreSQL, Redis, backend, Celery, nginx, certbot)
├── docker-compose.dev.yml    # Local development docker stack
├── example_env               # Template for backend environment variables
├── frontend/                 # React application (Create React App + Redux Toolkit)
├── init-letsencrypt.sh       # Helper script to issue or renew Let's Encrypt certificates
└── readme.md                 # Legacy deployment checklist (kept for historical reference)
```

> **Heads-up**
> The original repository contents were published as an archive.  The extracted directory now sits under `marketplace-master/` at the repository root so that Git can track it directly, which avoids the need to unzip the project before making changes.

## Backend overview

* Django project lives under `backend/src`.  It exposes REST endpoints under `/api/v1/` using Django REST Framework (DRF) with JWT authentication powered by Djoser.
* PostgreSQL is the primary database; Redis backs Celery for async tasks (notifications, shop management) and caching.
* Static and media files are uploaded to AWS S3 via the custom `PublicMediaStorage` backend configured in `settings/components/storages.py`.
* Key apps:
  * `accounts` — extended user profile, shops, orders, fulfilment logic, and Celery task triggers.
  * `products` — category tree (MPTT), product specifications/attributes, media, promotions.
  * `store_settings` — site-wide marketing banners, legal documents, and contact information (via `SingletonModel`).

## Frontend overview

* Built with Create React App (customized via `react-app-rewired`) and Redux Toolkit for state management.
* Environment variable `REACT_APP_API_URL` controls the backend API endpoint (see `frontend/example_env`).
* Component highlights:
  * `src/views/pages/main-page` — main landing page, categories drawer, banners, and recommendation carousels.
  * `src/views/pages/products` — catalog browsing, product detail views, reviews, and related products.
  * `src/navigation/vertical` and `src/navigation/horizontal` — menu definitions for the admin UI shell.

## Local development

1. Install prerequisites: Docker, Docker Compose, and Node.js (>= 16) if you plan to run the frontend outside Docker.
2. Copy environment templates:
   ```bash
   cp example_env .env
   cp frontend/example_env frontend/.env
   ```
   Update the variables (database credentials, S3 keys, SMTP, etc.) to match your environment.
3. Start the backend stack:
   ```bash
   docker compose -f docker-compose.dev.yml up --build
   ```
   The backend is exposed on `http://localhost:8000`, while pgAdmin is available on `http://localhost:5050` when enabled in the compose file.
4. Run frontend in development mode for hot reload:
   ```bash
   cd frontend
   yarn install
   yarn start
   ```
   Configure `REACT_APP_API_URL` to point to `http://localhost:8000` for local API access.

## Production build & deployment

1. Build the frontend bundle:
   ```bash
   cd frontend
   yarn install
   yarn build
   ```
2. Populate `.env` in the repository root with production secrets (see `example_env`).
3. Ensure `init-letsencrypt.sh` is executable (`chmod +x init-letsencrypt.sh`).
4. Launch the stack:
   ```bash
   docker compose up -d --build
   ```
5. If HTTPS certificates need to be (re)issued, run `./init-letsencrypt.sh` after the stack is up.

## Useful management commands

* Apply database migrations: `docker compose run --rm backend python manage.py migrate`
* Create an admin user: `docker compose run --rm backend python manage.py createsuperuser`
* Collect static files: `docker compose run --rm backend python manage.py collectstatic`

## Testing & quality

* Backend tests: `docker compose run --rm backend pytest`
* Frontend tests: `cd frontend && yarn test`
* Linting: `cd frontend && yarn lint` (ESLint rules defined via CRA configuration)

## Contributing

1. Create a feature branch from `develop` (or `main`, depending on workflow).
2. Make your changes and ensure tests pass.
3. Submit a pull request summarising the change.  Include deployment considerations when relevant.

---

For deployment-specific notes kept from the original maintainers, refer to [`readme.md`](marketplace-master/readme.md).
