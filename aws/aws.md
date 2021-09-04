# AWS CLI Cheatsheet

## Setup

### install Virtualbox Guest Additions, passwordless sudo
```shell
echo $USER
sudo echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers
sudo su
apt-get update
apt-get install -y build-essential dkms linux-headers-$(uname -r)
cd /media/aws-admin/
sh ./VBoxLinuxAdditions.run
shutdown now
```

### install AWS CLI
```shell
sudo apt-get install -y python-dev python-pip
sudo pip install awscli
aws --version
aws configure
```

## Cloudtrail - Logging and Auditing

5 Trails total, with support for resource level permissions

```shell
# list all trails
aws cloudtrail describe-trails

# list all S3 buckets
aws s3 ls

# create a new trail
aws cloudtrail create-subscription \
    --name awslog \
    --s3-new-bucket awslog2016

# list the names of all trails
aws cloudtrail describe-trails --output text | cut -f 8

# get the status of a trail
aws cloudtrail get-trail-status \
    --name awslog

# delete a trail
aws cloudtrail delete-trail \
    --name awslog

# delete the S3 bucket of a trail
aws s3 rb s3://awslog2016 --force

# add tags to a trail, up to 10 tags
aws cloudtrail add-tags \
    --resource-id awslog \
    --tags-list "Key=log-type,Value=all"

# list the tags of a trail
aws cloudtrail list-tags \
    --resource-id-list 

# remove a tag from a trail
aws cloudtrail remove-tags \
    --resource-id awslog \
    --tags-list "Key=log-type,Value=all"
```
<br/><br/><br/>

## IAM

### Users

```shell
# list all user's info
aws iam list-users

# list all user's usernames
aws iam list-users --output text | cut -f 6

# list current user's info
aws iam get-user

# list current user's access keys
aws iam list-access-keys

# crate new user
aws iam create-user \
    --user-name aws-admin2

# create multiple new users, from a file
allUsers=$(cat ./user-names.txt)
for userName in $allUsers; do
    aws iam create-user \
        --user-name $userName
done

# list all users
aws iam list-users --no-paginate

# get a specific user's info
aws iam get-user \
    --user-name aws-admin2

# delete one user
aws iam delete-user \
    --user-name aws-admin2

# delete all users
# allUsers=$(aws iam list-users --output text | cut -f 6);
allUsers=$(cat ./user-names.txt)
for userName in $allUsers; do
    aws iam delete-user \
        --user-name $userName
done
```



### Password policy

http://docs.aws.amazon.com/cli/latest/reference/iam/

```shell
# list policy
aws iam get-account-password-policy

# set policy
aws iam update-account-password-policy \
	--minimum-password-length 12 \
	--require-symbols \
	--require-numbers \
	--require-uppercase-characters \
	--require-lowercase-characters \
	--allow-users-to-change-password

# delete policy
aws iam delete-account-password-policy
```

### Access Keys

http://docs.aws.amazon.com/cli/latest/reference/iam/

```shell
# list all access keys
aws iam list-access-keys

# list access keys of a specific user
aws iam list-access-keys \
    --user-name aws-admin2

# create a new access key
aws iam create-access-key \
    --user-name aws-admin2 \
    --output text | tee aws-admin2.txt

# list last access time of an access key
aws iam get-access-key-last-used \
    --access-key-id AKIAINA6AJZY4EXAMPLE

# deactivate an acccss key
aws iam update-access-key \
    --access-key-id AKIAI44QH8DHBEXAMPLE \
    --status Inactive \
    --user-name aws-admin2

# delete an access key
aws iam delete-access-key \
    --access-key-id AKIAI44QH8DHBEXAMPLE \
    --user-name aws-admin2
```

### Groups, Policies, Managed Policies

http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html
http://docs.aws.amazon.com/cli/latest/reference/iam/

