# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## System Architecture

This is a RAG (Retrieval-Augmented Generation) chatbot system built with FastAPI, ChromaDB, and Anthropic Claude. The system processes educational course materials and provides intelligent responses through semantic search and AI generation.

### Core Components

- **RAG System (`rag_system.py`)**: Central orchestrator managing document processing, vector storage, AI generation, and tool-based interactions
- **Document Processor (`document_processor.py`)**: Parses structured course documents, extracts lessons, and performs sentence-based chunking with configurable overlap
- **Vector Store (`vector_store.py`)**: ChromaDB integration with dual collections - `course_catalog` (metadata) and `course_content` (chunks) for semantic search
- **AI Generator (`ai_generator.py`)**: Anthropic Claude API integration with tool-calling support and conversation history management
- **Search Tools (`search_tools.py`)**: Tool-based architecture allowing Claude to dynamically search course content with course name and lesson filtering
- **Session Manager (`session_manager.py`)**: Manages conversation context with configurable history limits

### Data Flow

1. Documents in `docs/` folder are auto-loaded on startup
2. Course documents are parsed for metadata (title, instructor, lessons)
3. Content is chunked using sentence boundaries with 800 char limit and 100 char overlap
4. Embeddings stored in ChromaDB using SentenceTransformers all-MiniLM-L6-v2
5. User queries trigger Claude with tool-calling capability
6. Claude uses CourseSearchTool for semantic content retrieval
7. Final responses synthesized with retrieved context and conversation history

## Development Commands

### Environment Setup
```bash
# Install dependencies (requires UV package manager)
uv sync

# Create environment file with Anthropic API key
echo "ANTHROPIC_API_KEY=your_key_here" > .env
```

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh && ./run.sh

# Manual start for debugging
cd backend
uv run uvicorn app:app --reload --port 8000

# With debug logging
uv run uvicorn app:app --reload --port 8000 --log-level debug
```

### Access Points
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

### Common Operations
```bash
# Clear vector database for fresh document loading
rm -rf backend/chroma_db/

# Add new dependencies
uv add package_name
```

## Configuration

### Required Environment Variables
- `ANTHROPIC_API_KEY`: Anthropic Claude API key (required)

### Key Configuration (`backend/config.py`)
- `CHUNK_SIZE: 800` - Text chunk size for vector storage
- `CHUNK_OVERLAP: 100` - Character overlap between chunks
- `MAX_RESULTS: 5` - Maximum search results returned
- `MAX_HISTORY: 2` - Conversation messages to remember
- `ANTHROPIC_MODEL: "claude-sonnet-4-20250514"` - Claude model version

## Document Format

Course documents in `docs/` should follow this structure:
```
Course Title: [Title]
Course Link: [URL]
Course Instructor: [Name]

Lesson 0: Introduction
Lesson Link: [Optional URL]
[Lesson content...]

Lesson 1: Next Topic
[Content continues...]
```

## API Endpoints

- `POST /api/query`: Main chatbot interaction
  - Request: `{"query": "string", "session_id": "optional"}`
  - Response: `{"answer": "string", "sources": ["string"], "session_id": "string"}`
- `GET /api/courses`: Course statistics and analytics

## Troubleshooting

### Common Issues
1. **500 errors on queries**: Check `.env` file exists with valid `ANTHROPIC_API_KEY`
2. **"Loaded 0 chunks"**: Documents exist but no content processed - clear ChromaDB with `rm -rf backend/chroma_db/`
3. **Windows compatibility**: Use Git Bash for shell commands, not PowerShell/CMD
4. **Startup delays**: First run downloads embedding models (~100MB+ from Hugging Face)

### Development Notes
- ChromaDB data persists in `backend/chroma_db/` directory
- Document processing skips existing courses based on title matching
- Frontend uses no-cache headers for development
- Session management is in-memory only
- always use uv to run de server do not use pip directly
- use uv to run Python files
- no corras el servidor usando ./run.sh Yo lo voy a correr por mi cuenta