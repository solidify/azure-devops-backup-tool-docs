# Custom configuration files, usage

You can use custom json configuration files to gain finer control over your backup/migration jobs.

Common scenarios for using custom config files include:

- Filtering git repositories by name (wildcard supported)
- Filtering build/release pipelines by name (wildcard supported)
- Filtering work items based on a WIQL query
- Overriding default git credentials for git repositories and wikis

You will need to supply the **useCustomConfigurations** and **customConfigurationPath** parameters in the task configuration, like this:

```yml
useCustomConfigurations: true
customConfigurationPath: '$(System.DefaultWorkingDirectory)/custom-configs'
```

This will configure the task to read your custom json configuration files from the `$(System.DefaultWorkingDirectory)/custom-configs` directory.

Our recommendation is to store the config files in a git repository in the same project as the backup pipeline. You can either store the config files:

- in the same repository as the backup pipeline definitions (specify the correct folder)
- OR in a separate repository and checkout your configuration repository in the pipeline with the `resources` keyword.

You can view templates for individual configuration files for each ADO component here <https://github.com/solidify/azure-devops-backup-tool-docs/tree/main/config-templates>.

You can find documentation and usage samples with example configuration files for each ADO component here: <https://github.com/solidify/azure-devops-backup-tool-docs/tree/main/adapter-docs>

You config files must follow the naming pattern `config-[resource]-[export/import].json`. The following list shows all the valid config file names:

- `config-areapath-export.json`
- `config-areapath-import.json`
- `config-artifact-export.json`
- `config-artifact-import.json`
- `config-board-export.json`
- `config-board-import.json`
- `config-dashboard-export.json`
- `config-dashboard-import.json`
- `config-deploymentgroup-export.json`
- `config-deploymentgroup-import.json`
- `config-environment-export.json`
- `config-environment-import.json`
- `config-git-export.json`
- `config-git-import.json`
- `config-gitbranchpolicy-export.json`
- `config-gitbranchpolicy-import.json`
- `config-iterationpath-export.json`
- `config-iterationpath-import.json`
- `config-pipeline-export.json`
- `config-pipeline-import.json`
- `config-pullrequest-export.json`
- `config-pullrequest-import.json`
- `config-query-export.json`
- `config-query-import.json`
- `config-team-export.json`
- `config-team-import.json`
- `config-testplan-export.json`
- `config-testplan-import.json`
- `config-variablegroup-export.json`
- `config-variablegroup-import.json`
- `config-wiki-export.json`
- `config-wiki-import.json`
- `config-witprocess-export.json`
- `config-witprocess-import.json`
- `config-witprocessxml-export.json`
- `config-witprocessxml-import.json`
- `config-workitem-export.json`
- `config-workitem-import.json`

For configuration samples and more documentation on usage of custom configuration files and individual export/import adapters, visit <https://github.com/solidify/azure-devops-backup-tool-docs>.
