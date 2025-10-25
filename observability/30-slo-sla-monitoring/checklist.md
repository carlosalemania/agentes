# SLO/SLA Monitoring - Checklist

---

## 📊 SLI Definition

- [ ] ¿SLIs claramente definidos y medibles?
- [ ] ¿Availability SLI tracked?
- [ ] ¿Latency SLI (p95/p99) tracked?
- [ ] ¿Error rate SLI tracked?
- [ ] ¿Data freshness SLI (si aplica)?

## 🎯 SLO Configuration

- [ ] ¿SLOs realistas basados en user needs?
- [ ] ¿99.9% availability o apropiado?
- [ ] ¿Latency targets (p95 < 200ms)?
- [ ] ¿Rolling window (28/30 days)?
- [ ] ¿Error budget calculated?

## 🚨 Alerting

- [ ] ¿Multi-window burn rate alerts configurados?
- [ ] ¿Fast burn (1h window) alerting?
- [ ] ¿Slow burn (6h/1d window) alerting?
- [ ] ¿Page vs ticket severity definido?
- [ ] ¿Alert fatigue evitado?

## 📈 Monitoring & Reporting

- [ ] ¿SLO dashboards en Grafana?
- [ ] ¿Error budget visibility?
- [ ] ¿Historical compliance reports?
- [ ] ¿Burn rate visualization?
- [ ] ¿Customer-facing status page?

## 📋 SLA Management

- [ ] ¿SLA terms claramente documentados?
- [ ] ¿Uptime guarantees definidos?
- [ ] ¿Compensation/credits policy?
- [ ] ¿SLA breach notifications?
- [ ] ¿Monthly SLA reports?

## 🔧 Best Practices

- [ ] ¿Error budget used for decision making?
- [ ] ¿Postmortems cuando SLO violated?
- [ ] ¿SLOs reviewed quarterly?
- [ ] ¿Balance entre reliability y velocity?

---

**Versión:** 1.0.0
