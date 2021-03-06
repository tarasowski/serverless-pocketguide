# AWS Account Security with CloudTrail, CloudWatch, and Amazon S3
## AWS Pop-Up Loft Berlin | October 10 – 19 | Berlin

* The Swiss cheese model of accident causation illustrates that, although many layers of defense lie between hazards and accidents, there are flaws in each layer that, if aligned, can allow the accident to occur.

[Source](https://en.wikipedia.org/wiki/Swiss_cheese_model)

* There will be no 100% of security, but we need to able to dectect, reat and recover from security leak. 

* Hazard leads to UOS (undesired operational outcome) which leads to an outcome (a leak)

* CloudTrail will store logs in s3, in order to audit logs we need IAM roles. You need IAM to go into logs and to aanlyze them.

* We also have Log File Validation, if someon breaks into you account, does something harmful and goes into logs and change the.

* We will create Cloudwatch events that monitor the logs and whenever they will be changed or delted so we know that something has happened.

* You can connect lambda to cloudwatch and if someone wants to modify logs, the lambda function deletes the user, role or policy of the property thtat tries to modify the logs.

* If you use multi-account strategy, you need to aggregate all the logs in another account which has the rights to aggregate all the logs. We can call is a security account! A few people in the company have the sdecurity confiuration, and this account can onitor all other accounts. If someone breaks into an account they cannot turn off the logs from the security account.

* We need to give a s3 bucket the right to write, other users the right to read, so no one can delete the logs

* If you have basic encryption, the encryption is transparent. ES256 the data is encrypted on the device it's stored. If you use KMS, you need to decrypt/encrypt right to the key. You need the access rights on the bucket and the rights on the key (KMS).

* Every API Call to a specific resource is going to be loged into the CloudTrail. The default log happens into the CloudTrail. In the console you have the recent events, or you have the event history and it's always enabled. Not including s3 events, not including lambda function calls. All the events, calls are stored except s3 and lambda, you need to activate them manually.

* CloudTrail you only pay for reading the logs, you pay for storage. If you have to much data you can think about lifecycle management, after 31 move to Glacier. 

* We have versioning on s3 because the file shouldn't be deleted or modified, otherwise if this is happening someone has hacked your account. So that's why we have versioning active. 

* If you do log file analytics search for `Denied` to see if some calls where denied, to see if someone is trying to access the account.

* If you start writing the application you start with blank permissions, no one is able to change anything. You start with `deny` and then just add more permissions to the resources as needed. If I create a specific resource and I don't know which permissions are needed. You go into the logs and see which permissions are needed, you look who is trying to call and is getting denied by the service. You run a query through your log file and search for `denied` to see which actions needs to be allowed.

* If you write logs you need to access the workers councel (Betriebsrat) what data can be tracked and analysed. They will make a decision to what to do with the log files. You usually move the logs to a separate account and it can be only accessed with a worker council person.

* For security reasons is important to aggregate the logs into specific security account so no one can break into this account. You can then analyse the logs with Ahtena, Splunk, ElasticSearch, Kibana etc.

* Now we need to create different roles: Admin who can change the logs, and Auditor who can audit the logs.

* IAM: 
a) root can do everyhting
b) below root you have users, groups, roles = they all are entetites.
c) below users, groups, roles you have policies = permisisons are codified in policies

* You can apply many policies as you want to users, to groups and to roles. The difference between: a user is a person who is managed inside IAM can log into the console. A user is always a person, and a group is a collections of users.

* A role can be assumed by a user or a service. If the lambda function wants to read from s3 bucket it assumes a role. And the permission of the role defines what lambda can do or cannot do. A role needs to be assumed, a user needs to be logged in. 

