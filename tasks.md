## 📋 Повний список завдань: Guest Recognition R&D Platform

### **PHASE 1: Infrastructure Foundation (Тижні 1-4)**

---

#### **Тиждень 1: Базова інфраструктура**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 1.1 | Створити Git репозиторій з `.gitignore`, `README.md` | Setup | — |
| 1.2 | Створити базову структуру директорій проєкту | Setup | 1.1 |
| 1.3 | Написати `docker-compose.yml` з сервісами: PostgreSQL, Redis, MinIO | DevOps | 1.2 |
| 1.4 | Налаштувати PostgreSQL з розширенням pgvector | DevOps | 1.3 |
| 1.5 | Створити skeleton FastAPI проєкту | Backend | 1.2 |
| 1.6 | Реалізувати health check endpoints (`/health`, `/ready`) | Backend | 1.5 |
| 1.7 | Налаштувати логування (structlog / loguru) | Backend | 1.5 |
| 1.8 | Створити базовий `Dockerfile` для API | DevOps | 1.5 |

---

#### **Тиждень 2: Database & Storage**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 2.1 | Налаштувати Alembic для міграцій | Backend | 1.4 |
| 2.2 | Створити міграцію: таблиця `guests` | Backend | 2.1 |
| 2.3 | Створити міграцію: таблиця `documents` | Backend | 2.2 |
| 2.4 | Створити міграцію: таблиця `ocr_results` | Backend | 2.3 |
| 2.5 | Створити міграцію: таблиця `face_photos` | Backend | 2.4 |
| 2.6 | Створити міграцію: таблиця `face_embeddings` (з VECTOR) | Backend | 2.5 |
| 2.7 | Створити міграцію: таблиця `processing_jobs` | Backend | 2.6 |
| 2.8 | Реалізувати SQLAlchemy моделі для всіх таблиць | Backend | 2.7 |
| 2.9 | Реалізувати Repository pattern для Guest | Backend | 2.8 |
| 2.10 | Налаштувати MinIO bucket для файлів | DevOps | 1.3 |
| 2.11 | Реалізувати FileStorage service (абстракція над MinIO/local) | Backend | 2.10 |
| 2.12 | Створити структуру директорій для файлів (`/raw/`, `/processed/`) | Backend | 2.11 |

---

#### **Тиждень 3: Mail Ingestion**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 3.1 | Дослідити IMAP бібліотеки Python (aioimaplib vs imapclient) | Research | — |
| 3.2 | Реалізувати IMAP клієнт з підтримкою SSL | Backend | 3.1 |
| 3.3 | Реалізувати polling механізм з налаштовуваним інтервалом | Backend | 3.2 |
| 3.4 | Реалізувати витяг вкладень з email | Backend | 3.3 |
| 3.5 | Реалізувати валідацію MIME-типів (JPEG, PNG, PDF) | Backend | 3.4 |
| 3.6 | Реалізувати валідацію розміру файлу (max 20MB) | Backend | 3.5 |
| 3.7 | Реалізувати маркування оброблених листів (move to folder) | Backend | 3.6 |
| 3.8 | Додати retry logic з exponential backoff | Backend | 3.7 |
| 3.9 | Реалізувати конфігурацію через environment variables | Backend | 3.8 |
| 3.10 | Написати unit тести для Mail Ingestion | Testing | 3.9 |
| 3.11 | Створити Dockerfile для mail ingestion daemon | DevOps | 3.9 |

---

#### **Тиждень 4: Web App Skeleton**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 4.1 | Ініціалізувати React проєкт з Vite | Frontend | — |
| 4.2 | Налаштувати Tailwind CSS + shadcn/ui | Frontend | 4.1 |
| 4.3 | Створити базовий layout (header, main area) | Frontend | 4.2 |
| 4.4 | Реалізувати Camera component (MediaDevices API) | Frontend | 4.3 |
| 4.5 | Реалізувати capture фото з камери | Frontend | 4.4 |
| 4.6 | Реалізувати File Upload component (drag & drop) | Frontend | 4.3 |
| 4.7 | Реалізувати preview зображень | Frontend | 4.5, 4.6 |
| 4.8 | Реалізувати форму Guest Information | Frontend | 4.3 |
| 4.9 | Інтегрувати React Query для API calls | Frontend | 4.8 |
| 4.10 | Реалізувати POST /api/v1/guests endpoint | Backend | 2.9 |
| 4.11 | Реалізувати POST /api/v1/guests/{id}/photos endpoint | Backend | 4.10, 2.11 |
| 4.12 | Реалізувати POST /api/v1/guests/{id}/documents endpoint | Backend | 4.10, 2.11 |
| 4.13 | Інтегрувати frontend з backend API | Frontend | 4.9, 4.12 |
| 4.14 | Написати E2E тест: upload photo + document flow | Testing | 4.13 |
| 4.15 | Створити Dockerfile для web app (nginx) | DevOps | 4.13 |

