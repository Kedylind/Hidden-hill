# Sprint 2 Tasks - Detailed In-Depth Guide

## 🎯 Overall Goal

**Transform the professor's CLI tool into a web application where users can:**
1. Enter a PubMed paper ID
2. Click "Generate Video"
3. Wait for AI to process the paper and create a video summary
4. Watch the video and share it on social media

---

## 📦 TASK 1: Backend API Server & Database

### 🎯 End Goal
A **FastAPI web server** that accepts requests from the frontend, stores data in a database, and coordinates with the job queue.

### Why This Matters
- The frontend needs somewhere to send requests to
- We need to store metadata about videos (which papers were converted, when, status, etc.)
- Users need to track their video generation status

### Detailed Steps

#### Step 1: Project Structure
```
backend/
├── main.py                 # FastAPI app entry point
├── requirements.txt        # Python dependencies
├── .env                    # Environment variables (API keys, DB URL)
├── app/
│   ├── __init__.py
│   ├── models.py          # Database models (User, Video, Job)
│   ├── schemas.py         # Request/response data structures
│   ├── database.py        # Database connection setup
│   ├── crud.py            # Database queries (Create, Read, Update, Delete)
│   └── routers/
│       ├── videos.py      # Video endpoints
│       └── health.py      # Health check endpoint
└── migrations/            # Database migration files
```

#### Step 2: Dependencies to Install
```
fastapi==0.104.0
uvicorn==0.24.0
sqlalchemy==2.0.0
psycopg2-binary==2.9.0  # PostgreSQL driver
python-dotenv==1.0.0
pydantic==2.0.0
```

#### Step 3: Create Database Models

**What is a model?** A model is a Python class that represents a table in the database.

**Users Table:**
```python
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True)
    created_at = Column(DateTime, default=datetime.utcnow)
```

**Videos Table:**
```python
class Video(Base):
    __tablename__ = "videos"
    
    id = Column(String, primary_key=True)  # Unique video ID
    user_id = Column(Integer, ForeignKey("users.id"))
    pubmed_id = Column(String)              # e.g., "PMC10979640"
    status = Column(String)                 # "pending", "processing", "completed", "error"
    video_url = Column(String, nullable=True)  # URL to final video
    error_message = Column(String, nullable=True)  # Error if generation failed
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

**Jobs Table:**
```python
class Job(Base):
    __tablename__ = "jobs"
    
    id = Column(String, primary_key=True)   # Celery task ID
    video_id = Column(String, ForeignKey("videos.id"))
    status = Column(String)                 # Job status from Celery
    progress = Column(Integer, default=0)   # 0-100%
    celery_task_id = Column(String)         # For tracking Celery task
    created_at = Column(DateTime, default=datetime.utcnow)
```

#### Step 4: Create API Endpoints

**Endpoint 1: POST /api/videos/generate**
- **Purpose:** User submits a PubMed ID to generate a video
- **Input:** `{ "pubmed_id": "PMC10979640" }`
- **What happens:**
  1. Validate the PubMed ID format
  2. Create a new Video record in database (status: "pending")
  3. Create a Job record
  4. Queue the video generation task with Celery (Task 2)
  5. Return: `{ "job_id": "abc123", "status": "queued" }`
- **Output:** Job ID that frontend uses to track progress

**Endpoint 2: GET /api/videos/:job_id**
- **Purpose:** Check the status of a video generation job
- **What happens:**
  1. Look up Job in database
  2. Get current status from Celery
  3. Get video metadata (pubmed_id, created_at, etc.)
  4. Return: `{ "status": "processing", "progress": 50, "video_url": null }`
- **Response updates every 2-3 seconds** so frontend can show progress

**Endpoint 3: GET /api/videos/:job_id/download**
- **Purpose:** Download the generated video file
- **What happens:**
  1. Check that job is completed
  2. Return the video file (video.mp4)

**Endpoint 4: GET /health**
- **Purpose:** Check if server is running
- **Input:** None
- **Output:** `{ "status": "ok" }`

#### Step 5: Database Setup
```bash
# Install PostgreSQL locally
brew install postgresql

# Start PostgreSQL
brew services start postgresql

# Create database
createdb hidden_hill_db

