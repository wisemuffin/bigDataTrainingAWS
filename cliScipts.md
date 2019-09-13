# creating/updating the stack testLogsServerGen from template

cd LogGen
aws cloudformation delete-stack --stack-name testLogsServerGen

aws cloudformation create-stack --stack-name testLogsServerGen --template-body file://./testLogsServerGen.yaml --parameters ParameterKey=Env,ParameterValue=np ParameterKey=KeyName,ParameterValue=david_wsl --capabilities CAPABILITY_IAM

# creating/updating the stack testKinesisFirehose from template

aws cloudformation delete-stack --stack-name testKinesisFirehose

aws cloudformation create-stack --stack-name testKinesisFirehose --template-body file://./testKinesisFirehose.yaml --parameters ParameterKey=Env,ParameterValue=np --capabilities CAPABILITY_IAM

# SAM testKinesisStreamToLambdaToDynamoDb - to DELETE

aws cloudformation delete-stack --stack-name test-kinesis-stream-lambda

sam package --template-file template.yaml --output-template-file sam.yaml --s3-bucket samdeploymentdg

aws cloudformation create-stack --stack-name test-kinesis-stream-lambda --template-body file://sam.yaml --parameters ParameterKey=Env,ParameterValue=np --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND

# SAM testKinesisStreamToLambdaToDynamoDb

aws cloudformation delete-stack --stack-name testKinesisStreamToLambdaToDynamoDb

sam package --template-file template.yaml --output-template-file sam.yaml --s3-bucket samdeploymentdg

aws cloudformation create-stack --stack-name testKinesisStreamToLambdaToDynamoDb --template-body file://sam.yaml --parameters ParameterKey=Env,ParameterValue=np --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND

# got ya

## got ya 1

in the ec2 instance is a agent.json file
agent.json has to ref the firehose name
however firehose name is dynamic based on the stack name
so need to ssh in and change it

cant use cloudformation export output on a file?

manual workaround:
cd /etc/aws-kinesis/
nano agent.json

# once up and running shedule 500k transactions to be sent to firehose

cd /home/ec2-user
sudo ./LogGenerator.py 500000

then check logs being created at
cd /var/log/cadabra
ls
tail -f /var/log/aws-kinesis-agent/aws-kinesis-agent.log

# useful cli

aws cloudformation list-stacks --region ap-southeast-2 --output table --query 'StackSummaries[*].StackName'

aws cloudformation describe-stacks --region ap-southeast-2 --stack-name testLogsServerGen --query 'Stacks[*].RoleARN'

# set up kinesis agent script

## pre requisite add aws-kinesis agent code to s3 bucket

aws s3 mb s3://testkinesisagentdg
aws s3 cp agent.json s3://testkinesisagentdg

## rest of code - all in UserData for ec2 bootstrapping

cd ~
sudo yum install -y aws-kinesis-agent

wget http://media.sundog-soft.com/AWSBigData/LogGenerator.zip

unzip LogGenerator.zip

chmod a+x LogGenerator.py

sudo mkdir /var/log/cadabra

cd /etc/aws-kinesis/

sudo rm agent.json
sudo aws s3 cp s3://testkinesisagentdg/agent.json agent.json

sudo service aws-kinesis-agent start
sudo chkconfig aws-kinesis-agent on

## kick off 500k records push to s3

cd ~
sudo ./LogGenerator.py 500000
