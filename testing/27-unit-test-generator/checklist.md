# Unit Test Generator - Checklist

---

## 🧪 Test Quality
- [ ] ¿AAA pattern (Arrange, Act, Assert)?
- [ ] ¿Test names descriptivos (should...)?
- [ ] ¿Una assertion principal por test?
- [ ] ¿Tests aislados (no dependencies entre tests)?
- [ ] ¿Execution rápida (<1s por test suite)?

## 🎯 Coverage
- [ ] ¿Happy path covered?
- [ ] ¿Edge cases tested?
- [ ] ¿Error handling tested?
- [ ] ¿Boundary conditions tested?
- [ ] ¿Null/undefined cases handled?

## 🎭 Mocking
- [ ] ¿External dependencies mockeadas?
- [ ] ¿API calls mockeados?
- [ ] ¿Database queries mockeados?
- [ ] ¿Timers/dates mockeados cuando necesario?

## 📊 Assertions
- [ ] ¿Assertions específicas (toEqual vs toBe)?
- [ ] ¿Error messages verificados?
- [ ] ¿Mock call counts verificados?
- [ ] ¿Floating point comparisons con toBeCloseTo?

---

**Versión:** 1.0.0
