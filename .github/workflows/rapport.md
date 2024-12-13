# Comprehensive Report: CI/CD Pipeline Implementation

This document provides a detailed overview of the CI/CD pipeline implementation for a software application using GitHub Actions. The pipeline includes steps for building, testing, Dockerizing, and analyzing code quality. Each step is accompanied by critical explanations and insights.

---

## **1. Step 1: Basic CI Setup for Backend Testing**

### **Objective**
To set up a foundational Continuous Integration (CI) pipeline that builds and tests the application automatically upon code changes.

### **Implementation**

#### **Triggering the Workflow**
- The pipeline is triggered on:
  - **Push events** to `main` and `develop` branches.
  - **Pull request events**, ensuring testing before merging code changes.

#### **Key Workflow Steps**
1. **Code Checkout**:
   - Uses `actions/checkout@v2.5.0` to pull the repository's latest code.
2. **Set Up JDK 17**:
   - Configures the Java Development Kit using `actions/setup-java@v3`.
   - Ensures compatibility with the application’s dependencies and build tools.
3. **Build and Test**:
   - Executes `mvn clean verify` to:
     - Clear cache of previous builds.
     - Build the application from scratch.
     - Run all unit and integration tests.

#### **Code Snippet**
```yaml
name: CI devops 2024
on:
  push:
    branches:
      - main
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: "17"
      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: simple-api
```

### **Outcome**
- The workflow successfully validated that the code builds and all tests pass on every push or pull request.
- Ensured faster feedback for developers about potential issues.

---

## **2. Step 2: Adding Docker Image Building**

### **Objective**
To extend the pipeline by building Docker images for the backend, database, and HTTP server, ensuring the application’s components can be containerized.

### **Implementation**

#### **Enhancements to the Workflow**
1. **Job Dependency**:
   - Introduced a new job `build-docker-images` that depends on the `test-backend` job.
   - Ensures images are built only if all tests pass.

2. **Docker Image Building**:
   - Used `docker/build-push-action@v3` to build Docker images for:
     - Backend (`./simple-api` directory).
     - Database (`./database` directory).
     - HTTP server (`./http-server` directory).
   - Tagged images with `latest` for ease of versioning.

#### **Code Snippet**
```yaml
  build-docker-images:
    needs: test-backend
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      - name: Build backend image
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          outputs: type=docker
          tags: backend:latest
      - name: Build database image
        uses: docker/build-push-action@v3
        with:
          context: ./database
          outputs: type=docker
          tags: database:latest
```

### **Outcome**
- Docker images for the application’s components were successfully built after tests passed.
- Created a robust base for deployment.

---

## **3. Step 3: Pushing Docker Images to DockerHub**

### **Objective**
To store the built Docker images on DockerHub for easy access and deployment.

### **Implementation**

#### **Authentication**
- Integrated DockerHub login using credentials stored in GitHub secrets (`DOCKER_USERNAME` and `DOCKER_PASSWORD`).

#### **Push Logic**
- Updated the workflow to push images only when the `main` branch is updated.
- Tagged images appropriately (e.g., `backend:latest`).

#### **Code Snippet**
```yaml
      - name: Login to DockerHub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./simple-api
          tags: ${{ secrets.DOCKER_USERNAME }}/backend:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```

### **Outcome**
- Successfully pushed images for the backend, database, and HTTP server to DockerHub.
- Enabled seamless deployment across environments.

---

## **4. Step 4: Integrating SonarCloud for Quality Gates**

### **Objective**
To integrate SonarCloud for automated code quality analysis.

### **Implementation**

#### **SonarCloud Setup**
1. **Authentication**:
   - Configured `SONAR_TOKEN` as a GitHub secret.
2. **Quality Gate Analysis**:
   - Added a Maven command for SonarCloud analysis:
     ```bash
     mvn sonar:sonar -Dsonar.projectKey=albiche_tp_devops -Dsonar.organization=albiche -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}
     ```
3. **Caching**:
   - Cached Maven dependencies using `actions/cache@v3` to optimize performance.

#### **Code Snippet**
```yaml
      - name: SonarCloud Analysis
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn sonar:sonar -Dsonar.projectKey=albiche_tp_devops -Dsonar.organization=albiche -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}
        working-directory: simple-api
```

### **Outcome**
- Real-time feedback on code quality, enabling better maintainability and reducing technical debt.

---

## **5. Step 5: Splitting Workflows**

### **Objective**
To split the workflows for modularity and better execution control.

### **Implementation**

#### **Workflow 1: Test Backend**
- Runs tests on `main` and `develop` branches.

#### **Workflow 2: Build Docker Images**
- Triggers only after `test-backend` passes.
- Pushes images to DockerHub for `main` branch updates.

#### **Code Snippet for Trigger**
```yaml
on:
  workflow_run:
    workflows:
      - Test Backend
    types:
      - completed
```

### **Outcome**
- Achieved modularity by splitting responsibilities across workflows.
- Improved clarity and debugging.

---

## **Conclusion**

### **Key Achievements**
1. **Continuous Integration**:
   - Automatic build, test, and code quality analysis.
   - Faster feedback for developers.
2. **Continuous Delivery**:
   - Built and pushed Docker images for application components.
   - Enabled seamless deployments using DockerHub.
3. **Best Practices**:
   - Secrets management for sensitive data.
   - Workflow modularity for better scalability.

This pipeline provides a robust foundation for modern software development and deployment workflows.

