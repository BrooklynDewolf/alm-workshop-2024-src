# Introduction

Hi everybody! Welcome to the ALM Workshop of 2024!

We give this workshop with a few goals in mind:
* Learn new (or unfamiliar) technologies
* Implement CI ALM concepts
    * You learned the theory, now it's time to implement

While it's easier to work with technologies that you use day-to-day, today I would like to try some technologies that you (probably) haven't really worked with yet:

* Golang
* Quay
* GitHub DevSpaces / DevContainers
* GitHub Actions
* RedHat OpenShift

# Workshop

The workshop will be divided in several tasks that each represent a checkpoint.
If one of the tasks is not working or you ran out of time you can always check-out the branch for that specific checkpoint and catch-up from there.

# Tasks
## Task 1
### Prerequisites

* ***Fork** the base repository to your own account!*
* Start your own CodeSpace from the "main" branch **or** run everything locally in VSCode with the devcontainers extension! (This exercise was mostly tested on the GitHub one, so safe to use that)
    ![Create Codespace](docs/github-create-codespace.png)
    * This can take a while (+-5mins), so don't panic.
    * You should get some pop-ups to install extensions from the devcontainer.json file
* Run the command `docker run -p 80:80 nginx` to create a NGINX container inside your devcontainer
    * VSCode should propose to open up the forwarded port with a button "Open in Browser" and you should see the NGINX Welcome page

        ![NGINX welcome](docs/nginx-welcome.png)
        * If you missed the popup you can click on the little transmission tower in the bottom left

            ![VSCode Transmission Tower](docs/vscode-portforward.png)
        * You can close the container in the terminal by using CTRL+C

* Go over the code & files in the repository and read the comments to explain what things do
    * ./.devcontainer/devcontainer.json = describes how your devcontainer is setup
    * ./workshop-service/* = Contains all the source code
    * ./Dockerfile = Docker multi-stage file to build & run our Go application
* Test the Dockerfile using `docker build -t test-workshop . --progress=plain` to build the image & run the image by using `docker run -p 3000:3000 test-workshop`
    * The container should be reachable on the URL provided by the GitHub workspace **or** on localhost from your browser
    * You can also surf to the /workshop endpoint

Cool! We've just managed to compile our application, without actually having to install any kind of SDK or tool manually! This is the power of the devcontainers & multi-stage docker builds for developers and can really simplify onboarding of new team members.

### Task
Right now we've got a working "local" development environment from we can develop, however our goal is as followed:
* We want to be able to work with multiple developers and have consistent & repeatable builds for our application
    * We want to have some SDLC conventions to make sure that we all work in the same way
* The application should eventually be accessibly publicly, so we need to make sure that we can get the container we create in the pipeline on a system somewhere

The assignment is as followed:
* Create a [GitHub Actions pipeline](https://docs.github.com/en/actions/writing-workflows/quickstart#creating-your-first-workflow):
    * Create the directory .github/workflows and add a ci.yaml file
        * You can now the file however you want just make it clear what it does for you :)
    * Run the container build & run pipeline to create a container image artifact
        * HINT: You've already done this locally
    * Push the created container to the [quay.io](https://quay.io/) container registry using a [Robot Account](https://docs.quay.io/glossary/robot-accounts.html)
        * HINT: The image name should be as such quay.io/<quay-user>/alm-workshop to properly push
        * HINT: You might need to register the repository in quay first before you can push.
        * HINT: How will you manage your Quay Robot User password in the pipeline?
    * You should be able to see your image in Quay.io, run it inside your devcontainer to validate
        * `docker run -p 3000:3000 quay.io/<yourname>/alm-workshop:<tag>` to pull & run inside your devcontainer

            ![Quay Repository Image](docs/quay-repository-tag.png)

## Task 2

* Add [SemVer](https://semver.org/) release images
    * This means that when you create a GitHub release with a certain SemVer, the pipeline should be ran again and the docker-image should be <quay-user>/alm-workshop:x.x.x
        * HINT: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#release
* Release your current code as version 1.0.0

## Task 3

:exclamation: **This task needs to be completed in the aelm-workshop-23-cfg configuration repository**

## Task 4

:exclamation: **This task needs to be completed in the aelm-workshop-23-cfg configuration repository**

## Task 5

* Update the content of the /workshop endpoint:
    * To include yourself in the list of participants
    * Add a field "SweaterScore" that holds a numeric value of 1-10 on the presentators Christmas Sweater.
        * Provide Validation on this range.
        * The default value of the SweaterScore is setup via an environment variable.
* Release this code as 1.1.0

## Task 6

:exclamation: **This task needs to be completed in the aelm-workshop-23-cfg configuration repository**

## Finish?

**Congratulations, you've now got a functioning code pipeline!! You can update, release and expose you're application on demand now** :clap::muscle:
The next step is to now improve upon this, so that you as a developer can focus solely on producing the code. Choose some of the extra tasks to do, in both repositories are different tasks related to their type.

## Extra's
* Do custom tags for your docker builds, right now it takes the branch name but try something as followed:
    * On "main" branch, should be tagged as such <name>:latest
    * On other branches, should be tagged as such <name>:dev-<commitHash> with the hash being a substring of 8 characters
* The dev Openshift environment should always the use the :latest image that is created when a developer pushes code to main and the build succeeds. Create a trigger in your pipeline that automatically restarts the deployment on Openshift
* Write a go-test & add tests to the multi-stage Dockerfile
* Add a metrics endpoint to your application that will give you insights into the application performance
* If you merge to "main" and your build succeeds at an extra step that Restarts your deployment on Openshift to redeploy the main image immediately
* Write a Go test for our /workshop endpoint

# Directory structure

```
/
|── /.devcontainer
    └── devcontainer.json
|── /.github
    |── /workflows
        └── ci.yaml
|── /docs
|── /workshop-service
    ├── main.go
    |── workshop.go
    ├── go.mod
    └── go.sum
|── .gitignore
|── Dockerfile
└── README.md
```

# Solutions / Tips

In general I tried to work in a branching structure which is named as followed solution/task-<number>-<description>

## Code repository

### Task 1 - Build pipeline

#### Application

If you want to build the application locally instead of inside the Docker container, you can try this.

**Golang**

`go run main.go workshop.go`

`go build -o ../bin/workshop -v ./...`

**POST command**

`curl -X POST -H "Content-Type: application/json" -d '{
  "name": "AELM Workshop",
  "date": "07/12/2023",
  "presentator": "Arnout Hoebreckx",
  "participants": ["Arnout Hoebreckx"],
  "sweaterscore": 8
}' http://localhost:3000/workshop`

#### Docker

Some basic Docker commands that you will probably need during this exercise.

**Build**

`docker build -t alm-workshop .`

**Run / Test**

`docker run -d --rm -p 3000:3000 alm-workshop`

`curl localhost:<whatever-github-or-vscode-tells-you>`

**Rename (tag) image**

`docker tag alm-workshop:main quay.io/$QUAY_USERNAME/alm-workshop:main`

**Login to remote repository**

`docker login -u $QUAY_USERNAME quay.io`

**Push to remote repository**

`docker push quay.io/$QUAY_USERNAME/alm-workshop:main`

## Task 2 - Release

GitHub has a built-in release mechanism, that you can access through the UI. Important for the builds to check out when