```shell
# list all groups
aws iam list-groups

# create a group
aws iam create-group --group-name FullAdmins

# delete a group
aws iam delete-group \
    --group-name FullAdmins

# list all policies
aws iam list-policies

# get a specific policy
aws iam get-policy \
    --policy-arn <value>

# list all users, groups, and roles, for a given policy
aws iam list-entities-for-policy \
    --policy-arn <value>

# list policies, for a given group
aws iam list-attached-group-policies \
    --group-name FullAdmins

# add a policy to a group
aws iam attach-group-policy \
    --group-name FullAdmins \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# add a user to a group
aws iam add-user-to-group \
    --group-name FullAdmins \
    --user-name aws-admin2

# list users, for a given group
aws iam get-group \
    --group-name FullAdmins

# list groups, for a given user
aws iam list-groups-for-user \
    --user-name aws-admin2

# remove a user from a group
aws iam remove-user-from-group \
    --group-name FullAdmins \
    --user-name aws-admin2

# remove a policy from a group
aws iam detach-group-policy \
    --group-name FullAdmins \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# delete a group
aws iam delete-group \
    --group-name FullAdmins
```
<br/><br/><br/>

## S3

https://docs.aws.amazon.com/cli/latest/reference/s3api/index.html#cli-aws-s3api

```shell
# list existing S3 buckets
aws s3 ls

# create a bucket name, using the current date timestamp
bucket_name=test_$(date "+%Y-%m-%d_%H-%M-%S")
echo $bucket_name

# create a public facing bucket
aws s3api create-bucket --acl "public-read-write" --bucket $bucket_name

# verify bucket was created
aws s3 ls | grep $bucket_name

# check for public facing s3 buckets (should show the bucket name you created)

aws s3api list-buckets --query 'Buckets[*].[Name]' --output text | xargs -I {} bash -c 'if [[ $(aws s3api get-bucket-acl --bucket {} --query '"'"'Grants[?Grantee.URI==`http://acs.amazonaws.com/groups/global/AllUsers` && Permission==`READ`]'"'"' --output text) ]]; then echo {} ; fi'

# check for public facing s3 buckets, updated them to be private

aws s3api list-buckets --query 'Buckets[*].[Name]' --output text | xargs -I {} bash -c 'if [[ $(aws s3api get-bucket-acl --bucket {} --query '"'"'Grants[?Grantee.URI==`http://acs.amazonaws.com/groups/global/AllUsers` && Permission==`READ`]'"'"' --output text) ]]; then aws s3api put-bucket-acl --acl "private" --bucket {} ; fi'

# check for public facing s3 buckets (should be empty)

aws s3api list-buckets --query 'Buckets[*].[Name]' --output text | xargs -I {} bash -c 'if [[ $(aws s3api get-bucket-acl --bucket {} --query '"'"'Grants[?Grantee.URI==`http://acs.amazonaws.com/groups/global/AllUsers` && Permission==`READ`]'"'"' --output text) ]]; then echo {} ; fi'
```

## EC2

### keypairs

http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html

```shell
# list all keypairs
aws ec2 describe-key-pairs

# create a keypair
aws ec2 create-key-pair \
    --key-name <value> --output text

# create a new local private / public keypair, using RSA 4096-bit
ssh-keygen -t rsa -b 4096

# import an existing keypair
aws ec2 import-key-pair \
    --key-name keyname_test \
    --public-key-material file:///home/apollo/id_rsa.pub

# delete a keypair
aws ec2 delete-key-pair \
    --key-name <value>
```

### Security Groups

http://docs.aws.amazon.com/cli/latest/reference/ec2/index.html

```shell
# list all security groups
aws ec2 describe-security-groups

# create a security group
aws ec2 create-security-group \
    --vpc-id vpc-1a2b3c4d \
    --group-name web-access \
    --description "web access"

# list details about a securty group
aws ec2 describe-security-groups \
    --group-id sg-0000000

# open port 80, for everyone
aws ec2 authorize-security-group-ingress \
    --group-id sg-0000000 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/24

