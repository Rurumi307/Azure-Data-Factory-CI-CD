# Azue Data Factory CI / CD 實作

[[_TOC_]]

## CI/CD lifecycle

  ![CI/CD lifecycle][data-factory-CI/CD]

## Prerequisites

- Azure Data Factory  
  Create three Data Factories environment(dev/qas/prd)
- Azure DevOps Project  
  Create Git Repository in Azure DevOps for Azure Data Factory

## Link Dev Data Factory to DevOps Repo

- Open Azure Data Factory Studio
- **Manage > Git configuration > Configure**  
![Link DevOps Repo][data-factory-link-devops-repo-connect-git-repo]

- Configure a repository  
  - Repository Type: **Azure DevOps Git**
  - Microsoft Entra ID: Select your Azure Active Directory tenant name
  - Click **`Continue`**  

  ![Link DevOps Repo][data-factory-Link-DevOps-Repo-configure-repository]

  - Azure DevOps organization name:  
  Select the organization name that was created during the `Creating an Azure DevOps organization and repo` step.
  - Project name: **ADF**, Select your Project name
  - Repository name: **DG-ADF-DevOps**, Select your Repository name
  - Collaboration branch: **master**, Select your Branch Name
  - Publish branch: **adf_publish**
  - Root folder: **/**
  - Since we have pipelines in our Data Factory, we want to import all the source code into the repo. We do this by 'check' the 'Import existing resources to repository and select **`master`** as the branch.
  - Click **`Apply`**.

  ![Link DevOps Repo][data-factory-link-devops-repo-configure-repository-2]
- Set working branch  
  **Use Existing**> Select the Working Branch **`master`**> Click **`Save`**  
  ![Link DevOps Repo][data-factory-link-devops-repo-set-working-branch]
- Check Link Dev Data Factory to DevOps Repo  
  Open Azure Data Factory Studio and check it is connected to Git  
  **Manage > Git configuration > Configure**  
  ![Link DevOps Repo][data-factory-link-devops-repo-check-setting]
- Open Azure DevOps
- Check Azure DevOps Repository  
  ![Link DevOps Repo][data-factory-link-devops-repo-check-devops-setting]

## Publish and generates ARM templates

- Open Azure Data Factory Studio
- **Author > Check branch > Publish**

  ![ADF-Publish][data-factory-first-publish]  
  ![ADF-Publish][data-factory-first-publish-pending-changes]  
  This Publish operation also generates ARM templates  
  ![ADF-Publish][data-factory-first-publish-generates-ARM-templates]

- Open Azure DevOps  
Check will be saved to adf_publish branch in Git  
**Repository > adf_publish**

  ![ADF-Publish][data-factory-first-publish-generates-ARM-templates-2]
- ARMTemplateForFactory.json:  
  will contain all the assets within our Data Factory and all their properties.
- ARMTemplateParametersForFactory.json:  
  will contain all the parameters used in any of the assets within our DEV Data Factory.

## Creating an Azure DevOps Pipeline

- Open Azure DevOps  

### Pre- and post-deployment PowerShell Script

> There is an **extremely important** `pre-and post-deployment` PowerShell script that needs to be run during each release. Without this script, if you were to delete a pipeline in your DEV Data Factory and deploy the current state of DEV into UAT and PROD, that deletion would NOT happen in UAT and PROD.

> You can find this script in the [Azure Data Factory documentation][R003] at the bottom of the page. Copy it over to your favorite code editor and save it as a PowerShell file. I called mine `ci-cd.ps1`.

- Azure DevOps > Repos > **master** branch > more actions > **Upload file(s)**  
![DevOps Pipeline][data-factory-devops-repo-ps1-pre-and-post-deployment]
![DevOps Pipeline][data-factory-devops-repo-upload-file]

### customize data factory triggers across environments with azure devops pipelines

- create another Powershell script which shall be named `triggerupdate.ps1`

  ![DevOps Pipeline][data-factory-devops-repo-ps1-triggerupdate]
- paste script

  <details>
    <summary><u>triggerupdate.ps1</u> (<i>click to show</i>)</summary>
  
  ```shell
  param
  (
      [parameter(Mandatory = $true)] [String] $TriggerConfigFilePath,
      [parameter(Mandatory = $true)] [String] $ResourceGroupName,
      [parameter(Mandatory = $true)] [String] $DataFactoryName
  )

  Write-Host "get trigger config json file from " $TriggerConfigFilePath
  $TriggerConfigJSON = Get-Content $TriggerConfigFilePath | ConvertFrom-Json

  Write-Host "classify triggers "
  $triggersToDelete = $TriggerConfigJSON.triggers | Where-Object { $_.action -eq "delete" }     | Select $_.name 
  $triggersToStop   = $TriggerConfigJSON.triggers | Where-Object { $_.action -eq "deactivate" } | Select $_.name 
  $triggersToStart  = $TriggerConfigJSON.triggers | Where-Object { $_.action -eq "activate" }   | Select $_.name 

  Write-Host "     triggers to delete : "     $triggersToDelete.name
  Write-Host "     triggers to deactivate : " $triggersToStop.name
  Write-Host "     triggers to activate : "   $triggersToStart.name

  # delete triggers
  Write-Host "delete triggers"
  $triggersToDelete | ForEach-Object {
      Write-Host "      delete trigger "  $_.Name
      $trig = Get-AzDataFactoryV2Trigger -name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
      if ($trig.RuntimeState -eq "Started") {
          if ($_.TriggerType -eq "BlobEventsTrigger" -or $_.TriggerType -eq "CustomEventsTrigger") {
              Write-Host "Unsubscribing trigger" $_.Name "from events"
              $status = Remove-AzDataFactoryV2TriggerSubscription -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
              while ($status.Status -ne "Disabled"){
                  Start-Sleep -s 15
                  $status = Get-AzDataFactoryV2TriggerSubscriptionStatus -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
              }
          }
          Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force 
      }
      Remove-AzDataFactoryV2Trigger -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
  }

  # deactivate triggers
  Write-Host "deactivate triggers"
  $triggersToStop | ForEach-Object {
      Write-Host "      deactivate trigger "  $_.Name
      $trig = Get-AzDataFactoryV2Trigger -name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
          if ($_.TriggerType -eq "BlobEventsTrigger" -or $_.TriggerType -eq "CustomEventsTrigger") {
              Write-Host "Unsubscribing trigger" $_.Name "from events"
              $status = Remove-AzDataFactoryV2TriggerSubscription -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
              while ($status.Status -ne "Disabled"){
                  Start-Sleep -s 15
                  $status = Get-AzDataFactoryV2TriggerSubscriptionStatus -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
              }
          }
          Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force 
  }

  # activate triggers
  Write-Host "activate triggers"
  $triggersToStart | ForEach-Object {
      Write-Host "      activate trigger "  $_.Name
      $trig = Get-AzDataFactoryV2Trigger -name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
          if ($_.TriggerType -eq "BlobEventsTrigger" -or $_.TriggerType -eq "CustomEventsTrigger") {
              Write-Host "Subscribing trigger" $_.Name "to events"
              $status = Remove-AzDataFactoryV2TriggerSubscription -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
              while ($status.Status -ne "Enabled"){
                  Start-Sleep -s 15
                  $status = Get-AzDataFactoryV2TriggerSubscriptionStatus -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
              }
          }
          Start-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force 
  }


  Write-Host "all triggers have been configured according to TriggerConfig.json file"
  ```

  </details>

- create the **trigger-config-json-files** for each data factory under the same release folder, Make sure to name it according to this pattern:  
**[datafacoryname]_TriggerConfig.json**

  ```json
  {
      "triggers": [
          
      ]
  }
  ```

  ![DevOps Pipeline][data-factory-devops-repo-trigger-config]

### Creating the Azure Pipeline for CI/CD

- Pipelines > Releases > New pipeline  
  ![DevOps Pipeline][data-factory-devops-create-pipeline]
- select Empty job  
  ![DevOps Pipeline][data-factory-devops-create-pipeline-empty-job]
- Pick a name for the pipeline as well as for the stage (QAS/PROD)  
  ![DevOps Pipeline][data-factory-devops-create-pipeline-stage]

### Setting artifacts

- Add two artifacts to the pipeline, select Azure Repository for branch `master` and `adf_publish`
  1. Click **`Add an artifact`**  
  ![DevOps Pipeline][data-factory-devops-pipeline-add-an-artifact]
  2. Link to `adf_publish`  
  ![DevOps Pipeline][data-factory-devops-pipeline-add-an-artifact-adf_publish]
  3. Link to `master`  
  ![DevOps Pipeline][data-factory-devops-pipeline-add-an-artifact-master]

### Setting Continuous deployment trigger

  > Enabling the trigger will create a new release every time a Git push happens to the selected repository.

- **`Enable` > Add > Include > adf_publish**  
  ![DevOps Pipeline][data-factory-devops-pipeline-continuous-deployment-trigger]

### Set the variables [datafactory] [resourcegroup]  

  > Your data factories are probably in different resource groups. In that case you need to specify them per Scope, too.  

  |Name|Value|Scope|
  |:---|-----|-----|
  |datafactoryName|DG-QAS-ADF|QAS|
  |resourceGroupName|EBGIT-DIV2-DataFactory-QAS|QAS|
  |datafactoryName|DG-PRD-ADF|Prod|
  |resourceGroupName|EBGIT-DIV2-DataFactory|Prod|

  ![DevOps Pipeline][data-factory-devops-pipeline-set-variables]

### Set Task Agent job (QAS, Prod)

> **Note:** In this section, the steps for the QAS and Prod tasks are the same

- Click Tasks > QAS / Prod

#### 1. Adding an Azure Powershell task with the name stop triggers

- Script Path:
  > Click on the ellipsis to the right of `Script Path` and select the PowerShell script we uploaded earlier. It will be in the Artifact drop.

  ![DevOps Pipeline][data-factory-devops-pipeline-task-stop-triggers-script-Path]

  **Or Paste Script**

  ```shell
  $(System.DefaultWorkingDirectory)/_adf_master_source/ci-cd.ps1
  ```

- Script Arguments:  

  ```shell
  -ArmTemplate "$(System.DefaultWorkingDirectory)/_adf_publish_source/DG-DEV-ADF/ARMTemplateForFactory.json" 
  -ResourceGroupName $(resourceGroupName) 
  -DataFactoryName $(datafactoryName) 
  -PreDeployment $true 
  -DeleteDeployment $false 
  -ArmTemplateParameters "$(System.DefaultWorkingDirectory)/_adf_publish_source/DG-DEV-ADF/ARMTemplateParametersForFactory.json"
  ```

- check other setting  
![DevOps Pipeline][data-factory-devops-pipeline-task-stop-triggers-2]
![DevOps Pipeline][data-factory-devops-pipeline-task-stop-triggers-other-setting]

#### 2. Adding an ARM template deployment task

- with the name `ARM Template deployment: Resource Group scope`  
  ![DevOps Pipeline][data-factory-devops-pipeline-task-ARM-template-deployment]


- set Azure Details for qas  
  ![DevOps Pipeline][data-factory-devops-pipeline-task-ARM-azure-detail-qas]
- set Template for qas
  - Template:

    ```shell
    $(System.DefaultWorkingDirectory)/_adf_publish_source/DG-DEV-ADF/ARMTemplateForFactory.json
    ```

  - Template parameters:

      ```shell
      $(System.DefaultWorkingDirectory)/_adf_publish_source/DG-DEV-ADF/ARMTemplateParametersForFactory.json
      ```

  - Override template parameters:  
      在不同環境中，重新定義或替換原有的模板參數，使其參數與dev不同(Linked services, Integration runtimes, Global parameters)

      ```shell
      -factoryName $(datafactoryName)
      -AzureKeyVault_ADF_properties_typeProperties_baseUrl "https://ebg-adf-qas-vault.vault.azure.net/"
      -DC_Sandbox_properties_typeProperties_connectionString_secretName "dc-sandbox-connection-string"
      -DG_properties_typeProperties_connectionString_secretName "dg-connection-string"
      -PDM_PRD_IECW9_properties_typeProperties_connectionString_secretName "PDM-PRD-IECW9-connection-string"
      -PDM_PRD_TAOC9_properties_typeProperties_connectionString_secretName "PDM-PRD-TAOC9-connection-string"
      -PDM_PRD_TAOW9_properties_typeProperties_connectionString_secretName "PDM-PRD-TAOW9-connection-string"
      -mx_actionlog_db_properties_typeProperties_connectionString_secretName "mx-actionlog-db-connection-string"
      -DG_SCM_0900_DailyTrigger_properties_SCM_Inventory_Pipeline_parameters_END_DT_CHAR "@formatDateTime(trigger().scheduledTime, 'yyyyMMdd')"
      -OnPremisesIntegrationRuntime_properties_typeProperties_linkedInfo_resourceId "/subscriptions/f40c2b20-df12-4cca-a001-9b2681e59747/resourcegroups/EBGIT-DIV2-DataFactory/providers/Microsoft.DataFactory/factories/ebg-div2-datafactory/integrationruntimes/OnPremisesIntegrationRuntime"
      -default_properties_environment_value "qas"
      -default_properties_subscription_id_value "bb4a4ba1-2924-4ac2-ac7c-fba6e6f3e74e"
      -default_properties_resource_group_name_value $(resourceGroupName)
      ```

  ![DevOps Pipeline][data-factory-devops-pipeline-task-ARM-template-setting-qas]

#### 3. Adding an Azure Powershell task with the name start triggers

- Script Path:

  ```shell
  $(System.DefaultWorkingDirectory)/_adf_master_source/ci-cd.ps1
  ```

- Script Arguments:  

  ```shell
  -ArmTemplate "$(System.DefaultWorkingDirectory)/_adf_publish_source/DG-DEV-ADF/ARMTemplateForFactory.json" -ResourceGroupName $(resourceGroupName) 
  -DataFactoryName $(datafactoryName) 
  -PreDeployment $false 
  -DeleteDeployment $true
  ```

- check other setting  
![DevOps Pipeline][data-factory-devops-pipeline-task-start-triggers]
![DevOps Pipeline][data-factory-devops-pipeline-task-stop-triggers-other-setting]

#### 4. Adding an Azure Powershell task with the name Change config of triggers

  > 依照不同環境TriggerConfig.json設定開關trigger

  ![DevOps Pipeline][data-factory-devops-pipeline-task-change-config-of-triggers]

- Script Path:

  ```shell
  $(System.DefaultWorkingDirectory)/_adf_master_source/triggerupdate.ps1
  ```

- Script Arguments:

  ```shell
  -TriggerConfigFilePath "$(System.DefaultWorkingDirectory)/_adf_master_source/$(datafactoryName)_TriggerConfig.json" -ResourceGroupName $(resourceGroupName) 
  -DataFactoryName $(datafactoryName) 
  ```

#### 5. Set Pre-deployment conditions

- Click Pre-deployment conditions  
![DevOps Pipeline][data-factory-devops-pipeline-set-Pre-deployment-conditions]

- Triggers: **select after release**  
![DevOps Pipeline][data-factory-devops-pipeline-Pre-deployment-conditions-triggers]

- Pre-deployment approvals:
  - Enable  
  ![DevOps Pipeline][data-factory-devops-pipeline-Pre-deployment-approvals]
  - approvals setting  
  ![DevOps Pipeline][data-factory-devops-pipeline-Pre-deployment-approvals-set]

#### 6. Save Pipeline

- Click **`Save`**

## Show Time

> Now that we have completed the crucial parts of our Release Pipeline, we can now test it. We will manually initiate the Azure Data Factory-CI pipeline which will trigger our Release Pipeline.

### Create Wait Pipeline in Azure Data Factory

- Open Azure Data Factory Studio(DG-DEV-ADF)
- Author > Click branch > New branch  
![ADF CI/CD][data-factory-create-new-branch]
- Give branch name  
![ADF CI/CD][data-factory-give-branch-name]

- Add New resource > Pipeline > Pipeline  
![ADF CI/CD][data-factory-create-wait-pipeline]
- select Activities: **wait**  
![ADF CI/CD][data-factory-create-wait-activities]
- Add trigger  
![ADF CI/CD][data-factory-add-trigger]  
![ADF CI/CD][data-factory-add-trigger-2]  
![ADF CI/CD][data-factory-add-trigger-3]
- Save

### Create PR feature branch into Master Branch

- Author > Click branch > Create pull request  
![ADF CI/CD][data-factory-create-PR]
- Create pull request  
![ADF CI/CD][data-factory-create-PR-into-master]
![ADF CI/CD][data-factory-create-PR-complete]
![ADF CI/CD][data-factory-create-PR-complete-merge]

### Set Trigger Config

> trigger1 在 PRD 不要執行, 設置deactivate  
> trigger1 在 QAS 執行, 設置activate  
> trigger1 在 DEV 執行, 設置activate  

![ADF CI/CD][data-factory-devops-repo-set-trigger-config]

- DG-DEV-ADF_TriggerConfig.json

  ```json
  {
      "triggers": [
          {"name":"trigger1", "action":"activate"}
      ]
  }
  ```

- DG-QAS-ADF_TriggerConfig.json

  ```json
  {
      "triggers": [
          {"name":"trigger1", "action":"activate"}
      ]
  }
  ```

- DG-PRD-ADF_TriggerConfig.json

  ```json
  {
      "triggers": [
          {"name":"trigger1", "action":"deactivate"}
      ]
  }
  ```

### Publish master into adf_publish

- Open Azure Data Factory Studio(DG-DEV-ADF)
- Set working branch
![ADF CI/CD][data-factory-set-working-branch-2]
- Click **publish**  
![ADF CI/CD][data-factory-click-publish]  
![ADF CI/CD][data-factory-publishing]  
![ADF CI/CD][data-factory-pending-changes]

### Deployment To QAS / PRD

- Open Azure DevOps
- Release > Pipeline(ADF-CD) > Release-1  
![ADF CI/CD][data-factory-devops-releases]
- Approve  
![ADF CI/CD][data-factory-devops-releases-approve]

### Check DG-PRD-ADF Trigger Status

- Open Azure Data Factory Studio(DG-PRD-ADF)
- Manage > Triggers > Check Status  
![ADF CI/CD][data-factory-ckeck-trigger-status]

## References

### [MS-Learn: Continuous integration and delivery in Azure Data Factory][R001]  
### [iterationInsights: Azure Data Factory CI/CD with DevOps Pipelines][R002]  
### [tacky tech: how to customize data factory triggers across environments with azure devops pipelines][R004]

[R001]: https://learn.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery
[R002]: https://iterationinsights.com/article/azure-data-factory-ci-cd-with-devops-pipelines
[R003]: https://github.com/Azure/Azure-DataFactory/blob/main/SamplesV2/ContinuousIntegrationAndDelivery/PrePostDeploymentScript.Ver2.ps1
[R004]: https://www.tackytech.blog/how-to-customize-data-factory-triggers-across-environments-with-azure-devops-pipelines/
[data-factory-CI/CD]: https://learn.microsoft.com/en-us/azure/data-factory/media/continuous-integration-delivery/continuous-integration-image12.png
[data-factory-link-devops-repo-connect-git-repo]: /azure/azure-data-factory/adf-ci-cd//data-factory-link-devops-repo-connect-git-repo.png
[data-factory-link-devops-repo-configure-repository]: /azure/azure-data-factory/adf-ci-cd//data-factory-link-devops-repo-configure-repository.png
[data-factory-link-devops-repo-configure-repository-2]: /azure/azure-data-factory/adf-ci-cd//data-factory-link-devops-repo-configure-repository-2.png
[data-factory-link-devops-repo-set-working-branch]: /azure/azure-data-factory/adf-ci-cd//data-factory-link-devops-repo-set-working-branch.png
[data-factory-link-devops-repo-check-setting]: /azure/azure-data-factory/adf-ci-cd//data-factory-link-devops-repo-check-setting.png
[data-factory-link-devops-repo-check-devops-setting]: /azure/azure-data-factory/adf-ci-cd//data-factory-link-devops-repo-check-devops-setting.png
[data-factory-first-publish]: /azure/azure-data-factory/adf-ci-cd//data-factory-first-publish.png
[data-factory-first-publish-pending-changes]: /azure/azure-data-factory/adf-ci-cd//data-factory-first-publish-pending-changes.png
[data-factory-first-publish-generates-ARM-templates]: /azure/azure-data-factory/adf-ci-cd//data-factory-first-publish-generates-ARM-templates.png
[data-factory-first-publish-generates-ARM-templates-2]: /azure/azure-data-factory/adf-ci-cd//data-factory-first-publish-generates-ARM-templates-2.png
[data-factory-devops-repo-ps1-pre-and-post-deployment]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-repo-ps1-pre-and-post-deployment.png
[data-factory-devops-repo-upload-file]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-repo-upload-file.png
[data-factory-devops-repo-ps1-triggerupdate]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-repo-ps1-triggerupdate.png
[data-factory-devops-repo-trigger-config]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-repo-trigger-config.png
[data-factory-devops-create-pipeline]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-create-pipeline.png
[data-factory-devops-create-pipeline-empty-job]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-create-pipeline-empty-job.png
[data-factory-devops-create-pipeline-stage]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-create-pipeline-stage.png
[data-factory-devops-pipeline-add-an-artifact]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-add-an-artifact.png
[data-factory-devops-pipeline-add-an-artifact-adf_publish]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-add-an-artifact-adf_publish.png
[data-factory-devops-pipeline-add-an-artifact-master]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-add-an-artifact-master.png
[data-factory-devops-pipeline-continuous-deployment-trigger]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-continuous-deployment-trigger.png
[data-factory-devops-pipeline-set-variables]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-set-variables.png
[data-factory-devops-pipeline-task-stop-triggers-2]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-stop-triggers-2.png
[data-factory-devops-pipeline-task-stop-triggers-script-Path]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-stop-triggers-script-Path.png
[data-factory-devops-pipeline-task-stop-triggers-other-setting]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-stop-triggers-other-setting.png
[data-factory-devops-pipeline-task-ARM-template-deployment]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-ARM-template-deployment.png
[data-factory-devops-pipeline-task-ARM-azure-detail-qas]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-ARM-azure-detail-qas.png
[data-factory-devops-pipeline-task-ARM-template-setting-qas]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-ARM-template-setting-qas.png
[data-factory-devops-pipeline-task-start-triggers]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-start-triggers.png
[data-factory-devops-pipeline-task-change-config-of-triggers]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-task-change-config-of-triggers.png
[data-factory-devops-pipeline-set-Pre-deployment-conditions]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-set-Pre-deployment-conditions.png
[data-factory-devops-pipeline-Pre-deployment-conditions-triggers]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-Pre-deployment-conditions-triggers.png
[data-factory-devops-pipeline-Pre-deployment-approvals]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-Pre-deployment-approvals.png
[data-factory-devops-pipeline-Pre-deployment-approvals-set]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-pipeline-Pre-deployment-approvals-set.png
[data-factory-create-new-branch]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-new-branch.png
[data-factory-give-branch-name]: /azure/azure-data-factory/adf-ci-cd//data-factory-give-branch-name.png
[data-factory-create-wait-pipeline]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-wait-pipeline.png
[data-factory-create-wait-activities]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-wait-activities.png
[data-factory-add-trigger]: /azure/azure-data-factory/adf-ci-cd//data-factory-add-trigger.png
[data-factory-add-trigger-2]: /azure/azure-data-factory/adf-ci-cd//data-factory-add-trigger-2.png
[data-factory-add-trigger-3]: /azure/azure-data-factory/adf-ci-cd//data-factory-add-trigger-3.png
[data-factory-create-PR]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-PR.png
[data-factory-create-PR-into-master]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-PR-into-master.png
[data-factory-create-PR-complete]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-PR-complete.png
[data-factory-create-PR-complete-merge]: /azure/azure-data-factory/adf-ci-cd//data-factory-create-PR-complete-merge.png
[data-factory-devops-repo-set-trigger-config]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-repo-set-trigger-config.png
[data-factory-set-working-branch-2]: /azure/azure-data-factory/adf-ci-cd//data-factory-set-working-branch-2.png
[data-factory-click-publish]: /azure/azure-data-factory/adf-ci-cd//data-factory-click-publish.png
[data-factory-publishing]: /azure/azure-data-factory/adf-ci-cd//data-factory-publishing.png
[data-factory-pending-changes]: /azure/azure-data-factory/adf-ci-cd//data-factory-pending-changes.png
[data-factory-devops-releases]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-releases.png
[data-factory-devops-releases-approve]: /azure/azure-data-factory/adf-ci-cd//data-factory-devops-releases-approve.png
[data-factory-ckeck-trigger-status]: azure/azure-data-factory/adf-ci-cd//data-factory-ckeck-trigger-status.png