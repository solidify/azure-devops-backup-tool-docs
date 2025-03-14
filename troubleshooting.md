# Troubleshooting guide

## Point of contact

You can submit an error report by emailing us at <support.adobackup@solidify.dev>.

## Troubleshooting guidelines

If the **Azure DevOps Backup Tool** fails to run properly, the problem can sometimes be remedied with some of the below steps:

1. Run the pipeline with `$(System.Debug) = 'true'` to collect more diagnostics (See the below section in this document).
1. Manually clear the workspace on the agent server and run the pipeline again.
1. Ensure that two or more backup jobs are not running concurrently on the same machine.
1. Ensure that the service account running the build agent has Read/Write access to the workspace folder (only applicable when running on a self-hosted build server).
1. Clear the task manually from `_tasks` inside the agent folder. Then run the pipeline to install the task again.

## How to gather diagnositcs

In order for us to resolve a customer's problem with the ADO Backup Tool, the customer must provide us with diagnostics information in order to assist with troubleshooting. The following steps need to be taken on their part:

1. Run the Backup/Restore pipeline with **Enable System Diagnostics** enabled in the Run Pipeline dialogue, as well as the **System.Debug** variable set to true, for example:
   ```yaml
   variables:
   - name: System.Debug
     value: true
   ```
1. Configure the Backup/Restore pipeline to publish the Migration Workspace as a **Pipeline artifact**. Here is an example pipeline definition to do this:
   ```yaml
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
       workspace: '$(System.DefaultWorkingDirectory)/MigrationWorkspace'
       resourceGit: true
     env:
       SYSTEM_ACCESSTOKEN: $(system.accesstoken)

    - task: ArchiveFiles@2
     condition: succeededOrFailed()
     inputs: 
       rootFolderOrFile: '$(System.DefaultWorkingDirectory)/MigrationWorkspace'
       includeRootFolder: false
       archiveType: 'zip'
       archiveFile: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber)-EXPORT.zip'
       replaceExistingArchive: false

    - task: PublishPipelineArtifact@1
     condition: succeededOrFailed()
     inputs:
       targetPath: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber)-EXPORT.zip'
       artifact: 'Export'
       publishLocation: 'pipeline'

    - task: ado-backup-tool-import@1
     displayName: 'ADO Backup Tool: Import'
     inputs:
       source: '$(sourceUrl)'
       target: '$(sourceUrl)'
       sourceOrgName: '$(sourceOrg)'
       targetOrgName: '$(sourceOrg)'
       sourceProject: '$(sourceProject)'
       targetProject: '$(targetProject)'
       targetUsername: '$(sourceUsername)'
       sourcePAT: '$(migrationToken)'
       targetPAT: '$(migrationToken)'
       onPrem: false
       workspace: '$(System.DefaultWorkingDirectory)/MigrationWorkspace'
       mappingFilePath: '$(System.DefaultWorkingDirectory)/custom-mappings'
       resourceGit: true
     env:
       SYSTEM_ACCESSTOKEN: $(system.accesstoken)

   - task: ArchiveFiles@2
     condition: succeededOrFailed()
     inputs: 
       rootFolderOrFile: '$(System.DefaultWorkingDirectory)/MigrationWorkspace'
       includeRootFolder: false
       archiveType: 'zip'
       archiveFile: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber)-IMPORT.zip'
       replaceExistingArchive: false

   - task: PublishPipelineArtifact@1
     condition: succeededOrFailed()
     inputs:
       targetPath: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber)-IMPORT.zip'
       artifact: 'Import'
       publishLocation: 'pipeline'
   ```
1. Finally, send us a copy of the following:
  - The **build definition** (.yaml file) of the Backup/Restore build pipeline.
  - The entire **log** from the ado-backup-tool-export/-import task.
  - The .zip file containing the pipeline artifacts, which was generated in step 2.

Email us at <support.adobackup@solidify.dev>

## How to troubleshoot migration errors due to erroneous HTTP responses (4XX or 5XX)

### Issue overview

If ADO Backup Tool could not not correctly import a resource, you will see an error message such as this in the log:

```
[12:48:54 INF] Creating a Build Definition: PartsUnlimitedE2E
[12:48:54 ERR] Message: Response status code does not indicate success: 400 (Bad Request)., InnerException: 
```

In order to gather additional diagnostics, we must perform the same Rest API query manually. This example will show you how to do this using **Postman**. This example assumes that the failing resource is a Build Definition on ADO Cloud.

### Step-by-step

Firstly, inside the MigrationWorkspace, locate the corresponding .json definition for the failing resource, and copy this file in its entirety, for example: `<Source Project>\Imported____\FailedBuildDefinitions\PartsUnlimitedE2E.json`

Use any of the following software to set up a manual Rest API query:

- Postman
- cUrl
- Powershell
- Other Rest API testing software

This example will use **Postman**. You must now set up a POST request to the correct endpoint (for example, in the case of Build Definitions for ADO Cloud, use <https://dev.azure.com/ORGANIZATION/PROJECTNAME/_apis/build/definitions?api-version=7.0>). Consultants at Solidify can find the correct API URL by looking at the source code for synchub.

Paste the .json file as the body. The resulting request should look like this:

![image](https://github.com/user-attachments/assets/05d681ef-7633-45f3-97f2-87cd1cccdd17)

The Authorization header should look like this:

![image](https://github.com/user-attachments/assets/d845064f-183f-4429-8c30-c31c0b8f0331)

If you are scripting the API call manually, the authorization header should be "Basic Base64Encode(:PAT)".

In the Response message, we can see the detailed message of the Bad Request response:

![image](https://github.com/user-attachments/assets/ff4ea872-fb5e-4ac7-b807-b7195279558d)

In this case, all we need to do is map the missing Agent Pool ID (126). In practise, the error message could be of many different kinds, but the most common issue is that you need to map some additional resource.

## Common errors

### Trial license cannot be activated. Error message: License file has not been specified. Please set the license in the extension settings.

If you are experiencing issues with activating the license, please try to enter the license file a couple of more times in the license input form, refreshing the web page in between. If it still does not work, please follow the below steps to generate some console output to help us with troubleshooting:
 
- What is the URL of the Azure DevOps organization you are trying to run the tool in? Is it coherent with the URL in the license file?
- Open the License input form under Organization Settings -> Azure DevOps Backup Tool
- Open the browser console (usually F12)
- Enter the license file contents and click "activate license"
- Screenshot the browser console and include any errors or warnings
- Send us the screenshot via email
