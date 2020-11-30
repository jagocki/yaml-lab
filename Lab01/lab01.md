#To do:

1. Add new pipeline with the wizard.

    1.1 Chose the appropriate repository

    1.2  Use the Starter Template

2. Replace the name of the agent queue with the name that is available in your environment
3. Save and run pipeline
4. Review the output
4. Edit a pipeline:

    5.1 Add New Task - script or powershell that will display some text. Use the UI ask editor to specify the task arguments. It could be something like this:

~~~

- task: CmdLine@2
  displayName: Extra command line
  env:
    envVar: envVal
  inputs:
    script: |
      echo variable value is:
      echo %envVar%
~~~

Review the output, does it shot the value in the output ?