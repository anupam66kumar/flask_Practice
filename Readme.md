**TASK 1: Flask CI/CD Pipeline with Jenkins & MongoDB Integration**

This repository contains a simple Flask web application (Student Registration System) integrated with a complete CI/CD pipeline using Jenkins, automated testing via pytest, and local MongoDB database support on an Ubuntu environment.

### **Prerequisites & Tech Stack**

*   **Operating System**: Ubuntu
    
*   **Backend**: Python 3.12+, Flask, Flask-PyMongo
    
*   **Database**: MongoDB (Local instance)
    
*   **CI/CD Tool**: Jenkins (configured with Pipeline and Git plugins)
    
*   **Testing Framework**: pytest
    

### **Setup Instructions**

1.  Bash    git clone https://github.com/anupam66kumar/flask\_Practice.git
            cd flask\_Practice
    
3.  Bash    python3 -m venv venvsource venv/bin/activate
    
4.  Bash    pip install --upgrade pippip install -r requirements.txt
    
5.  Create a .env file in the root project directory: Code snippetMONGO\_URI=mongodb://127.0.0.1:27017/students\_dbSECRET\_KEY=your\_secret\_key
    
6.  Ensure the local MongoDB service is active: Bashsudo systemctl start mongodsudo systemctl status mongod
    
7.  Bash    python app.py    Open your browser at http://localhost:5000.
    

### **Jenkins CI/CD Pipeline Configuration**

The automated pipeline is defined using a declarative Jenkinsfile stored in the root directory of the repository. It orchestrates three continuous deployment stages:

1.  **Build Stage**:
    
    *   Sets up an isolated Python virtual environment (python3 -m venv venv).
        
    *   Upgrades pip and installs all project requirements from requirements.txt.
        
2.  **Test Stage**:
    
    *   Activates the virtual environment and executes automated unit and integration tests using pytest against the local test database instance.
        
3.  **Deploy Stage**:
    
    *   Safely terminates any stale instances owned by the execution user.
        
    *   Spawns a background process running the Flask application via nohup on port 5000.
        

#### **Automated Triggers & Webhook Configuration**

To enable continuous integration on every code change:

*   **GitHub Webhooks**: Configured using a public tunneling tool (ngrok) pointing to the local Jenkins instance port (http:///github-webhook/) to capture push events instantly.
    
*   **Jenkins Triggers**: Enabled **GitHub hook trigger for GITScm polling** under the job configuration so that pushing updates to the main branch automatically kicks off a new build.
    

### **Challenges Faced & Respective Resolutions**

#### **1\. PyMongo Initialization Error During Test Collection**

*   **Challenge**: When running tests or importing app.py, pytest threw a ValueError: You must specify a URI or set the MONGO\_URI Flask config variable.
    
*   Pythonapp.config\["MONGO\_URI"\] = os.getenv("MONGO\_URI", "mongodb://127.0.0.1:27017/students\_db")
    

#### 2\. MongoDB Connection Refused Error (\[Errno 111\])

*   **Challenge**: PyTest failed during fixture setups with ServerSelectionTimeoutError because connection was refused on port 27017.
    
*   **Resolution**: The background mongod service was inactive or had configuration syntax errors. Started the service using sudo systemctl start mongod after cleaning up any formatting errors in /etc/mongod.conf.
    

#### **3\. YAML Configuration Parsing Errors in MongoDB**

*   **Challenge**: The MongoDB service failed to start with an Error parsing YAML config exit code.
    
*   YAMLstorage: dbPath: /var/lib/mongodb journal: enabled: truesystemLog: destination: file logAppend: true path: /var/log/mongodb/mongod.lognet: port: 27017 bindIp: 127.0.0.1
    

#### 4\. Jenkins User Permission Conflicts During Deployment (pkill failure)

*   **Challenge**: During the Deploy stage of the pipeline, pkill failed with Operation not permitted because Jenkins lacked permission to kill processes belonging to the primary user account (anupam).
    
*   **Resolution**: Scoped the command to target only processes owned by the specific jenkins user: pkill -u jenkins -f app.py || true.

--------------------------------------------------------------------------------------------------------------------------------------------

**TASK 2: Flask CI/CD Pipeline with GitHub Actions & MongoDB Integration**

This repository contains a Flask web application integrated with a complete CI/CD pipeline using GitHub Actions, automated testing via pytest, and an automated MongoDB service container.

### **Prerequisites & Tech Stack**

*   **Operating System**: Ubuntu / GitHub Actions Runners (ubuntu-latest)
    
*   **Backend**: Python 3.12+, Flask, Flask-PyMongo
    
*   **Database**: MongoDB (Integrated Service Container)
    
*   **CI/CD Tool**: GitHub Actions (Workflows)
    
*   **Testing Framework**: pytest
    

### **Steps Performed in This Project**

1.  **Repository and Branch Structure Setup**
    
    *   Configured the GitHub repository (flask\_Practice) with a dual-branch development strategy featuring a **main** branch for production and a **staging** branch for pre-production integration.
        
2.  **GitHub Actions Workflow Configuration**
    
    *   Created the required directory structure (.github/workflows/) and workflow definition file (ci-cd.yml) across both branches.
        
    *   Configured workflow triggers to handle code pushes to the main and staging branches, as well as published release events.
        
    *   Integrated a MongoDB service container (mongo:latest) running on port 27017 to enable isolated database testing environments within GitHub Actions.
        
3.  **Pipeline Stages Implementation**
    
    *   **Build and Test**: Configured the runner to set up Python 3.12, create a virtual environment, upgrade pip, install dependencies from requirements.txt, and execute pytest against the automated test database URI (mongodb://127.0.0.1:27017/test\_student\_db).
        
    *   **Deploy to Staging**: Added a conditional deployment job executing automatically upon pushes to the staging branch.
        
    *   **Deploy to Production**: Added a conditional deployment job triggered by published GitHub release tags linked to the main branch.
        
4.  **Environment Secrets Setup**
    
    *   Configured required repository security secrets under GitHub settings (**Settings > Secrets and variables > Actions**) to securely manage deployment parameters (STAGING\_SERVER\_IP, DEPLOY\_KEY, PROD\_SERVER\_IP, and PROD\_API\_TOKEN).
        
5.  **Release Tagging and Execution**
    
    *   Created and published valid semantic release tags targeting the main branch to successfully trigger, verify, and complete the production deployment workflow run.
        

### **Required Repository Secrets**

To configure deployments securely, add the following secrets under your repository settings (**Settings > Secrets and variables > Actions**):

*   STAGING\_SERVER\_IP: Target host address for your staging environment deployment.
    
*   DEPLOY\_KEY: Private SSH credentials for secure remote server authentication.
    
*   PROD\_SERVER\_IP: Target host address for your production environment deployment.
    
*   PROD\_API\_TOKEN: Authentication token for production publishing or target external services.
