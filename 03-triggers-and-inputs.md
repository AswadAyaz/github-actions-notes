# Triggers And Inputs
In this we are going to talk about inputs we give to an action and also different triggers.

for example this is an action:

 uses: actions/checkout@v7 => this action clones the source code in the runner.
 with: => and with this action we pass some parameters or input.
   fetch-depth: 0 => so with this its not just going to clone the source code its goingto get the previous commit history, logs and everything which might be required if you want to compare previous commit.

checkout genereally used to clone the source code in the runner so default value is fetch-depth:1 which even if we dont give its there iska matlab sirf latest commit clone hoga. fetch-depth: 0 complete git history download hogi sarre commits saari branches aur tags.

Kab use karte hain?

Jab workflow ko Git history ki zarurat ho, jaise:

git log
git diff
git describe
Version number generate karna
Changelog creation
SonarQube ya kuch analysis tools jo puri history dekhte hain

so thats how we give input you can go in checkout action and see there are more inputs like this . these are also called parameters give to this action. checkout different actions in marketplace and there inputs we need them in a real pipeline.

https://github.com/marketplace/ => search for checkout and see different types of inputs.

triggers:

#on:
#  push:
#    branches:
#      - main

we have seen this trigger if theres a push to this branch the workflow will get triggered.

now we will see another trigger which is pull_request.

  pull_request:
     branches:
       - main


When Developers are making changes to the code they dont directly make changes into themain branch in most cases they create new branch and they made their changes and push the code to the new branch first and then what they do is they merged the main and the new branch and when this happens this process triggers the pipeline and its name is pull_request event OR Trigger.

so first create a new branch in your source code 2 ways terminal and github ui:

1st way github ui:

go to your repo click on branches give branch name and click on create branch.

create a branch bugfix-1

now the branch have all of the code which main have.

now in terminal fetch the branch git fetch origin.

git switch bugfix-1 => switch to the branch now make some changes Like edit readme fileor create some new files etc.
 
git push origin bugfix-1 => now main have the previous code and bugfix-1 have the new code.

Now:

Go to your repositry and click on Pull requests.

New pull Request.

switch the compare branch to bugfix-1.

Down there you can see what are the changes.

Create pull request.

Give some description.

And create pull request.

Go to actions tab and see your workflow get triggered.

Now go to Pull requests again click on your Pull request.

Youl see an option called Merge pull request If everything is good like workflows gets executed successfully click on merge pull request it will merge the code to the main branch.

And then youll see an option called Delete Branch after the work is done Developers mostly delete the branch.

now its deleted and merged in the main branch.

now delete and merge the changes in our local repo.

git checkout main => switch to the main branch

git branch -D bugfix-1 => delete the new branch 

git fetch --prune => is use to download the latest updates from your remote repositry while simultaneously eleteing your stale remote-tracking branches.

git branch -a => see all the branches

git pull => pull the changes which have done in the main branch after merge.

terminal way:

 git checkout main => switch to main

 git pull origin main => make sure repo is update with code

 git switch -c bugfix-1 => create the branch.

 now make the changes.

 git add .
  
 git commit -m "fixed bugs"

 git push origin bugfix-1

lets say everthing is working now we have to merge this code in to main branch.

go in pull request.

New pull request.

select bugfix-1 --> below youre going to see the changes.

create pull request.

give desc.

Create pul request.

youre gonna see its automatically triggering the pipeline.

go in actions and see it.

Now go to Pull requests again click on your Pull request.

Youl see an option called Merge pull request If everything is good like workflows gets executed successfully click on merge pull request it will merge the code to the main branch.

And then youll see an option called Delete Branch after the work is done Developers mostly delete the branch.

now its deleted and merged in the main branch.

now delete and merge the changes in our local repo.

git checkout main => switch to the main branch

git branch -D bugfix-1 => delete the new branch

git fetch --prune => is use to download the latest updates from your remote repositry w
hile simultaneously eleteing your stale remote-tracking branches.

git branch -a => see all the branches

git pull => pull the changes which have done in the main branch after merge.


########################### 
other triggers:

workflow_dispatch ---> manually running the workflow.


schedule:
    - cron: '30 2 * * 1-5' # monday to friday  night 2 30 this is a cron job format.

it dosent matter if theres a push or a pull request at this time the job will get executed.

############################
name: Build Wf

on:
 
  pull_request:
     branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Maven Build
        run: mvn clean install -DskipTests

  testing:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - name: Run tests on main branch
        run: mvn test

      - name: checkstyle
        run: mvn checkstyle:checkstyle
