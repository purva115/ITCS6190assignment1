# ITCS 6190 Assignment 1: Docker Multi-Container Stack

## What the Stack Does

This project demonstrates a multi-container Docker application that analyzes trip data using PostgreSQL and Python. The system consists of a PostgreSQL database service that initializes with sample trip records and a Python application service that connects to the database, executes analytical queries, and generates statistical summaries. The application computes total trip counts, average fares by city, and identifies the longest trips by duration, outputting results both to the console and as a JSON file.

## Prerequisites

- Docker Desktop
- Docker Compose
- Git

## Project Structure

```
├── db/
│   ├── Dockerfile          # PostgreSQL container configuration
│   └── init.sql           # Database schema and seed data
├── app/
│   ├── Dockerfile          # Python application container
│   └── main.py            # Main application logic
├── out/                   # Output directory (auto-created)
├── compose.yml            # Multi-container orchestration
├── Makefile              # Build automation scripts
└── README.md             # Project documentation
```

## Commands to Run/Stop

### Start the Stack
```bash
# Recommended: Full clean build and run
make all

# Alternative: Direct Docker Compose
docker compose up --build
```

### Stop the Stack
```bash
# Using Makefile
make down

# Direct command
docker compose down -v
```

### Other Useful Commands
```bash
# Clean and reset output directory
make clean

# Build containers only
make build

# Start without rebuilding
make up
```

## Example Output

### Console Output
When you run `make all`, you should see output similar to this:

```
=== Summary ===
{
  "total_trips": 6,
  "avg_fare_by_city": [
    {"city": "Charlotte", "avg_fare": 16.25},
    {"city": "New York", "avg_fare": 19.0},
    {"city": "San Francisco", "avg_fare": 20.25}
  ],
  "top_by_minutes": [
    {"city": "San Francisco", "minutes": 28, "fare": 29.3},
    {"city": "New York", "minutes": 26, "fare": 27.1},
    {"city": "Charlotte", "minutes": 21, "fare": 20.0},
    {"city": "Charlotte", "minutes": 12, "fare": 12.5},
    {"city": "San Francisco", "minutes": 11, "fare": 11.2},
    {"city": "New York", "minutes": 9, "fare": 10.9}
  ]
}
```

### Expected Container Behavior
- Database container starts first and initializes with sample data
- Application waits for database to be ready (health check)
- Application connects, runs queries, and prints results
- Application exits successfully (exit code 0)
- Both containers stop gracefully

## Where Outputs Are Written

### Console Output
- JSON summary is printed directly to the terminal/console
- Database connection status and any error messages appear in logs

### File Output
- **`out/summary.json`** - Contains the complete statistical analysis in JSON format
- This file is created automatically in the `out/` directory
- The `out/` directory is mounted as a volume and persists after containers stop

### Viewing Output Files
```bash
# List output files
ls -la out/

# View JSON results
cat out/summary.json

# Open in editor
code out/summary.json
```

## Environment Variables

The application uses these environment variables (configured in `compose.yml`):

| Variable | Value | Description |
|----------|--------|-------------|
| `DB_HOST` | `db` | Database service name |
| `DB_PORT` | `5432` | PostgreSQL port |
| `DB_USER` | `appuser` | Database username |
| `DB_PASS` | `secretpw` | Database password |
| `DB_NAME` | `appdb` | Database name |
| `APP_TOP_N` | `10` | Number of top trips to retrieve |

## Troubleshooting

### Database Not Ready
**Symptom:** Application fails to connect or times out
```
Waiting for database...
Failed to connect to Postgres: connection refused
```

**Solutions:**
- Wait longer for database initialization (first run takes more time)
- Check database logs: `docker compose logs db`
- Verify database health: `docker compose ps`
- Restart the stack: `make down && make all`

### Permission Issues on `out/` Directory
**Symptom:** Cannot write to `/out/summary.json`
```
Permission denied: '/out/summary.json'
```

**Solutions:**
```bash
# Create and set permissions for output directory
mkdir -p out
chmod 755 out

# On Windows, ensure Docker has file sharing permissions
# Docker Desktop → Settings → Resources → File Sharing
```

### Port Conflicts
**Symptom:** Database fails to start due to port 5432 already in use
```
Error: port 5432 already allocated
```

**Solutions:**
```bash
# Check what's using the port
netstat -an | findstr :5432

# Stop conflicting services
# Or modify compose.yml to use different port:
ports:
  - "5433:5432"  # Use port 5433 instead
```

### Docker Compose Issues
**Symptom:** `docker compose` command not found
```bash
# Use older syntax
docker-compose up --build

# Or update Docker Desktop to latest version
```

### Makefile Issues
**Symptom:** `make: command not found` or tab/space errors

**Solutions:**
```bash
# On Windows, install make or use direct commands:
docker compose up --build
docker compose down -v

# For Makefile tab errors: ensure commands are indented with TABS, not spaces
```

### Container Build Failures
**Symptom:** Image build fails or times out

**Solutions:**
```bash
# Clean Docker cache
docker system prune -f

# Rebuild from scratch
docker compose build --no-cache

# Check Dockerfile syntax in db/ and app/ directories
```

### Application Exits Immediately
**Symptom:** `app-1 exited with code 1` or other non-zero exit code

**Solutions:**
```bash
# Check application logs
docker compose logs app

# Verify database is accessible
docker compose exec db psql -U appuser -d appdb -c "SELECT COUNT(*) FROM trips;"

# Check environment variables in compose.yml
```

## Development and Debugging

### View Container Logs
```bash
# All services
docker compose logs

# Specific service
docker compose logs db
docker compose logs app

# Follow logs in real-time
docker compose logs -f
```

### Interactive Container Access
```bash
# Access database directly
docker compose exec db psql -U appuser -d appdb

# Access application container
docker compose exec app bash

# Run SQL queries manually
docker compose exec db psql -U appuser -d appdb -c "SELECT * FROM trips;"
```

### Testing Individual Components
```bash
# Test database only
docker compose up db

# Test application with existing database
docker compose run app
```

## Technical Details

### Database Schema
```sql
CREATE TABLE trips (
    id SERIAL PRIMARY KEY,
    city TEXT NOT NULL,
    minutes INT NOT NULL,
    fare NUMERIC(6,2) NOT NULL
);
```

### Sample Data
- 6 trip records across 3 cities (Charlotte, New York, San Francisco)
- Trip durations from 9 to 28 minutes
- Fares ranging from $10.90 to $29.30

### Application Logic
1. **Connection Retry:** Waits up to 20 seconds for database availability
2. **Query Execution:** Runs three analytical queries
3. **Data Processing:** Computes statistics and formats results
4. **Output Generation:** Writes JSON to file and prints to console

## Assignment Completion Checklist

- [ ] Database service builds and starts successfully
- [ ] Application service connects to database
- [ ] All three queries execute without errors
- [ ] JSON output appears in console
- [ ] `out/summary.json` file is created
- [ ] Statistics are accurate (6 total trips, correct averages)
- [ ] `make all` command works end-to-end
- [ ] README.md documentation is complete
