<p align="center">
    <h1 align="center">
        What can be found in this repository?
    </h1>
</p>

## 🚀 Automation

### Actions

- [**get-configurations**](../.github/actions/get-configurations/action.yml): action to extract configuration from a JSON file respecting the format described [here](./Repository-Setup.md#5---update-global-configurations)
- [**run-import-solutions**](../.github/actions/run-import-solutions/action.yml): action to create runs of the [**import-solution-to-dev**](../.github/workflows/import-solution-to-dev.yml) for a list of solutions passed as an input
- [**set-canvasapps-instrumentation-key**](../.github/actions/set-canvasapps-instrumentation-key/action.yml): action to set an Azure Application Insights Instrumentation Key we get from the considered custom deployment settings file (*format described [here](./Custom-Deployment-Settings-File-Management.md)*) in the source code of the canvas apps of the considered solution before packing for deployment

### Workflows

- [**workspace-initialization.yml**](../.github/workflows/workspace-initialization.yml):
   - Trigger: issue assigned or labeled (*we only consider the `in progress` label - or the one you decided to use*)
   - Summary of actions:
      - create a branch
      - create a Dataverse Dev environment (*using the **create-dataverse-environment** reusable workflow*)
      - add developers to the new Dataverse Dev environment as System Administrators
      - import the solution to the new Dataverse Dev environment, if there is one in the repository
      - add comments to the issue to give visibility on the progress of the initialization of the workspace
- [**export-and-unpack-solution**](../.github/workflows/export-and-unpack-solution.yml):
   - Trigger: manual with inputs
   - Summary of actions:
      - export and unpack the considered solution
      - push the changes to the considered branch (*based on input value*)
- [**solution-quality-check-on-pr**](../.github/workflows/solution-quality-check-on-pr.yml):
   - Trigger: pull request tarteging the `main` branch and with changes on specific folders
   - Summary of actions:
      - get and set variables
      - create a just-in-time Dataverse Build environment (*using the **create-dataverse-environment** reusable workflow*)
      - pack the considered solution
      - execute the solution checker on the considered solution and generate an execute condition if thresholds are not met
      - test the solution type conversion (unmanaged to managed) using the just-in-time Dataverse Build environment
      - delete the just-in-time Dataverse Build environment
- [**import-solution-to-dev**](../.github/workflows/import-solution-to-dev.yml):
   - Trigger: manual with inputs (*runs created from [**run-import-solutions**](../.github/actions/run-import-solutions/action.yml) action called from the [**workspace-initialization.yml**](../.github/workflows/workspace-initialization.yml) workflow*)
   - Summary of actions:
      - set solution version (*version passed in an input*)
      - pack solution as unmanaged
      - import solution to a Dataverse Dev environment (*environment url passed in an input*)
      - add a comment to the considered issue (*issue number passed in an input*)
- [**import-solution-to-validation**](../.github/workflows/import-solution-to-validation.yml):
   - Trigger: push to the `main` branch with changes on specific folders
   - Summary of actions:
      - get and set variables
      - create a just-in-time Dataverse Build environment (*using the **create-dataverse-environment** reusable workflow*)
      - build a managed version of the solution using the just-in-time Build environment (*using the **build-managed-solution** reusable workflow*)
      - import the managed version of the considered solution to the Dataverse Validation environment (*using the **import-solution** reusable workflow*)
      - delete the just-in-time Dataverse Build environment
- [**create-deploy-release**](../.github/workflows/create-deploy-release.yml):
   - Trigger: manual with inputs
   - Summary of actions:
      - get and set variables
      - create a branch for the considered release
      - create a just-in-time Dataverse Build environment (*using the **create-dataverse-environment** reusable workflow*)
      - build a managed version of the solution using the just-in-time Build environment (*using the **build-managed-solution** reusable workflow*)
      - delete the just-in-time Dataverse Build environment
      - delete the release branch if deployment of the considered solution to the just-in-time Dataverse Build environment fails
      - create a GitHub release as draft with the unmanaged and managed versions of the considered solution
      - import the managed version of the considered solution to the Dataverse Production environment (*using the **import-solution** reusable workflow*)
      - publish the GitHub release initialized earlier
- [**clean-dev-workspace-when-issue-closed-or-deleted**](../.github/workflows/clean-dev-workspace-when-issue-closed-or-deleted.yml):
   - Trigger: issue closed or deleted
   - Summary of actions:
      - delete branch created for the development regarding the considered issue
      - delete the Dataverse Dev environment created for the considered issue
      - add a comment to the issue to give a status on the workspace created for the development

#### Reusable workflows

- [**create-dataverse-environment**](../.github/workflows/create-dataverse-environment.yml):
   - Triggers: called by another workflow (*reusable workflow*)
   - Summary of actions:
      - set some variables for dynamic display and domain name (*based on a condition on the value of an input*)
      - create the Dataverse environment (*2 methods considered and the choice is made based on the value of an input*)
      - update considered issue (*based on a condition on the value of an input*)
- [**build-managed-solution**](../.github/workflows/build-managed-solution.yml):
   - Triggers: called by another workflow (*reusable workflow*)
   - Summary of actions:
      - set some variables: path to packed solution (*zip*) and solution version
      - update some elements of the unpacked solution in the repository: cloud flows JSON files, pack canvas apps and update solution version in Solution.xml file
      - pack the solution from the considered branch in the repository
      - import solution as unmanaged to just-in-time Build environment and export it as managed
      - store packed solution(s) in GitHub artifact store for later user in calling workflow
- [**import-solution**](../.github/workflows/import-solution.yml):
   - Triggers: called by another workflow (*reusable workflow*)
   - Summary of actions:
      - get packed solution to import from GitHub artifact store
      - import solution to considered environment
      - execute post solution import steps (*turn on cloud flows and share canvas apps*)

#### Other workflows

- [**PSScriptAnalyzer**](../.github/workflows/powershell-analysis.yml):
   - Triggers:
      - push to the `main` branch with changes on specific folders
      - pull request tarteging the `main` branch and with changes on specific folders
   - Summary of actions:
      - run PSScriptAnalyzer on the PowerShell code in the repository
      - upload the generated sarif file for analysis

### PowerShell scripts

- [**Add-AADSecurityGroupTeamToDataverseEnvironment**](../Scripts/Add-AADSecurityGroupTeamToDataverseEnvironment.ps1): Add an Azure AD Security Group Team to a Dataverse environment
- [**Add-UserToDataverseEnvironment**](../Scripts/Add-UserToDataverseEnvironment.ps1): Add a user to a Dataverse environment (*not used anymore - replaced by **Add-AADSecurityGroupTeamToDataverseEnvironment***)
- [**Enable-CloudFlows**](../Scripts/Enable-CloudFlows.ps1): Turn on the Cloud Flows in a specific solution in a targeted Dataverse environment
- [**Grant-GroupsAccessToCanvasApps**](../Scripts/Grant-GroupsAccessToCanvasApps.ps1): Grant access to canvas apps to Azure AD groups based on a mapping in a configuration file

## 🧾 Configurations

- [**Configurations/configurations.json**](../Configurations/configurations.json): global configurations file used to simplify the management of information required in the GitHub workflows ([details](./Repository-Setup.md#5---update-global-configurations))

## 🖼 Issue / Pull request templates

- [**BUG.yml**](../.github/ISSUE_TEMPLATE/BUG.yml): issue template to allow people to report bugs
- [**config.yml**](../.github/ISSUE_TEMPLATE/config.yml): issues configuration to also present redirections to discussions for ideas and q&a when creating an issue in addition to the bug option
- [**pull_request_template.md**](../.github/pull_request_template.md): pull request template to have a minimum of consistency in the pull requests created in this repository

<h3 align="center">
  <a href="../README.md#-documentation">🏡 README - Documentation</a>
</h3>