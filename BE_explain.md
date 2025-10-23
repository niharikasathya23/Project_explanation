# 🧠 Project Portfolio

## 📘 Table of Contents
1. [Virufy](#1-virufy)  
2. [HIRO Lab – Robotics Perception & Big Data Systems](#2-hiro-lab--robotics-perception--big-data-systems)  
3. [Rakuten – Software Engineer Intern](#3-rakuten--software-engineer-intern)  
4. [DRDO (CAIR) – Software Development Intern](#4-drdo-cair--software-development-intern)  
5. [Major Projects](#5-major-projects)  
   - [StockWatch – Real-Time Big Data Pipeline for Stock Sentiment Analysis](#stockwatch--real-time-big-data-pipeline-for-stock-sentiment-analysis)  
   - [Movie Recommendation System](#movie-recommendation-system)  
   - [Automatic Hate Speech Detection Using Ensemble Methods](#automatic-hate-speech-detection-using-ensemble-methods)  
6. [Intro](#6-intro)  
7. [Deep Intro](#7-deep-intro)

---

## 1. Virufy

### S – Situation  
“Virufy was building an AI-driven health diagnostics platform that analyzes cough and breathing sounds to identify early indicators of respiratory conditions such as COVID-19, TB, and asthma.  
The system collects audio samples through a web interface, processes them through ML inference services, and provides real-time diagnostic feedback to clinicians and researchers.  
When I joined, the backend was still at a prototype stage and required a complete redesign to handle large-scale data ingestion, integrate AI inference, and ensure performance, scalability, and data integrity.”

### T – Task  
“I was primarily responsible for building and scaling the backend infrastructure to support these diagnostics and data collection workflows.  
My goals were to design the FastAPI-based architecture, ensure reliable data ingestion and storage for over 100,000 patient records, and automate our CI/CD pipelines for faster and safer model releases.”

### A – Action  
“I designed and implemented RESTful APIs using Python FastAPI and SQLAlchemy ORM, defining clear schemas for patients, diagnostics, and media assets.  
I managed database schema migrations with Alembic, optimized query performance, and added connection pooling to reduce request latency under high concurrency.  
To ensure scalability and uptime, I containerized services with Docker and deployed them on AWS Kubernetes (EKS) with PostgreSQL on RDS.  
I built CI/CD workflows in GitHub Actions to automate testing, database migrations, and rolling deployments.  
Since we were a small, cross-functional team, I also worked closely with our ML engineers to integrate TensorFlow inference endpoints, and with frontend developers to align API contracts using OpenAPI specifications.  
I collaborated with DevOps on AWS resource provisioning and observability — using CloudWatch and structured logging for error tracing.”

### R – Result  
“These efforts reduced backend API latency by about 30%, improved release reliability, and increased deployment frequency by 40%.  
The system scaled to handle 100K+ patient records and supported real-time diagnostic inference in under 300 milliseconds.”

---

### 🧭 System Walkthrough
“Sure. Let me walk you through how our backend system works from end to end.  
When a user logs into the web app, they can record and upload a short cough or breathing sample through their browser.  
The frontend sends that audio and related metadata to our backend API, which is built with FastAPI.  
The request first hits our API layer, where I defined separate routers for patients, diagnostics, and media uploads.  
Input is validated using Pydantic schemas, and authentication is handled through JWT-based OAuth2 tokens.  
Once the data passes validation, the backend generates a presigned S3 URL and returns it to the frontend so the audio file is uploaded directly to AWS S3.  
The backend simultaneously creates a record in PostgreSQL using SQLAlchemy ORM, linking the file’s metadata to a patient and diagnostic record.  
After the upload completes, an asynchronous task is triggered to call our inference microservice, which hosts the machine learning model.  
That service fetches the audio from S3, runs the inference, and returns a diagnosis label — like “likely healthy” or “possible respiratory anomaly” — with a confidence score.  
The backend then stores this result back in PostgreSQL, updating the diagnostic record.  
The frontend polls the /diagnostics/status endpoint or subscribes via WebSocket to show real-time feedback to the user.  
All of this runs inside Docker containers deployed on AWS EKS (Kubernetes), with AWS RDS for our database and CloudWatch for logs and metrics.  
Deployments are automated with GitHub Actions — every push runs tests, builds Docker images, applies Alembic migrations, and rolls out the update using blue-green deployment.  
From a user clicking ‘upload’ to getting a diagnostic result takes under a second and a half, and the backend comfortably scales to handle over 100,000 audio and diagnostic records.  
We continuously monitor performance and uptime through CloudWatch dashboards and structured JSON logs.  
In short, it’s a fully containerized, event-driven FastAPI system that manages secure audio ingestion, ML inference, and data storage — optimized for speed, scalability, and reliability.”

---

### 2️⃣ ‘100K+ records’ scale? What exactly does that represent?
“That figure represents the total volume of structured records we store across patients, diagnostics, and audio metadata.  
Each upload from a user generates multiple linked entries — a patient record, one or more diagnostic entries, and corresponding media records.  
During data collection campaigns, uploads were coming from thousands of users, each with multiple samples.  
So the backend had to efficiently handle over 100K relational records and maintain referential integrity across tables, while still supporting concurrent inference and retrieval operations.”

---

### 3️⃣ What were the main optimizations you applied to reduce latency?
“There were three big ones:  
**Connection Pooling:** I configured SQLAlchemy’s QueuePool to reuse DB connections instead of creating new ones per request.  
**Query Optimization:** I rewrote ORM-heavy nested queries into efficient JOINs, used selectinload for eager loading, and indexed time-series columns.  
**Async I/O:** I used FastAPI’s async routes for I/O-bound operations like file uploads and inference calls.  
Together, these changes reduced average response time by roughly 30% and improved throughput by over 35% during load testing.”

---

### 4️⃣ What was your approach to handling high concurrency?
“We handled concurrency mainly through async endpoints and connection pooling.  
File uploads and inference calls are I/O-bound, so async endpoints prevented blocking.  
Database operations used pooled connections, allowing multiple requests to share a small number of open connections efficiently.  
For background tasks like inference or cleanup, I used FastAPI’s background workers so these operations didn’t block the main request-response cycle.  
This setup comfortably handled hundreds of simultaneous uploads without latency spikes.”

---

### 5️⃣ How did you ensure the system could scale horizontally?
“Since each service was containerized, scaling was handled at the Kubernetes level.  
I defined autoscaling rules in EKS based on CPU and memory utilization — when traffic or inference load increased, new pods spun up automatically.  
The database ran on AWS RDS, which could scale vertically as needed, and I tuned the connection pool to avoid exhaustion when replicas were added.  
The stateless FastAPI layer made horizontal scaling straightforward — new pods could join or leave the load balancer without any data loss or session issues.”

---

### 6️⃣ Did you use any caching strategies?
“At that stage, we relied on in-memory caching for reference data and short-lived computations, but no external cache yet.  
However, I designed the API layer to be cache-ready — endpoints supported ETag and Last-Modified headers for frontend caching.  
My next step would have been to integrate Redis to cache frequent read queries like patient lookups or inference results, reducing DB hits for repeated accesses.”

---

### 7️⃣ How did you optimize database queries for scale?
“The biggest gains came from analyzing slow queries using EXPLAIN plans.  
I removed unnecessary subqueries generated by the ORM and replaced them with explicit JOINs.  
Added composite and partial indexes for time-based lookups.  
Introduced pagination for list endpoints to avoid full table scans.  
These database-level changes made the API response times much more consistent even as data grew past 100K+ records.”

---

### 🔟 What would you do differently to improve scalability even more?
“I’d introduce a Redis caching layer to handle repeated reads and reduce database load.  
For inference workloads, I’d decouple the model service further using a message queue (like Kafka or Redis Streams) to asynchronously process requests.  
I’d also like to add Prometheus + Grafana dashboards to visualize metrics over time and fine-tune auto-scaling rules based on real usage patterns.  
These steps would make the system more resilient to spikes and support higher concurrency.”

---

### 8️⃣ Tell me about a time you took ownership of a major improvement.
“When I realized our deployments were mostly manual, I proposed automating them with GitHub Actions.  
I built a CI/CD workflow that ran linting, tests, Docker builds, and migrations automatically on each push.  
This not only reduced human error but also increased release frequency by about 40%.  
The automation became the standard for all backend services after that.  
It was rewarding to see that small initiative significantly improve our delivery speed and confidence.”

---

### 9️⃣ How did you handle conflicting priorities or technical disagreements?
“We had debates about whether to optimize queries or add caching early.  
I proposed we first gather metrics using pg_stat_statements and CloudWatch before making assumptions.  
Once we saw data showing most latency came from unindexed joins, we agreed to focus on query optimization first.  
I’ve found that using data and metrics, rather than opinions, helps align teams quickly on technical decisions.”

---

### 🔟 What did you learn from this project that you’ll bring to Vivpro?
“Two things stand out:  
First, building scalable backend systems starts with clean design and observability — you can’t optimize what you can’t measure.  
Second, small teams can move fast if they invest early in automation, testing, and CI/CD discipline.  
I’d bring that same mindset to Vivpro — focusing on designing resilient Python backends that are easy to iterate on, scale, and monitor.”

---

### 🧠 Pillar 1 — Async vs Sync (Concurrency in FastAPI)
“So in our backend, concurrency was a big part of what made the system performant.  
FastAPI lets you define endpoints as either synchronous or asynchronous. A synchronous endpoint, the normal def, runs in a blocking thread — so if it’s waiting on I/O like a database query, file upload, or a network call, it blocks that thread until it’s done.  
Asynchronous endpoints, on the other hand, run on Python’s asyncio event loop — they don’t block. While one request is waiting for an S3 upload or a response from the inference API, that same thread can serve another request in parallel.  
Because most of our workload was I/O-bound — uploading files, hitting the ML service — switching to async endpoints allowed us to handle hundreds of concurrent uploads without spinning up more pods or threads.  
Under the hood, FastAPI runs on Uvicorn and Starlette, which leverage cooperative multitasking — it’s not OS threads, it’s all managed inside the event loop.  
Async gave us near real-time performance — response times under 300 milliseconds — while reducing CPU overhead by about 30 percent under load.”

---

### 🧱 Pillar 2 — Database Design & Scaling
“Our data model was fully relational because consistency and traceability were key.  
We had three main entities: Patients, Diagnostics, and MediaAssets.  
Each patient could have multiple diagnostic entries, and each diagnostic referenced a media record — essentially the uploaded audio file stored on S3.  
I designed the schema using SQLAlchemy ORM with PostgreSQL on AWS RDS, adding composite indexes on frequently filtered columns like patient_id and created_at to speed up queries.  
For concurrency, I configured SQLAlchemy’s QueuePool so each pod reused a small set of open connections — around 5 per pod — instead of opening new ones per request.  
To scale further, PostgreSQL handled 100,000-plus records easily, and the system was structured to add read replicas or Redis caching if read-heavy traffic increased.  
This design gave us the consistency of relational transactions, the flexibility of JSONB fields where needed, and the scalability to grow without redesigning the schema.”

---

### ⚙️ Pillar 3 — CI/CD Pipeline & Deployment Automation
“For CI/CD, we wanted a completely hands-free deployment process, so I set up the entire pipeline in GitHub Actions.  
Every time code was pushed, the pipeline automatically ran linting, unit and integration tests, built a Docker image, ran Alembic migrations against a test database, and then pushed the image to AWS ECR.  
From there, Kubernetes on AWS EKS pulled the image and rolled out the new deployment using a rolling update strategy — so there was zero downtime.  
Each step had health checks — if any smoke test failed or latency spiked, EKS rolled back to the previous ReplicaSet automatically.  
I also made sure migrations were safe — we used expand-contract migrations to avoid locking live tables.  
This automation reduced deployment errors by about 35 percent and improved release speed by 40 percent.  
The nice part is that the system could go from commit to production-ready deployment in under five minutes, fully verified.”

---

### 🤖 Pillar 4 — Async Inference Integration (ML Communication)
“The inference integration was one of the more interesting parts of the system.  
Once a user uploaded their audio, we didn’t process it directly in the same request. Instead, I built an asynchronous background task to call our inference service.  
That service was a separate containerized microservice hosting the ML model — it would fetch the audio from S3, run the TensorFlow model, and return a JSON response with the label and confidence score.  
On the backend side, I used FastAPI’s background task mechanism and the httpx AsyncClient, so inference calls didn’t block other uploads.  
The results were then written back to the diagnostic record in PostgreSQL, and the frontend could poll or subscribe via WebSocket to get the update in real time.  
This async architecture made the backend responsive even during large data collection drives — and if inference traffic ever exceeded capacity, we could easily switch to a message queue system using Redis Streams or Celery for full decoupling.”

---

### 🧩 Pillar 5 — Data Consistency & Reliability
“Data consistency between PostgreSQL and S3 was critical, because we had two systems — structured data in the DB and audio blobs in cloud storage.  
I made the upload process fully atomic. When a user started an upload, the backend first created a pending entry in PostgreSQL and generated a presigned URL for S3.  
Only after the upload completed and was confirmed did we commit that DB transaction and mark the record active.  
If anything failed mid-process, the DB record stayed in a pending state, and a cleanup coroutine deleted the orphaned S3 file.  
We also had a weekly integrity job that cross-checked DB records and S3 keys to detect mismatches.  
This approach ensured zero orphaned records, even when network failures or partial uploads occurred, and maintained complete consistency across systems.”

---

### ⚡ Pillar 6 — Performance Optimization & Monitoring
“Once the system was stable, I focused on performance tuning and observability.  
I started by profiling API latency with FastAPI middleware and used PostgreSQL’s pg_stat_statements and EXPLAIN ANALYZE to pinpoint slow queries.  
Most bottlenecks came from heavy joins and small connection pool sizes.  
I optimized those queries, added indexes, and increased the pool size — and that alone reduced p95 latency by about 30 percent.  
We verified improvements through load testing using Locust, simulating up to 500 concurrent users doing uploads and inference calls.  
For monitoring, I added structured JSON logging and CloudWatch dashboards tracking request latency, error rate, and DB connection usage.  
CloudWatch alarms alerted us if latency exceeded 500ms or error rates spiked.  
That combination of proactive monitoring and data-driven optimization kept the system fast and reliable even as we scaled.”

---

### ☁️ Pillar 7 — AWS / EKS Infrastructure
“We deployed everything on AWS using Kubernetes — specifically EKS.  
Each service — the FastAPI backend, the inference microservice, and the database — ran as separate components.  
The backend and inference were containerized with Docker, stored on AWS ECR, and orchestrated by EKS.  
We used an Application Load Balancer integrated with Kubernetes Ingress for routing and SSL termination. The ALB handled HTTPS termination via ACM certificates, while internal pod traffic stayed over HTTP inside the cluster.  
Our database ran on AWS RDS (PostgreSQL), media files went to S3, and we monitored everything with CloudWatch.  
Horizontal scaling was handled automatically through Kubernetes HPA rules — if CPU went above 60% or memory above 70%, new pods were added.  
This setup gave us elastic scaling, secure networking via private subnets, and a completely self-healing deployment environment.  
It allowed us to handle traffic spikes without downtime and made the infrastructure almost maintenance-free.”

---

### 🔒 Pillar 8 — Security, Reliability & Ownership
“Security and reliability were baked into every layer.  
On the application side, all endpoints were protected with OAuth2 and JWT-based authentication.  
Access was role-based — only clinicians or authorized users could access diagnostic results.  
On the infrastructure side, we enforced HTTPS across all endpoints, encrypted data at rest with AWS KMS, and stored secrets in AWS Secrets Manager. Nothing sensitive was ever hardcoded or checked into Git.  
I also implemented centralized exception handling — any unhandled exception would be logged with a correlation ID so it could be traced across systems.  
CloudWatch alerts notified us of any spikes in error rate or latency.  
One of the biggest lessons for me was around reliability during migrations — I once had a migration lock a table in production. I rolled back using Alembic’s downgrade script, then redesigned the migration to run in phases and only during maintenance windows.  
That taught me the importance of testing migrations in staging and automating rollbacks. It’s the same mindset I bring to all production engineering work — design for failure, monitor everything, and automate recovery.”


---

## 2. HIRO Lab – Robotics Perception & Big Data Systems

### 🤖 Overview
At the **Human Interaction & Robotics (HIRO) Lab, CU Boulder**, I developed scalable data pipelines and visualization systems for robotics perception research.

### 🧩 Key Contributions
- Built a **distributed data ingestion and processing pipeline** using **Kafka + Spark Streaming**.  
- Processed and visualized **1M+ sensor frames** (stereo, LiDAR, telemetry) in near real-time.  
- Developed a **web-based visualization portal** (Flask + WebSockets) for experiment replay and annotation.  
- Integrated robust data storage and query layers for cross-experiment analysis.  
- Reduced experiment processing time by **40%**, enabling faster perception model validation.

### ⚙️ Tech Stack
`Python` · `Flask` · `Apache Kafka` · `Apache Spark` · `AWS EC2` · `WebSockets` · `OpenCV` · `ROS`

---

## 3. Rakuten – Software Engineer Intern

### 🛒 Overview
At **Rakuten**, I optimized and refactored backend modules for the company’s **B2B Order Management Portal**, improving scalability and query efficiency.

### 🧩 Key Contributions
- Implemented **server-side pagination and dynamic filtering** using **Spring Boot + JPA**.  
- Optimized query performance with **indexing** and **entity graph fetch strategies**.  
- Created a **change-tracking microservice** that stored CRUD history for business users.  
- Integrated with a React frontend for interactive order audit timelines.  
- Reduced average API response time by **60%**.

### ⚙️ Tech Stack
`Java` · `Spring Boot` · `JPA` · `PostgreSQL` · `React` · `Bootstrap`

---

## 4. DRDO (CAIR) – Software Development Intern

### 🛰️ Overview
At **Defence Research & Development Organisation (CAIR)**, I built a secure file management and analytics web platform for researchers and administrators.

### 🧩 Key Contributions
- Developed **RESTful APIs** with **Flask** and **PostgreSQL** supporting JWT-based authentication.  
- Implemented **AES encryption** for file storage and **SSL/TLS** for secure communication.  
- Designed a **React-based dashboard** for data upload, search, and role-based access control.  
- Ensured compliance with secure data transmission and storage standards.

### ⚙️ Tech Stack
`Python` · `Flask` · `React` · `PostgreSQL` · `JWT` · `AES` · `SSL/TLS`

---

## 5. Major Projects

### 📊 StockWatch – Real-Time Big Data Pipeline for Stock Sentiment Analysis
- Built an **end-to-end real-time data pipeline** that collects, processes, and visualizes stock sentiment from social media and APIs.  
- Used **PySpark** for distributed processing and **Kafka Streams** for real-time event handling.  
- Integrated sentiment scoring models (BERT + VADER) to detect market trends with 85% accuracy.  
- Designed dashboards with **Plotly Dash** and **AWS RDS** for live analytics.

**Tech Stack:** `Python` · `Kafka` · `PySpark` · `AWS RDS` · `Plotly Dash` · `PostgreSQL`

---

### 🎬 Movie Recommendation System
- Implemented a **hybrid recommendation engine** combining **content-based filtering** and **collaborative filtering**.  
- Utilized **TF-IDF vectorization** for movie metadata and **cosine similarity** for recommendations.  
- Built an interactive Flask web interface with live search and filtering.

**Tech Stack:** `Python` · `Flask` · `scikit-learn` · `Pandas` · `NumPy`

---

### 🧠 Automatic Hate Speech Detection Using Ensemble Methods
- Designed an **ensemble ML model** combining **SVM**, **Logistic Regression**, and **Random Forests**.  
- Preprocessed large Twitter datasets using **NLTK**, **TF-IDF**, and **word embeddings**.  
- Achieved **92% classification accuracy**, reducing false positives by 18%.  
- Deployed model with a Flask API for text moderation.

**Tech Stack:** `Python` · `scikit-learn` · `Flask` · `NLTK` · `Pandas`

---

## 6. Intro

I am a **Software Engineer and Researcher** passionate about building intelligent, distributed systems at the intersection of **AI, data, and cloud infrastructure**.  
My experience spans robotics, backend engineering, DevOps, and machine learning — building scalable platforms for both research and production.

---

## 7. Deep Intro

My work combines:
- **Systems Thinking:** Designing robust, fault-tolerant architectures for data and AI pipelines.  
- **Applied ML:** Bridging classical and modern ML for real-world inference at scale.  
- **End-to-End Engineering:** From REST APIs to cloud deployment and observability.  

Across projects at **Virufy**, **HIRO Lab**, **Rakuten**, and **DRDO**, I’ve developed a strong foundation in designing production-ready AI systems,  
emphasizing **performance, reliability, and real-time insights**.

---
