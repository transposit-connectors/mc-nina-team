# Debugging workflow commands

## Introduction

This runbook enumerates all of the actions available in the transposit-workflows [Workflow Commands](https://console.staging.transposit.com/mc/t/transposit-workflows/runbooks/workflow_slash_commands)



Note: because we don't pass the bodies of HTTP responses up to the calling workflows, the error messages you get in MC are often just "HostException: Response status not ok. Status code: 500". There may very well be an informative message in that HTTP response, though! Go look in the Monitor tab of the calling workflow.

## Forking a workflow

[Fork a global workflow](https://console.transposit.com/mc/t/nina-team/actions/fork_global_workflow)



The fork_working_copy operation can accept parameters to specify the working org and the working environment to fork to, but by default, these are set in mc_helper's environment variables.

I'm not sure if it's absolutely necessary, but in fact fork_working_copy both forks the template *and* syncs the workflow repo from GitHub, making sure that the configs deployed in the new copy match the GitHub config.yml file. 

## Creating a new workflow from a template

[Create a new workflow](https://console.transposit.com/mc/t/nina-team/actions/fork_template_workflow)



## Forking block_kit_lib

You can also fork block_kit_lib if you need to make changes to it:

[Fork block_kit_lib](https://console.transposit.com/mc/t/nina-team/actions/fork_block_kit_lib)



## Syncing a workflow to GitHub:

[Sync workflow](https://console.transposit.com/mc/t/nina-team/actions/sync_workflow)

Syncing a workflow is the process of pushing a working copy of a workflow from a Transposit workspace to its GitHub source-of-truth repository in transposit-connectors. The underlying operation won't let you sync a copy from the transposit workspace, because that workspace is meant for the currently-deployed versions and will get rewritten from the GitHub source-of-truth every day.



Sync_workflow uses the syncTranspositApp lambda (via connector_utilities.sync_multiple_transposit_apps) to perform git push on the specified workflow/integrator to its GitHub repo. By default, the lambda does a force push, so as not to deal with merge conflicts, etc. If something fails here, though, it's worth checking the logs (https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2#logsV2:log-groups/log-group/$252Faws$252Flambda$252FsyncTranspositApp) to see the details of what went wrong.



    - it saves out the deployed MC configurations to config.yml in the GitHub repo 
    - it regenerates the README and public-facing documentation for the workflow/integrator
    - it posts the results of everything to Slack.

## Validating a README:

[Validate README](https://console.transposit.com/mc/t/nina-team/actions/validate_readme)

When one is publicly releasing a workflow/integrator for the first time, it's a good idea to check to make sure that the automatically-generated README has valid information for all of its fields. The validate_readme action runs the validate_readme_for_service workflow, which calls a webhook that basically does the autogenerating-readme part of sync_workflow. It uses data from the manifest.json file to create a basic README file with the information a user would need to have, and provides a nice summary of the specific errors relating to data it couldn't find, like descriptions for environment variables, etc. If specific information pertaining to this particular workflow is needed, it should be provided in a setup.md file in the workflow repo; the generator will append it to the end of README.md.

After validating the README, one should re-sync the workflow to GitHub with the last action.

## Validating a service

(https://console.transposit.com/mc/t/transposit-eng/actions/validate_service)



## Deploy latest release

[Deploy latest release](https://console.transposit.com/mc/t/nina-team/actions/deploy_release)

While it is possible to deploy configurations based only on git tags, I think it is best practice for "real" deployments (i.e. final versions meant for customer use) to correspond to releases in the GitHub source-of-truth repo. This way we can make sure that we've written a bit of description about what important changes happened.



## Publish from GitHub to servers

[Publish from GitHub](https://console.transposit.com/mc/t/nina-team/actions/publish_to_github)

At this point, the GitHub version of the workflow/integrator should be absolutely the way you want it. If this service is already listed in dynamic_config, it'll be published from GitHub to the transposit workspace on all servers by the daily syncer. You can also manually trigger publication by running this action.



## Create PR for dynamic_config

[Make dynamic_config PR](https://console.transposit.com/mc/t/nina-team/actions/make_dynamic_config_pr)



## Sync a Transposit app

[Sync Transposit app](https://console.transposit.com/mc/t/nina-team/actions/sync_transposit_app)




## Create a connector

[Create a connector](https://console.transposit.com/mc/t/nina-team/actions/create_a_connector)





A stub `test_whatever` app will be created on demo, which you can fill in to make a test you can add to the test_runner app.

## Generate a README

[Generate README](https://console.transposit.com/mc/t/nina-team/actions/generate_readme)



## Sync a runbook

[Sync runbook](https://console.transposit.com/mc/t/nina-team/actions/sync_runbook)



The sync_runbook operation uses a GitHub repository named `mc-workspace_name` to perform the synchronization. If this repo doesn't exist, it first creates it. From there, it pushes the source url's workspace and the target url's workspace to the appropriate env-named branches at GitHub, to make sure those sources of truth are as up-to-date as possible. 

Then it takes the runbook md file at the GitHub source branch, updates the action links to point to the target workspace, and pushes that runbook file to the GitHub target branch. 

It also updates all of the linked action yaml files from the GitHub source branch and copies them to the GitHub target branch.

Finally, it uses the syncTranspositApp lambda, via connector_utilities.sync_transposit_app, to push the GitHub target branch to the targeted workspace.

The result is a link to the diff between the GitHub target branch before and after the sync, so you can easily see what changes were made.
