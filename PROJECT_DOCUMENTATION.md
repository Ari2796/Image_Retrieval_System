# Jewellery Retrieval App - Project Documentation

## 1. Project Overview
The **Jewellery Retrieval App** is an AI-powered search engine designed to find jewellery items using multiple retrieval methods. Unlike traditional keyword search, this application leverages advanced machine learning models to understand images, sketches, voice commands, and natural language queries.

### Key Features
-   **Text Search**: "Find me a gold necklace with rubies".
-   **Image Search**: Upload a photo to find visually similar items.
-   **Sketch Search**: Draw a rough shape to find matching designs.
-   **Voice Search**: Speak your query, and the app will transcribe and search.
-   **OCR & Vision Analysis**: Extract text from images or use AI Vision to identify product types from photos.
-   **Dark Mode**: A modern, system-aware UI with light and dark themes.

---

## 2. Workflows

### A. Search Workflow (Text/Image/Sketch)
1.  **Input**: The user provides an input (Text query, Image file, or Sketch blob).
2.  **Embedding Generation**:
    -   The backend uses **CLIP (Contrastive Language-Image Pre-training)** to convert the input into a 512-dimensional vector (embedding).
    -   For Text: `CLIP Text Encoder`.
    -   For Image/Sketch: `CLIP Image Encoder`.
3.  **Similarity Search**:
    -   The generated embedding is compared against a pre-computed database of jewellery embeddings using **FAISS (Facebook AI Similarity Search)**.
    -   FAISS uses **IndexFlatIP** (Inner Product) or **IndexFlatL2** (Euclidean Distance) to efficiently find the "nearest neighbors" in the high-dimensional vector space.
    -   Current Configuration: `Top-K = 30` items are retrieved.
4.  **Retrieval**: The app returns the top matching images and their metadata.

### B. Sketch Search Workflow (Deep Dive)
**Goal**: Find a real jewellery items that visually matches a user's rough drawing.

**Challenge**: A sketch domain (black/white lines) is very different from a photo domain (color, texture, lighting). Direct comparison often fails.