# Set DATABASE_URL environment variable
export DATABASE_URL="postgresql://user:password@localhost:5432/hidden_hill_db"
```

#### Step 6: Run the Server
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
```

Server runs on `http://localhost:8000`  
API docs visible at `http://localhost:8000/docs` (interactive Swagger UI)

### ✅ Success Criteria
- ✅ FastAPI server starts without errors
- ✅ Can connect to PostgreSQL
- ✅ API endpoints return correct JSON
- ✅ Can create Video and Job records in database
- ✅ GET /health returns `{"status": "ok"}`
- ✅ Swagger docs are accessible

### 🚀 By End of Task 1
- Backend server running
- Database with all tables created
- API ready to receive requests (but Job Queue isn't processing them yet)
- Frontend can submit PubMed IDs (will be queued, not processed)

---

## 📦 TASK 2: Job Queue & Video Generation Pipeline

### 🎯 End Goal
A **Celery worker** that:
1. Receives video generation requests from the API
2. Runs the professor's pipeline code
3. Generates actual videos (or mock videos for testing)
4. Updates the database with progress and results

### Why This Matters
Video generation takes **5-10 minutes** per paper. If we ran this synchronously (blocking), the user's browser would hang. Celery runs this in the background so the API responds instantly.

### How It Works (The Flow)

```
User submits PubMed ID
        ↓
API creates Job, calls Celery task
        ↓
Celery queues task in Redis
        ↓
Celery Worker picks up task
        ↓
Worker runs professor's pipeline
        ↓
Pipeline: Fetch paper → Generate script → Create audio → Render video
        ↓
Worker saves video file to storage
        ↓
Worker updates database: status = "completed", video_url = "..."
        ↓
Frontend polls API and sees video is ready
        ↓
User downloads/watches video
```

### Detailed Steps

#### Step 1: Install Celery & Redis
```bash
pip install celery redis
brew install redis
brew services start redis
```

#### Step 2: Create Celery Configuration
```python
# celery_app.py
from celery import Celery

app = Celery(
    'hidden_hill',
    broker='redis://localhost:6379/0',  # Redis message broker
    backend='redis://localhost:6379/1'  # Redis result backend
)

app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
)
```

#### Step 3: Create Celery Tasks
```python
# tasks.py
from celery_app import app
from main import pipeline  # Professor's code

@app.task(bind=True)
def generate_video_task(self, pubmed_id, output_dir):
    """
    Main task: Generate video from PubMed paper
    
    self = task instance (for tracking progress)
    pubmed_id = e.g., "PMC10979640"
    output_dir = where to save files
    """
    
    try:
        # Step 1: Update job status
        self.update_state(state='PROGRESS', meta={'progress': 10})
        
        # Step 2: Fetch paper from PubMed
        paper = fetch_paper(pubmed_id)
        self.update_state(state='PROGRESS', meta={'progress': 25})
        
        # Step 3: Generate script with AI (Google Gemini)
        script = generate_script(paper)
        self.update_state(state='PROGRESS', meta={'progress': 50})
        
        # Step 4: Generate audio from script (text-to-speech)
        audio_file = generate_audio(script)
        self.update_state(state='PROGRESS', meta={'progress': 75})
        
        # Step 5: Generate video from scenes and audio (Runway ML)
        video_file = generate_video(script, audio_file)
        self.update_state(state='PROGRESS', meta={'progress': 90})
        
        # Step 6: Add captions to video
        final_video = add_captions(video_file, script)
        self.update_state(state='PROGRESS', meta={'progress': 100})
        
        return {
            'status': 'completed',
            'video_file': final_video,
            'pubmed_id': pubmed_id
        }
        
    except Exception as e:
        return {
            'status': 'error',
            'error_message': str(e)
        }
```

#### Step 4: Mock Video Generation (For Testing!)
While you're testing, you **don't have** the Gemini or Runway ML API keys yet. Create a mock version:

```python
@app.task(bind=True)
def generate_mock_video_task(self, pubmed_id, output_dir):
    """
    Mock version: Creates a dummy video for testing
    """
    import time
    from moviepy.editor import VideoClip, concatenate_videoclips
    
    try:
        # Simulate slow processing
        for i in range(1, 11):
            self.update_state(state='PROGRESS', meta={'progress': i * 10})
            time.sleep(1)  # Simulate work
        
        # Create a dummy video file
        # (In real version, this would be actual generated video)
        dummy_video = create_dummy_video(f"Paper: {pubmed_id}")
        
        # Save to output directory
        video_path = f"{output_dir}/{pubmed_id}.mp4"
        dummy_video.write_videofile(video_path)
        
        return {
            'status': 'completed',
            'video_file': video_path,
            'pubmed_id': pubmed_id
        }
    except Exception as e:
        return {'status': 'error', 'error_message': str(e)}
```

#### Step 5: Start Celery Worker
```bash
# Terminal 1: Start Redis
redis-server

# Terminal 2: Start Celery worker
celery -A tasks worker --loglevel=info

# Terminal 3: Start FastAPI server
uvicorn main:app --reload
```

#### Step 6: Integrate with API
```python
# In routers/videos.py
from tasks import generate_video_task

@router.post("/api/videos/generate")
async def generate_video(request: VideoGenerateRequest):
    # Task 1 creates Video record in database
    video = create_video_in_db(pubmed_id=request.pubmed_id)
    
    # Queue the Celery task
    celery_task = generate_video_task.delay(
        pubmed_id=request.pubmed_id,
        output_dir=f"outputs/{video.id}"
    )
    
    # Create Job record linked to Celery task
    job = create_job_in_db(
        video_id=video.id,
        celery_task_id=celery_task.id,
        status="queued"
    )
    
    return {"job_id": job.id, "status": "queued"}

@router.get("/api/videos/{job_id}")
async def get_video_status(job_id: str):
    # Look up Celery task status
    task_result = celery_app.AsyncResult(job_id)
    
    return {
        "status": task_result.state,  # "PENDING", "PROGRESS", "SUCCESS", "FAILURE"
        "progress": task_result.info.get('progress', 0),
        "video_url": task_result.info.get('video_file', None)
    }
```

### ✅ Success Criteria
- ✅ Redis running
- ✅ Celery worker starts and listens for tasks
- ✅ Can queue a task from API
- ✅ Task updates progress (10% → 20% → ... → 100%)
- ✅ Mock video generation completes successfully
- ✅ Database Job status updates as task progresses

### 🚀 By End of Task 2
- Video generation pipeline works (mock version)
- Celery worker processes tasks
- Progress updates visible
- Ready for Task 1 to submit requests

---

## 📦 TASK 3: Frontend React UI Components

### 🎯 End Goal
A **beautiful, responsive web interface** where users can:
1. Enter a PubMed ID
2. See generation progress
3. View and download the generated video

### Why This Matters
Users interact with the frontend, not the API. It needs to be:
- Fast and responsive
- Clear and easy to understand
- Mobile-friendly
- Professional-looking

### Architecture
```
Frontend (React)
├── Pages (displayed to users)
│   ├── Home.jsx          - Landing page
│   ├── Input.jsx         - Input form for PubMed ID
│   ├── Status.jsx        - Shows progress during generation
│   ├── Results.jsx       - Displays video + metadata
│   └── History.jsx       - Shows past videos
├── Components (reusable pieces)
│   ├── VideoPlayer.jsx   - Video player with controls
│   ├── PaperMetadata.jsx - Shows paper title, authors, abstract
│   ├── ProgressBar.jsx   - Shows generation progress %
│   ├── Button.jsx        - Reusable button
│   └── LoadingSpinner.jsx - Animated loading indicator
└── API (not yet connected in Task 3)
    └── client.js         - Will be filled in Task 4
```

### Detailed Steps

#### Step 1: Create React Project
```bash
npm create vite@latest hidden-hill-frontend -- --template react
cd hidden-hill-frontend
npm install
npm install axios react-router-dom tailwindcss
npm run dev
```

Project runs on `http://localhost:5173`

#### Step 2: Project Structure
```
src/
├── App.jsx               # Main app component with routing
├── App.css               # Global styles
├── components/
│   ├── VideoPlayer.jsx
│   ├── PaperMetadata.jsx
│   ├── ProgressBar.jsx
│   └── LoadingSpinner.jsx
├── pages/
│   ├── Home.jsx
│   ├── Input.jsx
│   ├── Status.jsx
│   ├── Results.jsx
│   └── History.jsx
├── api/
│   └── client.js         # HTTP client (Task 4)
├── styles/
│   └── tailwind.css      # Tailwind styles
└── main.jsx              # React entry point
```

#### Step 3: Create Home Page
```jsx
// pages/Home.jsx
export default function Home() {
  return (
    <div className="flex flex-col items-center justify-center h-screen bg-gradient-to-b from-blue-500 to-purple-600">
      <h1 className="text-5xl font-bold text-white mb-4">Hidden Hill</h1>
      <p className="text-xl text-white mb-8">Transform academic papers into social media videos</p>
      
      <button className="bg-white text-blue-600 px-8 py-3 rounded-lg font-bold hover:bg-gray-100">
        Get Started
      </button>
      
      <p className="text-white mt-8 max-w-md text-center">
        Paste a PubMed ID and we'll generate an AI-powered video summary of the paper
      </p>
    </div>
  );
}
```

#### Step 4: Create Input Form Page
```jsx
// pages/Input.jsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';

export default function Input() {
  const [pubmedId, setPubmedId] = useState('');
  const [error, setError] = useState('');
  const navigate = useNavigate();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Validate PubMed ID format (e.g., "PMC10979640" or "12345678")
    if (!pubmedId.match(/^(PMC)?\d+$/)) {
      setError('Invalid PubMed ID format. Use PMC12345678 or 12345678');
      return;
    }
    
    // TODO: Task 4 will add API call here
    // For now, store in localStorage for mock data
    localStorage.setItem('currentJobId', 'job-' + Date.now());
    localStorage.setItem('pubmedId', pubmedId);
    
    // Navigate to status page
    navigate('/status');
  };
  
  return (
    <div className="min-h-screen bg-gray-50 flex items-center justify-center p-4">
      <div className="bg-white rounded-lg shadow-lg p-8 max-w-md w-full">
        <h2 className="text-3xl font-bold mb-6">Enter PubMed ID</h2>
        
        <form onSubmit={handleSubmit}>
          <input
            type="text"
            value={pubmedId}
            onChange={(e) => setPubmedId(e.target.value)}
            placeholder="e.g., PMC10979640"
            className="w-full px-4 py-2 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          
          {error && <p className="text-red-500 text-sm mb-4">{error}</p>}
          
          <button
            type="submit"
            className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 font-bold"
          >
            Generate Video
          </button>
        </form>
        
        <p className="text-gray-500 text-sm mt-4">
          Find PubMed IDs at <a href="https://pubmed.ncbi.nlm.nih.gov/" target="_blank" rel="noopener noreferrer" className="text-blue-600 hover:underline">pubmed.ncbi.nlm.nih.gov</a>
        </p>
      </div>
    </div>
  );
}
```

#### Step 5: Create Status Page (with Progress)
```jsx
// pages/Status.jsx
import { useEffect, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import ProgressBar from '../components/ProgressBar';
import LoadingSpinner from '../components/LoadingSpinner';

export default function Status() {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState('processing');
  const navigate = useNavigate();
  
  useEffect(() => {
    // Mock progress simulation
    // Task 4 will replace this with real API polling
    const interval = setInterval(() => {
      setProgress(prev => {
        const next = prev + Math.random() * 15;
        if (next >= 100) {
          clearInterval(interval);
          setStatus('completed');
          // Redirect to results after 2 seconds
          setTimeout(() => navigate('/results'), 2000);
          return 100;
        }
        return next;
      });
    }, 2000);
    
    return () => clearInterval(interval);
  }, [navigate]);
  
  return (
    <div className="min-h-screen bg-gray-50 flex items-center justify-center p-4">
      <div className="bg-white rounded-lg shadow-lg p-8 max-w-md w-full">
        <h2 className="text-3xl font-bold mb-6">Generating Video...</h2>
        
        <LoadingSpinner />
        
        <ProgressBar progress={progress} />
        
        <p className="text-center text-gray-600 mt-4">{Math.round(progress)}% complete</p>
        
        <div className="mt-6 space-y-2 text-sm text-gray-600">
          <div className={progress > 25 ? 'text-green-600' : 'text-gray-400'}>
            ✓ Fetching paper
          </div>
          <div className={progress > 50 ? 'text-green-600' : 'text-gray-400'}>
            ✓ Generating script
          </div>
          <div className={progress > 75 ? 'text-green-600' : 'text-gray-400'}>
            ✓ Creating audio
          </div>
          <div className={progress > 90 ? 'text-green-600' : 'text-gray-400'}>
            ✓ Rendering video
          </div>
        </div>
      </div>
    </div>
  );
}
```

#### Step 6: Create Results Page
```jsx
// pages/Results.jsx
export default function Results() {
  const pubmedId = localStorage.getItem('pubmedId');
  
  return (
    <div className="min-h-screen bg-gray-50 p-8">
      <h1 className="text-4xl font-bold mb-8">Video Ready!</h1>
      
      <div className="grid md:grid-cols-2 gap-8">
        {/* Video Player */}
        <div className="bg-white rounded-lg shadow-lg overflow-hidden">
          <video 
            controls 
            className="w-full h-96 bg-black"
            src="https://via.placeholder.com/640x360/000000/ffffff?text=Generated+Video"
          ></video>
        </div>
        
        {/* Paper Metadata */}
        <div className="bg-white rounded-lg shadow-lg p-6">
          <h2 className="text-2xl font-bold mb-4">Paper Details</h2>
          
          <div className="space-y-4">
            <div>
              <h3 className="font-bold text-gray-600">PubMed ID</h3>
              <p>{pubmedId}</p>
            </div>
            
            <div>
              <h3 className="font-bold text-gray-600">Title</h3>
              <p>Understanding AI in Medical Research</p>
            </div>
            
            <div>
              <h3 className="font-bold text-gray-600">Authors</h3>
              <p>John Smith, Jane Doe, et al.</p>
            </div>
            
            <div>
              <h3 className="font-bold text-gray-600">Abstract</h3>
              <p className="text-sm text-gray-600">
                This paper explores the applications of artificial intelligence in medical research...
              </p>
            </div>
          </div>
          
          <div className="mt-6 space-y-2">
            <button className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700">
              Download Video
            </button>
            <button className="w-full bg-gray-300 text-gray-700 py-2 rounded-lg hover:bg-gray-400">
              Share to Twitter
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

#### Step 7: Add Routing
```jsx
// App.jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import Input from './pages/Input';
import Status from './pages/Status';
import Results from './pages/Results';

function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/input" element={<Input />} />
        <Route path="/status" element={<Status />} />
        <Route path="/results" element={<Results />} />
      </Routes>
    </Router>
  );
}

