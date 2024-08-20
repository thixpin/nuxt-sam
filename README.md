ကျွန်တော်တို့ **SSR** (Server-Side Rendering) ပါ တဲ့ application တွေ ကို AWS ပေါ်မှာ serverless run ချင်တဲ့အခါမှာ Lambda ကိုသုံးလို့ရပါတယ်။ ပုံမှန်ဆိုရင်တော့ serverless framework ကိုသုံးပြီး deploy လိုက်တာက မြန်မြန်ဆန်ဆန်နဲ့ အလုပ်ရှုပ်သက်သာပါတယ်။ Serverless v4 က စပြီး ဝယ်သုံးဖို့လိုလာပါပြီ free က လည်း 2 credits ပဲ ပေးသုံးတာဖြစ်တဲ့အတွက် uat + dev မှာတင် လမ်းဆုံးပြီး production အတွက်က ဝယ်သုံးရမလိုဖြစ်နေပါပြီ။ Serverless v3 လည်း 2024 ကုန်ရင် security support end တော့မှာဆိုတော့ ရေရှည်မှာ အဆင်မပြေတဲ့အတွက် ကျွန်တော်ကတော့ SST နဲ့ AWS SAM တို့ထဲက တခုခုကို ပြောင်းသုံးဖို့ စဉ်းစားထားပါတယ်။ ရုံးမှာ ရှိနေပြီးသား NUXT နဲ့ရေးထားတဲ့ Frontend app တစ်ခုကို SAM နဲ့ deploy လုပ်ခဲ့တဲ့ အဆင့်လေးတွေကို ပြန်ပြီး မျှဝေပေးချင်ပါတယ်။ SST အကြောင်းကိုတော့ နောက်ကြုံမှ ထပ်ရေးပေးပါအုံးမယ်။

## Build nuxt app with aws-lambda preset

အရင်ဆုံး build လိုက်တဲ့ output က  Lambda ပေါ်မှာ တင် run လို့ရအောင် ပြင်ဖို့လိုပါတယ်။  
nuxt build လိုက်တဲ့အခါ်မှာ nitro ရဲ့ default set က `node-server` ဖြစ်တဲ့အတွက် lambda ပေါ်ဒီအတိုင်း တင် run လို့ မရပါဘူး။ အဲ့တော့ ကျွန်တော်တို့ NITRO\_PRESET ကို ပြောင်းပြီး build ကြည့် ပါမယ်။

```bash
 NITRO_PRESET=aws-lambda npm run build
```

ဒါဆိုရင်တော့ ကျွန်တော်တို့ build လိုက်တဲ့ output က lambda ပေါ်မှာ run လို့ရတဲ့ output လေး ထွက်လာပါပြီ။ nitro preset မှာ aws-lambda လို့ ပြနေပါတယ်။

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723969807136/3103edb1-1b85-4a47-b715-95ae7cd2b8fb.png)

nuxt build လိုက်တဲ့ output တွေက default အတိုင်းဆို `.output` dir ထဲမှာ သိမ်းပါတယ်။ `.output` ထဲမှာ ကြည့်လိုက်ရင် folder နှစ်ခု တွေ့ရမှာပါမယ်။ `public` folder က public asset တွေသိမ်းတာ ဖြစ်ပြီး `server` folder ကတော့ lambda ပေါ်တင် run ရမဲ့ build ပြီးသား script ဖိုင်တွေဖြစ်ပါတယ်။ `nuxt.config.ts` ဖိုင် ထဲမှာ ပြင်ပြီး `public` နဲ့ `server` ကို folder တစ်ခုတည်း ပေါင်းထုတ်ခိုင်းလို့လည်းရပါတယ်။ အဲ့တာဆိုရင်တော့ cloudfront ကနေ အသေးစိတ် handle လုပ်စရာမလိုပဲနဲ့ အကုန်လုံးကို lambda ပေါ်ကို ပေါင်းပြီးတင်လိုက်လို့ရတဲ့အတွက် ပိုရှင်းလင်းပြီး လွယ်ကူပါတယ်။ ဒါပေမဲ့ ဒီနည်းလမ်းက production မှာ မသုံးသင့်ပါဘူး။ static file တွေကို request လုပ်တိုင်း lambda function ထ run ရမယ်ဆိုရင် လုံးဝ မတန်ပါဘူး။ ဒါ့ကြောင့် public assess တွေကို s3 ပေါ်တင်သုံးတာက ပို သင့်တော်ပါတယ်။

