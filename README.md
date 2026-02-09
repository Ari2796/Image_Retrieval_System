# Image Retrieval App

## Project Structure
- `backend/`: FastAPI backend for retrieval logic (Search, OCR, Sketch).
- `frontend/`: React/Vite frontend.
- `legacy/`: Old Streamlit prototype.

## Prerequisites
- Python 3.9+
- Node.js & npm

## How to Run

### 1. Backend (FastAPI)
Navigate to the project root:
```bash
# Install dependencies
pip install -r requirements.txt

# Run the server
python -m uvicorn backend.main:app --reload
```
The API will be available at `http://localhost:8000`.
API Docs: `http://localhost:8000/docs`

### 2. Frontend (React + Vite)
Open a new terminal and navigate to the `frontend` folder:
```bash
cd frontend

# Install dependencies (first time only)
npm install

# Start the development server
npm run dev
```
The UI typically runs at `http://localhost:5173`.

### 3. Legacy App (Optional)
If you need to access the old prototype:
```bash
streamlit run legacy/app.py
```
