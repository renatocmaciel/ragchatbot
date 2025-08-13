# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Course Materials RAG (Retrieval-Augmented Generation) system that enables users to query course materials and receive AI-powered responses. The system uses ChromaDB for vector storage, Anthropic's Claude for AI generation, and provides a web interface for interaction.

## Development Commands

### Environment Setup
```bash
# Install dependencies
uv sync

# Set up environment variables (create .env file)
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

### Running the Application
```bash
# Quick start using provided script
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Access Points
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Architecture

### Backend Structure (`backend/`)
- **`app.py`**: FastAPI application with CORS middleware, API endpoints for querying and course statistics
- **`rag_system.py`**: Main orchestrator that coordinates all components (DocumentProcessor, VectorStore, AIGenerator, SessionManager)
- **`vector_store.py`**: ChromaDB interface for storing and retrieving course content and metadata
- **`ai_generator.py`**: Anthropic Claude integration for generating responses
- **`document_processor.py`**: Processes course documents (PDF, DOCX, TXT) into chunks and course metadata
- **`session_manager.py`**: Manages conversation history for context-aware responses
- **`search_tools.py`**: Tool-based search system for the AI to query course content
- **`models.py`**: Pydantic models for Course, Lesson, and CourseChunk data structures
- **`config.py`**: Configuration management for chunk sizes, models, and API settings

### Frontend Structure (`frontend/`)
- **`index.html`**: Main web interface
- **`script.js`**: JavaScript for API interaction and UI logic
- **`style.css`**: Styling for the web interface

### Data Flow
1. Course documents in `docs/` are processed into chunks and stored in ChromaDB
2. User queries are sent to the RAG system via FastAPI endpoints
3. The AI generator uses search tools to find relevant course content
4. Responses are generated using retrieved context and conversation history
5. Results are returned with source references

### Key Components Integration
- **RAGSystem**: Central coordinator that initializes and manages all other components
- **Tool-based Search**: AI uses CourseSearchTool to dynamically search for relevant content
- **Session Management**: Maintains conversation context across multiple queries
- **Vector Storage**: ChromaDB stores both course metadata and content chunks for semantic search

## Dependencies

- **Core**: FastAPI, uvicorn, ChromaDB, Anthropic, sentence-transformers
- **Utilities**: python-multipart, python-dotenv

## Environment Requirements

- Python 3.13+
- uv package manager
- Anthropic API key

## Development Guidelines

- **Package Management**: Always use `uv` instead of `pip` directly for all Python package operations (install, add, remove, etc.)
- **Python Execution**: Always use `uv run` instead of `python` directly to run Python files and scripts