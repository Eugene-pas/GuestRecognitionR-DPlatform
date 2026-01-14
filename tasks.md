## 📋 Complete Task List: Guest Recognition R&D Platform

### **PHASE 1: Infrastructure Foundation (Weeks 1-4)**

---

#### **Week 1: Basic Infrastructure**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 1.1 | Create Git repository with `.gitignore`, `README.md` | Setup | — |
| 1.2 | Create basic project directory structure | Setup | 1.1 |
| 1.3 | Write `docker-compose.yml` with services: PostgreSQL, Redis, MinIO | DevOps | 1.2 |
| 1.4 | Configure PostgreSQL with pgvector extension | DevOps | 1.3 |
| 1.5 | Create FastAPI project skeleton | Backend | 1.2 |
| 1.6 | Implement health check endpoints (`/health`, `/ready`) | Backend | 1.5 |
| 1.7 | Configure logging (structlog / loguru) | Backend | 1.5 |
| 1.8 | Create basic `Dockerfile` for API | DevOps | 1.5 |

---

#### **Week 2: Database & Storage**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 2.1 | Configure Alembic for migrations | Backend | 1.4 |
| 2.2 | Create migration: `guests` table | Backend | 2.1 |
| 2.3 | Create migration: `documents` table | Backend | 2.2 |
| 2.4 | Create migration: `ocr_results` table | Backend | 2.3 |
| 2.5 | Create migration: `face_photos` table | Backend | 2.4 |
| 2.6 | Create migration: `face_embeddings` table (with VECTOR) | Backend | 2.5 |
| 2.7 | Create migration: `processing_jobs` table | Backend | 2.6 |
| 2.8 | Implement SQLAlchemy models for all tables | Backend | 2.7 |
| 2.9 | Implement Repository pattern for Guest | Backend | 2.8 |
| 2.10 | Configure MinIO bucket for files | DevOps | 1.3 |
| 2.11 | Implement FileStorage service (abstraction over MinIO/local) | Backend | 2.10 |
| 2.12 | Create directory structure for files (`/raw/`, `/processed/`) | Backend | 2.11 |

---

#### **Week 3: Mail Ingestion**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 3.1 | Research Python IMAP libraries (aioimaplib vs imapclient) | Research | — |
| 3.2 | Implement IMAP client with SSL support | Backend | 3.1 |
| 3.3 | Implement polling mechanism with configurable interval | Backend | 3.2 |
| 3.4 | Implement email attachment extraction | Backend | 3.3 |
| 3.5 | Implement MIME type validation (JPEG, PNG, PDF) | Backend | 3.4 |
| 3.6 | Implement file size validation (max 20MB) | Backend | 3.5 |
| 3.7 | Implement processed email marking (move to folder) | Backend | 3.6 |
| 3.8 | Add retry logic with exponential backoff | Backend | 3.7 |
| 3.9 | Implement configuration through environment variables | Backend | 3.8 |
| 3.10 | Write unit tests for Mail Ingestion | Testing | 3.9 |
| 3.11 | Create Dockerfile for mail ingestion daemon | DevOps | 3.9 |

---

#### **Week 4: Web App Skeleton**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 4.1 | Initialize React project with Vite | Frontend | — |
| 4.2 | Configure Tailwind CSS + shadcn/ui | Frontend | 4.1 |
| 4.3 | Create basic layout (header, main area) | Frontend | 4.2 |
| 4.4 | Implement Camera component (MediaDevices API) | Frontend | 4.3 |
| 4.5 | Implement photo capture from camera | Frontend | 4.4 |
| 4.6 | Implement File Upload component (drag & drop) | Frontend | 4.3 |
| 4.7 | Implement image preview | Frontend | 4.5, 4.6 |
| 4.8 | Implement Guest Information form | Frontend | 4.3 |
| 4.9 | Integrate React Query for API calls | Frontend | 4.8 |
| 4.10 | Implement POST /api/v1/guests endpoint | Backend | 2.9 |
| 4.11 | Implement POST /api/v1/guests/{id}/photos endpoint | Backend | 4.10, 2.11 |
| 4.12 | Implement POST /api/v1/guests/{id}/documents endpoint | Backend | 4.10, 2.11 |
| 4.13 | Integrate frontend with backend API | Frontend | 4.9, 4.12 |
| 4.14 | Write E2E test: upload photo + document flow | Testing | 4.13 |
| 4.15 | Create Dockerfile for web app (nginx) | DevOps | 4.13 |

