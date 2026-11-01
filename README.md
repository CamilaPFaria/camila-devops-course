# CloudFormation Assignment

## Resources Created

### Part A: Application Infrastructure

| Logical ID | Resource Type | Explanation |
| :--- | :--- | :--- |
| **`WebAssetsBucket`** | `AWS::S3::Bucket` | S3 bucket created to store the website content (`index.html`). |
| **`EC2S3AccessRole`** | `AWS::IAM::Role` | Role assigned to the EC2 instance to access the webpage stored in the bucket. |
| **`WebsiteUploadLambdaRole`** | `AWS::IAM::Role` | Role assigned to the Lambda function resource to upload website content to the S3 bucket. |
| **`S3AccessPolicy`** | `AWS::IAM::Policy` | IAM policy defining what resources and actions the role can access. |
| **`S3PutObjectPolicy`** | `AWS::IAM::Policy` | Policy allowing objects to be uploaded to and deleted from the S3 bucket. Assigned to `WebsiteUploadLambdaRole`. |
| **`EC2RoleProfile`** | `AWS::IAM::InstanceProfile` | Instance profile needed because EC2 cannot accept an IAM role directly. Passes role credentials to the EC2 instance. |
| **`WebsiteUploadLambda`** | `AWS::Lambda::Function` | Lambda function that uploads website content (`index.html`) to the S3 bucket. |
| **`UploadIndexHtml`** | `Custom::S3Objects` | CloudFormation Custom Resource used to trigger Lambda execution during stack creation. |
| **`EC2SecurityGroup`** | `AWS::EC2::SecurityGroup` | Security group defining firewall rules to enable inbound HTTP traffic (port 80). |
| **`WebServerEC2`** | `AWS::EC2::Instance` | EC2 instance created be a web server and fetch contents from the bucket |

---

### Part B: VPC Networking Infrastructure

| Logical ID | Resource Type | Description |
| :--- | :--- | :--- |
| **`CA1VPC`** | `AWS::EC2::VPC` | Custom Virtual Private Cloud defining the `10.0.0.0/16` isolated IP network space with DNS hostnames enabled. |
| **`InternetGateway`** | `AWS::EC2::InternetGateway` | Gateway component providing direct connectivity between the VPC and the public internet. |
| **`VPCGatewayAttachment`** | `AWS::EC2::VPCGatewayAttachment` | Attaches the `InternetGateway` to the `CA1VPC` network container. |
| **`PublicSubnet`** | `AWS::EC2::Subnet` | Public subnet (`10.0.1.0/24`) configured with `MapPublicIpOnLaunch` to auto-assign public IP addresses. |
| **`PublicRouteTable`** | `AWS::EC2::RouteTable` | Custom routing table for controlling network traffic rules within the public subnet. |
| **`PublicRoute`** | `AWS::EC2::Route` | Route rule mapping all outbound internet-bound traffic (`0.0.0.0/0`) through the `InternetGateway`. |
| **`PublicSubnetRouteTableAssociation`** | `AWS::EC2::SubnetRouteTableAssociation` | Binds the `PublicSubnet` to the `PublicRouteTable` to enforce public routing rules. |

---

## Implementation Steps

The CloudFormation stack was built and deployed following these steps:

1. **Create the S3 Bucket (`WebAssetsBucket`):** Set up a S3 bucket to store the website required `index.html` showing my name and student number.
2. **Define IAM Roles and Policies:**
   - Created `WebsiteUploadLambdaRole` and attached `S3PutObjectPolicy` so the Lambda function has permissions to upload files to the S3 bucket.
   - Created `EC2S3AccessRole` and attached `S3AccessPolicy` so the EC2 instance can read files from the S3 bucket.
   - Created `EC2RoleProfile` (`InstanceProfile`) to pass the IAM role credentials directly to the EC2 instance.
3. **Create the Lambda Function (`WebsiteUploadLambda`):** Wrote the Node.js script inside the Lambda function resource to handle uploading `index.html` to the bucket.
4. **Trigger the Upload (`UploadIndexHtml`):** Created the CloudFormation Custom Resource to automatically execute the Lambda function during stack creation.
5. **Create the VPC Infrastructure (Part B):**
   - Created `CA1VPC` with CIDR block `10.0.0.0/16`.
   - Created `InternetGateway` and attached it to the VPC using `VPCGatewayAttachment`.
   - Created `PublicSubnet` (`10.0.1.0/24`) with auto-assign public IP enabled.
   - Created `PublicRouteTable` and added `PublicRoute` (`0.0.0.0/0` to Internet Gateway).
   - Associated `PublicSubnet` with `PublicRouteTable` using `PublicSubnetRouteTableAssociation`.
6. **Create the Security Group (`EC2SecurityGroup`):** Configured inbound rules on port 80 (HTTP) and port 22 (SSH) inside the custom VPC.
7. **Deploy the EC2 Instance (`WebServerEC2`):** Launched a `t3.micro` instance inside `PublicSubnet`, attached `EC2RoleProfile`, and used `UserData` to install Apache (`httpd`), download `index.html` from S3 using the AWS CLI, and start the web service.

## Testing and Validation

Both templates were validated using Infrastructure Composer / Application Composer (Template Designer) within the AWS CloudFormation Console. The YAML files were syntactically validated before stacks were deployed for functional testing.


<img width="1438" height="536" alt="image" src="https://github.com/user-attachments/assets/2a6e392c-297e-4e4d-ada7-efa2b73d7ef6" />

<img width="1445" height="510" alt="image" src="https://github.com/user-attachments/assets/90893704-a053-446b-b3b1-c9b2ca981fb4" />

