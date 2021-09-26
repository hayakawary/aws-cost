# aws料金確認
awsの使用金額をLineに通知  

①事前準備  
Line Developerコンソールにログイン。  
プロパイダー作成後、新規チャネルを作成。  
Messaging API設定からチャネルアクセストークンを発行してコピー  
  
②template.yamlのLambda環境変数に作成したチャネルアクセストークンを貼り付け  
LINE_CHANNEL_ACCESS_TOKEN: XXXXXXXXXXXXXXXXXXXX  
  
##srcフォルダにLine　botに必要なパッケージをdownload ※実施済みのためやらなくてよい  
##cd src  
##pip3 install LINE-bot-sdk -t .  
  
③アップロードするためのs3バケットを作成  
aws s3 mb s3://aws-cost-line-code  
  
④package cloudformation  
aws cloudformation package  --s3-bucket aws-cost-line-code --template-file template.yaml --output-template-file gen/template-generated.yaml  
  
genフォルダにtemplate-generated.yamlが作成されること  
  
⑤deploy   
aws cloudformation deploy --template-file gen/template-generated.yaml --stack-name aws-cost-line --capabilities CAPABILITY_IAM  
  
cloudformationにスタックができること  
  
⑥IAMポリシー作成  
ポリシー名:MyCostExplorerRead  
<pre>
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ce:GetCostAndUsage",
            "Resource": "*"
        }
    ]
}
</pre>
ポリシー作成後Lambdaの実行ロールにアタッチ。  
マネジメントコンソールから作成したLambda関数、設定、アクセス権限、実行ロールの順にクリック  
  
⑦APIgatewayのURLをコピー  
作成したAPIからステージに遍移して  
ステージ名(Prod)、/(スラッシュ)、API 名、POST の順にクリックすると  
URL の呼び出しに API の URL が画面上部に表示されるのでURLをコピー。  
※URLの呼び出し:https://XXXXXXXXXXXXXXXXXX  
  
⑧WebhookURLにAPIGatewayでコピーしたURLを登録  
LINE DevelopersコンソールからMessaging APIを開く。  
LINEチャネル名が保存されているプロバイダ名、チャネル名を順にクリック。  
チャネルの「チャネル基本設定」の右側にある Messaging API をクリック。  
  
「Webhook 設定」からWebhookURLを編集する。  
WebhookURLはAPIGatewayでコピーしたAPIのURLを貼り付けて「更新」をクリック。  
Webhook更新後は「検証」をクリック。  
検証に成功したら「Webhookの利用」の利用にチェックが入っていない場合はオン  
  
⑨使用  
返信パターンは 2 パターン  
「料金」または適当な文字列を送信すると現時点における今月の料金を返信  
「YYYY MM」と送信すると指定された日付の月初から月末までの利用料金を返信  
  
※デフォルトの応答メッセージを表示させたくない場合は「Messaging API 設定」の 「応答メッセージ」を無効にする。  