## Creating Basic SAM Template

ကျွန်တော်တို့ SAM template file စရေးကြည့်ပါမယ်။ အရင်ဆုံး ထုံစံအတိုင်း env variables တွေ နဲ့ parameters  တွေကို ရေးလိုက်ပါမယ်။

```yaml
#template.yaml
---
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAM template for nuxt-lambda

Globals:
  Function:
    Runtime: nodejs18.x
    Timeout: 15
    Environment:
      Variables:
        NODE_ENV: !Ref NodeEnv
        NUXT_PUBLIC_ROOT_API: !Ref NuxtPublicRootApi
        NUXT_APP_URL: !Ref NuxtAppUrl


Parameters:
  Stage:
    Type: String
    Default: dev
  S3Bucket:
    Type: String
  NodeEnv:
    Type: String
    Default: production
  NuxtPublicRootApi:
    Type: String
  NuxtAppUrl:
    Type: String

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Application Configuration"
        Parameters:
          - Stage
          - NodeEnv
          - NuxtPublicRootApi
          - NuxtAppUrl
    ParameterLabels:
      Stage:
        default: "Deployment Stage"
      NodeEnv:
        default: "Node Environment"
      NuxtPublicRootApi:
        default: "Nuxt Public Root API"
      NuxtAppUrl:
        default: "Nuxt App URL"

Resources:

Outputs:
```

## Resources

နောက်တဆင့်မှာ Resource တွေထည့်ပါမယ်။ ကျွန်တော်တို့မှာ `cloudFront distribution`, `S3 Bucket`, `API Gateway`, နဲ့ `Lambda Function` ဆိုပြီး basic resource လေးခုထည့်ဖို့လိုပါမယ်။ နောက်ထပ် additional resources တွေအနေနဲ့ cloudfront ကနေ s3 ကို access ရဖို့အတွက် `s3 bucket policy` နဲ့ `CloudFront OAC` တို့ကို ထည့်ပေးဖို့လိုပါမယ်။  နောက်တဆင့်ကြရင်တော့ resource တစ်ခုခြင်းဆီအတွက် detail လေးတွေ ဆက်ရေးကြပါမယ်။

```yaml
Resources:
  # HttpApi to trigger the Lambda function
  HttpApi:

  # Lambda function to serve the Nuxt.js application
  LambdaFunction:

  # S3 bucket to store the Nuxt.js app's static assets
  StaticBucket:

  # S3 bucket policy to allow CloudFront to access the static assets
  StaticBucketPolicy:

  # CloudFront OAC to restrict access to the S3 bucket
  CloudFrontOriginAccessControl:

  # CloudFront distribution to serve the Nuxt.js app
  CloudFrontDistribution:
```

### HttpApi:

API GateWay အတွက် ကတော့ ဒီလိုလေးပဲ create လုပ်ပေးလိုက်ရင် လုံလောက်ပါတယ်။

```yaml
  # HttpApi to trigger the Lambda function
  HttpApi:
    Type: AWS::Serverless::HttpApi
```

### LambdaFunction:

Lambda function မှာတော့ လိုအပ်တဲ့ properties တွေနည်းနည်းများပါမယ်။ ဒီအထဲက အချို့ကိုရှင်းပြရရင်တော့

* BuildProperties ကတော့ lambda ပေါ်ကိုတင်တဲ့အခါမှာ မလိုအပ်တဲ့ ဖိုင်တွေပါမသွားပဲ `.output/server/` ထဲက ဖိုင်တွေပဲ ပါသွားဖို့အတွက်ဖြစ်ပါတယ်။
    
* Event &gt; ProxyResource ကတော့ အပေါ်မှာ create လုပ်ခဲ့တဲ့ HttpApi နဲ့ integrate လုပ်ဖို့ဖြစ်ပါတယ်။
    

```yaml
  # Lambda function to serve the Nuxt.js application
  LambdaFunction:
    Type: AWS::Serverless::Function
    Metadata:
      BuildMethod: nodejs18.x
      BuildProperties:
        Exclude:
          - '**/*'
        Include:
          - '.output/server/**'
    Properties:
      Handler: index.handler
      CodeUri: .output/server
      MemorySize: 256
      Policies: 
        - AWSLambdaBasicExecutionRole
      Events:
        ProxyResource:
          Type: HttpApi
          Properties:
            ApiId: !Ref HttpApi
            Path: $default
            Method: ANY
```

