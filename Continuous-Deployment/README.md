Continuous Deployment with EKS

 we build a CI/CD pipeline using AWS CodePipeline. The CI/CD pipeline will deploy a sample Kubernetes service. Committing a code change to the GitHub repository triggers the automated delivery of this change to the cluster.

Create IAM Role for CodeBuild
CodeBuild requires an IAM role to interacting with an EKS cluster.

This script creates an IAM role  This Role is used in the CodeBuild stage of CodePipeline to interact with EKS using kubectl.

./create-iam-role.sh

Add IAM role to aws-auth ConfigMap
This script patches aws-auth ConfigMap, adding the IAM role we created in the previous step. This allows kubectl in the CodeBuild stage of the pipeline to interact with EKS.

./patch-aws-auth.sh

You can manually edit the ConfigMap with this command:

kubectl edit -n kube-system configmap/aws-auth

Fork Github Repository
We must clone sample Kubernetes service so that we will be able modify the repository and trigger builds.

Login to GitHub and fork the sample service to your own account:

https://github.com/dashpradeep99/cicd-sample-application

Once the repo is forked, you can view it in your your GitHub repositories.

Github Access Token
In order for CodePipeline to receive callbacks from GitHub, we need to generate a personal access token.

Create a new personal access token here: https://github.com/settings/tokens/new

Enter a value for Token description, check the repo permission scope and scroll down and click the Generate token button.

Copy the personal access token and save it in a secure place.

CodePipeline
Use the codepipeline.yaml CloudFormation template to create the CodePipeline resources.

In the CloudFormation template parameters, enter your Github username and personal access token you create in the previous step.

Wait for the status to change from CREATE_IN_PROGRESS to CREATE_COMPLETE before moving on to the next step.

Open CodePipeline in the Management Console. You will see a CodePipeline that starts with eks-workshop-codepipeline. Click this link to view the details.

Once you are on the detail page for the specific CodePipeline, you can see the status along with a links to the build details. If you click on the details link in the build/deploy stage, you can see the output from the CodeBuild process.

To view the status of the Kubernetes deployment:

kubectl describe deployment hello-k8s
For the status of the service, run the following command:

kubectl describe service hello-k8s
Once the service is built and delivered, we can run the following command to get the Elastic Load Balancer (ELB) endpoint and open it in a browser. If the message is not updated immediately, give Kubernetes some time to deploy the change.

kubectl get services hello-k8s -o wide
This service was configured with a LoadBalancer so, an AWS Elastic Load Balancer is launched by EKS for the service. The EXTERNAL-IP column contains a value that ends with elb.amazonaws.com - the full value is the DNS address.

Trigger a New Deployment
Open GitHub and select the forked repository with the name cicd-sample-application

Click on the main.go file and then click on the edit button

Change the text where it says Hello World, add a commit message and then click the Commit changes button.

You should leave the master branch selected.

After you modify and commit your change in GitHub, in few minute you will see a new build triggered in CodePipeline.

Confirm the update by browsing the ELB URL you used previously.

Clean Up
Delete the ECR repository.

Empty and then delete the S3 bucket used by CodeBuild for build artifacts (bucket name starts with ekscodepipeline). First, select the bucket, then empty the bucket and finally delete the bucket.

Delete the CloudFormation stack in the AWS Management Console.

Delete the IAM role created for CodeBuild to permit changes to the EKS cluster, along with the the Kubernetes deployment and service:

./cleanup.sh
