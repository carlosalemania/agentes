# Microservices Architect - Examples

---

## Example: E-commerce Microservices Architecture

```yaml
# docker-compose.yml - Microservices stack
version: '3.8'

services:
  api-gateway:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - order-service
      - product-service

  user-service:
    build: ./services/user
    environment:
      - DATABASE_URL=postgres://user-db:5432/users
      - REDIS_URL=redis://redis:6379
    depends_on:
      - user-db
      - redis

  order-service:
    build: ./services/order
    environment:
      - DATABASE_URL=postgres://order-db:5432/orders
      - KAFKA_BROKERS=kafka:9092
    depends_on:
      - order-db
      - kafka

  product-service:
    build: ./services/product
    environment:
      - DATABASE_URL=postgres://product-db:5432/products
    depends_on:
      - product-db

  user-db:
    image: postgres:15
    environment:
      - POSTGRES_DB=users

  order-db:
    image: postgres:15
    environment:
      - POSTGRES_DB=orders

  product-db:
    image: postgres:15
    environment:
      - POSTGRES_DB=products

  redis:
    image: redis:7-alpine

  kafka:
    image: confluentinc/cp-kafka:latest
```

---

## Example: Service Mesh with Istio

```yaml
# Istio - Virtual Service and Circuit Breaker
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
    - order-service
  http:
    - route:
        - destination:
            host: order-service
            subset: v1
          weight: 90
        - destination:
            host: order-service
            subset: v2
          weight: 10
      retries:
        attempts: 3
        perTryTimeout: 2s
      timeout: 10s

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: order-service
spec:
  host: order-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

---

**Versión:** 1.0.0
