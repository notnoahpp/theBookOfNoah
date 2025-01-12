# serverless application model (SAM)

- AWS framework that simplifies creation and deployment of serverless applications
- an extension of AWS CloudFormation + a CLI for testing & deployments

## mythoughts

- if you're using AWS, its worth the investment in learning
- if you know terraform, alot of best practices will transfer over
  - the thing is, if you're good at terraform , why use SAM?
    - maybe because of SAM local, but if you're also good at localstack...

## links

- [deploy](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html)
- [deploying serverless applications](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-deploying.html)
- [policy templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html)
- [github with policy templates and other stuff](https://github.com/aws/serverless-application-model/tree/develop)
- [template anatomy](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-template-anatomy.html)

### sam CLI

- [build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html)

## best practices

- shorthand syntax express functions, apis, databases and event source mappings using YAML
  - instructions to build an environment and during deployment SAM transforms and expands it into AWS CloudFormation templates
  - all cloud formation optiosn are available
- start with prebuilt SAM policies to bootstrap commonly used templates to build for least privilege security access
- SAM supports swagger/openAPI for defining API gateway apis

### anti patterns

## features

- single framework to define all components required to test and deploy serverless applications
- help manage IAM policies: scopes the permissions of lambda fns to the resources used

## terms

- SAM template: streamlined CloudFormation template for serverless applications
- SAM CLI: cli tool for testing, debugging and deploying serverless applications

## basics

## considerations

### SAM CLI

- testing:
  - launches a docker container to test & debug lambda functions
  - FYI: only covers a subset of tests required for production launch
    - invoke fns and run automated tests locally
    - generate sample event source payloads
    - run API gateway locally
    - debug code
    - review lambda fn logs
    - validate AWS SAM templates
- common cli commands
  - init: initialize a serverless application
  - local: runs your application locally
  - validate: validates an AWS SAM template
  - deploy: deploy an AWS SAM application
    - use the `--guided` param to have an interactived deployment
    - requires an S3 bucker for the lambda deployment package
      - SAM CLI will create & manage this for you
  - build: a serverless application and prepare it for subsequent steps in the workflow
    - processes the AWS SAM template file, application code an any other files and deps
    - copies build artifacts in the format & expected locations for subsequent steps

### CI / CD pipeline

- CodeBuild: automate the process of packaging code & running tests before code is deployed
- CodeDeploy: use version control options to ensure sfae deployments to production
- test account: where you can deploy and test before deploying to a prod account
- prod account: the for production

## examples

### service folder hierarchy

- root
  - infrastructure: swagger.yaml, sam cloudformation template(s), etc
  - blah-service
    - deploy.sh script for deploying with sam
    - package.sh script for packaging with sam
    - rest of your normal app stuff

### sam cli

```sh
sam
  # generate a sample s3 put event
  # put it in launch.json.lambda.payload.json.{ ...copypasta}
  local generate-event s3 put

  # validate the template.yaml in the curdir
  validate

  # creates a zip archive ready for uploading to s3 and deployed to lambda
  # sets the code uri from local system path to the s3 path in the template definition
  package
    --template-file some-service.input.yaml
    --output-template-file some-service.output.yaml
    --s3-bucket some-bucket-to-deploy-to

```

### sam templates

- usually called `template.yaml` and defines the entire stack for a specific service
  - you generally want to start with a sample template provided by AWS (check the docs)
- resources.someName.type
  - AWS::S3::Bucket (s3)
  - AWS::Serverless::Function (lambda)
  - AWS::Serverless::SimpleTable (dynamodb)

```yaml
# functions
!GetAtt SomeThing.someProp # pull a value from another resource
!Ref SomeThing # ref to a resource

# sample of hella options
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2021-07-11 # indicates SAM and not merely cloudformation
Description: Fear me CopyPasta Service

Parameters:
  ServiceName:
    Description: Name of Service
    Type: String
    Default: FearMeCopyPastaService

Resources:
  # could be anything, this is an arbitray name for a lambda fn
  GetHtmlFunction:
    # this creates a lambda function
    Type: AWS::Serverless:Function
    Properties:
      # zipfile, handler, and runtime
      CodeUri: ../../src/todo_list # this gets built and deployed
      Handler: index.gethtml
      MemorySize: 512
      Timeout: 30
      Runtime: nodejs14.x
      AutoPublishAlias: live # detect new deployments and publish updated versions and aliases

      DeploymentPReference:
        Type: Canary10Percent10Minutes # implement canary traffic shifting
        Alarms: # specify auto rollbacks of deployments based on cloudwatch alarms
          - !Ref SomeCloudWatchAlarm
          - !Ref SomeClouodWatcAlarm
        Hooks: # run pre and post traffic shifting lambda fns
          PreTraffic: !Ref SomeLambdaFn
          PostTraffic: !Ref SomeLambdaFn
      # IAM policy
      Policies: AmazonDynamoDBReadOnlyAccess

      # environment configuration
      Evironment:
        Variables:
          NODE_CONFIG_DIR: './config'

      # lam da event sources
      Events:
        S3Event: # s3 event source for triggering lambda fns
          Type: S3
          Properties:
            Bucket:
              Ref: SomeBucketName
            Events: s3:ObjectCreated:* # any object created in SomeBucketName
        # create an API gateway endpoint
        # takes care of all necessary mapping/permissions
        GetHtml:
          Type: API
          Properties:
            Path: /(proxy+)
            Method: ANY
Outputs:
  ApiUrl:
    Description: URL of your API Endpoint
    value: !Join
      - ""
      - - https://abcdefg
```

## integrations

### lambda

- enables launching a local version of lambda fns that runs in a docker container
  - you can then connect to it and debug in an interactive session throught an IDE
