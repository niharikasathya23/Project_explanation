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

🧩 **S – Situation**  
“At the Human Interaction and Robotics Lab at CU Boulder, we were running large-scale robotics perception experiments using stereo cameras and 3D sensors.  
Each experiment produced millions of unstructured frames — depth maps, disparity data, and telemetry from multiple sources.  
The problem was that the lab didn’t have an efficient way to process or visualize that data. Researchers were manually opening logs and raw image files to analyze experiments, which was slow and error-prone.  
We needed a proper backend system that could process, store, and visualize robotics data in real time, so researchers could validate models and debug faster.”

---

🎯 **T – Task**  
“My role was to design and build two core components:  
First, a web-based visualization platform that allowed researchers to view, replay, and analyze perception data directly from the browser.  
And second, an optimized distributed data pipeline using Apache Spark and Kafka on AWS that could handle both real-time streaming and batch workloads at scale.  
The key requirements were scalability, low latency, and data consistency between multiple sensors and experiments.”

---

⚙️ **A – Action**

### 🧠 1. Web Application for Visualization  
“I started by developing internal web applications using React for the frontend and Flask with Python for the backend.  
The Flask API served experiment data — things like camera frames, disparity maps, and telemetry — which the React dashboard rendered dynamically in 3D.  
I built reusable React components that supported real-time polling, filtering, and playback of experiment data.  
I also implemented custom visualization hooks to overlay predicted and ground-truth disparity maps side-by-side, which helped researchers immediately see where their models failed.  
To make everything interoperable, I designed consistent data contracts between the frontend, the ETL pipeline, and our PostgreSQL database.  
This way, the visualization tool could fetch results directly from the distributed backend without manual intervention.”

---

### ⚙️ 2. Distributed Data Pipeline Optimization  
“Next, I reworked our data ingestion and transformation pipeline using Apache Kafka and Spark.  
Kafka handled real-time streaming of stereo camera and sensor data — for example, publishing messages to topics like `camera_frames` or `telemetry_data`.  
Spark Structured Streaming consumed those topics and performed parallel aggregation, deduplication, and transformations on AWS EC2 clusters.  
I tuned Spark’s partitioning, shuffle parameters, and executor configurations to improve performance under load.  
We stored the processed outputs in AWS S3 for binary data and PostgreSQL for structured experiment metadata.  
To guarantee reliability, I implemented checkpointing and exactly-once semantics, so even during job restarts, no frames were lost or duplicated.  
For observability, I set up CloudWatch and Prometheus exporters to track pipeline throughput, latency, and error rates in real time.”

---

🚀 **R – Result**  
“As a result, the optimized data pipeline achieved about a 35 percent improvement in throughput, sustaining stable ingestion and processing of over one million unstructured frames per experiment.  
The web-based visualization tool improved analysis and debugging speed by around 25 percent, since researchers could now visualize and replay experiments instantly instead of dealing with raw logs.  
Overall, the system became our lab’s primary backend for robotics perception research — supporting multiple researchers at once and reducing experiment turnaround time from several hours to just minutes.”

---

## ⚙️ 1️⃣ What is Apache Kafka (in simple and technical terms)

**Simple:**  
Kafka is a real-time data streaming platform.  
It lets different parts of a system send and receive data reliably — like a high-speed message queue.

**Technical:**  
It’s a distributed log-based message broker that:
- Stores streams of records in topics, divided into partitions.  
- Allows producers to publish messages and consumers to read them in order.  
- Guarantees durability, ordering, and scalability through partitioning and replication.

---

✅ **How I Used Kafka**  
“At the HIRO Lab, our robots produced continuous streams of stereo camera frames and sensor readings.  
We needed to process that data in real time, but direct ingestion would overwhelm our backend.  
So, I introduced Kafka as the data backbone — a middle layer that decoupled data producers (sensors) from consumers (Spark jobs and visualization APIs).  
Each sensor published data to a topic like `camera_frames` or `telemetry_data`.  
Kafka stored those messages durably and allowed multiple consumers — like Spark Streaming and logging tools — to process them independently.”

