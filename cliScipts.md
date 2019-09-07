aws cloudformation delete-stack --stack-name testLogsServerGen

aws cloudformation create-stack --stack-name testLogsServerGen --template-body file://./testLogsServerGen.yaml --parameters ParameterKey=Env,ParameterValue=np ParameterKey=KeyName,ParameterValue=david_wsl

aws cloudformation update-stack --stack-name testLogsServerGen --template-body file://./testLogsServerGen.yaml --parameters ParameterKey=Env,ParameterValue=np ParameterKey=KeyName,ParameterValue=david_wsl --capabilities CAPABILITY_IAM

sudo yum install -y aws-kinesis-agent

wget http://media.sundog-soft.com/AWSBigData/LogGenerator.zip

unzip LogGenerator.zip

chmod a+x LogGenerator.py

sudo mkdir /var/log/cadabra

cd /etc/aws-kinesis/

## manual

sudo nano agent.json

"firehose.endpoint": "firehose.ap-southeast-2.amazonaws.com"

attach an IAM role

# useful cli

aws cloudformation list-stacks --region ap-southeast-2 --output table --query 'StackSummaries[*].StackName'

aws cloudformation describe-stacks --region ap-southeast-2 --stack-name testLogsServerGen --query 'Stacks[*].RoleARN'
