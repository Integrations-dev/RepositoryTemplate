# RepositoryTemplate 📔
This is the Template Repository in which the GitHub Action can perform well on cloning.

Users can efficiently utilise the CliqInformer  by cloning the Repository using the commands. 

```
git clone https://<GITHUB PERSONAL ACCESS TOKEN>@github.com/Integrations-dev/RepositoryTemplate
```

## Generating Personal Access Token 🗝️
The **Personal Access Token** can be created either
  - from the **Tokens Section** of **Settings** at https://github.com/settings/tokens
  - or Click **Profile :arrow_right: Settings :arrow_right: Developer Settings :arrow_right: Personal Access Tokens :arrow_right: Tokens (classic)**

and Click on Generate new Token and select the Expiration Period and Select the Scope Required.

Since we will be pushing the code to the Repository, A Scope 🛡️ of **public_repo** is required for public repositories and **repo** is required for all (private and public) repositories.

Once cloned , you are all ready to utilise the CliqInformer GitHub Aciton.

## Custom Event Messages ⚙️

We have added a few custom messages you can try by removing the comment, which is as simple as removing the **hash (#)**. 

Suppose you need a notification in Cliq for only selected events or actions,
  - you may change the '**_on_**' key of the YAML File where the Action is called.
  - Or additionally, you can set all the required messages and declare the input '**set-message-if-none**' as '**false**' to avoid messages from events you don't want. 
  
You provide default messages to all kind of actions that triggers a workflow.

The messages can be customized by giving the input '**_event_-message**' (where '_event_' is the event for which you would like to customize the message).

For ex: To set a custom message for a Pull Request event, you must define the input as,

```yaml
  pull-request-message: 'A Pull Request has been Opened'
```

## Default Message 📓

If you wish to add a single custom message for all kinds of events, you may use the '**default-message**'. 

For ex: To set a default custom message , you must define the _default-message_ as

```yaml
  default-message: 'A (event) has been (action)'
```

## Shortcuts ⏩

We also provide several shortcuts to obtain the variables that you want to insert in the message, such as,
  - **(event)**: which will be replaced with the event that the workflow is triggered by
  - **(action)**: which will be replaced by the Action the Event is performing with
  - **(me)**: which will be replaced with the GitHub user performing the action.
  - **(repo)**: which will be replaced by the Repository where the GitHub action is performed
  - **(ref)**: which will be replaced by the Branch/Tag where the GitHub action is performed
  - **(workflow)**: which will be replaced by the workflow on which the GitHub Action is performed
  - **(rule)**: which will be replaced by the Branch Protection Rule (if the Event is Branch Protection Rule)
  - **(run)**: which will be replaced by the Check Run (if the Event is Check Run)
  - **(branch)**: which will be replaced by the Branch/Tag which is Created/Deleted (if the Event is Create / Delete)
  - **(deployment)**: which will be replaced by the deployment (if the Event is Deployment or Deployment Status)
  - **(discussion)**: which will be replaced by the discussion which is worked on
  - **(category)**: which will be replaced by the Category Name to which the discussion is changed to
  - **(issue)**: which will be replaced by the issue (if the Event is Issue or Issue Comment)
  - **(label)**: which will be replaced by the label that is being worked on or added to
  - **(milestone)**: which will be replaced by the milestone that is being worked on or added to
  - **(assignee)**: which will be replaced by the User Assigned to the Issue or Pull Request
  - **(pull)**: which will be replaced by the Pull Request that is worked on
  - **(package)**: which will be replaced the Registry Package that is being worked on
  - **(release)**: which will be replaced by the release that is being worked on
  - **(status)**: which will be replaced by the Status of the Event (if the Event is Deployment Status or Status)

## GitHub Secret for Channel Endpoint 🔗
You must add GitHub Secrets, which contains the channel endpoint in the format 

ENDPOINT
```
<Cliq Channel Endpoint>?zapikey=<Cliq Webhook Token>
```

You must create a GitHub Secret for providing the Channel Endpoint  by
  - Go to the Repository where the CliqInformer will be added and go to the '**Settings**' tab.
  - Select '**Secrets and variables**' and click on '**Actions**' in the dropdown.
  - Click on '**New repository secret**' and enter the name as '**ENDPOINT**' and also enter the Cliq channel endpoint  as the Secret (in above mentioned format)

## Base YAML Code 🗒

Don't worry about remembering a lot of stuff. Here is the minimal code that's required to start with. 

```yaml
name : Communicating with Cliq

on:
  #you may add the events you like to get notified
  push:
    
jobs:
  test_name:
    runs-on: ubuntu-latest
    steps:
      - uses: Integrations-dev/CliqInformer@v1
        with:
          channel-endpoint: ${{ secrets.ENDPOINT }}
```

That's all! You will start getting notified for each event occurring in GitHub through the GitHub Action.

Go to the Actions tab of the Repository to view the Message status.
