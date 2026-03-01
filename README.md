# HappyModel

A complete no-code machine learning modeling tool. Build, train, and compare ML models without writing code.

## What is HappyModel?

HappyModel is a user-friendly web application that guides you through the entire ML workflow:
1. **Import Data** - Upload CSV, Excel, Parquet, JSON, or TXT files
2. **Select Model** - Choose from 30+ models across regression, classification, time series, clustering, and deep learning
3. **Configure Features** - Pick target variable, features, and view correlations
4. **Set Parameters** - Adjust model hyperparameters with smart presets and plain-English explanations
5. **View Results** - Explore metrics, visualizations, and compare multiple models side-by-side

## Quick Start

### Prerequisites
- Python 3.8+
- Node.js 16+ (for React frontend)

### Installation & Setup

1. **Clone and install Python dependencies:**
   ```bash
   git clone https://github.com/aditi-py/happy-model.git
   cd happy-model
   pip install -r requirements.txt
   ```

2. **Start the backend:**
   ```bash
   python server.py
   ```
   Backend will run at `http://localhost:8000`

3. **In a new terminal, start the frontend:**
   ```bash
   cd frontend
   npm install
   npm run dev
   ```
   Frontend will run at `http://localhost:5173`

4. **Open your browser** and navigate to `http://localhost:5173`

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

### Time Series (3 models)
- ARIMA
- SARIMA
- Exponential Smoothing

### Clustering (3 models)
- K-Means
- DBSCAN
- Hierarchical Clustering

### Deep Learning (4 models - optional TensorFlow)
- Multi-Layer Perceptron (MLP)
- LSTM (Long Short-Term Memory)
- CNN-1D (1D Convolutional Neural Network)
- Autoencoder

## Features

- 🎨 **Beautiful UI** - Light/dark mode with indigo and violet accents
- 📊 **Visualizations** - Charts for results, confusion matrices, ROC curves, feature importance
- ⚡ **Fast Training** - Optimized sklearn, XGBoost, and TensorFlow models
- 🔄 **Model Comparison** - Train up to 3 models and compare side-by-side
- 💾 **Export Results** - Save results as JSON or generate HTML reports
- 📁 **Multiple Formats** - CSV, Excel, Parquet, JSON, and TXT support
- ⚙️ **Smart Defaults** - Quick presets (Conservative, Balanced, Aggressive)
- 🎓 **Educational** - Tooltips explain every parameter in plain English

## Architecture

### Backend (FastAPI)
- Single `server.py` file with all endpoints
- Session-based data storage (in-memory with UUID keys)
- Automatic preprocessing (handling nulls, encoding, scaling)
- Error handling with user-friendly messages

### Frontend (React + Vite)
- Single `App.jsx` component
- 5-step guided workflow
- State management with React hooks
- Recharts for visualizations
- Responsive design (mobile-friendly)

## Supported File Formats

- **CSV** (.csv)
- **Excel** (.xlsx, .xls)
- **Parquet** (.parquet)
- **JSON** (.json)
- **Text** (.txt, tab/comma-separated)

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Backend health check |
| POST | `/upload` | Upload and profile data |
| POST | `/train` | Train a single model |
| POST | `/compare` | Train and compare up to 3 models |

Full API documentation: see [API_SPECIFICATION.md](docs/API_SPECIFICATION.md)

## Deep Learning (Optional)

TensorFlow is optional. If not installed, deep learning models will be skipped gracefully. To use them:

```bash
pip install tensorflow
```

## Configuration

### Port Configuration
- Backend: `http://localhost:8000` (configurable in `server.py`)
- Frontend: `http://localhost:5173` (Vite default, configurable in `frontend/vite.config.js`)

### CORS Settings
Backend accepts requests from:
- `http://localhost:3000` (alternative React dev server)
- `http://localhost:5173` (Vite default)
- `*` (all origins, for testing)

## Project Structure

```
happy-model/
├── backend/
│   ├── server.py           # FastAPI application
│   ├── requirements.txt     # Python dependencies
│   └── start.sh            # Startup script
├── frontend/
│   ├── App.jsx             # Main React component
│   ├── package.json        # Node dependencies
│   └── index.html          # HTML entry point
├── docs/
│   ├── API_SPECIFICATION.md
│   ├── MODEL_REFERENCE.md
│   └── SETUP_GUIDE.md
└── README.md              # This file
```

## Development Notes

- **Backend**: Single self-contained `server.py` (no module splitting)
- **Frontend**: Single `App.jsx` component with inline styling
- **Styling**: Google Fonts (DM Sans, DM Mono) with indigo/violet accent colors
- **Charts**: Recharts library for all visualizations
- **Error Handling**: All API errors caught and displayed as user-friendly toasts

## Troubleshooting

### Backend not running?
- Ensure Python 3.8+ is installed: `python --version`
- Install dependencies: `pip install -r backend/requirements.txt`
- Check if port 8000 is available: `lsof -i :8000` (macOS/Linux) or `netstat -ano | findstr :8000` (Windows)

### Frontend not connecting?
- Check browser console for errors (F12)
- Verify backend is running at `http://localhost:8000/health`
- Clear browser cache if CSS/JS not updating

### TensorFlow issues?
- Optional library; deep learning models will be skipped if not installed
- Install: `pip install tensorflow` (may require additional GPU setup)

## License

MIT License

## Author

Built by Aditi (aditi-py)
