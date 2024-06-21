# Azure DevOps Backup and Restore Guide

## Introduction

This document provides detailed procedures for restoring your Azure DevOps (ADO) organization or project using the Azure DevOps Backup Tool. In the event that your source ADO organization or project becomes unavailable, this guide will help you recreate the components in your target organization.

### Process overview

1. **Creating and Setting Up the Git Repository for Mapping Files**
1. **Instructions to Map Azure DevOps Agent Queue IDs Before Restoring a Backup**
1. **Instructions to Map Azure DevOps Service Connections Before Restoring a Backup**
1. **Instructions to Map Azure DevOps Users Before Restoring a Backup**
1. **Setting Up the Restore Pipeline**

### Manual Mapping Process Overview

When restoring certain resources, we will need to provide mapping files to ensure that the tool can locate and reference all necessary infrastructure. Although the Azure DevOps Backup Tool infers most mappings automatically, manual mapping is required for the following resources:

- Users (identities)
- Agent Queues (project-level agent pools)
- Service Connections

The Azure DevOps Backup Tool will create some files to assist you with the manual mapping process. To retrieve these mapping files, you can publish the **workspace** folder as a pipeline artifact. In order to do this, you must add these tasks after your **Azure DevOps Backup Tool: Export** task in your pipeline:

```yml
- task: ArchiveFiles@2
  continueOnError: true
  inputs: 
    rootFolderOrFile: '$(workspace)'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber).zip'
    replaceExistingArchive: false

- task: PublishPipelineArtifact@1
  continueOnError: true
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)\$(Build.BuildNumber).zip'
    artifact: 'Export'
    publishLocation: 'pipeline'
```

**Note**: A full example of a working Restore pipeline is provided at the end of this document.

Within the published artifact, you will find a folder named **RESTORE**. This folder contains the following files, which can assist you in the mapping process (detailed documentation on each file will be provided later in this document):

- `identity_map.csv`
- `identities.csv`
- `queueid_map.csv`
- `queueIds.csv`
- `serviceconnection_map.csv`
- `serviceConnections.csv`

We strongly recommend that you use these files to facilitate the manual mapping of resources for an accurate and efficient restoration process.

## Creating and Setting Up the Git Repository for Mapping Files

1. **Create a New Git Repository**:
   - Create a new Git repository to hold the mapping files.
   - After going through this guide, the Git repository should contain the following files (the file names must match exactly):
     - `identity_map.csv`
     - `queueid_map.csv`
     - `serviceconnection_map.csv`

2. **Add and Commit the Mapping Files**:
   - Add the mapping files to the repository.
   - Commit the changes.
   - Example:
     ```sh
     git add identity_map.csv queueid_map.csv serviceconnection_map.csv
     git commit -m "Add mapping files for ADO restore"
     git push origin main
     ```

## Instructions to Map Azure DevOps Agent Queue IDs Before Restoring a Backup

When restoring a backup in Azure DevOps, it is crucial to map the agent queue IDs from the source project to the target project. This ensures that your pipelines can correctly reference the appropriate agent queues. Follow these steps to map the agent queue IDs:

### Files Provided by the Backup Job

1. **queueIds.csv**
   - This file contains the details of agent queues from the source project.
   - **Purpose**: Reference only. It lists the ID and name of each agent queue.
   - **Example**:
     ```
     "id","name"
     "1392","Default"
     "1393","Hosted"
     "1394","Hosted VS2017"
     "1395","Hosted macOS"
     ```

2. **queueid_map.csv**
   - This file is where you will map the IDs of the agent queues from the source project to the target project.
   - **Purpose**: To be filled in and committed to a Git repo before running the restore pipeline.
   - **Example**:
     ```
     1392,
     1393,
     1394,
     ```

### Steps to Map Agent Queue IDs

1. **Recreate or Share Agent Pools in the Target Project**:
   - Recreate the agent pools in the target project, or share the existing ones, ensuring they match the agent pools in the source project.
   - Use the `queueIds.csv` file to reference the names of the agent pools from the source project.

2. **Obtain IDs of the Agent Pools in the Target Project**:
   - Each agent pool in Azure DevOps has a unique ID. You need to map the source IDs to the corresponding target IDs.
   - To obtain the IDs of the agent pools in the target project, follow these instructions:

     **Manual Method**:
     - Navigate to the Azure DevOps portal.
     - Go to `Project Settings` > `Agent pools`.
     - Click on each agent pool to find its details, including the ID in the URL.

     **Programmatic Method Using PowerShell**:
     - You can fetch the agent pools programmatically using Azure DevOps REST API.

     ```powershell
     # PowerShell script to fetch agent pools in the target project
     $organization = "YourOrgName"
     $pat = "YourPAT"  # Personal Access Token
     $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($pat)"))

     $url = "https://dev.azure.com/$organization/_apis/distributedtask/pools?api-version=6.0"

     $response = Invoke-RestMethod -Uri $url -Method Get -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)}

     # Extract IDs and names
     $agentPools = $response.value | Select-Object id, name

     # Export to CSV for reference
     $agentPools | Export-Csv -Path "target_agentPools.csv" -NoTypeInformation

     # Display the agent pools
     $agentPools | ForEach-Object { "$($_.id),$($_.name)" }
     ```

