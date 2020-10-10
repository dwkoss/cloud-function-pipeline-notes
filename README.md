# cloud-function-pipeline-notes
Some notes on how to set up a super simple ci/cd pipeline with cloud functions, cloud source repositories, build triggers

# Create repository

## Create a repository in github
Supply `index.js`
```javascript
const async run = (req, res) {
    res.send('hello world');
}
```

## Link the repository in Cloud Source Repositories
https://source.cloud.google.com/[project-name]

1) Click "Add a repository"
1) "Connect external repository"
1) Select "Project" (the name of the project we're in)
1) "Git Provider" Github
1) Select from options under "Connect external repository"
1) Click "Connect selected repository"

## Create Cloud Function

https://console.cloud.google.com/functions

1. Click "Create a Function"
1. Set "Function name"
1. Trigger type: "HTTP"
1. Authentication: "Require authentication"
1. Click save
1. Set "Entrypoint" to "Run" (name of function in `index.js`)
1. Set "Source code" to "Cloud Source repository"
1. Select repository name as found in Cloud Source Repositories
1. "Branch name", "master", or "main" - whichever is prod code.
1. Deploy
1. Click "Testing" tab
1. "Test The function"
1. Observe "hello world"

## Create Build Trigger

1. Create a cloudbuild.yaml file in github source repository
```yaml
steps:
- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - functions
  - deploy
  - [name-of-cloud-function]
  - --source=.
  - --trigger-http
```
2. Go to [Cloud Build](https://console.cloud.google.com/cloud-build)
2. Click "Triggers"
2. Click "Create Trigger"
2. Enter a name for the trigger
2. Select "Push to branch"
2. Select Repository created in previous steps
2. Enter regex for name of branch that is prod source of truth
2. Select "Cloud Build configuration file"
2. Cloud Build configuration file location "/cloudbuild.yaml"
2. Build should run
2. Test by changing index.js file

# Bonus Setup
## For Cron Jobs
## Create a Scheduler

1. Go to [Cloud Scheduler](https://console.cloud.google.com/cloudscheduler)
1. Click "Create a job"
1. Enter a name for the scheduler
1. Set frequency using unix-cron format
1. Set Timezone to EST
1. Set Target to HTTP address of cloud function
1. "Show more"
1. Auth header "Add OIDC token
1. Add a service account that has "Cloud functions invoker" role
1. Audience is HTTP address of cloud function
1. Click "Create"