Build me a complete no-code machine learning modeling tool with two parts:
1. A React frontend (App.jsx)
2. A Python FastAPI backend (server.py)

Both should work together as a local application. The user runs `python server.py` and opens the app in a browser.

---

## PART 1: PYTHON BACKEND (server.py)

### Setup & Dependencies
Use these libraries (write a requirements.txt too):
- fastapi
- uvicorn
- pandas
- numpy
- scikit-learn
- xgboost
- statsmodels
- prophet (optional, wrap in try/except)
- tensorflow (optional, wrap in try/except)
- pyarrow (for parquet)
- openpyxl (for excel)
- scipy

### Server Configuration
- Run on http://localhost:8000
- Enable CORS for http://localhost:3000 and http://localhost:5173 and *
- All endpoints return JSON

### API Endpoints

**POST /upload**
- Accept multipart file upload
- Support: .csv, .xlsx, .xls, .parquet, .json, .txt
- Return:
  - columns: list of {name, dtype, null_count, unique_count, sample_values}
  - shape: {rows, cols}
  - preview: first 10 rows as list of dicts
  - inferred_types: {column_name: "numeric" | "categorical" | "datetime" | "text"}
  - stats: basic describe() output per numeric column
- Store the dataframe in a server-side session dict keyed by a file_id (uuid)

**POST /train**
- Accept JSON body:
  - file_id: str
  - model_type: str (e.g. "xgboost_classifier")
  - target_column: str
  - feature_columns: list[str]
  - date_column: str (optional, for time series)
  - params: dict
  - test_size: float (0.1 to 0.4)
  - cv_folds: int (optional)
- Preprocess automatically:
  - Label encode categoricals
  - Fill nulls with median/mode
  - Parse datetimes
- Train the requested model with given params
- Return results based on task type (see metrics below)
- Also return: feature_importances (if available), training_time_seconds

**POST /compare**
- Accept list of up to 3 {file_id, model_type, target_column, feature_columns, params, test_size}
- Train all 3, return all results in one response

**GET /health**
- Return {"status": "ok"}

### Models to Implement (map string names to actual sklearn/etc classes)

REGRESSION:
- "linear_regression" → LinearRegression
- "ridge" → Ridge
- "lasso" → Lasso
- "random_forest_regressor" → RandomForestRegressor
- "xgboost_regressor" → XGBRegressor
- "gradient_boosting_regressor" → GradientBoostingRegressor
- "svr" → SVR

CLASSIFICATION:
- "logistic_regression" → LogisticRegression
- "random_forest_classifier" → RandomForestClassifier
- "xgboost_classifier" → XGBClassifier
- "svm_classifier" → SVC (probability=True)
- "knn" → KNeighborsClassifier
- "naive_bayes" → GaussianNB
- "gradient_boosting_classifier" → GradientBoostingClassifier

TIME SERIES:
- "arima" → ARIMA from statsmodels
- "sarima" → SARIMAX from statsmodels
- "exponential_smoothing" → ExponentialSmoothing from statsmodels

CLUSTERING:
- "kmeans" → KMeans
- "dbscan" → DBSCAN
- "hierarchical" → AgglomerativeClustering

DEEP LEARNING (wrap in try/except, skip if tensorflow not installed):
- "mlp" → MLPRegressor or MLPClassifier
- "lstm" → Build with tensorflow.keras Sequential + LSTM layers
- "cnn_1d" → Build with tensorflow.keras Sequential + Conv1D layers
- "autoencoder" → Build with tensorflow.keras encoder-decoder architecture

For all deep learning models, support these backend params:
- optimizer: "adam" | "sgd" | "rmsprop" | "adagrad" | "adadelta"
- loss: auto-select based on task, but allow override
  - Regression: "mse" | "mae" | "huber"
  - Classification: "binary_crossentropy" | "categorical_crossentropy"
- activation: "relu" | "sigmoid" | "tanh" | "leaky_relu" | "elu"
- output_activation: auto-set but configurable ("sigmoid" for binary, "softmax" for multiclass, "linear" for regression)
- learning_rate: float (applied to optimizer)
- lr_scheduler: "none" | "step_decay" | "exponential_decay" | "cosine_annealing"
- regularization: "none" | "l1" | "l2" | "elasticnet" with lambda float
- weight_init: "glorot_uniform" (Xavier) | "he_normal" | "random_normal"
- validation_split: float (0.05–0.2, portion of training data used for live val loss)
- early_stopping: bool
- early_stopping_patience: int (5–50)
- For LSTM specifically:
  - return_sequences: bool
  - stateful: bool
  - bidirectional: bool
- epochs, batch_size, dropout, units, layers (already listed, keep these)

### Metrics to Return

REGRESSION: mae, mse, rmse, mape, r2
Also return: y_test (list), y_pred (list), residuals (list)

CLASSIFICATION: accuracy, precision, recall, f1, roc_auc
Also return: confusion_matrix (2D list), y_test (list), y_pred (list), y_prob (list, for ROC)