---

### **PHASE 2: Core Processing (Тижні 5-10)**

---

#### **Тиждень 5: Task Queue Setup**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 5.1 | Налаштувати Celery з Redis broker | Backend | Phase 1 |
| 5.2 | Створити базову структуру workers | Backend | 5.1 |
| 5.3 | Реалізувати OCR queue | Backend | 5.2 |
| 5.4 | Реалізувати Face processing queue | Backend | 5.2 |
| 5.5 | Реалізувати File processing queue | Backend | 5.2 |
| 5.6 | Налаштувати Flower для моніторингу Celery | DevOps | 5.1 |
| 5.7 | Реалізувати job status tracking (processing_jobs table) | Backend | 5.2, 2.7 |
| 5.8 | Додати retry logic для failed jobs | Backend | 5.7 |

---

#### **Тиждень 5-6: Azure OCR Integration**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 6.1 | Створити Azure Document Intelligence ресурс | Setup | — |
| 6.2 | Реалізувати Azure OCR клієнт | Backend | 6.1 |
| 6.3 | Реалізувати виклик prebuilt-idDocument model | Backend | 6.2 |
| 6.4 | Реалізувати парсинг response в StructuredDocumentData | Backend | 6.3 |
| 6.5 | Реалізувати збереження OCR результатів в БД | Backend | 6.4, 2.4 |
| 6.6 | Написати unit тести для OCR клієнта (з mocks) | Testing | 6.5 |

---

#### **Тиждень 6: OCR Preprocessing**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 7.1 | Реалізувати image resize (max dimension) | Backend | — |
| 7.2 | Реалізувати deskew (вирівнювання нахилу) | Backend | — |
| 7.3 | Реалізувати contrast enhancement (CLAHE) | Backend | — |
| 7.4 | Реалізувати PDF to image конвертацію | Backend | — |
| 7.5 | Об'єднати preprocessing в pipeline | Backend | 7.1-7.4 |
| 7.6 | Інтегрувати preprocessing в OCR worker | Backend | 7.5, 5.3 |

---

#### **Тиждень 6: MRZ Parsing**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 8.1 | Дослідити MRZ формат (TD1, TD2, TD3) | Research | — |
| 8.2 | Реалізувати MRZ line detection | Backend | 8.1 |
| 8.3 | Реалізувати MRZ parsing (поля) | Backend | 8.2 |
| 8.4 | Реалізувати check digit validation | Backend | 8.3 |
| 8.5 | Інтегрувати MRZ parsing в OCR pipeline | Backend | 8.4, 6.4 |
| 8.6 | Написати тести з sample MRZ даними | Testing | 8.5 |

---

#### **Тиждень 7: Data Normalization**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 9.1 | Реалізувати name normalization (uppercase, diacritics) | Backend | — |
| 9.2 | Реалізувати date normalization (різні формати → ISO) | Backend | — |
| 9.3 | Реалізувати country code mapping (ISO 3166-1 alpha-3) | Backend | — |
| 9.4 | Реалізувати transliteration (кирилиця → латиниця) | Backend | — |
| 9.5 | Об'єднати в DataNormalizationService | Backend | 9.1-9.4 |
| 9.6 | Інтегрувати нормалізацію в OCR worker | Backend | 9.5, 6.5 |

---

#### **Тиждень 7-8: Face Detection**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 10.1 | Дослідити MTCNN vs RetinaFace (accuracy, speed) | Research | — |
| 10.2 | Встановити обрану бібліотеку | Backend | 10.1 |
| 10.3 | Реалізувати FaceDetector class | Backend | 10.2 |
| 10.4 | Реалізувати detect_faces(image) → list[BoundingBox] | Backend | 10.3 |
| 10.5 | Реалізувати вибір primary face (largest/centered) | Backend | 10.4 |
| 10.6 | Реалізувати face cropping з padding | Backend | 10.5 |
| 10.7 | Реалізувати збереження cropped face в storage | Backend | 10.6, 2.11 |
| 10.8 | Інтегрувати в Face worker | Backend | 10.7, 5.4 |

