ဒါလေးကတော့ မနေ့က ရေးခဲ့တဲ့ article ([Deploy NuxtJs app to AWS with SAM](https://kalaung.org/deploy-nuxtjs-app-to-aws-with-sam)) ရဲ့ အဆက်ပါ။

ကျွန်တော်တို့ မနေ့က SAM နဲ့ manual deploy တဲ့ အကြောင်းပြောခဲ့ပြီးပါပြီ။ ဒီနေ့မှာတော့ SAM ကို GitHub Action သုံးပြီး ဘယ်လို CI/CD setup လုပ်မလဲ ဆက်ရေးပေးသွားပါမယ်။

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
    
* နောက်တဆင့်ကတော့ AWS SAM ကို install မလုပ်ခင် python3 ကို ကြိုပြီး install လုပ်ထားတာပါ။
    
* ပြီးရင်တော့ ကျွန်တော်တို့ရဲ့ မင်းသားကြီး SAM ကို `aws-actions/setup-sam` သုံးပြီး install လုပ်ပါမယ်။
    
* လိုအပ်တာတွေ သွင်းပြီးပြီဆိုရင်တော့ AWS ကို access ရဖို့အတွက် `aws-actions/configure-aws-credentials` run ပါမယ်၊ ဒီနေရာမှာသုံးဖို့ aws access key ထုတ်တဲ့အပိုင်းနဲ့ permission ပေးတဲ့အပိုင်းကိုတော့ စာရှည်မှာစိုးလို့ အသေးစိတ်မပြောတော့ပါဘူး။
    
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
      - uses: actions/setup-python@v3
      - uses: aws-actions/setup-sam@v2
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

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724085718724/f1931a85-c4c0-4417-9865-5563ac475976.png align="center")

Github Action လေး success ဖြစ်သွားပြီဆိုရင်တော့ create လုပ်သွားတဲ့ CloudFront CNAME လေးကို domain controller ထဲမှာ setup လုပ်ပေးလိုက်ပြီး ကျွန်တော်တို့ရဲ့ domain လေးကို ဖွင့်ကြည့်လိုက်ရင်တော့။ ကျွန်တော်တို့ရဲ့ website အသစ်လေးကို တွေ့နိုင်ပြီပြီပါတယ်။ ဒီ article မှာ လုပ်ပြသွားတဲ့ demo project လေးကို [ဒီလင့်](https://github.com/thixpin/nuxt-sam/)မှာ ကြည့်ကြည့်လို့ရပါတယ်။ ([https://github.com/thixpin/nuxt-sam/](https://github.com/thixpin/nuxt-sam/))

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724085994558/e2ccadd1-2809-41de-947d-d9b1c128055c.png align="center")

အချိန်ပေးပြီး ဖတ်ရှုပေးတဲ့အတွက် ကျေးဇူးတင်ပါတယ်။