# AWS-Jenkins-Pipeline

**Setting up a CI/CD pipeline by integrating Jenkins with AWS CodeBuild and AWS CodeDeploy**

Jenkins open-source automation server is used to deploy AWS CodeBuild artifacts with AWS CodeDeploy, creating a functioning CI/CD pipeline. When properly implemented, the CI/CD pipeline is triggered by code changes pushed to your GitHub repo, automatically fed into CodeBuild, then the output is deployed on CodeDeploy.

The functioning pipeline creates a fully managed build service that compiles your source code. It then produces code artifacts that can be used by CodeDeploy to deploy to your production environment automatically.

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/9ba041cc-663c-4d5b-a878-1f3f3116fc85)


**STEP 1:**
**CREATE THE S3 BUCKET TO STORE THE CODE FROM GITHUB AND STORES ARTIFACTS FROM CODE BUILD**

Click on create bucket

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/4525d9fb-0868-4b3c-b6e9-d3b363a06ee2)

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/f816eeeb-11dd-497f-bad8-692521403f90)

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/f63e5dda-7af0-4010-8cb2-4b3542b0df95)

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/2ded46d8-ae90-40a4-9663-22e6572cb7fa)

Click on create bucket 

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/8e229e8e-0595-4e8b-bf38-b339130401d9)


**STEP 2:CREATE IAM POLICY FOR JENKINS SERVER TO ACCESS THE S3 BUCKET**

Sign in to the AWS Management Console.

Go to the IAM (Identity and Access Management) service. In the left navigation pane, click on "Policies."

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/c636b8d6-4ac9-47fc-ade4-a8b1b3eb4de9)

Click on the "Create policy" button. Select the "JSON" tab to define the policy using JSON format.Replace the JSON with the following policy that allows the EC2 instance to access the specified S3 bucket:

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/e0e13b6d-97b9-40e8-bba2-d82705879bb7)

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/d1e034f0-4399-401d-9a48-748b6f6c9de1)

link for the script 

https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/blob/main/s3-jenkins-policy

Click on next which will take you to the review section

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/452ce744-a08e-488f-acf1-9f685b45eb4e)

Provide a name and description for the policy. Click on the "Create policy" button.

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/d4ac83a6-b0b3-4c4a-b2ef-e8f0e339be95)

**STEP 3:CREATE THE IAM ROLE FOR THE JENKINS EC2 INSTANCE AND ATTACH THE POLICY WHICH WE HAVE CREATED IN THE PREVIOUS STEP**

1. Sign in to the AWS Management Console.

2. Go to the IAM (Identity and Access Management) service.

3. In the left navigation pane, click on "Roles."

4. Click on the "Create role" button.

5. Select "EC2" as the trusted entity, then click on the "Next: Permissions" button.

6. In the "Attach permissions policies" step, search for the policy you created (by the name you provided) and select it.

7. Click on the "Next: Tags" button if you want to add any tags to the role (optional).

8. Click on the "Next: Review" button.

9. Provide a name and description for the role.

10. Click on the "Create role" button.


**STEP 4: create an IAM role for CodeDeploy service to enable it to read the tags applied to the instances, you can follow these steps:**

1. Sign in to the AWS Management Console.

2. Go to the IAM (Identity and Access Management) service.

3. In the left navigation pane, click on "Roles."

4. Click on the "Create role" button.

5. Select "AWS service" as the trusted entity, and then choose "CodeDeploy" from the list of services.

6. Under "Select your use case," choose "CodeDeploy - Amazon EC2" as the use case.

7. Click on the "Next: Permissions" button.

8. In the "Attach permissions policies" step, you can either choose an existing policy that provides the required permissions for 
   reading tags or creating a custom policy.
   
   * To choose an existing policy, search for and select the policy that allows CodeDeploy to read EC2 instance tags. For example, 
   "AmazonEC2ReadOnlyAccess" provides read-only access to EC2 resources, including tags.

   * To create a custom policy, click on the "Create Policy" button, and in the policy editor, define the permissions needed for 
     CodeDeploy to read the EC2 instance tags. The below link is an example of a policy that allows CodeDeploy to read tags:

     https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/blob/main/CodeDeployservicerolereadtags-policy

Note: The above policy allows CodeDeploy to describe tags for all EC2 resources in the AWS account. You can further refine the policy to limit the resources that CodeDeploy can access.

9. Click on the "Next: Tags" button if you want to add any tags to the role (optional).

10. Click on the "Next: Review" button.

11. Provide a name and description of the role.

12. Click on the "Create role" button.

