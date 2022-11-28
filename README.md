This is a sample laravel app. 
The cloudformation template that provisions the environment is cf-template.json .
cf-template.json contains all the necessary info to create the the environment, but due to cloudformation limitations
it doesn't create the docker image that will be used in ECS, so no ECS tasks will be created initially after creating the stack.
After creating the stack, you can compile this code into a docker image using the provided Dcokerfile:
```
docker image build -t example-app:latest .
```
You can then tag this with the url of the ECR repository and push it, to be deployed later to the ECS service:
```
accountId=$( aws sts get-caller-identity --output text | cut -f1 )
repositoryName=testenv2-main-repo ## it depends on the env name you provided in the template 
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin $accountId".dkr.ecr.eu-west-1.amazonaws.com"
docker tag example-app:latest "$accountId.dkr.ecr.eu-west-1.amazonaws.com/$repositoryName:latest"
docker push "$accountId.dkr.ecr.eu-west-1.amazonaws.com/$repositoryName:latest"
```
Stuff I liked to add but didn't have time to:
- Handle the deployment through SAM or Serverless
- Separate environment from the code, by adding the .env file variables as environment vars to the ECS task (this requires the creation of s3 bucket to fetch the .env file from)  
- add an option to create the Route53 hosted zone from scratch
- add an option to create dns verified ACM certificate
- add the option to create a new key pair to access the bastion instance