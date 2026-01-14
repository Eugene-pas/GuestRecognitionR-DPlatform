# Технічне завдання: Guest Recognition R&D Platform

**Версія:** 1.0  
**Дата:** 13 січня 2026  
**Тип проєкту:** R&D / Технодемо  
**Статус:** Draft  

---

## Зміст

1. [Загальний опис системи](#1-загальний-опис-системи)
2. [Архітектура](#2-архітектура)
3. [Технологічний стек](#3-технологічний-стек)
4. [Детальний опис модулів](#4-детальний-опис-модулів)
5. [Діаграми](#5-діаграми)
6. [Етапи реалізації](#6-етапи-реалізації-roadmap)
7. [Ризики та обмеження](#7-ризики-та-обмеження)
8. [Deliverables](#8-deliverables)

---

## 1. Загальний опис системи

### 1.1 Мета проєкту

Розробка дослідницького технодемо для готельного бізнесу, що забезпечує:

- Автоматизований збір фото гостей та їхніх документів
- OCR-розпізнавання документів різних країн
- Підготовку інфраструктури для face recognition
- Дослідження можливостей edge AI на базі NVIDIA Dev Kit

### 1.2 Scope проєкту

| Аспект | Опис |
|--------|------|
| **Тип** | R&D, не production |
| **Середовище** | Внутрішня мережа готелю |
| **Користувачі** | Персонал ресепшну (до 5 одночасних) |
| **Масштаб** | Single-location pilot |
| **Дані** | Персональні дані гостей (GDPR considerations для майбутнього) |

### 1.3 Обмеження

- **Не є production-ready системою** — відсутня повна обробка помилок, SLA, disaster recovery
- **Cloud OCR dependency** — на етапі R&D використовується зовнішній API
- **Без real-time вимог** — допустима затримка обробки до 30 секунд
- **Обмежений hardware** — NVIDIA Dev Kit буде доступний на пізніших етапах
- **Внутрішня мережа** — система не експонується в інтернет

### 1.4 Ключові терміни

| Термін | Визначення |
|--------|------------|
| **Guest Record** | Сукупність даних про гостя: фото, документи, OCR-результати |
| **Face Embedding** | Векторне представлення обличчя (128-512 dim) |
| **Ingestion** | Процес отримання та первинної обробки вхідних даних |
| **Edge AI** | Виконання ML inference на локальному пристрої |

---

## 2. Архітектура

### 2.1 Високорівнева архітектура

Система складається з таких компонентів:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              INGESTION LAYER                                │
├─────────────────────────────────┬───────────────────────────────────────────┤
│       Mail Ingestion            │           Web Ingestion                   │
│    (Internal Mail Server)       │         (React Web App)                   │
└─────────────────────────────────┴───────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            PROCESSING LAYER                                 │
├─────────────────────┬─────────────────────┬─────────────────────────────────┤
│   File Processor    │    OCR Service      │      Face Processor             │
│   (Validation,      │   (Cloud API        │    (Detection, Cropping,        │
│    Normalization)   │    Integration)     │     Embedding Prep)             │
└─────────────────────┴─────────────────────┴─────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             STORAGE LAYER                                   │
├─────────────────────┬─────────────────────┬─────────────────────────────────┤
│   PostgreSQL        │   File Storage      │   Vector DB (Future)            │
│   (Metadata,        │   (Raw images,      │   (Face embeddings,             │
│    OCR results)     │    documents)       │    similarity search)           │
└─────────────────────┴─────────────────────┴─────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           FUTURE: EDGE LAYER                                │
├─────────────────────────────────────────────────────────────────────────────┤
│   NVIDIA Dev Kit + Camera                                                   │
│   (Real-time face detection, recognition, tracking)                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Backend Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        API Gateway                              │
│                    (FastAPI / nginx)                            │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
       ┌───────────┐   ┌───────────┐   ┌───────────┐
       │  Guest    │   │  Document │   │  Face     │
       │  Service  │   │  Service  │   │  Service  │
       └───────────┘   └───────────┘   └───────────┘
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                    ┌─────────────────┐
                    │  Task Queue     │
                    │  (Celery/Redis) │
                    └─────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
       ┌───────────┐   ┌───────────┐   ┌───────────┐
       │  OCR      │   │  Face     │   │  File     │
       │  Worker   │   │  Worker   │   │  Worker   │
       └───────────┘   └───────────┘   └───────────┘
```

### 2.3 Ingestion Pipeline

#### Mail Ingestion Flow

```
Mail Server → IMAP Polling → Attachment Extraction → Validation → Queue → Processing
```

#### Web Ingestion Flow

```
Browser Camera/Upload → API Endpoint → Validation → Queue → Processing
```

### 2.4 Edge vs Cloud Distribution

| Компонент | Поточний етап (Cloud) | Майбутній етап (Edge) |
|-----------|----------------------|----------------------|
| OCR | Cloud API (Azure/Google) | Cloud API (залишається) |
| Face Detection | Cloud/Local (CPU) | NVIDIA Dev Kit (GPU) |
| Face Recognition | Не реалізовано | NVIDIA Dev Kit |
| File Storage | Local/NAS | Local/NAS |
| Database | Local PostgreSQL | Local PostgreSQL |
| Video Stream | — | NVIDIA Dev Kit + Camera |

---

## 3. Технологічний стек

### 3.1 Backend

| Компонент | Технологія | Обґрунтування |
|-----------|-----------|---------------|
| **Runtime** | Python 3.11+ | Екосистема ML/CV, швидка розробка |
| **Web Framework** | FastAPI | Async, автодокументація, типізація |
| **Task Queue** | Celery + Redis | Async processing, retry logic |
| **ORM** | SQLAlchemy 2.0 | Type hints, async support |
| **Validation** | Pydantic v2 | Data validation, serialization |

### 3.2 Frontend

| Компонент | Технологія | Обґрунтування |
|-----------|-----------|---------------|
| **Framework** | React 18+ | Component model, ecosystem |
| **Build Tool** | Vite | Fast HMR, modern bundling |
| **UI Library** | Tailwind CSS + shadcn/ui | Rapid prototyping |
| **Camera API** | MediaDevices API | Browser-native, no plugins |
| **State** | Zustand / React Query | Lightweight, server state |

### 3.3 OCR

| Компонент | Технологія | Обґрунтування |
|-----------|-----------|---------------|
| **Primary** | Azure AI Document Intelligence | MRZ support, multi-language |
| **Fallback** | Google Cloud Vision API | Alternative, comparison |
| **Local (future)** | Tesseract + EasyOCR | Offline capability research |

### 3.4 ML / Computer Vision

| Компонент | Технологія | Обґрунтування |
|-----------|-----------|---------------|
| **Face Detection** | MTCNN / RetinaFace | Accuracy, Python native |
| **Face Embeddings** | FaceNet / ArcFace | Standard embeddings |
| **Image Processing** | OpenCV, Pillow | Standard tools |
| **ML Framework** | PyTorch | NVIDIA compatibility |

### 3.5 Storage

| Компонент | Технологія | Обґрунтування |
|-----------|-----------|---------------|
| **RDBMS** | PostgreSQL 15+ | JSON support, reliability |
| **File Storage** | Local FS / MinIO | S3-compatible, local |
| **Vector DB (future)** | pgvector / Milvus | Embedding similarity search |
| **Cache** | Redis | Session, task broker |

### 3.6 Infrastructure

| Компонент | Технологія | Обґрунтування |
|-----------|-----------|---------------|
| **Containerization** | Docker + Docker Compose | Reproducibility |
| **Reverse Proxy** | nginx | Static files, routing |
| **Monitoring** | Prometheus + Grafana | Metrics, dashboards |
| **Logging** | Loki / ELK (simplified) | Centralized logs |

### 3.7 NVIDIA Stack (майбутнє)

| Компонент | Технологія | Призначення |
|-----------|-----------|-------------|
| **Hardware** | NVIDIA Jetson Orin | Edge AI compute |
| **SDK** | JetPack 6.x | Base OS, CUDA, TensorRT |
| **Inference** | TensorRT | Optimized inference |
| **Video** | DeepStream SDK | Video analytics pipeline |
| **Models** | TAO Toolkit | Model customization |

---

## 4. Детальний опис модулів

### 4.1 Mail Ingestion Module

#### 4.1.1 Опис

Сервіс для автоматичного отримання вкладень з внутрішнього поштового сервера.

#### 4.1.2 Функціональні вимоги

| ID | Вимога | Пріоритет |
|----|--------|-----------|
| MI-01 | Підключення до внутрішнього IMAP сервера | Must |
| MI-02 | Polling з налаштовуваним інтервалом (default: 30s) | Must |
| MI-03 | Витяг вкладень форматів: JPEG, PNG, PDF | Must |
| MI-04 | Валідація розміру файлу (max 20MB) | Must |
| MI-05 | Валідація MIME-типу | Must |
| MI-06 | Маркування оброблених листів | Should |
| MI-07 | Логування всіх операцій | Must |
| MI-08 | Retry logic при помилках з'єднання | Should |

#### 4.1.3 Конфігурація

```yaml
mail_ingestion:
  host: "internal-mail.local"
  port: 993
  protocol: "IMAP"
  use_ssl: true
  polling_interval_seconds: 30
  inbox_folder: "INBOX"
  processed_folder: "Processed"
  max_attachment_size_mb: 20
  allowed_mime_types:
    - "image/jpeg"
    - "image/png"
    - "application/pdf"
```

#### 4.1.4 Інтерфейс

```python
class MailIngestionService:
    async def start_polling() -> None
    async def stop_polling() -> None
    async def process_message(message_id: str) -> IngestionResult
    async def extract_attachments(message: Message) -> list[Attachment]
    async def validate_attachment(attachment: Attachment) -> ValidationResult
```

#### 4.1.5 Error Handling

| Помилка | Дія |
|---------|-----|
| Connection timeout | Retry з exponential backoff (max 5 attempts) |
| Invalid attachment | Log warning, skip attachment, continue |
| Storage failure | Retry 3 times, then dead-letter queue |

---

### 4.2 Web Ingestion Module

#### 4.2.1 Опис

React-застосунок для персоналу ресепшну з можливістю:
- Введення даних гостя
- Фотографування через камеру браузера
- Завантаження файлів документів

#### 4.2.2 Функціональні вимоги

| ID | Вимога | Пріоритет |
|----|--------|-----------|
| WI-01 | Форма введення даних гостя | Must |
| WI-02 | Доступ до камери браузера | Must |
| WI-03 | Capture фото в реальному часі | Must |
| WI-04 | Завантаження файлів (drag & drop) | Must |
| WI-05 | Preview завантажених/сфотографованих зображень | Must |
| WI-06 | Валідація на клієнті (формат, розмір) | Must |
| WI-07 | Індикатор статусу обробки | Should |
| WI-08 | Історія останніх записів | Should |
| WI-09 | Offline-режим з sync (future) | Could |

#### 4.2.3 UI Components

```
┌─────────────────────────────────────────────────────────────┐
│  Guest Registration                              [History]  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────────────────────────┐  │
│  │                 │  │  Guest Information              │  │
│  │   Camera Feed   │  │  ┌─────────────────────────┐    │  │
│  │   / Preview     │  │  │ First Name              │    │  │
│  │                 │  │  └─────────────────────────┘    │  │
│  │  [📷 Capture]   │  │  ┌─────────────────────────┐    │  │
│  │                 │  │  │ Last Name               │    │  │
│  └─────────────────┘  │  └─────────────────────────┘    │  │
│                       │  ┌─────────────────────────┐    │  │
│  ┌─────────────────┐  │  │ Room Number             │    │  │
│  │                 │  │  └─────────────────────────┘    │  │
│  │  Document       │  └─────────────────────────────────┘  │
│  │  Upload Zone    │                                       │
│  │  [Drag & Drop]  │  ┌─────────────────────────────────┐  │
│  │                 │  │        [Submit Guest]           │  │
│  └─────────────────┘  └─────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 4.2.4 API Endpoints

```
POST /api/v1/guests
  - Створення запису гостя

POST /api/v1/guests/{id}/photos
  - Завантаження фото гостя

POST /api/v1/guests/{id}/documents
  - Завантаження документа

GET /api/v1/guests/{id}/status
  - Отримання статусу обробки

GET /api/v1/guests/recent
  - Список останніх записів
```

---

### 4.3 OCR Service

#### 4.3.1 Опис

Сервіс для розпізнавання тексту з документів (паспорти, ID-картки) через cloud API.

#### 4.3.2 Підтримувані документи

| Тип документа | Підтримка | Примітки |
|---------------|-----------|----------|
| Закордонний паспорт (MRZ) | Full | Стандартизований формат |
| ID-картка (ЄС) | Partial | Залежить від країни |
| ID-картка (Україна) | Partial | Новий/старий формат |
| Водійське посвідчення | Experimental | Низький пріоритет |

#### 4.3.3 Функціональні вимоги

| ID | Вимога | Пріоритет |
|----|--------|-----------|
| OCR-01 | Інтеграція з Azure Document Intelligence | Must |
| OCR-02 | Fallback на Google Cloud Vision | Should |
| OCR-03 | Розпізнавання MRZ (Machine Readable Zone) | Must |
| OCR-04 | Витяг: ім'я, прізвище, номер документа | Must |
| OCR-05 | Витяг: країна, дата народження | Should |
| OCR-06 | Confidence score для кожного поля | Must |
| OCR-07 | Підтримка PDF (конвертація в image) | Must |
| OCR-08 | Preprocessing зображень (deskew, contrast) | Should |

#### 4.3.4 Data Model

```python
@dataclass
class OCRResult:
    document_type: DocumentType
    raw_text: str
    structured_data: StructuredDocumentData
    confidence: float
    processing_time_ms: int
    provider: str
    
@dataclass
class StructuredDocumentData:
    first_name: Optional[str]
    last_name: Optional[str]
    document_number: Optional[str]
    country_code: Optional[str]
    date_of_birth: Optional[date]
    expiry_date: Optional[date]
    mrz_line1: Optional[str]
    mrz_line2: Optional[str]
    field_confidences: dict[str, float]
```

#### 4.3.5 Processing Pipeline

```
Input Image
    │
    ▼
┌─────────────────┐
│ Preprocessing   │ ← Resize, deskew, contrast adjustment
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Document Type   │ ← Classify: passport, ID, unknown
│ Detection       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ OCR API Call    │ ← Azure Document Intelligence
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ MRZ Parsing     │ ← If MRZ detected
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Data            │ ← Normalize names, dates
│ Normalization   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Validation      │ ← Check digit verification (MRZ)
└────────┬────────┘
         │
         ▼
   OCRResult
```

---

### 4.4 Data Normalization Module

#### 4.4.1 Опис

Модуль для нормалізації та валідації даних з різних джерел.

#### 4.4.2 Функції

| Функція | Опис |
|---------|------|
| Name normalization | Uppercase, видалення спецсимволів, транслітерація |
| Date normalization | Уніфікація форматів дат |
| Country code mapping | ISO 3166-1 alpha-3 |
| MRZ checksum validation | Перевірка контрольних цифр |
| Duplicate detection | Пошук потенційних дублікатів |

#### 4.4.3 Правила нормалізації імен

```python
def normalize_name(raw_name: str) -> str:
    """
    1. Uppercase
    2. Remove diacritics (é → E, ü → U)
    3. Replace special chars with space
    4. Collapse multiple spaces
    5. Strip leading/trailing spaces
    """
```

---

### 4.5 Face Processing Module

#### 4.5.1 Опис

Модуль для виявлення облич та підготовки до розпізнавання.

#### 4.5.2 Функціональні вимоги

| ID | Вимога | Пріоритет |
|----|--------|-----------|
| FP-01 | Face detection на фото | Must |
| FP-02 | Face cropping з padding | Must |
| FP-03 | Face alignment (optional) | Should |
| FP-04 | Quality assessment | Should |
| FP-05 | Multiple faces handling | Must |
| FP-06 | Embedding generation (prep) | Must |
| FP-07 | Embedding storage | Must |

#### 4.5.3 Face Quality Metrics

| Метрика | Threshold | Дія при failure |
|---------|-----------|-----------------|
| Face size | ≥ 80x80 px | Warning |
| Blur score | < 100 (Laplacian variance) | Warning |
| Brightness | 40-220 (mean pixel value) | Warning |
| Face angle | ±30° (yaw/pitch) | Warning |

#### 4.5.4 Processing Pipeline

```
Input Photo
    │
    ▼
┌─────────────────┐
│ Face Detection  │ ← MTCNN / RetinaFace
│ (find all faces)│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Select Primary  │ ← Largest face / centered
│ Face            │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Face Alignment  │ ← Eye alignment
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Crop & Resize   │ ← 160x160 or 112x112
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Quality Check   │ ← Blur, brightness, angle
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Generate        │ ← FaceNet / ArcFace
│ Embedding       │
└────────┬────────┘
         │
         ▼
   512-dim vector
```

---

### 4.6 Database Module

#### 4.6.1 Schema Design

```sql
-- Core tables
CREATE TABLE guests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id VARCHAR(50),
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    room_number VARCHAR(20),
    check_in_date TIMESTAMP,
    check_out_date TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    source VARCHAR(20) NOT NULL, -- 'web', 'email'
    status VARCHAR(20) DEFAULT 'pending'
);

CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    guest_id UUID REFERENCES guests(id),
    document_type VARCHAR(30),
    file_path VARCHAR(500),
    file_hash VARCHAR(64),
    mime_type VARCHAR(50),
    file_size_bytes INTEGER,
    uploaded_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE ocr_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID REFERENCES documents(id),
    provider VARCHAR(30),
    raw_response JSONB,
    extracted_data JSONB,
    confidence FLOAT,
    processing_time_ms INTEGER,
    processed_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE face_photos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    guest_id UUID REFERENCES guests(id),
    original_path VARCHAR(500),
    cropped_path VARCHAR(500),
    quality_score FLOAT,
    quality_details JSONB,
    uploaded_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE face_embeddings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    face_photo_id UUID REFERENCES face_photos(id),
    model_name VARCHAR(50),
    model_version VARCHAR(20),
    embedding VECTOR(512), -- pgvector
    created_at TIMESTAMP DEFAULT NOW()
);

-- Processing tracking
CREATE TABLE processing_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type VARCHAR(30), -- 'document', 'photo'
    entity_id UUID,
    job_type VARCHAR(30), -- 'ocr', 'face_detection', 'embedding'
    status VARCHAR(20), -- 'pending', 'processing', 'completed', 'failed'
    attempts INTEGER DEFAULT 0,
    error_message TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    started_at TIMESTAMP,
    completed_at TIMESTAMP
);

-- Indexes
CREATE INDEX idx_guests_status ON guests(status);
CREATE INDEX idx_guests_created ON guests(created_at DESC);
CREATE INDEX idx_documents_guest ON documents(guest_id);
CREATE INDEX idx_processing_jobs_status ON processing_jobs(status);
CREATE INDEX idx_face_embeddings_vector ON face_embeddings 
    USING ivfflat (embedding vector_cosine_ops);
```

#### 4.6.2 File Storage Structure

```
/data/
├── raw/
│   ├── documents/
│   │   └── {year}/{month}/{day}/{guest_id}/{document_id}.{ext}
│   └── photos/
│       └── {year}/{month}/{day}/{guest_id}/{photo_id}.{ext}
├── processed/
│   ├── faces/
│   │   └── {guest_id}/{face_id}_cropped.jpg
│   └── documents/
│       └── {guest_id}/{document_id}_preprocessed.jpg
└── temp/
    └── {job_id}/
```

---

## 5. Діаграми

### 5.1 System Architecture Diagram

```mermaid
graph TB
    subgraph "Ingestion Layer"
        MS[Mail Server<br/>IMAP]
        WA[Web App<br/>React]
    end
    
    subgraph "API Layer"
        API[FastAPI<br/>Gateway]
    end
    
    subgraph "Processing Layer"
        Q[Redis<br/>Queue]
        W1[OCR Worker]
        W2[Face Worker]
        W3[File Worker]
    end
    
    subgraph "External Services"
        AZ[Azure Doc<br/>Intelligence]
        GC[Google Cloud<br/>Vision]
    end
    
    subgraph "Storage Layer"
        PG[(PostgreSQL)]
        FS[(File Storage)]
        VDB[(Vector DB<br/>Future)]
    end
    
    subgraph "Future: Edge Layer"
        NV[NVIDIA Dev Kit]
        CAM[Camera]
    end
    
    MS --> API
    WA --> API
    API --> Q
    Q --> W1
    Q --> W2
    Q --> W3
    W1 --> AZ
    W1 --> GC
    W1 --> PG
    W2 --> PG
    W2 --> VDB
    W3 --> FS
    
    CAM -.-> NV
    NV -.-> API
```

### 5.2 Data Flow Diagram

```mermaid
sequenceDiagram
    participant U as User/Email
    participant I as Ingestion
    participant Q as Queue
    participant O as OCR Worker
    participant F as Face Worker
    participant DB as Database
    participant S as Storage
    participant C as Cloud OCR
    
    U->>I: Submit photo + document
    I->>S: Store raw files
    I->>DB: Create guest record
    I->>Q: Enqueue OCR job
    I->>Q: Enqueue Face job
    
    Q->>O: Process OCR job
    O->>S: Get document image
    O->>O: Preprocess image
    O->>C: Call OCR API
    C-->>O: OCR result
    O->>DB: Save structured data
    O->>Q: Mark job complete
    
    Q->>F: Process Face job
    F->>S: Get photo
    F->>F: Detect faces
    F->>F: Crop & align
    F->>S: Save cropped face
    F->>F: Generate embedding
    F->>DB: Save embedding
    F->>Q: Mark job complete
```

### 5.3 AI Pipeline Diagram

```mermaid
graph LR
    subgraph "Input"
        IMG[Image]
    end
    
    subgraph "Preprocessing"
        RS[Resize]
        DN[Denoise]
        CT[Contrast]
    end
    
    subgraph "Detection"
        FD[Face<br/>Detection]
        DD[Document<br/>Detection]
    end
    
    subgraph "Processing"
        FA[Face<br/>Alignment]
        FC[Face<br/>Crop]
        FE[Embedding<br/>Generation]
        
        DP[Doc<br/>Preprocessing]
        OCR[OCR API]
        MRZ[MRZ<br/>Parsing]
    end
    
    subgraph "Output"
        EMB[Face<br/>Embedding]
        STR[Structured<br/>Data]
    end
    
    IMG --> RS --> DN --> CT
    CT --> FD --> FA --> FC --> FE --> EMB
    CT --> DD --> DP --> OCR --> MRZ --> STR
```

### 5.4 Future NVIDIA Dev Kit Integration Diagram

```mermaid
graph TB
    subgraph "Edge Device: NVIDIA Jetson Orin"
        CAM[USB/CSI Camera]
        DS[DeepStream SDK]
        INF[TensorRT Inference]
        
        subgraph "Models"
            FD[Face Detector]
            FR[Face Recognizer]
            TR[Tracker]
        end
        
        BUF[Frame Buffer]
        API_E[Edge API]
    end
    
    subgraph "Backend Server"
        API_S[Main API]
        DB[(Database)]
        VDB[(Vector DB)]
    end
    
    subgraph "Actions"
        ALR[Alert Service]
        LOG[Event Logger]
    end
    
    CAM --> DS
    DS --> BUF
    BUF --> INF
    INF --> FD
    FD --> FR
    FD --> TR
    FR --> API_E
    TR --> API_E
    
    API_E --> API_S
    API_S --> DB
    API_S --> VDB
    API_S --> ALR
    API_S --> LOG
```

### 5.5 Component Interaction Diagram

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                                                                                │
│   ┌──────────┐         ┌──────────┐         ┌──────────────────────────────┐  │
│   │  React   │  HTTP   │  FastAPI │  Queue  │        Celery Workers        │  │
│   │  Web App │◄───────►│  Server  │◄───────►│  ┌───────┐ ┌───────┐ ┌────┐  │  │
│   └──────────┘         └──────────┘         │  │  OCR  │ │ Face  │ │File│  │  │
│        │                    │               │  └───┬───┘ └───┬───┘ └──┬─┘  │  │
│        │                    │               └──────┼─────────┼────────┼────┘  │
│        │                    │                      │         │        │       │
│        │                    ▼                      │         │        │       │
│        │              ┌──────────┐                 │         │        │       │
│        │              │  Redis   │                 │         │        │       │
│        │              │  Cache   │                 │         │        │       │
│        │              └──────────┘                 │         │        │       │
│        │                    │                      │         │        │       │
│        │                    │                      ▼         ▼        ▼       │
│        │               ┌────┴────┐           ┌─────────────────────────────┐  │
│        │               │PostgreSQL│◄─────────│         File Storage        │  │
│        │               │   + pgvector│        │     (MinIO / Local FS)     │  │
│        │               └──────────┘           └─────────────────────────────┘  │
│        │                    ▲                                                  │
│        │                    │                                                  │
│   ┌────┴────────────────────┴───────────────────────────────────────────────┐  │
│   │                         Mail Ingestion Service                          │  │
│   │                      (IMAP Polling Daemon)                              │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                     ▲                                          │
│                                     │                                          │
│                              ┌──────┴──────┐                                   │
│                              │ Internal    │                                   │
│                              │ Mail Server │                                   │
│                              └─────────────┘                                   │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Етапи реалізації (Roadmap)

### 6.1 Огляд фаз

```
Phase 1          Phase 2          Phase 3          Phase 4
Infrastructure   Core Processing  AI Enhancement   Edge Integration
─────────────────────────────────────────────────────────────────►
   4 weeks          6 weeks          4 weeks          6 weeks
                                                        │
                                              Requires NVIDIA Dev Kit
```

### 6.2 Phase 1: Infrastructure Foundation (4 тижні)

**Мета:** Базова інфраструктура та ingestion

| Тиждень | Завдання | Deliverables |
|---------|----------|--------------|
| 1 | Налаштування середовища | Docker compose, PostgreSQL, Redis |
| 1 | Структура FastAPI проєкту | API skeleton, health checks |
| 2 | Database schema | Migrations, models, repositories |
| 2 | File storage setup | MinIO/local, upload endpoints |
| 3 | Mail ingestion service | IMAP client, attachment extraction |
| 3 | Basic validation | MIME types, file size checks |
| 4 | React app skeleton | Camera API, upload form |
| 4 | Integration testing | E2E ingestion flow |

**Залежності:** Немає (можна починати одразу)

**Hardware:** Стандартний сервер/workstation

### 6.3 Phase 2: Core Processing (6 тижнів)

**Мета:** OCR та базова обробка облич

| Тиждень | Завдання | Deliverables |
|---------|----------|--------------|
| 5 | Celery workers setup | Task queue, monitoring |
| 5 | Azure OCR integration | API client, auth |
| 6 | OCR preprocessing | Image enhancement pipeline |
| 6 | MRZ parsing | Check digit validation |
| 7 | Data normalization | Name/date/country handling |
| 7 | Face detection service | MTCNN integration |
| 8 | Face cropping & quality | Alignment, quality metrics |
| 9 | Embedding generation | FaceNet/ArcFace integration |
| 9 | pgvector setup | Vector storage, basic search |
| 10 | Web app completion | Status display, history |

**Залежності:** Phase 1 complete

**Hardware:** 
- Сервер з GPU recommended (для face processing)
- Можливо працювати на CPU (повільніше)

### 6.4 Phase 3: AI Enhancement (4 тижні)

**Мета:** Покращення якості та дослідження

| Тиждень | Завдання | Deliverables |
|---------|----------|--------------|
| 11 | OCR fallback (Google) | Multi-provider support |
| 11 | Confidence thresholds | Auto-review flagging |
| 12 | Face matching research | Similarity thresholds |
| 12 | Duplicate detection | Guest deduplication |
| 13 | Quality improvements | Edge cases handling |
| 13 | Performance optimization | Batch processing |
| 14 | Documentation | API docs, runbooks |
| 14 | Testing & hardening | Load testing, error handling |

**Залежності:** Phase 2 complete

**Hardware:** Existing infrastructure

### 6.5 Phase 4: Edge Integration (6 тижнів)

**Мета:** NVIDIA Dev Kit інтеграція

| Тиждень | Завдання | Deliverables |
|---------|----------|--------------|
| 15 | Dev Kit setup | JetPack, environment |
| 15 | Camera integration | USB/CSI camera stream |
| 16 | Model conversion | TensorRT optimization |
| 16 | DeepStream pipeline | Video analytics |
| 17 | Face detection edge | Real-time detection |
| 17 | Face recognition edge | Embedding on edge |
| 18 | Backend integration | Edge-to-cloud sync |
| 18 | Tracking research | Multi-face tracking |
| 19 | V-JEPA exploration | Video understanding |
| 19 | Performance tuning | FPS optimization |
| 20 | Demo preparation | End-to-end demo |
| 20 | Documentation | Edge deployment guide |

**Залежності:** 
- Phase 3 complete
- NVIDIA Jetson Orin Dev Kit
- USB/CSI camera

**Hardware:**
- NVIDIA Jetson Orin Developer Kit
- Compatible camera (USB 3.0 or CSI)
- Adequate power supply
- Network connectivity

### 6.6 Dependency Graph

```mermaid
gantt
    title Project Roadmap
    dateFormat  YYYY-MM-DD
    section Phase 1
    Docker/Infra          :p1a, 2026-01-20, 1w
    Database Setup        :p1b, after p1a, 1w
    Mail Ingestion        :p1c, after p1b, 1w
    Web App Skeleton      :p1d, after p1c, 1w
    
    section Phase 2
    Celery Workers        :p2a, after p1d, 1w
    OCR Integration       :p2b, after p2a, 2w
    Face Detection        :p2c, after p2b, 2w
    Embeddings            :p2d, after p2c, 1w
    
    section Phase 3
    OCR Fallback          :p3a, after p2d, 1w
    Face Matching         :p3b, after p3a, 2w
    Optimization          :p3c, after p3b, 1w
    
    section Phase 4
    Dev Kit Setup         :p4a, after p3c, 1w
    Edge Detection        :p4b, after p4a, 2w
    Edge Recognition      :p4c, after p4b, 2w
    Demo                  :p4d, after p4c, 1w
```

### 6.7 Що можна робити без NVIDIA Dev Kit

| Компонент | Без Dev Kit | З Dev Kit |
|-----------|-------------|-----------|
| Mail ingestion | ✅ Повна функціональність | ✅ |
| Web ingestion | ✅ Повна функціональність | ✅ |
| OCR (cloud) | ✅ Повна функціональність | ✅ |
| Face detection | ✅ CPU/cloud (повільніше) | ✅ Real-time |
| Face embeddings | ✅ CPU (batch) | ✅ Real-time |
| Face recognition | ✅ Batch matching | ✅ Real-time |
| Video stream | ❌ | ✅ |
| Tracking | ❌ | ✅ |
| V-JEPA research | ❌ | ✅ |

---

## 7. Ризики та обмеження

### 7.1 Technical Risks

| Ризик | Ймовірність | Вплив | Mitigation |
|-------|-------------|-------|------------|
| **OCR якість для non-MRZ** | Висока | Середній | Multi-provider fallback, manual review queue |
| **Різноманітність документів** | Висока | Середній | Focus на MRZ спочатку, ітеративне розширення |
| **Cloud OCR latency** | Середня | Низький | Async processing, batch optimization |
| **Face detection в поганих умовах** | Середня | Середній | Quality scoring, user feedback |
| **pgvector scale** | Низька | Середній | Партиціонування, Milvus як альтернатива |

### 7.2 Operational Risks

| Ризик | Ймовірність | Вплив | Mitigation |
|-------|-------------|-------|------------|
| **Internal mail server changes** | Низька | Високий | Abstraction layer, config-driven |
| **Cloud API quota/cost** | Середня | Середній | Rate limiting, caching |
| **Network issues** | Середня | Середній | Retry logic, circuit breaker |
| **Storage capacity** | Низька | Середній | Monitoring, cleanup policies |

### 7.3 Hardware Risks

| Ризик | Ймовірність | Вплив | Mitigation |
|-------|-------------|-------|------------|
| **Dev Kit delays** | Середня | Високий | Phase 1-3 не залежать від Dev Kit |
| **Camera compatibility** | Низька | Середній | Тестування перед закупкою |
| **Power requirements** | Низька | Низький | Proper power supply planning |

### 7.4 OCR Quality Considerations

#### Фактори впливу на якість

| Фактор | Вплив | Рекомендація |
|--------|-------|--------------|
| **Освітлення** | Високий | Guidelines для користувачів, auto-enhancement |
| **Кут нахилу** | Середній | Deskew preprocessing |
| **Роздільна здатність** | Високий | Мінімум 300 DPI або 2MP |
| **Фокус** | Високий | Blur detection, re-capture prompt |
| **Блики** | Середній | Anti-glare guidelines |
| **Зношеність документа** | Середній | Manual review для low confidence |

#### Очікувана якість OCR

| Тип документа | MRZ Accuracy | Full-text Accuracy |
|---------------|--------------|-------------------|
| Новий паспорт (MRZ) | 95-99% | 85-95% |
| Старий паспорт | 80-90% | 70-85% |
| ID-картка (ЄС) | 85-95% | 75-90% |
| ID-картка (Україна, нова) | 80-90% | 70-85% |
| ID-картка (Україна, стара) | 60-75% | 50-70% |

### 7.5 Lighting Considerations for Face Recognition

| Умова | Вплив | Рекомендація |
|-------|-------|--------------|
| **Недостатнє освітлення** | Face detection failures | Мінімум 300 lux |
| **Контрове світло** | Poor face quality | Avoid backlight |
| **Тіні на обличчі** | Embedding degradation | Diffused lighting |
| **Різке штучне світло** | Color cast | White balance |

### 7.6 Обмеження R&D проєкту

| Обмеження | Опис | Impact |
|-----------|------|--------|
| **Не production-ready** | Немає HA, DR, повного error handling | Не для реального використання |
| **Немає GDPR compliance** | Data retention, consent не реалізовано | Тільки для тестових даних |
| **Single location** | Не масштабується на мережу готелів | Потребує архітектурних змін |
| **Limited audit trail** | Базове логування | Не для compliance |
| **No user management** | Single-user mode | Потребує auth system |

---

## 8. Deliverables

### 8.1 Phase 1 Deliverables

| Deliverable | Формат | Опис |
|-------------|--------|------|
| Infrastructure code | Git repo | Docker compose, configs |
| Database migrations | SQL/Alembic | Schema + initial data |
| Mail ingestion service | Python | Daemon + tests |
| Web app MVP | React | Basic upload/capture |
| API documentation | OpenAPI | Swagger UI |

### 8.2 Phase 2 Deliverables

| Deliverable | Формат | Опис |
|-------------|--------|------|
| OCR service | Python | Azure integration + preprocessing |
| Face processing service | Python | Detection + embedding |
| Processing workers | Python/Celery | Async job processing |
| Web app complete | React | Full functionality |
| Integration tests | pytest | E2E test suite |

### 8.3 Phase 3 Deliverables

| Deliverable | Формат | Опис |
|-------------|--------|------|
| Multi-provider OCR | Python | Google fallback |
| Face matching POC | Python | Similarity search demo |
| Performance report | Markdown | Benchmarks, bottlenecks |
| Optimization implementation | Code | Batch processing, caching |
| Complete documentation | Markdown | All modules documented |

### 8.4 Phase 4 Deliverables

| Deliverable | Формат | Опис |
|-------------|--------|------|
| Edge deployment package | Docker/JetPack | Dev Kit setup |
| TensorRT models | Model files | Optimized inference |
| DeepStream pipeline | Python/C++ | Video analytics |
| Edge-cloud integration | Code | Sync protocol |
| Demo application | Working system | End-to-end demo |
| Research report | Markdown | V-JEPA findings |

### 8.5 Final Deliverables

| Deliverable | Опис |
|-------------|------|
| **Source code repository** | All code, properly structured, with README |
| **Documentation package** | Architecture docs, API docs, deployment guides |
| **Demo environment** | Working demo with sample data |
| **Technical report** | Findings, benchmarks, recommendations |
| **Recommendations document** | Next steps for production development |

### 8.6 Рекомендації для наступного етапу

Документ повинен містити:

1. **Production readiness gaps**
   - High availability requirements
   - Disaster recovery needs
   - Security hardening

2. **Scale considerations**
   - Multi-location architecture
   - Data synchronization
   - Central management

3. **Compliance requirements**
   - GDPR implementation
   - Data retention policies
   - Consent management
   - Audit trails

4. **ML/AI improvements**
   - Model fine-tuning recommendations
   - Edge model optimization
   - Continuous learning pipeline

5. **Hardware recommendations**
   - Production-grade Dev Kit
   - Camera specifications
   - Network requirements

---

## Appendix A: API Reference (Draft)

### A.1 Guest Endpoints

```yaml
POST /api/v1/guests:
  summary: Create new guest
  request:
    body:
      first_name: string (optional)
      last_name: string (optional)
      room_number: string (optional)
  response:
    201:
      id: uuid
      status: "pending"

GET /api/v1/guests/{id}:
  summary: Get guest details
  response:
    200:
      id: uuid
      first_name: string
      last_name: string
      documents: array
      photos: array
      status: string

GET /api/v1/guests/{id}/status:
  summary: Get processing status
  response:
    200:
      ocr_status: string
      face_status: string
      overall_status: string
```

### A.2 Document Endpoints

```yaml
POST /api/v1/guests/{id}/documents:
  summary: Upload document
  request:
    content-type: multipart/form-data
    body:
      file: binary
      document_type: string (optional)
  response:
    201:
      document_id: uuid
      status: "queued"

GET /api/v1/documents/{id}/ocr:
  summary: Get OCR results
  response:
    200:
      status: string
      extracted_data: object
      confidence: float
```

### A.3 Photo Endpoints

```yaml
POST /api/v1/guests/{id}/photos:
  summary: Upload photo
  request:
    content-type: multipart/form-data
    body:
      file: binary
  response:
    201:
      photo_id: uuid
      status: "queued"

GET /api/v1/photos/{id}/face:
  summary: Get face processing results
  response:
    200:
      status: string
      faces_detected: int
      quality_score: float
      cropped_url: string
```

---

## Appendix B: Configuration Reference

### B.1 Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/guestdb
REDIS_URL=redis://localhost:6379/0

# Storage
STORAGE_TYPE=local  # or 'minio'
STORAGE_PATH=/data
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# OCR
AZURE_DOCUMENT_INTELLIGENCE_ENDPOINT=https://xxx.cognitiveservices.azure.com/
AZURE_DOCUMENT_INTELLIGENCE_KEY=xxx
GOOGLE_CLOUD_VISION_CREDENTIALS=/path/to/credentials.json

# Mail
MAIL_IMAP_HOST=internal-mail.local
MAIL_IMAP_PORT=993
MAIL_IMAP_USER=ingestion@internal
MAIL_IMAP_PASSWORD=xxx
MAIL_POLLING_INTERVAL=30

# Processing
MAX_CONCURRENT_OCR_JOBS=5
MAX_CONCURRENT_FACE_JOBS=3
OCR_TIMEOUT_SECONDS=60
FACE_DETECTION_BACKEND=mtcnn  # or 'retinaface'
```

### B.2 Docker Compose Services

```yaml
services:
  api:
    image: guest-recognition-api
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
      
  worker-ocr:
    image: guest-recognition-worker
    command: celery -A app.worker worker -Q ocr
    
  worker-face:
    image: guest-recognition-worker
    command: celery -A app.worker worker -Q face
    
  mail-ingestion:
    image: guest-recognition-mail
    
  web:
    image: guest-recognition-web
    ports:
      - "3000:80"
      
  db:
    image: pgvector/pgvector:pg15
    
  redis:
    image: redis:7-alpine
    
  minio:
    image: minio/minio
```

---

## Appendix C: Glossary

| Term | Definition |
|------|------------|
| **ArcFace** | Face recognition model for embedding generation |
| **DeepStream** | NVIDIA SDK for video analytics |
| **Embedding** | Vector representation of data (face, document) |
| **FaceNet** | Google's face recognition model |
| **JetPack** | NVIDIA SDK for Jetson devices |
| **MRZ** | Machine Readable Zone on travel documents |
| **MTCNN** | Multi-task Cascaded Convolutional Networks for face detection |
| **OCR** | Optical Character Recognition |
| **pgvector** | PostgreSQL extension for vector similarity search |
| **RetinaFace** | Face detection model |
| **TensorRT** | NVIDIA's inference optimization library |
| **V-JEPA** | Meta's video understanding model |

---

*Кінець документа*