---

✅ **Why I Used Kafka**
- **Decoupling:** The robots (producers) and data processors (consumers) could work independently — if Spark was slow, data wouldn’t be lost.  
- **Scalability:** Kafka partitions allowed us to process multiple experiments in parallel.  
- **Fault Tolerance:** Kafka replicated data across brokers, so even if one node failed, messages persisted.  
- **Replayability:** We could replay streams to re-run failed jobs or debug experiments.

**In short:**  
“Kafka gave me a reliable, scalable way to move high-volume robotics data through the system in real time.”

---

## ⚡ 2️⃣ What is Apache Spark

**Simple:**  
Spark is a distributed computing framework that processes large datasets in parallel.  
It’s like a turbocharged Python loop that runs across many machines at once.

**Technical:**  
It uses **Resilient Distributed Datasets (RDDs)** and **DAG execution** to efficiently perform:
- Batch and stream processing  
- SQL queries  
- Machine learning, graph analytics, etc.  
It runs computations **in memory** (vs Hadoop’s on-disk MapReduce), so it’s much faster for iterative workloads.

---

✅ **How I Used Spark**  
“I used Apache Spark Structured Streaming to consume data directly from Kafka topics.  
Each incoming message represented a frame or sensor record.  
Spark read those in micro-batches, cleaned and aggregated them — for example, grouping frames by experiment ID, deduplicating out-of-order messages, and generating summaries like average disparity per second.  
Processed outputs were then written back to AWS S3 in Parquet format and indexed in PostgreSQL, so the visualization dashboard could retrieve them via Flask APIs.”

---

✅ **Why I Used Spark**
- **Parallel Processing:** We had to process over a million frames — Spark distributed the work across multiple EC2 nodes.  
- **Fault Tolerance:** Spark checkpoints and recomputes lost partitions automatically.  
- **Scalability:** I could scale up executors as data grew without changing any code.  
- **Integration with Kafka:** Spark Structured Streaming reads directly from Kafka, which made the architecture clean and event-driven.

**In short:**  
“Spark transformed Kafka’s raw data streams into structured, cleaned, and aggregated outputs — ready for visualization and analysis.”
---

## 3. Rakuten – Software Engineer Intern


“At Rakuten, I worked on the B2B order management portal that helped partner businesses place and track bulk orders.  
I built features like a **‘Save as Draft’** and **‘Order Template’** for faster repeat ordering, and added advanced filters and sorting to make order tracking smoother.  
I also fixed data sync delays, missing product details, and several UI bugs in AngularJS.  
Alongside that, I standardized API responses, added field validations, and helped integrate CI/CD pipelines, which improved deployment speed by about **20%** and reduced post-release defects by **20%**.”

---

### 🧩 1. “Save as Draft” Feature
**Problem:**  
Merchants placing large orders had to complete them in one go — if they navigated away, data was lost.  

**Solution:**  
- Introduced a new **DRAFT** state in the order lifecycle to store partially completed orders.  
- Designed a backend REST endpoint `POST /orders/saveDraft` using **Spring Boot**, which persisted incomplete orders in **MySQL** with a status ENUM (`DRAFT`, `SUBMITTED`, `APPROVED`, `CANCELLED`).  
- Added service-layer validation to ensure only draft orders could be edited or submitted.  
- On the frontend (**AngularJS**), implemented local form caching and triggered the API via **Axios** when users clicked “Save Draft”.  
- Displayed a real-time toast confirmation and updated the UI to reflect the draft status.

---

### 🧱 2. “Order Template” Feature
**Problem:**  
Many clients placed recurring bulk orders (same supplier, SKU, quantity) every week — they had to re-enter data each time.  

