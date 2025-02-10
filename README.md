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
*Situation:*
 "As part of my role as a Research Assistant at the HIRO Lab, University of Colorado Boulder, I was working on improving the spatial perception capabilities of low-SWaP robots—those that operate under strict constraints in size, weight, and power. Typically, achieving high-quality 3D perception for these robots is a challenge because most neural networks are too computationally intensive to run in real-time on embedded systems. Imagine trying to use a powerful desktop-class neural network on a tiny device running on battery—it just isn’t feasible."
*Task:*
 "My task was to develop a lightweight neural network that could provide dense spatial perception in real-time on embedded devices. Specifically, this involved creating a model that could perform image segmentation (labeling pixels as background, human, or object) while simultaneously refining sensor-generated depth data to produce a more accurate 3D map of the scene. This is particularly useful in scenarios like human-robot handovers, where the robot needs a precise understanding of where the person and object are in 3D space to avoid errors."
*Action:*
 "To tackle this, I developed Bilateral Segmentation and Disparity Refinement (BSDR)—a unique neural network that performs two tasks simultaneously: segmentation and depth refinement. Here’s how it works:
It takes a monocular image and a raw disparity map as input. The model’s architecture is inspired by BiSeNetv1, but I added a second head to refine depth data alongside segmentation.
We optimized the architecture by reducing the number of parameters and using a multi-layer perceptron (MLP) for sub-pixel disparity prediction. This reduces computational overhead while preserving accuracy.
I also curated a custom dataset for training using CREStereo, a highly accurate but computationally heavy depth estimation model. This gave us precise ground truth depth data, which helped train BSDR to improve on noisy sensor data."
"For example, the OAK camera’s standard depth map had holes and noise, especially around occluded regions like a person’s hand holding an object. BSDR not only smoothed the noisy data but also filled in missing depth information, making the 3D reconstruction much more reliable. I ran BSDR on a Luxonis OAK camera at 11 FPS, outperforming similar models that couldn’t achieve real-time performance."
*Result:*
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
A high-performance **Big Data pipeline** designed to process real-time stock-related tweets and analyze their sentiment to predict stock trends for **NYSE and NASDAQ** companies.

**Key Technologies:**
- **Kafka, PySpark, MongoDB, AWS**
- Sentiment analysis using **NLTK** and **spaCy**
- **<500ms processing latency**, handling 1000 tweets per hour

---

## 3. Movie Recommendation System
**Project Overview:**  
A **collaborative filtering-based recommendation system** for movies, designed using **FastAPI, ReactJS, and MongoDB**.

**Key Technologies and Features:**
- Dockerized for **expedited deployment** (25% reduction in deployment time)
- Scalable using **Google Cloud Platform (GCP)** to handle **35% surge in concurrent users**

---

## 4. Automatic Hate Speech Detection Using Ensemble Methods
**Project Overview:**  
Developed an efficient social media **hate speech detection system** that classifies multi-class categories like **hate, offensive, and non-hate** texts using **Random Forest and Support Vector Classifiers (SVM)**.

**Key Results:**
- **90.7% accuracy**, outperforming traditional methods by **9.3%**
- Published at **IEEE NMITCON 2023**

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
**Duration:** Aug 2021 – Dec 2021  
**Location:** Bangalore, India  

#### **Key Contributions:**  
- Built a secure data-sharing platform with **SSL/TLS encryption, OAuth2, and JWT authentication**  
- Developed responsive interfaces using **React and Bootstrap**, improving user experience by 30%  
- Streamlined deployments with **Jenkins CI/CD pipelines** following Test-Driven Development practices  
