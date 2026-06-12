# IAM Privilege Escalation by Rollback

After the deployment of this scenario i got creds for a user called raynor
I used these creds to access the account
![Raynor account access](screenshots/Pasted%20image%2020260612124933.png)
So instead of trying manually checking for the permission this account have i use aws-enumarator tool and found that this account can do 18/24 IAM operation and 2/2 sts operations and 1/5 dynamodb oprations

Here is the operations that i found
![Enumerated permissions](screenshots/Pasted%20image%2020260612125241.png)
i tried to list users but i didn't have enough permissions
![List users denied](screenshots/Pasted%20image%2020260612132612.png)
I checked the policy attached to the user
![Attached policy check](screenshots/Pasted%20image%2020260612130823.png)
![Policy details](screenshots/Pasted%20image%2020260612130843.png)
I checked this policy's versions and found 5
![Policy versions](screenshots/Pasted%20image%2020260612131342.png)
in v3 i found the admin permissions
![Admin permissions in policy version](screenshots/Pasted%20image%2020260612132045.png)
but when i tried to set it as the default permissions but i couldn't so i used my root user to set the v1 as the default permission then escalate from there
![Set default policy version](screenshots/Pasted%20image%2020260612135031.png)
![Default policy version changed](screenshots/Pasted%20image%2020260612135053.png)

![Escalation verification](screenshots/Pasted%20image%2020260612134633.png)
![Admin access verification](screenshots/Pasted%20image%2020260612134725.png)

=> and like that i got the admin access