---

#### **Тиждень 8: Face Quality Assessment**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 11.1 | Реалізувати blur detection (Laplacian variance) | Backend | — |
| 11.2 | Реалізувати brightness check (mean pixel value) | Backend | — |
| 11.3 | Реалізувати face angle estimation | Backend | — |
| 11.4 | Реалізувати face size check | Backend | — |
| 11.5 | Об'єднати в QualityAssessment service | Backend | 11.1-11.4 |
| 11.6 | Інтегрувати quality check в Face worker | Backend | 11.5, 10.8 |
| 11.7 | Зберігати quality_score та quality_details в БД | Backend | 11.6, 2.5 |

---

#### **Тиждень 9: Face Embeddings**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 12.1 | Дослідити FaceNet vs ArcFace (embedding quality) | Research | — |
| 12.2 | Завантажити pretrained model | Backend | 12.1 |
| 12.3 | Реалізувати face alignment (eye alignment) | Backend | 10.4 |
| 12.4 | Реалізувати EmbeddingGenerator class | Backend | 12.2 |
| 12.5 | Реалізувати generate_embedding(face_image) → vector | Backend | 12.4 |
| 12.6 | Інтегрувати embedding generation в Face worker | Backend | 12.5, 10.8 |
| 12.7 | Зберігати embeddings в face_embeddings (pgvector) | Backend | 12.6, 2.6 |

---

#### **Тиждень 9: Vector Search Setup**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 13.1 | Налаштувати pgvector індекс (IVFFlat) | Backend | 12.7 |
| 13.2 | Реалізувати similarity search (cosine distance) | Backend | 13.1 |
| 13.3 | Реалізувати find_similar_faces(embedding, threshold) | Backend | 13.2 |
| 13.4 | Написати тести для vector search | Testing | 13.3 |

---

#### **Тиждень 10: Web App Completion**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 14.1 | Реалізувати GET /api/v1/guests/{id}/status endpoint | Backend | 5.7 |
| 14.2 | Реалізувати GET /api/v1/guests/recent endpoint | Backend | 2.9 |
| 14.3 | Реалізувати status polling на frontend | Frontend | 14.1 |
| 14.4 | Додати індикатор статусу обробки (spinner, progress) | Frontend | 14.3 |
| 14.5 | Реалізувати History component (останні записи) | Frontend | 14.2 |
| 14.6 | Додати error handling та user feedback | Frontend | 14.4 |
| 14.7 | Стилізувати UI (responsive design) | Frontend | 14.6 |
| 14.8 | Написати integration tests для повного flow | Testing | 14.7 |

---

### **PHASE 3: AI Enhancement (Тижні 11-14)**

---

#### **Тиждень 11: Multi-Provider OCR**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 15.1 | Створити Google Cloud Vision ресурс | Setup | — |
| 15.2 | Реалізувати Google Vision OCR клієнт | Backend | 15.1 |
| 15.3 | Реалізувати OCRProvider abstract class | Backend | — |
| 15.4 | Адаптувати Azure та Google clients під OCRProvider | Backend | 15.3, 6.2, 15.2 |
| 15.5 | Реалізувати fallback logic (Azure → Google) | Backend | 15.4 |
| 15.6 | Реалізувати confidence-based provider selection | Backend | 15.5 |

---

#### **Тиждень 11: Confidence & Review**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 16.1 | Визначити confidence thresholds для auto-accept | Config | — |
| 16.2 | Реалізувати flagging для manual review (low confidence) | Backend | 16.1 |
| 16.3 | Додати status 'needs_review' для guests | Backend | 16.2 |
| 16.4 | Реалізувати GET /api/v1/guests/review-queue endpoint | Backend | 16.3 |
| 16.5 | Додати Review Queue UI component | Frontend | 16.4 |

---

