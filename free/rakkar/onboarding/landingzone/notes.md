# Note for Onboarding AWS LandingZone in Rakkar Digital

## Organization

Use Terraform to create Org in AWS

> Note: Don't use Terraform to create OUs, while It's hard to register to Control Tower later on

## Tool: Control tower

Recommended to use Control Tower 

Config of Control Tower for Rakkar

### Architecture

Root: devops@rakkardigital.com
  - OU:
    - AFT: 
      + aws-aft@rakkardigital.com
    - Infrastructure: 
      + infras@rakkardigital.com
      + aws-infras@rakkardigital.com
    - Security: 
      + security@rakkardigital.com
      + aws-security@rakkardigital.com
    - Workload: 
      + aws-workload-nonprod@rakkardigital.com
      + aws-workload-prod@rakkardigital.com
    - Sandbox (NonProd): 
      + aws-sandbox@rakkardigital.com

### Setup Landing Zone with Control Tower

- Home Region: Singapore - ap-southeast-1
- Region Deny Setting: Not enabled (can change later)
- Additional Regions for governance: empty (can add regions later)
- Foundation OU: Security
- New OU: Sandbox
- Management Account: devops@rakkardigital.com
- Log Archive account: infras@rakkardigital.com
- Audit: security@rakkardigital.com
- CloudTrail configuration: Enabled
- Log configuration for Amazon S3:
  - S3 bucket retention for logging: 30 days
  - S3 bucket retention for access logging: 7 days (a lot of logs will be stored here, should consider to store in different type of S3 to save cost)
  
> Creation time can take more than 60 minutes

## Create new OU

- Use Control Tower to create new OU through AWS Console/CLI in Root Account (with User that has AdministratorAccessRole)
- Don't create OU by using other, like Terraform. (Need to check more, while the OU created by Control Tower still can't register in it)
- Maybe because of the limits that prevent creating new OU & Account in Control Tower

```
Registration failed for Workload.
Check the external resources that apply to Workload and its member accounts. Choose Register OU again after the external resources are repaired.
```

```
Registration failed for Workload
AWS Control Tower detects issues that prevent registering this organizational unit. Try again, or contact AWS Support.
```

## Onboarding Account Factory with Terraform

AWS Control Tower Account Factory with Terraform

Manage AWS Accounts Using Control Tower Account Factory for Terraform

[Guide](https://learn.hashicorp.com/tutorials/terraform/aws-control-tower-aft)

### Prerequisites

- Terraform v0.15+
- AWS Account: non-root user with AdministratorAccess
- AWS Control Tower enabled
- AWS CLI
- GitHub Account

### Follow the guide to learn & understand how to create new Accounts by using AFT with Terraform

> Error: Account created by AFT with terraform can get below error, trying to figure out how to fix it

```
Account enrollment failed.
AWS Control Tower could not enroll your account for the following reason: AWS Control Tower requires that you configure an IAM Identity Center directory as an identity source for your management account.
```

## Decommission LandingZone & Recreate it

- Take up to 2 hours to be completed
- Got error: AWS Control Tower failed to set up your landing zone completely: User: arn:aws:sts::735211621211:assumed-role/AWSControlTowerAdmin/AssumeAdminRole is not authorized to perform: organizations:DescribeAccount on resource: arn:aws:organizations::735211621211:account/o-r9vzszu6xw/735211621211 because no identity-based policy allows the organizations:DescribeAccount action 
- Need to grant AdministratorAccess:
  + describe Organization units
  + describe StackSets
  + Delete S3 bucket: aws-controltower-s3-access-logs-273337921838-ap-southeast-1 & aws-controltower-logs-273337921838-ap-southeast-1
    - Access Log Account & delete this S3 bucket
- Maybe just need to attach these: 
  + AWSControlTowerServiceRolePolicy
  + AWSControlTowerStackSetRolePolicy
  + AWSControlTowerCloudTrailRolePolicy
  + AWSControlTowerAdminPolicy
- For some reasons, they had been removed out from this role
- AWS Control Tower failed to deploy stack(s): arn:aws:cloudformation:ap-southeast-1:735211621211:stack/AWSControlTowerBP-BASELINE-CLOUDTRAIL-MASTER/abb6fee0-3d78-11ed-ad56-0a02aecd712c.To continue, review the failed stack(s) and try again
  + Resource handler returned message: "Resource of type 'AWS::Logs::LogGroup' with identifier '{"/properties/LogGroupName":"aws-controltower/CloudTrailLogs"}' already exists." (RequestToken: c6b01994-7c98-0fc9-1571-c7fe55cd7595, HandlerErrorCode: AlreadyExists)
  + Resource handler returned message: "Resource of type 'AWS::Logs::LogGroup' with identifier '{"/properties/LogGroupName":"aws-controltower/CloudTrailLogs"}' already exists." (RequestToken: c6b01994-7c98-0fc9-1571-c7fe55cd7595, HandlerErrorCode: AlreadyExists)
  + Delete LogGroup: aws-controltower/CloudTrailLogs



## Control Tower: org structure before decommissioned

Root
r-86gy
AFT
ou-86gy-q0vs8apn
AFT
597823470715
  |  
aws-aft@rakkardigital.com
Created 2022/09/22
sandbox
443434829744
  |  
aws-sandbox@rakkardigital.com
Created 2022/09/26
Sandbox
ou-86gy-5gk9inmw
This resource is empty
Security
ou-86gy-6kz86fok
Audit
328372614571
  |  
security@rakkardigital.com
Created 2022/09/19
Log Archive
273337921838
  |  
infras@rakkardigital.com
Created 2022/09/19
Workload
ou-86gy-52afd22n
This resource is empty
rakkardigital
management account
735211621211
  |  
devops@rakkardigital.com

## OK! After decommission & setup again, all OU & accounts is registered! Enjoy!!