# NLP Specialist - Examples

---

## Example: Sentiment Analysis Pipeline

```python
from transformers import pipeline

# ✅ Pre-trained sentiment analysis
classifier = pipeline("sentiment-analysis")

texts = [
    "I love this product!",
    "This is terrible",
    "It's okay, nothing special"
]

results = classifier(texts)
# [{'label': 'POSITIVE', 'score': 0.99}, ...]
```

## Example: Zero-shot Classification

```python
from transformers import pipeline

# ✅ Classify without training
classifier = pipeline("zero-shot-classification")

text = "This is a course about Python programming"
candidate_labels = ["education", "politics", "business"]

result = classifier(text, candidate_labels)
# {'labels': ['education', 'business', 'politics'], 'scores': [0.95, 0.03, 0.02]}
```

---

**Versión:** 1.0.0
