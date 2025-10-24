# ML Pipeline Architect - Examples

---

## Complete ML Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV
import mlflow

# Define pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', RandomForestClassifier())
])

# Hyperparameter tuning
param_grid = {
    'model__n_estimators': [50, 100, 200],
    'model__max_depth': [5, 10, None]
}

grid_search = GridSearchCV(pipeline, param_grid, cv=5)

# Train with MLflow
with mlflow.start_run():
    grid_search.fit(X_train, y_train)

    # Log best params
    mlflow.log_params(grid_search.best_params_)
    mlflow.log_metric("accuracy", grid_search.best_score_)

    # Save model
    mlflow.sklearn.log_model(grid_search.best_estimator_, "model")
```

---

**Versión:** 1.0.0
