FLASK STUDENT REGISTRATION SYSTEM
CI/CD DEPLOYMENT & ASSIGNMENT DOCUMENTATION

============================================================
1. PROJECT OVERVIEW
============================================================

A Flask-based Student Registration System using MongoDB as the backend database.

The application supports:
- Viewing students
- Adding students
- Updating students
- Deleting students
- MongoDB database integration
- Health monitoring through /health

The application is containerized using Docker and deployed to AWS EC2 through an automated GitHub Actions CI/CD pipeline.

============================================================
2. TECHNOLOGY STACK
============================================================

Backend:
- Python
- Flask
- Flask-PyMongo

Database:
- MongoDB / MongoDB Atlas

Frontend:
- HTML
- Jinja2
- Bootstrap 5

Testing:
- Pytest

Containerization:
- Docker

Cloud:
- AWS EC2
- Amazon ECR
- AWS IAM

CI/CD:
- GitHub Actions

Deployment:
- SSH from GitHub Actions to EC2

Notifications:
- SMTP / Gmail

AWS Region:
- ap-south-1


============================================================
3. APPLICATION FEATURES
============================================================

- List all students on the home page
- Add a new student
- Update existing student details
- Delete student records
- MongoDB integration
- Flask health endpoint
- Docker containerization
- Automated CI/CD
- Amazon ECR image storage
- Automated EC2 deployment
- Git commit SHA Docker image tagging
- Automated deployment health check
- Success email notification
- Failure email notification
- Test-gated deployment


============================================================
4. PROJECT STRUCTURE
============================================================

flask_Practice/
|
|-- .github/
|   |-- workflows/
|       |-- ci-cd.yml
|       |-- securegate.yml
|       |-- securegate-summary.yaml
|
|-- templates/
|   |-- base.html
|   |-- index.html
|   |-- add_student.html
|   |-- update_student.html
|
|-- app.py
|-- test_app.py
|-- requirements.txt
|-- Dockerfile
|-- .dockerignore
|-- .gitignore
|-- .env.example
|-- README.md
|-- start_flask.sh


============================================================
5. LOCAL DEVELOPMENT SETUP
============================================================

Clone the repository:

git clone <repository-url>
cd flask_Practice

Create a virtual environment.

Windows:

python -m venv venv
venv\Scripts\activate

Linux/macOS:

python3 -m venv venv
source venv/bin/activate

Install dependencies:

pip install -r requirements.txt

Create a .env file:

MONGO_URI=<your-mongodb-connection-string>
SECRET_KEY=<your-secret-key>

Run the application:

python app.py

Application:

http://localhost:5000


============================================================
6. ENVIRONMENT VARIABLES
============================================================

Local application variables:

MONGO_URI
    MongoDB connection string.

SECRET_KEY
    Flask application secret.

The .env file is excluded from Git.

For CI/CD, sensitive values are stored as GitHub Repository
Secrets instead of being committed to the repository.


============================================================
7. TESTING
============================================================

The project uses Pytest.

Run tests locally:

pytest -v

Tests cover:

- Home page
- Add student
- Update student
- Delete student
- Health endpoint

The GitHub Actions pipeline executes tests before Docker
build and deployment.

If tests fail, the Docker build and deployment stages do not
execute.


============================================================
8. DOCKER
============================================================

Dockerfile uses:

python:3.12-slim

Build locally:

docker build -t flask-practice .

Run locally:

docker run -d \
  --name flask-practice \
  -p 5000:5000 \
  -e MONGO_URI="<your-mongodb-uri>" \
  -e SECRET_KEY="<your-secret-key>" \
  flask-practice

Check:

docker ps

Health check:

curl http://localhost:5000/health

Expected response:

{
  "status": "healthy",
  "database": "connected"
}


============================================================
9. AWS INFRASTRUCTURE
============================================================

AWS services used:

- AWS IAM
- Amazon ECR
- Amazon EC2

Architecture:

GitHub Repository
       |
       v
GitHub Actions
       |
       v
Docker Build
       |
       v
Amazon ECR
       |
       v
SSH
       |
       v
Amazon EC2
       |
       v
Docker Container
       |
       v
Flask Application
       |
       v
MongoDB


AWS Region:

ap-south-1


============================================================
10. AMAZON ECR
============================================================

Repository:

flask-practice

Registry format:

<account-id>.dkr.ecr.ap-south-1.amazonaws.com/flask-practice

Docker images are tagged using the Git commit SHA.

Example:

flask-practice:9a28593fbebcee15c692af84b8823ad2c1e23895