**STEP 5: create an IAM role and instance profile for the EC2 instances of CodeDeploy, with permissions to write files to the S3 bucket and create deployments in CodeDeploy, you can follow these steps:**

1. Sign in to the AWS Management Console.

2. Go to the IAM (Identity and Access Management) service.

3. In the left navigation pane, click on "Roles."

4. Click on the "Create role" button.

5. Select "AWS service" as the trusted entity, and then choose "EC2" from the list of services.

6. Under "Select your use case," choose "Allows EC2 instances to call AWS services on your behalf."

7. Click on the "Next: Permissions" button.

8. In the "Attach permissions policies" step, you can either choose an existing policy that provides the required permissions for 
   writing files to S3 and creating CodeDeploy deployments or creating a custom policy.

   * To choose an existing policy, search for and select the policy that allows the required permissions. For example, you can select 
     "AmazonS3FullAccess" to grant full access to S3 or "AWSCodeDeployFullAccess" to grant full access to CodeDeploy.

   * To create a custom policy, click on the "Create Policy" button, and in the policy editor, define the permissions needed.Below link 
     is an example of a policy that allows the required permissions:

     https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/blob/main/codedeployinstanceroletos3-policy

Replace "your-s3-bucket-name" with the actual name of your S3 bucket, and "region" and "account-id" with the appropriate values for your AWS account in the above link

9. Click on the "Next: Tags" button if you want to add any tags to the role (optional).

10. Click on the "Next: Review" button.

11. Provide a name and description for the role, e.g., "CodeDeployRole."

12. Click on the "Create role" button.

**STEP 6:create the CodeBuildRole IAM role manually through the AWS Management Console, follow these steps:**

1. Sign in to the AWS Management Console.
2. Open the IAM console.
3. In the left navigation pane, choose "Roles" and then click on the "Create role" button.
4. In the "Select type of trusted entity" section, choose "AWS service."
5. In the "Choose the service that will use this role" section, select "CodeBuild" from the list of services.
6. Click "Next: Permissions."
7. In the "Attach permissions policies" step, you can either choose from existing policies or create a custom policy for the 
   CodeBuildRole. For simplicity, you can use the following LINK document:

   https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/blob/main/CodeBuildRole%20IAM%20role-Policy

Replace "your-s3-bucket-name" in the policy with the actual name of your S3 bucket or the appropriate wildcard pattern for the resources you want to grant access to.

8. Click "Next: Tags" (you can optionally add tags for the IAM role here).
9. Click "Next: Review."
10. Provide a name for the role (e.g., "CodeBuildRole") and an optional description.
11. Click "Create role" to create the IAM role.

**Step 7: Create the Jenkins by following the below steps** 

Write a terraform script to create the Jenkins server, For that get inside the Jenkins directory and alter the AWS access_key, Secret_key, ami-ID, region, and keypair in Jenkins.tf file Alter the VPC id in jenkins_sg.tf file

Now execute the terraform lifecycle commands as follows:

Terraform init
Terraform fmt
Terraform validate
Terraform plan
Terraform apply

After creating select the instance go to the actions -> Security -> Modify IAM role attach the Jenkins IAM role which we have created earlier

**Step 8: Create autoscaling groups to deploy the application**

create an Auto Scaling group with EC2 instances running Apache and the CodeDeploy agent fronted by an Elastic Load Balancer, along with the associated launch configurations and security groups, you can use the AWS Management Console or Terraform. Here, I'll provide the steps to do it manually using the AWS Management Console:

Create an Auto Scaling Launch Configuration:

1. Sign in to the AWS Management Console.
2. Go to the EC2 service by searching for "EC2" in the AWS Management Console search bar and selecting "EC2" from the list of services.
3. In the EC2 console, click on "Launch Configurations" in the left-hand navigation pane.
4. Click the "Create launch configuration" button.
5. Choose an Amazon Machine Image (AMI) that includes Apache and the CodeDeploy agent. You can use an existing AMI or create a custom 
   one with Apache and CodeDeploy pre-installed.
6. Configure the instance type, security groups, key pair, IAM role, user data (if needed), etc.
    Click the "Create launch configuration" button.

Create a Load Balancer:

1. In the EC2 console, click on "Load Balancers" in the left-hand navigation pane.
2. Click the "Create Load Balancer" button.
3. Choose the load balancer type (e.g., Application Load Balancer or Network Load Balancer).
4. Configure the load balancer settings, such as listeners, target groups, security groups, etc.
5. Click the "Next: Configure Security Settings" button (if applicable) and configure SSL certificates (optional).
6. Click the "Next: Configure Security Groups" button.
7. Add the necessary security groups to allow traffic to the load balancer. You can create new security groups or use existing ones.
8. Click the "Next: Configure Routing" button (if applicable) and configure routing rules (optional).
9. Click the "Next: Register Targets" button and select the instances created from the Auto Scaling group.
10. Review the settings and Click the "Next: Review" button.
11. Click the "Create" button.

