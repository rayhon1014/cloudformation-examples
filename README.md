# cloudformation-examples

## S3
Create a static website purely using S3.
  * Create 2 S3 Buckets. One for name.com and one for www.name.com
  * Set up redirection from www bucket to non-www bucket.

![](https://raw.githubusercontent.com/widdix/aws-cf-templates/master/static-website/static-website.png)
  
  * **Parameters** - You can define a parameter (eg. RootDomainName) and being referenced by **!Ref** in the template
  * **AccessControl: PublicRead** - public read permissions are required for buckets set up for website hosting
  * **WebsiteConfiguration** - this section is needed to turn on website hosting. Under this section, you can define a redirection rules. In the example, www.[domain.com] will redirect to [domain.com]. And if **!Ref** is referencing the Resouce ID (eg. RootBucket), it will use the unique value of this bucket (its name in this case).
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
  S3BucketPolicy:    
    Type: 'AWS::S3::BucketPolicy'    
    Properties:      
      Bucket: !Ref RootBucket      
      PolicyDocument:        
      Statement:        
        - Action:          
          - 's3:GetObject'          
      Effect: Allow          
      Resource:          
        - !Sub 'arn:aws:s3:::${RootBucket}/*'          
      Principal: '*'
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
          Protocol: https
  ImageBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      AccessControl: PublicRead
      WebsiteConfiguration:
        IndexDocument: index.html
        RoutingRules:
          - RedirectRule:
              HttpRedirectCode: 307
              HostName: !Sub ${Api}.execute-api.${AWS::Region}.amazonaws.com
              Protocol: https
              ReplaceKeyPrefixWith: prod?key=
            RoutingRuleCondition:
              HttpErrorCodeReturnedEquals: 404
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

## Lambda
```
Lambda:
  Type: AWS::Lambda::Function,			
  Properties:
    Code:
       S3Bucket: !Ref" LambdaCodeBucket
       S3Key: !Ref LambdaCodeKey
    Handler: index.handler
    MemorySize: 128,	
    Role: !GetAtt LambdaRole.Arn
    Runtime: nodejs6.10
    Timeout: 60
    VpcConfig:
      SecurityGroupIds: !Ref LambdaSecurityGroup
      SubnetIds: !Ref Subnets
```

### Rules for references
```
# reference parameter
!Ref RootDomainName

# reference resource
!Ref RootBucket

# reference a property of a resource
!GetAtt RootBucket.WebsiteURL

```
