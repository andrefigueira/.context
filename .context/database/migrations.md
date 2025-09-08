# Database: Migration Strategy

This document defines the database migration strategy for [Your Project Name], including migration tooling, versioning, rollback procedures, and deployment workflows.

## Migration System Overview

### Migration Philosophy
- **Forward-only migrations**: Prefer additive changes over destructive ones
- **Atomic operations**: Each migration should be a single atomic transaction
- **Reversible changes**: All migrations must include rollback procedures
- **Data preservation**: Never lose data during schema changes
- **Zero-downtime**: Design migrations for production deployments with minimal downtime

### Migration Structure
```
migrations/
├── 000001_create_users_table.up.sql
├── 000001_create_users_table.down.sql
├── 000002_add_user_profiles_table.up.sql
├── 000002_add_user_profiles_table.down.sql
├── 000003_create_auth_tables.up.sql
├── 000003_create_auth_tables.down.sql
├── 000004_add_audit_logging.up.sql
├── 000004_add_audit_logging.down.sql
└── ...
```

## Migration Tool Implementation

### Custom Migration Runner
```go
type Migration struct {
    Version     int
    Name        string
    UpSQL       string
    DownSQL     string
    Checksum    string
    AppliedAt   *time.Time
}

type Migrator struct {
    db          *sql.DB
    logger      Logger
    migrationsPath string
    lockTimeout time.Duration
}

func NewMigrator(db *sql.DB, migrationsPath string, logger Logger) *Migrator {
    return &Migrator{
        db:             db,
        logger:         logger,
        migrationsPath: migrationsPath,
        lockTimeout:    5 * time.Minute,
    }
}

func (m *Migrator) Migrate() error {
    // Acquire advisory lock to prevent concurrent migrations
    if err := m.acquireLock(); err != nil {
        return fmt.Errorf("acquire migration lock: %w", err)
    }
    defer m.releaseLock()
    
    // Ensure schema_migrations table exists
    if err := m.ensureMigrationsTable(); err != nil {
        return fmt.Errorf("ensure migrations table: %w", err)
    }
    
    // Load migrations from filesystem
    migrations, err := m.loadMigrations()
    if err != nil {
        return fmt.Errorf("load migrations: %w", err)
    }
    
    // Get applied migrations from database
    applied, err := m.getAppliedMigrations()
    if err != nil {
        return fmt.Errorf("get applied migrations: %w", err)
    }
    
    // Find pending migrations
    pending := m.findPendingMigrations(migrations, applied)
    
    if len(pending) == 0 {
        m.logger.Info("No pending migrations")
        return nil
    }
    
    // Apply pending migrations
    for _, migration := range pending {
        if err := m.applyMigration(migration); err != nil {
            return fmt.Errorf("apply migration %d_%s: %w", migration.Version, migration.Name, err)
        }
        m.logger.Info("Applied migration", "version", migration.Version, "name", migration.Name)
    }
    
    return nil
}

func (m *Migrator) applyMigration(migration *Migration) error {
    tx, err := m.db.Begin()
    if err != nil {
        return fmt.Errorf("begin transaction: %w", err)
    }
    
    defer func() {
        if err != nil {
            tx.Rollback()
        }
    }()
    
    // Execute migration SQL
    if _, err = tx.Exec(migration.UpSQL); err != nil {
        return fmt.Errorf("execute migration SQL: %w", err)
    }
    
    // Record migration as applied
    now := time.Now()
    migration.AppliedAt = &now
    
    _, err = tx.Exec(`
        INSERT INTO schema_migrations (version, name, checksum, applied_at)
        VALUES ($1, $2, $3, $4)
    `, migration.Version, migration.Name, migration.Checksum, migration.AppliedAt)
    
    if err != nil {
        return fmt.Errorf("record migration: %w", err)
    }
    
    return tx.Commit()
}

func (m *Migrator) Rollback(targetVersion int) error {
    // Acquire lock
    if err := m.acquireLock(); err != nil {
        return fmt.Errorf("acquire migration lock: %w", err)
    }
    defer m.releaseLock()
    
    // Get applied migrations in reverse order
    applied, err := m.getAppliedMigrationsDesc()
    if err != nil {
        return fmt.Errorf("get applied migrations: %w", err)
    }
    
    // Load migration files
    migrations, err := m.loadMigrations()
    if err != nil {
        return fmt.Errorf("load migrations: %w", err)
    }
    
    migrationMap := make(map[int]*Migration)
    for _, migration := range migrations {
        migrationMap[migration.Version] = migration
    }
    
    // Roll back migrations greater than target version
    for _, appliedMigration := range applied {
        if appliedMigration.Version <= targetVersion {
            break
        }
        
        migration, exists := migrationMap[appliedMigration.Version]
        if !exists {
            return fmt.Errorf("migration file not found for version %d", appliedMigration.Version)
        }
        
        if err := m.rollbackMigration(migration); err != nil {
            return fmt.Errorf("rollback migration %d: %w", migration.Version, err)
        }
        
        m.logger.Info("Rolled back migration", "version", migration.Version, "name", migration.Name)
    }
    
    return nil
}

func (m *Migrator) rollbackMigration(migration *Migration) error {
    tx, err := m.db.Begin()
    if err != nil {
        return fmt.Errorf("begin transaction: %w", err)
    }
    
    defer func() {
        if err != nil {
            tx.Rollback()
        }
    }()
    
    // Execute rollback SQL
    if _, err = tx.Exec(migration.DownSQL); err != nil {
        return fmt.Errorf("execute rollback SQL: %w", err)
    }
    
    // Remove migration record
    _, err = tx.Exec(`DELETE FROM schema_migrations WHERE version = $1`, migration.Version)
    if err != nil {
        return fmt.Errorf("remove migration record: %w", err)
    }
    
    return tx.Commit()
}
```

