# Data Models Design

## TLDR
Data model design for bilingual veterinary dictation system, covering audio storage, text processing, and vector embeddings.

## Key Decisions Made
- **Audio Storage**: Compressed WAV/MP3 format with metadata
- **Text Storage**: Structured JSON with veterinary terminology annotations
- **Vector Storage**: 768-dimensional embeddings for semantic search
- **Database**: SQLite for local storage, Vector DB for embeddings

## Files
- `data_model_design.md` - Comprehensive data model specifications and relationships

## Research Findings
- **Audio Format**: Compressed storage to minimize space usage
- **Text Structure**: JSON format with veterinary terminology metadata
- **Vector Dimensions**: 768-dimensional embeddings for optimal semantic search
- **Storage Requirements**: <1GB total per user with efficient compression
- **Data Relationships**: Hierarchical structure for veterinary terminology
- **Privacy**: Local storage with optional cloud backup