# David Lin's Cloud Custodian Policies (in Production)

| Policy | Description |
|--------|-------------|
| [offhours.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/offhours.yml)<br> | Starts and stops instances during offhours via Lambda function. Instances filtered on presence of maid_offhours tag or StartAfterHours/StopAfterHours custom tags. [(See Offhour Examples)](#offhours) |
| [unused-sgroup-audit.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/unused-sgroup-audit.yml)<br> | Retrieves all unused security groups that match regex, deletes them, then sends notifications.  |
| [s3-public-audit.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/s3-public-audit.yml)<br> | Sends notification when public S3 bucket is created.  |
| [copy-instance-tags.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/copy-instance-tags.yml)<br> | Periodically copies tags from EC2 instances to respective EBS volumes. |

# David Lin's Cloud Custodian Policies (Examples)
| Policy | Description |
|--------|-------------|
| [security-groups-unused.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/security-groups-unused.yml) | Retrieves unused security groups using regex |
| [security-groups-unused-notify.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/security-groups-unused-notify.yml) | Retrieves unused security groups using regex and notifies via email |
| [iam.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/iam.yml)                    | Retrieves iam users using regex |
| [mfa.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/mfa.yml)                    | Retrieves iam users with MFA enabled |
| [roles.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/roles.yml)                  | Retrieves unused roles on EC2, Lambda, and ECS |
| [admin-group.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/admin-group.yml)            | Retrieves users in the group named 'Administrators' |
| [mfa-unused.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/mfa-unused.yml)             | Retrieves users who have MFA disabled in the group named 'Administrators' |
| [emailer.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/emailer.yml)                | Sends email notification via Simple Email Service (SES) using notify action |
| [ebs-garbage-collection.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/ebs-garbage-collection.yml) | Deletes all unattached volumes |
| [ebs-garbage-collection-lambda.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/ebs-garbage-collection-lambda.yml) | Deletes all unattached volumes using Lambda function |
| [public-subnet-instance-audit-notify.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/public-subnet-instance-audit-notify.yml) | Sends email notification via SES when EC2 instance launches in a public subnet |
| [public-subnet-instance-audit-whitelist.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/public-subnet-instance-audit-whitelist.yml) | Lambda that sends email notification via SES when EC2 instance launches in a public subnet and is NOT in the whitelist |
| [mark-unused-sgroups.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/mark-unused-sgroups.yml) | Mark unused security groups for deletion after N days ; to be used with delete-marked-sgroups.yml |
| [delete-marked-sgroups.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/delete-marked-sgroups.yml) | Unmarks used security groups that were marked for deletion then deletes remaining marked security |
| [slack-notify.yml](https://github.com/davidclin/cloudcustodian-policies/blob/master/slack-notify.yml) | Slack example |

# Cloud Custodian Architecture and AWS Services
<img src="./images/singlenodedeploy.png" width="550">

# Getting Started
<details>
<summary>Quick Install</summary>

```
*** Install repository***
$ git clone https://github.com/capitalone/cloud-custodian

*** Install dependencies (with virtualenv) ***
$ virtualenv c7n_mailer
$ source c7n_mailer/bin/activate
$ cd cloud-custodian/tools/c7n_mailer
$ pip install -r requirements.txt

*** Install extensions ***
$ python setup.py develop

*** Verify Installation ***
$ c7n-mailer
$ custodian
```
For more info, check out [Cloud Custodian in GitHub](https://github.com/capitalone/cloud-custodian)
</details>

# Usage
<details>
<summary>Getting Started</summary>

<pre>
Cloud Custodian must be run within a virtual environment.

$ cd ~
$ virtualenv c7n_mailer/bin/activate
$ cd cloudcustodian  (this is the IE/cloudcustodian repo where all the policies reside)

As a test, try
$ custodian run -s out mfa.yml
$ custodian report -s out mfa.yml --format grid

Cloud Custodian will create a log file in the ~/cloudcustodian/out/ subdirectory IF there are any matches. 
</pre>
</details>

# Environment Settings
<details>
<summary>mailer.yml</summary>

<pre>
# Which queue should we listen to for messages
queue_url: https://sqs.us-east-1.amazonaws.com/1234567890/sandbox

# Default from address
from_address: email@address.com

# Tags that we should look at for address infomation
contact_tags:
  - OwnerContact
  - OwnerEmail
  - SNSTopicARN

# Standard Lambda Function Config
region: us-east-1
role: arn:aws:iam::1234567890:role/CloudCustodianRole
slack_token: xoxb-bot_token_string_goes_here
</pre>
</details>

<details>
<summary>Cloud Custodian Lambda AWS Role</summary>
 
<pre>
Note: Based on your use case, additional permissions may be needed. 
Cloud Custodian will generate a msg if that is the case after invocation.

Trust relationship:
"Service": "lambda.amazonaws.com"

General policy permissions:
iam:PassRole
iam:ListAccountAliases
iam:ListUsers
iam:GetCredentialReport
ses:SendEmail
ses:SendRawEmail
lambda:CreateFunction
lambda:ListTags
lambda:GetFunction
lambda:AddPermission
lambda:ListFunctions
lambda:UpdateFunctionCode
events:DescribeRule
events:PutRule
events:ListTargetsByRule
events:PutTargets
events:ListTargetsByRule
tag:GetResources
cloudwatch:CreateLogGroup
cloudwatch:CreateLogStream
autoscaling:DescribeLaunchConfigurations
s3:GetBucketLocation
s3:GetBucketTagging
s3:GetBucketPolicy
s3:GetReplicationConfiguration
s3:GetBucketVersioning
s3:GetBucketNotification  
s3:GetLifeCycleConfiguration
s3:ListAllMyBuckets
s3:GetBucketAcl
s3:GetBucketWebsite
s3:GetBucketLogging 
s3:DeleteBucket 
</pre>
</details>

<details>
<summary>Slack Oauth Permissions for App with Bot User</summary>
  
<pre>
incoming-webhook
channels:history
channels:read
chat:write:bot
chat:write:user
groups:history
groups:read
im:write
users:read
users:read.email
</pre>
</details>

# Schemas Used
<details>
<summary>security-group</summary>

<pre>
(custodian) [hostname]$ custodian schema security-group
aws.security-group:
  actions: [auto-tag-user, delete, invoke-lambda, mark, mark-for-op, normalize-tag,
    notify, patch, put-metric, remove-permissions, remove-tag, rename-tag, tag, tag-trim,
    unmark, untag]
  filters: [and, default-vpc, diff, egress, event, ingress, json-diff, locked, marked-for-op,
    not, or, stale, tag-count, unused, used, value]
</pre>
</details>

<details>
<summary>iam-user</summary>

<pre>
(custodian) [hostname]$ custodian schema iam-user
aws.iam-user:
  actions: [delete, invoke-lambda, notify, put-metric, remove-keys]
  filters: [access-key, and, credential, event, group, mfa-device, not, or, policy,
    value]
</pre>
</details>

<details>
<summary>iam-role</summary>

<pre>
(custodian) [hostname]$ custodian schema iam-role
aws.iam-role:
  actions: [invoke-lambda, notify, put-metric]
  filters: [and, event, has-inline-policy, has-specific-managed-policy, no-specific-managed-policy,
    not, or, unused, used, value]
</pre>
</details>

<details>
<summary>ec2</summary>
<pre>

(custodian) [hostname]$ custodian schema ec2
aws.ec2:
  actions: [auto-tag-user, autorecover-alarm, invoke-lambda, mark, mark-for-op, modify-security-groups,
    normalize-tag, notify, put-metric, reboot, remove-tag, rename-tag, resize, set-instance-profile,
    snapshot, start, stop, tag, tag-trim, terminate, unmark, untag]
  filters: [and, default-vpc, ebs, ephemeral, event, health-event, image, image-age,
    instance-age, instance-uptime, marked-for-op, metrics, network-location, not,
    offhour, onhour, or, security-group, singleton, state-age, subnet, tag-count,
    termination-protected, value]
</pre>
</details>

# Artifacts
<details>
<summary>security-groups-unused.yml</summary>

<pre>
(custodian) [hostname]$ custodian run --dryrun -s . security-groups-unused.yml
2018-04-13 20:02:01,043: custodian.policy:INFO policy: security-groups-unused resource:security-group region:us-east-1 count:29 time:0.30

(custodian) [hostname]$ more ./security-groups-unused/resources.json | grep 'GroupName\|GroupId'
(custodian) [hostname]$ more ./security-groups-unused/resources.json | grep GroupName\"\:
    "GroupName": "rds-launch-wizard-5",
    "GroupName": "rds-launch-wizard",
    "GroupName": "rds-launch-wizard-2",
    "GroupName": "launch-wizard-17",
    "GroupName": "launch-wizard-5",
    "GroupName": "launch-wizard-7",
    "GroupName": "launch-wizard-6",
    "GroupName": "launch-wizard-1",
    "GroupName": "rds-launch-wizard-4",
    "GroupName": "launch-wizard-4",
    "GroupName": "launch-wizard-2",
    "GroupName": "launch-wizard-3",
    etc.
</pre>
</details>

<details>
<summary>iam.yml</summary>

<pre>
(custodian) [ec2-user@ip-10-100-0-195 custodian]$ custodian run --dryrun -s . iam.yml
2018-04-13 22:51:05,472: custodian.policy:INFO policy: iam-user-filter-policy resource:iam-user region:us-east-1 count:1 time:0.01

(custodian) [hostname]$ more ./iam-user-filter-policy/resources.json | grep UserName\"\:
    "UserName": "david.lin",
</pre>
</details>

<details>
<summary>mfa.yml</summary>

<pre>
(custodian) [hostname]$ custodian run --dryrun mfa.yml -s .
2018-04-13 23:47:40,901: custodian.policy:INFO policy: mfa-user-filter-policy resource:iam-user region:us-east-1 count:15 time:0.01

(custodian) [hostname]$ more ./mfa-user-filter-policy/resources.json | grep UserName\"\:
    "UserName": "username_1",
    "UserName": "username_2,
    "UserName": "username_3",
    "UserName": "username_4",
     etc.
</pre>
</details>

<details>
<summary>roles.yml</summary>

<pre>
(custodian) [hostname]$ custodian run --dryrun roles.yml -s .
2018-04-14 07:11:22,425: custodian.policy:INFO policy: iam-roles-unused resource:iam-role region:us-east-1 count:55 time:1.92

(custodian) [hostname]$ more ./iam-roles-unused/resources.json | grep RoleName
    "RoleName": "AmazonSageMaker-ExecutionRole-20180412T161207",
    "RoleName": "autotag-AutoTagExecutionRole-KA3LH5ARKJ2E",
    "RoleName": "autotag-AutoTagMasterRole-3VSL2AF3480E",
    "RoleName": "AWS-Cloudera-Infrastructu-ClusterLauncherInstanceR-1HUTDQJUYVGVE",
    etc.
</pre>
</details>

<details>
<summary>admin-group.yml</summary>

<pre>
(custodian) [hostname]$ custodian run --dryrun admin_group.yml -s .
2018-04-14 07:54:08,198: custodian.policy:INFO policy: iam-users-in-admin-group resource:iam-user region:us-east-1 count:14 time:3.67

(custodian) [hostname]$ more ./iam-users-in-admin-group/resources.json | grep UserName
    "UserName": "username_1",
    "UserName": "username_2",
    "UserName": "username_3",
    "UserName": "username_4",
    etc.
</pre>
</details>

<details>
<summary>mfa-unused.yml</summary>

<pre>
(custodian) [hostname]$ custodian run --dryrun mfa-unused.yml -s .
2018-04-14 08:13:07,214: custodian.policy:INFO policy: mfa-unused resource:iam-user region:us-east-1 count:2 time:2.54

(custodian) [ec2-user@ip-10-100-0-195 custodian]$ more ./mfa-unused/resources.json | grep UserName
    "UserName": "username_1",
    "UserName": "username_2"
</pre>
</details>

<details>
<summary>emailer.yml</summary>

<pre>
(custodian) [hostname]$ custodian run -s . emailer.yml
2018-04-23 22:25:12,614: custodian.policy:INFO policy: mfa-unused resource:iam-user region:us-east-1 count:2 time:8.41
2018-04-23 22:25:12,812: custodian.actions:INFO sent message:71ba67dd-731a-4734-bf63-15991754249e policy:mfa-unused template:default.html count:2
2018-04-23 22:25:12,813: custodian.policy:INFO policy: mfa-unused action: notify resources: 2 execution_time: 0.20
</pre>
</details>

<details>
<summary>public-subnet-instance-audit-notify.yml</summary>
<pre>
(custodian) $ custodian run -s . public-subnet-instance-audit-notify.yml
2018-05-04 01:07:56,937: custodian.policy:INFO Provisioning policy lambda public-subnet-instance-audit-notification
</pre>
</details>

# Usage Considerations
<a name="offhours"></a><details>
<summary>Offhour Examples</summary>

<pre>
-------------------------------------------------------
Option 1: Using a Single Tag with key = "maid_offhours"
-------------------------------------------------------
# up mon-fri from 7am-7pm; eastern time
off=(M-F,19);on=(M-F,7);tz=est
# up mon-fri from 6am-9pm; up sun from 10am-6pm; pacific time
off=[(M-F,21),(U,18)];on=[(M-F,6),(U,10)];tz=pt

<img align="center" src="./images/maid_hours.png" height="200" />

---------------------------------------------------------------------
Option 2: Using Tags with Names "StartAfterHours" and "StopAfterHours"
---------------------------------------------------------------------
# Using key "StartAfterHours"
# up mon-fri starting 7am; eastern time
on=(M-F,7);tz=est

#Using key "StopAfterHours"
# off mon-fri after 5pm; pacific time
off=(M-F,17);tz=pt

<img align="center" src="./images/custom_offhours.png" height="200" />

Important Note: When you stop an instance, the data on any instance store volumes is erased. 
                Therefore, if you have any data on instance store volumes that you want to 
                keep, be sure to back it up to persistent storage.

More Examples : http://capitalone.github.io/cloud-custodian/docs/quickstart/offhours.html#offhours
</pre>
</details>

<details>
<summary>Other Misc Usage Considerations</summary>

*copy-tag* and *tag-team* policies require addtional enhancements that were added to c7n/tags.py.
A modified version that tracks these changes can be found [here](https://github.com/capitalone/cloud-custodian/compare/master...mikegarrison:master).

*emailer.yml* requires the custodian mailer described [here](https://github.com/capitalone/cloud-custodian/tree/master/tools/c7n_mailer). 

*ebs-garbage-collection.yml* can be run across all regions with the --region all option.<p>
 
 For example: <br>
 
```
 custodian run --dryrun -s out --region all ebs-garbage-collection.yml
```
</details>

<details>
<summary>More</summary>

*offhours.yml* is run as a Lambda with CloudWatch periodic scheduler. It filters for EC2 instances tagged with "maid_offhours"
and obeys rules set forth in the corresponding value pair per [Cloud Custodian Offhours Policy](http://capitalone.github.io/cloud-custodian/docs/quickstart/offhours.html#offhours). When specifying on/off/tz values, the values in the policies are overrided by the EC2 instance maid_offhours tag. So you can set the onhour/offhour to anything in the policy and it will not do anything.  

*emailer.yml* requires the custodian mailer described [here](https://github.com/capitalone/cloud-custodian/tree/master/tools/c7n_mailer). 

*ebs-garbage-collection.yml* can be run across all regions with the --region all option.<p>
 
 For example: <br>
 
```
 custodian run --dryrun -s out --region all ebs-garbage-collection.yml
```
</details>

# Troubleshooting Tips
Use 'custodian validate' to find syntax errors<br>
Check 'name' of policy doesn't contain spaces<br>
Check SQS to see if Custodian payload is entering the queue<br>
Check cloud-custodian-mailer lambda CloudWatch rule schedule (5 minute by default)<br>
Check Lambda error logs (this requires CloudWatch logging)<br>
Check role for lambda(s) have adequate permissions<br>
Remember to update the cloud-custodian-mailer lambda when making changes to a policy that uses notifications<br>
Clear the cache if you encounter errors due to stale information (rm ~/.cache/cloud-custodian.cache)<br>

# Log Messages
If you see the following CloudWatch log when sending notifications via Slack, ignore it:<br>

<pre>
[WARNING]	2018-06-06T23:42:21.321Z	413b5506-69e3-11e8-8a8c-6f167e23dc1a	Error: An error occurred (InvalidCiphertextException) when calling the Decrypt operation: Unable to decrypt slack_token with kms, will assume plaintext.
</pre>

# Canned Code Cheatsheet
<details>
<summary>Invoking Lambda Funtions</summary>

```
mode:
  type: cloudtrail
  role: arn:aws:iam::929292782238:role/CloudCustodian
  events:
    - CreateBucket
```

```
mode:
  type: periodic
  role: arn:aws:iam::929292782238:role/CloudCustodian
  schedule: "rate(15 minutes)"```
```
</details>

<details>
<summary>Sending Notifications via SES and Slack</summary>
  
```
actions:
 - type: notify
   template: default.html
   slack_template: slack-default
   template_format: 'html'
   priority_header: '5'
   subject: 'Security Audit: Unused Security Groups'
   to:
     - <your-email-address-goes-here>
     - slack://#<slack-channel-name>
   owner_absent_contact:
     - <your-emails-address-goes-here>
   transport:
     type: sqs
     queue: https://sqs.us-east-1.amazonaws.com/1234567890/cloud-cloudcustodian
```
</details>

<details>
<summary>Filtering with regex and whitelist</summary>
  
```
filters:
  - not:
    - type: value
      key: "tag:Name"
      value: (MyJenkinsInstance|MyCloudCustodianInstance)
      op: regex
  - and:
    - type: subnet 
      key: "tag:Name"
      value: "david.lin-subnet" 
```
</details>

# Updating Latest Merges to Master
From your virtualenv

```
cd ~/cloud-custodian
git pull
python setup.py install
```
This will reflect changes in your virtualenv Python lib such that the schema validation uses the latest fixes/updates.

# Resources
[Custom msg-templates for c7n_mailer](https://github.com/capitalone/cloud-custodian/issues/1127)<br>
[Slack API and Token](https://github.com/capitalone/cloud-custodian/issues/2340)<br>
[Using ec2-instance-state, lessons around roles, how to view lambda logs, and more](https://github.com/capitalone/cloud-custodian/issues/2321)<br>
[How does garbage collection get enforced?](https://github.com/capitalone/cloud-custodian/issues/2384)<br>
[EC2 Offhours Support](http://capitalone.github.io/cloud-custodian/docs/usecases/ec2offhours.html#ec2offhours)<br>
[Example offhours support](http://capitalone.github.io/cloud-custodian/docs/quickstart/offhours.html)<br>
[Lambda Support](http://capitalone.github.io/cloud-custodian/docs/policy/lambda.html)<br>
[AWS CloudWatch Schedule Rules](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html)<br>
[iam-user feature enhancement](https://github.com/capitalone/cloud-custodian/pull/2454)<br>
[Offhours Examples](http://capitalone.github.io/cloud-custodian/docs/quickstart/offhours.html)<br>
[CloudWatch Rules Expressions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html)<br>