<img width="859" height="146" alt="image" src="https://github.com/user-attachments/assets/ee326033-3201-4961-a58e-07761792ab3b" />




### Issues Encountered & Resolved

1. **Indentation Error:** Syntax error in the template regarding the alignment of the `ManagedPolicyArns` property inside the IAM role definition.

2. <img width="1411" height="646" alt="image" src="https://github.com/user-attachments/assets/22886a8d-10f3-434a-be9f-28d06994c683" />

3. **Web Content Fetch Failure:** The default Apache HTTP Server landing page was displayed instead of the custom `index.html` file containing my name and student number.

4. <img width="1181" height="519" alt="image" src="https://github.com/user-attachments/assets/12c93fe6-5216-46f1-83f0-9abd543614d8" />

**Troubleshooting Procedure:**
CloudWatch logs were analyzed to trace the deployment lifecycle. The logs confirmed that the Lambda function successfully uploaded `index.html` to the S3 bucket. However, the AWS CLI command required to fetch and download `index.html` from S3 to `/var/www/html/` was missing from the `UserData` bootstrap section of the `WebServerEC2` instance. Adding the appropriate `aws s3 cp` command resolved the issue.


<img width="1395" height="232" alt="image" src="https://github.com/user-attachments/assets/136a672f-b95a-4feb-a45d-0b4abb073e8e" />

<img width="975" height="165" alt="image" src="https://github.com/user-attachments/assets/301ff37b-e072-4049-bdc3-19aacbcab8f3" />




---



## References

### AWS IAM & Instance Profiles
* **AWS IAM Role Reference:** [AWS::IAM::Role Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-iam-role.html)
* **IAM Role Code Examples:** [AWS CloudFormation IAM Role Snippets](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-iam-role.html#aws-resource-iam-role--examples)
* **EC2 IAM Roles:** [IAM Roles for Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
* **IAM Quick Reference:** [AWS CloudFormation IAM Quick Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-iam.html)
* **Instance Profile Reference:** [AWS::IAM::InstanceProfile Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-iam-instanceprofile.html)
* **Using Instance Profiles:** [IAM User Guide - Switching Roles on EC2 Instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)

### AWS Lambda & Custom Resources
* **Custom Resource Reference:** [AWS::CloudFormation::CustomResource Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-cloudformation-customresource.html)
* **Custom Resource Response Reference:** [CloudFormation Custom Resource Handler Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/crpg-ref.html)

### AWS S3 Storage
* **S3 Bucket Configuration:** [AWS::S3::Bucket Documentation & Static Website Hosting](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-s3-bucket.html)
* **AWS Workshop Guide:** [AWS CloudFormation Hands-On Workshop (Lab 2)](https://catalog.us-east-1.prod.workshops.aws/workshops/df7f72cf-4f10-4664-acb6-b30dc8d4bcf0/en-US/lab2)

### AWS VPC Networking
* **VPC Resource Reference:** [AWS::EC2::VPC Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-vpc.html)
* **NAT Gateway Reference:** [AWS::EC2::NatGateway Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-natgateway.html)
* **VPC Quick Reference:** [AWS CloudFormation EC2 VPC Template Snippets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-ec2-vpc.html)

----

## Declaration of AI Tool Usage

In accordance with ATU academic integrity policies, Google Gemini was consulted as a technical reference, code debugging, and documentation formatting tool during this project.

* **Prompts & Queries Used:** 
  * Querying CloudFormation syntax for `AWS::Lambda::Function`, `CustomResource` triggers, and IAM execution roles (Part A).
  * Structuring custom Virtual Private Cloud (VPC) network components, including subnets, Internet Gateway attachments, and public route table associations (Part B).
  * Troubleshooting `UserData` bootstrap script execution issues and resolving YAML indentation syntax errors.
  * Formatting and proofreading the project `README.md` to structure resource tables, organize implementation steps, and correct grammatical errors.
* **Independent Verification & Refactoring:** All AI-generated code blocks were independently tested, modified to include strict IAM least-privilege scoping, linked to the `WebAssetsBucket` resource using CloudFormation pseudo-parameters, and integrated with explicit `DependsOn` dependencies to ensure deterministic stack creation.
* **Independent Verification & Refactoring:** All AI-generated code blocks were independently tested, modified to include strict IAM least-privilege scoping, linked to the `WebAssetsBucket` resource using CloudFormation pseudo-parameters, and integrated with explicit `DependsOn` dependencies to ensure deterministic stack creation.

### AI Declaration & Refactoring Log

#### 1. Initial Prompt & Question
> **User Prompt:** "How to create an index.html file inside S3 using CloudFormation?"

#### 2. Baseline Code Provided by LLM (Google Gemini)

```yaml
# Baseline snippet provided for Lambda Custom Resource setup
S3UploadLambda:
  Type: AWS::Lambda::Function
  Properties:
    Runtime: nodejs18.x
    Handler: index.handler
    Role: !GetAtt S3UploadLambdaRole.Arn
    Code:
      ZipFile: |
        const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
        const response = require('cfn-response');
        const s3 = new S3Client();
        
        exports.handler = async (event, context) => {
          if (event.RequestType === 'Delete') {
            return await response.send(event, context, response.SUCCESS);
          }
          try {
            await s3.send(new PutObjectCommand({
              Bucket: event.ResourceProperties.TargetBucket,
              Key: 'index.html',
              Body: '<html><body><h1>Hello World from S3!</h1></body></html>',
              ContentType: 'text/html'
            }));
            await response.send(event, context, response.SUCCESS);
          } catch (error) {
            console.error(error);
            await response.send(event, context, response.FAILED);
          }
        };

---
