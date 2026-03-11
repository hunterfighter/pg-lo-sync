# pg-lo-sync

> PostgreSQL Large Object Sync for Restricted Managed Databases

## The Problem

PostgreSQL logical replication doesn't replicate `pg_largeobject` - and managed database users often lack direct system catalog access to fix it manually.

**This breaks:**
- Cloud migrations (on-prem → RDS)
- RDS-to-RDS replication (cross-region DR)
- Cross-cloud migrations (AWS → Azure → GCP)
- Application deployments relying on large objects

## The Solution

pg-lo-sync extracts large objects from source and recreates them on destination using **only allowed API functions** - no superuser required!

## Supported Scenarios

### Source (requires read access):
- ✅ Self-hosted PostgreSQL
- ✅ AWS RDS (master user)
- ✅ EC2 PostgreSQL
- ✅ Azure Database (admin user)
- ✅ Any PostgreSQL with pg_largeobject access

### Destination (works with restrictions):
- ✅ AWS RDS (even non-master users!)
- ✅ Azure Database for PostgreSQL
- ✅ Google Cloud SQL
- ✅ Heroku Postgres
- ✅ Any managed PostgreSQL

## How It Works
```
Source Database                      Destination Database
(Full access)                        (Restricted access)
┌──────────────┐                    ┌──────────────┐
│ Read from:   │                    │ Cannot read: │
│ pg_large     │     pg-lo-sync     │ pg_large     │
│ object       │  ══════════════>   │ object       │
│              │                    │              │
│ Extract data │                    │ BUT can use: │
│ via lo_get() │                    │ lo_create()  │
│              │                    │ lo_write()   │
└──────────────┘                    └──────────────┘
```

1. **Extract**: Read large objects from source using `lo_get()`
2. **Transport**: JSON/CSV with base64 or direct DB connection
3. **Recreate**: Use `lo_create()` and `lo_write()` on destination
   - Works even with restricted permissions!
   - No superuser required!

## Key Features

✅ **No superuser required** on destination
✅ **Works with RDS restrictions**
✅ **Supports multiple cloud providers**
✅ **Data integrity verification** (MD5)
✅ **Incremental sync** (only new/changed)
✅ **Batch processing** for large datasets

## Status

🚧 **Under Active Development**

Currently building MVP with Docker test environment.

## Quick Start (Coming Soon)
```bash
# Install
pip install pg-lo-sync

# Configure
pg-lo-sync init --source-db "host=source..." --dest-db "host=dest..."

# Sync
pg-lo-sync run

# Verify
pg-lo-sync verify
```

## Roadmap

- [x] Project scoped
- [x] Test environment created
- [ ] MVP extraction script
- [ ] MVP loading script
- [ ] Data verification
- [ ] Error handling
- [ ] CLI tool
- [ ] Documentation
- [ ] v1.0 release

## Contributing

Contributions welcome! This project is in early development.

## Use Cases

**Cloud Migration:**
```
On-prem PostgreSQL → AWS RDS
(Full access)      → (Restricted rds_admin)
```

**Cross-Region DR:**
```
RDS us-east-1 → RDS eu-west-1
(Master user) → (Application user)
```

**Multi-Cloud:**
```
AWS RDS → Azure Database
        → Google Cloud SQL
```

## License

MIT
