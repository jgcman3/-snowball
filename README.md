# Table of Contents

- [Table of Contents](#table-of-contents)
- [Welcome to the Snowball!](#welcome-to-the-snowball)
- [Goal](#goal)
- [Concept](#concept)
- [Work Flow](#work-flow)
  - [1. Configure](#1-configure)
  - [2. Initialize](#2-initialize)
  - [3. Make the lambda with the rules](#3-make-the-lambda-with-the-rules)
  - [4. Deploy](#4-deploy)
  - [5. Test](#5-test)
  - [6. Destroy](#6-destroy)
  - [7. Info](#7-info)
  - [8. Release](#8-release)

<br>

# Welcome to the Snowball!

This is the lambda repository of Snowball.<br>
This is a tool for easily developing lambdas on your local pc.<br>
You need to develop a lambda, check its behavior locally and deploy it to the cloud server.<br>
You can easily install the package, deploy lambda to the cloud server and delete it using script using aws-cli. <br>
Let's enjoy cloud world ;-)

<br>

# Goal

- <b>Initialize automatically</b>
  - Install tools for development
  - Check if AWS configuration is correct
- <b>Deploy automatically</b>
  - Install packages
  - Zip packages
  - Zip function code
  - Check If the function size ([<3MB](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)) is available
  - Check If the lambda size ([<250MB](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)) is available
  - Deploy the function and its layer to AWS
- <b>Destroy automatically</b>
  - Destroy the function and its layer from AWS

# Concept

![concepts](https://github.com/jgcman3/easylamda/blob/images/easylambda-concepts.png?raw=true)

<br>

# Work Flow

<br>

## 1. Configure

Firstly you should review and modify the aws account congifure of the project. <br>
See the configure folder. The configure folder path is <b>`./utility`</b>.
There is a configure file like following, <br>

- <b>`configure.json`</b> : This is for the aws account configure of the project.

The below json is a sample configure format. <br>
The name of profile must match with your profile and region of aws credentials(~/.aws/config or ~/.aws/credentials)

```json
{
  "profile": "aws-snowball-dev",
  "region": "ap-northeast-2"
}
```

<br>

## 2. Initialize

Please run <b> `./utilty/init.sh` </b>script.

- <b>`init.sh`</b> : The script to initialize the project

It initialize the environment by setting the aws profile of the project. <br>
The name of the aws profile that needs to be set is the value set in the project configure file.<br>
And It install the necessary packages. The installed packages are as follows.<br>

- <b>packages:</b> jq, zip, pip, aws-cli
  - jq is a lightweight and flexible command-line JSON processor
  - zip is a command-line to compresse one or more files or deirectories
  - pip is the package installer for Python
  - aws-cli is AWS command-line interface

<br>

## 3. Make the lambda with the rules

There are rules to follow in order to be easily distributed and deleted in the cloud by script.

The rules are as follows.

   * <b>Rule 1 : </b> All function codes should be located in the <b>`./{lambda folder}/source`</b> folder.
   * <b>Rule 2 : </b> All packages should be located in the <b>`./{lambda folder}/layer/python`</b> folder.
   * <b>Rule 3 : </b> <b>`function.json`</b> file should be in the root of your lambda folder.
   * <b>Rule 4 : </b> <b>`layer-internal.json`</b> file should be in the root of your lambda folder.
   * <b>Rule 5 : </b> <b>`layer-external.json`</b> file should be in the root of your lambda folder.
   * <b>Rule 6 : </b> If you don't want to use internal layer, you have to delete all package names in <b>./{lambda folder}/source/requirement.txt</b> file.

<b>`function.json`</b> file is used to set environment values for the configuration of lambda function.

See <b>`$ aws lambda create-function help`</b> for detailed configuration information.

The json file format is as follows.

```json
{
    "FunctionName": "snowball-dev-lambda-function-abcdef",
    "Runtime": "python3.8",
    "Role": "arn:aws:iam::123456789112:role/lambda-ex",
    "Handler": "app.lambda_handler",
    "Description": "It is just snowball development code deployed by the script.",
    "Timeout": 10,
    "MemorySize": 128,
    "Publish": true,
    "VpcConfig": {
      "SubnetIds": ["subnet-03f05510c6449edc4", "subnet-0404dcdf34fb2b62b"],
      "SecurityGroupIds": ["sg-0cbf8349516c57f44"]
    },
    "Environment": {
        "Variables": {
            "KeyName": ""
        }
    },
    "KMSKeyArn": "",
    "TracingConfig": {
        "Mode": "Active"
    },
    "Tags": {
      "project": "snowball",
      "environment": "dev",
      "name": "abcdef",
      "application": "lambda",
      "feature": "function",
      "cost-center": "a team",
      "owner": "jgcman3"
    }
}
```

<b>`layer-internal.json`</b> file is used to set environment values for the configuration of internal lambda layer.<br>
See <b>`$ aws lambda publish-layer-version help`</b> for detailed configuration information.

The json file format is as follows.

```json
{
    "LayerName": "snowball-dev-lambda-layer-abcdef",
    "Description": "It is just snowball development code deployed by the script.",
    "CompatibleRuntimes": [
        "python3.8"
    ],
    "LicenseInfo": ""
}
```

<b>`layer-external.json`</b> file is used to set environment values for the configuration of external lambda layer.<br>

The json file format is as follows.

```json
{
  "Layers": [
    {
      "Name": "snowball-dev-lambda-layer-basic-util"
    }
  ]
}
```

<br>

## 4. Deploy

If you want to deploy the function with common packages,
Please run the <b> `./utilty/run.sh -f {The path of function folder}` </b>script. <br>
The script automatically compresses functions and layer, And than deploys them to the cloud server.<br>
The script refers to the <b>`layer.json`</b> and <b>`function.json`</b> files in the functions folder.<br>

If you want to deploy the specific custom layer,
Please run the <b> `./utilty/run.sh -l {The path of layer folder}` </b>script. <br>
The script refers to the <b>`layer.json`</b> file in the layers folder. <br>

```
usage: ./utility/run.sh [-options]
# Deployment
  -f : deploy the path of lambda function including internal layer to aws
  -l : deploy the path of external layer to aws
```
<br>

## 5. Test

If you want to test the function remotely,<br>
Please run the <b> `./utilty/run.sh -t {The path of function folder}` </b>script.<br>
The script refers to the <b>`events/event.json`</b> file in the function folder. <br>
The script automatically test functions from the cloud server.

```
usage: ./utility/run.sh [-options]
# Test
  -t : test the specific function in aws with pre-defined event
```
<br>

## 6. Destroy

If you want to destroy the layer and function from cloud server,<br>
Please run the <b> `./utilty/run.sh -df {The path of function folder}` </b>script.<br>
The script automatically delete functions and layer from the cloud server.

If you want to destroy the specific custom layer from cloud server,<br>
Please run the <b> `./utilty/run.sh -dl {The path of custom layer folder}` </b>script.<br>
The script automatically delete the layer from the cloud server.

```
usage: ./utility/run.sh [-options]
# Destroyment
  -df : delete the specific function deployed in aws
  -dl : delete the specific layer deployed in aws
```
<br>

## 7. Info

If you want to get(show) the information of specific function from aws,<br>
Please run the <b> `./utilty/run.sh -sf {The path of function folder}` </b>script.<br>

If you want to get(show) the information of specific layer from aws,<br>
Please run the <b> `./utilty/list.sh -l {The path of the custom layer}` </b>script.<br>

```
usage: ./utility/run.sh [-options]
# Information
  -sf : show the specific function information deployed in aws
  -sl : show the specific layer information deployed in aws
  -saf : show all of functions deployed in aws
```
<br>

## 8. Release

If you want to release all resources of lambda to cdk,<br>
Please run the <b> `./utilty/run.sh -release` </b>script.<br>
It packages all resources and generates <b> `./out/release_{project name}_lambda_{git tag info}.zip`</b> file.<br>
The project owner is responsible for providing the file to the distributor. <br>
Check if the git information is properly released in <b>`release.txt`</b> file.

```
usage: ./utility/run.sh [-options]
# Release
  -release : release all resources
```
