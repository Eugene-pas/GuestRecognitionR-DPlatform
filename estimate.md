# ID Card Parser — Technical Demo Estimate

---

### 1. Project Title

**ID Card Parser Tech Demo** — Basic technical demo for ID card recognition system

---

### 2. Brief Description

A console service for automatic recognition and parsing of ID cards (passports, identity cards, etc.). The system receives a photo or scan of a document as input, extracts key data (name, document number, date of birth, country, etc.), saves results to a database, and provides a primitive API for accessing stored data.

The goal is to create a working prototype (tech demo) that demonstrates basic functionality without complex infrastructure, UI, or ML platforms (Nvidia, vector databases). This allows for a quick start and obtaining initial results for further demonstration.

---

### 3. Scope (included)

- Console ID card parser (backend utility)
- OCR text recognition from documents
- Extraction of main fields: first name, last name, document number, date of birth, issue date, country, gender, etc.
- Extraction and saving of person's photo from document
- Results formatting in JSON format
- Saving recognized data to database
- Primitive REST API:
  - `POST /upload` — photo/scan upload
  - `GET /documents` — get list of documents
  - `GET /documents/{id}` — get specific document
- Testing on public demo ID cards from the internet

---

### 4. Out of Scope (not included)

- User Interface (UI)
- React application
- Email service for document reception
- Nvidia Dev Kit / GPU infrastructure
- Vector databases
- Complex ML architecture
- Production deployment
- Authentication and authorization
- Video stream processing
- Multilingual OCR support (basic languages only)

---

### 5. Main Work Phases

#### Phase 1: Environment and Project Structure Setup
- Repository setup, dependencies, basic structure
- **Output:** Ready project with basic configuration

#### Phase 2: OCR Engine Integration
- Integration of OCR library (Tesseract, EasyOCR, or similar)
- Basic text recognition from images
- **Output:** Module returning raw text from photo

#### Phase 3: ID Card Parser
- Logic for field extraction from recognized text
- Processing different ID card formats (EU, UA, etc.)
- Data validation and normalization
- **Output:** JSON with recognized document fields

#### Phase 4: Photo Extraction from Document
- Detection and cropping of person's photo from ID card
- Saving photo to file system or database
- **Output:** Separate photo file + path in results

#### Phase 5: Database
- Database schema design (documents, fields, photos)
- Implementation of data saving and reading
- **Output:** Working database with stored documents

#### Phase 6: REST API
- Endpoint for photo upload (POST)
- Endpoints for data retrieval (GET)
- Basic error handling
- **Output:** Working API for system interaction

#### Phase 7: Testing and Documentation
- Testing on demo ID cards
- Basic API documentation
- **Output:** Tested solution + README

---

### 6. Estimate (approximate time assessment)

| Phase | Description | Time Estimate |
|-------|-------------|---------------|
| 1 | Environment and project structure setup | 2-4 hours |
| 2 | OCR engine integration | 4-8 hours |
| 3 | ID card parser (field extraction) | 8-16 hours |
| 4 | Photo extraction from document | 4-8 hours |
| 5 | Database (schema + CRUD) | 4-6 hours |
| 6 | REST API (POST + GET) | 4-8 hours |
| 7 | Testing and documentation | 4-6 hours |

---

### 7. Total Estimate

| Scenario | Estimate |
|----------|----------|
| **Minimum** | 30 hours (~4 working days) |
| **Maximum** | 56 hours (~7 working days) |
| **Realistic estimate** | 40-48 hours (~5-6 working days) |

> ⚠️ **This is a pre-estimate** — a preliminary assessment for understanding work scope. A more accurate estimate is possible after detailed requirements analysis and specific technology selection.

---

### 8. Assumptions and Risks

**Assumptions:**
- Scan/photo quality will be sufficient for OCR
- Using ready-made OCR libraries (no model training)
- ID card formats are limited (EU, UA)
- Test data is available from open sources

**Risks:**
- 🔴 Low input image quality — may significantly reduce recognition accuracy
- 🟡 ID card format variety — requires additional time for parser adaptation
- 🟡 Photo extraction from document — may be more complex for non-standard layouts
- 🟢 OCR engine selection — different libraries have varying quality for different languages
- 🟢 Date and name formats — may differ across countries