### Schema Migrations Table
```sql
CREATE TABLE IF NOT EXISTS schema_migrations (
    version INTEGER PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    checksum VARCHAR(64) NOT NULL,
    applied_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Index for performance
CREATE INDEX IF NOT EXISTS idx_schema_migrations_applied_at 
    ON schema_migrations(applied_at);
```

## Migration Examples

### 000001: Create Users Table
**Up Migration** (`000001_create_users_table.up.sql`):
```sql
-- Create users table with all required fields
CREATE TABLE app.users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'active' 
        CHECK (status IN ('active', 'inactive', 'suspended')),
    email_verified BOOLEAN NOT NULL DEFAULT FALSE,
    email_verified_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

-- Create unique index for email (excluding soft deleted users)
CREATE UNIQUE INDEX idx_users_email_unique 
    ON app.users(email) WHERE deleted_at IS NULL;

-- Create indexes for common queries
CREATE INDEX idx_users_status ON app.users(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_created_at ON app.users(created_at);

-- Create updated_at trigger
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_users_updated_at 
    BEFORE UPDATE ON app.users 
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Add email validation constraint
ALTER TABLE app.users ADD CONSTRAINT users_email_valid 
    CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

**Down Migration** (`000001_create_users_table.down.sql`):
```sql
-- Drop trigger and function
DROP TRIGGER IF EXISTS update_users_updated_at ON app.users;
DROP FUNCTION IF EXISTS update_updated_at_column();

-- Drop indexes
DROP INDEX IF EXISTS idx_users_email_unique;
DROP INDEX IF EXISTS idx_users_status;
DROP INDEX IF EXISTS idx_users_created_at;

