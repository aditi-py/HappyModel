# HappyModel

A complete no-code machine learning tool. Upload a dataset, pick a model, configure it, train it, and compare results — all without writing a single line of code.

---

## What is HappyModel?

HappyModel guides you through the full ML lifecycle in 5 steps:

1. **Import Data** — Drag-and-drop or browse for a file (CSV, Excel, Parquet, JSON, TXT). An automatic data profile shows row/column counts, column types, null percentages, and a preview of the first few rows.
2. **Select Model** — Choose from 30+ models across regression, classification, time series, clustering, and deep learning. A quick-filter bar and category tabs help narrow options.
3. **Configure Features** — Select the target column and input features. A correlation heatmap highlights relationships. Features can be toggled individually or in bulk.
4. **Set Parameters** — Tune hyperparameters with sliders and dropdowns. Quick presets (Conservative / Balanced / Aggressive) apply sensible defaults with one click. Every parameter has a plain-English tooltip.
5. **View Results** — Explore performance metrics, feature importance charts, actual-vs-predicted scatter plots, residual plots, confusion matrices (classification), ROC curves, PCA cluster views, and training time. Add up to 3 models to a side-by-side comparison panel. Export a single-model HTML report or a full comparison report.

---

## Quick Start

### Prerequisites

- Python 3.8+
- Node.js 16+

### 1 — Install Python dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 2 — Start the backend

```bash
python server.py
```

The FastAPI backend starts at `http://localhost:8000`. A health-check endpoint is available at `http://localhost:8000/health`.

### 3 — Start the frontend

```bash
cd frontend
npm install
npm run dev
```

The React dev server starts at `http://localhost:5173`.

### 4 — Open the app

Navigate to `http://localhost:5173` in your browser.

---

## Supported File Formats

| Format | Extensions | Notes |
|--------|-----------|-------|
| CSV | `.csv` | Any delimiter auto-detected |
| Excel | `.xlsx`, `.xls` | First sheet loaded |
| Parquet | `.parquet` | PyArrow required |
| JSON | `.json` | Array of objects or records format |
| Text | `.txt` | Tab or comma-separated |

### Sample Datasets

Ready-to-use sample files are in the `docs/` folder:

| File | Task | Target |
|------|------|--------|
| `sample_data.csv` | Regression | `price` |
| `house_prices.csv` | Regression | `price` |
| `customer_churn.csv` | Classification | `churned` |
| `diabetes_risk.csv` | Classification | `at_risk` |
| `retail_sales.xlsx` | Regression | `weekly_sales` |
| `student_performance.xlsx` | Regression | `final_grade` |
| `loan_approval.xlsx` | Classification | `approved` |
| `energy_consumption.parquet` | Regression | `energy_kwh` |
| `fraud_detection.parquet` | Classification | `is_fraud` |
| `traffic_flow.parquet` | Regression | `vehicle_count` |
| `user_engagement.json` | Regression | `session_duration_sec` |
| `medical_diagnosis.json` | Classification | `diagnosis` |
| `wine_quality.json` | Regression | `quality` |
| `air_quality.txt` | Regression | `aqi` |
| `delivery_time.txt` | Regression | `delivery_days` |
| `crop_yield.txt` | Regression | `yield_tons_ha` |

---

## Supported Models

### Regression (7 models)
- Linear Regression
- Ridge Regression
- Lasso Regression
- Random Forest Regressor
- XGBoost Regressor
- Gradient Boosting Regressor
- Support Vector Regressor (SVR)

### Classification (8 models)
- Logistic Regression
- Random Forest Classifier
- XGBoost Classifier
- Support Vector Machine (SVM)
- K-Nearest Neighbors (KNN)
- Naive Bayes
- Gradient Boosting Classifier
- Decision Tree Classifier

### Time Series (3 models)
- ARIMA
- SARIMA
- Exponential Smoothing

### Clustering (3 models)
- K-Means
- DBSCAN
- Hierarchical Clustering

### Deep Learning (4 models — optional TensorFlow)
- Multi-Layer Perceptron (MLP)
- LSTM (Long Short-Term Memory)
- CNN-1D (1D Convolutional Neural Network)
- Autoencoder

Deep learning models are skipped gracefully if TensorFlow is not installed.

---

## Features

### Data Import
- Drag-and-drop zone or file picker (format-specific filters)
- Automatic data profiling: shape, column types, null counts, numeric ranges
- Preview of the first 5 rows

### Model Training
- Session-based: each browser tab gets its own UUID session
- Automatic preprocessing: null imputation, one-hot encoding, label encoding, feature scaling
- Training time tracked and displayed

### Results & Visualizations
- **Regression**: R², MAE, MSE, RMSE, MAPE; scatter plot (actual vs predicted); residual plot; feature importances
- **Classification**: Accuracy, Precision, Recall, F1, AUC-ROC; confusion matrix; ROC curve; feature importances
- **Clustering**: Silhouette score, number of clusters; PCA 2D cluster scatter plot
- **Time Series**: MSE, RMSE, MAE; actual vs predicted line chart

### Model Comparison
- Pin up to 3 trained models to the comparison panel
- Side-by-side metric table with the best value highlighted in cyan
- Per-model parameter, feature importance, and data tables in the comparison export

