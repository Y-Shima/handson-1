# handson-1

1. コンソールからスマホにショートメールを送ってみる。

2. コマンドラインからスマホにショートメールを送ってみる。  
- EC2起動  
  IAMプロファイルを作成  
  IAMポリシーは AmazonSSMManagedInstanceCoreをつける。  
- EC2に接続する
  ec2-userになる。
  ```
  sudo su - ec2-user
  ```
- AWS CLIからトピックにメッセージを発行する。  
  https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sns/publish.html
  ```
  aws sns publish --topic-arn "arn:aws:sns:ap-northeast-1:<アカウント>:handson-sns" --message "こんにちは" --subject test
  ```  
  <details>
  <summary>ヒント</summary>
    EC2のIAMポリシーに AmazonSNSFullAccess が必要。
  </details>

3. Pythonのプログラムからスマホにショートメールを送ってみる。  
- Lambda（ラムダ）関数を作成する。
  Pythonを選択する。
  ``` Python
  import json
  import boto3
  
  client = boto3.client('sns')
  
  def lambda_handler(event, context):
      response = client.publish(
          TopicArn='arn:aws:sns:ap-northeast-1:486069495702:handson-sns',
          Message='こんにちは',
          Subject='test from Lambda',
      )
      return {
          'statusCode': 200,
          'body': json.dumps('Hello from Lambda!')
      }
  ```
  https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns.html
  https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns/client/publish.html
- Deploy　→　Test
    <details>
  <summary>ヒント</summary>
    LambdaのIAMポリシーに AmazonSNSFullAccess が必要。  
    設定 → アクセス権限　実行ロールがIAMロール。ここにポリシーを追加する。  
  </details>

4. APIを使って、スマホにショートメールを送ってみる。その1
- API Gatewayを作成する。
  REST APIを作成  
  - メソッドを作成 Get  
    Lambdaプロキシ統合をON  
  - APIをデプロイ  
    ステージを作成する
- URLを呼び出す
  以下の形式にしたい。  
  ```
  https://<API ID>.execute-api.ap-northeast-1.amazonaws.com/api?subject=test&message=こんにちは
  ```
  いったん呼び出してみる。
- Lambdaを更新する。
  関数(lambda_handler)の1行目に以下を追加する。
  ```
  print(json.dumps(event, ensure_ascii=False))
  ```
  もう一度、URLを呼び出す。  
  ログを確認  
  Lambdaのモニタリング → CloudWatchログ  
  queryStringParametersに、subjectとmessageがあることが分かる。これは、「Lambdaプロキシ統合」による。  
  Lambdaコード  
```Python
import json
import boto3

client = boto3.client('sns')

def lambda_handler(event, context):
    print(json.dumps(event, ensure_ascii=False))
    
    if 'queryStringParameters' in event:
        my_message = event.get('queryStringParameters').get('message')
        my_subject = event.get('queryStringParameters').get('subject')
    else:
        my_message = 'こんにちは'
        my_subject = 'test from Lambda'
    
    response = client.publish(
        TopicArn='arn:aws:sns:ap-northeast-1:486069495702:handson-sns',
        Message=my_message,
        Subject=my_subject,
    )
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }

```
  もう一度発行してみる。  
  ```
  https://xb2n1f3blb.execute-api.ap-northeast-1.amazonaws.com/api?subject=test&message=こんにちは
  ```

5. APIを使って、スマホにショートメールを送ってみる。その2  
  もう一度EC2に接続して、curlで実行してみる。  
   ```
   curl https://xb2n1f3blb.execute-api.ap-northeast-1.amazonaws.com/api?subject=test&message=こんにちは
   ```
  …うまくいくには？  

6. 後片付け  
   EC2を削除する。  
   他は課金対象はありません。  
