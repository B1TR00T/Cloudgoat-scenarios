In this scenario we are provided with a user creds.
Using a tool called shadowperms I found that this user have permission to use a lot of iam modules and only 2 sts modules nothing else.
![[Pasted image 20260724111110.png]]
Checking the roles. I found a role that allows the user to assume it 
![[Pasted image 20260724111742.png]]
It gave us temporary creds to use this role.
![[Pasted image 20260724112554.png]]
The role was successfully assumed.
![[Pasted image 20260724112839.png]]
I found a lambda function that allows to set a policy for a user validated against a db.
![[Pasted image 20260724114314.png]]
Getting that function provided a link that automatically downloads a zip file.
![[Pasted image 20260724114805.png]]
The downloaded file was the lambda function source code.
![[Pasted image 20260724114826.png]]
Checking the main python script I found interesting things
![[Pasted image 20260724115434.png]]
In this script. We can see the available policies one of them is adiministrator_access publi is set to false and the others are set to true.
However, in this section the script will get a policy from the list and has public "true" and assign it to the user. But the for function uses the statement variable without sanitization meaning we can use SQL injection vulnerabitlity to use the adiministration_access.
![[Pasted image 20260724115751.png]]
But before that I checked the db but found only the policies
![[Pasted image 20260724120339.png]]
I used SQL injection while invoking the lambda funciton which resulted in giving admin privs to user bilbo
![[Pasted image 20260725104210.png]]
![[Pasted image 20260724121623.png]]
As shown here no bilbo can use much more services
![[Pasted image 20260724122626.png]]
Since we have more power now. I checked some services like secretsmanager and found one!
![[Pasted image 20260725111147.png]]
And in that secret I found the flag value.
![[Pasted image 20260725111408.png]]