# get my public ip
my_ip=$(dig +short myip.opendns.com @resolver1.opendns.com);
echo $my_ip

# open port 22, just for my ip
aws ec2 authorize-security-group-ingress \
    --group-id sg-0000000 \
    --protocol tcp \
    --port 80 \
    --cidr $my_ip/24

# remove a firewall rule from a group
aws ec2 revoke-security-group-ingress \
    --group-id sg-0000000 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/24

# delete a security group
aws ec2 delete-security-group \
    --group-id sg-00000000
```

## Images

https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-images.html

```shell
# list all private AMI's, ImageId and Name tags
aws ec2 describe-images --filter "Name=is-public,Values=false" \
    --query 'Images[].[ImageId, Name]' \
    --output text | sort -k2

# delete an AMI, by ImageId
aws ec2 deregister-image --image-id ami-00000000

```

## Instances

http://docs.aws.amazon.com/cli/latest/reference/ec2/index.html

```shell
# list all instances (running, and not running)
# http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html
aws ec2 describe-instances

# list all instances running
aws ec2 describe-instances --filters Name=instance-state-name,Values=running

# create a new instance
aws ec2 run-instances \
    --image-id ami-f0e7d19a \	
    --instance-type t2.micro \
    --security-group-ids sg-00000000 \
    --dry-run

# stop an instance
aws ec2 terminate-instances \
    --instance-ids <instance_id>

# list status of all instances
aws ec2 describe-instance-status

# list status of a specific instance
aws ec2 describe-instance-status \
    --instance-ids <instance_id>
    
# list all running instance, Name tag and Public IP Address
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query 'Reservations[].Instances[].[PublicIpAddress, Tags[?Key==`Name`].Value | [0] ]' \
  --output text | sort -k2
```

### Tags

```shell
# list the tags of an instance
aws ec2 describe-tags

# add a tag to an instance
aws ec2 create-tags \
    --resources "ami-1a2b3c4d" \
    --tags Key=name,Value=debian

# delete a tag on an instance
aws ec2 delete-tags \
    --resources "ami-1a2b3c4d" \
    --tags Key=Name,Value=
```
<br/><br/><br/>

## Cloudwatch

### Log Groups

##### create a group
```shell
aws logs create-log-group \
	--log-group-name "DefaultGroup"
```

##### list all log groups
```shell
aws logs describe-log-groups

aws logs describe-log-groups \
	--log-group-name-prefix "Default"
```

##### delete a group
```shell
aws logs delete-log-group \
	--log-group-name "DefaultGroup"
```



### Log Streams
```shell
# create a log stream
aws logs create-log-stream \
	--log-group-name "DefaultGroup" \
	--log-stream-name "syslog"

# list details on a log stream
aws logs describe-log-streams \
	--log-group-name "syslog"

aws logs describe-log-streams \
	--log-stream-name-prefix "syslog"

# delete a log stream
aws logs delete-log-stream \
	--log-group-name "DefaultGroup" \
	--log-stream-name "Default Stream"
