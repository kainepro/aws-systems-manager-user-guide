# aws:executeAutomation – Run another automation<a name="automation-action-executeAutomation"></a>

Runs a secondary automation by calling a secondary runbook\. With this action, you can create runbooks for your most common operations, and reference those runbooks during an automation\. This action can simplify your runbooks by removing the need to duplicate steps across similar runbooks\.

The secondary automation runs in the context of the user who initiated the primary automation\. This means that the secondary automation uses the same IAM role or user account as the user who started the first automation\.

**Important**  
If you specify parameters in a secondary automation that use an assume role \(a role that uses the iam:passRole policy\), then the user or role that initiated the primary automation must have permission to pass the assume role specified in the secondary automation\. For more information about setting up an assume role for Automation, see [Method 2: Use IAM to configure roles for Automation](automation-permissions.md)\.

**Input**

------
#### [ YAML ]

```
name: Secondary_Automation
action: aws:executeAutomation
maxAttempts: 3
timeoutSeconds: 3600
onFailure: Abort
inputs:
  DocumentName: secondaryAutomation
  RuntimeParameters:
    instanceIds:
    - i-1234567890abcdef0
```

------
#### [ JSON ]

```
{
   "name":"Secondary_Automation",
   "action":"aws:executeAutomation",
   "maxAttempts":3,
   "timeoutSeconds":3600,
   "onFailure":"Abort",
   "inputs":{
      "DocumentName":"secondaryAutomation",
      "RuntimeParameters":{
         "instanceIds":[
            "i-1234567890abcdef0"
         ]
      }
   }
}
```

------

DocumentName  
The name of the secondary runbook to run during the step\. For runbooks in the same AWS account, specify the runbook name\. For runbooks shared from a different AWS account, specify the Amazon Resource Name \(ARN\) of the runbook\. For information about using shared runbooks, see [Using shared SSM documents](ssm-using-shared.md)\.  
Type: String  
Required: Yes

DocumentVersion  
The version of the secondary runbook to run\. If not specified, Automation runs the default runbook version\.  
Type: String  
Required: No

RuntimeParameters  
Required parameters for the secondary runbook\. The mapping uses the following format: \{"parameter1" : "value1", "parameter2" : "value2" \}  
Type: Map  
Required: NoOutput

Output  
The output generated by the secondary automation\. You can reference the output by using the following format: *Secondary\_Automation\_Step\_Name*\.Output  
Type: StringList  
Here is an example:  

```
- name: launchNewWindowsInstance
  action: 'aws:executeAutomation'
  onFailure: Abort
  inputs:
    DocumentName: launchWindowsInstance
  nextStep: getNewInstanceRootVolume
- name: getNewInstanceRootVolume
  action: 'aws:executeAwsApi'
  onFailure: Abort
  inputs:
    Service: ec2
    Api: DescribeVolumes
    Filters:
    - Name: attachment.device
      Values:
      - /dev/sda1
    - Name: attachment.instance-id
      Values:
      - '{{launchNewWindowsInstance.Output}}'
  outputs:
  - Name: rootVolumeId
    Selector: '$.Volumes[0].VolumeId'
    Type: String
  nextStep: snapshotRootVolume
- name: snapshotRootVolume
  action: 'aws:executeAutomation'
  onFailure: Abort
  inputs:
    DocumentName: AWS-CreateSnapshot
    RuntimeParameters:
    VolumeId:
    - '{{getNewInstanceRootVolume.rootVolumeId}}'
    Description:
    - 'Initial root snapshot for {{launchNewWindowsInstance.Output}}'
```

ExecutionId  
The ID of the secondary automation\.  
Type: String

Status  
The status of the secondary automation\.  
Type: String