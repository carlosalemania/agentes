# Docker Container Expert - Examples

---

## Example: Optimized Node.js Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
USER node
CMD ["node", "server.js"]
```

---

**Versión:** 1.0.0
