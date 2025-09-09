# V1 to V2 Database Migration System


### Key Changes V1 → V2
- **V1**: Separate `Teacher`, `Parent`, `Student` tables with direct authentication
- **V2**: Central `User` table with role-based relationships via `UserRole`
- **Challenge**: Creating User records for every migrated teacher, parent, and student

### Migration Flow
```
1. Schools → 2. Teachers → 3. Parents → 4. Students
```

Each step creates necessary User records and maintains referential integrity.

## 📋 Prerequisites

### 1. Install Dependencies

```bash
pip install -r requirements.txt
```

### 2. Environment Configuration

Create a `.env` file with your database configurations:

```bash
# V1 Database (Source)
V1_DB_HOST=localhost
V1_DB_PORT=5432
V1_DB_NAME=v1_database
V1_DB_USER=your_user
V1_DB_PASSWORD=your_password

# V2 Database (Target)
V2_DB_HOST=localhost
V2_DB_PORT=5432
V2_DB_NAME=v2_database
V2_DB_USER=your_user
V2_DB_PASSWORD=your_password
```

### 3. Setup V2 Database Structure

Before running migrations, initialize the V2 database with required reference data:

```bash
python setup_v2.py
```

This creates:
- Permission modules and permissions
- Grade systems and levels  
- Default curriculums
- Class levels
- Subject categories and subjects
- User types and roles

## 🚀 Quick Start

### Basic Migration

```bash
# Full migration (recommended)
python migrate.py

# Dry run (preview changes)
python migrate.py --dry-run

# Migrate specific entities
python migrate.py --entities schools,teachers

# Skip confirmation prompts
python migrate.py --force
```

### Check Migration Status

```bash
# Analyze source schemas
python analyze_schemas.py

# View migration progress
python migrate.py --status
```

## 🔧 Core Components

### Database Utilities (`db_utils.py`)
- `DatabaseManager`: Handles V1/V2 database connections
- Async and sync connection support
- Query execution and bulk operations
- Connection pooling and error handling

### User Management (`user_utils.py`)  
- `UserManager`: Central user creation logic
- Email/phone uniqueness handling
- Role-based user creation (Teacher/Parent/Student)
- Password generation and hashing

### Individual Migrators
- `SchoolMigrator`: Schools and admin users
- `TeacherMigrator`: Teachers with User records
- `ParentMigrator`: Parents with User records  
- `StudentMigrator`: Students with parent relationships

### Configuration (`config.py`)
- Database connection settings
- Migration order and dependencies
- Default values and mappings

## 📊 Migration Details

### User Creation Loop Pattern

For each migrated entity (Teacher/Parent/Student), the system:

1. **Checks Uniqueness**: Verifies email/phone not already used
2. **Resolves Conflicts**: Adds counter suffixes for duplicates  
3. **Creates User Record**: With appropriate role and profile data
4. **Links Relationships**: Connects to schools, classes, parents as needed

### Sequential Processing

```bash
Schools First   → Provides school mappings for other entities
Teachers Next   → Creates teacher users and assignments  
Parents Then    → Creates parent users with school links
Students Last   → Creates student users with parent relationships
```

### Data Transformations

- **Phone Numbers**: Standardized format with country codes
- **Addresses**: Mapped from V1 structure to V2 components
- **Names**: Split full names into first/middle/last components
- **Emails**: Generated for missing email addresses
- **Passwords**: Auto-generated secure passwords for all users

## 🔍 Troubleshooting

### Common Issues

**Database Connection Errors**
```bash
# Check connection settings
python -c "from config import V1_DB_CONFIG, V2_DB_CONFIG; print('V1:', V1_DB_CONFIG); print('V2:', V2_DB_CONFIG)"

# Test connections individually  
python -c "from db_utils import DatabaseManager; dm = DatabaseManager(); dm.connect_v1_sync()"
```

**Migration Failures**
```bash
# Check logs for detailed error messages
tail -f migration.log

# Run with verbose output
python migrate.py --verbose --dry-run
```

**Duplicate Data**
```bash
# Check for existing data in V2
python -c "from db_utils import DatabaseManager; dm = DatabaseManager(); print(dm.execute_query('SELECT COUNT(*) FROM \"User\"', db='v2'))"

# Clean V2 database if needed (⚠️ DESTRUCTIVE)
python setup_v2.py --clean
```

### Performance Optimization

- **Batch Processing**: Migrators use bulk inserts for better performance
- **Connection Pooling**: Async connections reduce overhead  
- **Progress Tracking**: Real-time progress indicators
- **Memory Management**: Processes data in chunks

## 🧪 Testing

### Unit Tests
```bash
# Run all tests
python -m pytest tests/

# Run specific test categories
python -m pytest tests/unit/
python -m pytest tests/integration/
```

### Integration Tests  
```bash
# Test with sample data
python -m pytest tests/integration/test_full_migration.py

# Test individual migrators
python -m pytest tests/integration/test_teacher_migrator.py
```

## 📈 Monitoring

### Progress Tracking

The migration system provides detailed progress information:

- **Entity Counts**: Shows total entities to migrate
- **Success Rates**: Tracks successful vs failed migrations  
- **Error Details**: Logs specific failure reasons
- **Time Estimates**: Provides completion time estimates

### Validation

After migration:

```bash
# Verify data integrity
python migrate.py --validate

# Compare record counts
python analyze_schemas.py --compare-counts

# Test user authentication  
python -c "from user_utils import UserManager; um = UserManager(); um.verify_user_login('test@example.com')"
```
