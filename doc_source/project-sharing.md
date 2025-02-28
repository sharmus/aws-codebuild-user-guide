# Working with Shared Projects<a name="project-sharing"></a>

Project sharing allows project owners to share their CodeBuild projects with other AWS accounts or users\. In this model, the account that owns the project \(owner\) shares a project with other accounts \(consumers\)\. A consumer cannot edit or run a project\.

**Topics**
+ [Prerequisites for Sharing Projects](#project-sharing-prereqs)
+ [Prerequisites for Accessing Shared Projects Shared with You](#project-sharing-access-prereqs)
+ [Related Services](#project-sharing-related)
+ [Sharing a Project](#project-sharing-share)
+ [Unsharing a Shared Project](#project-sharing-unshare)
+ [Identifying a Shared Project](#project-sharing-identify)
+ [Shared Project Permissions](#project-sharing-perms)

## Prerequisites for Sharing Projects<a name="project-sharing-prereqs"></a>

 To share a project, your AWS account must own it\. You cannot share a project that has been shared with you\. 

## Prerequisites for Accessing Shared Projects Shared with You<a name="project-sharing-access-prereqs"></a>

To access a shared report group, a consumer's IAM role requires the `BatchGetProjects` permission\. You can attach the following policy to their IAM role: 

```
{
    "Effect": "Allow",
    "Resource": [
        "*"
    ],
    "Action": [
        "codebuild:BatchGetProjects"
    ]
}
```

 For more information, see [Using Identity\-Based Policies for CodeBuild](auth-and-access-control-iam-identity-based-access-control.md)\. 

## Related Services<a name="project-sharing-related"></a>

Project sharing integrates with AWS Resource Access Manager \(AWS RAM\), a service that makes it possible for you to share your AWS resources with any AWS account or through AWS Organizations\. With AWS RAM, you share resources by creating a *resource share* that specifies the resources and the consumers to share them with\. Consumers can be individual AWS accounts, organizational units in AWS Organizations, or an entire organization in AWS Organizations\.

For more information, see the *[AWS RAM User Guide](https://docs.aws.amazon.com/ram/latest/userguide/)*\.

## Sharing a Project<a name="project-sharing-share"></a>

The consumer can use the AWS CLI but not the CodeBuild console to view the project and builds you've shared\. The consumer cannot edit or run the project\.

You can add a project to an existing resource share or you can create one in the [AWS RAM console](https://console.aws.amazon.com/ram)\.

**Note**  
 You cannot delete a project with builds that has been added to a resource share\. 

To share a project with organizational units or an entire organization, you must enable sharing with AWS Organizations\. For more information, see [Enable Sharing with AWS Organizations](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html) in the *AWS RAM User Guide*\.

You can use the CodeBuild console, AWS RAM console, or the AWS CLI to share a project that you own\.

**To share a project that you own \(CodeBuild console\)**

1. Open the AWS CodeBuild console at [https://console\.aws\.amazon\.com/codesuite/codebuild/home](https://console.aws.amazon.com/codesuite/codebuild/home)\.

1. In the navigation pane, choose **Build projects**\.
**Note**  
By default, only the 10 most recent build projects are displayed\. To view more build projects, choose the gear icon, and then choose a different value for **Projects per page** or use the back and forward arrows\.

1.  Choose the project you want to share, and then choose **Share**\. For more information, see [Create a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html#getting-started-sharing-create) in the *AWS RAM User Guide*\. 

**To share a project that you own \(AWS RAM console\)**  
See [Creating a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing.html#working-with-sharing-create) in the *AWS RAM User Guide*\.

**To share a project that you own \(AWS RAM command\)**  
Use the [create\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/create-resource-share.html) command\.

 **To share a project that you own \(CodeBuild command\)** 

Use the [put\-resource\-policy](https://docs.aws.amazon.com/cli/latest/reference/codebuild/put-resource-policy.html) command:

1.  Create a file named `policy.json` and copy the following into it\. 

   ```
   {
      "Version":"2012-10-17",
      "Statement":[{
        "Effect":"Allow",
        "Principal":{
          "AWS":"consumer-aws-account-id-or-user"
        },
        "Action":[
          "codebuild:BatchGetProjects",
          "codebuild:BatchGetBuilds",
          "codebuild:ListBuildsForProject"],
        "Resource":"arn-of-project-to-share"
      }]
    }
   ```

1. Update `policy.json` with the project ARN and identifiers to share it with\. The following example grants read\-only access to the project with the ARN `arn:aws:codebuild:us-west-2:123456789012:project/my-project` to Alice and the root user for the AWS account identified by 123456789012\. 

   ```
   {
      "Version":"2012-10-17",
      "Statement":[{
        "Effect":"Allow",
        "Principal":{
          "AWS": [
             "arn:aws:iam::123456789012:user/Alice",
             "123456789012"
           ]
        },
        "Action":[
          "codebuild:BatchGetProjects",
          "codebuild:BatchGetBuilds",
          "codebuild:ListBuildsForProject"],
        "Resource":"arn:aws:codebuild:us-west-2:123456789012:project/my-project"
      }]
    }
   ```

1.  Run the following command\. 

   ```
   aws codebuild put-resource-policy --resource-arn project-arn --policy file://policy.json
   ```

## Unsharing a Shared Project<a name="project-sharing-unshare"></a>

An unshared project, including its builds, can be accessed only by its owner\. If you unshare a project, any AWS account or user you previously shared it with cannot access the project or its builds\.

To unshare a shared project that you own, you must remove it from the resource share\. You can use the CodeBuild console, AWS RAM console, or AWS CLI to do this\.

**To unshare a shared project that you own \(AWS RAM console\)**  
See [Updating a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing.html#working-with-sharing-update) in the *AWS RAM User Guide*\.

**To unshare a shared project that you own \(AWS CLI\)**  
Use the [disassociate\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/disassociate-resource-share.html) command\.

 ** To unshare project that you own \(CodeBuild command\)** 

Run the [delete\-resource\-policy](https://docs.aws.amazon.com/cli/latest/reference/codebuild/delete-resource-policy.html) command and specify the ARN of the project you want to unshare:

```
aws codebuild delete-resource-policy --resource-arn project-arn
```

## Identifying a Shared Project<a name="project-sharing-identify"></a>

Owners and consumers can use the AWS CLI to identify shared projects\.

**To identify projects shared with your AWS account or user \(AWS CLI\)**  
Use the [list\-shared\-project](https://docs.aws.amazon.com/cli/latest/reference/ram/list-shared-project.html) command to return the projects that are shared with you\.

## Shared Project Permissions<a name="project-sharing-perms"></a>

### Permissions for Owners<a name="project-perms-owner"></a>

A project owner can edit the project and use it to run builds\.

### Permissions for Consumers<a name="project-perms-consumer"></a>

A project consumer can view a project and its builds, but cannot edit a project or use it to run builds\.