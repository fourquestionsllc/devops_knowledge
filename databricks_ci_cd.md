# CI/CD pipeline for Databricks

Creating a CI/CD pipeline for Databricks involves automating the deployment of notebooks, jobs, and configurations from a version control system (e.g., GitHub) to your Databricks workspace. Below is a complete example of setting up CI/CD for Databricks using **GitHub Actions**.

---

## **Step 1: Setup the Environment**

### **1.1 Prerequisites**
- A Databricks workspace.
- GitHub repository for your code (notebooks, configuration files).
- Databricks CLI installed locally (for manual testing).
- A Databricks Personal Access Token (PAT).

### **1.2. Install Databricks CLI**
Install the Databricks CLI on your local machine:

```bash
pip install databricks-cli
```

Configure the CLI with your Databricks workspace:

```bash
databricks configure --token
```

Provide:
- **Host**: `https://<your-databricks-instance>.cloud.databricks.com`
- **Token**: Your Databricks PAT.

Test the setup:

```bash
databricks workspace list /
```

---

## **Step 2: Create a Databricks Notebook**

For this example, let's create a notebook `example_notebook.py`:

```python
# example_notebook.py
# Databricks notebook source
dbutils.widgets.text("input", "default", "Enter a value")

# COMMAND ----------
input_value = dbutils.widgets.get("input")
print(f"Hello, {input_value}!")
```

---

## **Step 3: Create a Deployment Script**

Create a Python script `deploy.py` to deploy the notebook to Databricks.

```python
import os
import subprocess

# Databricks workspace and file paths
DATABRICKS_WORKSPACE_PATH = "/Shared/example_notebook"
LOCAL_NOTEBOOK_PATH = "example_notebook.py"

def deploy_notebook():
    """
    Deploy the notebook to Databricks workspace.
    """
    try:
        # Remove existing notebook (if any)
        subprocess.run(
            ["databricks", "workspace", "rm", DATABRICKS_WORKSPACE_PATH, "--recursive"],
            check=False,
        )
        # Import the new notebook
        subprocess.run(
            [
                "databricks",
                "workspace",
                "import",
                LOCAL_NOTEBOOK_PATH,
                DATABRICKS_WORKSPACE_PATH,
                "--language",
                "PYTHON",
            ],
            check=True,
        )
        print(f"Notebook deployed to {DATABRICKS_WORKSPACE_PATH}")
    except Exception as e:
        print(f"Failed to deploy notebook: {e}")

if __name__ == "__main__":
    deploy_notebook()
```

---

## **Step 4: Configure CI/CD with GitHub Actions**

### **4.1. Create a Workflow File**
Create a GitHub Actions workflow file `.github/workflows/databricks-ci-cd.yml`:

```yaml
name: Databricks CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Setup Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.8'

    - name: Install Dependencies
      run: |
        pip install databricks-cli

    - name: Configure Databricks CLI
      env:
        DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
        DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
      run: |
        databricks configure --token <<< "${DATABRICKS_HOST}\n${DATABRICKS_TOKEN}"

    - name: Deploy Notebook
      run: python deploy.py
```

### **4.2. Add Secrets to GitHub**
1. Go to your GitHub repository settings.
2. Navigate to **Secrets and variables > Actions**.
3. Add the following secrets:
   - `DATABRICKS_HOST`: Your Databricks workspace URL (`https://<your-databricks-instance>.cloud.databricks.com`).
   - `DATABRICKS_TOKEN`: Your Databricks PAT.

---

## **Step 5: Testing the Pipeline**

1. Commit your code and push it to the `main` branch:

   ```bash
   git add .
   git commit -m "Add CI/CD pipeline for Databricks"
   git push origin main
   ```

2. Go to your GitHub repository and check the **Actions** tab. You should see your workflow running.

3. Verify the notebook deployment in your Databricks workspace under `/Shared/example_notebook`.

---

## **Step 6: Run the Notebook on Databricks**

To automate running the notebook after deployment:

### **6.1. Update the Deployment Script**
Modify `deploy.py` to also run the notebook as a Databricks job.

```python
import requests
import json

# Add these configurations
DATABRICKS_HOST = os.getenv("DATABRICKS_HOST")
DATABRICKS_TOKEN = os.getenv("DATABRICKS_TOKEN")
JOB_NAME = "Example Job"

def create_and_run_job():
    """
    Create and run a Databricks job to execute the notebook.
    """
    headers = {"Authorization": f"Bearer {DATABRICKS_TOKEN}"}
    job_config = {
        "name": JOB_NAME,
        "new_cluster": {
            "spark_version": "11.3.x-scala2.12",
            "node_type_id": "i3.xlarge",
            "num_workers": 1,
        },
        "notebook_task": {"notebook_path": DATABRICKS_WORKSPACE_PATH},
    }
    # Create the job
    response = requests.post(
        f"{DATABRICKS_HOST}/api/2.1/jobs/create",
        headers=headers,
        data=json.dumps(job_config),
    )
    response.raise_for_status()
    job_id = response.json().get("job_id")
    print(f"Job {JOB_NAME} created with ID {job_id}")

    # Run the job
    run_response = requests.post(
        f"{DATABRICKS_HOST}/api/2.1/jobs/run-now",
        headers=headers,
        data=json.dumps({"job_id": job_id}),
    )
    run_response.raise_for_status()
    print(f"Job {JOB_NAME} started: {run_response.json()}")

if __name__ == "__main__":
    deploy_notebook()
    create_and_run_job()
```

---

### **6.2. Update GitHub Actions Workflow**

Replace the `Deploy Notebook` step in the workflow with:

```yaml
- name: Deploy and Run Notebook
  run: python deploy.py
```

---

## **Step 7: Verify the CI/CD Pipeline**

Push the updated pipeline, and check:
1. **Deployment:** Notebook is deployed in Databricks.
2. **Job Execution:** The notebook runs successfully as a job.

---

This pipeline ensures that changes to your code in GitHub automatically deploy to Databricks and execute. You can expand it to include additional stages, like testing or notifying via Slack.