export default App;
```

### ✅ Success Criteria
- ✅ React app runs on localhost:5173
- ✅ All pages load and display correctly
- ✅ Form validates PubMed ID
- ✅ Progress bar animates
- ✅ Video player displays
- ✅ Responsive on mobile and desktop
- ✅ No errors in console

### 🚀 By End of Task 3
- Beautiful UI complete
- User flow working (navigate through pages)
- Uses mock data (Task 4 will connect to real API)
- Ready to connect to backend

---

## 📦 TASK 4: Frontend API Integration

### 🎯 End Goal
Connect the React frontend to the FastAPI backend so real video generation works end-to-end.

### Detailed Steps

#### Step 1: Create API Client
```javascript
// src/api/client.js
import axios from 'axios';

const API_BASE_URL = 'http://localhost:8000/api';

export const videoAPI = {
  // Submit PubMed ID for video generation
  async generateVideo(pubmedId) {
    const response = await axios.post(`${API_BASE_URL}/videos/generate`, {
      pubmed_id: pubmedId
    });
    return response.data;  // Returns { job_id: "...", status: "queued" }
  },
  
  // Check status of video generation
  async getStatus(jobId) {
    const response = await axios.get(`${API_BASE_URL}/videos/${jobId}`);
    return response.data;  // Returns { status: "processing", progress: 50 }
  },
  
  // Download video file
  async downloadVideo(jobId) {
    const response = await axios.get(`${API_BASE_URL}/videos/${jobId}/download`, {
      responseType: 'blob'
    });
    return response.data;
  }
};
```

#### Step 2: Update Input Page to Call API
```jsx
// pages/Input.jsx
import { videoAPI } from '../api/client';