**Solution:**  
- Created a reusable “template” model linked to the orders table.  
- Added backend endpoint `POST /orders/{id}/clone` that duplicated order metadata and items but reset status to **DRAFT**.  
- Reused the existing order creation service to ensure business rules stayed consistent.  
- Added frontend support in AngularJS for “Save as Template” and “Use Template” actions.  
- **Tech Detail:** Used Spring’s `BeanUtils.copyProperties()` for deep copy, with selective field reset logic (timestamp, status, audit fields).  

**Impact:**  
Saved merchants several minutes per large order and reduced backend input validation errors.

---

### ⚙️ 3. Advanced Order Filtering and Sorting
**Problem:**  
Filtering orders by supplier, category, or date range caused slow queries and sometimes timeouts.  

**Solution:**  
- Re-engineered the filtering mechanism in **Spring Boot (Spring Data JPA)** to perform all filtering and pagination on the server side, reducing frontend load and network payload.  
- Designed a **FilterCriteria DTO** to dynamically construct JPQL queries based on active filters instead of generating multiple hard-coded queries.  
- **Optimized SQL performance by:**  
  - Adding **composite indexes** on frequently queried columns like `supplier_id`, `status`, and `created_at`.  
  - Replacing nested joins with optimized `JOIN FETCH` and selective column retrieval using **JPA projections**.  
  - Using `LIMIT/OFFSET` pagination for incremental data loading instead of fetching full result sets.  
  - Implementing query caching for recently accessed filter combinations to speed up repeated lookups.  
- Added **request throttling and debouncing** on the AngularJS frontend to prevent redundant calls during user input.

---

### 🕓 4. Order History and Audit Log
**Problem:**  
No visibility into who changed what in orders.  

**Solution:**  
Implemented a comprehensive **order audit trail** to automatically record every significant CRUD (Create, Read, Update, Delete) event associated with an order.  

**Backend Implementation (Spring Boot + JPA):**  
- Designed a new table `order_audit_log` with schema:  
id | order_id | action | user_id | old_value | new_value | timestamp

markdown
Copy code
- `action` captured operations like `CREATED`, `UPDATED`, `STATUS_CHANGED`, `APPROVED`, `CANCELLED`.  
- `old_value` and `new_value` stored serialized JSON snapshots for key fields that changed.  
- `timestamp` recorded when the change occurred.  
- Foreign keys linked to both the **orders** and **users** tables for traceability.  
- Leveraged **Spring JPA’s** `@EntityListeners` and lifecycle annotations (`@PrePersist`, `@PreUpdate`, `@PreRemove`) to automatically log changes whenever an entity was modified.  
- Created a utility component **AuditLogger** that compared entity state before and after persistence and saved only the changed fields into `order_audit_log`.  
- Exposed a REST endpoint:  
GET /orders/{orderId}/history

markdown
Copy code
Returns a chronological list of all changes, grouped by user and action, for easy frontend consumption.

---

### 🧩 5. Key Bug Fixes and Backend Enhancements
**Data Sync Delay:**  
- Fixed caching logic that delayed new orders appearing in dashboards.  
- Used `@CacheEvict` on successful order creation and event-driven updates to invalidate cache immediately.  

**Missing Product Details:**  
- Switched from lazy-loading to explicit **fetch joins** in JPA for `order_items → product` relationships, preventing partial payloads.  

**UI & Validation Fixes:**  
- Added **AngularJS form validation** for required fields like SKU, quantity, and supplier to prevent incomplete API calls.  
- Fixed broken date filters, dropdowns, and layout alignment issues.  

**API Error Standardization:**  
- Created a global `@ControllerAdvice` in **Spring Boot** that returned structured JSON errors:  
```json
{
  "code": "ORDER_NOT_FOUND",
  "message": "No order exists for given ID",
  "details": "order_id=12345"
}
```
Unified error handling across services and simplified frontend exception mapping.

### 6. CI/CD and Testing
Wrote JUnit unit and integration tests for order creation, template cloning, and filtering services — achieved ~85% coverage for core modules.

Created Postman regression suites to automatically validate top APIs after each build.

