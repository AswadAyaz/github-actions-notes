# Building and pushing docker image to aws ecr
# Building and pushing Docker image to amazon ecr
so we are going to build docker image and push it into amazon ecr.

Steps:

Create amazon ecr repo

Create IAM Access keys and store it in GitHub Secrets

Create docker file

Create job to build and publish docker image.

#######################################################################

login to you aws account.

Ecr => create a repo => vprofile-appimg => create.

IAM => users => create user => actions-ecr => attach policies => AmazonEC2ContainerRegistryFullAccess => create => go to the user => security credentials => create access keys => CLI => next => create access key.

go to your github repositry => settings => Secret and variables => actions => Manage environment secrets => Create your env => Add environment secret => AWS_ECR_ACCESS_KEY paste access key => make another secret AWS_ECR_SECRET_KEY paste secret key => now we also want a variable for region => add env variable AWS_ECR_SER_REGION paste the region 

Save it all.


#############################################################################

work on docker file.

Docker-files ---> app ---> multistage ---> Dockerfile open it.

FROM maven:3.9.9-eclipse-temurin-21-jammy AS BUILD_IMAGE

WORKDIR /app

COPY . /app 

RUN mvn install

FROM tomcat:10-jdk21

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE /app/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]

###############################################################################
name: CI/CD

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Maven Build
        if: github.ref == 'refs/heads/main'
        run: mvn clean install -DskipTests

      - name: Upload artifact
        if: github.ref == 'refs/heads/main'
        uses: actions/upload-artifact@v7
        with:
          name: vprofile-artifact
          path: target/*.war

      - name: run build on other branches
        if: github.ref != 'refs/heads/main'
        run: echo "Build is only run on main branch. Skipping build for this branch."

      - name: Notify on failure
        if: failure()
        run: echo "Build failed. Please check the logs for details."

  security_scan:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v7

      - name: Security Scan With Trivy
        uses: aquasecurity/trivy-action@v0.36.0
        with:
          scan-type: fs
          scan-ref: .
          format: json
          exit-code: 0
          vuln-type: os,library
          output: trivy-result.json

      - name: Uploading artifact
        uses: actions/upload-artifact@v7
        with:
          name: trivy-scan-result
          path: trivy-result.json

  testing:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Maven test
        if: github.ref == 'refs/heads/main'
        run: mvn test

      - name: Checkstyle tests
        if: github.ref == 'refs/heads/main'
        run: mvn checkstyle:checkstyle

      - name: Run tests on other branches
        if: github.ref != 'refs/heads/main'
        run: echo "Tests are only run on main branch. Skipping tests for this branch."

  build_and_publish:
    runs-on: ubuntu-latest
    needs: ["build", "testing", "security_scan"]
    environment: Production
    if: github.ref == 'refs/heads/main'

    steps:
      - name: code checkout
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: configure AWS creds
        uses: aws-actions/configure-aws-credentials@v6
        with:
          aws-access-key-id: ${{ secrets.AWS_ECR_ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.AWS_ECR_SECRET_KEY }}
          aws-region: ${{ vars.AWS_ECR_SER_REGION }}

      - name: login to ecr
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: build and push docker image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -f Docker-files/app/multistage/Dockerfile -t $ECR_REGISTRY/${{ vars.ECR_REPOSITORY }}:$IMAGE_TAG .
          docker push $ECR_REGISTRY/${{ vars.ECR_REPOSITORY }}:$IMAGE_TAG
