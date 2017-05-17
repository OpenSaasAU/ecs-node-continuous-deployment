# ECS Reference Architecture: Continuous Deployment

This project is currently setup to run a single instance keystone.js application with a mongodb database, without an elastic loadbalancer, and an nginx proxy with letsencrypt.

The ECS Continuous Deployment reference architecture demonstrates how to achieve [continuous deployment][continuous-deployment] of an application to Amazon ECS using AWS CodePipeline, AWS CodeBuild, and AWS CloudFormation. With continuous deployment, software revisions are deployed to a production environment automatically without explicit approval from a developer, making the entire software release process automated.

Launching this AWS CloudFormation stack provisions a continuous deployment process that uses AWS CodePipeline to monitor a GitHub repository for new commits, AWS CodeBuild to create a new Docker container image and to push it into Amazon ECR, and AWS CloudFormation to deploy the new container image to production on Amazon ECS.


[![](images/architecture.png)][architecture]

## Running

#### 1. Fork the GitHub repository

[Fork](https://help.github.com/articles/fork-a-repo/) the [OpenSaas Amazon ECS node app](https://github.com/OpensaasAU/ecs-node-continuous-deployment) GitHub repository into your GitHub account.

From your terminal application, execute the following command (make sure to replace `<your_github_username>` with your actual GitHub username):

```console
git clone https://github.com/<your_github_username>/ecs-node-continuous-deployment
```

This creates a directory named `ecs-node-continuous-deployment` in your current directory, which contains the code for the Amazon ECS sample app.

#### 2. Upload files to S3

Zip the Templates folder and upload it to an S3 Bucket and change th ecs-refarch-contnuos-deployment.yaml to relect the location of your `TemplateBucket`

Copy configEnvTemplate.json to configEnv.json and enter your app environment details. Upload configEnv.json to your S3 config bucket.


#### 3. Create the CloudFormation stack

```console
aws cloudformation deploy --template-file ecs-refarch-continuous-deployment.yaml --stack-name <Stack_Name --capabilities CAPABILITY_NAMED_IAM --parameter-overrides GitHubRepo=<Git_Repo_Name> GitHubBranch=<Git_Branch> GitHubUser=<Git_Username> GitHubToken=<Git_Token> EC2KeyName=<EC2_Key_Name> ConfigBucket=<S3_Config_Bucket_Name>
```

The CloudFormation template requires the following parameters:

- GitHub configuration
  - **Repo**: The repo name of the sample service.
  - **Branch**: The branch of the repo to deploy continuously.
  - **User**: Your username on GitHub.
  - **Personal Access Token**: Token for the user specified above. ([https://github.com/settings/tokens](https://github.com/settings/tokens))
- Stack configuration
  - **EC2KeyName**: The name of your EC2 Key to login to the EC2 Instance
  - **ConfigBucket**: Name of your S3 Bucket that contains your configEnv.zip which has your configEnv.json file

The CloudFormation stack provides the following output:

- **PipelineUrl**: The continuous deployment pipeline in the AWS Management Console.

### Cleaning up the example resources

To remove all resources created by this example, do the following:

1. Delete the main CloudFormation stack which deletes the substacks and resources created by the exercises.
1. Manually delete resources which may contain files:
  - S3 bucket: `ArtifactBucket`
  - ECR repository: `Repository`

## CloudFormation template resources

The following sections explains all of the resources created by the CloudFormation template provided with this example.

#### [DeploymentPipeline](templates/deployment-pipeline.yaml)

  Resources that compose the deployment pipeline include the CodeBuild project, the CodePipeline pipeline, an S3 bucket for deployment artifacts, and all necessary IAM roles used by those services.

#### [Service](templates/service.yaml)

  An ECS task definition, service, IAM role, and ECR repository for the sample application. This template is used by the CodePipeline pipeline to deploy the sample service continuously.

#### [Cluster](templates/ecs-cluster.yaml)

  An ECS cluster backed by an Auto Scaling group of EC2 instances running the Amazon ECS-optimized AMI.

#### [Load Balancer](templates/load-balancer.yaml)

  Not used in the current configuration

#### [VPC](templates/vpc.yaml)

  A VPC with two public subnets on two separate Availability Zones, an internet gateway, and a route table with a default route to the public internet.

## License

This reference architecture sample is [licensed][license] under Apache 2.0.


[continuous-deployment]: https://aws.amazon.com/devops/continuous-delivery/
[architecture]: images/architecture.pdf
[license]: LICENSE