#### **Тиждень 12: Face Matching**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 17.1 | Дослідити optimal similarity threshold | Research | 13.3 |
| 17.2 | Реалізувати match_face(guest_id) → list[MatchResult] | Backend | 17.1, 13.3 |
| 17.3 | Реалізувати ranking по similarity score | Backend | 17.2 |
| 17.4 | Додати match results в guest record | Backend | 17.3 |
| 17.5 | Реалізувати UI для показу potential matches | Frontend | 17.4 |

---

#### **Тиждень 12: Duplicate Detection**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 18.1 | Реалізувати document-based duplicate check | Backend | 6.5 |
| 18.2 | Реалізувати face-based duplicate check | Backend | 17.2 |
| 18.3 | Об'єднати в DuplicateDetector service | Backend | 18.1, 18.2 |
| 18.4 | Додати duplicate warning в guest creation flow | Backend | 18.3 |
| 18.5 | Додати duplicate notification в UI | Frontend | 18.4 |

---

#### **Тиждень 13: Quality Improvements**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 19.1 | Покращити обробку low-quality зображень | Backend | 7.5 |
| 19.2 | Додати auto-rotation based on EXIF | Backend | 7.5 |
| 19.3 | Покращити handling різних document orientations | Backend | 6.3 |
| 19.4 | Додати support для multi-page PDF | Backend | 7.4 |
| 19.5 | Покращити error messages для користувачів | Frontend | — |

---

#### **Тиждень 13: Performance Optimization**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 20.1 | Реалізувати batch OCR processing | Backend | 6.5 |
| 20.2 | Реалізувати batch embedding generation | Backend | 12.6 |
| 20.3 | Додати caching для OCR results | Backend | 6.5 |
| 20.4 | Оптимізувати database queries (explain analyze) | Backend | — |
| 20.5 | Налаштувати connection pooling | Backend | — |

---

#### **Тиждень 14: Documentation & Testing**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 21.1 | Написати API documentation (OpenAPI/Swagger) | Docs | — |
| 21.2 | Написати deployment guide (docker-compose) | Docs | — |
| 21.3 | Написати runbook для operations | Docs | — |
| 21.4 | Написати architecture decision records (ADRs) | Docs | — |
| 21.5 | Провести load testing (locust/k6) | Testing | — |
| 21.6 | Провести security review | Testing | — |
| 21.7 | Виправити знайдені issues | Backend | 21.5, 21.6 |

---

### **PHASE 4: Edge Integration (Тижні 15-20)**

> ⚠️ **Вимагає:** NVIDIA Jetson Orin Dev Kit + Camera

---

#### **Тиждень 15: Dev Kit Setup**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 22.1 | Встановити JetPack 6.x на Dev Kit | DevOps | Hardware |
| 22.2 | Налаштувати CUDA та cuDNN | DevOps | 22.1 |
| 22.3 | Встановити TensorRT | DevOps | 22.2 |
| 22.4 | Встановити DeepStream SDK | DevOps | 22.3 |
| 22.5 | Налаштувати network connectivity | DevOps | 22.1 |
| 22.6 | Підключити та протестувати камеру | Hardware | 22.1 |
| 22.7 | Написати test script для camera stream | Backend | 22.6 |

---

#### **Тиждень 16: Model Conversion**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 23.1 | Конвертувати face detection model в ONNX | ML | 10.3 |
| 23.2 | Конвертувати ONNX в TensorRT engine | ML | 23.1, 22.3 |
| 23.3 | Конвертувати embedding model в ONNX | ML | 12.4 |
| 23.4 | Конвертувати embedding ONNX в TensorRT | ML | 23.3, 22.3 |
| 23.5 | Benchmark TensorRT models (FPS, latency) | Testing | 23.2, 23.4 |
| 23.6 | Оптимізувати models (FP16, INT8 quantization) | ML | 23.5 |

---

#### **Тиждень 16-17: DeepStream Pipeline**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 24.1 | Створити базовий DeepStream pipeline config | ML | 22.4 |
| 24.2 | Додати camera source element | ML | 24.1, 22.6 |
| 24.3 | Додати face detection inference | ML | 24.2, 23.2 |
| 24.4 | Додати tracker element (NvDCF/IOU) | ML | 24.3 |
| 24.5 | Реалізувати metadata extraction | ML | 24.4 |
| 24.6 | Додати visualization (bounding boxes) | ML | 24.5 |
| 24.7 | Benchmark pipeline (FPS, GPU usage) | Testing | 24.6 |

---