export default function Input() {
  const [pubmedId, setPubmedId] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');
    
    try {
      // Call backend API
      const response = await videoAPI.generateVideo(pubmedId);
      
      // Store job ID for polling
      localStorage.setItem('jobId', response.job_id);
      
      // Go to status page
      navigate('/status');
    } catch (err) {
      setError(err.response?.data?.detail || 'Failed to generate video');
      setLoading(false);
    }
  };
  
  return (
    // ... (form same as before)
  );
}
```

#### Step 3: Update Status Page to Poll API
```jsx
// pages/Status.jsx
import { useEffect, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { videoAPI } from '../api/client';

export default function Status() {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState('processing');
  const [error, setError] = useState('');
  const navigate = useNavigate();
  const jobId = localStorage.getItem('jobId');
  
  useEffect(() => {
    if (!jobId) {
      navigate('/');
      return;
    }
    
    // Poll backend every 2 seconds
    const interval = setInterval(async () => {
      try {
        const data = await videoAPI.getStatus(jobId);
        
        setStatus(data.status);
        setProgress(data.progress || 0);
        
        // When complete, redirect to results
        if (data.status === 'SUCCESS') {
          setTimeout(() => navigate('/results'), 1000);
        }
        
        // On error, show message
        if (data.status === 'FAILURE') {
          setError(data.error_message || 'Video generation failed');
          clearInterval(interval);
        }
      } catch (err) {
        console.error('Error fetching status:', err);
      }
    }, 2000);
    
    return () => clearInterval(interval);
  }, [jobId, navigate]);
  
  return (
    // ... (same UI as before, but now real data)
  );
}
```

### ✅ Success Criteria
- ✅ Frontend successfully calls backend API
- ✅ Real progress updates from Celery task
- ✅ Video displays when complete
- ✅ No CORS errors
- ✅ Errors handled gracefully

---

## 📦 TASK 5: DevOps & Deployment

### 🎯 End Goal
Set up **Docker**, **GitHub Actions**, and **deployment infrastructure** so:
- Developers can run entire stack with one command
- Code automatically deploys to staging on every push
- Production deployment is one click away

### Detailed Steps

#### Step 1: Docker Setup for Local Development
```dockerfile
# backend/Dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```dockerfile
# frontend/Dockerfile
FROM node:18

WORKDIR /app

COPY package*.json .
RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev"]
```

#### Step 2: Docker Compose for All Services
```yaml
# docker-compose.yml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: hidden_hill_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  # Redis Cache
  redis:
    image: redis:7
    ports:
      - "6379:6379"
  
  # Backend API
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://user:password@postgres:5432/hidden_hill_db
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - postgres
      - redis
  
  # Celery Worker
  celery_worker:
    build: ./backend
    command: celery -A tasks worker --loglevel=info
    environment:
      DATABASE_URL: postgresql://user:password@postgres:5432/hidden_hill_db
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - postgres
      - redis
      - backend
  
  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    depends_on:
      - backend

volumes:
  postgres_data:
```

**Now developers can run:**
```bash
docker-compose up
```
And everything starts: database, cache, backend API, Celery worker, frontend!

#### Step 3: GitHub Actions CI/CD
```yaml
# .github/workflows/deploy.yml
name: Deploy to Staging

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      # Run backend tests
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.11
      
      - name: Install backend dependencies
        run: pip install -r backend/requirements.txt
      
      - name: Run backend tests
        run: pytest backend/tests
      
      # Run frontend tests
      - name: Set up Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install frontend dependencies
        run: npm install
        working-directory: frontend
      
      - name: Run frontend tests
        run: npm test
        working-directory: frontend

  deploy:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      # Deploy backend to Render
      - name: Deploy backend to Render
        env:
          RENDER_API_KEY: ${{ secrets.RENDER_API_KEY }}
        run: |
          curl -X POST https://api.render.com/v1/services/deploy \
            -H "Authorization: Bearer ${{ env.RENDER_API_KEY }}" \
            -d '{"serviceId": "'${{ secrets.RENDER_SERVICE_ID }}'"}'
      
      # Deploy frontend to Vercel
      - name: Deploy frontend to Vercel
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
        run: npm run deploy
        working-directory: frontend
```

#### Step 4: Environment Variables Setup
```bash
# .env (local development)
DATABASE_URL=postgresql://user:password@localhost:5432/hidden_hill_db
REDIS_URL=redis://localhost:6379/0
GEMINI_API_KEY=your_key_here
RUNWAYML_API_KEY=your_key_here
```

In GitHub Settings → Secrets, add these same variables for production.

### ✅ Success Criteria
- ✅ `docker-compose up` starts entire stack
- ✅ GitHub Actions workflow runs on every push
- ✅ Tests pass before deployment
- ✅ Backend deploys to staging URL
- ✅ Frontend deploys to staging URL
- ✅ Environment variables configured

---

## 📦 TASK 6: Testing, Documentation & Integration

### 🎯 End Goal
**Ensure all 5 tasks work together**, with tests proving quality and docs explaining how to use the system.

### Detailed Steps

#### Step 1: Backend Tests (pytest)
```python
# backend/tests/test_api.py
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_health_endpoint():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}

