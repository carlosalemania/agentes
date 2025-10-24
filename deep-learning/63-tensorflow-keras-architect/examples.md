# TensorFlow/Keras Architect - Examples

---

## CNN for Image Classification

```python
model = keras.Sequential([
    keras.layers.Conv2D(32, 3, activation='relu', input_shape=(28, 28, 1)),
    keras.layers.MaxPooling2D(),
    keras.layers.Conv2D(64, 3, activation='relu'),
    keras.layers.MaxPooling2D(),
    keras.layers.Flatten(),
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dense(10, activation='softmax')
])
```

## LSTM for Time Series

```python
model = keras.Sequential([
    keras.layers.LSTM(64, return_sequences=True, input_shape=(timesteps, features)),
    keras.layers.LSTM(32),
    keras.layers.Dense(1)
])
```

---

**Versión:** 1.0.0