### StaticBucket:

static file တွေ သိမ်းဖို့အတွက် s3 bucket လေး create လုပ်ပါမယ်။

```yaml
  # S3 bucket to store the Nuxt.js app's static assets
  StaticBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      BucketName: !Ref S3Bucket
      OwnershipControls:
        Rules:
          - ObjectOwnership: BucketOwnerPreferred
```

### StaticBucketPolicy:

CloudFront နေဝင်လာတဲ့ request တွေကို access လုပ်ဖို့အတွက် s3 bucket မှာ policy လေးထည့်ပေးဖို့လိုပါတယ်။

```yaml
  # S3 bucket policy to allow CloudFront to access the static assets
  StaticBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref StaticBucket
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: "cloudfront.amazonaws.com"
            Action: "s3:GetObject"
            Resource: !Sub "${StaticBucket.Arn}/*"
            Condition:
              StringEquals:
                "AWS:SourceArn": !Sub "arn:aws:cloudfront::${AWS::AccountId}:distribution/${CloudFrontDistribution}"
```

### CloudFrontOriginAccessControl:

CloudFront ကနေ s3 ကို OAC နဲ့ လှမ်းခေါ်လို့ရဖို့ OAC တစ်ခု ကြိုပြီး create လုပ်ပါမယ်။

```yaml
  # CloudFront OAC to restrict access to the S3 bucket
  CloudFrontOriginAccessControl:
    Type: AWS::CloudFront::OriginAccessControl
    Properties:
      OriginAccessControlConfig:
        Name: !Sub "${AWS::StackName}-OAC"
        SigningProtocol: sigv4
        SigningBehavior: always
        OriginAccessControlOriginType: s3
```

### CloudFrontDistribution:

နောက်ဆုံးအနေနဲ့ CloudFront distribution create လုပ်ပါမယ်။ ကျွန်တော်တို့ရဲ့ distribution ထဲမှာ origin နှစ်ခုလိုပါမယ်။ တစ်ခုက S3 ဖြစ်ပြီး နောက်တစ်ခုကတော့ HttpApi ပါ။ Default Cache Behavior ထဲမှာတော့ Target Origin အဖြစ်နဲ့ HttpApi ကိုသုံးပါမယ်။ static file တွေကို request လုပ်တဲ့အခါမှာ S3 ကို forward လုပ်ပေးဖို့အတွက် Cache Behaviors တွေ ထပ်ထည့်ရပါမယ်။ public folder ထဲမှာ `_nuxt` , `images` folder နှစ်ခု နဲ့ `favicon.ico` ဖိုင် တခုရှိတဲ့အတွက် Cache behaviors စုစုပေါင်း သုံးခုလိုအပ်ပါတယ်။ ဒါကတော့ ကိုယ့် app ရဲ့ public folder ထဲမှာ ဘယ်နှစ်ခုရှိိလဲဆိုတာပေါ် မူတည်ပြီး ရေးဖို့လိုပါတယ်။

Aliases (Alternate domain name) နဲ့ ViewerCertificate certificate ကိုတော့ domain setup မလုပ်ချင်တော့လို့ comment ပိတ်ခဲ့ပါမယ်။ သုံးချင်ရင် comment ဖွင့်ပြီး နည်းနည်းပြင်သုံးလို့ရပါတယ်။

