#                     **ec2_ssrf**

This scenario provides aws creds after deployement
These creds are for a user called solus
![[01-initial-access-solus-identity.png]]
I used aws-enumerator to check what services and modules can this account use.
![[02-enum-lambda-sts-services.png|154]]
All of the modules of this service can be used by this account. The most important one is lambda list-functions.
![[03-lambda-list-functions-hardcoded-creds.png|697]]
Here I found a lambda python function with hardcoded ec2 instance credentials.
I used it to access the new account.
![[04-wrex-identity-confirmed.png]]
Enumerating this account I found that it can use 69 functions out of 74 of the EC2 instance service
![[05-enum-ec2-service-permissions.png]]
One of them is *describe-instances* function this showed the details of all available instances for the this user such as the ips both public and private and public and private dns names and the OS which amazon linux according to the AMI
![[06-describe-instances-ip-output.png]]
![[06-describe-instances-DNS-output.png]]
![[06-describe-instances-ami-output.png]]
I nmaped the public ip address and found 2 open services ssh and http
![[07-nmap-scan-results.png]]
Checking the http service i found this web site 
![[08-ssrf-demo-webapp.png]]
EC2 instances are know to have something called IMDS (Instance MetaData Service) which is a service that has the metadata about that instance. This service can be accessed by Its designated ip which is *169.254.169.254*. There are 2 versions of this service IMDSv1 and IMDSv2 **v1** allows metadata access via simple `GET` requests (no tokens), making it vulnerable to SSRF attacks.
**v2** requires a `PUT` request to create a session token (with `X-aws-ec2-metadata-token-ttl-seconds`), then uses that token in subsequent `GET`requests.
This website explicitly mentions a web vulnerability called SSRF (Server Side Request Forgery) which is a vulnerability that allow the server to execute commands on behalf of the attacker.
Based on all of this the attack path is clear. Use the SSRF vulnerability to access the IMDS.
![[09-ssrf-request-to-imds-root.png]]
The vulnerability worked it displayed the instance's updates one of them is latest.
![[10-imds-latest-folder.png]]
Accessing latest I found the meta-data folder
![[10-imds-latest-metadata-folder.png]]
In this folder I found the iam folder that has temporary ec2 creds
![[11-imds-iam-security-credentials.png]]
I used these creds to access the new account 
![[12-role-creds-identity-confirmed.png]]
This account has some controle over the S3 service
![[13-s3-bucket-discovery.png]]
![[13-s3-bucket-discovery.png]]
I found a secret s3 bucket. In this bucket there another file that has credentials
![[14-s3-credentials-file-extracted.png]]
I accessed this account but before that i must delete the temporary token
![[15-shepard-identity-confirmed.png]]
The enumeration of this account showed that it can use 4 lambda modules one of them is listfunctions like earlier. Now with this account we can invoke the previous lambda function to get this message indicating that the scenario is done.
![[16-scenario-complete-output.png]]

--- 
## **Attack Chain Summary**

Starting from leaked credentials for IAM user `solus`, enumeration revealed a Lambda function with hardcoded EC2 credentials for a second user, `wrex`. Those credentials exposed a public EC2 web server vulnerable to SSRF, which was used to reach the instance's IMDS endpoint and steal temporary role credentials. The stolen role granted access to a private S3 bucket holding credentials for a third user, `shepard`, whose Lambda invoke rights completed the scenario. No persistence was established — this was a linear four-hop chain across three identities and four AWS services (Lambda, EC2, IMDS, S3), and the SSRF-to-IMDS step is exactly the path IMDSv2 enforcement is designed to close.