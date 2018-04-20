![Publish Status](https://codebuild.eu-west-1.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoibGhwMFZSeEN0V3kzV1pDWmRiYkZKbWVDLzFORG81aytqbm93WWU4aTJpVUxNOVp4UTk1WklZakJIdWt3RDFnUEpVbHY5MlU2MEl3ZG5qMjRIS2tVVWo0PSIsIml2UGFyYW1ldGVyU3BlYyI6IklnYlFBaEtyZkVoaHFkWkoiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)
AWS deployment of HelloWorld using Gitlab runner in AWS
=======================================================


This repo demonstrates deployment of an AWS resource to AWS using a Gitlab runner in AWS. When the Gitlab pipeline is triggered, a Cloudformation template is <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-validate-template.html" target="_blank">validated</a> and <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-cli-creating-stack.html" target="_blank">deployed</a> to a specified AWS account creating an an <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html" target="_blank">AWS ECS Cluster</a>.
___
The Gitlab runner used in the instructions below is setup from the Github repo [https://github.com/scaniadevtools/gitlab-runner](https://github.com/scaniadevtools/gitlab-runner) using AWS ECS, Docker and Cloudformation. If you have configured your runner differently the instructions in this repo may not apply.

# Setting up this project
## Before you start
To run this project you should  have the following ready:
* An AWS account to deploy the cluster to. 
* A Gitlab runner in an AWS account (does not need to be the same account you deploy to). The runner should have been setup from the Github repo: [https://github.com/scaniadevtools/gitlab-runner](https://github.com/scaniadevtools/gitlab-runner)
* An account on the Gitlab server the runner is connected to. This can be gitlab.com or an enterprise Gitlab installation.

## Setup and install
### Copy this repo to your Gitlab account
Create a copy of this repo to a Gitlab project in your Gitlab account:
* In Gitlab, click "New project"
* Click the "Import project" tab
* Select "Repo by URL"
* In the "Git repository URL" enter "https://github.com/scaniadevtools/gitlab-aws-helloworld"
* Click "Crete project" button at the bottom of the page.
> <a href="https://docs.gitlab.com/ee/user/project/import/repo_by_url.html" target="_blank">Read more on import project with Repo from URL</a>

### Assign the runner to the project
When the project copy is created in Gitlab we need to assign a runner to it. In the Gitlab project:
1. Click "Settings" then click "CI / CD"
2. Next to "Runner settings", click the "Expand" button 
3. Under "Available specific runners", find the runner tagged with "vanilla" and click the "Enable for this project" button.

You runner is now assigned to the project and can start picking up jobs. However, for the jobs not to fail due to insufficient permissions we need to create a deploy role in AWS. See next section.

> If you don't have an existing runner in AWS to assign to the project, you can deploy one from <a href="https://github.com/scaniadevtools/gitlab-runner" target="_blank">https://github.com/scaniadevtools/gitlab-runner</a>.

> <a href="https://docs.gitlab.com/ee/ci/runners/#assigning-a-runner-to-another-project" target="_blank">Read more on assigning runners to project</a>


### Create the deploy role for the Gitlab runner
We will now deploy a cloudformation template in AWS that creates a role with the neccessary permissions that the Gitlab runner host can assume to be able to deploy resources to AWS for you. If you want to deploy to a different account from where your Gitlab runner is make sure you do the following steps in the correct AWS account(s).
1. Click on <a href="https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=helloworld-deploy-permissions&amp;templateURL=https://s3-eu-west-1.amazonaws.com/scaniadevtools-aws-templates/helloworld-deploy-permissions.yml" target="_blank"><img src="https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg"></a> (you may need to be logged in to AWS first).
2. Click "Next"
3.  On the "Specify Details" page, enter the following parameters for the stack:
    - Stack name, (defaults to `helloworld-deploy-permissions`)
    - The AWS account number where the Gitlab runner is running. Can be found at <a href="https://console.aws.amazon.com/support/home" target="_blank">https://console.aws.amazon.com/support/home</a>, under "Account number". __Note: You need to be logged in to the AWS account where the runner is to get the correct account number__.
    - The Gitlab runner role name. The name can be found in the <a href="https://console.aws.amazon.com/iam/home#/roles" target="_blank">AWS IAM console</a>. __Note: You need to be logged in to the AWS account where the runner is to find it__. **Tip**: on the IAM Console for roles, search for "*GitlabRunnerRole*".

4. Click "Next".
5. Click "Next" also on the "Options" page 
6. On the "Review" page, check the "I acknowledge that AWS CloudFormation might create IAM resources with custom names." at the bottom of the page 
7. Click "Create"
8. Wait for the stack to be created
9. When the stack is finished you should write down the name of the role that was created because we need it later. Select the newly created stack name in the Cloudformation console and select the "Outputs" tab and write down the value in the "Export Name" column for the stack.
![Cloudoformation output](images/cloudformation-output.png)

### Setup the AWS account and role in Gitlab
We need to tell the Gitlab runner which account to deploy to and which role to assume. We do that by creating two secret variables in Gitlab.
1. In the project in Gitlab click "Settings"
2. Click "CI / CD"
3. Next to "Secret variables", click the "Expand" button
4. For the account number. In the "Input variable key" field enter "AWS_ACCOUNTNO".
5. As the "Input variable value" enter the AWS account number where you created the deploy role above. Can be found at <a href="https://console.aws.amazon.com/support/home" target="_blank">https://console.aws.amazon.com/support/home</a>, under "Account number". __Note: You need to be logged in to the AWS account where you deployed the permissions template above to get the correct account number__.
6. (Recommended) Click the "Protected" switch to avoid having the account number showing up in build logs etc.
7. In a new "Input variable key" field enter "DEPLOY_ROLE_NAME"
8. As the "Input variable value" enter the name of the IAM role we created above (the export value you wrote down). This can also be found in <a href="https://console.aws.amazon.com/iam/home#/roles" target="_blank">AWS IAM console</a>. __Note: You need to be logged in to the AWS account where you deployed the "hello-truck-ecs-deploy-permissions.yml" template to find it__. **Tip**: in the IAM Console for roles, try searching for "*DeployRole*".

Example of the secret variables setup in Gitlab:
![Example of the secret variables setup](images/secret-variables-setup-example.png)

9. Click the "Save variables" button.


## Run the deployment
Now it is time to deploy the AWS ECS cluster to your AWS account.
1. In Gitlab, click on "CI / CD" and then "Pipelines"
2. Click "Run Pipeline"
3. Click "Create pipeline" button on the "New Pipeline" page 
4. Lean back and wait for your Gitlab runner to deploy the ECS cluster to your AWS account.

## The end result
Logged in to AWS you can after a while follow the progress in the <a href="https://console.aws.amazon.com/cloudformation/home?#/stacks" target="_blank">Cloudformation console</a>.

When the stack is created you can find the Cluster in AWS by navigating to the <a href="https://console.aws.amazon.com/ecs/home?#/clusters" target="_blank">ECS console</a>.

## Deleting
When you do not longer want your Gitlab project and AWS Cluster you can easily remove them.

Remove the ECS Cluster and the permissions stacks by the following steps when logged in to AWS:
1. Navigate to the <a href="https://console.aws.amazon.com/cloudformation/home" target="_blank">Cloudformation console</a>. 
2. Select the `gitlab-aws-helloworld` stack. 
3. Click on "Actions" followed by "Delete Stack". 
4. Repeat step 2-3 for the permissions stack.

Remove the Gitlab project by the following steps when you are logged into Gitlab:
 * For the project, click on "Settings"
 * Click "General"
 * Next to "Advanced settings", click the "Expand" button
 * Click the "Remove project" at the bottom of the page.
 * In the confirmation window, enter the project name (`gitlab-aws-helloworld`)
 * Click "Confirm"
 


__Happy Hacking__

*Scania Devtools Team*

## Want to contribute?
Got to the <a href="CONTRIBUTING.md">CONTRIBUTING</a> page.

## References

More on [AWS ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)








