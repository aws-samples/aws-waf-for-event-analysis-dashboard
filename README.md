# WAF Logs Dashboard For DDoS Analysis Project

During major online events like live broadcasts, security teams need a fast and clear understanding of attack patterns and behaviors to distinguish between normal and malicious traffic flows. The solution outlined here allows filtering flow logs by "Client IP", "URI", "Header name", and "Header value" to analyze these fields and pinpoint values specifically associated with attack traffic versus normal traffic. For example, the dashboard can identify the top header values that are atypical for normal usage. The security team can then create an AWS WAF rule to block requests containing these header values, stopping the attack.
 
This project demonstrates using AWS Glue crawlers to categorize and structure WAF flow log data and Amazon Athena for querying. Amazon Quicksight is then employed to visualize query results in a dashboard. Once deployed, the dashboard provides traffic visualization similar to the example graphs shown below, empowering security teams with insight into attacks and defense:

# Deployment
**Step1: install Pre-requisites**

 Install the cid-cmd tool using the following Command Line: 
    
`pip3 install --upgrade cid-cmd`

**Step2: AWS WAF Logs Dashboard Data Set Deployment**
2. Sign in to the **AWS Management Console**
3. Navigate to the **AWS CloudFormation **console >** Create Stack** > **“With new resources” **

* Note: the template will ask you to enter an S3 bucket name. The bucket name has to start with aws-waf-logs prefix as required by AWS WAF service 

4. Download the AWS CloudFormation template **"AWS-WAF-Logs-Dashboard-Set-UP-CFN.yaml"** file from this git repository. In the create wizard specify the **AWS-WAF-Logs-Dashboard-Set-UP-CFN.yaml**  template.
5. fill the following parameter **"S3BucketName"** with a bucket name where you store the waf logs. it has to start with aws-waf-logs.
6. Specify a **“Stack name” **and choose Next
7. Leave the **“Configure stack options”** at default values and choose Next
8. Review the details on the final screen and under **“Capabilities”** check the box for **“I acknowledge that AWS CloudFormation might create IAM resources with custom names”**
9. Choose **Submit**

Once the Stack is created successfully, you will see the following resources deployed:
AWS Glue Crawler, AWS Glue Database and  Amazon Athena Query (under '**“Saved Queries”** tab to create the view in Athena) and the corresponding AWS IAM Roles and Policies are created successfully.

10. go to the **Crawlers**  section in AWS Glue, and select the crawler that has been created as part of the cloudformation stack you have just deployed. It should be named " crawl-aws-waf-logs"
11. Edit the Crawler configuration, and edit the **Step 4: Set output and scheduling**
12. Under **Advanced options** select **"Update all new and existing partitions with metadata from the table"**. Select **Next ** and then **Update**.

**Step 3:  Create View in Amazon Athena**

*Step 3.1 prepare Amazon Athena * 

If this is the first time you will be using Athena you will need to complete a few setup steps before you are able to create the views needed. If you are already a regular Athena user you can skip these steps and move on to the _Step3.2: create the view in Amazon Athena section below_. 

1. From the services list, choose S3
2. Create a new S3 bucket for Athena queries to be logged to. Keep to the same region as the S3 bucket created for your WAF logs.
3. From the services list, choose Athena
4. Select Get Started to enable Athena and start the basic configuration
5. At the top of this screen select Before you run your first query, you need to set up a query result location in Amazon S3.
6. Validate your Athena primary workgroup has an output location by

    * Open a new tab or window and navigate to the Athena console
    * Select Workgroup: primary

7.  Confirm your Query result location is configured with an S3 bucket path. If not configured, continue to setting up by clicking Edit workgroup
8.  Add the S3 bucket path you have selected for your Query result location and click save 

*Step 3.2 create the view in Amazon Athena *

We will use  saved query created as part of the AWS CloudFormation stack to create a view that will be used for the dashboard:

1. Go to Amazon Athena, Under query editor, on the top right, under **Worgroup**, select **“waf-logs-athena”** which has been created by the AWS CloudFormation stack.
2. under Query editor, go to **Saved quries **tab and choose the query name **aws_waf_insights_logging**
3. on the Query editor, check that the Data source, database and table names while running the query. if the run is successful, the view named **“waf_logs_antiddos_view” ** should be created

**Step4 : Prepare Amazon QuickSight**

 Amazon QuickSight is the AWS Business Intelligence tool that will allow you to not only view the Standard AWS provided insights into all of your accounts, but will also allow to produce new versions of the Dashboards we provide or create something entirely customized to you. If you are already a regular Amazon QuickSight user you can skip these steps. 

1. Log into your AWS Account and search for QuickSight in the list of Services
2. You will be asked to sign up before you will be able to use it
3. After pressing the Sign up button you will be presented with 2 options, please ensure you select the Enterprise Edition during this step
4. Select continue and you will need to fill in a series of options in order to finish creating your account.
    1. Ensure you select the region that is most appropriate based on where your S3 Bucket is located containing your AWS WAF Logs  report files.
5. Enable the Amazon S3 option and select the bucket where your AWS WAF Log data created via Data Collection Lab are located 

**Step5 : Deploy the Dashboard**


We will be using the **AWS-WAF-Logs-Dashboard-For-Event-Analysis-V1.yaml** template to deploy the dashboard in the quicksight. This template will also create a dataset in Amazon Quicksight.

1. Open your favorite terminal, go where you saved AWS-WAF-Logs-For-Anti-DDoS.yaml and run the following cli:

`cid-cmd deploy —resources ./AWS-WAF-Logs-Dashboard-For-Event-Analysis-V1.yaml`

2. during the creation you will be prompted to select: 
[dashboard-id] Please select dashboard to install: 
Select  _AWS-WAF-Logs-Dashboard-For-Event-Analysis-V1_
[athena-workgroup] Select AWS Athena workgroup to use:
Select _primary_
[athena-database] Select AWS Athena database to use: 
Select _waflogsdb_


after a successful run , the output will be :

#######
####### Congratulations!
####### AWS-WAF-Logs-Dashboard-For-Event-Analysis-V1 is available at: https://us-east-1.quicksight.aws.amazon.com/sn/dashboards/aws-waf-logs-dashboard-for-event-analysis-v1
#######

You can now start analysing your traffic pattern.








