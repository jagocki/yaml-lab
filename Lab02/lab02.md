Todo:

1. Open pipeline from the previous lab to edit
2. Add job to the code, for task that will display all environment variables. Try to use the intellisense
3. You can use below snippet

~~~
trigger:
- master

pool:
  name: Default

jobs:
  - job: GetTheJobDone
    steps:
    - script: echo Hello, world!
      displayName: 'Run a one-line script'

    - script: |
        echo Add other tasks to build, test, and deploy your project.
        echo See https://aka.ms/yaml
      displayName: 'Run a multi-line script'
    - task: CmdLine@2
      displayName: Extra command line
      env:
        envVar: envVal
      inputs:
        script: |
          echo variable value is:
          echo %envVar%
  - job: GetTheEnvironmentVariables
    displayName: A nicer job description for getting the environment variable
    steps:
    - task: CmdLine@2
      displayName: GetAllEnvVariables
      inputs:
        script: |
          set

~~~

4.  Add Job dependency with the following:

~~~
  - job: GetTheEnvironmentVariables
    displayName: A nicer job description for getting the environment variable
    dependsOn: GetTheJobDone
~~~

Did it change the pipeline behaviour? Why?

5. Add FAIL step to the first job. to explicitly fail the task, and then run the pipeline. You can refere to below example

~~~
      inputs:
        script: |
          echo variable value is:
          echo %envVar%
          echo ##vso[task.complete result=Failed;]Lets Fail Here
~~~
The secondjob didnt start, but we can change it/

5. Add the condition to the second job

~~~
    dependsOn: GetTheJobDone
    condition: Failed('GetTheJobDone')
    steps:
~~~

6. Now it is time to extend our pipelines with stages

Add the stages key word, and the first stage, and the stages properties. Remember to fix the indendation

~~~
stages:
  - stage: FirstStage
    jobs:
      - job: GetTheJobDone
        steps:
~~~~

7. Add a second stage, to handle the second job. You can comment out the dependency for now with the '#' character

~~~
  - stage: SecondStage
    jobs:
    - job:
      steps:
        - script: echo Runnig second stage!
    - job: GetTheEnvironmentVariables
      displayName: A nicer job description for getting the environment variable
      steps:
      - task: CmdLine@2
        displayName: GetAllEnvVariables
        inputs:
          script: |
            set
~~~

7. Before we had some jobs dependencies, if we would like to keep, even if we would introduce more stages. It is a little more tricky, but still perfectly find
We have to define the dependcies on the stages and jobs level. Also we need to explictly tell that we want to execute the steps even if the First stage fails
Take a look on the full snippet.
~~~
  - stage: SecondStage
    dependsOn: FirstStage
    condition: in(dependencies.FirstStage.result, 'Failed', 'SucceededWithIssues', 'Skipped')
    jobs:
    - job:
      steps:
        - script: echo Runnig second stage!
    - job: GetTheEnvironmentVariables
      displayName: A nicer job description for getting the environment variable
      condition: eq(stageDependencies.FirstStage.GetTheJobDone.result, 'Failed')
      steps:
      - task: CmdLine@2
        displayName: GetAllEnvVariables
        inputs:
          script: |
            set
~~~