3. **Map the IDs in queueid_map.csv**:
   - Open `queueid_map.csv` and fill in the target project’s IDs corresponding to the source IDs.
   - **Example**:
     ```
     <source ID>,<target ID>
     1392,1234
     1393,5678
     1394,9101
     ```

4. **Commit the queueid_map.csv**:
   - After filling in the `queueid_map.csv`, commit this file to the specified Git repository.

5. **Run the Restore Pipeline**:
   - Execute the restore pipeline. The pipeline will use the mappings in `queueid_map.csv` to map the agent queues correctly.

## Instructions to Map Azure DevOps Service Connections Before Restoring a Backup

When preparing to restore a backup in Azure DevOps, it is essential to map the service connections from the source project to the target project. This process ensures that your pipelines and other configurations continue to work seamlessly after the restore. Follow these steps to correctly map the service connections:

### Files Provided by the Backup Job

1. **serviceConnections.csv**
   - This file contains the details of service connections from the source project.
   - **Purpose**: Reference only. It lists the ID, name, type, and URL of each service connection.
   - **Example**:
     ```
     "id","name","type","url"
     "87fd1810-81ee-484f-a02e-8836d33cac3b","ContosoAir","git","https://github.com/Microsoft/ContosoAir.git"
     "1de71cea-4e5c-4ed9-967b-c1840f7a8f10","github.com_Alexander-Hjelm","github","https://github.com"
     ```

2. **serviceconnection_map.csv**
   - This file is where you will map the GUIDs of the service connections from the source project to the target project.
   - **Purpose**: To be filled in and committed to a Git repo before running the restore pipeline.
   - **Example**:
     ```
     87fd1810-81ee-484f-a02e-8836d33cac3b,
     1de71cea-4e5c-4ed9-967b-c1840f7a8f10,
     ```

### Steps to Map Service Connections

1. **Recreate Service Connections in the Target Project**:
   - Recreate the service connections in the target project with the same types as those in the source project.
   - Use the `serviceConnections.csv` file to reference the names, types, and URLs of the service connections from the source project.

2. **Obtain GUIDs of the Target Project’s Service Connections**:
   - Each service connection in Azure DevOps has a unique GUID. You need to map the source GUIDs to the corresponding target GUIDs.
   - To obtain the GUIDs of the service connections in the target project, follow these instructions:

     **Manual Method**:
     - Navigate to the Azure DevOps portal.
     - Go to `Project Settings` > `Service connections`.
     - Click on each service connection to find its details, including the GUID in the URL.

     **Programmatic Method Using PowerShell**:
     - You can fetch the service connections programmatically using Azure DevOps REST API.

     ```powershell
     # PowerShell script to fetch service connections in the target project
     $organization = "YourOrgName"
     $project = "YourTargetProjectName"
     $pat = "YourPAT"  # Personal Access Token
     $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($pat)"))

     $url = "https://dev.azure.com/$organization/$project/_apis/serviceendpoint/endpoints?api-version=6.0-preview.4"

     $response = Invoke-RestMethod -Uri $url -Method Get -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)}

     # Extract GUIDs and names
     $serviceConnections = $response.value | Select-Object id, name

     # Export to CSV for reference
     $serviceConnections | Export-Csv -Path "target_serviceConnections.csv" -NoTypeInformation

     # Display the service connections
     $serviceConnections | ForEach-Object { "$($_.id),$($_.name)" }
     ```

3. **Map the GUIDs in serviceconnection_map.csv**:
   - Open `serviceconnection_map.csv` and fill in the target project’s GUIDs corresponding to the source GUIDs.
   - **Example**:
     ```
     <source ID>,<target ID>
     87fd1810-81ee-484f-a02e-8836d33cac3b,abc12345-6789-def0-1234-56789abcdef0
     1de71cea-4e5c-4ed9-967b-c1840f7a8f10,12345678-90ab-cdef-1234-567890abcdef
     ```

4. **Commit the serviceconnection_map.csv**:
   - After filling in the `serviceconnection_map.csv`, commit this file to the specified Git repository.

5. **Run the Restore Pipeline**:
   - Execute the restore pipeline. The pipeline will use the mappings in `serviceconnection_map.csv` to map the service connections correctly.

## Instructions to Map Azure DevOps Users Before Restoring a Backup

When restoring a backup in Azure DevOps, it is crucial to map the user identities from the source project to the target project. This ensures that your project retains the correct user permissions and settings. Follow these steps to map the user identities:

### Files Provided by the Backup Job

1. **identities.csv**
   - This file contains the details of users from the source project.
   - **Purpose**: Reference only. It lists the ID, principal name, and display name of each user.
   - **Example**:
     ```
     "id","$_.user.principalName","$_.user.displayName"
     "844b3435-b85f-6229-b83a-2569d498b367","simon.liolios@solidify.se","Simon Liolios"
     "9db2d0e5-6a4b-607b-a298-25aab8990d10","Manuele@solidify.se","Manuel Ericstam"
     ```

