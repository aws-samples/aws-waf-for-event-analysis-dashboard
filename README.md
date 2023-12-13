# WAF Logs Dashboard For DDoS Analysis Project

During major online events like live broadcasts, security teams need a fast and clear understanding of attack patterns and behaviors to distinguish between normal and malicious traffic flows. The solution outlined here allows filtering flow logs by "Client IP", "URI", "Header name", and "Header value" to analyze these fields and pinpoint values specifically associated with attack traffic versus normal traffic. For example, the dashboard can identify the top header values that are atypical for normal usage. The security team can then create an AWS WAF rule to block requests containing these header values, stopping the attack.
 
This project demonstrates using AWS Glue crawlers to categorize and structure WAF flow log data and Amazon Athena for querying. Amazon Quicksight is then employed to visualize query results in a dashboard. Once deployed, the dashboard provides traffic visualization similar to the example graphs shown below, empowering security teams with insight into attacks and defense:

# Deployment
**Step1: install Pre-requisites**

    install the cid-cmd tool using the following Cli `pip3 install --upgrade cid-cmd`

**Step2: AWS WAF Logs Dashboard Data Set Deployment**
2. Sign in to the **AWS Management Console**
3. Navigate to the **AWS CloudFormation **console >** Create Stack** > **“With new resources” **

* Note: the template will ask you to enter an S3 bucket name. The bucket name has to start with aws-waf-logs prefix as required by AWS WAF service 

4. Download the AWS CloudFormation template **"AWS-WAF-Logs-Dashboard-Set-UP-CFN.yaml"** file from this git repository. In the create wizard specify the **AWS-WAF-Logs-Dashboard-Set-UP-CFN.yaml**  template.
5. fill the following parameter **"S3BucketName"** with a bucket name where to store your waf logs. it has to start with aws-waf-logs.
**Note**: if the S3 bucket already exists, the creation of the stack will fail.
6. Specify a **“Stack name” **and choose Next
7. Leave the **“Configure stack options”** at default values and choose Next
8. Review the details on the final screen and under **“Capabilities”** check the box for **“I acknowledge that AWS CloudFormation might create IAM resources with custom names”**
9. Choose **Submit**

Once the Stack is created successfully, you will see the following resources deployed:
Amazon S3 Bucket, AWS Glue Crawler, AWS Glue Database, Amazon Kinesis Data Stream and Amazon Athena Query (under '**“Saved Queries”** tab to create the view in Athena) and the corresponding AWS IAM Roles and Policies are created successfully.

**Step 3: First AWS WAF Logs activation**

If this the first time you activate the AWS WAF Logs, proceed with the following steps.
 If you have already AWS WAF Logs activated and sent to an S3 bucket, skip these steps and move on to Step 4: AWS WAF logs Already activated and sent to an S3 Bucket 

1. In the AWS WAF console, and select the WebACL for which you want to enable the logging
2. go to **logging and metrics** and click on **Enable** 
3. under **“Logging destination”**, choose S3 bucket
4. under **“ Amazon S3 bucket”**, click on **“Create new”** you will be redirected to Amazon S3 to create the bucket. 
5. Under Bucket name, enter a bucket name that start with **“aws-waf-logs-”**. The bucket name has to be unique.
6.  Leave all remaining parameters as they are and click Create bucket

The AWS WAF Flow logs for that ACL will now be stored in the Amazon S3 you have just created

**Step 4: AWS WAF logs Already activated and sent to an S3 Bucket**

1. Go to AWS Glue, Select the crawler that has been just created by the AWS CloudFormation Template **crawl-aws-waf-logs**.
2. Under **Data Source**, select the S3 bucket and  hit **Edit**.
3. Change the S3 Path to the bucket where your AWS WAF Logs are stored. Select Update S3 Data Source to save the change.
4. You have to grant access to the Amazon S3 bucket from the Crawler, to do that, you have to change the IAM Poliy attached to IAM role assumed by the crawler
5. Select the crawler that has been just created by the AWS CloudFormation Template crawl-aws-waf-logs and note down the IAM role name.
6. Go to AWS Identity and Access Management, under Roles, search for the IAM Role name you have noted down.
7. Under Permissions Policies, select WAF-GlueCrawlerRolePolicy and change the Reource arn of the S3 bucket with the one where you AWS WAF Logs are stored
8. Go back to AWS Glue, Under Crawlers, select the Crawlers that has been created with the AWS CloudFormation template and hit Run. It should time few minutes to the crawler to complete.

**Step 4:  Create View in Amazon Athena**
*Step 4.1 prepare Amazon Athena * 
If this is the first time you will be using Athena you will need to complete a few setup steps before you are able to create the views needed. If you are already a regular Athena user you can skip these steps and move on to the Step7.2: create the view in Amazon Athena section below. 

1. From the services list, choose S3
2. Create a new S3 bucket for Athena queries to be logged to. Keep to the same region as the S3 bucket created for your Compute Optimizer data created via Data Collection Lab.
3. From the services list, choose Athena
4. Select Get Started to enable Athena and start the basic configuration
5. At the top of this screen select Before you run your first query, you need to set up a query result location in Amazon S3.
6. Validate your Athena primary workgroup has an output location by

    * Open a new tab or window and navigate to the Athena console
    * Select Workgroup: primary

7.  Confirm your Query result location is configured with an S3 bucket path. If not configured, continue to setting up by clicking Edit workgroup
8.  Add the S3 bucket path you have selected for your Query result location and click save 

*Step 4.2 create the view in Amazon Athena *

We will use  saved query created as part of the AWS CloudFormation stack to create a view that will be used for the dashboard:

1. Go to Amazon Athena, Under query editor, on the top right, under Worgroup, select “waf-logs-athena” which has been created by the AWS CloudFormation stack.
2. under Query editor, go to Saved quries tab and choose the query name aws_waf_insights_logging
3. on the Qury editor, check that the Data source, database and table names while running the query. if the run is successful, the view named “waflogs” should be created

**Step6 : Prepare Amazon QuickSight**

 Amazon QuickSight is the AWS Business Intelligence tool that will allow you to not only view the Standard AWS provided insights into all of your accounts, but will also allow to produce new versions of the Dashboards we provide or create something entirely customized to you. If you are already a regular Amazon QuickSight user you can skip these steps. 

1. Log into your AWS Account and search for QuickSight in the list of Services
2. You will be asked to sign up before you will be able to use it
3. After pressing the Sign up button you will be presented with 2 options, please ensure you select the Enterprise Edition during this step
4. Select continue and you will need to fill in a series of options in order to finish creating your account.
    1. Ensure you select the region that is most appropriate based on where your S3 Bucket is located containing your AWS WAF Logs  report files.
5. Enable the Amazon S3 option and select the bucket where your AWS WAF Log data created via Data Collection Lab are located 

**Step7 : Deploy the Dashboard**


We will be using the **WAF-logs-Dashboard-for-DDoS-Analysis-Quicksight-Template.yaml** template to deploy the dashboard in the quicksight. This template will also create a dataset in Amazon Quicksight.

1. Open your favorite terminal, go where you saved AWS-WAF-Logs-For-Anti-DDoS.yaml and run the following cli:

`cid-cmd deploy —resources ./AWS-WAF-Logs-For-Anti-DDoS.yaml`
2. during the creation you will be prompted to select: 
[dashboard-id] Please select dashboard to install: 
Select aws-waf-logs-for-anti-ddos
[athena-database] Select AWS Athena database to use: 
Select waflogs


after a successful run , the dashboard will be installed in your Amazon quicksight account.