### Export
- **Single-model HTML report**: overview, metrics, hyperparameters, feature configuration, and appendix tables (feature importances, actual vs predicted, residuals, ROC data, PCA data). Captions accurately reflect how many rows are shown vs total.
- **Comparison HTML report**: side-by-side metrics + per-model detail cards; best metric value highlighted.
- **JSON snapshot**: raw result object saved locally.

### UI / UX
- Cyberpunk dark theme: neon cyan `#00ffd5`, magenta `#ff006e`, neon green `#39ff14`
- Orbitron + Share Tech Mono fonts
- Ambient matrix rain canvas background (very slow, subtle)
- Custom SVG icon set (no emoji): upload, cpu, hex, sliders, chart, warn, close, check, info
- Light / dark mode toggle (persisted in localStorage)
- Toast notifications (success / error / info) with auto-dismiss
- Step-based Navbar and collapsible Sidebar with progress indicators
- Keyboard-friendly — all controls accessible

---

## Architecture

### Backend (`backend/server.py`)
- **Framework**: FastAPI + Uvicorn
- **Session management**: in-memory dict keyed by UUID; sessions expire after 2 hours
- **Preprocessing pipeline**: pandas for null-fill and encoding; scikit-learn `StandardScaler` or `MinMaxScaler`
- **Endpoints**:

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Liveness check |
| POST | `/upload` | Parse file, store DataFrame, return profile |
| POST | `/train` | Train one model, return metrics + chart data |
| POST | `/compare` | Train up to 3 models, return all results |

- Full API spec: [`docs/API_SPECIFICATION.md`](docs/API_SPECIFICATION.md)

### Frontend (`frontend/src/App.jsx`)
- **Framework**: React 18 + Vite
- **State**: single `useReducer` with an `initialState` object; no external state library
- **Charts**: Recharts (LineChart, ScatterChart, BarChart, ResponsiveContainer)
- **Styling**: 100% inline styles; design token object `t` derived from dark/light mode flag
- **Key components**:
  - `StepImport` — file upload, drag-and-drop, data profile card
  - `StepModel` — model picker with category tabs and quick-filter
  - `StepFeatures` — feature selector and correlation heatmap
  - `StepParams` — hyperparameter form with presets and tooltips
  - `StepResults` — metric cards, all chart panels, comparison table, export buttons
  - `CyberIcon` — inline SVG icon component (9 icons)
  - `CyberpunkBackground` — matrix rain canvas animation
  - `Navbar` / `Sidebar` — step navigation with progress state
  - `ToastContainer` — auto-dismissing notification stack

---

## Project Structure

```
Aditi/
├── backend/
│   ├── server.py            # FastAPI application (single file)
│   └── requirements.txt     # Python dependencies
├── frontend/
│   ├── src/
│   │   └── App.jsx          # React application (single component file)
│   ├── index.html           # HTML entry point
│   ├── package.json         # Node dependencies
│   └── vite.config.js       # Vite configuration
├── docs/
│   ├── API_SPECIFICATION.md # Full API reference
│   ├── MODEL_REFERENCE.md   # Model parameters reference
│   ├── SETUP_GUIDE.md       # Detailed setup instructions
│   ├── sample_data.csv      # Original sample dataset
│   ├── house_prices.csv     # Sample: regression
│   ├── customer_churn.csv   # Sample: classification
│   ├── diabetes_risk.csv    # Sample: classification
│   ├── retail_sales.xlsx    # Sample: regression
│   ├── student_performance.xlsx # Sample: regression
│   ├── loan_approval.xlsx   # Sample: classification
│   ├── energy_consumption.parquet # Sample: regression
│   ├── fraud_detection.parquet    # Sample: classification
│   ├── traffic_flow.parquet       # Sample: regression
│   ├── user_engagement.json   # Sample: regression
│   ├── medical_diagnosis.json # Sample: classification
│   ├── wine_quality.json      # Sample: regression
│   ├── air_quality.txt        # Sample: regression
│   ├── delivery_time.txt      # Sample: regression
│   └── crop_yield.txt         # Sample: regression
└── README.md
```

---

## Configuration

### Ports
| Service | Default URL | Config location |
|---------|------------|----------------|
| Backend | `http://localhost:8000` | Top of `backend/server.py` |
| Frontend | `http://localhost:5173` | `frontend/vite.config.js` |

### CORS
The backend allows requests from `localhost:3000`, `localhost:5173`, and all origins (`*`) for development. Restrict `*` for production deployment.

### TensorFlow (optional)
```bash
pip install tensorflow
```
Without it the app works fully; only MLP, LSTM, CNN-1D, and Autoencoder are unavailable.

---

## Troubleshooting

### Backend not starting
- Check Python version: `python --version` (need 3.8+)
- Install deps: `pip install -r backend/requirements.txt`
- Check port conflict: `netstat -ano | findstr :8000` (Windows) or `lsof -i :8000` (macOS/Linux)
- The app shows a red banner at the top if the backend is unreachable

### Frontend shows blank page
- Ensure backend is running first
- Open browser DevTools (F12) → Console tab for errors
- Try a hard refresh: Ctrl+Shift+R

### File upload fails
- Confirm the file matches a supported format/extension
- For large files (>50 MB) you may need to increase the server body size limit in `server.py`

### TensorFlow errors
- Install: `pip install tensorflow`
- On Apple Silicon: `pip install tensorflow-macos`
- GPU setup is not required; TensorFlow runs on CPU by default

---

## License

MIT License

## Author

Built by Aditi (aditi-py)
