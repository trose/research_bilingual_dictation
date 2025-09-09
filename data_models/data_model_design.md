# Data Model Design for Bilingual Veterinary Dictation System

## Core Data Models

### 1. Dictation Session Model
```json
{
  "session_id": "uuid",
  "user_id": "uuid",
  "timestamp": "2024-01-15T10:30:00Z",
  "duration_seconds": 120,
  "language_detected": "en",
  "language_confidence": 0.95,
  "audio_quality_score": 0.88,
  "processing_status": "completed",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:32:00Z"
}
```

### 2. Audio File Model
```json
{
  "audio_id": "uuid",
  "session_id": "uuid",
  "file_path": "/audio/session_123.wav",
  "file_size_bytes": 2048000,
  "duration_seconds": 120,
  "sample_rate": 44100,
  "channels": 1,
  "format": "wav",
  "compression_ratio": 0.75,
  "quality_metrics": {
    "snr_db": 25.5,
    "clarity_score": 0.92,
    "noise_level": 0.15
  },
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 3. Transcription Model
```json
{
  "transcription_id": "uuid",
  "session_id": "uuid",
  "audio_id": "uuid",
  "raw_text": "The patient presents with acute lameness in the left hind limb",
  "processed_text": "The patient presents with acute lameness in the left hind limb",
  "language": "en",
  "confidence_score": 0.94,
  "processing_time_ms": 2500,
  "model_version": "whisper-v1.0",
  "segments": [
    {
      "start_time": 0.0,
      "end_time": 3.2,
      "text": "The patient presents with",
      "confidence": 0.96
    },
    {
      "start_time": 3.2,
      "end_time": 6.8,
      "text": "acute lameness in the left hind limb",
      "confidence": 0.92
    }
  ],
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 4. Veterinary Terminology Model
```json
{
  "term_id": "uuid",
  "term": "lameness",
  "spanish_term": "cojera",
  "category": "symptom",
  "species": ["dog", "cat", "horse"],
  "body_system": "musculoskeletal",
  "severity_levels": ["mild", "moderate", "severe"],
  "related_terms": ["limping", "gait abnormality", "pain"],
  "context_examples": [
    "acute lameness",
    "chronic lameness",
    "lameness evaluation"
  ],
  "confidence": 0.98,
  "source": "vet_lexicon",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 5. Vector Embedding Model
```json
{
  "embedding_id": "uuid",
  "transcription_id": "uuid",
  "text_segment": "acute lameness in the left hind limb",
  "embedding_vector": [0.1, -0.3, 0.7, ...],
  "vector_dimensions": 768,
  "model_name": "sentence-transformers/vet-specialized",
  "similarity_threshold": 0.85,
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 6. Structured Output Model
```json
{
  "output_id": "uuid",
  "session_id": "uuid",
  "transcription_id": "uuid",
  "output_type": "soap_note",
  "structured_data": {
    "subjective": {
      "chief_complaint": "acute lameness",
      "history": "Owner reports sudden onset of limping",
      "duration": "2 days"
    },
    "objective": {
      "physical_exam": {
        "gait": "abnormal",
        "weight_bearing": "reduced on left hind",
        "pain_response": "positive on palpation"
      },
      "vital_signs": {
        "temperature": "102.1°F",
        "heart_rate": "120 bpm",
        "respiratory_rate": "24 bpm"
      }
    },
    "assessment": {
      "primary_diagnosis": "suspected musculoskeletal injury",
      "differential_diagnoses": [
        "soft tissue injury",
        "fracture",
        "joint inflammation"
      ]
    },
    "plan": {
      "diagnostic_tests": ["radiographs", "blood work"],
      "treatment": ["pain management", "rest"],
      "follow_up": "recheck in 1 week"
    }
  },
  "confidence_scores": {
    "overall": 0.91,
    "subjective": 0.88,
    "objective": 0.94,
    "assessment": 0.85,
    "plan": 0.92
  },
  "created_at": "2024-01-15T10:30:00Z"
}
```

## Database Schema Design

### SQLite Schema (Local Storage)
```sql
-- Sessions table
CREATE TABLE sessions (
    session_id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    timestamp DATETIME NOT NULL,
    duration_seconds INTEGER,
    language_detected TEXT,
    language_confidence REAL,
    audio_quality_score REAL,
    processing_status TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Audio files table
CREATE TABLE audio_files (
    audio_id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    file_path TEXT NOT NULL,
    file_size_bytes INTEGER,
    duration_seconds REAL,
    sample_rate INTEGER,
    channels INTEGER,
    format TEXT,
    compression_ratio REAL,
    quality_metrics TEXT, -- JSON
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (session_id) REFERENCES sessions(session_id)
);

-- Transcriptions table
CREATE TABLE transcriptions (
    transcription_id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    audio_id TEXT NOT NULL,
    raw_text TEXT,
    processed_text TEXT,
    language TEXT,
    confidence_score REAL,
    processing_time_ms INTEGER,
    model_version TEXT,
    segments TEXT, -- JSON
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (session_id) REFERENCES sessions(session_id),
    FOREIGN KEY (audio_id) REFERENCES audio_files(audio_id)
);

-- Veterinary terminology table
CREATE TABLE veterinary_terms (
    term_id TEXT PRIMARY KEY,
    term TEXT NOT NULL,
    spanish_term TEXT,
    category TEXT,
    species TEXT, -- JSON array
    body_system TEXT,
    severity_levels TEXT, -- JSON array
    related_terms TEXT, -- JSON array
    context_examples TEXT, -- JSON array
    confidence REAL,
    source TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Vector embeddings table
CREATE TABLE vector_embeddings (
    embedding_id TEXT PRIMARY KEY,
    transcription_id TEXT NOT NULL,
    text_segment TEXT,
    embedding_vector BLOB, -- Serialized vector
    vector_dimensions INTEGER,
    model_name TEXT,
    similarity_threshold REAL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (transcription_id) REFERENCES transcriptions(transcription_id)
);

-- Structured outputs table
CREATE TABLE structured_outputs (
    output_id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    transcription_id TEXT NOT NULL,
    output_type TEXT,
    structured_data TEXT, -- JSON
    confidence_scores TEXT, -- JSON
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (session_id) REFERENCES sessions(session_id),
    FOREIGN KEY (transcription_id) REFERENCES transcriptions(transcription_id)
);
```

### Vector Database Schema (Embeddings)
```sql
-- Vector similarity search table
CREATE VIRTUAL TABLE vector_search USING fts5(
    embedding_id,
    text_segment,
    content='vector_embeddings',
    content_rowid='rowid'
);

-- Index for fast similarity search
CREATE INDEX idx_vector_embeddings_model ON vector_embeddings(model_name);
CREATE INDEX idx_vector_embeddings_transcription ON vector_embeddings(transcription_id);
```

## Data Relationships

### Entity Relationship Diagram
```
Sessions (1) ──→ (N) Audio Files
Sessions (1) ──→ (N) Transcriptions
Sessions (1) ──→ (N) Structured Outputs
Audio Files (1) ──→ (1) Transcriptions
Transcriptions (1) ──→ (N) Vector Embeddings
Transcriptions (1) ──→ (1) Structured Outputs
Veterinary Terms (N) ──→ (N) Transcriptions (Many-to-Many)
```

### Data Flow Relationships
```
Audio Input → Audio File → Transcription → Vector Embedding
                ↓              ↓              ↓
            Session ← Structured Output ← Veterinary Terms
```

## Data Storage Strategies

### Local Storage (SQLite)
**Advantages:**
- No network dependency
- Fast local access
- Complete privacy
- Offline operation

**Use Cases:**
- Primary data storage
- User data and preferences
- Cached results
- Offline operation

### Vector Database (Local)
**Options:**
- **Chroma**: Lightweight vector database
- **FAISS**: Facebook's similarity search library
- **Annoy**: Spotify's approximate nearest neighbors

**Use Cases:**
- Semantic search
- Similarity matching
- Recommendation systems
- Knowledge retrieval

### Cloud Storage (Optional)
**Options:**
- **AWS S3**: Object storage
- **Google Cloud Storage**: Object storage
- **Azure Blob Storage**: Object storage

**Use Cases:**
- Backup and sync
- Cross-device access
- Long-term storage
- Sharing and collaboration

## Data Processing Pipeline

### 1. Audio Processing
```
Raw Audio → Preprocessing → Segmentation → Quality Assessment → Storage
```

### 2. Text Processing
```
Audio → Speech-to-Text → Language Detection → Veterinary Processing → Storage
```

### 3. Vector Processing
```
Text → Embedding Generation → Vector Storage → Index Creation → Search Ready
```

### 4. Structured Output Processing
```
Text → Veterinary Analysis → Structure Generation → Validation → Storage
```

## Data Quality and Validation

### Quality Metrics
**Audio Quality:**
- Signal-to-noise ratio
- Clarity score
- Noise level
- Duration validation

**Text Quality:**
- Confidence scores
- Language detection accuracy
- Veterinary terminology accuracy
- Completeness validation

**Vector Quality:**
- Embedding dimensions
- Similarity thresholds
- Model version tracking
- Performance metrics

### Validation Rules
**Data Integrity:**
- Required field validation
- Data type validation
- Range validation
- Relationship validation

**Business Rules:**
- Veterinary terminology accuracy
- Language consistency
- Medical accuracy
- User privacy compliance

## Performance Optimization

### Indexing Strategy
**Primary Indexes:**
- Session ID (primary key)
- User ID (for user queries)
- Timestamp (for time-based queries)
- Language (for language filtering)

**Secondary Indexes:**
- Audio quality score
- Confidence scores
- Model versions
- Processing status

### Query Optimization
**Common Queries:**
- User session history
- Audio quality analysis
- Transcription accuracy
- Vector similarity search

**Optimization Techniques:**
- Composite indexes
- Query plan analysis
- Caching strategies
- Lazy loading

## Data Security and Privacy

### Encryption
**At Rest:**
- Database encryption
- File system encryption
- Key management
- Access control

**In Transit:**
- TLS encryption
- API security
- Authentication
- Authorization

### Privacy Protection
**Data Minimization:**
- Only necessary data collection
- Automatic data expiration
- User data deletion
- Anonymization options

**Access Control:**
- User authentication
- Role-based access
- Data isolation
- Audit logging

## Backup and Recovery

### Backup Strategy
**Local Backups:**
- Incremental backups
- Compressed storage
- Version control
- Integrity checking

**Cloud Backups:**
- Encrypted backups
- Geographic distribution
- Version retention
- Recovery testing

### Recovery Procedures
**Data Recovery:**
- Point-in-time recovery
- Selective recovery
- Integrity validation
- Performance testing

**Disaster Recovery:**
- Backup validation
- Recovery procedures
- Testing protocols
- Documentation
