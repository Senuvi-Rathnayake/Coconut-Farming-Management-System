# Coconut Farming Management System

A full-stack farm management platform for coconut growers, agricultural officers, and administrators. The system combines day-to-day plantation management with financial, inventory, labor, community, weather, mapping, AI disease-detection, and farming-assistant features.

## Contents

- [Key features](#key-features)
- [System architecture](#system-architecture)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation and setup](#installation-and-setup)
- [Running the application](#running-the-application)
- [Configuration](#configuration)
- [Service and API reference](#service-and-api-reference)
- [Testing and verification](#testing-and-verification)
- [Troubleshooting](#troubleshooting)

## Key features

- **Role-based access:** separate dashboards and workflows for farmers, agricultural officers, and administrators.
- **User accounts:** registration, login, JWT authentication, Google authentication, email verification, password reset, profile management, and administrative user approval.
- **Plantation management:** register land, capture map coordinates and area, track land status, manage fertilizer schedules, record fertilizer applications, and maintain harvest records.
- **Field visits:** farmers can request visits, officers can accept or reject them, and both parties can follow the visit workflow.
- **Financial management:** income and expense records, dashboard summaries, category breakdowns, monthly comparisons, and coconut market-price history.
- **Inventory management:** stock items, low-stock monitoring, suppliers, purchases, and usage tracking.
- **Labor management:** workers, tasks, work logs, task categories, productivity metrics, completion rates, labor costs, and work distribution.
- **Community portal:** articles, categories, comments, likes, moderation, dashboards, and agricultural news.
- **Weather and location tools:** current weather, forecasts, coconut-growing advice, saved GPS locations, and map links.
- **AI disease detection:** an EfficientNet-based TensorFlow model identifies coconut leaf conditions and stores prediction history with treatment guidance.
- **AI farming assistant:** an English/Sinhala retrieval-augmented chatbot backed by coconut-farming documents, ChromaDB, Hugging Face embeddings, and Groq-hosted Llama.
- **Voice input:** local Whisper speech-to-text support for chatbot questions.
- **Reports and notifications:** administrative summaries, reports, and in-app notifications.

## System architecture

```text
Browser
  |
  |  http://localhost:3000
  v
React + Vite frontend
  |-- /api/* ----------> Spring Boot API (http://localhost:8080)
  |                         |-- MySQL (localhost:3306/coconut_db)
  |                         |-- OpenWeatherMap
  |                         |-- GNews
  |                         |-- Coconut Development Authority market data
  |                         `--> Disease model API (http://localhost:8001)
  |
  `-- /chatbot/* -----> FastAPI chatbot (http://localhost:8000)
                            |-- Chroma vector database
                            |-- Hugging Face embeddings
                            |-- Groq Llama API
                            `-- Whisper + FFmpeg
```

The Vite development server proxies backend and chatbot requests, allowing the browser to use relative `/api` and `/chatbot` URLs during local development.

## Technology stack

| Layer | Technologies |
| --- | --- |
| Frontend | React 19, Vite 8, React Router, Material UI, Axios, Leaflet, Turf, Chart.js, Recharts, jsPDF |
| Main backend | Java 17, Spring Boot 3.2.5, Spring Web, Spring Security, Spring Data JPA, Spring Mail, JWT, Maven |
| Database | MySQL, Hibernate/JPA |
| Disease detection | Python, FastAPI, TensorFlow/Keras, EfficientNet model, NumPy, Pillow |
| Chatbot | Python, FastAPI, Groq, Llama 3.3, LangChain, ChromaDB, Hugging Face embeddings |
| Voice processing | OpenAI Whisper, FFmpeg |
| External data | OpenWeatherMap, GNews, Coconut Development Authority website |

## Project structure

```text
.
|-- package.json                         # Starts and coordinates all services
|-- Coconut-Farming-System/
|   |-- Backend/                         # Spring Boot REST API
|   |   |-- pom.xml
|   |   |-- src/main/java/com/coconutfarming/
|   |   |   |-- admin/
|   |   |   |-- community/
|   |   |   |-- disease/
|   |   |   |-- financial/
|   |   |   |-- gps/
|   |   |   |-- inventory/
|   |   |   |-- labor/
|   |   |   |-- notification/
|   |   |   |-- plantation/
|   |   |   |-- user/
|   |   |   |-- visit/
|   |   |   `-- weather/
|   |   `-- src/main/resources/application.properties
|   `-- Frontend/frontend/               # React web application
|       |-- src/api/
|       |-- src/components/
|       |-- src/context/
|       `-- src/pages/
|-- Coconut_Disease_Detection/           # TensorFlow prediction API
|   |-- main.py
|   |-- models/
|   `-- notebooks/
`-- CoconutChatbot/                      # RAG and voice-enabled chatbot API
    |-- app.py
    |-- coconut_db/                      # Included Chroma vector database
    |-- data/                            # Coconut reference documents
    |-- member1/                         # Retrieval and knowledge pipeline
    |-- member2/                         # LLM and evaluation logic
    `-- member3/                         # API and voice routes
```

## Prerequisites

Install the following before starting the project:

- **Node.js 20.19+** and npm (required by the included Vite version)
- **Java Development Kit 17**
- **MySQL 8.x**
- **Python 3.11** (recommended for the existing AI stack)
- **FFmpeg** available on the system `PATH` for voice transcription
- API credentials for:
  - [Groq](https://console.groq.com/) for chatbot responses
  - [OpenWeatherMap](https://openweathermap.org/api) for weather data
  - [GNews](https://gnews.io/) for agricultural news
  - A Google OAuth client if Google sign-in is required
- SMTP credentials if email verification and password-reset emails are required

The first chatbot startup may download the Hugging Face embedding model and the Whisper `base` model, so an internet connection and additional disk space are required.

## Installation and setup

### 1. Open the main project directory

```bash
cd Coconut-Farming-Management-System
```

All commands in this README assume this directory is the current working directory.

### 2. Install JavaScript dependencies

Install the root process manager and the frontend dependencies:

```bash
npm install
npm run install:all
```

### 3. Create and activate a Python virtual environment

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Install Python dependencies

The repository does not currently contain Python requirements files. Install the packages used by both Python services:

```bash
python -m pip install --upgrade pip
pip install fastapi "uvicorn[standard]" python-multipart pydantic tensorflow numpy pillow
pip install groq langchain-huggingface langchain-chroma langchain-community langchain-text-splitters sentence-transformers pdfminer.six openai-whisper
```

Optional local disease-model interface:

```bash
pip install gradio
```

### 5. Configure MySQL and the Spring Boot application

1. Start MySQL.
2. Open `Coconut-Farming-System/Backend/src/main/resources/application.properties`.
3. Set `spring.datasource.username` and `spring.datasource.password` for your MySQL server.
4. Keep the configured database URL or change it to the desired database.

The default URL uses the database `coconut_db` and includes `createDatabaseIfNotExist=true`. Hibernate is configured with `ddl-auto=update`, so the application creates or updates mapped tables when the backend starts, provided the MySQL user has the necessary permissions.

SQL schema files are also available under `Coconut-Farming-System/Backend/` and `Coconut-Farming-System/Backend/src/main/resources/` for reference or manual initialization.

### 6. Configure API and email credentials

Update the relevant values in `application.properties`:

```properties
weather.api.key=YOUR_OPENWEATHERMAP_KEY
news.api.key=YOUR_GNEWS_KEY

spring.mail.host=YOUR_SMTP_HOST
spring.mail.port=YOUR_SMTP_PORT
spring.mail.username=YOUR_SMTP_USERNAME
spring.mail.password=YOUR_SMTP_PASSWORD

google.client.id=YOUR_GOOGLE_CLIENT_ID
```

Set the Groq credential in the terminal used to start the application.

Windows PowerShell:

```powershell
$env:GROQ_API_KEY="YOUR_GROQ_API_KEY"
```

macOS/Linux:

```bash
export GROQ_API_KEY="YOUR_GROQ_API_KEY"
```

Do not commit real database passwords, SMTP passwords, or API keys. If any credential has previously been committed or shared, rotate it before using the project.

## Running the application

### Run all services together

Activate the Python virtual environment, confirm MySQL is running, and then execute:

```bash
npm run dev
```

This starts the frontend, Spring Boot backend, chatbot API, and disease-detection API concurrently.

Open [http://localhost:3000](http://localhost:3000) in a browser.

### Run services separately

Running services separately is useful for debugging. Use a separate terminal for each service.

Frontend:

```bash
npm run dev:frontend
```

Spring Boot backend:

```bash
npm run dev:backend
```

On macOS/Linux, if the root backend script cannot locate the Maven wrapper, run it directly:

```bash
cd Coconut-Farming-System/Backend
./mvnw spring-boot:run
```

Chatbot API:

```bash
npm run dev:chatbot
```

Disease-detection API:

```bash
npm run dev:model
```

## Configuration

### Local service ports

| Service | Default URL | Purpose |
| --- | --- | --- |
| React frontend | `http://localhost:3000` | Web user interface |
| Spring Boot backend | `http://localhost:8080` | Main REST API and uploaded images |
| Chatbot API | `http://localhost:8000` | RAG chat and voice transcription |
| Disease model API | `http://localhost:8001` | Coconut leaf image prediction |
| MySQL | `localhost:3306` | Persistent application data |

If these ports change, update the Vite proxy configuration, Spring application URLs/CORS settings, and the disease-service URL used by the Spring backend.

### Important Spring properties

| Property | Description |
| --- | --- |
| `spring.datasource.*` | MySQL connection URL and credentials |
| `weather.api.key` | OpenWeatherMap API credential |
| `news.api.key` | GNews API credential |
| `spring.mail.*` | SMTP settings for verification and password-reset email |
| `google.client.id` | Google OAuth client ID |
| `app.cors.allowed-origins` | Allowed frontend development origins |
| `app.base-url` | Public backend base URL used in generated links |
| `app.frontend-url` | Frontend URL used for redirects |
| `spring.servlet.multipart.*` | Upload limits; currently configured for files up to 10 MB |

### AI resources

- The trained Keras disease model is included at `Coconut_Disease_Detection/models/coconut_disease_efficientnet.keras`.
- The disease API resizes uploaded images to `224 x 224` pixels before inference.
- The chatbot's Chroma database is included at `CoconutChatbot/coconut_db`.
- Source knowledge documents are stored in `CoconutChatbot/data`.
- To rebuild the chatbot vector database after changing its PDF or text sources, activate the Python environment and run:

```bash
cd CoconutChatbot
python member1/member1_rag.py
```

## Service and API reference

### Main backend API groups

The Spring Boot service exposes REST endpoints under these main paths:

| Area | Base path |
| --- | --- |
| Authentication | `/api/auth` |
| Profiles and users | `/api/profile`, `/api/admin` |
| Plantation and land | `/api/lands` |
| Harvests and fertilizer | `/api/harvests`, `/api/fertilizers` |
| Field visits | `/api/visits` |
| Finance and market prices | `/api/financial` |
| Inventory, suppliers, purchases, usage | `/api/inventory`, `/api/suppliers`, `/api/purchases`, `/api/usage` |
| Labor and analytics | `/api/workers`, `/api/tasks`, `/api/work-logs`, `/api/analytics` |
| Community content | `/api/articles`, `/api/categories`, `/api/comments`, `/api/news` |
| Weather and GPS | `/api/weather`, `/api/gps` |
| Disease detection and history | `/api/disease` |
| Notifications | `/api/notifications` |
| Admin dashboard and reports | `/api/admin/dashboard`, `/api/admin/reports` |

Most authenticated frontend requests send a JWT in the `Authorization: Bearer <token>` header. Administrative frontend routes require the `ADMIN` role, officer land-oversight routes require `OFFICER`, and AI disease detection is exposed to `ADMIN` and `FARMER` users.

### Disease model API

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/predict` | Accepts a multipart image field named `file` and returns the predicted condition and confidence |

Interactive FastAPI documentation is available at [http://localhost:8001/docs](http://localhost:8001/docs) while the service is running.

The model currently recognizes healthy leaves and several Coconut Caterpillar Infestation (CCI) and Weligama Coconut Leaf Wilt Disease (WCLWD) symptom classes.

### Chatbot API

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/health` | Chatbot health check |
| `POST` | `/chat/llama` | Answers a coconut-farming question using RAG and Llama |
| `POST` | `/chat/qwen` | Compatibility endpoint currently routed to the same Llama pipeline |
| `POST` | `/compare` | Returns the compatibility responses together |
| `POST` | `/voice/transcribe` | Transcribes an uploaded audio field named `audio` |

Chat requests use this JSON shape:

```json
{
  "query": "What fertilizer should I use for a mature coconut tree?"
}
```

Interactive FastAPI documentation is available at [http://localhost:8000/docs](http://localhost:8000/docs) while the service is running.

## Testing and verification

### Frontend production build

```bash
cd Coconut-Farming-System/Frontend/frontend
npm run build
```

### Backend tests

Windows:

```powershell
cd Coconut-Farming-System\Backend
.\mvnw.cmd test
```

macOS/Linux:

```bash
cd Coconut-Farming-System/Backend
./mvnw test
```

### Service health checks

After starting all services:

```bash
curl http://localhost:8000/health
curl http://localhost:8080/api/weather?city=Colombo
```

The weather request requires a valid OpenWeatherMap key.

To test the model API with an image:

```bash
curl -X POST http://localhost:8001/predict -F "file=@path/to/coconut-leaf.jpg"
```

An optional Gradio model tester is also included:

```bash
cd Coconut_Disease_Detection
python test_local.py
```

## Troubleshooting

### The backend cannot connect to MySQL

- Confirm the MySQL service is running on port `3306`.
- Verify the database username and password in `application.properties`.
- Ensure the MySQL user can create and update the configured database.

### The frontend opens but API requests fail

- Confirm the Spring backend is running on port `8080`.
- Confirm the chatbot is running on port `8000`.
- Start the frontend through Vite on port `3000` so its development proxies are active.

### Disease detection fails

- Confirm the model API is running on port `8001`.
- Start it from the repository root through `npm run dev:model`, or from `Coconut_Disease_Detection` so its relative model path resolves correctly.
- Confirm TensorFlow can load `models/coconut_disease_efficientnet.keras`.

### The chatbot fails during startup

- Confirm `GROQ_API_KEY` is set in the same terminal session.
- Allow time for the embedding and Whisper models to download on the first run.
- Confirm `CoconutChatbot/coconut_db` exists.
- Check that the installed Python version is compatible with TensorFlow, PyTorch, and Whisper.

### Voice transcription is unavailable

- Install FFmpeg and ensure the `ffmpeg` executable is on the system `PATH`.
- Confirm `openai-whisper` is installed in the active virtual environment.
- Grant microphone permission to the browser.

## License

No license file is currently included in this repository. Add a license before distributing or reusing the project outside its intended context.
