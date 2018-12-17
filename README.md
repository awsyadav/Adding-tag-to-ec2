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
