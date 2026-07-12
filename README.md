# Case Study: RadBase Cloud Analytics Architecture
**An Enterprise Data Infrastructure & Automation Pipeline**

> **Project Status:** Production Deployed  
> **Target Role Alignment:** Data Infrastructure, Systems Automation, Computer Engineer - Level I  

---

## System Architecture Overview
This project serves as a full-scale deployment case study for a containerized, cloud-hosted analytics environment. It translates local database processing workflows into a production-ready system featuring automatic database replication, strict environment boundary configurations, and zero-downtime integration pipelines.

### The Technology Stack
*   **Database Layer:** Neon Serverless PostgreSQL Cloud Architecture
*   **Local Runtime & Virtualization:** Docker Engine running on a Linux backend (WSL2 Ubuntu)
*   **Infrastructure Hosting:** Render Cloud Platform Web Services
*   **Data Orchestration & Presentation:** Python, Streamlit UI Framework, Plotly Analytics Engines
*   **CI/CD Deployment Matrix:** GitHub Automated Version Control Hooks

---

## Infrastructure Implementation & Logic

### 1. Clinical Dataset: RSNA Pneumonia Detection Challenge
The foundational data for this pipeline is sourced from the **RSNA Pneumonia Detection Challenge** via Kaggle. This enterprise-scale clinical dataset contains complex radiographic metadata representing over 26,000 unique patient imaging records. 

The raw data is ingested via clinical CSV manifests (e.g., `stage_2_train_labels.csv`), which map unique patient IDs to binary diagnostic targets (`0` = Normal, `1` = Lung Opacity). For positive diagnoses, the dataset provides exact spatial bounding-box coordinates (`x`, `y`, `width`, `height`) used to localize the pneumatic opacities within the chest radiographs.

![RSNA Dataset CSV Overview](RSNA_Challenge.png)

### 2. Database Layer & Clinical Schema (Neon PostgreSQL)
To manage the heavy data load of the Kaggle RSNA Pneumonia Detection Challenge, the data layer utilizes a serverless Neon PostgreSQL architecture. The database is strictly normalized to separate static patient demographics from the dynamic diagnostic coordinates (bounding boxes and classification targets), ensuring highly efficient, indexed queries when visualizing data in the dashboard.

### Database Entity-Relationship Diagram

```mermaid
erDiagram
    Patients ||--o{ Scans : "undergoes"
    Equipment ||--o{ Scans : "performs"
    Radiologists ||--o{ Scans : "reviews"
    Scans ||--o{ Diagnoses : "results in"
    
    Patients {
        string patient_id PK
        int age
        string gender
        date registration_date
    }
    
    Equipment {
        string equipment_id PK
        string manufacturer
        string model_name
        string facility_location
    }
    
    Radiologists {
        string radiologist_id PK
        string first_name
        string last_name
        string specialty
    }
    
    Scans {
        string scan_id PK
        string patient_id FK
        string equipment_id FK
        string radiologist_id FK
        timestamp scan_timestamp
        string image_file_path
        int image_width
        int image_height
        boolean quality_flag
    }
    
    Diagnoses {
        int diagnosis_id PK
        string scan_id FK
        string finding_label
        decimal confidence_score
        int bounding_box_x
        int bounding_box_y
        int bounding_box_width
        int bounding_box_height
    }
```
### 3. Containerized Runtime Environment (Docker)
To guarantee environmental parity across local development and cloud production, the entire analytical application is containerized using Docker. This approach isolates the Python runtime, the Streamlit framework, and the PostgreSQL connection drivers, completely eliminating cross-platform dependency issues.

As shown below in the local Docker environment, the application is highly optimized. The running `radbase-dashboard` container operates with minimal overhead (consuming under 50MB of memory) while successfully mapping the internal Streamlit service to local port `8501`.

![Live Docker Container Runtime](Docker_Container.png)

The container is constructed using a stripped-down Python image to keep the deployment package small and fast:

```dockerfile
# Production-ready environmental baseline
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install dependencies efficiently
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application logic
COPY . .

# Expose Streamlit default port
EXPOSE 8501

# Initialize the dashboard
CMD ["streamlit", "run", "app.py"]
```

### 4. Interactive Clinical Dashboard (Streamlit)
The presentation layer of the application is built using Python and Streamlit, serving as the user-facing command center for the clinical data. The dashboard directly queries the PostgreSQL backend to visualize the results of the ETL pipeline, providing administrators with immediate insights into operational trends and data quality.

![Streamlit Clinical Dashboard](Dashboard.png)

**Key Dashboard Features:**
*   **Hardware Diagnostic Tracking:** Aggregates diagnostic outcomes by equipment manufacturer (Philips, Siemens, GE Healthcare) to monitor positivity rates and ensure hardware calibration consistency across the dataset.
*   **Staff Resource Management:** Visualizes radiologist scan volumes distributed across different hospital sectors (ER, North Wing, South Wing). This grouped metric allows administrators to track individual staff workloads and identify facility bottlenecks.
*   **ETL Pipeline Visualization:** Seamlessly translates the highly normalized SQL schema—joining the `Scans`, `Radiologists`, and `Equipment` tables—into instantly readable, high-level business intelligence charts.
*   **Quality Assurance Monitoring:** Continuously audits the database records to ensure data integrity, automatically verifying that all positive pneumonia diagnoses include valid spatial bounding box coordinates.

![Quality Assurance Data Integrity Check](QA_Monitor.png)

### 5. Cloud Deployment & CI/CD Pipeline (Render)
To make the analytics dashboard accessible to external stakeholders, the containerized application is deployed to the cloud using **Render** as a web service. 

Rather than relying on manual uploads, the system utilizes a modern Continuous Integration and Continuous Deployment (CI/CD) pipeline to manage updates. 

**The Deployment Architecture:**
*   **GitHub Integration:** The Render service is securely hooked into the repository's `main` branch. 
*   **Automated Builds:** Whenever new code, updated SQL queries, or UI changes are pushed to GitHub, Render automatically intercepts the web-hook and triggers a new build sequence.
*   **Container Reconstruction:** Render reads the repository's `Dockerfile`, pulls the `python:3.11-slim` image, installs the dependencies, and spins up the container in the cloud—guaranteeing that production behaves exactly like the local development environment.
*   **Zero-Downtime Routing:** Once the container passes health checks, Render routes the exposed `8501` port to a secure HTTPS domain.

**Live Application:** 
The live, continuously updated dashboard can be accessed here: [https://radbase-analytics.onrender.com/](https://radbase-analytics.onrender.com/)
