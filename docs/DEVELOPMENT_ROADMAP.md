# Hidden Hill - Development Roadmap

## 📱 App Overview

**Project:** Hidden Hill  
**Goal:** Create a web application that generates social media videos from academic papers  
**Core Feature:** Input a PubMed/PMC paper ID → AI generates a summarized video with script, audio, and captions

**Professor's Starter Code Provides:**
- CLI pipeline for video generation (Python backend)
- PubMed paper fetching
- AI script generation (using Google Gemini)
- Audio generation (text-to-speech)
- Video generation (using Runway ML)
- Caption generation

---

## 🎯 What We Need to Build (MVP Scope)

### Phase 1: Core Backend (Sprint 2)
- [ ] **API Server Setup**
  - [ ] Create REST API wrapper around professor's pipeline
  - [ ] Endpoints: POST /generate-video, GET /video/:id, GET /status/:id
  - [ ] Framework: Django or FastAPI (TBD)
  - [ ] Authentication: Basic auth or API keys

- [ ] **Database Setup**
  - [ ] Database: PostgreSQL
  - [ ] Tables: users, videos, jobs, papers, generation_logs
  - [ ] Store video generation history and metadata

- [ ] **Job Queue System**
  - [ ] Video generation is async (takes time)
  - [ ] Use Celery + Redis or similar for background tasks
  - [ ] Track generation progress (pending → processing → complete → error)

- [ ] **Error Handling & Logging**
  - [ ] Graceful error messages for failed generations
  - [ ] Log all API calls and generation attempts
  - [ ] Retry logic for failed jobs

### Phase 2: Web Frontend (Sprint 2-3)
- [ ] **Landing Page**
  - [ ] Explain what the app does
  - [ ] Simple, clear UI
  - [ ] Link to start generating

- [ ] **Input Form**
  - [ ] Single input field for PubMed ID
  - [ ] Input validation (PubMed ID format check)
  - [ ] Submit button
  - [ ] Loading state during generation

- [ ] **Generation Status Page**
  - [ ] Show progress: "Fetching paper... → Generating script... → Creating audio... → Rendering video..."
  - [ ] Estimated time remaining
  - [ ] Cancel button (optional)

- [ ] **Video Output Page**
  - [ ] Display generated video player
  - [ ] Show paper metadata (title, authors, abstract)
  - [ ] Download video button
  - [ ] Share to social media buttons (optional)
  - [ ] Link to original paper on PubMed

- [ ] **User Dashboard (Optional for MVP)**
  - [ ] View generation history
  - [ ] Re-generate videos
  - [ ] Delete old videos

### Phase 3: Deployment & Polish (Sprint 3-4)
- [ ] **Deployment**
  - [ ] Deploy backend to Render / Fly.io / Railway
  - [ ] Deploy frontend to Vercel / Netlify
  - [ ] Set up environment variables (API keys)
  - [ ] Configure database backups

- [ ] **Third-Party Integrations**
  - [ ] Google Gemini API key management
  - [ ] Runway ML API key management
  - [ ] PubMed API integration

- [ ] **Performance & Optimization**
  - [ ] Cache generated videos
  - [ ] Optimize video file sizes
  - [ ] CDN for video distribution (optional)

- [ ] **Testing**
  - [ ] Unit tests for API endpoints
  - [ ] Integration tests for pipeline
  - [ ] End-to-end tests (from PubMed ID to video)
  - [ ] Load testing for concurrent requests

- [ ] **Documentation**
  - [ ] API documentation (Swagger/OpenAPI)
  - [ ] Setup guide for developers
  - [ ] User guide (how to use the app)
  - [ ] Deployment runbook

---

## 📋 Detailed Task Breakdown

### Backend Tasks

#### 1. Project Setup
- [ ] Choose framework: Django or FastAPI?
- [ ] Set up project structure
- [ ] Install dependencies (requirements.txt or Poetry)
- [ ] Configure environment variables

#### 2. Database & Models
- [ ] Create database schema
- [ ] Design ORM models (SQLAlchemy for FastAPI or Django ORM)
- [ ] Create migrations

#### 3. API Endpoints
```
POST /api/videos/generate
  Input: { "pubmed_id": "PMC10979640" }
  Output: { "job_id": "uuid", "status": "queued" }

GET /api/videos/:job_id
  Output: { "status": "processing", "progress": 50%, "video_url": "..." }

GET /api/videos/:job_id/status
  Output: { "status": "completed", "message": "Video ready" }

GET /api/videos/:job_id/download
  Output: Download video file
```

#### 4. Job Queue
- [ ] Set up Redis
- [ ] Set up Celery or similar
- [ ] Create task: generate_video(pubmed_id)
- [ ] Track job status in database

