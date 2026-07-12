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

### 1. The Container Strategy (Docker & WSL2)
To bypass cross-platform operating system dependencies, the entire analytical system runtime is completely containerized. By standardizing the environment via a custom `Dockerfile` mapped across Windows Subsystem for Linux (WSL2), local development parity perfectly mirrors the production cloud runtime.

```dockerfile
# Production-ready environmental baseline
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8501
CMD ["streamlit", "run", "app.py"]
