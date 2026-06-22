# AWIP - AI Bench Management Platform

A full-stack application for managing employee bench/allocation and project requirements using skill matrices.

## Architecture

- **Client**: React + TypeScript + Vite + MUI
- **Server**: ASP.NET Core + JWT + PostgreSQL + Entity Framework Core

## Folder Structure

```
awip/
├── client/          # React SPA
├── server/          # ASP.NET Core API
├── docs/            # Design docs & schema
└── README.md
```

## Development Setup

### Prerequisites
- Node.js 18+ / npm or yarn
- .NET 8 SDK
- PostgreSQL 14+
- Git

### Quick Start

1. **Clone the repo**
   ```bash
   git clone https://github.com/yourorg/awip.git
   cd awip
   ```

2. **Client Setup** (Day 2 — AWIP-4)
   ```bash
   cd client
   npm install
   npm run dev
   ```

3. **Server Setup** (Day 2 — AWIP-6)
   ```bash
   cd server
   dotnet restore
   dotnet ef database update
   dotnet run
   ```

## Sprint 1 Tickets

| Ticket | Owner | Date | Status |
|--------|-------|------|--------|
| AWIP-3 | DS | Jun 22 | In Progress |
| AWIP-4 | DS | Jun 23 | Pending |
| AWIP-6 | DS | Jun 23 | Pending |
| AWIP-7 | HK | Jun 24 | Pending |
| AWIP-8 | HK | Jun 25 | Pending |
| AWIP-9 | DS | Jun 24 | Pending |
| AWIP-10 | HK | Jun 25 | Pending |
| AWIP-12 | DS | Jun 26 | Pending |

## Database Connection

PostgreSQL connection string in `server/appsettings.json`:
```json
"ConnectionStrings": {
  "DefaultConnection": "Host=localhost;Port=5432;Database=awip_db;Username=postgres;Password=yourpassword"
}
```

## More Info

See [docs/schema.md](docs/schema.md) for database schema details.
