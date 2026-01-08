# Branchgres (BG) (Work in Progress)

**Git-like version control for PostgreSQL databases**

[![.NET 7.0](https://img.shields.io/badge/.NET-7.0-blue)](https://dotnet.microsoft.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview

Branchgres is a state version control system for PostgreSQL databases that brings Git-like semantics to database management. It captures database changes via triggers, stores them as immutable blobs, and enables powerful time-travel operations including snapshots, rollbacks, and Docker-based environment spin-ups.

## Features

- ✅ **Git-like Interface**: Familiar commands (init, commit, log, diff, status)
- 🔄 **Change Tracking**: Automatic capture of INSERT, UPDATE, DELETE, and DDL operations
- 📦 **Immutable Storage**: Content-addressable blob storage using SHA-256 hashing
- ⏱️ **Time Travel**: Snapshot and rollback to any commit point
- 🐳 **Docker Integration**: Spin up PostgreSQL containers seeded with historical state
- 📊 **Interactive CLI**: Rich terminal UI powered by Spectre.Console
- 📈 **Column History**: Export per-column history for granular auditing
- 🎯 **Schema Support**: Works with any PostgreSQL schema
- 🔁 **Auto-commit**: Background worker for continuous change ingestion

## Quick Start

### Installation

```bash
dotnet build
dotnet publish -c Release -o ./publish
```

### Basic Workflow

```bash
# Initialize repository
bg init

# Install database sidecar
bg install-sidecar --conn "Host=localhost;Database=mydb;Username=postgres;Password=pass"

# Or set connection string as environment variable
export BG_CONN="Host=localhost;Database=mydb;Username=postgres;Password=pass"

# Start worker (captures changes and auto-commits)
bg start

# Manual commit
bg commit -m "My commit message"

# View commit history
bg log

# Interactive browse
bg browse

# Create snapshot
bg snapshot abc123def --out snapshot.sql

# Spin up Docker container with historical state
bg spin-up abc123def --port 5433

# Rollback to previous state
bg rollback abc123def --dry-run false
```

## Architecture

### Components

```
┌─────────────────────────────────────────────────────────┐
│                     Branchgres CLI                       │
├─────────────────────────────────────────────────────────┤
│  Services Layer                                          │
│  ├── IRepositoryService    (repo management)            │
│  ├── ICommitService        (commit operations)          │
│  ├── IBlobService          (blob storage)               │
│  ├── IDatabaseService      (PostgreSQL integration)     │
│  ├── ISnapshotService      (snapshot/rollback)          │
│  ├── IWorkerService        (background ingestion)       │
│  ├── IDockerService        (container management)       │
│  └── IExportService        (data export)                │
├─────────────────────────────────────────────────────────┤
│  Storage Layer (.bg directory)                          │
│  ├── commits.jsonl         (commit history)            │
│  ├── objects/              (immutable blobs)            │
│  ├── pending_blobs         (uncommitted changes)        │
│  └── last_change           (worker checkpoint)          │
└─────────────────────────────────────────────────────────┘
```

### How It Works

1. **Installation**: `bg install-sidecar` creates trigger functions in your PostgreSQL database
2. **Capture**: Database changes trigger storage in `pg_git_sidecar` table
3. **Ingestion**: Worker polls sidecar table and stores changes as immutable blobs
4. **Commit**: Blobs are grouped into commits with SHA-256 hashes
5. **Time Travel**: Snapshots replay blobs to reconstruct historical state

## Commands

| Command | Description |
|---------|-------------|
| `init` | Initialize local repository metadata |
| `install-sidecar` | Install triggers into database |
| `install-sidecar-single <table>` | Install trigger on single table |
| `start` | Run background worker (auto-commit mode) |
| `commit -m "message"` | Create manual commit |
| `log` | List commit history |
| `status` | Show repository status |
| `show <hash>` | Show commit details |
| `diff <hash>` | Show changes in commit |
| `browse` | Interactive commit browser |
| `snapshot <hash> -o file.sql` | Create SQL snapshot |
| `rollback <hash>` | Rollback database to commit |
| `spin-up <hash>` | Start Docker container with snapshot |
| `export-table <table> <id>` | Export row with column history |

## Configuration

Configuration can be provided via `appsettings.json` or environment variables:

```json
{
  "Branchgres": {
    "ConnectionString": "Host=localhost;Database=mydb;...",
    "DefaultSchema": "public",
    "PollIntervalMs": 2000,
    "BatchSize": 1000,
    "EnableAutoCommit": true,
    "DockerImage": "postgres:15-alpine",
    "DockerPort": 5433,
    "CommandTimeoutSeconds": 30
  }
}
```

Environment variables (override config file):
- `BG_CONN`: PostgreSQL connection string

## Development

### Prerequisites

- .NET 7.0 SDK
- PostgreSQL 12+
- Docker (for spin-up features)

### Building

```bash
dotnet restore
dotnet build
```

### Testing

```bash
dotnet test
```

## Use Cases

- **Database Auditing**: Track who changed what and when
- **Development Environments**: Quickly spin up databases at specific points in time
- **Rollback Capability**: Safely revert problematic changes
- **Data Migration**: Test migrations with easy rollback
- **Compliance**: Maintain complete change history for regulatory requirements
- **Debugging**: Reproduce bugs by restoring exact database states

## Performance Considerations

- Blob storage is optimized with content-addressable hashing (deduplication)
- Worker uses configurable polling intervals to balance latency vs load
- Batch processing for efficient change ingestion
- Indexed commit lookups support partial hash matching

## Security

- Parameterized queries throughout to prevent SQL injection
- Connection strings support PostgreSQL SSL/TLS
- Blobs stored with SHA-256 integrity checking
- Supports PostgreSQL role-based access control

## Limitations

- Currently optimized for single-schema tracking
- Large binary columns may impact blob storage size
- Requires PostgreSQL 12+ for trigger functionality
- Docker features require Docker daemon access

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## License

MIT License - see LICENSE file for details

## Acknowledgments

- Built with [Spectre.Console](https://spectreconsole.net/) for rich CLI
- Uses [System.CommandLine](https://github.com/dotnet/command-line-api) for command parsing
- PostgreSQL integration via [Npgsql](https://www.npgsql.org/)

## Support

- Issues: [GitHub Issues](https://github.com/yourusername/branchgres/issues)
- Discussions: [GitHub Discussions](https://github.com/yourusername/branchgres/discussions)

---

**Made with ❤️ for database version control**



