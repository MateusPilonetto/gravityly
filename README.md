# GravityX

GravityX is a full-stack social networking application built with **Vue 3** and **Laravel**. It provides a modern, responsive interface for sharing posts, interacting with other users, discovering profiles, and publishing temporary photo or video stories.

The project is organized as a monorepo containing a Vue single-page application and a Laravel REST API.

## Features

- User registration and authentication with Laravel Sanctum bearer tokens
- User profiles with editable name, username, bio, and profile photo
- Follow and unfollow other users
- User search and profile suggestions
- Create text and image posts
- Like and unlike posts
- Comment on posts
- Delete your own posts
- Photo and video stories
- Stories automatically expire after 24 hours
- Automatic story playback and progress indication
- Responsive layouts for desktop, tablet, and mobile devices
- Progressive Web App (PWA) support with offline capabilities
- Docker-based development environment
- SQLite support for local development
- PostgreSQL-ready production configuration

## Tech Stack

### Frontend

- Vue 3
- Vue Router
- Vite
- JavaScript
- CSS
- PWA / Service Worker
- Nginx for the Docker production build

### Backend

- PHP 8.3+
- Laravel 13
- Laravel Sanctum
- REST API
- SQLite for local development
- PostgreSQL support for production

### Infrastructure

- Docker
- Docker Compose
- Render-compatible deployment configuration

## Project Structure

```text
GravityX/
├── gravityX-frontend/     # Vue 3 + Vite frontend
├── gravityX-api/          # Laravel REST API
├── compose.yaml           # Local Docker environment
├── .dockerignore
└── README.md
```

## Getting Started with Docker

Using Docker is the recommended way to run the complete project locally.

### Requirements

- Docker
- Docker Compose plugin

From the repository root, run:

```bash
docker compose up -d --build
```

After the containers become healthy, the application will be available at:

- **Frontend:** `http://localhost:8080`
- **API:** `http://localhost:8000`

The Docker environment uses SQLite stored in a named volume. On the first startup, it automatically:

- creates the application key when necessary;
- creates the SQLite database;
- runs Laravel migrations;
- creates the public storage link;
- starts the API and frontend services.

You do not need to create `.env` files when using the provided Docker Compose configuration.

### Useful Docker Commands

Check the running containers:

```bash
docker compose ps
```

Follow the logs:

```bash
docker compose logs -f
```

Stop the project:

```bash
docker compose down
```

To remove the containers **and all local Docker data**, including the SQLite and storage volumes, run:

```bash
docker compose down -v
```

> **Warning:** `docker compose down -v` permanently deletes the local GravityX data stored in Docker volumes.

## Running without Docker

You can also run the backend and frontend directly on your machine.

### Requirements

- PHP 8.3 or newer
- Composer
- Node.js 22.18+ or 24.12+
- npm
- SQLite PHP extension

### 1. Start the Laravel API

Open a terminal in the repository root and run:

```bash
cd gravityX-api
cp .env.example .env
touch database/database.sqlite
composer install
php artisan key:generate
php artisan migrate
php artisan storage:link
php artisan serve
```

The API will be available at:

```text
http://localhost:8000
```

### 2. Start the Vue Frontend

Open a second terminal and run:

```bash
cd gravityX-frontend
cp .env.example .env
npm ci
npm run dev
```

The development frontend will normally be available at:

```text
http://localhost:5173
```

By default, the frontend connects to:

```text
http://localhost:8000/api
```

## Environment Variables

### Frontend

The frontend uses `VITE_API_URL` to locate the Laravel API.

Example:

```env
VITE_API_URL=http://localhost:8000/api
```

Do not add a trailing slash.

### Backend

For local development, copy the example environment file:

```bash
cd gravityX-api
cp .env.example .env
```

Important variables include:

```env
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000
FRONTEND_URL=http://localhost:5173
DB_CONNECTION=sqlite
```

Generate an application key with:

```bash
php artisan key:generate
```

## Frontend Commands

Run the development server:

```bash
cd gravityX-frontend
npm run dev
```

Create a production build:

```bash
npm run build
```

Preview the production build locally:

```bash
npm run preview
```

The production files are generated inside `gravityX-frontend/dist`.

## Backend Commands

Run database migrations:

```bash
cd gravityX-api
php artisan migrate
```

Reset and recreate the local database:

```bash
php artisan migrate:fresh
```

Create the public storage link:

```bash
php artisan storage:link
```

Start the Laravel development server:

```bash
php artisan serve
```

## Stories

GravityX supports image and video stories.

Accepted formats include:

- JPEG
- PNG
- WebP
- MP4

The maximum story upload size is **10 MB**. Stories expire automatically after **24 hours** and expired story records/media are pruned by Laravel's scheduled model pruning.

When running the production backend, make sure the Laravel scheduler remains active so expired media can be cleaned up.

## Production Deployment on Render

GravityX can be deployed with the frontend and backend as separate Render services.

### Backend

Use `gravityX-api/Dockerfile` with the **repository root** as the Docker build context.

At minimum, configure:

```env
APP_ENV=production
APP_DEBUG=false
APP_KEY=<your-generated-key>
APP_URL=https://<your-api-domain>
FRONTEND_URL=https://<your-frontend-domain>
DB_CONNECTION=pgsql
DATABASE_URL=<your-render-postgresql-url>
```

Generate an application key once with:

```bash
php artisan key:generate --show
```

Store that value securely as the `APP_KEY` environment variable in Render.

### Frontend

Deploy `gravityX-frontend` as a static site.

Use the following build command:

```bash
npm ci && npm run build
```

Set the publish directory to:

```text
dist
```

Configure the API URL during the frontend build:

```env
VITE_API_URL=https://<your-api-domain>/api
```

If `VITE_API_URL` is not configured in production, the frontend falls back to the local API address.

## API Overview

The API includes authenticated endpoints for:

- profile management;
- user search and suggestions;
- following and unfollowing users;
- posts;
- likes;
- comments;
- stories.

Authentication is handled using bearer tokens issued by Laravel Sanctum.

## Responsive Design

GravityX is designed for desktop and mobile use. The interface includes responsive profiles, feed cards, forms, search results, navigation, post details, and a full-screen mobile story viewer with safe-area support for modern devices.

## PWA Support

The frontend includes a web app manifest and service worker generation during production builds, allowing supported browsers to install GravityX as a Progressive Web App and provide basic offline behavior.

To generate the service worker, run the normal production build:

```bash
npm run build
```

## Security Notes

- Never commit `.env` files or production secrets.
- Keep `APP_DEBUG=false` in production.
- Use HTTPS for both the frontend and API in production.
- Store `APP_KEY`, database credentials, and other secrets using your hosting provider's environment variable system.

## Contributing

When making changes, create a separate branch and verify that the frontend builds successfully before merging:

```bash
git checkout -b feature/your-change
cd gravityX-frontend
npm ci
npm run build
```

Then review your changes and commit them normally.
