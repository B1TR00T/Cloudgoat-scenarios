# EC2 SSRF

This scenario provides aws creds 
![Screenshot](screenshots/Pasted%20image%2020260715113235.png)
These creds are for a user called solus
![Screenshot](screenshots/Pasted%20image%2020260715113324.png)
I used aws-enumerator to check what services and modules can this account use.
![Screenshot](screenshots/Pasted%20image%2020260715114334.png)
![Screenshot](screenshots/Pasted%20image%2020260715114346.png)
All of the modules of these 2 services can be used by this account the most important one is lambda so i checked what are the functions
![Screenshot](screenshots/Pasted%20image%2020260715115429.png)
Here is a lambda python function with hardcoded ec2 instance credentials
I used it to access the new account
![Screenshot](screenshots/Pasted%20image%2020260715115736.png)
Enumerating this account I found that it can use 69 functions out of 74 of the EC2 instance service
![Screenshot](screenshots/Pasted%20image%2020260715120920.png)
One of them is *describe-instances* function this showed the details of all available instances for the this user such as the ips both public and private and public and private dns names and the OS which amazon linux according to the AMI
![Screenshot](screenshots/Pasted%20image%2020260715121756.png)
![Screenshot](screenshots/Pasted%20image%2020260715121825.png)
![Screenshot](screenshots/Pasted%20image%2020260715122028.png)
I nmaped the public ip address and found 2 open services ssh and http
![Screenshot](screenshots/Pasted%20image%2020260715122112.png)
Checking the http service i found this web site 
![Screenshot](screenshots/Pasted%20image%2020260715123334.png)
EC2 instances are know to have something called IMDS (Instance MetaData Service) which is a service that has the metadata about that instance. This service can be accessed by Its designated ip which is *169.254.169.254*. There are 2 versions of this service IMDSv1 and IMDSv2 **v1** allows metadata access via simple `GET` requests (no tokens), making it vulnerable to SSRF attacks.
**v2** requires a `PUT` request to create a session token (with `X-aws-ec2-metadata-token-ttl-seconds`), then uses that token in subsequent `GET`requests.
This website explicitly mentions a web vulnerability called SSRF (Server Side Request Forgery) which is a vulnerability that allow the server to execute commands on behalf of the attacker.
Based on all of this the attack path is clear. Use the SSRF vulnerability to access the IMDS.
![Screenshot](screenshots/Pasted%20image%2020260715124134.png)
The vulnerability worked it displayed the instance's updates one of them is latest.
![Screenshot](screenshots/Pasted%20image%2020260715124308.png)
Accessing latest I found the meta-data folder
![Screenshot](screenshots/Pasted%20image%2020260715124419.png)
In this folder I found the iam folder that has temporary ec2 creds
![Screenshot](screenshots/Pasted%20image%2020260715125614.png)
I used these creds to access the new account 
![Screenshot](screenshots/Pasted%20image%2020260715125638.png)
This account has some controle over the S3 service
![Screenshot](screenshots/Pasted%20image%2020260715130104.png)
![Screenshot](screenshots/Pasted%20image%2020260715130121.png)
I found a secret s3 bucket. In this bucket there another file that has credentials
![Screenshot](screenshots/Pasted%20image%2020260715130423.png)
I accessed this account but before that i must delete the temporary token
![Screenshot](screenshots/Pasted%20image%2020260715130604.png)
The enumeration of this account showed that it can use 4 lambda modules one of them is listfunctions like earlier. Now with this account we can invoke the previous lambda function to get this message indicating that the scenario is done.
![Screenshot](screenshots/Pasted%20image%2020260715133512.png)

--- 
## **Attack Chain Summary**

Starting from leaked credentials for IAM user `solus`, enumeration revealed a Lambda function with hardcoded EC2 credentials for a second user, `wrex`. Those credentials exposed a public EC2 web server vulnerable to SSRF, which was used to reach the instance's IMDS endpoint and steal temporary role credentials. The stolen role granted access to a private S3 bucket holding credentials for a third user, `shepard`, whose Lambda invoke rights completed the scenario. No persistence was established - this was a linear four-hop chain across three identities and four AWS services (Lambda, EC2, IMDS, S3), and the SSRF-to-IMDS step is exactly the path IMDSv2 enforcement is designed to close.

