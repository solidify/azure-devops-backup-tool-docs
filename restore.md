### Instructions to Map Azure DevOps Agent Queue IDs Before Restoring a Backup

When restoring a backup in Azure DevOps, it is crucial to map the agent queue IDs from the source project to the target project. This ensures that your pipelines can correctly reference the appropriate agent queues. Follow these steps to map the agent queue IDs:

#### Files Provided by the Backup Job

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

#### Steps to Map Agent Queue IDs

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
     1392,1234
     1393,5678
     1394,9101
     ```

4. **Commit the queueid_map.csv**:
   - After filling in the `queueid_map.csv`, commit this file to the specified Git repository.

5. **Run the Restore Pipeline**:
   - Execute the restore pipeline. The pipeline will use the mappings in `queueid_map.csv` to map the agent queues correctly.

By following these steps, you will ensure that all agent queues are accurately mapped, allowing for a smooth restoration of your Azure DevOps project.

### Instructions to Map Azure DevOps Service Connections Before Restoring a Backup

When preparing to restore a backup in Azure DevOps, it is essential to map the service connections from the source project to the target project. This process ensures that your pipelines and other configurations continue to work seamlessly after the restore. Follow these steps to correctly map the service connections:

#### Files Provided by the Backup Job

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

#### Steps to Map Service Connections

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
     87fd1810-81ee-484f-a02e-8836d33cac3b,abc12345-6789-def0-1234-56789abcdef0
     1de71cea-4e5c-4ed9-967b-c1840f7a8f10,12345678-90ab-cdef-1234-567890abcdef
     ```

4. **Commit the serviceconnection_map.csv**:
   - After filling in the `serviceconnection_map.csv`, commit this file to the specified Git repository.

5. **Run the Restore Pipeline**:
   - Execute the restore pipeline. The pipeline will use the mappings in `serviceconnection_map.csv` to map the service connections correctly.

By following these steps, you will ensure that all service connections are accurately mapped, allowing for a smooth restoration of your Azure DevOps project.
