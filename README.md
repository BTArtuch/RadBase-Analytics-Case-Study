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

### 1. Database Layer & Clinical Schema (Neon PostgreSQL)
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
