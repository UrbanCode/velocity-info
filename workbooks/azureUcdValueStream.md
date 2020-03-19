# Creating an Azure and UrbanCode Deploy (UCD) Value Stream

## Objective
Create a value stream with Azure and UrbanCode Deploy integrations and exercise it from beginning (a new issue) to end (deployment to Prod).

## Workbook Guidelines

**< > Placeholders/Substitution**
Placeholders should be indicated with angle brackets "<" and ">". For instance, if the workbook says navigate to `"https://dev.azure.com/\<your org name\>"`, and your Azure organization name is "SkyScubaDiving", then navigate to `"https://dev.azure.com/SkyScubaDiving"`.

**🔀 Alternative Paths**
Workbooks should guide users beyond rote exercise towards exploration and discovery. As you go through the workbook, there are certain steps intended to simplify the process; however, some alternative paths will be called out along the way.

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=8 orderedList=false} -->

<!-- code_chunk_output -->
- [Creating an Azure and UrbanCode Deploy (UCD) Value Stream](#creating-an-azure-and-urbancode-deploy-ucd-value-stream)
  - [Objective](#objective)
  - [Workbook Guidelines](#workbook-guidelines)
- [1. Setup Azure](#1-setup-azure)
  - [1.1 Create Workbook Project](#11-create-workbook-project)
  - [1.2 Initialize Workbook Repository](#12-initialize-workbook-repository)
  - [1.3 Setup Workbook Pipeline](#13-setup-workbook-pipeline)
  - [1.4 Create an Azure Access Token](#14-create-an-azure-access-token)
- [2. Setup UCD](#2-setup-ucd)
  - [2.1 Create Workbook Component](#21-create-workbook-component)
  - [2.1.1 Configure Component](#211-configure-component)
  - [2.1.2 Create Component Process](#212-create-component-process)
  - [2.2 Create Workbook Application](#22-create-workbook-application)
  - [2.2.1 Create Environments](#221-create-environments)
  - [2.2.2 Create Application Process](#222-create-application-process)
  - [2.3 Create UCD Access Token](#23-create-ucd-access-token)
- [3. Setup Velocity](#3-setup-velocity)
  - [3.1 Create a Velocity User Access Key](#31-create-a-velocity-user-access-key)
  - [3.2 Setup UCD within Velocity](#32-setup-ucd-within-velocity)
  - [3.2.1 Create UCD Integration](#321-create-ucd-integration)
  - [3.2.2 Upgrade UCD Integration](#322-upgrade-ucd-integration)
  - [3.2.3 Add user to UCD Team](#323-add-user-to-ucd-team)
  - [3.3 Setup Azure within Velocity](#33-setup-azure-within-velocity)
  - [3.3.1 Create Azure Integration](#331-create-azure-integration)
  - [3.3.1 Upgrade Azure Integration](#331-upgrade-azure-integration)
  - [3.4 Create a Dummy Integration](#34-create-a-dummy-integration)
  - [3.5 Create a New Value Stream](#35-create-a-new-value-stream)
  - [3.5.1 Create a vsm.json file](#351-create-a-vsmjson-file)
  - [3.5.2 Upload the vsm.json File](#352-upload-the-vsmjson-file)
  - [3.6 Configure the Value Stream Pipeline](#36-configure-the-value-stream-pipeline)
  - [3.6.1 Add UCD to the Pipeline](#361-add-ucd-to-the-pipeline)
  - [3.6.2 Map Environments](#362-map-environments)
- [4. Use Value Stream: Planning to Merged](#4-use-value-stream-planning-to-merged)
  - [4.1 Backlog (create a new work item)](#41-backlog-create-a-new-work-item)
  - [4.3 In Progress](#43-in-progress)
  - [4.4 In Review](#44-in-review)
  - [4.4.1 Create a new Branch](#441-create-a-new-branch)
  - [4.4.2 Make Changes to the Branch](#442-make-changes-to-the-branch)
  - [4.4.3 Commit Changes](#443-commit-changes)
  - [4.4.4 Create Pull Request](#444-create-pull-request)
  - [4.4.5 Observe Dot](#445-observe-dot)
  - [4.5 Merging Changes](#45-merging-changes)
- [5. Use Value Stream: Deployment](#5-use-value-stream-deployment)
  - [5.1 Create a UCD snapshot](#51-create-a-ucd-snapshot)
  - [5.1 Deploy to Dev](#51-deploy-to-dev)
  - [5.2 Deploy to QA](#52-deploy-to-qa)
  - [5.3 Deploy to Prod](#53-deploy-to-prod)

<!-- /code_chunk_output -->

# 1. Setup Azure

Requirements: 
  1. Azure account with ability to create a new project.

## 1.1 Create Workbook Project 

Navigate to `https://dev.azure.com/<your org name>` to create a new project. To learn more about creating Azure projects visit  [https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project](https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project)

Configure your new project as follows:
  - *Project Name:* AzureWorkbook
  - *Visibility:* Private
  - *Version Control:* Git
  - *Work item process:* Agile

![](azure/new-project-2.png)

(🔀 Alternative: See how far you can get with an existing project)

## 1.2 Initialize Workbook Repository

In your new project, navigate to "Repos" to create a new repository. It can be the bare minimum, so just "initialize with a README or gitignore".

![](azure/new-repo-1.png)

## 1.3 Setup Workbook Pipeline

We also want an Azure pipeline in this workbook to represent our build step. We need to set that up with a little secret sauce: we want successful runs to tag our repo. You can learn more about Azure pipelines at [https://docs.microsoft.com/en-us/azure/devops/pipelines/create-first-pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/create-first-pipeline).

1. Navigate to Pipelines and click "Create Pipeline". For the type of repository select "Azure Repos Git" and select the workbook repository we just created "AzureWorkbook".
![](azure/new-pipeline-2.png)
![](azure/new-pipeline-3.png)

2. The default "hello world" .yml is good enough for this workbook's pipeline. Click save (avoid running for now).
![](azure/new-pipeline-4.png)

3. After the pipeline is created we need to setup repository tagging. Click "edit" from the pipeline view.
![](azure/new-pipeline-5.png)

4. After clicking edit you should see the yml. Click on the vertical ellipsis in the upper right beside "Run" and click "Triggers" from the dropdown.
![](azure/new-pipeline-6.png)

5. There's a lot to unpack under "Triggers", but we're going to be laser focussed on one thing right now. Click "YAML" on the left of the upper horizontal tabs. Click "Get Sources" under YAML. Scroll down in the menu that shows on the right. Under "Tag sources" click "On Success". Go ahead and click "Save & queue". Our pipeline should now create repository tags for successful builds, and since we queued it, we should have a build pending.
![](azure/new-pipeline-7.png)

6. Confirm that the pipeline ran successfully and created a tag for the repository.
![](azure/repo-tags-1.png)

## 1.4 Create an Azure Access Token

Navigate to `https://dev.azure.com/<your org name>/_usersSettings/tokens`.

Create a new token. Give it a reasonable name an expiration date for your usage. The easiest option is to select "Full Access". Make sure to copy and save your key (you'll need it later in Velocity).

![](azure/access-token-2.png)

# 2. Setup UCD

UCD should already be setup with an available agent. We will setup a component and application for this workbook.

## 2.1 Create Workbook Component

Create a component for this workbook.

## 2.1.1 Configure Component
First obtain the repository URL and generated credentials for this workbook's Azure git repo.

![](\azure\repository-url-1.png)
![](\azure\repository-url-2.png)


Create an AzureWorkbook component within UCD. It is important that this component be configured as follows:


  - **Name:** something like AzureWorkbookComponent
  - **Source Configuration Type:** Git
  - **Repository URL:** `https://dev.azure.com/<org name>/AzureWorkbook/_git/AzureWorkbook` (note: if copied from Azure you will probably need to remove a user name portion "\<user name\>@")
  - **User name:** Copied from Azure git repo's generated credentials
  - **Password:** Copied from Azure git repo's generated credentials
  - Check **"Watch for Tags"**
  - Check **"Import Versions Automatically"**

![](\ucd\azureWorkbook\component-config-1.png)
![](\ucd\azureWorkbook\component-config-2.png)
![](\ucd\azureWorkbook\component-config-3.png)

Confirm that the component version was imported from the Azure tag we created when we ran the Azure pipeline earlier.

![](\ucd\azureWorkbook\component-versions.png)

## 2.1.2 Create Component Process

We just need a bare minimum deploy process for this workbook. Make sure it is "Process Type" Deployment. The process itself will just be a one second wait (The wait step is available under utilities).

![](\ucd\azureWorkbook\component-process-1.png)
![](\ucd\azureWorkbook\component-process-2.png)


## 2.2 Create Workbook Application

Create an application and name it "AzureWorkbook".

## 2.2.1 Create Environments

Create three environments. You can use the same base resource for each environment. It should contain an agent with the workbook component we just created.

![](\ucd\azureWorkbook\application-env-1.png)

## 2.2.2 Create Application Process

Create a bare minimum install process for the application to install our component.

![](\ucd\azureWorkbook\application-process-1.png)
![](\ucd\azureWorkbook\application-process-2.png)


## 2.3 Create UCD Access Token

Navigate to `<UCD URL>/#security/tokens` ("Settings" --> "Tokens") and click "Create Token". Make sure to copy and save token to use later.

![](\ucd\azureWorkbook\create-token.png)


# 3. Setup Velocity

Requirements: 

1. Access to Velocity user with full permissions.


## 3.1 Create a Velocity User Access Key

Navigate to `<Velocity URL>/settings/myprofile` to create a user access key. Name the key so you can identify and delete it later if needed. Make sure to copy the access key for future use.

![](velocity/access-key-1.png)

## 3.2 Setup UCD within Velocity

## 3.2.1 Create UCD Integration

Navigate to `<Velocity URL>/settings/integrations`. Click "Plugins" and click "Add Integration" for the "UrbanCode Deploy" plugin. Fill out the fields as shown below:

  - **Name:** something like UcdWorkbook
  - **UrbanCode Deploy Server URL:** Your UCD server URL. Note that if Velocity is running from a Docker-Compose instance replace "localhost" and/or 127.0.0.1 with "host.docker.internal".
  - **UrbanCode Deploy Access Token:** The UCD access token we previously created.]

![](\velocity\add-integration-ucd.png)

## 3.2.2 Upgrade UCD Integration

After creating the integration, click on the vertical ellipses on the right of the integration's row and click "upgrade" if available.

![](\velocity\integration-vertical-ellipses.png)

## 3.2.3 Add user to UCD Team

UCD teams are imported into Velocity. Make sure that your Velocity user has access to the UCD resources for this workbook.

![](\velocity\add-user-to-team-ucd.png)

## 3.3 Setup Azure within Velocity

## 3.3.1 Create Azure Integration

Navigate to `<Velocity URL>/settings/integrations`. Click "Plugins" and click "Add Integration" for the Azure plugin.

![](velocity/add-integration-azure.png)

  - *Integration Name:* Use "AzureWorkbook" for this workbook.
  - *Loggin level:* "INFO"
  - *UrbanCode Velocity User Access Key:* Use the velocity user access key that was just created in section 2.1 above.

![](velocity/add-integration-azure-form-1.png)

  - *URL:* This example uses https://dev.azure.com
  - *Username:* What you use to login to Azure. Usually an email address.

![](velocity/add-integration-azure-form-2.png)

  - *Password:* leave blank (using access token instead)
  - *Access Token:* Paste the Azure access token we created earlier.

![](velocity/add-integration-azure-form-3.png)

  - *Organization:* You Azure organization name. Can be obtained and confirmed with your Azure home URL: `https://dev.azure.com/<your org name>`.
  - *Project:* Your Azure project name. Use "AzureWorkbook" for the workbook. This can be derived and confirmed against the project URL: `https://dev.azure.com/<your org name>/<project name>`
  - *Repositories:* You Azure repository (repo) name. Use "AzureWorkbook" for the workbook. This can be derived and confirmed against the repo URL: `https://dev.azure.com/<your org name>/_git/<repo name>`

![](velocity/add-integration-azure-form-4.png)

Depending on your version of Velocity you might see an extra field for "Logger Level" at the button of the form. You can leave this blank.

![](velocity/add-integration-azure-form-5.png)

## 3.3.1 Upgrade Azure Integration

After creating the integration, click on the vertical ellipses on the right of the integration's row and click "upgrade" if available.

![](\velocity\integration-vertical-ellipses.png)

## 3.4 Create a Dummy Integration

Because of the way we are going to configure our value stream, this workbook requires a dummy integration. Add another integration from the integrations page by Clicking on "Plugins" and "Add Integration" for GitHub for instance. Name the integration dummyIntegration. The rest of the configuration can be anything so long as it gets created. We should now have two integrations: "AzureWorkbook" and "dummyIntegration".

## 3.5 Create a New Value Stream

  1. Navigate to the *Value Streams* page and click "Create"   
![Jenkins Global Config Page](valueStream/newValueStream/1.png)
  2. Name your value stream and select a team for it. The workbook uses the name **"Azure Workbook"** and **"Default Team"**. A value stream description is optional.   
![Jenkins Global Config Page](valueStream/newValueStream/form-Azure.png)


| Field | Description | Required | 
|-------|-------------|----------|
| Name | value stream name. | yes |
| Description | value stream description | no | 
| Team | Team of users who can access this value stream. | yes | 

## 3.5.1 Create a vsm.json file

UrbanCode Velocity value streams are fully configurable by downloading and uploading a value stream map (VSM) json file. Save the vsm.json content provided below as a .json file. We will then upload this file to configure our value stream.

> **Because this json content references integrations by name, the integrations must be created and named correctly before uploading it.**

```json
{
  "_id": "5e702311e1819863a523107a",
  "pipelineId": "83225f55-d456-4de1-af65-636eeb7916a9",
  "tenantId": "5ade13625558f2c6688d15ce",
  "description": null,
  "query": null,
  "phases": [
    {
      "name": "Planning",
      "description": null,
      "stages": [
        {
          "name": "Backlog-Demonstration",
          "query": "issue.status='New'",
          "description": null,
          "wipLimit": null,
          "gates": null
        }
      ]
    },
    {
      "name": "Development",
      "description": null,
      "stages": [
        {
          "name": "In Progress",
          "query": "(issue.status=\"Active\" and pr.status!=\"active\" and pr.status!=\"completed\")",
          "description": null,
          "wipLimit": null,
          "gates": null
        },
        {
          "name": "In Review",
          "query": "pr.status=active and issue.status!=\"Closed\"",
          "description": null,
          "wipLimit": null,
          "gates": null
        }
      ]
    },
    {
      "name": "Build",
      "description": null,
      "stages": [
        {
          "name": "Merged",
          "query": "pr.status = 'completed' and deployment.env != 'DEV' and deployment.env != 'QA' and deployment.env != 'PROD'",
          "description": null,
          "wipLimit": null,
          "gates": null
        }
      ]
    },
    {
      "name": "Deploy",
      "description": null,
      "stages": [
        {
          "name": "DEV",
          "query": "deployment.env = 'DEV' and deployment.env != 'QA' and deployment.env != 'PROD' and pr.status = 'completed'",
          "description": null,
          "wipLimit": null,
          "gates": null
        },
        {
          "name": "QA",
          "query": "deployment.env = 'QA' and deployment.env != 'PROD' and pr.status = 'completed'",
          "description": null,
          "wipLimit": null,
          "gates": null
        },
        {
          "name": "PROD",
          "query": "deployment.env = 'PROD' and pr.status = 'completed'",
          "description": null,
          "wipLimit": null,
          "gates": null
        }
      ]
    }
  ],
  "leadTime": null,
  "cycleTime": null,
  "mappings": null,
  "integrations": [
    {
      "name": "AzureWorkbook"
    },
    {
      "name": "dummyIntegration"
    }
  ],
  "linkRules": [
    {
      "fromIntegrationName": "AzureWorkbook",
      "toIntegrationName": "AzureWorkbook",
      "fromField": "pr.name",
      "toField": "issue.id",
      "pattern": "([0-9]+)"
    }
  ],
  "metrics": null,
  "metricsBar": null
}
```

> **_Explanation of VSM JSON_**
> - **Integrations Array**   
> This vsm.json content defines an integration for Jira. Integrations are included based on integration name. This workbook uses the names **"AzureWorkbook"** and **"dummyIntegration"**. This part of the vsm.json needs changed if the integrations happen to be named differently.
>```json
>"integrations":[
>    {
>      "name":"AzureWorkbook"
>    },
>    {
>      "name":"dummyIntegration"
>    }
>]
> ```
>    
>    
> - **Phases and Stages**  
> Value streams are organized as phases and stages. The json file provides phase and stage definitions. An important part of stages are stage queries which logically define whether a work item (dot) should be included in a stage or not.
> ```javascript
>"phases": [
>  {
>    "name": "Planning",
>    "description": null,
>    "stages": [
>      {
>        "name": "Backlog-Demonstration",
>        "query": "issue.status='New'",
>        "description": null,
>        "wipLimit": null,
>        "gates": null
>      }
>    ]
>  }
>]
>```
> ![Value Stream Graphic](valueStream/phasesStages.png)
>
>
>  - **Link Rules**
>  Link rules are important for this workbook. They define how pull requests (PRs) get linked to work items. In this case based on the regex pattern `([0-9]+)`.
>
>
>```javascript
>  "linkRules": [
>    {
>      "fromIntegrationName": "AzureWorkbook",
>      "toIntegrationName": "AzureWorkbook",
>      "fromField": "pr.name",
>      "toField": "issue.id",
>      "pattern": "([0-9]+)"
>    }
>  ]
>```
>


## 3.5.2 Upload the vsm.json File

If you have a brand-new value stream, then the upload button will be directly available.

![](jira/uploadVsm.png)

After the first vsm.json file is uploaded, the value stream can be modified with additional uploads via the tools and utilities dropdown option "Replace value stream map". 

<img src="valueStream/vsmUploadDownload.png" alt="drawing" style="width:300px;"/>

After uploading the vsm.json file you should see the appropriate phases and stages in your value stream:

![](AzureValueStream/1-dot-nothing.png)


## 3.6 Configure the Value Stream Pipeline

## 3.6.1 Add UCD to the Pipeline

On the Value Stream page, click "Pipeline" and click "Add App", then  choose "UrbanCode Deploy" from the dropdown. You will also need to select the application and process for this workbook.

![](\velocity\pipeline-add-ucd.png)

## 3.6.2 Map Environments

Click on the "+" icon under each environment to map a UCD environment to the Velocity pipeline environment.

![](\velocity\pipeline-with-ucd.png)

![](\velocity\pipeline-env-mapping.png)

# 4. Use Value Stream: Planning to Merged

Shew, that was a lot of setup. Now let's get that dot moving!

> **Pro Tip**: We're about to do a lot of external activity to Velocity. To speed things up, you can force Velocity's Azure integration to sync by clicking disable/enable every time we change an external state (like create a work item or merge a PR)
>
> ![](AzureValueStream/integration-sync.png)


## 4.1 Backlog (create a new work item)

As observed above, our value stream has a bunch of stages but no dots, that's perfectly accurate for our empty board in Azure. Click on "New Item" to create a new work item.

![](AzureValueStream/2-new-work-item-1.png)

You can title this work item anything you like, the example below names it "do the workbook" and assigns it a work item ID of "5". **Take note of this work item ID (5 is shown here but for a first time workbook it's probably 1), we'll need it later.**

![](AzureValueStream/2-new-work-item-2.png)

Once Velocity syncs up with Azure (disable/enable integration if you don't want to wait) you will see the dot show up in your backlog. Neat!

![](AzureValueStream/2-dot-backlog.png)

but why is it red, it seems a little upset 😡?... Aww, because it has an alert: "Issue not assigned".

![](AzureValueStream/2b1.png)

Go ahead and assign it to someone back in Azure. There's nothing like a little delegating to get the ball rolling (even if it is to yourself).

![](AzureValueStream/2b2.png)

Now our dot is a happy little dot 😇 (and we see the beginnings of its history)

![](AzureValueStream/2b3.png)

## 4.3 In Progress

Turns out, we are in fact doing the workbook :) so grab that "do the workbook" card from the backlog and move it to "active".

![](AzureValueStream/3-card-active.png)

Wait for Velocity to sync (or force with disable/enable). The dot will move to "In Progress".

![](AzureValueStream/3-dot-in-progress.png)

## 4.4 In Review

Now that the card is active, we have to do some actual work to move the dot to review. That's a good thing; Velocity is a lot more than just moving cards around!

## 4.4.1 Create a new Branch

Navigate to branches in Azure and click "New branch".

![](AzureValueStream/4-new-branch-1.png)

The branch name doesn't really matter; you can just name it "workbook". Click create.

## 4.4.2 Make Changes to the Branch

![](AzureValueStream/4-new-branch-2.png)

Once the branch is created we can directly edit the `README.md` file from Azure. You can make any changes you want, like change the "Introduction" section to say "workbook". 

![](AzureValueStream/4-edit-branch.png)

## 4.4.3 Commit Changes

Once you've made changes click "Commit".

![](AzureValueStream/4-make-commit.png)

## 4.4.4 Create Pull Request

After committing changes our branch is now different than master. That's great, we've added value! Click "Create a Pull Request".

![](AzureValueStream/4-create-pr-1.png)

For the most part, the default values are fine for the Pull Request (PR), except one critical piece: **we need the work item ID to be in the PR title**. This is because we configured our value stream back in Velocity to link work items to PRs based on the work item ID in the PR title. The example here shows "5" but the ID for a new Azure project will be "1". Once correctly titled, click "Create" to create the PR.

![](AzureValueStream/4-create-pr-2.png)

## 4.4.5 Observe Dot

Wait for Velocity to sync (or disable/enable integration). The dot will move from "In Progress" to "In Review".

![](AzureValueStream/4-dot-in-review.png)

## 4.5 Merging Changes

You have reviewed your changes and lo, they are beyond perfect, even sublime. Your PR is approved, go ahead and merge (click "Complete").

![](AzureValueStream/5-merge-1.png)

The default values for "Complete pull request" are fine, click "Complete merge".

![](AzureValueStream/5-merge-2.png)

Wait for Velocity to sync. The dot will move to "Merged".

![](AzureValueStream/5-dot-merged.png)

# 5. Use Value Stream: Deployment

## 5.1 Create a UCD snapshot

In UCD, for the workbook application, click on the play icon on the left of the DEV environment to request a process.

![](\ucd\env.png)

We will run our "deploy" process that installs our component. We need to choose a component version though; click "Choose Versions".

![](\ucd\env-request-process.png)

We should be fine with choosing "Latest Available" for our version. This should give us the version we synced over to UCD from our Azure git tag created by our Azure pipeline.

![](\ucd\env-request-process-version.png)

The process should run and deploy the component. Click the camera icon for DEV to take a snapshot of the deployment.

![](\ucd\env.png)

## 5.1 Deploy to Dev

From UCD, click the play icon again except this time let's use our snapshot for the deployment.

![](\ucd\env.png)

Go back to Velocity and give it time to sync. The dot will move to the deployment stage for DEV.

## 5.2 Deploy to QA

From Velocity, go to our pipeline. Click the play icon for QA. Give it time to run and sync, then navigate back to the value stream and observe the dot in QA.

## 5.3 Deploy to Prod

Finally, use the Velocity pipeline to deploy to PROD. Observe the dot move to PROD. We have completed the value stream at this point from planning to production.