---

### **PHASE 2: Core Processing (Weeks 5-10)**

---

#### **Week 5: Task Queue Setup**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 5.1 | Configure Celery with Redis broker | Backend | Phase 1 |
| 5.2 | Create basic worker structure | Backend | 5.1 |
| 5.3 | Implement OCR queue | Backend | 5.2 |
| 5.4 | Implement Face processing queue | Backend | 5.2 |
| 5.5 | Implement File processing queue | Backend | 5.2 |
| 5.6 | Configure Flower for Celery monitoring | DevOps | 5.1 |
| 5.7 | Implement job status tracking (processing_jobs table) | Backend | 5.2, 2.7 |
| 5.8 | Add retry logic for failed jobs | Backend | 5.7 |

---

#### **Week 5-6: Azure OCR Integration**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 6.1 | Create Azure Document Intelligence resource | Setup | — |
| 6.2 | Implement Azure OCR client | Backend | 6.1 |
| 6.3 | Implement prebuilt-idDocument model call | Backend | 6.2 |
| 6.4 | Implement response parsing to StructuredDocumentData | Backend | 6.3 |
| 6.5 | Implement OCR results saving to DB | Backend | 6.4, 2.4 |
| 6.6 | Write unit tests for OCR client (with mocks) | Testing | 6.5 |

---

#### **Week 6: OCR Preprocessing**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 7.1 | Implement image resize (max dimension) | Backend | — |
| 7.2 | Implement deskew (tilt correction) | Backend | — |
| 7.3 | Implement contrast enhancement (CLAHE) | Backend | — |
| 7.4 | Implement PDF to image conversion | Backend | — |
| 7.5 | Combine preprocessing into pipeline | Backend | 7.1-7.4 |
| 7.6 | Integrate preprocessing into OCR worker | Backend | 7.5, 5.3 |

---

#### **Week 6: MRZ Parsing**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 8.1 | Research MRZ format (TD1, TD2, TD3) | Research | — |
| 8.2 | Implement MRZ line detection | Backend | 8.1 |
| 8.3 | Implement MRZ parsing (fields) | Backend | 8.2 |
| 8.4 | Implement check digit validation | Backend | 8.3 |
| 8.5 | Integrate MRZ parsing into OCR pipeline | Backend | 8.4, 6.4 |
| 8.6 | Write tests with sample MRZ data | Testing | 8.5 |

---

#### **Week 7: Data Normalization**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 9.1 | Implement name normalization (uppercase, diacritics) | Backend | — |
| 9.2 | Implement date normalization (various formats → ISO) | Backend | — |
| 9.3 | Implement country code mapping (ISO 3166-1 alpha-3) | Backend | — |
| 9.4 | Implement transliteration (Cyrillic → Latin) | Backend | — |
| 9.5 | Combine into DataNormalizationService | Backend | 9.1-9.4 |
| 9.6 | Integrate normalization into OCR worker | Backend | 9.5, 6.5 |

---

#### **Week 7-8: Face Detection**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 10.1 | Research MTCNN vs RetinaFace (accuracy, speed) | Research | — |
| 10.2 | Install chosen library | Backend | 10.1 |
| 10.3 | Implement FaceDetector class | Backend | 10.2 |
| 10.4 | Implement detect_faces(image) → list[BoundingBox] | Backend | 10.3 |
| 10.5 | Implement primary face selection (largest/centered) | Backend | 10.4 |
| 10.6 | Implement face cropping with padding | Backend | 10.5 |
| 10.7 | Implement cropped face saving to storage | Backend | 10.6, 2.11 |
| 10.8 | Integrate into Face worker | Backend | 10.7, 5.4 |