TIME SERIES: mae, mse, rmse, mape
Also return: actual (list), forecast (list), dates (list), confidence_lower (list), confidence_upper (list)

CLUSTERING: silhouette_score, inertia (if kmeans)
Also return: labels (list), pca_2d (list of [x,y] for scatter plot)

DEEP LEARNING (additional): also return training_history as {epoch: [], loss: [], val_loss: [], accuracy: [] (if classification)}

---

## PART 2: REACT FRONTEND (App.jsx)

### Design System
- Light mode default, dark mode toggle (store in localStorage)
- Accent colors: indigo #6366f1, violet #7c3aed
- Font: Import "DM Sans" (UI) and "DM Mono" (metrics/numbers) from Google Fonts via @import in a style tag
- Card-based layout, 8px radius, subtle box shadows
- Smooth 150ms transitions everywhere
- Sidebar + main content layout
- Mobile-aware (doesn't break on smaller screens)

### Layout
- Top navbar: "ModelKit" logo (indigo), step progress bar, dark mode toggle button
- Left sidebar: collapsible, shows step list with checkmarks
- Main area: step content
- Bottom bar: Back / Next Step buttons, sticky

### 5-Step Flow

**STEP 1 — Import Data**
- Drag & drop zone + click to upload
- Accepted formats shown: CSV, Excel, Parquet, JSON, TXT
- On upload: POST to /upload (FormData)
- Show loading skeleton while uploading
- On success display:
  - File summary card: filename, rows, columns, file size
  - Column cards: name, type badge (color-coded), null%, unique count
  - Scrollable data preview table (first 10 rows)
  - Inferred type badges: 🔢 Numeric, 🔤 Categorical, 📅 Datetime, 📝 Text
- Store file_id from response

**STEP 2 — Select Model**
- Show model categories as tabs: Deep Learning | Statistical | Regression | Classification | Clustering
- Each model = a card with: name, short description, best-use badge, complexity badge (Low/Med/High)
- "Recommended" badge shown on models that match data profile:
  - Datetime column detected → badge on ARIMA, SARIMA, LSTM
  - Binary target → badge on XGBoost Classifier, Random Forest Classifier
  - Continuous target → badge on XGBoost Regressor, Random Forest Regressor
  - No target → badge on Clustering models
- Top banner: "Based on your data, we recommend [Model] — [one line reason]"
- Clicking a card: select it + show right panel with full description, when to use, data requirements
- Selected card gets indigo border highlight

**STEP 3 — Configure Features**
- Two-column layout:
  - Left: All columns listed with checkboxes (select as features)
  - Right: Target column dropdown + Date column dropdown (if time series selected)
- Below: Correlation heatmap table (color scale from red→white→blue)
- Warning chips for: high null %, low variance, high cardinality
- Smart feature suggestions: "These features are most correlated with your target"

**STEP 4 — Set Parameters**
- Dynamic form based on selected model
- Each param: label, input (slider / dropdown / number), tooltip (?) with plain-English explanation
- Standard params per classical model:
  - Random Forest: n_estimators (10-500), max_depth (1-30), min_samples_split (2-20)
  - XGBoost: learning_rate (0.01-0.3), n_estimators (50-500), max_depth (1-15), subsample (0.5-1.0), colsample_bytree (0.5-1.0)
  - Logistic Regression: C (0.01-10), max_iter (100-1000), solver dropdown
  - ARIMA: p (0-5), d (0-2), q (0-5) with plain-English explanation of each
  - SARIMA: p, d, q + P, D, Q, m (seasonal period)
  - KMeans: n_clusters (2-15), init (kmeans++ / random), max_iter
  - SVM: C, kernel dropdown (linear/rbf/poly), gamma

- Deep Learning params (show for MLP, LSTM, CNN-1D, Autoencoder):
  - Architecture section:
    - units per layer: slider (16–256)
    - number of layers: slider (1–4)
    - dropout rate: slider (0–0.5) with tooltip "Randomly disables neurons during training to prevent overfitting"
    - activation function: dropdown → ReLU, Sigmoid, Tanh, LeakyReLU, ELU
      - tooltip per option: e.g. "ReLU is the standard choice for hidden layers"
    - weight initialization: dropdown → Xavier (Glorot Uniform), He Normal, Random Normal
      - tooltip: "He Normal works best with ReLU. Xavier is better for Sigmoid/Tanh."
  - Training section:
    - optimizer: dropdown → Adam, SGD, RMSprop, AdaGrad, AdaDelta
      - tooltip per option: "Adam adapts the learning rate automatically — best default. SGD is slower but can generalize better with tuning."
    - learning rate: slider (0.0001–0.1, log scale display)
    - learning rate scheduler: dropdown → None, Step Decay, Exponential Decay, Cosine Annealing
      - tooltip: "Schedulers reduce the learning rate over time to fine-tune convergence"
    - loss function: dropdown (auto-selected but overridable)
      - Regression: MSE, MAE, Huber Loss
      - Classification: Binary Crossentropy, Categorical Crossentropy
      - tooltip per option: "Huber Loss is more robust to outliers than MSE"
    - epochs: slider (10–200)
    - batch size: dropdown (16, 32, 64, 128, 256)
      - tooltip: "Smaller batches = noisier but sometimes better generalization. Larger = faster training."
    - validation split: slider (5%–20%)
      - tooltip: "Portion of training data used to monitor val loss each epoch"
  - Regularization section:
    - regularization type: dropdown → None, L1, L2, ElasticNet
      - tooltip: "L2 (Ridge) penalizes large weights. L1 (Lasso) can zero out weights entirely."
    - lambda (regularization strength): slider (0.0001–0.1, shown only if regularization selected)
  - Early Stopping section:
    - toggle: on/off
    - patience: slider (5–50) — tooltip: "Training stops if val loss doesn't improve for this many epochs"
  - LSTM-specific section (only shown when LSTM selected):
    - bidirectional: toggle — tooltip: "Processes sequence forwards AND backwards. Better for most NLP/signal tasks."
    - return sequences: toggle — tooltip: "Required if stacking multiple LSTM layers"
    - stateful: toggle — tooltip: "Carries hidden state across batches. Useful for very long sequences."
  - Training History Chart: After training, show a Recharts LineChart of loss vs val_loss per epoch

- "Quick Presets" row for ALL models: [Conservative] [Balanced] [Aggressive] buttons — auto-fill all params
- Recommendation note: "For [N] rows, we suggest Balanced settings"
- Train/Test split slider: 60/40 to 90/10, default 80/20, show "Training: X rows | Testing: Y rows"
- Cross-validation toggle (not for deep learning): k-fold, k slider (3-10)
- Big "Train Model" button at bottom (indigo, prominent)

**STEP 5 — Results**
- Show training time badge
- Metrics cards at top (large numbers, color-coded by performance):
  - Green = good, yellow = okay, red = poor (thresholds differ per metric)
  - Each metric card has a (?) tooltip with plain-English explanation
- Charts using Recharts:
  - Regression: Actual vs Predicted scatter + line, Residuals histogram
  - Classification: Confusion matrix heatmap, ROC curve
  - Time Series: Forecast line with confidence interval shading
  - Clustering: 2D scatter with colored cluster groups
  - Deep Learning (all types): Training history chart — loss vs val_loss per epoch as a line chart
- Feature importance bar chart (if model supports it)
- "Add to Comparison" button (max 3 models)
- "Save Results" → download JSON: model name, params, all metrics, timestamp
- "Export Report" → generate and download clean HTML report card

**Model Comparison Panel** (appears when 2+ models added):
- Side-by-side metrics table, best value in each row highlighted indigo
- Overlaid charts (togglable)
- "Best Model" badge on winner
- "Clear Comparison" button

### API Integration
- Base URL: http://localhost:8000
- On app load: GET /health — if fails, show banner: "Backend not running. Start it with: python server.py"
- All fetch calls with proper error handling
- Loading states: spinner + "Training [Model Name]..." text
- Error toasts for failed requests

### State Shape (React)
```js
{
  data: { fileId, fileName, columns, types, preview, shape },
  model: { category, id, name, taskType },
  features: { inputs, target, dateColumn },
  params: { ...modelParams },
  split: { testSize, cvFolds, useCv },
  results: { current: {...}, comparison: [] },
  ui: { step, darkMode, loading, error, healthOk }
}
```

### Toast Notifications
Show for: file uploaded, model selected, training started, training complete, results saved, error occurred

---

## PART 3: ADDITIONAL FILES

### requirements.txt
List all Python dependencies with pinned major versions

### README.md
Include:
- What this tool does
- Installation steps:
  1. `pip install -r requirements.txt`
  2. `python server.py`
  3. Open App.jsx in a React project (Vite) or serve it
- How to use (5 steps)
- Supported models list with descriptions
- Supported file formats
- Deep learning setup note (TensorFlow optional)

### start.sh
```bash
#!/bin/bash
echo "Starting ModelKit backend..."
python server.py &
echo "Backend running at http://localhost:8000"
echo "Open your React app to get started."
```

---

## FINAL REQUIREMENTS

- server.py must be a single self-contained file
- App.jsx must be a single self-contained file
- All model training errors must be caught and returned as {"error": "message"} with HTTP 400
- Backend must handle: missing values, wrong column types, mismatched shapes, TensorFlow not installed
- Frontend must never crash — all API errors show as user-friendly messages
- Deep learning params section must be cleanly separated into: Architecture / Training / Regularization / Early Stopping / Model-Specific
- Every single parameter (classical and deep learning) must have a plain-English tooltip
- The UI must feel like a real SaaS product — polished, guided, and approachable
- Dark mode must look genuinely good — not just color-inverted light mode
- Indigo/violet accent must be consistent throughout both light and dark modes