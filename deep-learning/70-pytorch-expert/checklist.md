# PyTorch Expert - Checklist

---

## 🏗️ Model
- [ ] ¿Heredando de nn.Module?
- [ ] ¿Layers en __init__, forward implementado?
- [ ] ¿Activations apropiadas?
- [ ] ¿Regularization (Dropout, BatchNorm)?

## 🎓 Training
- [ ] ¿optimizer.zero_grad() antes de backward?
- [ ] ¿loss.backward() y optimizer.step()?
- [ ] ¿model.train() / model.eval()?
- [ ] ¿torch.no_grad() en validation?

## 💾 Data
- [ ] ¿DataLoader con batch_size?
- [ ] ¿Data en device correcto (.to(device))?
- [ ] ¿Transforms aplicados?

## 🚀 Performance
- [ ] ¿GPU utilizado cuando disponible?
- [ ] ¿Mixed precision (torch.cuda.amp)?

---

**Versión:** 1.0.0
