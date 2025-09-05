# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start using provided script
chmod +x run.sh
./run.sh

# Manual start (from backend directory)
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add package-name
```

### Environment Setup
- Create `.env` file in root with `ANTHROPIC_API_KEY=your_key_here`
- Application runs on `http://localhost:8000`
- API docs available at `http://localhost:8000/docs`

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for course material Q&A with a FastAPI backend serving a static HTML/JS frontend.

### Core RAG Pipeline (backend/rag_system.py)
The `RAGSystem` class orchestrates the entire pipeline:
1. **Document Processing** → chunks course documents (PDF/DOCX/TXT)
2. **Vector Storage** → ChromaDB with semantic embeddings 
3. **Tool-Based Search** → Claude AI uses search tools to find relevant content
4. **AI Generation** → contextual responses with source citations
5. **Session Management** → conversation history for follow-ups

### Key Components

**Backend Architecture:**
- `app.py` - FastAPI server with `/api/query` and `/api/courses` endpoints
- `rag_system.py` - Main orchestrator connecting all components
- `ai_generator.py` - Claude API integration with tool execution
- `vector_store.py` - ChromaDB vector database operations
- `search_tools.py` - Tool definitions for AI-powered search
- `document_processor.py` - Text chunking and course metadata extraction
- `session_manager.py` - Conversation context handling
- `models.py` - Pydantic models (Course, Lesson, CourseChunk)
- `config.py` - Configuration with dataclass pattern

**Frontend:**
- `frontend/index.html` - Chat interface
- `frontend/script.js` - API calls, session management, UI updates

### Data Flow
1. **Startup**: Documents in `/docs` auto-loaded into ChromaDB
2. **Query**: User question → Frontend → FastAPI → RAGSystem
3. **Search**: Claude AI uses search tools to query vector database
4. **Response**: AI synthesizes findings → Backend → Frontend display
5. **Session**: Conversation history maintained for context

### Configuration Patterns
- Environment variables via `.env` file
- Centralized config in `backend/config.py` using dataclass
- Key settings: `CHUNK_SIZE=800`, `MAX_RESULTS=5`, `MAX_HISTORY=2`
- Embedding model: `all-MiniLM-L6-v2`
- AI model: `claude-sonnet-4-20250514`

### Tool System Design
The AI uses a tool-based approach rather than direct vector search:
- `ToolManager` registers and executes search tools
- `CourseSearchTool` performs semantic search on course content
- AI decides when to search vs. use general knowledge
- Sources automatically tracked from tool execution

### Document Processing
- Supports PDF, DOCX, TXT files in `/docs` folder
- Extracts course metadata (title, lessons, instructor)
- Creates overlapping chunks (800 chars, 100 char overlap)
- Deduplication prevents re-processing existing courses
- Metadata and content stored separately in ChromaDB

### Session Management
- UUID-based sessions for conversation continuity
- Stores query-response pairs with configurable history limit
- Frontend maintains session ID across page refreshes
- Context automatically included in subsequent AI calls

## Development Notes

### Adding New Document Types
Extend `document_processor.py` to handle additional formats. The system expects documents to contain course structure information.

### Modifying Search Behavior
Update `search_tools.py` to change search parameters or add new search tools. The AI will automatically use available tools based on the query context.

### Vector Database Management
ChromaDB data persists in `./chroma_db/`. Delete this directory to rebuild from scratch. Use `clear_existing=True` in `add_course_folder()` for programmatic clearing.

### Frontend Customization
The frontend uses vanilla HTML/JS with Marked.js for markdown rendering. Modify `frontend/script.js` for UI changes or additional API integrations.
- Use uv package to run python files