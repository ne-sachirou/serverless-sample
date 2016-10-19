Serverless Framworkサンプル
==
當文書の[Serverless](https://serverless.com/)のver.は1.0.2である。

事前に準備するもの
--
1. AWSアカウント
2. 最新のNode.jsとNPM

準備
--
### Serverless Framworkを安裝する
```sh
npm install -g serverless
serverless -v
```

serverless若しくはsls若しくはslssコマンドが使へるやうになる。

### Serverless用のIAMユーザーを作る
AWSコンソールのIAM Management Consoleへゆく。新規IAMユーザーを作り、AdministratorAccessポリシーを當てる。或いは既存の、AdministratorAccessポリシーを持ったIAMユーザーを使ってもよい。AWS_ACCESS_KEY_IDとAWS_SECRET_ACCESS_KEYを覺えておく。

<small>※ServerlessはAWSの各種リソースを廣汎にデプロイできるものなのでAdministratorAccessを當ててゐる。精査できる場合は必要な權限を精査して制限してもよい。</small>

Serverlessプロジェクトを作る
--
### Serverlessプロジェクトを作る
serverless-sampleと云ふプロジェクトを作る。

```sh
mkdir serverless-sample
cd serverless-sample
sls create --template aws-nodejs --name serverless-sample
```

- event.json :slsコマンドからLamndaを實行する時の引數
- handler.js :AWS Lambdaで實行されるコード
- serverless.yml :Serverlessでデプロイするものの定義。函數 (複數個)、環境變數、それぞれの函數を呼び出すイベント、函數が使ふリソース等を定義する

が作られる。serverless.ymlのproviderの項に以下を追記する。

```yaml
provider:
  # ...
  stage: dev
  region: ap-northeast-1
```

### デプロイする
AWS_ACCESS_KEY_IDとAWS_SECRET_ACCESS_KEYを思ひ出す ([direnv](http://direnv.net/)等べんり)。デプロイすると、

- CloudFormation :serverless-sample-dev
- Lambda :serverless-sample-dev-hello

が作られる。

```sh
export AWS_ACCESS_KEY_ID=キー
export AWS_SECRET_ACCESS_KEY=シークレット
sls deploy -s dev
```

```
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
...........
Serverless: Stack create finished...
Serverless: Packaging service...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading service .zip file to S3...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
.......
Serverless: Stack update finished...

Service Information
service: serverless-sample
stage: dev
region: ap-northeast-1
api keys:
  None
endpoints:
  None
functions:
  serverless-sample-dev-hello: arn:aws:lambda:ap-northeast-1:云々:function:serverless-sample-dev-hello
```

實行するとJSONが返される。

```sh
sls invoke -f hello -s dev
```

```json
{
    "statusCode": 200,
    "body": "{\"message\":\"Go Serverless v1.0! Your function executed successfully!\",\"input\":{}}"
}
```

### API Getewayをイベントに設定する
serverless.ymlのfunctionsの項にeventを追記する。

```yaml
functions:
  hello:
    # ...
    events:
      - http: GET momonga
```

デプロイする。

```sh
sls deploy -s dev
```

```
Serverless: Packaging service...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading service .zip file to S3...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
....................
Serverless: Stack update finished...

Service Information
service: serverless-sample
stage: dev
region: ap-northeast-1
api keys:
  None
endpoints:
  GET - https://云々.execute-api.ap-northeast-1.amazonaws.com/dev/momonga
functions:
  serverless-sample-dev-hello: arn:aws:lambda:ap-northeast-1:云々:function:serverless-sample-dev-hello
```

先程に加えてAPI Gatewayにdev-serverless-sampleが作成されてゐる。httpでAPI Getewayを呼ぶとJSONが返される。

```sh
http https://云々.execute-api.ap-northeast-1.amazonaws.com/dev/momonga
```

```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 1310
Content-Type: application/json
Date: Wed, 19 Oct 2016 05:09:36 GMT
Via: 1.1 云々.cloudfront.net (CloudFront)
X-Amz-Cf-Id: 云々
X-Cache: Miss from cloudfront
x-amzn-RequestId: 云々

{
    "input": {
        "body": null,
        "headers": {
            "Accept": "*/*",
            "Accept-Encoding": "gzip, deflate",
            "CloudFront-Forwarded-Proto": "https",
            "CloudFront-Is-Desktop-Viewer": "true",
            "CloudFront-Is-Mobile-Viewer": "false",
            "CloudFront-Is-SmartTV-Viewer": "false",
            "CloudFront-Is-Tablet-Viewer": "false",
            "CloudFront-Viewer-Country": "JP",
            "Host": "云々.execute-api.ap-northeast-1.amazonaws.com",
            "User-Agent": "HTTPie/0.9.6",
            "Via": "1.1 云々.cloudfront.net (CloudFront)",
            "X-Amz-Cf-Id": "云々",
            "X-Forwarded-For": "云々",
            "X-Forwarded-Port": "443",
            "X-Forwarded-Proto": "https"
        },
        "httpMethod": "GET",
        "path": "/momonga",
        "pathParameters": null,
        "queryStringParameters": null,
        "requestContext": {
            "accountId": "云々",
            "apiId": "云々",
            "httpMethod": "GET",
            "identity": {
                "accountId": null,
                "apiKey": null,
                "caller": null,
                "cognitoAuthenticationProvider": null,
                "cognitoAuthenticationType": null,
                "cognitoIdentityId": null,
                "cognitoIdentityPoolId": null,
                "sourceIp": "云々",
                "user": null,
                "userAgent": "HTTPie/0.9.6",
                "userArn": null
            },
            "requestId": "云々",
            "resourceId": "云々",
            "resourcePath": "/momonga",
            "stage": "dev"
        },
        "resource": "/momonga",
        "stageVariables": null
    },
    "message": "Go Serverless v1.0! Your function executed successfully!"
}
```
