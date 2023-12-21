# WAF Logs Dashboard For DDoS Analysis Project

During major online events like live broadcasts, security teams need a fast and clear understanding of attack patterns and behaviors to distinguish between normal and malicious traffic flows. The solution outlined here allows filtering flow logs by "Client IP", "URI", "Header name", and "Header value" to analyze these fields and pinpoint values specifically associated with attack traffic versus normal traffic. For example, the dashboard can identify the top header values that are atypical for normal usage. The security team can then create an AWS WAF rule to block requests containing these header values, stopping the attack.
 
This project demonstrates using AWS Glue crawlers to categorize and structure WAF flow log data and Amazon Athena for querying. Amazon Quicksight is then employed to visualize query results in a dashboard. Once deployed, the dashboard provides traffic visualization similar to the example graphs shown in _Images_ folder in under project , empowering security teams with insight into attacks and defense

# Deployment
**Step1: install Pre-requisites**

 Install the cid-cmd tool using the following Command Line: 
    
`pip3 install cid-cmd`

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
AWS Glue Crawler, AWS Glue Database and the corresponding AWS IAM Roles and Policies are created successfully.

10. go to the **Crawlers**  section in AWS Glue, and select the crawler that has been created as part of the cloudformation stack you have just deployed. It should be named " crawl-aws-waf-logs"
11. Edit the Crawler configuration, and edit the **Step 4: Set output and scheduling**
12. Under **Advanced options** select **"Update all new and existing partitions with metadata from the table"**. Select **Next ** and then **Update**.
13. Select the crawler "crawl-aws-waf-logs" and hit **run**. This will create a table within the database. As per this project, we set the frequency of crawnler's run at once a day. This can be changes by editing the crawler.


**Step3 : Prepare Amazon QuickSight**

 Amazon QuickSight is the AWS Business Intelligence tool that will allow you to not only view the Standard AWS provided insights into all of your accounts, but will also allow to produce new versions of the Dashboards we provide or create something entirely customized to you. If you are already a regular Amazon QuickSight user you can skip these steps. 

1. Log into your AWS Account and search for QuickSight in the list of Services
2. You will be asked to sign up before you will be able to use it
3. After pressing the Sign up button you will be presented with 2 options, please ensure you select the Enterprise Edition during this step
4. Select continue and you will need to fill in a series of options in order to finish creating your account.
    1. Ensure you select the region that is most appropriate based on where your S3 Bucket is located containing your AWS WAF Logs  report files.
5. Enable the Amazon S3 option and select the bucket where your AWS WAF Log data created via Data Collection Lab are located 

**Step4 : Deploy the Dashboard**


We will be using the **AWS-WAF-Logs-Dashboard-For-Event-Analysis.yaml** template to deploy the dashboard in the quicksight. This template will also create a dataset in Amazon Quicksight.

1. Open your favorite terminal, under the directory where you saved AWS-WAF-Logs-Dashboard-For-Event-Analysis.yaml template and run the following cli:

`cid-cmd deploy --resources ./AWS-WAF-Logs-Dashboard-For-Event-Analysis.yaml`

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
####### AWS-WAF-Logs-Dashboard-For-Event-Analysis-V1 is available at: _[https://[region].quicksight.aws.amazon.com/sn/dashboards/aws-waf-logs-dashboard-for-event-analysis-v1]_
#######

You can now start analysing your traffic pattern.