def test_generate_video():
    response = client.post("/api/videos/generate", json={
        "pubmed_id": "PMC10979640"
    })
    assert response.status_code == 200
    assert "job_id" in response.json()
    assert "status" in response.json()

def test_get_status():
    # First generate a video
    gen_response = client.post("/api/videos/generate", json={
        "pubmed_id": "PMC10979640"
    })
    job_id = gen_response.json()["job_id"]
    
    # Then check status
    status_response = client.get(f"/api/videos/{job_id}")
    assert status_response.status_code == 200
    assert "status" in status_response.json()
    assert "progress" in status_response.json()

def test_invalid_pubmed_id():
    response = client.post("/api/videos/generate", json={
        "pubmed_id": "invalid"
    })
    assert response.status_code == 400  # Bad request
```

Run tests:
```bash
pytest backend/tests -v
```

#### Step 2: Frontend Tests (Jest)
```javascript
// frontend/__tests__/Input.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Input from '../src/pages/Input';

test('renders input form', () => {
  render(<Input />);
  expect(screen.getByPlaceholderText(/Enter PubMed ID/i)).toBeInTheDocument();
});

test('validates PubMed ID format', () => {
  render(<Input />);
  const input = screen.getByPlaceholderText(/Enter PubMed ID/i);
  const button = screen.getByRole('button', { name: /Generate/i });
  
  // Enter invalid ID
  fireEvent.change(input, { target: { value: 'invalid' } });
  fireEvent.click(button);
  
  // Should show error
  expect(screen.getByText(/Invalid PubMed ID format/i)).toBeInTheDocument();
});
```

Run tests:
```bash
npm test
```

#### Step 3: End-to-End Test
```python
# backend/tests/test_e2e.py
"""
End-to-end test: Entire user flow
1. User submits PubMed ID via frontend
2. Backend queues video generation
3. Celery worker processes task
4. Frontend polls status
5. Video is ready to download
"""

