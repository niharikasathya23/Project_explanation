# Project_explanation

# Niharika's Project Portfolio

Welcome to my project portfolio! This README provides a list of my key projects with detailed explanations for each. You can click on the project headings to directly navigate to their descriptions and details.

---

## Table of Contents
1. [Lightweight Dense Spatial Perception for Low-SWaP Robots](#1-lightweight-dense-spatial-perception-for-low-swap-robots)
2. [StockWatch: Real-Time Big Data Pipeline for Stock Sentiment Analysis](#2-stockwatch-real-time-big-data-pipeline-for-stock-sentiment-analysis)
3. [Movie Recommendation System](#3-movie-recommendation-system)
4. [Automatic Hate Speech Detection Using Ensemble Methods](#4-automatic-hate-speech-detection-using-ensemble-methods)
5. [Tequed Attendance Management App](#5-tequed-attendance-management-app)
6. [Rakuten - Software Engineer Intern](#6-rakuten---software-engineer-intern)
7. [Defence Research & Development Organisation (CAIR) - Software Development Intern](#7-defence-research--development-organisation-cair---software-development-intern)

---

## 1. Lightweight Dense Spatial Perception for Low-SWaP Robots
**Project Overview:**  

**Situation:**
 "As part of my role as a Research Assistant at the HIRO Lab, University of Colorado Boulder, I was working on improving the spatial perception capabilities of low-SWaP robots—those that operate under strict constraints in size, weight, and power. Typically, achieving high-quality 3D perception for these robots is a challenge because most neural networks are too computationally intensive to run in real-time on embedded systems. Imagine trying to use a powerful desktop-class neural network on a tiny device running on battery—it just isn’t feasible."
 
**Task:**
 "My task was to develop a lightweight neural network that could provide dense spatial perception in real-time on embedded devices. Specifically, this involved creating a model that could perform image segmentation (labeling pixels as background, human, or object) while simultaneously refining sensor-generated depth data to produce a more accurate 3D map of the scene. This is particularly useful in scenarios like human-robot handovers, where the robot needs a precise understanding of where the person and object are in 3D space to avoid errors."
 
**Action:**
 "To tackle this, I developed Bilateral Segmentation and Disparity Refinement (BSDR)—a unique neural network that performs two tasks simultaneously: segmentation and depth refinement. Here’s how it works:
It takes a monocular image and a raw disparity map as input. The model’s architecture is inspired by BiSeNetv1, but I added a second head to refine depth data alongside segmentation.
We optimized the architecture by reducing the number of parameters and using a multi-layer perceptron (MLP) for sub-pixel disparity prediction. This reduces computational overhead while preserving accuracy.
I also curated a custom dataset for training using CREStereo, a highly accurate but computationally heavy depth estimation model. This gave us precise ground truth depth data, which helped train BSDR to improve on noisy sensor data."
"For example, the OAK camera’s standard depth map had holes and noise, especially around occluded regions like a person’s hand holding an object. BSDR not only smoothed the noisy data but also filled in missing depth information, making the 3D reconstruction much more reliable. I ran BSDR on a Luxonis OAK camera at 11 FPS, outperforming similar models that couldn’t achieve real-time performance."

**Result:**
 "The results were exciting! BSDR produced real-time dense 3D scene reconstructions on embedded hardware while using only 7.4 GFlops and 1.9 million parameters, compared to CREStereo’s 47.3 GFlops and 15.2 million parameters. This reduced the computational load by over 80% while improving accuracy in critical tasks like detecting and segmenting objects in close-proximity human-robot interaction.
In practical terms, this allowed robots to recognize a person and the object they’re holding much more accurately. For human-robot handovers, this means fewer errors and smoother interactions. It also opens up possibilities for low-power robotics, like drones and mobile manipulators, to operate in more complex environments without needing high-power computing hardware."

---

### **Model Architecture Explanation with Design Justification and Advantages**

**Model Overview:**  
"In this project, I developed **Bilateral Segmentation and Disparity Refinement (BSDR)**, a lightweight neural network designed for **real-time dense spatial perception** on low-SWaP (Size, Weight, and Power) robots. This model is based on **BiSeNetv1**, a well-known architecture for real-time image segmentation. I extended BiSeNetv1 by adding a second head for **disparity refinement**, making it capable of performing **two tasks simultaneously**:  
1. **Pixel-wise segmentation** – Classifying pixels as background, person, or handheld object.  
2. **Disparity map refinement** – Correcting and improving raw sensor depth maps for better 3D perception."

---

### **Why BiSeNetv1 as the Base Model?**  
"I chose **BiSeNetv1** because it strikes an ideal balance between **accuracy and real-time performance**. Unlike heavy models like DeepLabv3 or U-Net, BiSeNetv1 is lightweight and well-suited for resource-constrained devices like the Luxonis OAK camera. Its dual-path structure captures both **low-level spatial details** and **high-level semantic features**, which is critical for tasks like close-proximity interaction in robotics."

**Advantages:**  
- **Low computational cost** with real-time performance  
- **Accurate pixel-wise segmentation**, even for small objects  
- **Fewer parameters**, making it feasible to deploy on embedded systems  

---

### **Architecture and Key Components**  

**Input and Preprocessing:**  
"The model takes a **two-channel input**, which includes a **monocular image** (grayscale or RGB) and a **raw disparity map** from the OAK camera. I set the input resolution to **640x384**, ensuring the model captures enough detail while keeping it fast enough for real-time inference."

**Core Architecture:**  
1. **Spatial Path:** Captures **low-level spatial features** like edges and fine details, which are essential for refining depth information.  
2. **Context Path:** Focuses on **high-level features** to improve semantic understanding of objects and their relationships in the scene.  
3. **Feature Fusion Module:** Combines both paths to generate richer feature representations for segmentation and depth refinement.  

**Dual-Head Structure:**  
"I added a second head to the original BiSeNetv1 architecture:  
- **Segmentation Head**: Classifies each pixel into background, person, or handheld object using a multi-layer perceptron (MLP).  
- **Disparity Refinement Head**: Refines the raw disparity map using sub-pixel disparity prediction, filling holes and smoothing noisy depth data. This head is inspired by **Neural Disparity Refinement (NDR)**, but customized for better performance on embedded devices."

---

### **Why Add Disparity Refinement?**  
"Standard stereo depth sensors often produce noisy and incomplete depth maps, especially around occluded regions like hands holding objects. By adding a disparity refinement head, BSDR reduces errors and fills in missing depth data, which significantly improves 3D scene reconstruction."

**Advantages:**  
- **Improved accuracy** in 3D perception, especially in occlusion-heavy scenes  
- **Real-time refinement** without external post-processing  
- **Smoother and more reliable depth maps**, enhancing robot interaction in close proximity  

---

### **Loss Function and Training**  
"I used a custom loss function with three components to ensure both segmentation and depth refinement were optimized:  
1. **Segmentation Loss:** A combination of Focal Loss and Water-Obstacle Separation Loss (WSL) to improve pixel classification, even for small or hard-to-detect objects.  
2. **Integer Disparity Loss:** A cross-entropy loss with soft labels to predict discrete disparity values.  
3. **Disparity Offset Loss:** An L1 loss that refines the final disparity to match ground truth values."

**Training Setup:**  
- **AdamW optimizer** with a learning rate of **1e-4**  
- **946 epochs** on Nvidia Tesla GPUs with batch size 8  
- **Custom data augmentation** (random crops, Gaussian noise, simulated stereo matching errors) to improve generalization  
- Dataset curated using **CREStereo**, a state-of-the-art depth estimation model, to ensure accurate ground truth disparity.  

---

### **Performance Optimizations**  
1. **Downsampling and Upsampling:** Features were downsampled by a factor of 8 during processing and restored using 1D upsampling to reduce computational cost without sacrificing detail.  
2. **Reduced Parameters and GFlops:** BSDR has only **1.9M parameters and 7.4 GFlops**, compared to CREStereo’s **15.2M parameters and 47.3 GFlops**. This optimization made real-time performance on the Luxonis OAK camera possible, running at **11 FPS**.  

---

### **Performance Results and Metrics**  
"The model’s performance was evaluated using two main metrics:  
1. **Mean Intersection over Union (mIoU):** Achieved an overall mIoU of **89.89%**, with **96.3% for background**, **90.43% for person**, and **82.95% for handheld objects**.  
2. **Disparity Non-Occluded Mean Average Error (MAEdno):** BSDR reduced occlusion errors by **14.62% on average**, especially improving person disparity by **15.07% compared to raw sensor data**."

---

### **Example in Real-World Scenario**  
"In a human-robot handover scenario, raw depth data from the OAK camera often has gaps and noise around the person’s hand and the object being held. BSDR fills in these gaps, producing a **smoother and more reliable 3D representation**. This enables the robot to accurately detect and track objects in real-time, ensuring safe and precise interactions.  

For instance, before using BSDR, the depth map might miss parts of the hand and object, causing the robot to miscalculate its position. After refinement with BSDR, the robot has a clear, complete depth map, reducing handover errors and enhancing overall interaction quality."

---

Let me know if you want to further expand on **model benchmarks, specific datasets, or hardware comparisons**!
---

## 2. StockWatch: Real-Time Big Data Pipeline for Stock Sentiment Analysis
**Project Overview:**  
Here’s the detailed **STAR explanation** for **StockWatch: Real-Time Big Data Pipeline for Stock Sentiment Analysis**.

---

## StockWatch: Real-Time Big Data Pipeline for Stock Sentiment Analysis  

### **Situation:**  
"In the financial industry, the correlation between public sentiment and stock market performance has been widely observed. Companies are greatly influenced by social media reactions, especially on platforms like **Twitter**. For instance, a positive or negative tweet about a company can result in a noticeable impact on its stock price.  

The problem, however, is collecting, processing, and analyzing large-scale real-time data from multiple sources in an efficient manner. Traditional batch-processing systems are too slow for this use case. What we needed was a **real-time big data pipeline** that could gather and analyze stock-related tweets and correlate them with live stock prices."

---

### **Task:**  
"My task was to **design and implement a real-time data pipeline** that could:  
1. **Ingest stock-related tweets** from **Twitter APIs** and **Yahoo Finance** in real time.  
2. **Process and store the data** with minimal latency for efficient querying and analysis.  
3. **Perform sentiment analysis** on each tweet to determine its polarity (positive, negative, neutral) and **correlate the sentiment with stock price trends**.  
4. Ensure **scalability** to handle large volumes of data and **processing latency under 500 milliseconds**."

---

### **Action:**  
"To achieve this, I designed a **high-performance big data pipeline** using a combination of modern technologies for **data ingestion, processing, and storage**. Here's how it worked:

1. **Data Ingestion:**  
   - I used **Kafka** as a message broker to stream real-time data from **Twitter and Yahoo Finance APIs**. Kafka's distributed architecture ensured high throughput and fault tolerance.  

2. **Data Processing:**  
   - **PySpark** was used to process the data in real-time. I wrote **custom PySpark jobs** to clean, transform, and analyze the data.  
   - Sentiment analysis was performed on each tweet using **NLTK** and **spaCy**. These libraries allowed us to tokenize and classify the text, generating a sentiment score for each tweet.  

3. **Data Storage:**  
   - I chose **MongoDB** for its flexibility in handling unstructured data like tweets. MongoDB’s **fast read-write operations** made it ideal for real-time querying and storage.  

4. **Deployment and Scaling:**  
   - The entire pipeline was deployed on **AWS** to ensure scalability. Kafka and PySpark clusters were configured to scale up or down based on data load.  
   - I used **AWS Lambda** for auxiliary tasks like cleaning older data, and **CloudWatch** for monitoring system health."

---

### **Example in Action:**  
"Let’s say the system receives a tweet like: *‘Tesla is releasing a new self-driving feature next week! #TSLA’.*  
- **Ingestion:** This tweet is picked up by the Kafka pipeline.  
- **Processing:** PySpark processes it in real time, running sentiment analysis to classify it as **positive**.  
- **Correlation:** This positive sentiment is then linked with Tesla’s stock price. Analysts or trading algorithms can then use this information to make decisions."

---

### **Result:**  
"The pipeline successfully processed and analyzed **1,000 tweets per hour** with an average processing latency of **less than 500 milliseconds**.  

---

## **StockWatch: Real-Time Big Data Pipeline for Stock Sentiment Analysis**  

### **Overview**  
StockWatch is a high-performance **real-time data pipeline** designed to analyze stock-related tweets and correlate them with live stock prices. The system processes data from multiple sources, performing **sentiment analysis** on tweets and providing real-time insights into stock trends. The pipeline is built with **scalability, fault tolerance, and low-latency processing** in mind, ensuring reliable performance even under high data loads.  

### **Design Choices and Architecture**  

#### **Data Ingestion with Kafka**  
The first step in the pipeline is **data ingestion**. **Apache Kafka** was chosen for its ability to handle large-scale data streams in real time with high throughput and fault tolerance. Kafka’s **distributed architecture** allows it to manage incoming data from multiple sources without data loss. Each source stream is assigned to a separate **Kafka topic** for efficient partitioning and consumption.

**How Kafka was used:**  
- **Stock-related tweets** were streamed from the **Twitter API** into the `tweets_topic`.  
- **Stock price data** from **Yahoo Finance** was ingested into the `stock_data_topic`.  
- Kafka’s **consumer groups** allowed the PySpark jobs to process data in parallel, ensuring high availability and scalability.  

Kafka was configured with **replication and offset management**, ensuring the pipeline could recover from failures without data loss. Its **exactly-once semantics** guaranteed that each message was processed exactly once.

---

#### **Data Processing with PySpark**  
To process and analyze the real-time data, **PySpark** was selected for its distributed data processing capabilities. PySpark provides **Structured Streaming**, which enables efficient, low-latency stream processing. Compared to plain Python, PySpark allows for parallel processing across a cluster, significantly improving performance.

**Processing Steps:**  
1. **Sentiment Analysis:**  
   Sentiment analysis was performed on each tweet using **NLTK** and **spaCy**. NLTK provided tokenization and basic sentiment scoring, while **spaCy’s Named Entity Recognition (NER)** was used to extract relevant entities like company names and stock tickers (e.g., `$AAPL` for Apple Inc.).  
   
2. **Data Enrichment:**  
   The system enriched each tweet by correlating it with the current stock price and other metadata from Yahoo Finance. This allowed for real-time insights into how public sentiment was affecting stock performance.  

3. **Aggregation and Rolling Sentiment:**  
   PySpark performed **windowed aggregations** to calculate rolling sentiment scores over the past 1-minute and 5-minute intervals for each stock ticker. These rolling metrics provided valuable trends in sentiment shifts.  

---

#### **Data Storage with MongoDB**  
**MongoDB** was chosen as the primary data storage solution because of its ability to handle unstructured data like tweets. MongoDB’s **document-based model** provided the flexibility needed to store and query data without the constraints of a fixed schema.

**Storage Details:**  
- Processed tweets with sentiment scores were stored in the `tweets_data` collection.  
- Each document included fields such as the tweet content, timestamp, sentiment score, and associated stock ticker.  
- Aggregated results were stored in the `stock_sentiment` collection for quick retrieval by a real-time dashboard.  

MongoDB’s **horizontal scaling** and fast read-write operations ensured that the system could handle large volumes of data while providing near-instantaneous query responses.

---

#### **Cloud Infrastructure and Deployment on AWS**  
The entire pipeline was deployed on **AWS**, leveraging various services to ensure **scalability, high availability, and cost-efficiency**. AWS offered the flexibility to manage infrastructure while automating auxiliary tasks.

**Key AWS Components:**  
1. **EC2 for Kafka and PySpark:**  
   Kafka and PySpark were deployed on **EC2 instances** for better control and configuration. These instances were set up in an **auto-scaling group** to dynamically adjust the resources based on incoming data volume. This approach provided **cost efficiency** while ensuring enough resources were available during peak data loads.  

2. **AWS S3 for Data Backup:**  
   All raw data from the Twitter API and Yahoo Finance was archived in **AWS S3** for long-term storage and backup. This archival strategy ensured a reliable data recovery mechanism in case of failures.  

3. **AWS Lambda for Automation:**  
   **AWS Lambda** was used for periodic tasks such as cleaning up old data in MongoDB and generating alerts when sentiment scores crossed predefined thresholds. This serverless approach reduced the operational overhead and kept auxiliary services lightweight.  

4. **Monitoring with AWS CloudWatch:**  
   **AWS CloudWatch** was configured to monitor the health of the EC2 instances, Kafka brokers, and PySpark jobs. Metrics such as CPU usage, memory consumption, and Kafka consumer lag were tracked in real time. CloudWatch alerts were triggered for any anomalies, ensuring immediate action when needed.  

---

### **Performance and Results**  
The StockWatch pipeline achieved impressive results in terms of scalability, processing speed, and accuracy.  

- **Processing Latency:** The pipeline processed and analyzed **1,000 tweets per hour** with an average latency of **less than 500 milliseconds** from ingestion to storage.  
- **Sentiment Accuracy:** The combination of NLTK and spaCy improved sentiment classification, especially for stock-related language and financial terms.  
- **Scalability:** Kafka and PySpark handled high data volumes without performance degradation, thanks to their distributed architectures and auto-scaling on AWS.  

The resulting data allowed for **real-time insights** into how social media sentiment affected stock performance. Traders and analysts could use this information to make faster, more informed decisions.

---

### **Why This Architecture Worked Best**  
The combination of Kafka, PySpark, MongoDB, and AWS offered a **perfect balance of speed, flexibility, and scalability**.  

- **Kafka** ensured fault-tolerant, real-time data ingestion.  
- **PySpark** provided powerful stream processing with low latency and high throughput.  
- **MongoDB** allowed for easy storage and retrieval of unstructured data.  
- **AWS** provided the infrastructure to scale and manage resources efficiently, keeping costs in check while maintaining performance.  

By combining these technologies, the StockWatch pipeline became a highly scalable, real-time analytics platform ready for large-scale adoption.

---

## 3. Movie Recommendation System
**Project Overview:**  

## Situation  
"In the era of streaming platforms, users are constantly searching for personalized movie recommendations. Many platforms struggle with **real-time recommendation generation** for large user bases due to the **sheer volume of data** and the **computational complexity** of collaborative filtering.  

I set out to build a **scalable movie recommendation system** that could provide **personalized suggestions** based on user preferences and behavior, with a focus on reducing latency and ensuring the system could handle high traffic."

---

## Task  
"My task was to design and implement a recommendation engine that could:  
1. **Generate personalized movie recommendations** using **collaborative filtering** techniques.  
2. Build a **full-stack application** with a scalable backend and an intuitive, responsive frontend.  
3. Ensure **real-time performance and scalability**, enabling the system to handle concurrent users efficiently.  
4. **Deploy the application on the cloud** with **Docker** for easy scalability and maintainability."

---

## Action  

### 1. Collaborative Filtering-based Recommendation Engine  
The recommendation engine was implemented using **collaborative filtering** to predict a user’s interest in a movie based on the behavior of other similar users.  

**Implementation Steps:**  
- The engine was developed in **Python** using **Pandas and NumPy** for data processing.  
- I used **cosine similarity** to calculate user-item similarities and generate movie suggestions.  
- All data was stored in **MongoDB**, which allowed for fast and flexible querying of user preferences and movie metadata.  

---

### 2. Backend with FastAPI  
**Why FastAPI?**  
- **High performance and low latency** due to its asynchronous capabilities.  
- Built-in **data validation** and **automatic API documentation** using OpenAPI.  

**Implementation Details:**  
- Developed the backend with **FastAPI** to expose RESTful endpoints for user management, movie search, and recommendation requests.  
- The backend fetched movie data from **MongoDB** and returned real-time personalized recommendations.  

---

### 3. Frontend with ReactJS  
The frontend was built using **ReactJS** for a **modern, dynamic user experience**. It allowed users to search for movies, view personalized recommendations, and manage their preferences.  

**Features:**  
- **Dynamic search** with autocomplete for fast movie lookups.  
- **User-friendly recommendation interface** showing personalized movie suggestions.  
- **React hooks** for managing state and ensuring efficient component updates.  

---

### 4. Deployment with Docker and Google Cloud Platform  
**Why Docker?**  
- Docker provided a **containerized environment**, ensuring consistency across development, testing, and production.  
- It simplified deployment, enabling **faster updates** and **easy scalability**.  

**Google Cloud Platform (GCP):**  
- The application was hosted on **Google Cloud**, providing **high availability and scalability** to handle spikes in traffic.  
- The backend API and recommendation engine were deployed in **Docker containers**, managed by GCP’s **Kubernetes Engine**, allowing for **automatic scaling** and **load balancing**.  

---

## Result  
The system successfully generated **personalized movie recommendations** in real-time with **minimal latency**, even under high traffic conditions.  

### Key Results:  
- **25% reduction in deployment times** thanks to Docker containerization.  
- The system scaled to support a **35% surge in concurrent users** without performance degradation.  
- The recommendation engine provided **accurate predictions**, improving user engagement and satisfaction.  
- Hosting on **Google Cloud Platform** ensured **high availability and easy maintenance**.  

---
Here’s an even **more detailed explanation** for your **Movie Recommendation System**, written to address every potential follow-up question about **design choices, technology selection, scalability, deployment, and algorithmic decisions**.

---

# **Movie Recommendation System**  
A **collaborative filtering-based recommendation engine** built with **FastAPI, MongoDB, and ReactJS**. The goal of this project was to create a **real-time, scalable solution** for generating personalized movie suggestions while ensuring high availability and fast performance using **Docker** and **Google Cloud Platform (GCP)**.

---

## **Overview and Objective**  
In modern streaming platforms, personalized recommendations have become essential for improving user engagement and retention. Traditional recommendation systems struggle with handling **real-time prediction generation** for large-scale users because of high computational complexity and the sheer volume of data. The objective was to **design a recommendation system** that could provide users with **real-time suggestions** using collaborative filtering, supported by a scalable architecture for both **backend services** and **frontend delivery**.

Key goals for the project:  
1. **Generate accurate and personalized movie recommendations** based on user preferences using **collaborative filtering techniques**.  
2. Design a **full-stack architecture** with a **FastAPI-based backend** and a **ReactJS frontend** for a seamless user experience.  
3. **Deploy the system on the cloud** using **Docker** to ensure scalability and consistency across environments.  
4. Provide real-time performance, enabling the system to handle concurrent users and high traffic efficiently.  

---

## **System Architecture and Technology Stack**  

The system was designed with the following key components:  
- **Recommendation Engine**: Collaborative filtering-based recommendation engine using **Python**, **Pandas**, and **NumPy**.  
- **Backend API**: Built with **FastAPI** for real-time communication and data processing.  
- **Database**: **MongoDB** for storing user preferences, movie metadata, and recommendation results.  
- **Frontend Interface**: Developed with **ReactJS** for a modern, responsive user experience.  
- **Deployment**: Dockerized and hosted on **Google Cloud Platform (GCP)** for scalability and reliability.

---

## **Detailed Technical Explanation**  

### **1. Collaborative Filtering-based Recommendation Engine**  
The recommendation engine was built using **collaborative filtering**, which generates suggestions based on similarities between users or items. Unlike content-based methods that rely on movie metadata, collaborative filtering predicts a user’s interest by analyzing the behavior of similar users.  

#### **Algorithm and Implementation:**  
- The engine was implemented in **Python**, using **Pandas and NumPy** for data manipulation and similarity computations.  
- **User-User Collaborative Filtering** was used with a **cosine similarity measure** to calculate the similarity between users based on their movie ratings and preferences.  
- The recommendation process involved computing the weighted average of other users’ ratings for unseen movies and ranking the top suggestions.  

#### **Why Collaborative Filtering?**  
- It **captures complex relationships** between users and movies that content-based systems often miss.  
- **Scalability:** Easily extensible to new users and items without requiring extensive feature engineering.  

#### **Data Storage:**  
The user-item rating matrix and movie metadata were stored in **MongoDB** for efficient querying and integration with the backend.

---

### **2. Backend with FastAPI**  
The backend was built using **FastAPI**, a modern web framework for Python known for its high performance and asynchronous capabilities. It served as the **central API layer** for the recommendation engine and user management.

#### **Why FastAPI?**  
- **Asynchronous support:** FastAPI leverages Python’s async capabilities, ensuring fast responses and better handling of concurrent requests.  
- **Built-in data validation:** Using **Pydantic models**, FastAPI provides automatic validation and serialization for API requests and responses.  
- **Automatic API documentation:** FastAPI generates OpenAPI documentation, making it easier to test and maintain the APIs.

#### **Implementation Details:**  
- **Endpoints for user management:** Register new users, update preferences, and retrieve personalized recommendations.  
- **Integration with MongoDB:** Real-time queries to fetch user data and movie recommendations.  
- **Recommendation API:** The API handled real-time recommendation requests, processed the data through the collaborative filtering engine, and returned the top movie suggestions.  

---

### **3. Frontend with ReactJS**  
The frontend was developed using **ReactJS** for an intuitive and responsive user interface. It allowed users to search for movies, view recommendations, and manage their preferences.

#### **Features and Implementation:**  
- **Dynamic search with autocomplete** for fast and seamless movie lookup.  
- **Personalized recommendation interface** that displayed real-time suggestions based on user preferences.  
- **State management with React hooks** to ensure efficient UI updates and smooth interactions.  
- **Integration with the FastAPI backend** for real-time communication using RESTful APIs.  

---

### **4. Deployment with Docker and Google Cloud Platform (GCP)**  
The system was containerized using **Docker** to ensure consistent deployment across development, testing, and production environments.  

#### **Why Docker?**  
- **Environment consistency:** Docker eliminated "it works on my machine" issues by providing a uniform environment.  
- **Simplified scaling:** Docker containers made it easier to scale the backend and recommendation engine independently.  
- **Rapid deployment:** Faster deployments and rollback with Docker images.  

#### **Hosting on Google Cloud Platform (GCP):**  
- **Compute Engine (GCE):** Hosted the backend API and recommendation engine on scalable instances.  
- **Cloud Load Balancer:** Ensured high availability and distributed incoming requests across multiple instances.  
- **Google Kubernetes Engine (GKE):** Managed Docker containers and automatically scaled services based on traffic.  

---

## **Performance and Scalability**  
The system was designed to handle high traffic while maintaining low latency and accurate recommendations.  

**Key Results:**  
- **Real-time recommendations** with a response time of less than **200 milliseconds** per request.  
- **25% reduction in deployment time** with Docker containerization.  
- Scaled to handle a **35% increase in concurrent users** without performance degradation.  
- **High availability** ensured by Google Cloud Platform with automatic scaling and load balancing.  

---

## **Why This Architecture Worked Best**  
- **Scalable and Flexible:** The combination of FastAPI, MongoDB, and ReactJS provided a flexible and scalable architecture that could adapt to changing user needs.  
- **High Performance:** FastAPI’s asynchronous processing and Docker-based deployment ensured fast, reliable performance.  
- **Cost-effective and Reliable Deployment:** Google Cloud Platform provided a robust infrastructure with minimal operational overhead.  

---

## **Future Enhancements**  
1. **Incorporating Content-Based Filtering:** Combine collaborative filtering with metadata-based recommendations for better accuracy.  
2. **Hybrid Models:** Use hybrid approaches that blend collaborative filtering with deep learning-based recommendation systems.  
3. **Real-time User Behavior Tracking:** Implement real-time tracking to refine recommendations based on immediate user actions.  
___

## 4. Automatic Hate Speech Detection Using Ensemble Methods

### **Situation:**  
"With the increasing use of social media, hate speech and offensive content have become serious issues. Traditional models often struggle with detecting hate speech in **multilingual or code-mixed languages** like Hinglish (Hindi-English). Most existing systems only offer **binary classification** (hate vs. non-hate) and fail to classify content into multiple categories such as **hate, offensive, or neither**."

---

### **Task:**  
"Our goal was to **build a robust hate speech detection model** that could accurately classify social media content into **three categories: hate, offensive, or neither**, while handling both **English and Hinglish** text. We aimed to improve accuracy using an **ensemble machine learning approach** and provide the system via a **web-based interface** for real-time analysis."

---

### **Action:**  
"To accomplish this, we:  
1. **Collected data** from Kaggle (English) and GitHub (Hinglish) datasets.  
2. **Preprocessed the data** (tokenization, stop-word removal, and case folding) and extracted features using **TF-IDF** with n-grams.  
3. Developed an **ensemble model** combining **Random Forest and Support Vector Classifier (SVC)** after evaluating multiple classifiers.  
4. **Trained and validated** the model on an 80-20 train-test split, achieving high accuracy.  
5. Created a **web application** to make the model accessible for real-time hate speech detection."

---

### **Result:**  
"The ensemble model achieved **90.7% accuracy**, outperforming other combinations and single classifiers. It effectively classified **English and Hinglish content** and provided a practical, easy-to-use solution for detecting hate speech in real time. The **web application** made the tool accessible for real-world use, improving online content moderation and safety."
---

## **1. Data Collection**  
### **Why These Datasets?**  
- We collected **English dataset from Kaggle** and a **Hinglish (Hindi-English code-mixed) dataset from GitHub**. This combination ensured that the model could detect hate speech in both pure English and the increasingly common Hinglish content.  
- **Multilingual data** posed challenges but made the model highly adaptable for real-world social media data, where such code-mixed content is prevalent.

### **Advantages:**  
- **Diverse Data:** Improved generalization by training on a variety of text inputs.  
- **Practical Application:** Focused on real-world social media scenarios with mixed-language content.  

### **Implementation:**  
- The collected data was labeled into three classes: **Hate**, **Offensive**, and **Neither**. This labeling helped in multi-class classification.

---

## **2. Data Preprocessing**  
### **Why Preprocessing is Essential?**  
Raw text data contains noise, such as special characters, stop words, and case variations, that can confuse machine learning models. Preprocessing ensures that the data is clean and standardized.

### **Preprocessing Steps:**  
1. **Case Folding:** Converted all text to lowercase to avoid treating "Hate" and "hate" as different words.  
2. **Tokenization:** Split sentences into individual words, enabling feature extraction techniques to analyze them effectively.  
3. **Stop-word Removal:** Removed common but irrelevant words (like “is”, “and”, “the”), which do not contribute to the classification task.  
4. **Special Character Removal:** Cleaned the dataset by removing punctuation, hashtags, emojis, and special symbols.  

### **Advantages:**  
- **Noise Reduction:** Helps the model focus on meaningful content.  
- **Consistency:** Creates a uniform structure for the data, improving model accuracy.  

---

## **3. Feature Extraction with TF-IDF**  
### **Why Use TF-IDF?**  
- **Term Frequency-Inverse Document Frequency (TF-IDF)** is a proven feature extraction technique in Natural Language Processing (NLP). It captures the importance of a word in a document relative to the entire dataset.  
- **n-grams (unigrams, bigrams, trigrams)** were used to detect patterns of words (e.g., "hate speech", "offensive term") that indicate hate or offensive content.

### **Advantages:**  
- **Effective Numerical Representation:** Converts text into numerical vectors for machine learning models.  
- **Focus on Key Words:** TF-IDF emphasizes rare but important words while de-emphasizing commonly used ones.  
- **Improves Pattern Detection:** n-grams enable the model to understand word combinations rather than individual words in isolation.  

### **Implementation:**  
- The TF-IDF vectorizer was applied to convert preprocessed text into a feature matrix that served as input to the machine learning models.

---

## **4. Model Selection and Ensemble Learning**  
### **Why an Ensemble Model?**  
Single classifiers often fail to capture the complex patterns present in multilingual and code-mixed data. An ensemble approach combines the strengths of multiple classifiers to improve accuracy and reduce errors.

### **Model Selection Process:**  
1. **Evaluated Individual Classifiers:**  
   - **Random Forest:** Strong at handling high-dimensional data and reducing overfitting.  
   - **Support Vector Classifier (SVC):** Known for its ability to find optimal decision boundaries, especially in multi-class classification tasks.  
   - **Logistic Regression, Naive Bayes, Decision Tree:** Used as benchmarks for performance comparison.

2. **Best Combination:** The ensemble model combining **Random Forest and Support Vector Classifier (SVC)** outperformed other combinations in accuracy and robustness.

### **Advantages:**  
- **Higher Accuracy:** The ensemble model achieved **90.7% accuracy**, significantly better than individual classifiers.  
- **Reduced Overfitting:** Leveraging multiple classifiers prevents the model from focusing too much on specific patterns in the training data.  
- **Improved Generalization:** Handles complex patterns in code-mixed language more effectively.

### **Implementation:**  
- The ensemble model was built using **Scikit-learn**, with hyperparameter tuning to optimize performance.  
- An **80-20 train-test split** ensured that the model was trained on a substantial dataset while retaining enough data for accurate validation.

---

## **5. Model Evaluation and Validation**  
### **Why Evaluation is Crucial?**  
Evaluation ensures that the model generalizes well to new, unseen data and performs consistently across different categories (hate, offensive, neither).

### **Metrics Used:**  
- **Accuracy:** Overall correctness of predictions.  
- **Precision, Recall, and F1-Score:** Provided a deeper understanding of how well the model identified hate and offensive speech while minimizing false positives and false negatives.  
- **Confusion Matrix:** Helped analyze misclassifications and improve model performance through iterative tuning.  

### **Advantages:**  
- **Comprehensive Assessment:** Precision and recall balance ensured that both minority and majority classes were handled well.  
- **Confusion Matrix Insights:** Revealed where the model struggled (e.g., distinguishing offensive from hate speech).  

---

## **6. Web Application Development**  
### **Why Build a Web App?**  
The web application made the model accessible for real-time use, providing practical value for social media monitoring and content moderation.

### **Implementation Details:**  
- **Flask Backend:** Served the trained ensemble model for real-time predictions.  
- **User Interface:** A clean and simple interface allowed users to input text and get instant classification results (Hate, Offensive, or Neither).  
- **Deployment:** The app was containerized using **Docker** for consistency across environments.

### **Advantages:**  
- **Real-Time Predictions:** Enabled immediate feedback, making it practical for real-world usage.  
- **Scalable:** Dockerized architecture ensured easy deployment and scaling.  

---

## **7. Performance and Results**  
- The ensemble model achieved **90.7% accuracy**, outperforming other combinations and single classifiers.  
- **Robust Multilingual Handling:** Successfully classified both English and Hinglish content.  
- The web application provided a practical tool for detecting hate speech in real time, promoting safer online spaces.

---

## **Why This Approach Worked Best**  
- **Ensemble Learning:** Combined strengths of multiple models, improving accuracy and robustness.  
- **TF-IDF with n-grams:** Enhanced pattern detection in both English and Hinglish data.  
- **Web Application:** Translated the research into a practical tool for real-time use.

---

### **Future Enhancements:**  
1. **Deep Learning Models:** Incorporate transformers like BERT for better contextual understanding.  
2. **Multilingual Support Expansion:** Extend the model to handle more languages like Tamil-English or Bengali-English.  
3. **Real-Time Social Media Integration:** Automate hate speech detection on live streams from platforms like Twitter and Facebook.  

---

## 5. Tequed Attendance Management App
**Project Overview:**  
A mobile app for managing attendance using **Android Studio and Java**. The app features a **customized UI/UX** and a robust backend with **Firebase** for real-time updates.

**Key Results:**
- Reduced manual tracking errors by **20%**
- Increased attendance record accuracy by **25%**

---

## 6. Rakuten - Software Engineer Intern
**Duration:** Jan 2022 – Jul 2022  
**Location:** Bangalore, India  

#### **Key Contributions:**  
- Developed and enhanced features for the **B2B order management portal** using AngularJS, TypeScript, Spring Boot, and MySQL  
- Collaborated with cross-functional teams and implemented **CI/CD pipelines**, reducing feature deployment time by 20%  
- Improved application performance by resolving 35 high-priority bugs, reducing post-release defects by 20%  

---

## 7. Defence Research & Development Organisation (CAIR) - Software Development Intern

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
- **AES (Advanced Encryption Standard) for File Encryption:** Ensured that all uploaded files were encrypted before being stored in the database.  
- **Why AES?** AES is a widely adopted and secure encryption standard that provides high performance and robust security.  

---

## **4. Security Enhancements**  
Security was the top priority for this project. Several measures were implemented to protect data from unauthorized access and cyberattacks.

### **1. SSL/TLS for Secure Communication:**  
**Why SSL/TLS?**  
- Ensures all communication between the client and server is encrypted.  
- Prevents man-in-the-middle (MITM) attacks.  
- Protects sensitive data (passwords, tokens) during transmission.

### **2. Input Validation and Sanitization:**  
Prevented common vulnerabilities like **SQL injection**, **cross-site scripting (XSS)**, and **cross-site request forgery (CSRF)** by sanitizing user inputs and using prepared statements for database queries.

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

## **Project Outcomes and Results**  
1. **High-Security Standards:** Achieved **100% compliance with DRDO’s security protocols** by implementing encryption, access control, and monitoring.  
2. **Improved Data Sharing:** The platform enabled secure and efficient file sharing among research teams, reducing manual processes and errors.  
3. **Scalable and Robust Architecture:** Designed for scalability, ensuring future expansion and integration with other DRDO systems.  

---

## **Why This Architecture Worked Best:**  
- **Scalable and Secure:** PostgreSQL and Flask provided a strong foundation for managing sensitive data.  
- **High Performance:** ReactJS ensured a responsive UI, while Docker and Jenkins improved deployment speed and reliability.  
- **Multi-Layered Security:** The combination of encryption, OAuth2 authentication, and MFA ensured comprehensive protection against cyber threats.

---

### **Future Enhancements:**  
1. **Integration with Cloud Storage Solutions:** To improve file availability and redundancy.  
2. **AI-based Anomaly Detection:** Automatically flag unusual user activities for security analysis.  
3. **Support for More File Formats and Large Data Sets.**