```yaml
  # CloudFront distribution to serve the Nuxt.js app
  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Enabled: true
        # Aliases:
        #   - !Ref CloudFrontDomain
        # ViewerCertificate:
        #   AcmCertificateArn: !Ref CloudFrontCertificate
        #   SslSupportMethod: sni-only
        Comment: !Sub "${AWS::StackName} ${Stage} Nuxt App"
        CustomErrorResponses:
          - ErrorCachingMinTTL: 0
            ErrorCode: 500
          - ErrorCachingMinTTL: 0
            ErrorCode: 504
        Origins:
          - Id: SsrOrigin
            # DomainName: !Select [2, !Split ["/", !Ref LambdaFunctionUrl]]
            DomainName: !Sub "${HttpApi}.execute-api.${AWS::Region}.amazonaws.com"
            OriginPath: ""
            CustomOriginConfig:
              HTTPSPort: 443
              OriginProtocolPolicy: https-only
              OriginSSLProtocols: 
                - TLSv1.2
          - Id: S3Origin
            # DomainName: !GetAtt StaticBucket.DomainName
            DomainName: !Sub "${StaticBucket}.s3.${AWS::Region}.amazonaws.com"
            OriginAccessControlId: !Ref CloudFrontOriginAccessControl
            S3OriginConfig: {}
        DefaultCacheBehavior:
          TargetOriginId: SsrOrigin
          ViewerProtocolPolicy: redirect-to-https
          AllowedMethods:
            - GET
            - HEAD
            - OPTIONS
            - PUT
            - PATCH
            - POST
            - DELETE
          CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad # CachingDisabled
          OriginRequestPolicyId: b689b0a8-53d0-40ab-baf2-68738e2966ac # AllViewerExceptHostHeader
          Compress: true
        CacheBehaviors:
          - PathPattern: "/_nuxt/*"
            TargetOriginId: S3Origin
            ViewerProtocolPolicy: redirect-to-https
            AllowedMethods:
              - GET
              - HEAD
              - OPTIONS
            Compress: true
            CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6 # CachingOptimized
            OriginRequestPolicyId: 88a5eaf4-2fd4-4709-b370-b4c650ea3fcf # CORS-S3Origin
          - PathPattern: "/images/*"
            TargetOriginId: S3Origin
            ViewerProtocolPolicy: redirect-to-https
            AllowedMethods:
              - GET
              - HEAD
              - OPTIONS
            Compress: true
            CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6 # CachingOptimized
            OriginRequestPolicyId: 88a5eaf4-2fd4-4709-b370-b4c650ea3fcf # CORS-S3Origin
          - PathPattern: "/favicon.ico"
            TargetOriginId: S3Origin
            ViewerProtocolPolicy: redirect-to-https
            AllowedMethods:
              - GET
              - HEAD
              - OPTIONS
            Compress: true
            CachePolicyId: 658327ea-f89d-4fab-a63d-7e88639e58f6 # CachingOptimized
            OriginRequestPolicyId: 88a5eaf4-2fd4-4709-b370-b4c650ea3fcf # CORS-S3Origin
```

## Outputs:

Resource တွေအကုန်ထည့်ပြီးတဲ့အခါမှာတော့ Output ထုတ်ပါမယ်။ ကျန်တာကိုတော့ အထွေအထူး မထုတ်တော့ပဲ cloudfront distribution domain တစ်ခုပဲ ထုတ်ကြည့်လိုက်ပါမယ်။

```yaml
Outputs:
  CloudFrontDistributionDomainName:
    Value: !GetAtt CloudFrontDistribution.DomainName
    Description: Domain name of the CloudFront distribution
```

## Deploy commands

ကျွန်တော်တို့ template file ရေးပြီးပြီဆိုရင်တော့ အရင်ဆုံး validate လုပ်ကြည့်ပါမယ်

```bash
sam validate --lint
```

validate လုပ်ကြည့်လို့ အတင်ပြေတယ်ဆိုရင်တော့ deploy လို့ရပါပြီ။ deploy မလုပ်ခင် build ထားပြီးသားဖြစ်ဖို့တော့ လိုပါမယ်နော်။ deploy တဲ့အခါမှာ guided mode နဲ့ deploy ဖို့အတွက် `sam deploy -g` command ကို သုံးပါမယ်။

```bash
NUXT_NITRO_PRESET=aws-lambda npm run build

 sam deploy -g
```

တစ်ခုသတိထားရမှာက sam deploy မလုပ်ခင် aws configure လုပ်ပြီး credentials တွေ ထည့်ထားဖို့ လိုပါလိမ့်မယ်။

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723975259956/d2e65ddc-ddb4-491f-8d9f-f21b1138ffdb.png)

