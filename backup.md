# Azure DevOps Backup Tool: Guide to creating your first backup pipeline

## Introduction

This document provides a detailed procedure for backing up your Azure DevOps (ADO) organization or project using the **Azure DevOps Backup Tool** from Solidify.

### Process overview

1. **Determining the correct Azure DevOps project to house our backup infrastructure**
1. **Creating a git repository to hold our backup pipeline resources**
1. **Creating our first backup pipeline**
1. **Storing pipeline variables and secrets in Variable Groups**
1. **Advanced config file usage + examples for various scenarios**
1. **Running our backup pipeline and assessing the results**
1. **Troubleshooting and support requests**

### 1. Determining the correct Azure DevOps project to house our backup infrastructure

Before creating your backup pipeline, select the appropriate Azure DevOps project to house your backup infrastructure. Consider the following guidelines:

- **Dedicated Backup Project**: Create a dedicated project in Azure DevOps to isolate the backup pipelines, repositories, and related resources from your development activities. This helps in managing access control and ensures that backup-related activities do not interfere with your other projects.
- **Permissions**: Ensure that the project has the necessary permissions for accessing the resources you want to back up. Administrators and pipeline contributors should have access to this project.
- **Naming Conventions**: Use clear and descriptive names for the project to easily identify its purpose, such as `Backup-Infrastructure` or `Project-Backup`.

### 2. Creating a git repository to hold our backup pipeline resources

After selecting the appropriate Azure DevOps project, the next step is to create a Git repository to store the resources required for the backup pipeline.

Navigate to your Azure DevOps project, go to the **Repos** section, and create a new repository. Name it something descriptive like `Backup-Pipelines`.

![image](https://github.com/user-attachments/assets/5d9e1472-41eb-4201-a661-e65f878b63df)

![image](https://github.com/user-attachments/assets/18c00353-8b7f-403c-ac8a-9c7e48fe7ae8)

Our repository should now be initialized with an empty README:

![image](https://github.com/user-attachments/assets/d6e6aef2-62d5-4c0c-bd77-3ee321d3b738)

### 3. Creating our first backup pipeline

With your Git repository ready, you can now create your first backup pipeline.

In Azure DevOps, go to the **Pipelines** section and create a new pipeline:

![image](https://github.com/user-attachments/assets/c0b44c64-74d4-4bdd-97b9-9dc6484cd133)

When asked **Where is your code?**, select **Azure Repos Git (YAML)**:

![image](https://github.com/user-attachments/assets/d2dcebde-6a7f-415b-ace4-dae733991870)


  - **Select Repository**: Choose the repository you created in the previous step (`Backup-Pipelines`).
  - **YAML Pipeline**: Opt for a YAML-based pipeline for better version control and flexibility. Use a template provided by the Azure DevOps Backup Tool or create your own from scratch.
  - **Pipeline Script Example**: Below is a basic YAML configuration for a backup pipeline:
    ```yaml
    trigger:
      branches:
        include:
          - main

    pool:
      vmImage: 'ubuntu-latest'

    steps:
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '6.x'
        installationPath: $(Agent.ToolsDirectory)/dotnet

    - script: |
        dotnet restore
        dotnet build --configuration Release
        dotnet run --project BackupTool --configuration Release
      displayName: 'Run Backup Tool'
    ```
  - **Customization**: Modify the pipeline script to match your backup needs, including the source project, artifacts to back up, and storage location.

### 4. Storing pipeline variables and secrets in Variable Groups

### 5. Advanced config file usage + examples for various scenarios

### 6. Running our backup pipeline and assessing the results

### 7. Troubleshooting and support requests