Using commit SHA tags provides traceability between a deployed
Docker image and the source-code commit that produced it.


============================================================
11. EC2 CONFIGURATION
============================================================

The application is deployed to an Ubuntu EC2 instance.

Docker is installed on the EC2 instance.

The EC2 instance uses an IAM role for AWS access.

Application port:

5000

Security Group:

SSH:
TCP 22

Application:
TCP 5000

GitHub Actions connects to EC2 through SSH for deployment.


============================================================
12. GITHUB ACTIONS CI/CD
============================================================

Workflow:

.github/workflows/ci-cd.yml

Trigger:

Push to main branch.

Pipeline:

Git Push
   |
   v
Checkout Source
   |
   v
Install Python Dependencies
   |
   v
Run Pytest
   |
   | PASS
   v
Build Docker Image
   |
   v
Tag Image With Git Commit SHA
   |
   v
Push Image To Amazon ECR
   |
   v
SSH Into EC2
   |
   v
Login To ECR
   |
   v
Pull New Image
   |
   v
Stop Existing Container
   |
   v
Remove Existing Container
   |
   v
Start New Container
   |
   v
Health Check
   |
   | PASS
   v
Email Notification


============================================================
13. PIPELINE STAGE 1 — TEST
============================================================

GitHub Actions:

1. Checks out the source code.
2. Installs Python 3.12.
3. Installs dependencies.
4. Runs Pytest.

Command:

pytest -v

This stage acts as the deployment gate.

If tests fail, subsequent build and deployment stages are
skipped.


============================================================
14. PIPELINE STAGE 2 — BUILD AND PUSH TO ECR
============================================================

After successful tests:

1. AWS credentials are configured.
2. GitHub Actions logs into Amazon ECR.
3. Docker image is built.
4. Image is tagged using the Git commit SHA.
5. Image is pushed to ECR.

Image format:

<registry>/<repository>:<commit-sha>


============================================================
15. PIPELINE STAGE 3 — DEPLOY TO EC2
============================================================

GitHub Actions connects to EC2 using SSH.

The deployment process:

1. Login to ECR.
2. Pull the new SHA-tagged image.
3. Stop the existing Flask container.
4. Remove the existing container.
5. Start the new container.
6. Perform a health check.

Pull:

docker pull <image>

Stop:

docker stop flask-practice

Remove:

docker rm flask-practice

Run:

docker run -d \
  --name flask-practice \
  -p 5000:5000 \
  --restart unless-stopped \
  -e MONGO_URI="$MONGO_URI" \
  -e SECRET_KEY="$SECRET_KEY" \
  <image>


============================================================
16. HEALTH CHECK
============================================================

After deployment, GitHub Actions checks:

http://localhost:5000/health

Expected:

{
  "status": "healthy",
  "database": "connected"
}

The health check confirms:

- Flask application is running.
- MongoDB connection is working.

If the health check fails, the deployment fails.


============================================================
17. GITHUB REPOSITORY SECRETS
============================================================

AWS / ECR:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
ECR_REPOSITORY

EC2:

EC2_HOST
EC2_USER
EC2_SSH_KEY

Application:

MONGO_URI
SECRET_KEY

Email:

SMTP_HOST
SMTP_PORT
SMTP_USERNAME
SMTP_PASSWORD
MAIL_TO
MAIL_FROM

Secret values are not committed to Git.


============================================================
18. EMAIL NOTIFICATIONS
============================================================

The pipeline sends an email after execution.

SUCCESS EMAIL

Subject:

[SUCCESS] Flask CI/CD Deployment

Contains:

- Pipeline result
- Commit SHA
- Branch
- Docker image tag
- ECR repository
- EC2 target
- Health endpoint
- GitHub Actions run


FAILURE EMAIL

Subject:

[FAILURE] Flask CI/CD Pipeline

Contains:

- Failed pipeline stage
- Commit SHA
- Branch
- GitHub Actions run
- Repository information

Possible failed stages:

test
build/push
deploy


============================================================
19. FAILURE GATE DEMONSTRATION
============================================================

An intentional test failure was introduced to verify the CI/CD
gate.

Expected behavior:

Test                  FAILED
Build and Push ECR    SKIPPED
Deploy to EC2         SKIPPED
Email Notification    FAILURE

This confirms that the pipeline prevents deployment when
automated tests fail.

After the demonstration, the test was restored and the final
pipeline was executed successfully.


============================================================
20. MANUAL DEPLOYMENT FALLBACK
============================================================

If GitHub Actions is unavailable, deployment can be performed
manually from EC2.

Login to ECR:

aws ecr get-login-password --region ap-south-1 | \
docker login \
  --username AWS \
  --password-stdin \
  <account-id>.dkr.ecr.ap-south-1.amazonaws.com

Pull the image:

docker pull \
  <account-id>.dkr.ecr.ap-south-1.amazonaws.com/flask-practice:<commit-sha>

Stop existing container:

docker stop flask-practice || true
docker rm flask-practice || true

Start new container:

docker run -d \
  --name flask-practice \
  -p 5000:5000 \
  --restart unless-stopped \
  -e MONGO_URI="<your-mongodb-uri>" \
  -e SECRET_KEY="<your-secret-key>" \
  <account-id>.dkr.ecr.ap-south-1.amazonaws.com/flask-practice:<commit-sha>

Verify:

curl http://localhost:5000/health


============================================================
21. DEPLOYMENT VERIFICATION
============================================================

The final deployment was successfully verified.

EC2 container:

docker ps

The deployed container used a SHA-tagged ECR image.

Example:

591667019613.dkr.ecr.ap-south-1.amazonaws.com/flask-practice:
9a28593fbebcee15c692af84b8823ad2c1e23895

Port mapping:

0.0.0.0:5000->5000/tcp

Local EC2 health check:

curl http://localhost:5000/health

Response:

{
  "status": "healthy",
  "database": "connected"
}

External health check:

curl http://<EC2_PUBLIC_IP>:5000/health


============================================================
22. APPLICATION SCREENSHOTS
============================================================

HOME PAGE

Lists students with Edit/Delete buttons.

Existing GitHub screenshot:

https://github.com/user-attachments/assets/a58a6a6d-4978-4769-8074-232e4d31e69d


ADD STUDENT

Form used to add a student.

Existing GitHub screenshot:

https://github.com/user-attachments/assets/d65d25c3-ebb5-410a-adb1-e130ad7c5878


UPDATE STUDENT

Form pre-filled with student information.

Existing GitHub screenshot:

https://github.com/user-attachments/assets/04febf01-879f-431f-ab07-abcfb993acf1


============================================================
23. REQUIRED ASSIGNMENT EVIDENCE
============================================================

The following evidence was captured during implementation:

1. AWS IAM / EC2 configuration
2. EC2 instance information
3. EC2 security group inbound rules
4. AWS STS caller identity
5. ECR authentication
6. ECR repository/image
7. Successful GitHub Actions pipeline
8. Docker build and ECR push
9. Successful EC2 deployment
10. EC2 docker ps output
11. EC2 localhost health check
12. External health check
13. GitHub Secrets configuration (secret names only)
14. Successful email notification
15. Intentional failed test pipeline
16. Skipped build/deployment stages
17. Failure email notification
18. Final restored successful pipeline


============================================================
24. SECURITY CONSIDERATIONS
============================================================

- .env is excluded from Git.
- AWS credentials are stored as GitHub Secrets.
- MongoDB credentials are stored as GitHub Secrets.
- Flask SECRET_KEY is stored as a GitHub Secret.
- EC2 SSH private key is stored as a GitHub Secret.
- Docker images use Git commit SHA tags.
- Sensitive credentials are not hard-coded into the repository.
- EC2 security group controls SSH and application access.


============================================================
25. TROUBLESHOOTING
============================================================

PYTEST FAILS

Verify:

MONGO_URI
SECRET_KEY

exist as GitHub Secrets.

SSH DEPLOYMENT TIMES OUT

Check:

- EC2_HOST
- EC2_USER
- EC2_SSH_KEY
- EC2 Security Group
- TCP port 22

DOCKER CONTAINER RESTARTS

Run:

docker ps -a

Then:

docker logs flask-practice

HEALTH CHECK FAILS

Run:

docker ps -a

docker logs flask-practice

curl http://localhost:5000/health

ECR PULL FAILS

Run:

aws sts get-caller-identity

Verify that the EC2 IAM role has permission to access ECR.


============================================================
26. FINAL CI/CD RESULT
============================================================

The completed pipeline successfully performs:

Git Push
   |
   v
Automated Tests
   |
   v
Docker Build
   |
   v
SHA-Based Image Tag
   |
   v
Amazon ECR Push
   |
   v
SSH Deployment To EC2
   |
   v
Docker Container Replacement
   |
   v
MongoDB Connectivity Check
   |
   v
Health Check
   |
   v
Email Notification

The pipeline was also tested with an intentional test failure
to confirm that failed tests prevent subsequent build and
deployment stages.

The final restored pipeline completed successfully.


============================================================
27. LICENSE
============================================================

MIT License