**Solution: Hybrid Retrieval & Domain Adaptation**
1.  **Preprocessing (The "Scanner" Effect)**:
    -   User's sketch is often messy (shadows, paper texture).
    -   We use `cv2.adaptiveThreshold` to binarize the image (pure black & white), removing noise.
    -   We resize it to 224x224 (CLIP's native resolution) and center it on a white canvas.
2.  **Dual-Path Search**:
    -   **Path A: Semantic Understanding (Text)**: 
        -   We send the sketch image to a **Vision LLM** (e.g., GPT-4.1-nano) to *describe* it. 
        -   Example Output: "A solitaire diamond ring with a thin band".
        -   We then run a **Text Search** using this description to find items that match the *concept*.
    -   **Path B: Visual Shape Match (Image)**:
        -   We generate a CLIP embedding of the processed sketch.
        -   We explicitly compare this against a special **Sketch Index**.
        -   *Note*: To build this index, we pre-processed the original database photos using a **"Photo-to-Sketch"** algorithm (Invert -> Gaussian Blur -> Color Dodge Blend). This forces the database to "look like" sketches, bridging the domain gap.
3.  **Hybrid Fusion**:
    -   We merge results from Path A (Text/Concept) and Path B (Shape).
    -   Duplicate IDs are combined.
4.  **Reranking**:
    -   The combined list is reranked against the generated text description to ensure the best matches appear first.

### C. Voice Search Workflow
1.  **Recording**: The user records audio in the browser.
2.  **Transcription**:
    -   Audio is sent to the backend.
    -   **OpenAI Whisper** (running locally via `transformers`) processes the audio.
    -   `ffmpeg` is used to decode the audio stream.
3.  **Search**: The transcribed text is fed into the Text Search Workflow.

### D. OCR & Vision Workflow
1.  **Standard OCR**:
    -   Uses **TrOCR (Transformer OCR)** to read handwriting/text from an image.
    -   An LLM (GPT-4) refines the raw text.
2.  **AI Vision (LLM Mode)**:
    -   The image is sent directly to a Vision-capable LLM (e.g., GPT-4o).
    -   The model analyzes the image to extract text, identify the object type (Ring, Earring, etc.), and infer intent.

---

## 3. Technology Stack

### Frontend
-   **Framework**: [React](https://react.dev/) + [Vite](https://vitejs.dev/) - Fast, modern UI development.
-   **Styling**: [Tailwind CSS](https://tailwindcss.com/) - Utility-first styling with Dark Mode support.
-   **Animations**: [Framer Motion](https://www.framer.com/motion/) - Smooth transitions and layout animations.
-   **Icons**: [Lucide React](https://lucide.dev/) - Clean, consistent iconography.
-   **HTTP Client**: [Axios](https://axios-http.com/) - API requests.
-   **Canvas**: `react-sketch-canvas` - For the sketching interface.

### Backend
-   **Server**: [FastAPI](https://fastapi.tiangolo.com/) - High-performance Python web framework.
-   **ML/AI Models**:
    -   **CLIP (OpenAI/HuggingFace)**: Embeddings for semantic search.
    -   **Whisper (OpenAI/HuggingFace)**: Speech-to-text.
    -   **TrOCR (Microsoft/HuggingFace)**: Optical Character Recognition.
    -   **FAISS (Meta)**: Vector database for fast similarity search.
-   **Data Processing**:
    -   `Pillow (PIL)`: Image manipulation.
    -   `NumPy`: Vector math operations.
    -   `imageio-ffmpeg`: Audio decoding.
-   **LLM Integration**: `OpenAI API` - For query refinement and refined OCR.

---

## 4. File Structure & Explanation

### Backend (`/backend`)

#### Core Files
-   **`main.py`**: The entry point of the FastAPI application.
    -   Defines API routes (`/search/text`, `/search/image`, `/voice/transcribe`, etc.).
    -   Handles server lifecycle (loading models, initializing FAISS index on startup).
    -   Manages static file serving for images.
-   **`config.py`**: Configuration settings.
    -   Loads environment variables (`.env`).
    -   Defines paths for data, indexes, and uploads.
    -   Sets constants like `DEVICE` (CPU/CUDA) and Model IDs.
-   **`schemas.py`**: Pydantic models.
    -   Defines the expected structure of Request and Response bodies (e.g., `SearchResult`, `TextSearchRequest`) to ensure type safety.

#### Search Module (`/backend/search`)
-   **`image_search.py`**:
    -   Contains logic to search using an image file.
    -   Generates embedding for the uploaded image and queries FAISS.
-   **`sketch_search.py`**:
    -   Handles sketch inputs. Sketches are often inverted or pre-processed before being embedded to match the style of natural images better.

#### AI & Processing (`/backend/models`, `/backend/ocr`, `/backend/voice`)
-   **`models/clip.py`**:
    -   Loads the CLIP model.
    -   Provides functions to generate embeddings for text and images (`get_text_embedding`, `get_image_embedding`).
-   **`ocr/ocr_pipeline.py`**:
    -   Manages the OCR workflow.
    -   `extract_text_from_image`: Uses TrOCR model.
    -   `llm_refine_ocr_text`: Uses GPT to clean up OCR errors.
    -   `extract_text_with_llm_vision`: Direct Vision LLM implementation.
-   **`voice/transcriber.py`**:
    -   Loads the Whisper model.
    -   `transcribe_audio`: Converts audio files to text. Handles `ffmpeg` path injection.

#### Utilities (`/backend/utils`)
-   **`captioning.py`**: 
    -   Uses a Vision LLM to generate descriptive captions for images ("A gold ring with a ruby stone"). 
    -   These captions are essential for text search accuracy because they provide the "ground truth" for what an image contains.
-   **`sketch_utils.py`**: 
    -   **`photo_to_sketch_database`**: The core "Domain Adaptation" function. It converts real photos into realistic pencil sketches using `cv2.divide(gray, 255-blurred, scale=256)`. This creates the "Sketch Index".
    -   **`preprocess_sketch`**: Cleans up user uploads using `cv2.adaptiveThreshold` to ensure they look like the database sketches.
-   **`reranker.py`**: 
    -   Implements logic to re-order search results. It might use a Cross-Encoder or simply re-measure cosine similarity between the query and the retrieved candidates to improve precision.

### Frontend (`/frontend/src`)

#### Core Components
-   **`main.jsx`**: The React entry point. Mounts the execution to the DOM.
-   **`App.jsx`**: The main application component.
    -   Manages global state (`theme`, `ocrMode`, `results`).
    -   Handles routing/tabs (Search, Scan, Sketch).
    -   Contains the main layout structure (Navbar params, Hero section).
-   **`index.css`**: Global styles and Tailwind configuration. Includes the `:root` and `.dark` theme variables.

#### Sub-Components (`/frontend/src/components`)
-   **`SketchPad.jsx`**: A wrapper around `react-sketch-canvas`. Provides tools (Pen, Eraser, Undo) and handles exporting the drawing as a blob for search.
-   **`VoiceModal.jsx`**: A modal interface for recording audio. Visualizes volume levels and handles the API call to `/voice/transcribe`.

#### Configuration
-   **`vite.config.js`**: Vite build configuration. Sets up proxying (if needed) and plugin support.
-   **`tailwind.config.js`** (or v4 equivalent in CSS): Configures the design system, colors, and content paths for tree-shaking.