prompt တွေအကုန်လုံးကို ဒီလို ရွေးပြီးသွားတဲ့အခါ်မှာတော့ aws ပေါ်မှာ deploy လုပ်ဖို့ ပြင်ဆင်ပြီးတဲ့အခါမှာ အောက်ကလိုမျိုး ပေါ်လာတဲ့အခါမှာ Y ကို ရွေးပြီး ဆက်သွားလိုက်ပါ။ resource တွေကို AWS ပေါ်မှာ deploy လုပ်ပေးပါလိမ့်မယ်။

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723975316800/71b4ed43-3379-4cc9-97e4-81c2938cee6e.png)

Deployment ပြီးတဲ့အချိန်မှာတော့ output ကို ဒီလိုမြင်ရပါမယ်

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723976293532/8855dd75-215b-4a06-beae-9efc88798513.png)

SAM deployment ပြီးရင် ကျွန်တော်တို့ တစ်ခုတော့ လုပ်ဖို့ကျန်ပါသေးတယ်။ .output/public ထဲက ဖိုင်တွေကို S3 ပေါ်တင်ပေးရပါမယ် `aws s3 sync` command ကို သုံးပြီးတင်ပေးလိုက်ပါ။

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723976365040/8119c00d-2ea4-4aef-8f5c-3e40b6078f5d.png)

ဒါတွေပြီးရင်တော့ ကျွန်တော်တို့ ရဲ့ deployment က အားလုံးပြီးသွားပါပြီ။ Result ကို sam deploy ပြီးတဲ့အခါမှာထွက်လာတဲ့ output ထဲက cloudfront domain ကို ဖွင့်ကြည့်ခြင်းအားဖြင့် မြင်တွေ့နိုင်ပါပြီ။

SAM ကို GitHub Action သုံးပြီး ဘယ်လို CI/CD setup လုပ်မလဲ ဆက်ရေးပေးသွားပါမယ်။

## Preparing SAM Template

ဒီတစ်ခါမှာတော့ Domain ပါ ထည့်သွားမှာ ဖြစ်တဲ့ အတွက် မနေ့က template ထဲမှာ Domain setup လုပ်ဖို့ နည်းနည်း ပြင်ပါမယ်။ Prameters ထဲမှာ `CertificateId` လေးထပ်ထည့်ပေးပြီး၊ `CloudFrontDistribution` ရဲ့ `DistributionConfig` ထဲက `ViewerCertificate` မှာ ပြန်ယူသုံးပါမယ်။ နောက်ပြီးတော့ alternative domain အတွက် `Aliases` ကိုတော့ `NuxtAppUrl` ထဲကနေ ယူပြီးထည့်ပေးလိုက်ပါမယ်။ ဒါဆိုရင်တော့ template မှာ ပြင်ဆင်လို့ပြီးသွားပါပြီ။

```yaml
Parameters:
  ...
  CertificateId:
    Type: String

....
Resources:
  ....
  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Enabled: true
        Aliases:
          - !Select ["2", !Split ["/", !Ref NuxtAppUrl]]
        ViewerCertificate:
          AcmCertificateArn: !Sub "arn:aws:acm:us-east-1:${AWS::AccountId}:certificate/${CertificateId}"
          SslSupportMethod: sni-only
        Comment: !Sub "${AWS::StackName} ${Stage} Nuxt App"
```