def test_complete_flow():
    # 1. Generate video
    response = client.post("/api/videos/generate", json={
        "pubmed_id": "PMC10979640"
    })
    job_id = response.json()["job_id"]
    
    # 2. Poll status (simulate frontend polling)
    max_polls = 60  # 60 * 2 seconds = 2 minutes max
    for i in range(max_polls):
        status_response = client.get(f"/api/videos/{job_id}")
        data = status_response.json()
        
        if data["status"] == "SUCCESS":
            assert data["video_url"] is not None
            break
        
        time.sleep(2)  # Wait 2 seconds between polls
    
    # 3. Download video
    download_response = client.get(f"/api/videos/{job_id}/download")
    assert download_response.status_code == 200
    assert len(download_response.content) > 0  # File not empty
```

#### Step 4: API Documentation
FastAPI automatically generates **interactive API docs** at `http://localhost:8000/docs`

Add docstrings:
```python
@app.post("/api/videos/generate")
async def generate_video(request: VideoGenerateRequest):
    """
    Generate a video from a PubMed paper.
    
    - **pubmed_id**: PubMed or PMC ID (e.g., "PMC10979640" or "12345678")
    
    Returns: Job ID to track progress
    
    Example:
        POST /api/videos/generate
        {
          "pubmed_id": "PMC10979640"
        }
        
        Response:
        {
          "job_id": "abc-123-def",
          "status": "queued"
        }
    """
```

