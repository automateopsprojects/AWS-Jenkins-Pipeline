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


