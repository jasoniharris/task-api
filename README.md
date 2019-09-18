# Install Steps
# Get Lambda ARN
aws cloudformation describe-stacks --stack-name apigateway --query Stacks[0].Outputs --profile jh-developer

# Update swagger def with Lambda ARN
sed -i '.bak' 's/arn:aws:lambda:us-east-1:YYY:function:apigateway-Lambda-XXX/arn:aws:lambda:us-east-1:1234:function:apigateway-Lambda-1234/g' swagger.json

sed -i '.bak' 's/$AWSRegion/us-east-1/g' swagger.json

# Import swagger def to APIGW
aws apigateway import-rest-api --fail-on-warnings --body file://swagger.json --profile jh-developer

# Update stack with the API key
aws cloudformation update-stack --stack-name apigateway --template-body file://template.json --capabilities CAPABILITY_IAM --parameters ParameterKey=S3Bucket,UsePreviousValue=true ParameterKey=S3Key,UsePreviousValue=true ParameterKey=ApiId,ParameterValue=5678 --profile jh-developer

# Deploy APIGW Stage
aws apigateway create-deployment --rest-api-id 5678 --stage-name v1 --profile jh-developer