#### Step 5: Developer Setup Guide
```markdown
# Developer Setup Guide

## Prerequisites
- Docker & Docker Compose
- Python 3.11
- Node 18+
- Git

## Quick Start (5 minutes)

1. Clone repository:
   ```bash
   git clone https://github.com/Kenza-R/Hidden-hill.git
   cd Hidden-hill
   ```

2. Start everything:
   ```bash
   docker-compose up
   ```

3. Access services:
   - Frontend: http://localhost:5173
   - Backend: http://localhost:8000
   - API Docs: http://localhost:8000/docs
   - Database: localhost:5432

4. Run tests:
   ```bash
   # Backend
   docker-compose exec backend pytest
   
   # Frontend
   cd frontend && npm test
   ```

## Architecture

```
User → Frontend (React) → Backend API (FastAPI) → Database (PostgreSQL)
                            ↓
                       Celery Worker → Redis → Professor's Pipeline
```

## Development Workflow

1. Create feature branch: `git checkout -b feature/my-feature`
2. Make changes
3. Run tests: `pytest` and `npm test`
4. Commit: `git commit -m "feat: my feature"`
5. Push: `git push origin feature/my-feature`
6. Create PR on GitHub

## API Endpoints

See http://localhost:8000/docs for interactive API docs

## Troubleshooting

**Database connection error?**
- Make sure PostgreSQL is running: `docker-compose logs postgres`

**Celery task not processing?**
- Check Celery worker logs: `docker-compose logs celery_worker`

**Frontend can't reach API?**
- Check CORS configuration in backend
- API should be at http://localhost:8000
```