---

#### **Week 8: Face Quality Assessment**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 11.1 | Implement blur detection (Laplacian variance) | Backend | — |
| 11.2 | Implement brightness check (mean pixel value) | Backend | — |
| 11.3 | Implement face angle estimation | Backend | — |
| 11.4 | Implement face size check | Backend | — |
| 11.5 | Combine into QualityAssessment service | Backend | 11.1-11.4 |
| 11.6 | Integrate quality check into Face worker | Backend | 11.5, 10.8 |
| 11.7 | Save quality_score and quality_details to DB | Backend | 11.6, 2.5 |

---

#### **Week 9: Face Embeddings**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 12.1 | Research FaceNet vs ArcFace (embedding quality) | Research | — |
| 12.2 | Download pretrained model | Backend | 12.1 |
| 12.3 | Implement face alignment (eye alignment) | Backend | 10.4 |
| 12.4 | Implement EmbeddingGenerator class | Backend | 12.2 |
| 12.5 | Implement generate_embedding(face_image) → vector | Backend | 12.4 |
| 12.6 | Integrate embedding generation into Face worker | Backend | 12.5, 10.8 |
| 12.7 | Save embeddings to face_embeddings (pgvector) | Backend | 12.6, 2.6 |

---

#### **Week 9: Vector Search Setup**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 13.1 | Configure pgvector index (IVFFlat) | Backend | 12.7 |
| 13.2 | Implement similarity search (cosine distance) | Backend | 13.1 |
| 13.3 | Implement find_similar_faces(embedding, threshold) | Backend | 13.2 |
| 13.4 | Write tests for vector search | Testing | 13.3 |

---

#### **Week 10: Web App Completion**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 14.1 | Implement GET /api/v1/guests/{id}/status endpoint | Backend | 5.7 |
| 14.2 | Implement GET /api/v1/guests/recent endpoint | Backend | 2.9 |
| 14.3 | Implement status polling on frontend | Frontend | 14.1 |
| 14.4 | Add processing status indicator (spinner, progress) | Frontend | 14.3 |
| 14.5 | Implement History component (recent records) | Frontend | 14.2 |
| 14.6 | Add error handling and user feedback | Frontend | 14.4 |
| 14.7 | Style UI (responsive design) | Frontend | 14.6 |
| 14.8 | Write integration tests for complete flow | Testing | 14.7 |

---

### **PHASE 3: AI Enhancement (Weeks 11-14)**

---

#### **Week 11: Multi-Provider OCR**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 15.1 | Create Google Cloud Vision resource | Setup | — |
| 15.2 | Implement Google Vision OCR client | Backend | 15.1 |
| 15.3 | Implement OCRProvider abstract class | Backend | — |
| 15.4 | Adapt Azure and Google clients to OCRProvider | Backend | 15.3, 6.2, 15.2 |
| 15.5 | Implement fallback logic (Azure → Google) | Backend | 15.4 |
| 15.6 | Implement confidence-based provider selection | Backend | 15.5 |

---

#### **Week 11: Confidence & Review**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 16.1 | Define confidence thresholds for auto-accept | Config | — |
| 16.2 | Implement flagging for manual review (low confidence) | Backend | 16.1 |
| 16.3 | Add 'needs_review' status for guests | Backend | 16.2 |
| 16.4 | Implement GET /api/v1/guests/review-queue endpoint | Backend | 16.3 |
| 16.5 | Add Review Queue UI component | Frontend | 16.4 |

---

#### **Week 12: Face Matching**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 17.1 | Research optimal similarity threshold | Research | 13.3 |
| 17.2 | Implement match_face(guest_id) → list[MatchResult] | Backend | 17.1, 13.3 |
| 17.3 | Implement ranking by similarity score | Backend | 17.2 |
| 17.4 | Add match results to guest record | Backend | 17.3 |
| 17.5 | Implement UI for showing potential matches | Frontend | 17.4 |

