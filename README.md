## Lambda functions to report and cleanup EBS snapshots and AMIs

# Requirements

## Build

You will need docker engine and `zip` utility to build project. Also, build script uses `bash` shell

```
$ scripts/build.sh 
  Restoring packages for /project/Base2.Lambdas.csproj...
  Lock file has not changed. Skipping lock file write. Path: /project/obj/project.assets.json
  Restore completed in 2.06 sec for /project/Base2.Lambdas.csproj.
  
  NuGet Config files used:
      /root/.nuget/NuGet/NuGet.Config
  
  Feeds used:
      https://api.nuget.org/v3/index.json
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

  Base2.Lambdas -> /project/bin/Debug/netcoreapp1.0/Base2.Lambdas.dll
  adding: AWSSDK.AutoScaling.dll (deflated 70%)
  adding: AWSSDK.Core.dll (deflated 66%)
  adding: AWSSDK.EC2.dll (deflated 70%)
  adding: AWSSDK.S3.dll (deflated 63%)
  adding: Amazon.Lambda.Core.dll (deflated 57%)
  adding: Amazon.Lambda.Serialization.Json.dll (deflated 56%)
  adding: Base2.Lambdas.deps.json (deflated 74%)
  adding: Base2.Lambdas.dll (deflated 55%)
  adding: Base2.Lambdas.pdb (deflated 40%)
  adding: Newtonsoft.Json.dll (deflated 60%)
  adding: System.Collections.NonGeneric.dll (deflated 60%)
  adding: System.Runtime.Serialization.Primitives.dll (deflated 48%)

```

## Automated deployment

There is no automated deployment at this point of development - lambda hanlders in this project
are intended for manual execution.

## Lambda configuration 

### Code Package

`scripts/build.sh` script will create lambda package in root directory called `Base2.Lambdas.zip`. Upload
this package as lambda code

### Handler 

Use following entry points (Lambda function handlers)

- Report generation for EBS - `Base2.Lambdas::Base2.Lambdas.Handlers.EBSReportAndCleanup::UploadEBSReport`
- Report generation for AMI - `Base2.Lambdas::Base2.Lambdas.Handlers.AMIReportAndCleanup::UploadAMIReport`
- Cleanup from CSV info for EBS - `Base2.Lambdas::Base2.Lambdas.Handlers.EBSReportAndCleanup::CleanupFromReport`

### IAM Role

Iam role configured for lambda should have following policies 

- read only access to EC2 service
- write acces to S3 bucket passed in as argument
- DeleteSnapshot permissions

### Timeout

All of operations can be time consuming, so it's recommended to set all runtimes to 5 minutes

### Runtime

Use C# as runtime

### Memory

This functions do not require more than 128MB of memory, even when working with ~10k EBS snapshots (highest tested value)

### Other

There is no VPC configuration required

## Test / Run

Both report generation and cleanup tasks are accepting location of csv file to write/read 
in event parameters

e.g.
```
{
    "Bucket":"aws.amis-cleanup.reports.base2.services",
    "Key":"ebs_report_prod.csv"
}
```