#### Step 6: User Guide
```markdown
# How to Use Hidden Hill

## Step 1: Find a Paper
1. Go to https://pubmed.ncbi.nlm.nih.gov/
2. Search for a paper
3. Copy the PMC ID (e.g., PMC10979640)

## Step 2: Generate Video
1. Visit http://localhost:5173
2. Click "Get Started"
3. Paste the PMC ID
4. Click "Generate Video"

## Step 3: Wait
- The app will show progress (10% → 100%)
- This takes 5-10 minutes
- Don't close the browser

## Step 4: Watch & Share
- Video automatically displays when ready
- Click "Download" to save video
- Click "Share" to post on Twitter

## Tips
- Videos are cached (generating same paper is instant second time)
- Supported formats: PubMed ID, PMC ID, PMID
- Any questions? Contact the team!
```

### ✅ Success Criteria
- ✅ Backend tests: 80%+ code coverage
- ✅ Frontend tests: All components render correctly
- ✅ E2E test: Full user flow works
- ✅ API docs accessible and clear
- ✅ Setup guide works for new developers
- ✅ All 5 tasks integrate without errors

---

## 🎯 THE BIG PICTURE

### What Users See (End Goal)
1. **Visit website** → Clean landing page
2. **Enter PubMed ID** → Form validates input
3. **Click Generate** → API accepts request, queues job
4. **Wait with progress bar** → Celery processes video generation
5. **Watch video** → Video displays with paper info
6. **Download/Share** → Save video or post to Twitter

### What Happens Behind the Scenes
```
Frontend (React App)
    ↓ (HTTP POST /api/videos/generate)
Backend API (FastAPI)
    ↓ (store in database, queue task)
Redis Message Broker
    ↓ (pick up task)
Celery Worker
    ↓ (run professor's pipeline)
    ├─ Fetch paper from PubMed
    ├─ Generate script with Gemini AI
    ├─ Create audio with text-to-speech
    ├─ Render video with Runway ML
    └─ Add captions
    ↓ (save to storage)
Database (PostgreSQL)
    ↓ (store video URL, mark as complete)
Frontend (React App)
    ↓ (polling sees video is done)
User sees video ✅
```

### Timeline
- **Sprint 2 (Nov 11-24)**: All 6 tasks complete, working MVP
- **Sprint 3 (Nov 25-Dec 2)**: Real API keys, polish UI, staging deployment
- **Sprint 4 (Dec 2-11)**: Production deployment, final testing, demo

---

**Total Effort**: ~200 hours for 6 people over 6 weeks  
**Per Person**: ~35 hours (easily doable with 10 hrs/week)  
**Result**: Fully functional web app converting papers to videos

This breakdown transforms a complex system into **6 manageable, parallel tasks** that integrate seamlessly! 🚀
