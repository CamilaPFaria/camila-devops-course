# camila-devops-course
Cloud Formation Assignment














## Declaration of AI Tool Usage

In accordance with ATU academic integrity policies, **Google Gemini** was consulted as a technical reference tool during the initial planning phase of this project. 

* **Prompts Used:** Querying CloudFormation syntax for `AWS::Lambda::Function`, `CustomResource` triggers, and IAM execution roles.
* **Independent Verification & Refactoring:** All AI-generated code blocks were independently tested, modified to include strict IAM least-privilege scoping, linked to the `WebAssetsBucket` resource using CloudFormation pseudo-parameters, and integrated with explicit `DependsOn` dependencies to ensure deterministic stack creation.


## AI Declaration & Refactoring Log

### 1. Initial Prompt & Question
> **User Prompt:** "how to create an index.html file inside s3 using cloud formation?"

### 2. Baseline Code Provided by LLM (Google Gemini)
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
```

