# Image Retrieval System

A sophisticated, AI-powered jewellery search engine that allows users to find their favorite pieces using multiple search modalities: text, images, sketches, and voice.

---

##  Key Features

- *🔍 Multi-Modal Search*: Find jewellery using any of the following methods:
  - *Natural Language*: Search using descriptive text (e.g., "Golden ring with diamond").
  - *Image Search*: Upload a photo to find similar-looking jewellery.
  - *Sketch Search*: Draw a rough sketch and let the AI find matching pieces.
  - *OCR Search*: Upload an image containing text to extract and search for products.
  - *Voice Search*: Search by simply speaking your query.
- *🖼️ Visual Intelligence: Powered by OpenAI's **CLIP* model for deep visual understanding.
- *⚡ High-Performance Retrieval: Uses **FAISS* (Facebook AI Similarity Search) for lightning-fast indexing and searching.
- *🎨 Premium UI*: A luxurious "Midnight Gold" themed interface built with React and Tailwind CSS.

---

## 🛠️ Tech Stack

### Backend
- *Framework*: FastAPI (Python)
- *AI Models*: CLIP (OpenAI), TrOCR (OCR), BLIP (Captioning)
- *Database/Index*: FAISS (Visual & Sketch Indices)
- *Audio Processing*: OpenAI Whisper (via local transcriber)

### Frontend
- *Library*: React.js (Vite)
- *Styling*: Tailwind CSS 4.0
- *Animations*: Framer Motion
- *Icons*: Lucide React
- *Drawing*: React Sketch Canvas

---

## 📁 Project Structure

text
jewellery_search/
├── backend/            # FastAPI Backend
│   ├── models/         # AI Model loading (CLIP, etc)
│   ├── ocr/            # OCR Pipeline
│   ├── search/         # Search logic (Image, Sketch, Text)
│   ├── utils/          # Image and captioning utilities
│   └── voice/          # Voice transcription
├── frontend/           # React + Vite Frontend
│   ├── src/
│   │   ├── components/ # UI Components (SketchPad, VoiceModal)
│   │   └── App.jsx     # Main Application logic
├── data/               # Product Images (excluded from git)
├── indexes/            # FAISS indices (excluded from git)
└── requirements.txt    # Python dependencies


---

## 📦 Getting Started

### 1. Prerequisites
- Python 3.9+
- Node.js 18+
- [FFMPEG](https://ffmpeg.org/) (Required for voice search)

### 2. Backend Setup
bash
# Navigate to root
pip install -r requirements.txt

# Start the server
python backend/main.py


### 3. Frontend Setup
bash
cd frontend
npm install
npm run dev


---

## 📝 License
This project is for academic/research purposes. 

---