#### **Тиждень 17: Edge Face Recognition**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 25.1 | Інтегрувати embedding inference в pipeline | ML | 24.5, 23.4 |
| 25.2 | Реалізувати face cropping на edge | ML | 25.1 |
| 25.3 | Реалізувати embedding generation на edge | ML | 25.2 |
| 25.4 | Реалізувати local embedding cache | Backend | 25.3 |
| 25.5 | Реалізувати local similarity search | Backend | 25.4 |
| 25.6 | Benchmark recognition pipeline | Testing | 25.5 |

---

#### **Тиждень 18: Edge-Backend Integration**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 26.1 | Реалізувати Edge API (FastAPI на Jetson) | Backend | 25.5 |
| 26.2 | Реалізувати sync protocol (edge → backend) | Backend | 26.1 |
| 26.3 | Реалізувати event streaming (detected faces) | Backend | 26.2 |
| 26.4 | Реалізувати periodic embedding sync | Backend | 26.3 |
| 26.5 | Реалізувати offline mode handling | Backend | 26.4 |
| 26.6 | Додати edge status в backend dashboard | Frontend | 26.3 |

---

#### **Тиждень 18: Multi-Face Tracking**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 27.1 | Дослідити tracking algorithms (DeepSORT, ByteTrack) | Research | — |
| 27.2 | Налаштувати tracker parameters | ML | 27.1, 24.4 |
| 27.3 | Реалізувати track lifecycle management | ML | 27.2 |
| 27.4 | Реалізувати re-identification при втраті track | ML | 27.3 |
| 27.5 | Benchmark tracking accuracy | Testing | 27.4 |

---

#### **Тиждень 19: V-JEPA Research**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 28.1 | Дослідити V-JEPA архітектуру | Research | — |
| 28.2 | Оцінити hardware вимоги для inference | Research | 28.1 |
| 28.3 | Протестувати V-JEPA на Dev Kit (якщо можливо) | Research | 28.2 |
| 28.4 | Оцінити applicability для guest behavior analysis | Research | 28.3 |
| 28.5 | Документувати findings та recommendations | Docs | 28.4 |

---

#### **Тиждень 19: Performance Tuning**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 29.1 | Profile GPU usage | Testing | 25.6 |
| 29.2 | Оптимізувати batch size для inference | ML | 29.1 |
| 29.3 | Налаштувати power modes (MAX-N, 15W, 30W) | DevOps | 29.2 |
| 29.4 | Оптимізувати memory usage | ML | 29.3 |
| 29.5 | Досягти target FPS (мінімум 15 FPS) | ML | 29.4 |

---

#### **Тиждень 20: Demo & Documentation**

| # | Завдання | Тип | Залежність |
|---|----------|-----|------------|
| 30.1 | Підготувати demo environment | DevOps | All |
| 30.2 | Створити sample dataset (тестові обличчя) | Data | — |
| 30.3 | Написати demo script (use cases) | Docs | 30.1, 30.2 |
| 30.4 | Записати demo video | Docs | 30.3 |
| 30.5 | Написати Edge deployment guide | Docs | — |
| 30.6 | Написати technical report (findings, benchmarks) | Docs | — |
| 30.7 | Написати recommendations для production | Docs | 30.6 |
| 30.8 | Провести final review з stakeholders | Review | 30.7 |

---

## 📊 Підсумок

| Phase | Тривалість | Кількість завдань | Hardware |
|-------|------------|-------------------|----------|
| Phase 1 | 4 тижні | ~45 завдань | Standard server |
| Phase 2 | 6 тижнів | ~55 завдань | GPU recommended |
| Phase 3 | 4 тижні | ~35 завдань | Existing |
| Phase 4 | 6 тижнів | ~50 завдань | NVIDIA Dev Kit + Camera |
| **Total** | **20 тижнів** | **~185 завдань** | |

---

## 🏷️ Категорії завдань

| Категорія | Опис | Приблизна кількість |
|-----------|------|---------------------|
| **Backend** | Python/FastAPI розробка | ~80 |
| **Frontend** | React розробка | ~20 |
| **DevOps** | Docker, infrastructure | ~20 |
| **ML** | Machine Learning, моделі | ~25 |
| **Testing** | Unit/Integration/E2E тести | ~20 |
| **Docs** | Документація | ~15 |
| **Research** | Дослідження технологій | ~10 |