* In a production account you will only have 1 user. And this user is the user of cloudformation that provisions your resources. There should be no reason to log into your production account. If you need to log in you will have certain roles that can be assumed. You need always create different account that can assume the role to see specific things e.g. billing details. You just create bunch of accounts for different things like finance, security etc. [More Information here](https://aws.amazon.com/answers/account-management/aws-multi-account-security-strategy/)

* You need a specific user that can deploy the cloudformation the resources and this user will have only the access key and this will be used by the user (cloudformation). 

* There is a subset of entities which are called principals, these all threee are entities, but onyl roles and users are principles. A group cannot be a principal, a user can be a principal and a role can be a principal. 

* Deny always overwrites the allow rule! If there is no explicit allow it's also deny. Use the principle of least privilege and always allow what is necessary. 

**Note:** Sid is the statement id, each statement has an id, if you have multiple statements you need an identfiier for statements. If you have only 1 statement you don't need sid if you have multiple statements you need a statement id

* If there is a prinicpal inside a policy, it means we are giving a user or a role to do something on the specific resouerce we have defined in the policy. 

* **AWS managed policies they SHOULDN'T be used.** For two reasons, because they could change at any time and if aws implements new features and changes the policy could break your application. Second reason, they most of the time they are to open, they are too broad. Default managed policies by AWS are way to broad, way to open. 

* You always want to create customer based policies and they will never changed. You can also deny or allow the things you want to, they are more specific and times better in terms of security. You can create your own policy and can use conditions to narrow it down e.g. give access between 9am and 10pm, but not during the night. 

* Don't use ACL in s3, use always bucket policy for giving access. Also use cloudfront and let people access everything through cloudfront. And Cloudfront then does the API call to s3. 

* Simple urls are going through ACL and api calls are going through Policy. So you can mess up the rights if you use both they you can mess up everything. So always use only s3 bucket in conjunction with the bucket policy, not ACL!!!

* Execution role what lambda is allowed to do and function role what resource can do with the lambda itself. [See more](https://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html)

* Execution role what lambda is allowed to do and function role what resource can do with the lambda itself. [See more](https://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html)

* If there is trust relation ship `Principal: ....:root` eveyone in this account can assume this role. So you need to specify a specific accout who can assume this role. 

* When we creates a CloudTrail, we have activated log file validation. This creates digest files, these are hashes for every log file that has been written. For every log file we have a fitting digest file, which contains a hash. If someone modifies a file after the fact, we can see it. We can do it by running a lambda function that checks the files every 24 hours. You need a TrailARN (every Amazon has an ARN) - `aws cloudtrail validate-logs --start-time 20180101 --trail-arn arn:aws:cloudtrail:.....`by running this command you can validate the logs to see if they are not modified by anyone. e.g. if you modify or delete a file, by running validation you can see that something was deleted.

* Create a SNS topic `security` and put all security related things to that topic. You can put a message to a topic via Cloudwatch Event Rules. Event Pattern:

- Service Name: S3
- Event Type: LevelObjectOperation
- Specific operation: DeleteObject, DeleteObject, DeleteBucket on the bucket where CloudTrail writes logs

* I want to put a message on my security topic. If deletion happens, I'll get an email from SNS that someone deleted an object.

* You should subscribe a Lambda function to the `security` topic and if the user tries to delete something, the lambda function deletes all the rights and removes the access from the user that is trying to malicious your account. Most of the operöational work is about creating of monitoring and notifying people if something happens when it shouldn't. 

* CIS Benchmark on AWS: Deploy a standardized architecture for the CIS AWS Foundations Benchmark [Source](https://aws.amazon.com/quickstart/architecture/compliance-cis-benchmark/)

* Activate GuardDuty: Amazon GuardDuty is a threat detection service that continuously monitors for malicious or unauthorized behavior to help you protect your AWS accounts and workloads. It monitors for activity such as unusual API calls or potentially unauthorized deployments that indicate a possible account compromise. [Source](https://aws.amazon.com/guardduty/)

* In the bucket policy we need explicitly to set `deny` deletion of any objects inside the bucket. Principal `*` everyone will be denied when they try to delete object version, deleteobject, deleteobject, deletebuckets. The resources are you bucket. Also one another rule, codition {Boolifexits: {aws: MultifactorAuthFalse} (means you cannot delete anything without MFA token). 

* Every layer in security has holes, but the more layers I have the more tight are my security measures!

* In order to save consts, you need to archive data after e.g. 31 days to Glacier. Use s3 lifecycle rules for this.

### Additional Resources:
* [AWS Answers Account Management / Security ](https://aws.amazon.com/answers/account-management/)
* [Boilerplate CloudFormation Security Templates](https://aws.amazon.com/quickstart/architecture/compliance-cis-benchmark/)
