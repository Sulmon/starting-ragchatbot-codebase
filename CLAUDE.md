# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start the application:**
```bash
./run.sh
```
or manually:
```bash
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Install dependencies:**
```bash
uv sync
```

**Environment setup:**
Create `.env` file with:
```
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for course materials with a full-stack architecture:

### Core Architecture Pattern
The system follows a **tool-augmented RAG pattern** where Claude autonomously decides when to search course content:

1. **User Query** → FastAPI endpoint (`/api/query`)
2. **RAG System** orchestrates the flow (`rag_system.py`)  
3. **AI Generator** calls Claude with available search tools (`ai_generator.py`)
4. **Claude decides** whether to use search tool based on query
5. **Search Tool** performs semantic search if needed (`search_tools.py`)
6. **Vector Store** queries ChromaDB with embeddings (`vector_store.py`)
7. **Response flows back** with sources for transparency

### Key Components

**Backend (`backend/`):**
- `app.py`: FastAPI server with CORS, query endpoint, course stats endpoint
- `rag_system.py`: Main orchestrator that coordinates all components
- `ai_generator.py`: Claude API integration with tool calling support
- `search_tools.py`: Tool interface for Claude to search course content
- `vector_store.py`: ChromaDB wrapper with semantic search and filtering
- `document_processor.py`: Parses course documents, creates chunks with context
- `session_manager.py`: Conversation history management
- `models.py`: Pydantic models for Course, Lesson, CourseChunk
- `config.py`: Centralized configuration with environment variables

**Frontend (`frontend/`):**
- `index.html`: Single-page chat interface
- `script.js`: Handles user interaction, API calls, session management  
- `style.css`: UI styling with chat bubbles and responsive design

### Document Processing Flow
1. **Parse structure**: Course title, instructor, lessons from structured text files
2. **Chunk creation**: Sentence-based chunking with configurable overlap
3. **Context enhancement**: Chunks prefixed with course/lesson context
4. **Vector storage**: Embeddings stored in ChromaDB with metadata

### Query Processing Flow
1. **Session continuity**: Frontend maintains session_id for conversation context
2. **Tool autonomy**: Claude decides when to search based on query analysis
3. **Semantic search**: Vector similarity search with course/lesson filtering
4. **Source tracking**: Search results tracked and returned to frontend for display
5. **Conversation history**: Previous exchanges included in each Claude call

### Configuration
All settings centralized in `config.py`:
- `CHUNK_SIZE`: Text chunk size for embeddings (default 800)
- `CHUNK_OVERLAP`: Character overlap between chunks (default 100)
- `MAX_RESULTS`: Search result limit (default 5)
- `MAX_HISTORY`: Conversation history limit (default 2)
- `ANTHROPIC_MODEL`: Claude model version
- `EMBEDDING_MODEL`: Sentence transformer model for embeddings

### Expected Document Format
Course documents in `docs/` should follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [lesson_url]
[lesson content...]

Lesson 1: [title]
[lesson content...]
```

### Data Models
- **Course**: title (unique ID), optional link/instructor, lessons list
- **Lesson**: number, title, optional link
- **CourseChunk**: content with course/lesson context, metadata for search

### API Endpoints
- `POST /api/query`: Main query interface (query, session_id → answer, sources, session_id)
- `GET /api/courses`: Course analytics (total_courses, course_titles)
- `GET /`: Serves frontend static files

### Vector Store Collections
- `course_catalog`: Course metadata for semantic course name matching
- `course_content`: Actual course chunks with lesson/course metadata

The system automatically loads documents from `docs/` folder on startup and avoids re-processing existing courses.