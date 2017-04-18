# cloudformation-examples

## S3
  * **Parameters** - You can define a parameter (eg. RootDomainName) and being referenced by **!Ref** in the template
  * **AccessControl: PublicRead** - public read permissions are required for buckets set up for website hosting
  * **WebsiteConfiguration** - this section is needed to turn on website hosting
  * **DeletionPolicy: Retain** -  Because this bucket resource has a DeletionPolicy attribute set to Retain, AWS CloudFormation will not delete this bucket when it deletes the stack.
  * **!Sub**: ${Domain} is a custom variable with its value filled by $GetAttr defined at the next line
  
```
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  RootDomainName:
    Description: Domain name for your website (example.com)
    Type: String
Resources:
  RootBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref RootDomainName
      AccessControl: PublicRead
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: 404.html
  WWWBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub
          - www.${Domain}
          - Domain: !Ref RootDomainName
      AccessControl: BucketOwnerFullControl
      WebsiteConfiguration:
        RedirectAllRequestsTo:
          HostName: !Ref RootBucket
Outputs:
  WebsiteURL:
    Value: !GetAtt RootBucket.WebsiteURL
    Description: URL for website hosted on S3
  S3BucketSecureURL:
    Value: !Sub
        - https://${Domain}
        - Domain: !GetAtt RootBucket.DomainName
    Description: Name of the S3 bucket to hold website content
```