-- Drop table
DROP TABLE IF EXISTS app.users;
```

### 000002: Add User Profiles
**Up Migration** (`000002_add_user_profiles_table.up.sql`):
```sql
-- Create user profiles table
CREATE TABLE app.user_profiles (
    user_id UUID PRIMARY KEY REFERENCES app.users(id) ON DELETE CASCADE,
    bio TEXT,
    location VARCHAR(255),
    website VARCHAR(255),
    avatar_url VARCHAR(500),
    date_of_birth DATE,
    phone VARCHAR(20),
    timezone VARCHAR(50) DEFAULT 'UTC',
    locale VARCHAR(10) DEFAULT 'en',
    preferences JSONB DEFAULT '{}',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Add validation constraints
ALTER TABLE app.user_profiles 
    ADD CONSTRAINT user_profiles_website_valid 
    CHECK (website IS NULL OR website ~* '^https?://');

ALTER TABLE app.user_profiles 
    ADD CONSTRAINT user_profiles_phone_valid
    CHECK (phone IS NULL OR phone ~ '^\+?[1-9]\d{1,14}$');

-- Create updated_at trigger
CREATE TRIGGER update_user_profiles_updated_at 
    BEFORE UPDATE ON app.user_profiles 
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Create JSONB index for preferences
CREATE INDEX idx_user_profiles_preferences 
    ON app.user_profiles USING gin(preferences);

-- Create index for common preference queries
CREATE INDEX idx_user_profiles_pref_theme 
    ON app.user_profiles((preferences->>'theme'));
```

**Down Migration** (`000002_add_user_profiles_table.down.sql`):
```sql
-- Drop trigger
DROP TRIGGER IF EXISTS update_user_profiles_updated_at ON app.user_profiles;

-- Drop indexes
DROP INDEX IF EXISTS idx_user_profiles_preferences;
DROP INDEX IF EXISTS idx_user_profiles_pref_theme;

-- Drop table
DROP TABLE IF EXISTS app.user_profiles;
```

### 000003: Add Column Migration Pattern
**Up Migration** (`000003_add_user_last_login.up.sql`):
```sql
-- Add last_login column to users table
ALTER TABLE app.users 
    ADD COLUMN last_login_at TIMESTAMP,
    ADD COLUMN last_login_ip INET;

-- Create index for last login queries
CREATE INDEX idx_users_last_login_at 
    ON app.users(last_login_at) WHERE deleted_at IS NULL;

-- Backfill existing users with null values (already null by default)
-- No data migration needed as this is a new optional field
```

**Down Migration** (`000003_add_user_last_login.down.sql`):
```sql
-- Drop index
DROP INDEX IF EXISTS idx_users_last_login_at;

-- Drop columns
ALTER TABLE app.users 
    DROP COLUMN IF EXISTS last_login_at,
    DROP COLUMN IF EXISTS last_login_ip;
```

### 000004: Data Migration Pattern
**Up Migration** (`000004_migrate_user_preferences.up.sql`):
```sql
-- Migration to move user settings from separate table to JSONB preferences
-- This is a more complex migration involving data transformation

BEGIN;

-- First, ensure the preferences column exists (should be from previous migration)
-- ALTER TABLE app.user_profiles ADD COLUMN IF NOT EXISTS preferences JSONB DEFAULT '{}';

-- Migrate data from old user_settings table to preferences JSONB
UPDATE app.user_profiles 
SET preferences = preferences || jsonb_build_object(
    'theme', COALESCE(us.theme, 'light'),
    'language', COALESCE(us.language, 'en'),
    'notifications_enabled', COALESCE(us.notifications_enabled, true),
    'email_notifications', COALESCE(us.email_notifications, true)
)
FROM user_settings us 
WHERE app.user_profiles.user_id = us.user_id;

-- Verify migration completed successfully
DO $$
DECLARE
    old_count INTEGER;
    new_count INTEGER;
BEGIN
    SELECT COUNT(*) INTO old_count FROM user_settings;
    SELECT COUNT(*) INTO new_count FROM app.user_profiles 
        WHERE preferences ? 'theme' OR preferences ? 'language';
    
    IF old_count != new_count THEN
        RAISE EXCEPTION 'Migration verification failed: old_count=%, new_count=%', old_count, new_count;
    END IF;
    
    RAISE NOTICE 'Migration verified: migrated % user preferences', old_count;
END $$;

-- Drop old table after successful migration
DROP TABLE IF EXISTS user_settings;

COMMIT;
```

**Down Migration** (`000004_migrate_user_preferences.down.sql`):
```sql
-- Rollback data migration - recreate old table and restore data
BEGIN;

-- Recreate the old user_settings table
CREATE TABLE user_settings (
    user_id UUID PRIMARY KEY REFERENCES app.users(id) ON DELETE CASCADE,
    theme VARCHAR(20) DEFAULT 'light',
    language VARCHAR(10) DEFAULT 'en',
    notifications_enabled BOOLEAN DEFAULT TRUE,
    email_notifications BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Restore data from JSONB preferences back to individual columns
INSERT INTO user_settings (user_id, theme, language, notifications_enabled, email_notifications)
SELECT 
    user_id,
    COALESCE(preferences->>'theme', 'light'),
    COALESCE(preferences->>'language', 'en'),
    COALESCE((preferences->>'notifications_enabled')::boolean, true),
    COALESCE((preferences->>'email_notifications')::boolean, true)
FROM app.user_profiles 
WHERE preferences IS NOT NULL 
    AND (preferences ? 'theme' OR preferences ? 'language' 
         OR preferences ? 'notifications_enabled' OR preferences ? 'email_notifications');

-- Remove migrated keys from preferences JSONB
UPDATE app.user_profiles 
SET preferences = preferences - 'theme' - 'language' - 'notifications_enabled' - 'email_notifications'
WHERE preferences ? 'theme' OR preferences ? 'language' 
    OR preferences ? 'notifications_enabled' OR preferences ? 'email_notifications';

COMMIT;
```

## Zero-Downtime Migration Strategies

### Safe Schema Changes
```sql
-- ✅ SAFE: Adding nullable columns
ALTER TABLE app.users ADD COLUMN new_field VARCHAR(255);

-- ✅ SAFE: Adding indexes concurrently
CREATE INDEX CONCURRENTLY idx_users_new_field ON app.users(new_field);

-- ✅ SAFE: Creating new tables
CREATE TABLE app.new_feature (...);

-- ⚠️  CAREFUL: Adding non-nullable columns (requires default)
ALTER TABLE app.users ADD COLUMN required_field VARCHAR(255) NOT NULL DEFAULT 'default_value';

-- ❌ UNSAFE: Dropping columns (breaks old app versions)
-- ALTER TABLE app.users DROP COLUMN old_field;

-- ❌ UNSAFE: Renaming columns (breaks old app versions)  
-- ALTER TABLE app.users RENAME COLUMN old_name TO new_name;
```

### Multi-Phase Migrations

#### Phase 1: Add New Column
```sql
-- Migration 001: Add new column (nullable initially)
ALTER TABLE app.users ADD COLUMN email_normalized VARCHAR(255);
CREATE INDEX CONCURRENTLY idx_users_email_normalized ON app.users(email_normalized);
```

#### Phase 2: Backfill Data
```sql
-- Migration 002: Backfill normalized email data
UPDATE app.users 
SET email_normalized = LOWER(TRIM(email)) 
WHERE email_normalized IS NULL;
```

#### Phase 3: Add Constraints
```sql
-- Migration 003: Make column required after backfill
ALTER TABLE app.users ALTER COLUMN email_normalized SET NOT NULL;
ALTER TABLE app.users ADD CONSTRAINT users_email_normalized_unique UNIQUE(email_normalized);
```

#### Phase 4: Remove Old Column (after app deployment)
```sql
-- Migration 004: Drop old column after app no longer uses it
DROP INDEX IF EXISTS idx_users_email;
ALTER TABLE app.users DROP COLUMN email;
ALTER TABLE app.users RENAME COLUMN email_normalized TO email;
```

## Migration Testing

### Migration Test Suite
```go
func TestMigrations(t *testing.T) {
    // Create test database
    db := setupTestDB(t)
    defer db.Close()
    
    migrator := NewMigrator(db, "migrations", testLogger)
    
    // Test forward migrations
    err := migrator.Migrate()
    require.NoError(t, err)
    
    // Verify final schema state
    verifySchema(t, db)
    
    // Test rollback migrations
    err = migrator.Rollback(0)
    require.NoError(t, err)
    
    // Verify clean rollback
    verifyEmptySchema(t, db)
}

func TestMigrationIdempotency(t *testing.T) {
    db := setupTestDB(t)
    defer db.Close()
    
    migrator := NewMigrator(db, "migrations", testLogger)
    
    // Apply migrations twice
    err := migrator.Migrate()
    require.NoError(t, err)
    
    err = migrator.Migrate()
    require.NoError(t, err) // Should not fail
}

func TestPartialRollback(t *testing.T) {
    db := setupTestDB(t)
    defer db.Close()
    
    migrator := NewMigrator(db, "migrations", testLogger)
    
    // Apply all migrations
    err := migrator.Migrate()
    require.NoError(t, err)
    
    // Rollback to specific version
    targetVersion := 2
    err = migrator.Rollback(targetVersion)
    require.NoError(t, err)
    
    // Verify only later migrations were rolled back
    applied, err := migrator.GetAppliedVersions()
    require.NoError(t, err)
    
    for _, version := range applied {
        require.LessOrEqual(t, version, targetVersion)
    }
}
```

## Deployment Integration

### CI/CD Pipeline Integration
```yaml
# .github/workflows/migrate.yml
name: Database Migration

on:
  pull_request:
    paths:
      - 'migrations/**'
  
jobs:
  test-migrations:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Go
        uses: actions/setup-go@v3
        with:
          go-version: '1.19'
      
      - name: Test migrations
        run: |
          export DATABASE_URL="postgres://postgres:postgres@localhost:5432/testdb?sslmode=disable"
          go test ./migrations -v
      
      - name: Test migration rollbacks
        run: |
          export DATABASE_URL="postgres://postgres:postgres@localhost:5432/testdb?sslmode=disable"
          go run ./cmd/migrate rollback 0
          go run ./cmd/migrate up
```

### Production Deployment Script
```bash
#!/bin/bash
# deploy-migrations.sh

set -euo pipefail

DB_URL="${DATABASE_URL:-}"
DRY_RUN="${DRY_RUN:-false}"
BACKUP_BEFORE="${BACKUP_BEFORE:-true}"

if [ -z "$DB_URL" ]; then
    echo "ERROR: DATABASE_URL is required"
    exit 1
fi

echo "=== Database Migration Deployment ==="

# Create backup before migration
if [ "$BACKUP_BEFORE" = "true" ]; then
    echo "Creating database backup..."
    backup_file="backup_$(date +%Y%m%d_%H%M%S).sql"
    pg_dump "$DB_URL" > "$backup_file"
    echo "Backup created: $backup_file"
fi

# Dry run check
if [ "$DRY_RUN" = "true" ]; then
    echo "DRY RUN: Checking pending migrations..."
    ./migrate -database "$DB_URL" -path ./migrations version
    ./migrate -database "$DB_URL" -path ./migrations up -dry-run
    exit 0
fi

# Apply migrations with lock
echo "Acquiring migration lock..."
echo "Applying pending migrations..."

if ./migrate -database "$DB_URL" -path ./migrations up; then
    echo "✅ Migrations completed successfully"
else
    echo "❌ Migration failed"
    
    if [ "$BACKUP_BEFORE" = "true" ]; then
        echo "To restore from backup:"
        echo "  psql $DB_URL < $backup_file"
    fi
    
    exit 1
fi

echo "=== Migration deployment completed ==="
```

## Decision History & Trade-offs

### Custom Migration Tool vs Third-Party
**Decision**: Implement custom migration runner instead of using migrate/golang-migrate
**Rationale**:
- Full control over migration logic and error handling
- Integration with application logging and monitoring
- Custom validation and safety checks
- Simplified deployment workflow

**Trade-offs**:
- More code to maintain and debug
- Missing features from mature tools
- Potential for custom bugs in migration logic
- Team learning curve for custom tooling

### Forward-Only Migration Philosophy
**Decision**: Prefer additive changes and avoid destructive migrations
**Rationale**:
- Safer for production deployments with zero downtime
- Easier rollback procedures for application code
- Reduces risk of data loss during deployments
- Compatible with blue-green deployments

**Trade-offs**:
- Temporary schema bloat with unused columns
- More complex multi-phase migrations
- Longer migration timelines for schema changes
- Additional cleanup migrations required

### Atomic Migration Transactions
**Decision**: Wrap each migration in a database transaction
**Rationale**:
- Ensures migration consistency (all or nothing)
- Simplifies error handling and cleanup
- Prevents partial migration states
- Standard database practice

**Trade-offs**:
- Long-running migrations can hold locks
- Some DDL operations cannot be rolled back
- Memory usage for large data migrations
- Potential deadlock issues with concurrent operations

---

This completes the database domain documentation with comprehensive migration strategy, tooling, and deployment procedures.