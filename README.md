# Azure DevOps Self Hosted Agent
This repo contains the json config file, needed to create the self hosted agents within Azure DevOps. Microsoft references the following Github repo to build their own Azure DevOps agents.
https://github.com/actions/virtual-environments
Always reference this repo for the latest build agent enhancements and fixes. The process uses HCL Packer to build and provision the image. The json config file is used to build the image, referencing the installation binaries in the build directory. To install all the necessary applications onto the image.
