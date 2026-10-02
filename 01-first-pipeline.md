# 01 — First Pipeline (Build & Test)

login to your github account.

create a private repo. 

create ssh keys n your local computer ssh-keygen and give file a name like vprofile-private.

In this we are using a java project name vprofile project. you can access the project here from my github repo https://github.com/AswadAyaz/vprofile-project

now make a config file and add the data below. this config file tells ssh when the command is for this repo use this private file to authenticate because our project is private.

Host github.com-vprofile-private => we will give this name in ssh command. and this should match with the private key file name you give while making key files.
	HostName github.com
	User git
	IdentityFile ~/.ssh/vprofile-private => private file path
	IdentitiesOnly yes

now copy the public key and give it in ssh-gpg keys in github settings not repo settings.

now make a directory. where youre going to clone the repo.

git clone git@github.com-vprofile-private:AswadAyaz/vprofile-private.git

now open the repo in vscode.

code .

So there are no files yet.

go to your source code of vprofile project https://github.com/AswadAyaz/vprofile-project
and download a zip format and unzip it files will be in vscode. select the branch docker.

###############################################################################

now go to your github repo and go in actions tab some concepts about directory structure and yaml file.

in actions ---> there are many prebuilt actions which we can use and there is a simple workflow for understanding click on it.

youre going to see a path like this your-repo-name/.github/workflows/blank.yml this is the full path where we write code for cicd when we make our own file we have to give samepath and file name could be any.

name: CI => this defines the name of the workflow

on: => here we give how the workflow will get triggered
 
  push: => if a push happen on (main branch) workflow will get triggered
    branches: [ "main" ] => its in a list format we can write more branches also
  
  pull_request:
    branches: [ "main" ] => if a pull happens the workflow will get triggered also.

  workflow_dispatch: => this option allows us to run workflow manually from actions tab

Remember if your project has other branches and you run push or pull events on them the pipeline will not get triggered unless you mention it in your workflow.

jobs: => under this we can define multiple jobs what we want to do.

   build: => job name means we have to build something

       # The type of runner that the job will run on job will not run on github itself there will be a runner and its a predefined runner on this runner the source code will get cloned and next steps will get executed there. you can also create runners and add it into github actions and then you can mention your runner here.
       
       runs-on: ubuntu-latest

     # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
     
       - uses: actions/checkout@v6 => this is how you use predefined actions, checkout is the action name and this action is used to clone the source code into the runner.
   
       - name: Run a one-line script => step name 
         run: echo Hello, world! => run means run a shell command.


       - name: Run a multi-line script
         run: |
          echo Add other actions to build,
          echo test, and deploy your project.

https://github.com/marketplace => This is the market place where you can see predfined actions.

###########################################################################

Open Your directory in vscode where you have the project cloned make a directory in your root folder, .github => in this workflows => in this main.yml 

name: Build Wf => pipeline name

on:
  push: 
    branches: 
      - main

jobs:
  build: => my first job name is build

    runs-on: ubuntu-latest => this is the runner

    steps:
      - name: Checkout code => step name
        uses: actions/checkout@v7 => this will clone the source code in ubuntu-latest

      - name: Maven Build => here we build it
        run: mvn clean install -DskipTests

  testing: => second job is testing

    runs-on: ubuntu-latest
    needs: build => this means testing job can only run after build job is completed.

    steps:
      - name: Checkout code
        uses: actions/checkout@v7

      - name: Run tests
        run: mvn test

      - name: checkstyle test 
        run: mvn checkstyle:checkstyle

in github actions all the jobs runs paralelly because every job have its different runner.
if you need some job to get executed after any particular job than you have to give need.

########################################################################

now click on source controller icon and commit.

a file will be opened for writing a commit message. 

save it or down theres a button commit click it. and if the publish option comes click on it.

and go to your github repo click on actions click on the commit message and see your jobis in progress.

Fail Your pipeline give Some different versions of checkout actions and trigger the pipeline again, Now go in actions and see the errors see which job failed and why it gets failed.

Now solve the issue like there is a version mismatch issue go to the market place searchyour action checkout open it and see which version is in use.