2. **identity_map.csv**
   - This file is where you will map the principal names and GUIDs of the users from the source project to the target project.
   - **Purpose**: To be filled in and committed to a Git repo before running the restore pipeline.
   - **Example**:
     ```
     simon.liolios@solidify.se,12345678-abcd-1234-abcd-12345678abcd
     Manuele@solidify.se,23456789-abcd-2345-abcd-23456789abcd
     ```

### Steps to Map User Identities

1. **Onboard Users in the Target Organization**:
   - Ensure all necessary users are onboarded in the target Azure DevOps organization.
   - Use the `identities.csv` file to reference the users from the source project.

2. **Obtain GUIDs and Principal Names of Users in the Target Organization**:
   - Each user in Azure DevOps has a unique GUID and principal name. You need to map the source GUIDs to the corresponding target GUIDs.
   - To obtain the GUIDs and principal names of the users in the target project, follow these instructions:

     **Manual Method**:
     - Navigate to the Azure DevOps portal.
     - Go to `Organization settings` > `Users`.
     - Click on each user to find their details, including the GUID.

     **Programmatic Method Using PowerShell**:
     - You can fetch the users programmatically using Azure DevOps REST API.

     ```powershell
     # PowerShell script to fetch users in the target project
     $organization = "YourOrgName"
     $pat = "YourPAT"  # Personal Access Token
     $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($pat)"))

     $url = "https://vsaex.dev.azure.com/$organization/_apis/userentitlements?api-version=6.0-preview.3"

     $response = Invoke-RestMethod -Uri $url -Method Get -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)}

     # Extract GUIDs and principal names
     $users = $response.value | Select-Object @{Name="id";Expression={$_.user.id}}, @{Name="principalName";Expression={$_.user.principalName}}

     # Export to CSV for reference
     $users | Export-Csv -Path "target_users.csv" -NoTypeInformation

     # Display the users
     $users | ForEach-Object { "$($_.principalName),$($_.id)" }
     ```

3. **Map the GUIDs and Principal Names in identity_map.csv**:
   - Open `identity_map.csv` and fill in the target project’s GUIDs corresponding to the source principal names.
   - **Example**:
     ```
     <source ID>,<target ID>
     <source email>,<target email>
     simon.liolios@solidify.se,simon.liolios@solidify.se
     12345678-abcd-1234-abcd-12345678abcd,1d189893-085d-4cab-9949-f08b0d801cb9
     Manuele@solidify.se,Manuele@solidify.se
     23456789-abcd-2345-abcd-23456789abcd,bec2dfca-1e85-4cd4-b808-3f1f5356d8ef
     ```

4. **Commit the identity_map.csv**:
   - After filling in the `identity_map.csv`, commit this file to the specified Git repository.

5. **Run the Restore Pipeline**:
   - Execute the restore pipeline. The pipeline will use the mappings in `identity_map.csv` to map the user identities correctly.

## Setting Up the Restore Pipeline

Here is a sample YAML build pipeline demonstrating the restore functionality:

```yaml
trigger: none

variables:
- name: sourceUrl
  value: https://dev.azure.com/solidifydemo
- name: sourceOrg
  value: solidifydemo
- name: sourceProject
  value: ContosoAir
- name: targetProject
  value: 'ContosoAirMigrated'
- name: sourceUsername
  value: john.doe@solidify.dev
- name: workspace
  value: '$(System.DefaultWorkingDirectory)/MigrationWorkspace'
- name: System.Debug
  value: true

pool:
  vmImage: 'ubuntu-latest'

steps:
# Extract the snapshot from the backup server (here denoted with X:\)
- task: ExtractFiles@1
  inputs:
    archiveFilePatterns: 'X:\AdoBackupFolder\Backup-20230614.4.zip'
    destinationFolder: '$(workspace)'
    cleanDestinationFolder: true

# Run import job
- task: ado-backup-tool-import@1
  inputs:
    target: '$(sourceUrl)'
    sourceOrgName: '$(sourceOrg)'
    targetOrgName: '$(sourceOrg)'
    sourceProject: '$(sourceProject)'
    targetProject: '$(targetProject)'
    targetUsername: '$(sourceUsername)'
    sourcePAT: '$(migrationToken)'
    targetPAT: '$(migrationToken)'
    onPrem: false
    workspace: '$(workspace)'
    resourceWorkItem: true
    mappingFilePath: '$(System.DefaultWorkingDirectory)/mapping-files'
  env:
    SYSTEM_ACCESSTOKEN: $(system.accesstoken)
```

**Instructions for the Pipeline Task**:
1. **Point to the directory holding the Mapping Files**:
   - You can either commit the mapping files to the same **git repository as the restore pipeline**, or commit them to a **separate git repository** and include a step to clone the repository.
   - Use the task parameter **mappingFilePath** to sepcify the path to the .csv mapping files. The .csv files must be located at the root of this exact directory, and the file names must match exactly:
     - `identity_map.csv`
     - `queueid_map.csv`
     - `serviceconnection_map.csv`
