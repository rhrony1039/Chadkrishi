# ChadKrishi 

**ChadKrishi** is a Bangladesh-focused rooftop and urban gardening web
application. It helps people manage plants, follow simple care tasks,
learn from weather-aware recommendations, connect with other gardeners,
and use a community marketplace.

The project is built with **React + Vite**, a **Go/Gin API**, and
**MySQL/MariaDB**.

## Contents

-   [Features](#features)
-   [Technology Stack](#technology-stack)
-   [Requirements](#requirements)
-   [Run Locally](#run-locally)
-   [Database Setup](#database-setup)
-   [Environment Configuration](#environment-configuration)
-   [Useful Commands](#useful-commands)
-   [Admin Access](#admin-access)
-   [Project Structure](#project-structure)
-   [Documentation](#documentation)
-   [Important Product Rules and
    Limitations](#important-product-rules-and-limitations)
-   [Security and Deployment Notes](#security-and-deployment-notes)

## Features

-   **Account and profile:** registration, login/session handling,
    profile information and profile photos.
-   **My Plants:** save and manage personal plants, planting details,
    observations, and private notes.
-   **Plant care:** daily observation-based care checklists and custom
    tasks.
-   **Weather and recommendations:** weather information and gardening
    suggestions based on available profile/catalog data.
-   **Gardening community:** share posts and photos, comment, and react
    with Like, Love, Angry, or Sad.
-   **Marketplace:** publish listings, reserve a complete listing, and
    track buying/selling history.
-   **Manual payment confirmation:** buyer and seller can record
    payment-status steps for an external payment.
-   **Notifications:** in-app notifications for supported community,
    marketplace, and messaging events.
-   **Private text chat:** conversations between users, message history,
    and read receipts.
-   **Admin archive:** archive sold marketplace listings while retaining
    transaction history.

## Technology Stack

  Layer        Technology
  ------------ -----------------------------------------------------
  Frontend     React 19, Vite 6, React Router, Bootstrap
  Backend      Go, Gin
  Database     MySQL 8 / MariaDB
  Testing      Vitest, Go test tools, API smoke-test script
  Deployment   Docker Compose and Caddy configuration are included

## Requirements

Install these tools before starting:

-   **Node.js 20.19 or newer** (Node.js 22 LTS is suitable) and npm
-   **Go 1.26 or newer**, as specified by this project
-   **MySQL 8 or MariaDB 10.4+**; XAMPP can be used for local
    development
-   Git (optional, for version control)
-   A modern browser

> The project has not been verified on every operating system or
> environment. Use compatible versions and check the linked setup guides
> if a command fails.

## Run Locally

### 1. Get the source

Extract the project ZIP, or clone the repository if you have its Git
URL. Open the `chadkrishi` project root in VS Code. The root folder
should contain `package.json`, `backend/`, and `migrations/`.

### 2. Configure the environment

The backend reads environment values from `.env` in its working
directory and from the project root. Create a `.env` file using the
example below. **Do not overwrite an existing `.env` without backing it
up, and do not commit passwords or secrets to Git.**

``` env
APP_ENV=development
PORT=8080
FRONTEND_ORIGIN=http://localhost:5173
DATABASE_DSN=root:YOUR_MYSQL_PASSWORD@tcp(127.0.0.1:3306)/chadkrishi?parseTime=true&charset=utf8mb4
SESSION_COOKIE_SECURE=false
WEATHER_CACHE_MINUTES=30

# Optional: set a persistent location for uploaded photos.
# UPLOAD_DIR=uploads
```

Update `DATABASE_DSN` with your own MySQL username, password, port, and
database name. If your local MySQL root account has no password, the DSN
format is
`root:@tcp(127.0.0.1:3306)/chadkrishi?parseTime=true&charset=utf8mb4`.

### 3. Create and prepare the database

Create a database named `chadkrishi` in MySQL/MariaDB, for example using
phpMyAdmin. Import the SQL migration files **once each and in order**,
from:

1.  `migrations/001_initial.sql`
2.  `migrations/002_seed_crops.sql`
3.  `migrations/003_auth_profile.sql`
4.  `migrations/004_verified_crop_catalog.sql`
5.  `migrations/005_expand_verified_container_crops.sql`
6.  `migrations/006_fix_bengali_crop_names.sql`
7.  `migrations/007_social_commerce.sql`
8.  `migrations/008_notifications.sql`

Do not rerun migrations blindly on an existing database. Back up the
database first and check which migrations have already been applied.

### 4. Start the backend

Open a terminal at the project root:

``` bash
cd backend
go run . -check-db
go run .
```

The first command checks the database connection and required commerce
schema. Run it successfully before starting the server. The API should
be available at:

-   API health check: <http://localhost:8080/api/v1/health>

If the notifications table is missing, the project also provides a
repair command:

``` bash
cd backend
go run . -repair-notifications
```

Use this only when needed; it is designed to create a missing
notifications table while preserving existing data. It does not
automatically fix every possible schema mismatch.

### 5. Start the frontend

Open a **second terminal** at the project root (the folder containing
`package.json`):

``` bash
npm ci
npm run dev
```

Open the local address printed by Vite, normally:

-   Frontend: <http://localhost:5173>

Keep both terminals running while using the application.

## Database Setup

-   The database stores accounts, plants, posts, marketplace records,
    messages, and other application data.
-   Uploaded photos are stored under `backend/uploads` by default,
    unless `UPLOAD_DIR` is configured.
-   Back up the database and uploaded-photo directory separately.
-   A fresh installation does not contain your old accounts or personal
    data; those are stored in your own database and upload directory.

## Useful Commands

Run these from the project root unless the command says otherwise.

### Frontend

``` bash
npm ci
npm run dev
npm test
npm run build
```

### Backend

``` bash
cd backend
go test ./...
go vet ./...
```

### API smoke test

The API smoke-test script is intended for a **running, isolated test
installation**:

``` bash
python scripts/api_smoke.py --test-only
```

Run it from the project root, and only after setting up a separate test
database and starting the API. The script creates test data. **Never run
smoke tests against a live database or real community.**

A browser smoke-test script is also included at
`scripts/browser_smoke.cjs`; browser automation may require separately
installed Playwright/Chromium. Review `docs/progress.md` for the
project's current validation notes.

## Admin Access

There is no default seeded administrator password. First, register a
normal account. Then, as the database owner, grant admin access to that
account by running the following SQL after replacing the example email
with the exact registered email:

``` sql
UPDATE users
SET is_admin = TRUE
WHERE email = 'your-actual-email@example.com';
```

Refresh the application after updating the account. Only perform this
operation on your own database and for an account you control.

## Project Structure

``` text
chadkrishi/
├── backend/             # Go/Gin API and backend tests
├── migrations/          # Ordered SQL database migrations
├── scripts/             # API and browser smoke-test scripts
├── deploy/              # Container build and Caddy configuration
├── docs/                # API, schema, setup, deployment and progress docs
├── src/                 # React frontend source
├── package.json         # Frontend dependencies and commands
├── compose.yaml          # Docker Compose deployment definition
└── README.md             # This guide
```

## Documentation

-   [Bangla start and recovery guide](START_HERE_BN.md)
-   [Bangla implementation and feature guide](docs/IMPLEMENTATION_BN.md)
-   [Deployment guide in Bangla](docs/DEPLOYMENT_BN.md)
-   [API documentation](docs/API.md)
-   [Database schema notes](docs/schema.md)
-   [Requirements](docs/requirements.md)
-   [Project progress and validation notes](docs/progress.md)

## Important Product Rules and Limitations

-   A marketplace **Buy** action reserves the complete listing at its
    advertised total price; partial purchases are not supported.
-   Payment is handled externally (for example, cash or bKash). The
    application does not include a payment gateway or automatic payment
    verification. A seller's confirmation is what marks a transaction as
    completed/sold.
-   The gardening checklist is based on user observations and general
    prompts. It is not sensor data, a plant-disease diagnosis, or
    guaranteed agronomic advice.
-   Weather information and recommendations depend on available external
    data and the project's limited crop catalog.
-   Private chat is text-only and is not an end-to-end encrypted
    messenger.
-   Admin archiving hides sold listings from the main feed while
    preserving transaction records.

## Security and Deployment Notes

This repository is suitable as a course project and local development
application; it should not be assumed to be hardened for a public
production marketplace.

Before public deployment:

-   Use HTTPS and production-safe secure session cookies.
-   Keep database credentials and secrets out of source control.
-   Configure persistent, backed-up database and upload storage.
-   Review `compose.yaml` and `docs/DEPLOYMENT_BN.md` before deploying.
-   Add appropriate rate limits, monitoring, moderation/reporting
    processes, backup recovery, and a payment/dispute policy.
-   Test on the exact deployment environment and keep evidence of test
    results.

## License

No license information was supplied with this project. Add a license
file if you intend to publish or redistribute the source.