```


EC2
===

Generate ssh key
----------------

To be used for a user in an EC2 instance:
```
$ ssh-keygen -b 4096 -t rsa -f ~/.ssh/$(whoami)-<suffix-for-ec2-environment>
$ ssh-keygen -t rsa -b 4096 -C "<your_email>"
```

```
$ aws ec2 describe-instances --instance-ids i-2a85b2c0 i-2cb27669  --profile <PROFILE_NAME>
```

Describe AMIs
-------------

```
$ aws ec2 describe-images --image-ids ami-7923a66e --profile <PROFILE_NAME>
```

AWS Kinesis
===========

```
$ aws --profile <PROFILE_NAME> kinesis list-streams
$ aws --profile <PROFILE_NAME> kinesis describe-stream --stream-name <YOUR_STREAM_NAME>
$ aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name <YOUR_STREAM_NAME> --profile <PROFILE_NAME>
$ KINESIS_SHARD_ITERATOR=$(aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name <YOUR_STREAM_NAME> --query 'ShardIterator' --profile <PROFILE_NAME>)
$ echo $KINESIS_SHARD_ITERATOR
$ aws kinesis get-records --shard-iterator $KINESIS_SHARD_ITERATOR --profile <PROFILE_NAME>
$ aws kinesis get-records --shard-iterator $KINESIS_SHARD_ITERATOR --profile <PROFILE_NAME> --limit 2
```

AWS S3
======

List S3 buckets
---------------

```
$ aws s3 ls --profile <PROFILE_NAME>
```

Download a file from a bucket
-----------------------------

```
$ aws s3 --profile <PROFILE_NAME> cp s3://<HOST_NAME>/<PATH>/foo.gz ~/foo.gz
```

Upload a file to a bucket
-------------------------

```
$ aws s3 --profile <PROFILE_NAME> cp s3://<HOST_NAME>/<PATH>/foo.gz . # copy from a remote bucket to here
$ aws --profile <PROFILE_NAME> s3 cp <LOCAL_FILE>.gz s3://<BUCKET_NAME>/ # copy from a local file to another remote bucket
```

Security groups
===============

```
$ aws ec2 describe-security-groups --group-names <SECURITY_GROUP_NAME> --profile <PROFILE_NAME>
```

Load balancers
==============

```
$ aws elb describe-load-balancers --load-balancer-name <SECURITY_GROUP_NAME> --profile <PROFILE_NAME>
```

SNS
===

```
$ aws sns list-topics --profile <PROFILE_NAME>
$ aws sns get-topic-attributes --topic-arn "arn:aws:sns:us-east-1:945109781822:<custom_suffix>" --profile <PROFILE_NAME>
$ aws sns list-subscriptions --profile <PROFILE_NAME>
$ aws sns get-subscription-attributes --subscription-arn "arn:aws:sns:us-east-1:945109781822:<custom_part>:6d92f5d3-f299-485d-b6fb-1aca6d9a497c" --profile <PROFILE_NAME>
```

CloudWatch
==========

```
$ aws cloudwatch describe-alarms --alarm-names <CUSTOM_NAME> --profile <PROFILE_NAME>
```

Logs
----

```
$ aws logs get-log-events --log-group-name '/aws/lambda/<CUSTOM_SUFFIX>' --log-stream-name '2016/06/16/[$LATEST]283ba30f8dcb44268f7090c2b7f38b5b' --output text --profile <PROFILE_NAME> > a.log 
```

AWS Lambda
==========

```
$ aws lambda get-function-configuration --function-name <CUSTOM_FUNCTION_NAME> --profile <PROFILE_NAME>
```

IAM Roles
=========

Describe a role
---------------

```
$ aws iam get-role --role-name <ROLE_NAME> --profile <PROFILE_NAME>
```

Describe a policy associated with that role
-------------------------------------------

```
$ aws iam get-role-policy --role-name <ROLE_NAME> --policy-name <POLICY_NAME> --profile <PROFILE_NAME>
```

Autoscaling
===========

Describe launch configurations
------------------------------

```
$ aws autoscaling describe-launch-configurations --launch-configuration-names <CUSTOM_CONFIGURATION_NAME> --profile <PROFILE_NAME>
```

Describe autoscaling groups
---------------------------

```
$ aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name <CUSTOM_AG_NAME> --profile <PROFILE_NAME>
```

Describe autoscaling policies
-----------------------------

```
$ aws autoscaling describe-policies --auto-scaling-group-name <CUSTOM_AG_NAME> --profile <PROFILE_NAME>
```

RDS
===

RDS is the MySQL version on Amazon.

```
$ aws rds describe-db-security-groups --db-security-group-name <DB_SG_NAME> --profile <PROFILE_NAME>
$ aws rds describe-db-instances --db-instance-identifier <DB_INSTANCE_ID> --profile <PROFILE_NAME>
```


