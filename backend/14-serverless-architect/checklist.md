# Serverless Architect - Checklist

---

## ⚡ Performance

- [ ] ¿Cold start optimization (bundle size, layers)?
- [ ] ¿Provisioned concurrency para funciones críticas?
- [ ] ¿Connection pooling implementado?
- [ ] ¿Timeout apropiado configurado?

## 💰 Cost Optimization

- [ ] ¿Memory size optimizado?
- [ ] ¿Reserved concurrency para controlar costos?
- [ ] ¿DynamoDB on-demand vs provisioned?
- [ ] ¿S3 lifecycle policies configuradas?

## 🔒 Security

- [ ] ¿IAM roles con least privilege?
- [ ] ¿Secrets en Secrets Manager/Parameter Store?
- [ ] ¿VPC configuration si es necesario?
- [ ] ¿Cognito/API Gateway authorization?

## 📊 Observability

- [ ] ¿CloudWatch Logs habilitados?
- [ ] ¿X-Ray tracing configurado?
- [ ] ¿CloudWatch Alarms para errores?
- [ ] ¿Metrics customizados?

---

**Versión:** 1.0.0