Integrated both into Jenkins pipelines, which ran on every commit to dev or release branches.

Added build stages:
compile → unit tests → integration tests → deploy → regression tests → approval.

Collaborated via JIRA with QA and design teams to track tickets and release notes.



---

## 4. DRDO (CAIR) – Software Development Intern

During my internship at **DRDO's Centre for Artificial Intelligence and Robotics (CAIR)**, I was responsible for developing a **Secure Data-Sharing Platform** for internal use. This platform aimed to facilitate **secure communication and data sharing** between research teams while adhering to strict security protocols. The focus was on building a web-based system with **role-based access control, end-to-end encryption, and comprehensive logging and monitoring.**

---

## **1. Objective: Secure Data-Sharing Platform**  
### **What Was the Problem?**  
In research institutions like DRDO, teams often need to share **confidential and sensitive information**. Previously, this was done manually or via email, which posed several risks:  
- **Unauthorized access** to sensitive data.  
- **Lack of a centralized system** for managing file access and permissions.  
- **No tracking or logging**, making it difficult to monitor data usage.  

**Our Goal:**  
To build a **centralized and highly secure web platform** that allows multiple roles (e.g., Admin, Researcher, Analyst) to **upload, share, and access files** securely, ensuring only authorized users could interact with the data.

On the backend, I built RESTful APIs using Python Flask integrated with PostgreSQL, implementing JWT authentication, OAuth2 role-based access control, and AES-256 encryption for files. All data transfer was secured through SSL/TLS, ensuring complete protection in transit and at rest. I also added audit logging, multi-factor authentication, and input sanitization to prevent unauthorized access and common vulnerabilities.

On the frontend, I developed a React + Bootstrap interface that allowed users to upload, download, and manage files based on their roles, with real-time dashboards for activity tracking and permissions. Role-aware UI components ensured that admins, researchers, and analysts each had restricted, contextual views.

For deployment and automation, I configured Jenkins CI/CD pipelines and Docker containers to enable consistent, test-driven releases. I integrated Prometheus and Grafana dashboards for live monitoring and alerting, ensuring stability and traceability of all actions.

## **Project Outcomes and Results**  
1. **High-Security Standards:** Achieved **100% compliance with DRDO’s security protocols** by implementing encryption, access control, and monitoring.  
2. **Improved Data Sharing:** The platform enabled secure and efficient file sharing among research teams, reducing manual processes and errors.  
3. **Scalable and Robust Architecture:** Designed for scalability, ensuring future expansion and integration with other DRDO systems.  

---

## **2. Backend Development (Python Flask)**  
### **Why Python Flask?**  
We chose **Flask** for backend development because:  
- **Lightweight and Flexible:** Flask is highly customizable and well-suited for building APIs.  
- **Built-in Support for RESTful APIs:** It allowed us to quickly design and expose APIs for user authentication, file management, and access control.  
- **Integration with Security Libraries:** Flask seamlessly integrates with libraries for encryption, token-based authentication, and security protocols.

---

### **How the Backend Was Implemented:**  
1. **RESTful APIs:** Developed APIs for authentication, file upload/download, role management, and audit logging.  
2. **JWT (JSON Web Tokens):** Used for secure session management and token-based authentication.  
   - **Why JWT?** It provides a stateless authentication mechanism, reducing the need for server-side session storage and enhancing security.  
3. **Role-Based Access Control (RBAC):**  
   - Implemented **RBAC** to restrict access to sensitive resources based on user roles (Admin, Researcher, Analyst).  
   - Admins could manage roles and permissions, while researchers could only access data specific to their projects.  

---

## **3. Database Management (PostgreSQL)**  
### **Why PostgreSQL?**  
We selected **PostgreSQL** for its:  
- **ACID Compliance:** Ensures transactional integrity, which is crucial for handling sensitive data.  
- **Role-based security:** PostgreSQL supports granular access control at the table, column, and row level.  
- **JSON Support:** Simplified storage of metadata related to file uploads and user activities.

