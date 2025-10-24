# Database Migration Specialist - Examples

---

## Example: Flyway Configuration

```properties
# flyway.conf
flyway.url=jdbc:postgresql://localhost:5432/mydb
flyway.user=postgres
flyway.password=password
flyway.schemas=public
flyway.locations=filesystem:./migrations
flyway.baselineOnMigrate=true
flyway.validateOnMigrate=true
```

```bash
# Run migrations
flyway migrate

# Get migration status
flyway info

# Validate migrations
flyway validate

# Repair migration history
flyway repair
```

---

## Example: Django Migrations

```python
# models.py
from django.db import models

class User(models.Model):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=255)
    created_at = models.DateTimeField(auto_now_add=True)

# Generate migration
# python manage.py makemigrations

# Run migrations
# python manage.py migrate

# Show migrations
# python manage.py showmigrations
```

---

**Versión:** 1.0.0