---

#### **Week 12: Duplicate Detection**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 18.1 | Implement document-based duplicate check | Backend | 6.5 |
| 18.2 | Implement face-based duplicate check | Backend | 17.2 |
| 18.3 | Combine into DuplicateDetector service | Backend | 18.1, 18.2 |
| 18.4 | Add duplicate warning to guest creation flow | Backend | 18.3 |
| 18.5 | Add duplicate notification to UI | Frontend | 18.4 |

---

#### **Week 13: Quality Improvements**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 19.1 | Improve low-quality image processing | Backend | 7.5 |
| 19.2 | Add auto-rotation based on EXIF | Backend | 7.5 |
| 19.3 | Improve handling of various document orientations | Backend | 6.3 |
| 19.4 | Add support for multi-page PDF | Backend | 7.4 |
| 19.5 | Improve error messages for users | Frontend | — |

---

#### **Week 13: Performance Optimization**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 20.1 | Implement batch OCR processing | Backend | 6.5 |
| 20.2 | Implement batch embedding generation | Backend | 12.6 |
| 20.3 | Add caching for OCR results | Backend | 6.5 |
| 20.4 | Optimize database queries (explain analyze) | Backend | — |
| 20.5 | Configure connection pooling | Backend | — |

---

#### **Week 14: Documentation & Testing**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 21.1 | Write API documentation (OpenAPI/Swagger) | Docs | — |
| 21.2 | Write deployment guide (docker-compose) | Docs | — |
| 21.3 | Write operations runbook | Docs | — |
| 21.4 | Write architecture decision records (ADRs) | Docs | — |
| 21.5 | Conduct load testing (locust/k6) | Testing | — |
| 21.6 | Conduct security review | Testing | — |
| 21.7 | Fix found issues | Backend | 21.5, 21.6 |

---

### **PHASE 4: Edge Integration (Weeks 15-20)**

> ⚠️ **Requires:** NVIDIA Jetson Orin Dev Kit + Camera

---

#### **Week 15: Dev Kit Setup**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 22.1 | Install JetPack 6.x on Dev Kit | DevOps | Hardware |
| 22.2 | Configure CUDA and cuDNN | DevOps | 22.1 |
| 22.3 | Install TensorRT | DevOps | 22.2 |
| 22.4 | Install DeepStream SDK | DevOps | 22.3 |
| 22.5 | Configure network connectivity | DevOps | 22.1 |
| 22.6 | Connect and test camera | Hardware | 22.1 |
| 22.7 | Write test script for camera stream | Backend | 22.6 |

---

#### **Week 16: Model Conversion**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 23.1 | Convert face detection model to ONNX | ML | 10.3 |
| 23.2 | Convert ONNX to TensorRT engine | ML | 23.1, 22.3 |
| 23.3 | Convert embedding model to ONNX | ML | 12.4 |
| 23.4 | Convert embedding ONNX to TensorRT | ML | 23.3, 22.3 |
| 23.5 | Benchmark TensorRT models (FPS, latency) | Testing | 23.2, 23.4 |
| 23.6 | Optimize models (FP16, INT8 quantization) | ML | 23.5 |

---

#### **Week 16-17: DeepStream Pipeline**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 24.1 | Create basic DeepStream pipeline config | ML | 22.4 |
| 24.2 | Add camera source element | ML | 24.1, 22.6 |
| 24.3 | Add face detection inference | ML | 24.2, 23.2 |
| 24.4 | Add tracker element (NvDCF/IOU) | ML | 24.3 |
| 24.5 | Implement metadata extraction | ML | 24.4 |
| 24.6 | Add visualization (bounding boxes) | ML | 24.5 |
| 24.7 | Benchmark pipeline (FPS, GPU usage) | Testing | 24.6 |

---

