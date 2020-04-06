# Continuous Integration Demo

This repository has several different python and yaml files designed to automate various continuous integration workflows on Looker projects using Spectacles, pyLookML, and Looker’s python SDK.

These files are meant as inspiration for seasoned developers who want to minimize  the number of errors that end up in front of end-users due to changing data sources, evolving schema, human error in SQL development, and general model chaos that can result from several developers working on a single project over several years.

Feel free to use and modify this code as you wish, but do note that this is not officially supported by Looker, so DCL won’t  be able to help you redevelop or troubleshoot your code. Instead, please post any questions you might have here or in Looker’s discourse community!

________________________________

# Overview

There are three general workflows contained in this repository. These are:

Spectacles - automatically run SQL validation tests on your LookML project upon opening a pull request (spectacles.yml)

pyLookML - periodically schedule the auto-generation of lookML for new fields in your database schema (generate_lookml.py + generate_lookml.yml)

Looker Python SDK - periodically run the content validator and send the results to Slack in order to find broken Looker content before your end users do (content_validate.yml  + content_validate.py)


All three of these workflows are automated using github actions (more on those here: https://github.com/features/actions). The associated .yml files spin up VMs through github, install all specified dependencies, and run the code with the variables specified in these steps. Where those variables are sensitive ( Looker API credentials and slack webhook), the actual values were added as secrets to the git repository and referenced as variables (see these steps: https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets).

All three workflows include sending notifications to slack channels using a slack webhook (that can be generated following these steps: https://api.slack.com/messaging/webhooks#posting_with_webhooks ), but these are not required for the underlying tools to run. 

For our demo, we used github actions free tier. Information on github action’s free tier can be found here: https://help.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions

________________________________

# General Requirements

These workflows assume you have access to your own git repository (as opposed to a looker-hosted git repository) and have pull requests enabled on your Looker project. To add these workflows to your project, you will need to add a .github folder to the git repository where your project lives, as well as a ‘workflows’ folder within that hidden .github folder - so the .yml files would all live in .github/workflows as demonstrated in this repository. You also need to add the following 3  secrets to your git repository:

`my_client_id` - Looker API client ID
`my_client_secret` - Looker API client Secret
`SLACK_WEBHOOK` - Slack webhook generated following these steps: https://api.slack.com/messaging/webhooks#posting_with_webhooks

For specifics on how to add  those secrets to your git repository, see these steps: https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets.

You will also need to make  some modifications to the files themselves in order to specify your own instance + credentials. Those modifications will be called out in the following sections.

________________________________

# Spectacles

Spectacles is an open-source continuous integration tool designed to perform a variety of tests on a Looker branch before it is pushed to production in order to reduce the amount of errors that make it in front of end-users as a result of changes in the LookML model or upstream schema. More information on how it works here: https://spectacles.dev/

The file spectacles.yml is a workflow that automatically runs spectacle's SQL validation tests upon opening a pull request on your Looker project. Once the validation tests complete, a notification of the success or failure of the event is posted in slack.

In order to stand this yml file up on your own instance, you will need to insert:
      your own base URL ( --base-url https://<my_looker_instance_url>)
      Your own project name (--project <my_project>)

________________________________

# pyLookML

pyLookML is a metaprogramming interface for the LookML language. It allows you to programmatically manipulate lookML files in an object oriented environment using python. More information about how it works here: https://pylookml.readthedocs.io/en/latest/introduction.html

The file generate_lookml.py must be updated with the query that contains the fields needed to generate the new LookML fields (model, explore, fields). This python script is designed to either create or append the new fields to the file ‘custom_events.view.lkml’. 

The file generate_lookml.yml schedules the generate_lookml.py file to run via a cron schedule and notify a slack channel upon generating the new fields.

In order to stand this yml file up on your own instance, you will need to insert:
      your own base URL ( --base-url https://<my_looker_instance_url>)
      Your own project name ( --project <my_project>)
      Your own repo path ( --repo <user_or_organization>/<repo_name> )
      Your own project name ( --project <project_name> > generate.txt)


________________________________

# Looker Python SDK

Looker’s python SDK can be used to call Looker’s API to run just about any Looker workflow you might want. In this case,  it allows us to run the content validator programmatically, letting you find broken content before your end users do. More on the python SDK as a whole can be found here: https://github.com/looker-open-source/sdk-codegen/tree/master/python

The file content_validation.py does not require any modifications to run on your instance. It calls the content  validator  and pipes the results of this to several text files.
The file content_validation.yml schedules the content_validation.py to  run on a cron schedule and notify a slack channel based on those results.

In order to stand this yml file up on your own instance, you will need to insert:
      your own base URL ( --base-url https://<my_looker_instance_url>)
      Your own url link in the slack payload ( "text": "Check your content in the <https:/<my_looker_instance_url>/content_validator|Content Validator>." ) 



