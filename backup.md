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

Under **Select Repository**, choose the repository you created in the previous step (`Backup-Pipelines`):

![image](https://github.com/user-attachments/assets/612a9c85-7c83-48c7-97ec-ba862b2464b1)

Under **Configure your pipeline**, choose **Starter pipeline**:

![image](https://github.com/user-attachments/assets/b5c11231-8469-4410-a215-0e2836e45d60)

You can now design your pipeline. You can bring in the tasks provided by **Azure DevOps Backup Tool** by clicking **Show assistant** and then searching for `Azure DevOps Backup Tool` in the assistant column.
![image](https://github.com/user-attachments/assets/833e5276-92b2-4700-b082-158bde1923fd)

![image](https://github.com/user-attachments/assets/146a849d-5542-454c-89bc-77a130056677)

This will bring you into the task configuration. Here you you can view and change how the backup pipeline behaves, where it should look for components to back up, as well as API credentials for Azure DevOps:

![image](https://github.com/user-attachments/assets/34e41969-f363-40bc-8c87-10a6f8bd2326)


Below is a basic YAML configuration for a backup pipeline:
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
