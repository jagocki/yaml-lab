1. Lets create the variable group in the Azure DevOps. 

    1.1 Choose Pipelines -> Library

    1.2 Create new variable group and add a variable named ManagedVariable, and give it value of true
2. Add the following code to the pipeline to refer the variable group.
It references the variable group as a parameter, which can be provided when the pipeline starts
~~~~
pool:
  name: Default

parameters:
  - name: GlobalSettings
    default: GlobalVariables
    type: string

variables:
  - group: ${{ parameters.GlobalSettings }}
  - name: valueCounter
    value: $[counter(variables['build.reason'], 0)]


~~~~
Further, you can use the provided variables in tasks, and create new variables. Note that from the variables we use here, one comes from variable groups, the other is set up directly in 
pipeline.
~~~~
    - task: CmdLine@2
          displayName: Extra command line
          env:
            envVar: $(ManagedVariable)-$(valueCounter)
          name: setVar
          inputs:
            script: |
              echo variable value is:
              echo %envVar%
              echo expression value is:
              echo $(valueCounter)
              echo ##vso[task.setvariable variable=varFromTask;isoutput=true]taskValue
~~~~

The new variable can be consumed with following syntax, in another tasks, jobs and stages:

* The same job
~~~~
        - script: |
              echo varFromTask = $(setVar.varFromTask)
~~~~

* Same stage, but another job
~~~~
        variables:
          varFromTask: $[ dependencies.GetTheJobDone.outputs['setVar.varFromTask'] ]
        steps:
          - script: |
              echo varFromTask = $(varFromTask)
~~~~

* Another stage
~~~~
  - stage: SecondStage
    dependsOn: FirstStage
    jobs:
    - job:
      variables: 
       varFromTask:  $[ stageDependencies.FirstStage.GetTheJobDone.outputs['setVar.varFromTask'] ]
        steps:
        - script: |
            echo Runnig second stage!
            echo $(varFromTask)
~~~~


3. In order to simplify the yaml pipeline, we can create the template to reuse the steps.
Create the new file with .yml extension in the same location as your pipeline, and paste the following content there
~~~~
parameters:
- name: ManagedVariable
  type: string
  default: ''
- name: valueCounter
  type: string
  default: ''
 
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
      envVar: $(ManagedVariable)-$(valueCounter)
      inputs:
      script: |
        echo variable value is:
        echo %envVar%
        echo expression value is:
        echo $(valueCounter)
        echo ##vso[task.setvariable variable=varFromTask;isoutput=true]taskValue
      name: setVar
  - script: |
          echo varFromTask = $(setVar.varFromTask)
- job: GetVariable
  dependsOn: GetTheJobDone
  condition: always()
  variables:
      varFromTask: $[ dependencies.GetTheJobDone.outputs['setVar.varFromTask'] ]
  steps:
  - script: |
         echo varFromTask = $(varFromTask)
~~~~

In the main pipeline, we can 'invoke' the above tempalate in the following way:
~~~~
stages:
  - stage: FirstStage
    jobs:
      - template: jobs-tamplate.yml
        parameters:
          ManagedVariable: $(ManagedVariable)
          valueCounter: $(valueCounter)
~~~~