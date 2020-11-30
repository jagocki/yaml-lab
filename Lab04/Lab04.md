1. Add the following snippet to the last stage:
~~~~
    - task: CmdLine@2
        displayName: GetAllEnvVariables
        inputs:
          script: |
            set
    - deployment : 'Stage'
      displayName: 'Deployment'
      environment: Stage
      strategy:
        runOnce:
          deploy:            
            steps:
            - script: |
                echo deployment is done!
~~~~

Run the pipeline and navigate to the environments in the pipelines.