Create an Auto Scaling Group:

1. In the EC2 console, click on "Auto Scaling Groups" in the left-hand navigation pane.
2. Click the "Create Auto Scaling group" button.
3. Choose the launch configuration you created in Step 1.
4. Configure the group size, scaling policies, load balancer, health checks, etc.
5. Click the "Next: Configure Tags" button (optional) and add any tags you want to associate with the Auto Scaling group.
6. Click the "Next: Configure Notifications" button (optional) and configure notification settings (e.g., for scaling events).
7. Click the "Next: Configure Scaling Policies" button (optional) and configure any additional scaling policies (if needed).
8. Review the settings and Click the "Next: Review" button.
9. Click the "Create Auto Scaling group" button.

**Step 9: Crete the Code build in aws**

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/0ae540c2-56e2-423a-b030-cd19cd4f77d3)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/50892828-7d5b-4146-a8a4-183f19c7b95c)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/63646430-de6a-4c53-9593-589e3ce038d6)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/40e21133-dab1-4b0c-bf99-458a3441c81d)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/2818daf4-e312-4b9c-8e51-a2a80e945d78)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/52ff8cfc-519b-40ba-b51a-61af45f64033)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/c6a28acc-104d-4b06-b906-46e23efd78c2)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/50c3ecba-19aa-4471-b5d7-310bb1bb86d5)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/f0f3d3d5-c50b-4bc4-8534-9eec1bd5d33f)


**Step 10: Create the code deploy in aws**

![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/9d256e6e-5591-4513-bffc-d12cc58d586b)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/75133753-a812-4647-9bcc-10183c1a7c90)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/5cdfb7b6-1afc-4efc-9fb5-e7f9b613387a)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/498979ea-05ab-4765-96ed-99f8abd27e6e)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/412556b6-b791-4429-91b0-22bec18b12bb)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/c57a22cd-8608-4078-b231-3ec3a407e2c7)
![image](https://github.com/automateopsprojects/AWS-Jenkins-Pipeline/assets/120359592/ad62a849-efbc-47fd-8ff3-de4921a29251)


**Step11: Jenkins Configuration**

To connect to Jenkins and download the AWS CodeBuild and AWS CodeDeploy plugins, follow these steps:

1. Open your Jenkins web interface in a web browser.
2. Click on "Jenkins" in the top-left corner to go to the Dashboard.
3. Click on "Manage Jenkins" from the left-hand sidebar.
4. Click on "Manage Plugins" from the dropdown menu.

>Installing AWS CodeBuild Plugin:

1. In the "Manage Plugins" page, click on the "Available" tab.
2. In the search bar, type "AWS CodeBuild" and press Enter.
3. Find the "AWS CodeBuild" plugin in the list and check the checkbox next to it.
4. Click on the "Install without restart" button to install the plugin.

>Installing AWS CodeDeploy Plugin:

1. In the "Manage Plugins" page, click on the "Available" tab.
2. In the search bar, type "AWS CodeDeploy" and press Enter.
3. Find the "AWS CodeDeploy" plugin in the list and check the checkbox next to it.
4. Click on the "Install without restart" button to install the plugin.

> Create a Jenkins job with a Free-style project that uses Git as the SCM, AWS CodeBuild as the build step, and AWS CodeDeploy as the post-build action, follow the steps below:

1. Open your Jenkins web interface in a web browser.
2. Click on "New Item" on the Jenkins Dashboard.
3. Enter a name for your project and select "Freestyle project."
4. Click "OK" to create the project.
5. In the project configuration page, scroll down to the "Source Code Management" section, and select "Git."
6. Enter your Git repository URL and configure the credentials if required.
7. In the "Build" section, click on "Add build step" and select "AWS CodeBuild."
8. In the "AWS CodeBuild" build step, you need to configure the AWS credentials, AWS Region, and other build settings as per your 
   requirements. Select the CodeBuild project you want to use for the build.
9. Click on "Add post-build action" and select "AWS CodeDeploy."
10. In the "AWS CodeDeploy" post-build action, configure the AWS credentials and AWS Region.
11. Choose the "CodeDeploy Application Name" and "Deployment Group Name" that you want to use for the deployment.
12. Save the project configuration.

Build now 

Once you build the application will get to build and store the artifact in an S3 bucket and on EC2 instance