#### 5. Integration with Professor's Code
- [ ] Wrap professor's CLI in Python functions
- [ ] Call from job queue tasks
- [ ] Store outputs in proper locations
- [ ] Return video paths to API

#### 6. Error Handling
- [ ] Try-catch for all external API calls (PubMed, Gemini, Runway)
- [ ] Meaningful error messages
- [ ] Logging for debugging

### Frontend Tasks

#### 1. Setup
- [ ] Choose framework: React, Vue, or Svelte?
- [ ] Set up project with Vite or Create React App
- [ ] Install dependencies

#### 2. Pages & Components
- [ ] Homepage/Landing
- [ ] Input form component
- [ ] Status polling component
- [ ] Video player component
- [ ] Paper metadata display

#### 3. API Integration
- [ ] Create API client (fetch or axios)
- [ ] POST to /api/videos/generate
- [ ] Poll GET /api/videos/:job_id every 2-5 seconds
- [ ] Display status updates to user

#### 4. Styling
- [ ] Choose CSS framework (Tailwind, Bootstrap, etc.)
- [ ] Design responsive layout
- [ ] Mobile-friendly UI

#### 5. State Management
- [ ] Track current job ID
- [ ] Track generation status
- [ ] Handle error states
- [ ] Manage user input

### DevOps & Deployment

#### 1. Backend Deployment
- [ ] Choose platform (Render, Fly.io, Railway, Heroku)
- [ ] Set up CI/CD pipeline (GitHub Actions)
- [ ] Configure production database
- [ ] Set up environment secrets (API keys)
- [ ] Configure logging and monitoring

#### 2. Frontend Deployment
- [ ] Choose platform (Vercel, Netlify)
- [ ] Set up automatic builds on git push
- [ ] Configure production API URL
- [ ] Set up CDN for assets

#### 3. Infrastructure
- [ ] Set up PostgreSQL database
- [ ] Set up Redis cache
- [ ] Configure backups
- [ ] Set up monitoring/alerting

---

## 🔄 Development Workflow

### Sprint 2 Goals (Nov 11 - Nov 18)
- **Goal:** Get a working MVP with basic API + frontend
- **Must Have:**
  - API endpoint that accepts PubMed ID
  - Integration with professor's video generation code
  - Simple frontend form to input PubMed ID
  - Basic status display during generation

### Sprint 3 Goals (Nov 18 - Dec 2)
- **Goal:** Polish MVP, add user experience improvements
- **Must Have:**
  - Video player on results page
  - Better error handling
  - Paper metadata display
  - Deployed to staging

### Sprint 4 Goals (Dec 2 - Dec 11)
- **Goal:** Final polish, deployment to production
- **Must Have:**
  - Production deployment
  - All tests passing
  - Documentation complete
  - Ready for demo

---

## 🛠️ Technology Stack (Recommended)

### Backend
- **Framework:** FastAPI (faster to build, modern) or Django (more batteries included)
- **Database:** PostgreSQL
- **Job Queue:** Celery + Redis
- **API Documentation:** FastAPI auto-docs or Swagger

### Frontend
- **Framework:** React (most common) or Vue (simpler)
- **Build Tool:** Vite (fast)
- **CSS:** Tailwind CSS (utility-first, fast to style)
- **HTTP Client:** Axios or Fetch API

### Infrastructure
- **Backend Hosting:** Render or Fly.io
- **Frontend Hosting:** Vercel or Netlify
- **Database:** Managed PostgreSQL (AWS RDS, Render, Railway)
- **Cache:** Redis (for job queue)
- **CI/CD:** GitHub Actions

---

## ⚠️ Dependencies & External APIs

**Required API Keys:**
1. **Google Gemini API** - For script generation
2. **Runway ML API** - For video generation
3. **PubMed API** - For fetching papers (free)

**Python Libraries (from professor's code):**
- `click` - CLI framework
- `runwayml` - Runway ML SDK
- `google-generativeai` (assumed) - Gemini API

**Infrastructure:**
- Redis - for job queue
- PostgreSQL - for data storage

---

## 📊 Success Metrics

By end of Sprint 4, the app should:
- ✅ Accept any valid PubMed ID
- ✅ Generate a video within 5-10 minutes
- ✅ Display video to user with paper metadata
- ✅ Handle errors gracefully
- ✅ Be deployed and accessible online
- ✅ Be responsive on mobile and desktop
- ✅ Have good code quality (tests, docs)

---

## 🚀 Quick Start for Team

1. **Backend Developer:** Review professor's code structure, start building FastAPI wrapper
2. **Frontend Developer:** Create React app with form + video display
3. **DevOps:** Set up database, Redis, staging environment
4. **Full Team:** Integrate API + frontend by end of Sprint 2

---

**Document Created:** November 10, 2025  
**Next Step:** Create detailed user stories for each task above
