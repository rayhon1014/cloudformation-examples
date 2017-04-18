# cloudformation-examples

## S3
  * **AccessControl: PublicRead** - public read permissions are required for buckets set up for website hosting
  * **WebsiteConfiguration** - this section is needed to turn on website hosting
  * **DeletionPolicy: Retain** -  Because this bucket resource has a DeletionPolicy attribute set to Retain, AWS CloudFormation will not delete this bucket when it deletes the stack.
  * **!Sub**: ${Domain} is a custom variable with its value filled by $GetAttr defined at the next line**
```
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      AccessControl: PublicRead
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
    DeletionPolicy: Retain
Outputs:
  WebsiteURL:
    Value: !GetAtt S3Bucket.WebsiteURL
    Description: URL for the website hosted on S3
  S3BucketSecureURL:
    Value: !Sub
        - https://${Domain}
        - Domain: !GetAtt S3Bucket.DomainName
    Description: Name of the S3 bucket to hold website content
```
