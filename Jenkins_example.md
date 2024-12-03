Here’s a comprehensive end-to-end Jenkins example to set up a CI/CD pipeline from scratch:

---

### **Project Setup**
- **Objective**: Deploy a simple Python Flask application to a staging server.
- **Stack**: Jenkins, Git, Docker, Flask, and a sample AWS EC2 instance for deployment.
  
---

### **Steps**

#### **1. Install Jenkins**
1. Install Jenkins on your server (AWS EC2, local VM, etc.).
    ```bash
    sudo apt update
    sudo apt install -y openjdk-11-jdk
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt install -y jenkins
    sudo systemctl start jenkins
    ```
2. Access Jenkins at `http://<your_server_ip>:8080`.

---

#### **2. Setup Git Repository**
- Create a new repository, e.g., `flask-ci-cd`.
- Add the sample Flask app to your repository:

    **`app.py`**:
    ```python
    from flask import Flask

    app = Flask(__name__)

    @app.route('/')
    def home():
        return "Hello, CI/CD with Jenkins!"

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5000)
    ```

    **`requirements.txt`**:
    ```
    flask==2.0.3
    ```

    **`Dockerfile`**:
    ```dockerfile
    FROM python:3.9-slim
    WORKDIR /app
    COPY . /app
    RUN pip install -r requirements.txt
    CMD ["python", "app.py"]
    ```

    Push the repository to GitHub or your preferred Git service.

---

#### **3. Configure Jenkins**
1. **Install Plugins**:
   - Navigate to `Manage Jenkins` > `Plugin Manager`.
   - Install the following plugins:
     - Git
     - Pipeline
     - Docker Pipeline

2. **Create a Jenkins Pipeline Job**:
   - Go to `New Item` > Choose `Pipeline` > Name it `flask-ci-cd`.

---

#### **4. Jenkinsfile for CI/CD**
Create a `Jenkinsfile` in your repository root:

**`Jenkinsfile`**:
```groovy
pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/<your-username>/flask-ci-cd.git'
        DOCKER_IMAGE = 'yourdockerhub/flask-app'
        DEPLOY_SERVER = 'ec2-user@<your-server-ip>'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git REPO_URL
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sshagent(['deploy-key']) {
                        sh '''
                            ssh ${DEPLOY_SERVER} "docker pull ${DOCKER_IMAGE}:latest && docker run -d -p 5000:5000 ${DOCKER_IMAGE}:latest"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
```

---

#### **5. Setup Jenkins Credentials**
1. **GitHub Credentials**: Add your GitHub credentials (if private repository).
   - Go to `Manage Jenkins` > `Credentials` > `System` > `Add Credentials`.
   - Add your GitHub username and personal access token.

2. **Docker Hub Credentials**:
   - Add your Docker Hub username and password.

3. **SSH Key for Deployment**:
   - Generate an SSH key on Jenkins.
     ```bash
     ssh-keygen -t rsa -b 2048 -f ~/.ssh/deploy-key
     ```
   - Copy the public key to the staging server's `~/.ssh/authorized_keys`.

---

#### **6. Execute the Pipeline**
1. Push your `Jenkinsfile` to the GitHub repository.
2. In Jenkins, configure the pipeline job to pull from your repository.
3. Trigger the build.

---

#### **Actions in Each Stage**
- **Clone Repository**: Pulls the latest code from the Git repository.
- **Build Docker Image**: Builds the Docker image for the Flask app.
- **Push Docker Image**: Pushes the built image to Docker Hub.
- **Deploy to Staging**:
  - SSH into the staging server.
  - Pull the latest Docker image.
  - Deploy the container on the server.

---

#### **7. Validation**
- Access the app at `http://<staging-server-ip>:5000`.
- The page should display: **"Hello, CI/CD with Jenkins!"**

---

This setup can be further enhanced with error handling, automated testing, and monitoring. 
