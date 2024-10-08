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
    useCustomConfigurations: false
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
#    resourceTFS: true      # Uncomment if there are any TFVC repositories in the source Project
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

We are now done with creating our Backup Pipeline. Before we run it, we must specify some parameters to let the pipeline know our API credentials, where our resources are, etc.

### 4. Storing pipeline variables and secrets in Variable Groups

We will use **Variable Groups** to store our pipeline parameters, which will enable us to use the backup pipeline in a dynamic fashion.

Under the **Pipelines** -> **Library** menu, add a new Variable Group:

![image](https://github.com/user-attachments/assets/693619be-ca39-431a-9624-9b8a69b9ffab)

Name the variable group "backup-pipeline" (must match the variable group name in the YAML definition of the backup pipeline). You can change the name at any time as long as your consistently update the variable group name in the YAML definition of the pipeline. Enter the same variables as shown in the screenshot below (replace the values with your own scenario) and click **Save**.

![image](https://github.com/user-attachments/assets/f2640a4c-fbe3-441b-a082-2482caf3427a)

The necessary variables for the Backup Job are as follows:

- `migrationToken` (The Personal Access Token to be used by the backup job)
- `sourceOrg` (The source Azure DevOps organization name)
- `sourceProject` (The source ADO Project name)
- `sourceUrl` (The URL of the ADO organization)
- `sourceUsername` (The email address of the user account to be used by the backup job. Must be the same account as the owner of the `migrationToken`)

Optionally, you may opt to store your Backup Job variables inside the YAML build pipeline definition itself, or in an Azure KeyVault (use the **Link secrets from an Azure key vault as variables** setting).

### 5. Advanced config file usage + examples for various scenarios

You can extend the functionality of **ADO Backup Tool** by using custom configuration files. These files will override the default configuration of the underlying application behind the pipeline task, and enable you to change the behavior on a per-component basis.

If you are simply demoing the backup functionality or otherwise have no use for custom configuration files, you can skip ahead to step 6.

#### Description of custom configuration file usage

Common use cases for using custom config files are:

- Filtering git repositories by name (wildcard supported)
- Filtering build/release pipelines by name (wildcard supported)
- Filtering work items based on a WIQL query
- Overriding default git credentials for git repositories and wikis

#### How to enable custom configuration files

Edit your back up pipeline again. You will need to supply the **useCustomConfigurations** and **customConfigurationPath** parameters in the task configuration, like this:

```yml
useCustomConfigurations: true
customConfigurationPath: '$(System.DefaultWorkingDirectory)/custom-configs'
```

Your pipeline should now look like this:

![image](https://github.com/user-attachments/assets/9f3c1e48-ad20-4d12-8c7b-c388c13f7eab)

Save the pipeline before continuing with the next step.

#### How to create custom configuration files

You provide custom configuration files by placing them in the **same git repository** as your backup/restore pipeline. The naming convention of the configuration files is always "config-[**type of component**]-[**export/import**].json.

You can find additional information about the usage of custom configuration files here: <https://github.com/solidify/azure-devops-backup-tool-docs/blob/main/custom-configs.md>.

You can view templates for individual configuration files for each ADO component here <https://github.com/solidify/azure-devops-backup-tool-docs/tree/main/config-templates>.

You can find documentation and usage samples with example configuration files for each ADO component here: <https://github.com/solidify/azure-devops-backup-tool-docs/tree/main/adapter-docs>

As an exercise, let us create a custom configuration file which will let us granularly filter which Work Items to export in the Backup Pipeline.

Start by creating a new folder inside the git repository named **Backup-Pipelines**:

![image](https://github.com/user-attachments/assets/446d3000-5527-48eb-beaf-7efbdee2332b)

The new folder name should be `custom-configs` and the residing file name should be `config-workitem-export.json`. This will create a configuration file which will affect only **Work Items** during the backup phase (and not the restore phase):

![image](https://github.com/user-attachments/assets/dec0ee0e-b4bc-47a2-95a5-5deccbf6521c)

Paste the following file contents into the new file, and replace `ORGNAME` with your Azure DevOps organization name, and `PROJECTNAME` with your target project to be backed up:

```yaml
{
    "configuration":
    {
        "source": "https://dev.azure.com/ORGNAME",
        "target": "D:\\a\\1\\s\\MigrationWorkspace",
        "batchSize": 50,
        "skip": 0,
        "projects": [
            {
              "name":"PROJECTNAME",
              "query":"Select * From WorkItems Where [System.TeamProject] = 'PROJECTNAME' AND [System.CreatedDate] < '2024-08-08T00:00:00.0000000' Order By [Id] Asc"
            }
        ],
        "SkipAlreadyDownloadedRevisions": true,
        "apiVersion": "7.0"
    }
}
```
The resulting file should look like this:

![image](https://github.com/user-attachments/assets/b8eff65a-a872-4b0d-8d30-c20e69be90a2)

Save the new file by committing it to the version control history:

![image](https://github.com/user-attachments/assets/41f3a869-6b37-4ff6-8fda-121ab78e6d88)

We are now set up to use our custom configuration file.

#### Verify the configuration file in use

Let us verify that the pipeline has successfully picked up our new custom configuration file.

Go ahead and **edit** the pipeline, and set the `System.Debug` property to `true`, like in the below screenshot:

![image](https://github.com/user-attachments/assets/f3763470-1284-4d38-881b-a16e53cca4aa)

Now, save and run the pipeline. Inspect the log for the **ADO Backup Tool: Export** task and verify that the task is picking up the correct configuration file. The configuration should appear just like you entered it in the file `custom-configs/config-workitem-export.json`:

![image](https://github.com/user-attachments/assets/4372e788-aa3b-4fb3-b186-9d1c6e5dc828)

If the configuration as reported by the pipeline is identical to the configuration you specified earlier, then everything is now set up correctly, and you can continue to step 6.

### 6. Running our backup pipeline and assessing the results

Finally, let us run our newly created Backup Job and verify that the results are as expected.

From the pipeline menu under **Pipelines**, click **Run pipeline**:

![image](https://github.com/user-attachments/assets/2051e879-6748-4efe-b99d-0c1e709a273f)

In the **Run pipeline** dialogue, simply click **Run**:

![image](https://github.com/user-attachments/assets/cc981eb3-5263-4129-9d0f-f09bfb6e1cd5)

After clicking **Run**, you will be taken to the Run results screen. Simply wait here until the pipeline finishes or times out (you might need to refresh your browser tab). You may also need to authorize the pipeline to access all of the necessary variable groups.

If you ever need to get back to the Run results screen, simply navigate to **Pipelines** -> **Backup-Pipelines** -> (click the latest Run) from anywhere within Azure DevOps.

The pipeline should finish without errors.

![image](https://github.com/user-attachments/assets/a694901a-e350-44ee-8a27-bf0e84fa904a)

From the results screen of the run, you can do either of the following from the screen:

- Click on the **Job** to view the log file.
- Click on the build artifact to view the components that were successfully backed up. We also require you to send us the build artifact in order to troubleshoot any issues.

![image](https://github.com/user-attachments/assets/fa2dde04-00c0-408a-b834-217543b147b0)

From the **Artifacts** menu, in order to download and inspect the build artifact, simply click the **downward-facing arrow to the left of "Export"** -> click the **3 dots to the right** -> **Download artifacts**. This will download a .zip-file to your computer which contains the backed up resources.

![image](https://github.com/user-attachments/assets/bd7ebdf4-443b-4909-a30a-2490c301c1bf)

### 7. Troubleshooting and support requests

In the event that the ADO Backup Tool fails, you will receive an error message similar to this one at the bottom of the task log:

![image](https://github.com/user-attachments/assets/fc82e6f2-0a8c-4709-b194-9df51b871b01)

Click anywhere in the log to bring the focus to the log file area, then press **ctrl+F** to search the log file. Enter the search string `ERR]`. All relevant errors should become highlighted, and you can use the search panel to navigate between them:

![image](https://github.com/user-attachments/assets/090cc3d1-04bf-4fa5-9af9-098efc7a9b34)

If you are making a support request, please make a note of each error and forward all relevant error messages to us via e-mail.

For a more comprehensive guide on troubleshooting the **ADO Backup Tool** and submitting support requests, please read this article: <https://github.com/solidify/azure-devops-backup-tool-docs/blob/main/troubleshooting.md>.
