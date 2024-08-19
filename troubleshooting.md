# Troubleshooting guide

## Point of contact

You can submit an error report by emailing us at <support.adobackup@solidify.dev>.

## Troubleshooting guidelines

If the **Azure DevOps Backup Tool** fails to run properly, the problem can sometimes be remedied with some of the below steps:

1. Run the pipeline with `$(System.Debug) = 'true'` (See the above section in this document).
1. Manually clear the workspace on the agent server and run the pipeline again.
1. Ensure that two or more backup jobs are not running concurrently on the same machine.
1. Ensure that the service account running the build agent has Read/Write access to the workspace folder.
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

## Common errors

### Trial license cannot be activated. Error message: License file has not been specified. Please set the license in the extension settings.

If you are experiencing issues with activating the license, please try to enter the license file a couple of more times in the license input form, refreshing the web page in between. If it still does not work, please follow the below steps to generate some console output to help us with troubleshooting:
 
- What is the URL of the Azure DevOps organization you are trying to run the tool in? Is it coherent with the URL in the license file?
- Open the License input form under Organization Settings -> Azure DevOps Backup Tool
- Open the browser console (usually F12)
- Enter the license file contents and click "activate license"
- Screenshot the browser console and include any errors or warnings
- Send us the screenshot via email
