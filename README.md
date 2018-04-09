AWS deployment of HelloWorld using Gitlab runner in AWS
=======================================================


This repo demonstrates deployment of an AWS resource to AWS using a Gitlab runner in AWS. When the Gitlab pipeline is triggered, a Cloudformation template is <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-validate-template.html" target="_blank">validated</a> and <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-cli-creating-stack.html" target="_blank">deployed</a> to a specified AWS account creating an an <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_clusters.html" target="_blank">AWS ECS Cluster</a>.
___

### __NOTE#1!__ This repo is based on the Gitlab runner configuration in the <a href="https://github.com/scaniadevtools/gitlab-runner" target="_blank">https://github.com/scaniadevtools/gitlab-runner</a> repo. If you have another configuration of your Gitlab runner the content in this repo may not apply.

### __NOTE#2!__ If you are at the Github version of this repo you should head over to the Gitlab version <a href="https://gitlab.com/scaniadevtools/gitlab-samples/gitlab-aws-helloworld">here</a> and continue from there.
___

## Setting up this project
### Before you start
To get this project up and running you need to have the folloing ready:
* A Gitlab runner in AWS from the repo <a href="https://github.com/scaniadevtools/gitlab-runner" target="_blank">https://github.com/scaniadevtools/gitlab-runner</a> up and running.
* Access to an AWS account (does not need to be the same account the Gitlab runner is deployed in)
* A Gitlab account

### Setup and install

#### Fork the project
<a href="https://docs.gitlab.com/ee/gitlab-basics/fork-project.html" target="_blank">Fork</a> this project into your own Gitlab account.

Now you should have your own copy of the project in your Gitlab account. From your terminal application, execute the following command (make sure to replace `<your_gitlab_username>` with your actual GitHub username):
```bash
git clone https://gitlab.com/<your_gitlab_username>/gitlab-aws-helloworld.git 
```

This creates a directory named `gitlab-aws-helloworld` in your current directory, which contains the code for the Helloworld project.

#### Allow the Gitlab runner to deploy to AWS by creating a Deploy role in AWS
The Gitlab runner need to have permissions to deploy resources to your AWS account. This is done by creating a new role in the AWS account with the permissions needed that the Gitlab runner can assume. The role is specified in the `aws-permissions/helloworld-deploy-permissions.yml` template and must be deployed to the AWS account where the resulting AWS ECS Cluster should be deployed. __Note__ This may or may not be  the same AWS account the Gitlab runner is running in. For example, you may have your Gitlab runner host running in your *Development* account but want to deploy the AWS resources to your *Production* account. 

##### Before you create the role 
Take a note of the AWS account number where your Gitlab runner is running before you deploy the role. This account number can be found when logged in to the AWS account by selecting <a href="https://console.aws.amazon.com/support/home" target="_blank">`Support->Support Center`</a> in the AWS console.

##### Create the role 

Deploy the role to your AWS account by clicking <a href="https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=helloworld-deploy-permissions&amp;templateURL=https://s3-eu-west-1.amazonaws.com/scaniadevtools-aws-templates/helloworld-deploy-permissions.yml" target="_blank"><img src="https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg"></a> and follow the instructions.

You can also upload the file `aws-permissions/helloworld-deploy-permissions.yml` in the AWS Cloudformation console <a href="https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/new" target="_blank">here</a>

> <a href="https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html" target="_blank">Read more about assuming roles in AWS.</a>

> <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html" target="_blank">Read more about creating Cloudformation stacks.</a>


After creating the role using `aws/helloworld-deploy-permissions.yml` you will have a new role in your account named ``DeployRole`` with the following AWS permissions:

* cloudformation:ValidateTemplate
* cloudformation:CreateStack
* cloudformation:DescribeStacks
* ecs:DescribeClusters
* ecs:CreateCluster

You should be able to inspect it in the AWS IAM Console <a href="https://console.aws.amazon.com/iam/home?#/roles/DeployRole" target="_blank">here</a>.

### Assign the runner to this project
To make the runner build accoring to the `.gitlab-ci.yml` file you must <a href="https://docs.gitlab.com/ee/ci/runners/#assigning-a-runner-to-another-project" target="_blank">assign the runner</a>  to this project. 

### Configure the .gitlab-ci.yml to use correct AWS account
Now we are ready to configure the project pipeline to use our roles and AWS account(s).

Open the ``.gitlab-ci.yml`` in your favorite editor and change the ``AWS_ACCOUNTNO: "123456789012"`` to the account id of the AWS account where you applied the `aws/helloworld-deploy-permissions.yml` template.

![](images/configure-gitlab-ci.PNG)


### Test run
To fire up everything and test your new pipeline just commit your code and push it to your remote repo on Gitlab. Your pipeline should start building in a few seconds resulting in a AWS ECS Cluster named ``hello-world`` in your  AWS account. 

__Happy Hacking__

*Scania Devtools team*