Complete template ကို [ဒီလင့်](https://github.com/thixpin/nuxt-sam/blob/main/template.yaml)မှာ ယူလို့ရပါတယ်။ ([template.yaml](https://github.com/thixpin/nuxt-sam/blob/main/template.yaml))

## Creating GitHub workflow

အရင်ဆုံး .`github/workflows` ဖိုဒါထဲမှာ YAML ဖိုင် တစ်ခု crearte လုပ်လိုက်ပါ။

ကျွန်တော်ကတော့ develop branch မှာ စမ်းပြီး deploy မှာ ဖြစ်တဲ့အတွက် `deploy-dev.yml` ဆိုပြီး create လုပ်လိုက်ပါတယ်။ ပြီးရင်တော့ ကျွန်တော်တို့ work flow စရေးလို့ရပါပြီ။ develop branch ကို push လုပ်တဲ့ အခါတိုင်း run ဖို့အတွက် ဒီလိုလေးစရေးပါတယ်။ သူ့အောက်မှာတော့ env ဆိုပြီး ကျွန်တော်တို့ အသုံးပြုမဲ့ environment variables သတ်မှတ်ပါမယ်။

```yaml
on:
  push:
    branches:
      - develop
```

### Environment Variables

ဒီထဲမှာ `${{ vars.*** }}` ဆိုပြီး သတ်မှခဲ့တဲ့ variables တွေကိုတော့ github action variable ထဲမှာ create လုပ်ပေးဖို့လိုပြီး `${{ secrets.*** }}` ဆိုပြီး သတ်မှတ် ခဲ့တာတွေကိုတော့ github action secret ထဲမှာ ထည့်ပေးဖို့လိုပါတယ်။ CERTIFICATE\_ID အတွက် ထည့်ပေးရမဲ့ value ကတော့ ကိုယ်သုံးမဲ့ domain အတွက် aws certificate manager ထဲမှာ ထုတ်ပြီး ယူထည့်ပေးရမှာပါ။

```yaml
env:
  DEPLOY_STAGE: dev
  NODE_ENV: local
  APP_NAME: thixpin-nuxt
  PUBLIC_ROOT_API: ${{ vars.PUBLIC_ROOT_API }}
  APP_URL: ${{ vars.APP_URL }}
  AWS_REGION: ap-southeast-1
  S3_BUCKET_NAME: thixpin-nuxt-static-dev
  CERTIFICATE_ID: ${{ secrets.CERTIFICATE_ID }}
```

### Jobs setups

jobs တွေတော့ ကျွန်တော် အများကြီး မခွဲတော့ပါဘူး deploy ဆိုပြီး တစ်ခုပဲ ထားလိုက်ပါမယ်။ setups ထဲမှာတော့ action တွေနည်းနည်းများပါမယ်။

* ပထမဆုံး actions/checkout ကိုသုံးပြီး working dir ထဲကို source code တွေယူလိုက်ပါမယ်။
    
* ပြီးရင်တော့ node project ဖြစ်တဲ့အတွက် actions/setup-node ကိုသုံးပါမယ်။
    
* ပြီးရင်တော့ ကျွန်တော်တို့ရဲ့ မင်းသားကြီး SAM ကို `aws-actions/setup-sam` သုံးပြီး install လုပ်ပါမယ်။ `use-installer:` ဆိုတဲ့ option လေးကို true ပေးခဲ့သင့်ပါတယ်။ ဒါမှ install လုပ်တာကမြန်မှာပါ၊ off ထားရင်တော့  python တွေပါ ထည့်မှာဖြစ်တဲ့အတွက် runtime မှာ အချိန် တော်တော်ပေးရပါတယ်။
    
* လိုအပ်တာတွေ သွင်းပြီးပြီဆိုရင်တော့ AWS ကို access ရဖို့အတွက် `aws-actions/configure-aws-credentials` run ပါမယ်၊ ဒီနေရာမှာသုံးဖို့ aws access key ထုတ်တဲ့အပိုင်းနဲ့ permission ပေးတဲ့အပိုင်းကိုတော့ စာရှည်မှာစိုးလို့ အသေးစိတ်မပြောတော့ပါဘူး၊ ဒီမှာတော့ demo project ဖြစ်တာမို့ AWS Access key ကို ပဲ အလွယ်သုံးလိုက်တာပါ။ လက်တွေ့မှာ ပိုမို လုံခြုံဖို့အတွက်တိုရင် GitHub ရဲ့ OIDC provider နဲ့ချိိတ်ပြီး Assume Role နဲ့ သုံးတာက ကိုကောင်းပါတယ်။ AccessKey ကပေါက်သွားနိုင်တဲ့ အန္တရာယ်ရှိပါတယ်။ Action run မဲ့ User or Role ကိုလည်း permission ကို လိုအပ်သလောက်ပဲ ပေးသင့်ပါတယ်။ first time run ပြီးသွားတဲ့အခါမှာ နောက်ပိုင်းက lambda နဲ့ s3 လောက်ကိုပဲ အဓိက ပြင်တော့မှာ ဖြစ်တဲ့အတွက် ဆက်မသုံးတော့မယ့် permission တွေဖြုတ်ထားလို့ရပါတယ်။ permission ပေးတဲ့ နေရာမှာလည်း resource အတိအကျနဲ့ပဲ သုံးလို့ရအောင်ပေးထားခြင်းအားဖြင့် အခြား resource တွေကို access မရအောင် ကာကွယ်ထားသင့်ပါတယ်။
    
* နောက်တဆင့်မှာတော့ build မလုပ်ခင် .env ဖိုင်ဆောက်ပါမယ်
    
* .env ဖိုင်လည်းရပြီဆိုရင်တော့ npm install လေး run ဖို့ကျန်ပါသေးတယ်။
    
* ပြင်ဆင်စရာတွေ အားလုံးပြီးတဲ့အခါမှာတော့ `NITRO_PRESET=aws-lambda npm run build` command လေး run ပြီး source code ကို build လိုက်ပါမယ်။
    
* build လို့လည်း ပြီးသွားတဲ့ခါမှာ `sam deploy` comand သုံးပြီး၊ deploy လိုက်လို့ရပြီပြီ။
    
* အကုန်ပြီးပြီဆိုရင်တော့ နောက်ဆုံးအနေနဲ့ `aws s3 sync` command လေးသုံးပြီး public dir ထဲက file တွေကို s3 ပေါ်တင်ပေးလိုက်ပါမယ်။
    

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      - uses: aws-actions/setup-sam@v2
        with:
          use-installer: true
      - uses: aws-actions/configure-aws-credentials@v3
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      - run: |
          touch .env
          echo "NODE_ENV=${{ env.NODE_ENV }}" >> .env
          echo "NUXT_APP_NAME=${{ env.APP_NAME }}" >> .env
          echo "NUXT_PUBLIC_ROOT_API=${{ env.PUBLIC_ROOT_API }}" >> .env
          echo "NUXT_APP_URL=${{ env.APP_URL }}" >> .env
          echo "NUXT_APP_I18N_FALLBACK_LOCALE=en" >> .env
      - run: npm install
      - run: NITRO_PRESET=aws-lambda npm run build
      - run: |
          sam deploy --no-confirm-changeset \
            --no-fail-on-empty-changeset --resolve-s3 \
            --capabilities CAPABILITY_IAM \
            --stack-name ${{ env.APP_NAME }}-${{ env.DEPLOY_STAGE }} \
            --s3-prefix ${{ env.APP_NAME }}-${{ env.DEPLOY_STAGE }} \
            --parameter-overrides \
                Stage=${{ env.DEPLOY_STAGE }} \
                NodeEnv=${{ env.NODE_ENV }} \
                NuxtAppName=${{ env.APP_NAME }} \
                NuxtPublicRootApi=${{ env.PUBLIC_ROOT_API }} \
                NuxtAppUrl=${{ env.APP_URL }} \
                S3Bucket=${{ env.S3_BUCKET_NAME }} \
                CertificateId=${{ env.CERTIFICATE_ID }}
      - run: aws s3 sync ./.output/public s3://${{ env.S3_BUCKET_NAME }} --delete
```

အပေါ်က ရေးခဲ့တဲ့ ဖိုင်အပြည့်အစုံကို ဒီလင့်မှာ ယူလို့ရပါတယ်။

Develop branch ထဲမှာ workflos file လေး create လုပ်ပြီး push လိုက်မယ်ဆိုရင်တော့ GitHub Action လေးက အောက်ကလို run သွားပါလိမ့်မယ်။

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724085718724/f1931a85-c4c0-4417-9865-5563ac475976.png)

Github Action လေး success ဖြစ်သွားပြီဆိုရင်တော့ create လုပ်သွားတဲ့ CloudFront CNAME လေးကို domain controller ထဲမှာ setup လုပ်ပေးလိုက်ပြီး ကျွန်တော်တို့ရဲ့ domain လေးကို ဖွင့်ကြည့်လိုက်ရင်တော့။ ကျွန်တော်တို့ရဲ့ website အသစ်လေးကို တွေ့နိုင်ပြီပြီပါတယ်။ ဒီ article မှာ လုပ်ပြသွားတဲ့ demo project လေးကို [ဒီလင့်](https://github.com/thixpin/nuxt-sam/)မှာ ကြည့်ကြည့်လို့ရပါတယ်။ ([https://github.com/thixpin/nuxt-sam/](https://github.com/thixpin/nuxt-sam/))

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724085994558/e2ccadd1-2809-41de-947d-d9b1c128055c.png)

အချိန်ပေးပြီး ဖတ်ရှုပေးတဲ့အတွက် ကျေးဇူးတင်ပါတယ်။