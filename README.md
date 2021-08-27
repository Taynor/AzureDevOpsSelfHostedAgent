# Azure DevOps Self Hosted Agent
This repo contains the json config file, needed to create the self hosted agents within Azure DevOps. Microsoft references the following Github repo to build their own Azure DevOps agents.
https://github.com/actions/virtual-environments
Always reference this repo for the latest build agent enhancements and fixes. The process uses HCL Packer to build and provision the image. The json config file is used to build the image, referencing the installation binaries in the build directory. To install all the necessary applications onto the image.

Building the image can be performed locally, using Visual Studio Code usiog the Packer syntax. Or within an Azure DevOps pipeline, I have used the following example YAML to do this:

trigger: none

pool:
  vmImage: ubuntu-latest

jobs:
- job: buildAzdoAgents
  timeoutInMinutes: 720
  cancelTimeoutInMinutes: 5  

  steps:
  - task: ExtractFiles@1
    inputs:
      archiveFilePatterns: $(Build.SourcesDirectory)**/*.zip 
      destinationFolder: $(Build.SourcesDirectory)/imagegeneration
      cleanDestinationFolder: true 
      overwriteExistingFiles: false   

  - task: AzureCLI@2
    inputs:
      azureSubscription: 'azdevopsspn'
      scriptType: 'bash'
      scriptLocation: inlineScript
      inlineScript: 'sudo apt-get update && sudo apt-get install packer'

  - task: PowerShell@2
    inputs:
      targetType: 'inline'
      script: 'packer build $(Build.SourcesDirectory)/imagegeneration/imagegeneration/ubuntu2004.json'  
      
What the local build would fail installing the necessary binaries, until I added the commmands to install the utils for the Linux image:

 "provisioners": [
        {
            "type": "shell",
            "inline": [
                "mkdir {{user `image_folder`}}",
                "chmod 777 {{user `image_folder`}}",
                "echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections",
                "apt-get update",
                "apt-get upgrade -y",
                "sudo apt-get install dialog apt-utils -y"
            ],
            "execute_command": "sudo sh -c '{{ .Vars }} {{ .Path }}'"
        }
        
 After this amendment from the original repo, I discovered the image would build successfully.       
