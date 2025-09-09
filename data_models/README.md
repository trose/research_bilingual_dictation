# Data Models

This directory contains data model designs and schemas for the bilingual veterinary dictation system.

## Contents

### Data Model Design
- **`data_model_design.md`** - Comprehensive data models and schemas for the entire system

## TLDR

The data model design defines the core data structures for the bilingual veterinary dictation system:

**Key Models:**
- **Dictation Session**: User sessions with metadata and processing status
- **Audio File**: Audio storage with quality metrics and compression
- **Transcription**: Text output with confidence scores and veterinary terminology
- **Veterinary Terms**: Specialized terminology with translations and relationships
- **User Profile**: User preferences and language settings
- **Performance Metrics**: System performance and accuracy tracking

**Design Principles:**
- JSON-based schemas for flexibility
- UUID-based identifiers for scalability
- Comprehensive metadata for analytics
- Bilingual support with language detection
- Veterinary terminology integration
- Performance monitoring and quality metrics

The models support both mobile and cloud deployment with efficient data structures optimized for real-time processing.
