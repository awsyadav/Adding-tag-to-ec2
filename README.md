# Adding-tag-to Ec2, ENI, EBS
How to use a Lambda function to propagate the tags applied to an EC2 instance to the attached EBS volumes and ENIs when they are updated.

# Short Description
When a new instance is created, whether it is created on the web console, using the CLI or one of the AWS SDKs or through terraform, they will be created with at least one EBS volume (except for instance store instances) and one network interface, however the API call CreateTags will not propagate the tags to these dependent resources. To resolve this problem, we are going to use CloudWatch Events and Lambda to detect when instance tags are modified and then automatically tag the EBS volumes and ENIs that are attached to the instance.
We are going to take advantage a feature of CloudWatch Events that can trigger events every time an EC2 API call is invoked and specifically we will trigger it when CreateTags is invoked by an IAM user or a role and then forward the event to Lambda, Lambda will then check if the tagged resources were instances and if they are, since the event will contain the instance ids, we can use the event information to tag the EBS volumes and the ENIs attached to the instance.

# Resolution
In order to use CloudWatch Events to trigger events on API calls CloudTrail needs to be enabled in the account and region, CloudTrail is a service that keeps track of the AWS API calls that have been invoked as well as the details of the call and the IAM user or role who invoked it. 
The first step is go to the CloudTrail console and create a Trail for the region, it is advisable that you apply the trail to all the regions as this will log all the events in your AWS account, which is recommended for auditing and troubleshooting.
Once the we have CloudTrail setup we can go to the Lambda console and create a new Lambda function with the name tag_ec2_dependencies. This function will have the following setup:

* Runtime: python 2.7
* Role with permissions to log to CW Logs, describe the instances and tag the EBS volumes and ENIs
* Memory: 128MB
* Timeout: 10 seconds

When creating the function, select Edit code inline and use the *Addtag.py*

For the IAM role select create a custom role and then on the role creation page select Create a new IAM Role with name lambda_ec2_tagging and update the policy document with this policy:

Then complete the creation of the function and go to the CloudWatch Events console, this can be located in the CloudWatch console under Events -> Rules and the steps are:
1.	Select Create Rule
2.	Select Build event pattern to match events by service
3.	Service Name: EC2
4.	Event Type: AWS API Call via CloudTrail
5.	Select Specific operation(s)
6.	Select CreateTags
7.	Select Add target
8.	Select Lambda function
9.	Function: tag_ec2_dependencies (or the name of the function if you used a different name)
10.	We leave version alias and input in their default values
11.	Select Configure details
12.	Give a name to the rule: tag_ec2_dependencies
13.	Add a description
14.	State: enabled


Now every time the tags of an instance are modified or a new instance is created the tags will propagate to the EBS volumes and ENIs attached to the instance, you can test it by creating a new instance or adding new tags to an existing one on web console (if you add tags to an existing instance only the new tags will be added).
Note: this will propagate new tags to EBS volumes and ENIs when they are added to the instance but will not delete tags from them when they are deleted from the instance, for this you will need to create a new rule in CloudWatch Events to capture DeleteTags and create a new Lambda function that deletes the tags from the dependent resources. 
