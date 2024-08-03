# -IAM-Best-Practices-and-Deep-Dive

## IAM Fundamentals
### Table of contents

- [Understanding IAM roles, users, and groups](#understanding-iam-roles-users-and-groups)
- [IAM permissions and policies](#iam-permissions-and-policies)
- Best practices for IAM structure

## 2. IAM Policy Deep Dive

- Policy syntax and elements (Version, Statement, Action, Resource, Condition)
- Policy evaluation logic
- Writing effective IAM policies (least privilege, separation of duties)
- Policy templates and examples
- Using IAM tags for access control
  
## 3. IAM Best Practices

IAM security best practices
Preventing privilege escalation
Auditing and monitoring IAM
IAM compliance (PCI DSS, HIPAA, SOC 2)
IAM troubleshooting
4. Advanced IAM Topics

IAM roles for EC2 instances
IAM roles for Lambda functions
Federated identity management
IAM access keys and security tokens
IAM password policy
5. IAM Use Cases

Practical examples of IAM implementation
Case studies of IAM in different environments (development, production, etc.)

## Understanding IAM roles, users, and groups

### Users
Users are identities that can interact with AWS and its APIs. They consist of a name and credentials and their AWS access type(s). It’s recommendable to use speaking, “friendly” names for your users.

The access type can be either programmatic (via access keys that can be used to make calls to the AWS API), via the AWS Management Console (via password), or both.

For each account, there is one root user, which is the owner of your account and has all privileges to manage, modify or delete all resources or even the account itself. Some configurations can only be set by the root user like changing the account name, updating payment information, or assigning the account to an organization.

💡 The best way to grant human users access to large AWS environments is to make use of federation. This allows you to make use of a dedicated service that's whole purpose is managing identities and therefore controlling access: an identity provider. A better way of directly creating IAM users in an account is to make use of AWS Identity Center. AWS Identity Center allows to create IAM users in a root account of an AWS organization, so permissions can be delegated through different accounts.

### Roles
Roles have some similarities to users: it’s an identity that is associated with permissions to determine which actions can be taken at AWS. However, roles are not associated with a single person but can be assumed by anyone or anything who needs it.

This includes AWS service principals that are used for example for Lambda functions or container agents at ECS.

### Groups
With increasing users of an AWS account, maintaining individual permissions can become tedious. It’s recommendable to cluster users into dedicated permissions groups based on their requirements for their daily work.

Let’s look at a simple example, a team that consists of four different roles: administrators, developers, quality assurance, and business analysts.

Each of the roles can be put into a group for which fitting permissions are assigned.

administrators: having enhanced IAM permissions to create and manage users while the company or team grows, shrinks, or adapts in its structure.

developers: permissions to create AWS resources to build and deploy applications on AWS.

quality assurance: permissions to access delivery pipelines, databases, and reports.

business analysts: permissions to access production data and usage reports to gather detailed insights about customer behavior.

Even if not noted in this example, users can also be part of multiple groups which will then gain the commutated permissions of all of the assigned groups.

## IAM permissions and policies

### Policies
Every request to AWS goes through an enforcement check to determine if the requesting principal is authenticated and authorized for the targeted action. The decision is based on the assigned policies either directly to the IAM user or the role that is currently assumed.

A policy is an object in AWS that determines allow or deny actions for services and resources. They are mostly stored as structured JSON documents and come in different types.

Each policy comes with one or several statements that define which actions are granted to which resource under what conditions.


```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowGetObject",
      "Action": "s3:GetObject",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::mydata/*"
    }
  ]
}
```
Example A: This policy allows the user or group that the policy is attached to retrieve objects from the mydata bucket. The Resource element specifies the Amazon Resource Name (ARN) of the bucket, and the /* at the end of the ARN allows access to all objects in the bucket.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyIPRange",
      "Action": "*",
      "Effect": "Deny",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": ["192.0.2.0/24", "203.0.113.0/24"]
        }
      }
    }
  ]
}
```
Example B: This IAM policy denies all actions (Action: "*") on all resources (Resource: "*") if the request does not originate from the specified IP ranges (192.0.2.0/24 and 203.0.113.0/24).

Let’s have a detailed look at each of the elements of our two example policies:

Version: specifying the version of the policy language you want to use. The latest one is 2012-10-17 and you shouldn’t use a previous version.

Statement: a container holding one or several items that define the permissions.

Effect: stating whether actions are granted (allow) or denied (deny). A deny statement always overwrites allow statements.

Principal: missing in our example, but for example used with resource-based policies to determine the account, user, or role for which the statement applies.

Action: a list of API actions for the target service that is either allowed or denied.

Resource: the resources for which the statement should allow or deny the given actions. This is not required for resource-based policies, as it’s implicitly applied to the resource to which the policy is attached.

Condition: specifying under which circumstances the statement should be applied. This is optional but can be used to further drill-down permissions.

Policies come with size restrictions and it’s a best practice to split your policies by the resources you’re granting access to.