#### **Week 17: Edge Face Recognition**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 25.1 | Integrate embedding inference into pipeline | ML | 24.5, 23.4 |
| 25.2 | Implement face cropping on edge | ML | 25.1 |
| 25.3 | Implement embedding generation on edge | ML | 25.2 |
| 25.4 | Implement local embedding cache | Backend | 25.3 |
| 25.5 | Implement local similarity search | Backend | 25.4 |
| 25.6 | Benchmark recognition pipeline | Testing | 25.5 |

---

#### **Week 18: Edge-Backend Integration**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 26.1 | Implement Edge API (FastAPI on Jetson) | Backend | 25.5 |
| 26.2 | Implement sync protocol (edge → backend) | Backend | 26.1 |
| 26.3 | Implement event streaming (detected faces) | Backend | 26.2 |
| 26.4 | Implement periodic embedding sync | Backend | 26.3 |
| 26.5 | Implement offline mode handling | Backend | 26.4 |
| 26.6 | Add edge status to backend dashboard | Frontend | 26.3 |

---

#### **Week 18: Multi-Face Tracking**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 27.1 | Research tracking algorithms (DeepSORT, ByteTrack) | Research | — |
| 27.2 | Configure tracker parameters | ML | 27.1, 24.4 |
| 27.3 | Implement track lifecycle management | ML | 27.2 |
| 27.4 | Implement re-identification when losing track | ML | 27.3 |
| 27.5 | Benchmark tracking accuracy | Testing | 27.4 |

---

#### **Week 19: V-JEPA Research**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 28.1 | Research V-JEPA architecture | Research | — |
| 28.2 | Assess hardware requirements for inference | Research | 28.1 |
| 28.3 | Test V-JEPA on Dev Kit (if possible) | Research | 28.2 |
| 28.4 | Assess applicability for guest behavior analysis | Research | 28.3 |
| 28.5 | Document findings and recommendations | Docs | 28.4 |

---

#### **Week 19: Performance Tuning**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 29.1 | Profile GPU usage | Testing | 25.6 |
| 29.2 | Optimize batch size for inference | ML | 29.1 |
| 29.3 | Configure power modes (MAX-N, 15W, 30W) | DevOps | 29.2 |
| 29.4 | Optimize memory usage | ML | 29.3 |
| 29.5 | Achieve target FPS (minimum 15 FPS) | ML | 29.4 |

---

#### **Week 20: Demo & Documentation**

| # | Task | Type | Dependencies |
|---|------|------|--------------|
| 30.1 | Prepare demo environment | DevOps | All |
| 30.2 | Create sample dataset (test faces) | Data | — |
| 30.3 | Write demo script (use cases) | Docs | 30.1, 30.2 |
| 30.4 | Record demo video | Docs | 30.3 |
| 30.5 | Write Edge deployment guide | Docs | — |
| 30.6 | Write technical report (findings, benchmarks) | Docs | — |
| 30.7 | Write production recommendations | Docs | 30.6 |
| 30.8 | Conduct final review with stakeholders | Review | 30.7 |

---

## 📊 Summary

| Phase | Duration | Number of Tasks | Hardware |
|-------|----------|----------------|----------|
| Phase 1 | 4 weeks | ~45 tasks | Standard server |
| Phase 2 | 6 weeks | ~55 tasks | GPU recommended |
| Phase 3 | 4 weeks | ~35 tasks | Existing |
| Phase 4 | 6 weeks | ~50 tasks | NVIDIA Dev Kit + Camera |
| **Total** | **20 weeks** | **~185 tasks** | |

---

## 🏷️ Task Categories

| Category | Description | Approximate Count |
|----------|-------------|------------------|
| **Backend** | Python/FastAPI development | ~80 |
| **Frontend** | React development | ~20 |
| **DevOps** | Docker, infrastructure | ~20 |
| **ML** | Machine Learning, models | ~25 |
| **Testing** | Unit/Integration/E2E tests | ~20 |
| **Docs** | Documentation | ~15 |
| **Research** | Technology research | ~10 |