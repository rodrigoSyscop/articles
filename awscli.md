# AWS Command Line Interface

The awscli is a unified tool to manage your AWS services.
With just one tool you can control multiple AWS services from the command line and automate them through scripts.

## Requirements

You must have a AWS account and an access key in order to make programmatic requests from awscli to your aws account. See [how to get your access key ID and secret access key?](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html?shortFooter=true#id_users_create_console) if you don't have them yet.

## Installing

If you are on macOS, the easiest way to get awscli running is using [homebrew](https://brew.sh):

```bash
$ brew install awscli
```

Or just use pip, but I strongly recommend that you run pip from a virtual env, _do not mess up your base system!_

```bash
$ pip install awscli
```

## Configuring

Just type `aws configure` in your shell and fill the questions with your aws access key, secret access key and a default region name:

```bash
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: ENTER

Optionally you can set those vars as environment vars, some third party might try to access these env vars:

```bash
$ export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
$ export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
$ export AWS_DEFAULT_REGION=us-west-2
```


## Testing 

Lets have some fun:

```bash
$ aws ec2 describe-instances --output table --region us-west-2
------------------------------------------------------------------------------
|                              DescribeInstances                             |
+----------------------------------------------------------------------------+
||                               Reservations                               ||
|+-------------------------------------+------------------------------------+|
||  OwnerId                            |  012345678901                      ||
||  ReservationId                      |  r-abcdefgh                        ||
|+-------------------------------------+------------------------------------+|
|||                                Instances                               |||
||+------------------------+-----------------------------------------------+||
|||  AmiLaunchIndex        |  0                                            |||
|||  Architecture          |  x86_64                                       |||
...
```

The `table` output format is nice when you are working on a terminal. However when working from scripts, you may want to use the `json` output format.

```bash
$ aws ec2 describe-regions --output json
{
    "Regions": [
        {
            "Endpoint": "ec2.ap-south-1.amazonaws.com", 
            "RegionName": "ap-south-1"
        }, 
        {
            "Endpoint": "ec2.eu-west-2.amazonaws.com", 
            "RegionName": "eu-west-2"
        }, 
        {
            "Endpoint": "ec2.eu-west-1.amazonaws.com", 
            "RegionName": "eu-west-1"
        }, 
        {
            "Endpoint": "ec2.ap-northeast-2.amazonaws.com", 
            "RegionName": "ap-northeast-2"
        }, 
        {
            "Endpoint": "ec2.ap-northeast-1.amazonaws.com", 
            "RegionName": "ap-northeast-1"
        }, 
        {
            "Endpoint": "ec2.sa-east-1.amazonaws.com", 
            "RegionName": "sa-east-1"
        }, 
        {
            "Endpoint": "ec2.ca-central-1.amazonaws.com", 
            "RegionName": "ca-central-1"
        }, 
        {
            "Endpoint": "ec2.ap-southeast-1.amazonaws.com", 
            "RegionName": "ap-southeast-1"
        }, 
        {
            "Endpoint": "ec2.ap-southeast-2.amazonaws.com", 
            "RegionName": "ap-southeast-2"
        }, 
        {
            "Endpoint": "ec2.eu-central-1.amazonaws.com", 
            "RegionName": "eu-central-1"
        }, 
        {
            "Endpoint": "ec2.us-east-1.amazonaws.com", 
            "RegionName": "us-east-1"
        }, 
        {
            "Endpoint": "ec2.us-east-2.amazonaws.com", 
            "RegionName": "us-east-2"
        }, 
        {
            "Endpoint": "ec2.us-west-1.amazonaws.com", 
            "RegionName": "us-west-1"
        }, 
        {
            "Endpoint": "ec2.us-west-2.amazonaws.com", 
            "RegionName": "us-west-2"
        }
    ]
}
```

There is a whole [world of commands to learn](http://docs.aws.amazon.com/cli/latest/userguide/tutorial-ec2-ubuntu.html), like these ones:

```bash
# creating a new SecurityGroup and authorizing ssh ingress 
$ aws ec2 create-security-group --group-name devenv-sg --description "security group for development environment in EC2"
$ aws ec2 authorize-security-group-ingress --group-name devenv-sg --protocol tcp --port 22 --cidr 0.0.0.0/0

# NOTE: for old aws accounts that still support ec2-classic instances and thus
# do not have a default VPC, YOU MUST create a new VPC first!
# Then you will need to use the vpc-id or subnet-id in your commands.
aws ec2 create-security-group --group-name devenv-sg --description "security group for development environment in EC2" --vpc-id vpc-64a4f002
{
    "GroupId": "sg-95c0b9ea"
}
aws ec2 authorize-security-group-ingress --group-id sg-95c0b9ea --protocol tcp --port 22 --cidr 0.0.0.0/0 

# creating a new key pair
$ aws ec2 create-key-pair --key-name devenv-key --query 'KeyMaterial' --output text > devenv-key.pem

# protect your new key
$ chmod 400 devenv-key.pem

# launch a new instance (it will outputs the instance ID after a few seconds)
$ aws ec2 run-instances --image-id ami-6d1c2007 --security-group-ids sg-b018ced5 --count 1 --instance-type t2.micro --key-name devenv-key --query 'Instances[0].InstanceId'
"i-ec3e1e2k"

# FOR OLD AWS ACCOUNTS you have to inform the --subnet-id param
aws ec2 run-instances --image-id ami-6d1c2007 --subnet-id subnet-766a1e13 --security-group-ids sg-95c0b9ea --count 1 --instance-type t2.micro --key-name devenv-key --query 'Instances[0].InstanceId'
"i-09c4a6ec87d807bf4"

# wait a minute or two to get your new instance created
# then you can query for its public ip address
$ aws ec2 describe-instances --instance-ids i-09c4a6ec87d807bf4 --query 'Reservations[0].Instances[0].PublicIpAddress'
"54.183.22.255"
# and finally connect to it
$ ssh -i devenv-key.pem centos@54.183.22.255
```

Remember you will incur in costs. To not pay too much lets terminate our instance:

```bash
$ aws ec2 terminate-instances --instance-ids i-09c4a6ec87d807bf4
```



