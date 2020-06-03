
AWS Cost optimization notes

The # section 1 #, show general recommendation to optimize costs on AWS.

The # section 2 #, show an implementation of scheduled environment with lambda function.

## Section 1 ##
For new project, select the latest instance architecture available on AWS, the new CPU instances type cost less than old ones.

A Sample: Paris Region (not the more cheap EU region)  (less is better)

T2-small - OnDemand x Month = $19.272000 monthly
T3-small - OnDemand x Month = $17.228000 monthly
T3a-small - OnDemand x Month = $15.476000 monthly >> the cheaper

As you can see, the T3a-xxx instances are the most cheaper and provide similar performance than T3-xxx.
The trick is that AMD power the CPU core of the T3a-xxx instances which reduce the cost compared with INTEL CPU core used on T2-xx, T3-xxx and so on.


For medium/Small project. could be between 20-30 instances, the facto option is use OnDemand capacity to run short project with a lifecycle of a few months.
For long-term project (+1 year or more) the facto option is to use RI (Reserved Instances) more appropriate for medium/large DB instances.

In addition one cheap and not disruptive option (meaning without replace your instance) is to implement a schedule environment to start / stop instances every day,outside of business hours.
This solution can result in up to 70% cost savings for  instances (OnDemand) that are only necessary during regular business hours. 
(with weekly schedule, the utilization reduced from 168 hours to 50 hours).

T2-small - OnDemand x Month (without schedule) cost = $19.272000 monthly

T2-small - OnDemand x Month (with weekly schedule) (4 * 50 = 200 hours approx) = $5,28 monthly (reduce >= 70%)

   - T2-small - OnDemand x hour cost = $0.026400 hourly
              200 hours * $0.026400 hourly = $5,28

Same thing, but using T3a-small

T3a-small - OnDemand x Month (without schedule) cost =  $15.476000 monthly  (Cost > - $3,88 than T2-small x month )

T3a-small - OnDemand x Month (with weekly schedule) (4 * 50 = 200 hours approx) = $4,24 monthly

   - T3a-small - OnDemand x hour cost = $0.021200 hourly
              200 hours * $0.021200 hourly =  $4,24


## Section 2 ##

This guide show you how to build and deploy lambda functions with serverless and CF.
You can start from scratch with an empty template or if you have already a function that you want to automate you can "export function" .
From lambda console select your lambda function and select Actions > "Export function". 
Download both files AWS SAM file and deployment package.

You will download 2 files:
- one with the lambda_name.yaml (SAM template).
- Second one a .zip with the lambda.py (function code).

In my case I created initially 2 lambda function to stop/start  EC2 instances, with a cloudwatch schedule to start at 9AM and stop at 6PM Mon-Fri.
These functions must be created on each region where they are needed.
The easy way to automate this is using serverless framework for AWS and CF (for home working, could be implemented this with TF instead of CFC ).

- start_ec2/lambda_function.py
- stop_ec2/lambda_function.py


Follow hese steps to automate the deploy a lambda function to any region on demand.

pre-requisite:
- Lambda fuction
- S3 bucket to usa as artifact repo.

1- First, you need to create the CF template using aws cli with CF package parameter.
   #Run CF package to create package file output.yaml
   #Pay attention that all the content of your current directory will by uploaded to the new lambda function.

   #Sample, you should put the lambda code in the same directory with the sam template.

    /home/start_ec2]$ ls -lrt
        total 6
        -rw-r--r-- 1 vittorio vittorio 706 26 mag 14.43 lambda_function.py
        -rw-r--r-- 1 vittorio vittorio 553  3 giu 16.11 EC2-lambda-start.yaml
        -rw-r--r-- 1 vittorio vittorio 608  3 giu 16.20 output.yaml

   $aws cloudformation package --template-file EC2-lambda-start.yaml --output-template-file output.yaml \
   --s3-bucket sam-deploy-lambda-serverless --profile <your-aws-profile>

Uploading to 83b4d36693d0972fc4830dda933bceab  1898 / 1898.0  (100.00%)
Successfully packaged artifacts and wrote output template to file output.yaml.
Execute the following command to deploy the packaged template

aws cloudformation deploy --template-file output.yaml --stack-name <YOUR STACK NAME>

2- Follow the instructions from the above output to deploy the template.
   This will create a new template on CF services with the name invoked on *--stack-name* and will deploy the lambda function.
   The lambda function name is defined in the output.yaml file in the "Resources" section, in my sample, I defined the name as "EC2lambdastart". 
   #Deploy the CF template, this will create the lambda function

   $aws cloudformation deploy --template-file output.yaml \
    --stack-name stack-sam --profile <your-aws-profile> --region <wanted-region>

    Waiting for changeset to be created..
    Waiting for stack create/update to complete
    Successfully created/updated stack - SAM-lambda-deploy

3- Go to Lambda Services on AWS and select the region that you invoked previously on step 2, and now, 
   you should see a new lambda function just created a seconds ago.

The name of the new lambda function will be "stack-name-function-name-slackID"

