# Uploading artifact, Setting conditions, and permissions
So in this we are going to see conditions, how to get artifact and github_token.

when our workflow gets executed workflow will get a GITHUB-TOKEN so by defaut this token have read, and write permissions to the repositry. so when we are running workflow on our main branch we usually dont want write permissions to our repositry.

So if you want to change permissions of github token you can write in your file.

permissions:
  contents: read


now this workflow will have only read from our repositry.


#######################################################################

how to get artifact so when we build mvn create artifact how to get it.

  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Maven Build
        run: mvn clean install -DskipTests

      - name: Upload build artifact
        uses: actions/upload-artifact@v7 => this is the prebuilt action from github market place.
        with:
          name: vprofile-artifactpath => this will be the artifact name
          path: target/*.war => and this is the path where maven saves its artifact file name can be changed according to your pom.xml file.


Now after this we will get a zip file in our actions folder and we can download the file then. so thats how we get our artifact in github actions.

#######################################################################

conditions in file:
 for example like you want to get notified if previous step fails so you can add if.

  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Maven Build
        run: mvn clean install -DskipTests

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: vprofile-artifactpath
          path: target/*.war

      - name: Notify if build fails
        if: failure() => if the previous step failed the workflow will not get executed and show this error.
        run: echo "Build failed. Please check the logs for details."


so thats how we can send notification usually we send slack or email notification.

failure() is a built in function we call if upper code fails for some reason its value will true and we get notified.

run the whole code 1 time and check.

name: Build Wf

on:
  push:
    branches: ["main", "docker", "local"]

  pull_request:
    branches: ["main"]
 
  schedule:
    - cron: "10 14 * * 1-5"

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

      - name: Upload build artifact
        if: github.ref == 'refs/heads/main'
        uses: actions/upload-artifact@v7
        with:
          name: vprofile-artifactpath
          path: target/*.war

      - name: run build on other branches
        if: github.ref != 'refs/heads/main'
        run: echo "Build is skipped for non-main branches."

      - name: Notify if build fails
        if: failure()
        run: echo "Build failed. Please check the logs for details."

  testing:
    runs-on: ubuntu-latest

    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Run tests on main branch
        if: github.ref == 'refs/heads/main'
        run: mvn test

      - name: checkstyle
        if: github.ref == 'refs/heads/main'
        run: mvn checkstyle:checkstyle

      - name: Run tests on other branches
        if: github.ref != 'refs/heads/main'
        run: echo "Tests are skipped for non-main branches."


############################################
if: github.ref == 'refs/heads/main' => if the trigger is on main branch then only the commans which have the condition will get executed otherwise they will through an echo message as defined.

checking with if statments that branch should be main if theres a push on another branch workflow will get executed but the commands which have these if statements will notbe executed if the push is on main branch then only these commands will get executed.

https://docs.github.com/en/actions/reference/workflows-and-actions/expressions => see more expresiions here
