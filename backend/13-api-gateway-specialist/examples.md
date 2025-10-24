# API Gateway Specialist - Examples

---

## Example: Kong with Docker Compose

```yaml
# docker-compose.yml - Kong API Gateway
version: '3.8'

services:
  kong-database:
    image: postgres:15
    environment:
      POSTGRES_DB: kong
      POSTGRES_USER: kong
      POSTGRES_PASSWORD: kong
    volumes:
      - kong-db-data:/var/lib/postgresql/data

  kong-migrations:
    image: kong:3.4
    command: kong migrations bootstrap
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
    depends_on:
      - kong-database

  kong:
    image: kong:3.4
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "8000:8000"  # Proxy
      - "8443:8443"  # Proxy SSL
      - "8001:8001"  # Admin API
    depends_on:
      - kong-migrations

  konga:
    image: pantsel/konga
    environment:
      NODE_ENV: production
    ports:
      - "1337:1337"
    depends_on:
      - kong

volumes:
  kong-db-data:
```

---

## Example: NGINX Rate Limiting

```nginx
# Rate limiting with burst
http {
    limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
    limit_req_zone $http_api_key zone=api_key:10m rate=100r/s;

    server {
        location /api/public {
            limit_req zone=general burst=20 nodelay;
            limit_req_status 429;

            proxy_pass http://backend;
        }

        location /api/premium {
            limit_req zone=api_key burst=50 nodelay;

            proxy_pass http://backend;
        }
    }
}
```

---

**Versión:** 1.0.0
