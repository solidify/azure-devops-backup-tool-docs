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

Below is a basic YAML configuration for a backup pipeline. In order to make the setup process as simple as possible, please copy the below code block in its entirety and paste into your YAML pipeline text editor:

```yaml
trigger: none

workspace:
  clean: all

variables:
- group: backup-pipeline
- name: workspace
  value: '$(System.DefaultWorkingDirectory)/MigrationWorkspace'
- name: System.Debug
  value: false       # Set to "true" to enable system diagnostics for troubleshooting or support requests

pool:
  vmImage: windows-latest

steps:
- task: ado-backup-tool-export@1
  displayName: 'ADO Backup Tool: Export'
  inputs:
    source: '$(sourceUrl)'
    sourceOrgName: '$(sourceOrg)'
    sourceProject: '$(sourceProject)'
    sourceUsername: '$(sourceUsername)'
    sourcePAT: '$(migrationToken)'
    onPrem: false
    workspace: '$(workspace)'
    useCustomConfigurations: true
    customConfigurationPath: '$(System.DefaultWorkingDirectory)/custom-configs'
    resourceAreaPath: true
    resourceArtifact: true
    resourceAuditLog: true
    resourceBoard: true
    resourceDeploymentGroup: true
    resourceEnvironment: true
    resourceGit: true
    resourceGitBranchPolicy: true
    resourceIterationPath: true
    resourcePipeline: true
    resourcePullRequest: true
    resourceQuery: true
    resourceTeam: true
    resourceTestplan: true
#    resourceTFS: true      # Uncomment if there are any TFVC repositories in the target Project
    resourceVariableGroup: true
    resourceWiki: true
    resourceWorkItem: true
  env:
    SYSTEM_ACCESSTOKEN: $(system.accesstoken)

- task: ArchiveFiles@2
  continueOnError: true
  condition: succeededOrFailed()
  inputs: 
    rootFolderOrFile: '$(workspace)'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber)-export.zip'
    replaceExistingArchive: false
        
- task: PublishPipelineArtifact@1
  condition: succeededOrFailed()
  continueOnError: true
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber)-export.zip'
    artifact: 'Export'
    publishLocation: 'pipeline'
```

Optionally, you may comment our or delete any of the `resource*` inputs if those components are not desired to be backed up.

The resulting pipeline should now look like this:

![image](https://github.com/user-attachments/assets/7743b6d6-6c86-4a3f-b6d6-ae4a0402d35c)

Now click **Save**:

![image](https://github.com/user-attachments/assets/033f98d9-08b1-4a2f-af08-a4e96d698c4b)

In the **Save** dialogue, click **Save**:

![image](https://github.com/user-attachments/assets/0b717dc2-6b07-4dd8-b1f1-35470c855e26)

We are now done with creating our 

### 4. Storing pipeline variables and secrets in Variable Groups

![image](https://github.com/user-attachments/assets/693619be-ca39-431a-9624-9b8a69b9ffab)

Name the variable group "backup-pipeline" (must match the variable group name in the YAML definition of the backup pipeline). Enter the same variables as shown in the screenshot below and click **Save**.

![image](https://github.com/user-attachments/assets/f2640a4c-fbe3-441b-a082-2482caf3427a)

The neccessary variables for the Backup Job are as follows:

- `migrationToken` (The Personal Access Token to be used by the backup job)
- `sourceOrg` (The source Azure DevOps organization name)
- `sourceProject` (The source ADO Project name)
- `sourceUrl` (The URL of the ADO organization)
- `sourceUsername` (The email address of the user account to be used by the backup job. Must be the same account as the owner of the `migrationToken`)

Optionally, you may opt to store your Backup Job variables inside the YAML build pipeline definition itself, or in an Azure KeyVault (use the **Link secrets from an Azure key vault as variables** setting).

### 5. Advanced config file usage + examples for various scenarios

![image](https://github.com/user-attachments/assets/446d3000-5527-48eb-beaf-7efbdee2332b)

![image](https://github.com/user-attachments/assets/dec0ee0e-b4bc-47a2-95a5-5deccbf6521c)

![image](https://github.com/user-attachments/assets/b8eff65a-a872-4b0d-8d30-c20e69be90a2)


### 6. Running our backup pipeline and assessing the results

![image](https://github.com/user-attachments/assets/2051e879-6748-4efe-b99d-0c1e709a273f)


### 7. Troubleshooting and support requests
