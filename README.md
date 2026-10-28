# StockPilot — Inventory Management System

[![.NET](https://img.shields.io/badge/.NET-10.0-512BD4?logo=dotnet)](https://dotnet.microsoft.com/)
[![Angular](https://img.shields.io/badge/Angular-20-DD0031?logo=angular)](https://angular.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**StockPilot** is a full-stack Inventory Management System with a .NET 10 ASP.NET Core Web API backend, a PostgreSQL database, and an Angular 20 frontend built with Angular Material. It tracks products, categories, suppliers, purchases, sales, and stock levels, with JWT-based authentication, role-based access, and PDF report generation.

**StockPilot branding and README customization:** [Akshata Bhoje](https://github.com/akshatab15)

This repository is a rebranding and documentation customization of an existing Inventory Management System codebase, presented as **StockPilot — Inventory Management System**. The application codebase is preserved as-is; no application features were removed or rewritten, and no original code is claimed as my own. See [Maintainer](#-maintainer) below.

## 📑 Table of Contents

- [✨ Features](#-features)
- [🛠️ Tech Stack](#%EF%B8%8F-tech-stack)
- [🏗️ Architecture](#%EF%B8%8F-architecture)
- [🔐 Environment Variables](#-environment-variables)
- [🚀 Quick Start with Docker](#-quick-start-with-docker)
- [💻 Manual Development Setup](#-manual-development-setup)
- [🔧 Entity Framework Migrations](#-entity-framework-migrations)
- [💡 Suggested 30-Minute Feature](#-suggested-30-minute-feature)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)
- [👤 Maintainer](#-maintainer)

## ✨ Features

The following features are implemented and verified in the source code:

- **📦 Product Management** — Create, read, update, and delete products with SKU, price, and stock information; includes search, pagination, and sorting.
- **🗂️ Category Management** — Organize products into categories.
- **🏢 Supplier Management** — Maintain supplier information.
- **🛒 Purchase Management** — Record purchase orders with product, quantity, price, dates, PO/invoice numbers; PDF report export with filters (product, date range, sort).
- **💰 Sales Management** — Record sales with automatic stock validation (cannot sell more than available stock); PDF report export with filters.
- **📊 Stock Tracking** — View current stock levels per product with search, pagination, and sorting.
- **👤 User Authentication** — JWT-based login/registration, role-based access control (Admin), access token stored in an `accessToken` cookie, "me" profile endpoint, and refresh-token storage.
- **📄 PDF Reports** — Professionally formatted A4 landscape PDF reports for sales and purchases (QuestPDF), triggered via the `Accept: application/pdf` header.
- **🎨 Modern UI** — Responsive Angular Material interface with sidebar navigation, protected routes, and login guard.
- **⚡ State Management** — NgRx Component Store for feature state, with Angular Signals for reactive UI state.
- **🗄️ Database** — PostgreSQL 16; migrations and seed data (admin user, categories, suppliers, products) applied automatically on application startup.

## 🛠️ Tech Stack

Only technologies confirmed in the source code are listed below.

| Layer | Technology | Version / Source |
|-------|-----------|------------------|
| Backend | ASP.NET Core Web API | .NET 10 (`net10.0`) |
| Database | PostgreSQL | 16 (`postgres:16.9-bullseye` image) |
| Data access | Dapper | 2.1.66 (primary repository layer) |
| Migrations | Entity Framework Core + Npgsql | 9.0.x (`EnsureCreatedAsync`/`MigrateAsync` on startup) |
| Authentication | ASP.NET Core JWT Bearer | 10.0.0 |
| Password hashing | BCrypt.Net-Next | 4.0.3 |
| PDF generation | QuestPDF | 2025.7.4 (Community license) |
| Frontend | Angular | 20.1.x |
| UI components | Angular Material | 20.1.x |
| State management | NgRx Component Store | 20.0.0 |
| Reactive state | Angular Signals + RxJS | Signals; RxJS ~7.8.0 |
| Language | TypeScript | ~5.8.3 |
| Production web server | Nginx | `nginxinc/nginx-unprivileged` |
| Containerization | Docker / Docker Compose | `compose.yml` in `src/` |

## 🏗️ Architecture

```
StockPilot
├── src/compose.yml              # Docker Compose: backend, frontend, PostgreSQL
├── src/backend/                 # .NET 10 solution
│   ├── InventoryMgt.Api/        # ASP.NET Core Web API (Controllers, Middleware, Services)
│   ├── InventoryMgt.Data/       # Dapper repositories, EF Core DbContext, Migrations, seed data
│   └── InventoryMgt.Shared/     # DTOs, contracts, shared exceptions, shared service interfaces
└── src/frontend/                # Angular 20 application
    └── src/app/
        ├── auth/                # Login, auth store (signals), auth service, guards
        ├── products/            # Product CRUD, NgRx Component Store, service
        ├── category/            # Category CRUD, Component Store
        ├── suppliers/           # Supplier CRUD, Component Store (signals)
        ├── purchases/           # Purchase CRUD + PDF export, Component Store
        ├── sales/               # Sales CRUD + stock validation + PDF export, Component Store
        ├── stocks/              # Stock levels, Component Store
        ├── shared/              # Models, error handler, HTTP interceptors
        ├── dashboard/           # Dashboard view
        └── app.routes.ts        # Route definitions (all non-login routes are auth-guarded)
```

**Request flow:** Angular frontend → ASP.NET Core API (JWT authentication + authorization) → Dapper repositories → PostgreSQL. EF Core is used only for schema migrations and seeding at startup. PDF reports are generated server-side with QuestPDF and streamed back to the browser.

## 🔐 Environment Variables

### Backend (`InventoryMgt.Api`)

| Variable | Example | Description |
|----------|---------|-------------|
| `ConnectionStrings__default` | `Host=localhost;Port=5432;Database=inventory_mgt;Username=postgres;Password=p@55w0rd` | PostgreSQL connection string (ASP.NET Core configuration key format) |
| `ASPNETCORE_ENVIRONMENT` | `Production` or `Development` | ASP.NET Core environment |
| `ASPNETCORE_URLS` | `http://+:8080` | URLs the Kestrel server listens on |
| `JWT__Secret` | `your-32-characters-long-super-strong-jwt-secret-key` | JWT signing secret |
| `JWT__ValidIssuer` | `http://localhost:5131` | JWT issuer |
| `JWT__ValidAudience` | `http://localhost:5131` | JWT audience |

These map to `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "default": "Host=localhost;Port=5432;Database=inventory_mgt;Username=postgres;Password=p@55w0rd"
  },
  "JWT": {
    "ValidAudience": "http://localhost:5131",
    "ValidIssuer": "http://localhost:5131",
    "Secret": "your-32-characters-long-super-strong-jwt-secret-key"
  }
}
```

Docker Compose overrides the connection string via `ConnectionStrings__default=Host=db;Database=inventorydb;Username=postgres;Password=p@55w0rd!`.

### Frontend

| File | Variable | Development value | Production value |
|------|----------|-------------------|------------------|
| `src/environments/environment.development.ts` | `API_BASE_URL` | `https://localhost:7015/api` | — |
| `src/environments/environment.ts` | `API_BASE_URL` | — | `http://localhost:8080/api` |

## 🚀 Quick Start with Docker

Docker Compose is the easiest way to run the full stack (backend, frontend, PostgreSQL).

1. Navigate to the `src` directory:
   ```bash
   cd src
   ```
2. Start the containers:
   ```bash
   docker compose up -d
   ```
3. Open your browser: `http://localhost:3001/`

The backend API listens on `http://localhost:8080` and PostgreSQL on `http://localhost:5432`. The database is migrated and seeded automatically on first startup.

Default credentials:
```
Username: admin
Password: Admin@123
```

## 💻 Manual Development Setup

### Prerequisites

- .NET 10.0 SDK
- Node.js (LTS)
- Angular CLI (`npm install -g @angular/cli`)
- PostgreSQL 16

### 1. Clone the repository

```bash
git clone <your-fork-url>
cd StockPilot
```

### 2. Database

Create a PostgreSQL database (default name `inventory_mgt`, user `postgres`, password `p@55w0rd`) and update `src/backend/InventoryMgt.Api/appsettings.json` if your credentials differ.

### 3. Backend

```bash
cd src/backend/InventoryMgt.Api
dotnet restore
dotnet run
```

The API runs on `http://localhost:5131` (Kestrel default) unless `ASPNETCORE_URLS` is overridden. The database is created/migrated and seeded automatically on startup.

### 4. Frontend

Open a new terminal:

```bash
cd src/frontend
npm install
ng serve --open
```

The Angular development server runs on `http://localhost:4200` and calls the API at `https://localhost:7015/api` (development environment). Log in with `admin` / `Admin@123`.

## 🔧 Entity Framework Migrations

EF Core is used strictly for schema migrations (data access is via Dapper). Migrations live in `src/backend/InventoryMgt.Data/Migrations` and are applied automatically at runtime.

If you need to manage migrations manually, run from the `src/backend` directory:

**Add a migration:**
```bash
dotnet ef migrations add YourMigrationName --project InventoryMgt.Data --startup-project InventoryMgt.Api
```

**Update the database:**
```bash
dotnet ef database update --project InventoryMgt.Data --startup-project InventoryMgt.Api
```

## 💡 Suggested 30-Minute Feature

**Low Stock Alert** (not yet implemented — safe to add without breaking changes)

The Stocks page already displays current stock levels per product. A useful, low-effort addition is a low-stock indicator:

1. In `src/frontend/src/app/stocks/stock.component.ts`, add a threshold constant (e.g., `LOW_STOCK_THRESHOLD = 10`) and compute a count of stocks below it.
2. Display a badge/alert banner on the Stocks page showing the low-stock count, with a link/filter to view only those items.
3. Optionally add a "Low stock" CSS highlight to rows where `quantity <= threshold`.

This is frontend-only (no API or database change required) because stock quantities are already returned by the existing `/api/stock` endpoint. Clearly mark it as a new addition if you implement it.

## 🤝 Contributing

Contributions are welcome! If you'd like to contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👤 Maintainer

**Akshata Bhoje** — StockPilot project customization and maintenance.

GitHub: https://github.com/akshatab15