---

### **Database Structure:**  
- **Users Table:** Stores user information, roles, and permissions.  
- **Files Table:** Tracks uploaded files, associated metadata (uploader, timestamp), and encrypted file paths.  
- **Access Logs Table:** Maintains detailed logs of user activities for auditing purposes.

### **Data Encryption:**  
- **AES (Advanced Encryption Standard) for File Encryption:** Ensured that all uploaded files were encrypted before being stored in the database.  AES (Advanced Encryption Standard) is a symmetric key encryption algorithm, meaning the same key is used for both encrypting and decrypting data.It operates on fixed-size blocks of data (usually 128-bit blocks)
- **Why AES?** AES is a widely adopted and secure encryption standard that provides high performance and robust security.  

---

## **4. Security Enhancements**  
Security was the top priority for this project. Several measures were implemented to protect data from unauthorized access and cyberattacks.

### **1. SSL/TLS for Secure Communication:**  
SSL (Secure Sockets Layer) and TLS (Transport Layer Security) are cryptographic protocols that encrypt data between the client (browser/app) and the server (your backend).
**Why SSL/TLS?**  
- Ensures all communication between the client and server is encrypted.  
- Prevents man-in-the-middle (MITM) attacks.  
- Protects sensitive data (passwords, tokens) during transmission.

### **2. Input Validation and Sanitization:**  
Prevented common vulnerabilities like **SQL injection**, users entering unexpected or harmful data to exploit your system. **cross-site scripting (XSS)**Attackers inject malicious JavaScript into a web page seen by others., and **cross-site request forgery (CSRF)** Attackers inject malicious JavaScript into a web page seen by others. by sanitizing user inputs and using prepared statements for database queries.

### **3. Multi-Factor Authentication (MFA):**  
**Why MFA?**  
- Added an extra layer of security for accessing the platform.  
- Combined **password-based authentication with one-time passcodes (OTP)** to ensure only verified users could access sensitive data.

---

## **5. Frontend Development (ReactJS)**  
### **Why ReactJS?**  
We chose **ReactJS** for its modern UI capabilities, making it easier to build a dynamic and responsive user interface:  
- **Component-based architecture** simplified the development and maintenance of the application.  
- **Bootstrap integration** helped create a clean and mobile-friendly interface.  
- **Real-time UI updates** for file uploads, role management, and activity monitoring provided a smooth user experience.

---

### **Frontend Features:**  
1. **User Dashboard:** Displays uploaded files, user activity logs, and role-based permissions.  
2. **File Upload/Download:** Supports secure file upload and download, with real-time progress indicators.  
3. **Audit Logs View:** Allows admins to monitor all user activities in a timeline view for better control and security.

---

## **6. Continuous Integration and Deployment (CI/CD)**  
### **Why CI/CD Was Necessary?**  
Automating the build, test, and deployment processes ensured consistent and reliable deployments, reducing manual errors.  

### **Tools Used:**  
- **Jenkins for CI/CD Pipelines:** Automated testing and deployment to staging and production environments.  
- **Docker for Containerization:** Ensured consistency across development, testing, and production environments.  
- **SonarQube for Code Quality Analysis:** Integrated into the pipeline to check for vulnerabilities and maintain high code standards.  

### **Pipeline Steps:**  
1. **Build and Unit Tests:** Automatically triggered on every code commit.  
2. **Security Scans:** Detect potential vulnerabilities before deployment.  
3. **Automated Deployment:** Ensured smooth and consistent releases.

---

## **7. Monitoring and Logging**  
Monitoring was crucial to ensure the stability and security of the platform.  

### **Tools Used:**  
- **Prometheus and Grafana:** For real-time monitoring of server health, API performance, and database usage.  
- **Audit Logs:** Recorded every user action (file upload, download, permission changes), helping with compliance and forensic analysis.  
- Improved accountability with detailed audit trails.

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
