# FlowiseApp

This is the central repository for building and deploying the AI agent Web App.

## Goals

- Host and version backend + frontend code
- Automate build, test, and deployment using GitHub Actions
- Replace manual AWS EC2/Docker workflows with a robust CI/CD pipeline

---

## Project Status

- [X] Create repo
- [ ] Add backend components
- [ ] Add frontend code
- [ ] Implement user authentication and role-based model access

---

## CI/CD Pipeline (GitHub Actions)

### 1. Configure AWS Credentials

Use [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials) to allow GitHub Actions to interact with AWS.

````yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-2
````
> Use GitHub Secrets to store variables 


### 2. Build and Push Docker Image to ECR

Build the Docker image and push it to Amazon ECR:

````yaml
- name: Log in to Amazon ECR
  uses: aws-actions/amazon-ecr-login@v1

- name: Build, tag, and push Docker image
  run: |
    IMAGE_URI="$ECR_REGISTRY/$REPO:$GITHUB_SHA"
    docker build -t $IMAGE_URI .
    docker push $IMAGE_URI
````

> Replace `$ECR_REGISTRY` and `$REPO` with your actual values or GitHub Action environment variables.

### 3. Deploy to EC2 via SSH

Use [appleboy/ssh-action](https://github.com/appleboy/ssh-action) to remotely deploy your container to EC2:

````yaml
- name: Deploy to EC2
  uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ${{ secrets.EC2_USER }}
    key: ${{ secrets.EC2_SSH_KEY }}
    script: |
      cd /home/ubuntu/project-name
      git pull origin main
      docker compose down
      docker compose up -d --build
````
> Use GitHub Secrets to store these variables as well 

---

## Security & Best Practices

### Lint Dockerfiles

Use [hadolint](https://github.com/hadolint/hadolint-action):

````yaml
- name: Lint Dockerfile
  uses: hadolint/hadolint-action@v3.1.0
````

### Scan Docker Images for Vulnerabilities

Use [Trivy](https://github.com/aquasecurity/trivy-action):

````yaml
- name: Scan Docker image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ env.IMAGE_URI }}
````

---

## Why GitHub Actions?

-  Native integration with codebase
-  Full CI/CD lifecycle management
-  Secure secrets management
-  Built-in logs and status checks

---

## Next Steps for CI/CD

- [ ] Define Dockerfile and `docker-compose.yml`
- [ ] Set up GitHub Actions workflow YAML files in `.github/workflows/`
- [ ] Write setup documentation 

---
  
