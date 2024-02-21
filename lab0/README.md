# AWS Lab 0 - 初期設定

## Dynatrace環境

ワークショップで使用するDynatraceにログインするためのURLが記載されたメールを送付しております。メールにログインURL、ID・パスワードが記載されておりますので、そちらの情報を元にログインをしてください。

![image](img/handson-mail.png)

ログインができないなどがございましたら、講師までお知らせください。

## AWSアカウント

AWSアカウントについてはAWS Workshop studioサービスを通じて提供されます。ログインに必要なURLについても先ほどのメールに記載がございます。

1. 普段ご利用のAWSアカウントでログインしている場合は、ログアウトいただくか、ブラウザのプライベートウィンドウを利用してください。

2. メールに記載のURLにアクセスしていただくと以下の画面が表示されます。`Email one-time password (OTP)`を選びます。

![image](img/one-time-password.png)

3. メールアドレスを入力し、`Send passcode`ボタンをクリックします。

![image](img/send-passcode.png)

4. しばらくすると登録したメールアドレスにpasscodeが届くので、それを入力し,`Sign in`ボタンをクリックします。

![image](img/enter-passcode.png)

5. **Completing sign-in**というページが表示されますので、しばらく待ちます。

![image](img/Completing_sign-in.png)

6. **Review and join**のページが表示されたら、下にスクロールし`I agree with the Terms and Conditions`にチェックを入れ、`Join event`ボタンをクリックしてください。

![image](img/Review-and-join.png)
![image](img/Join-event.png)

7. **Event information**画面が表示されたら、画面左側の`Open AWS Console (ap-southeast-1)`をクリックします。

![image](img/Event-information.png)

8. 新しいブラウザのタブが開き、AWSのコンソールが表示されます。

![image](img/aws-console.png)

9. ここまで完了したら、環境の準備は完了です。

### 💥 **TECHNICAL NOTE**

*有効期限が切れると、作成されたすべてのリソースは自動的に削除され、アカウントにアクセスすることができなくなります。*

## 監視対象サーバーの構築

<!--
### 1. Make sure you are in the correct region

Click the region button in the top right corner of your AWS console and make sure you are in `Oregon us-west-2` for consistency in this lab.

![image](img/lab2-change-region.png)
-->

### 1. Cloudshellの開始

このラボでは、AWS Cloudshellを使用します。Cloudshellはブラウザベースのシェルで、AWSリソースを安全に管理、利用することが可能です。

Cloudshellを開くには、AWSコンソールの上部にあるCloudshellアイコンをクリックします。 これには1分ほど時間がかかります。

また、リージョンは**シンガポール**が選択されていることを確認してください。

![image](img/setup-cloud-shell-icon.png)

下の画像のページが開く場合があります。その場合は**Close**ボタンをクリックしてウィンドウを閉じます。

![image](img/cloudshell-splash-page.png)

ポップアップを閉じた後、Cloudshellが初期化されるのを1分ほど待ちます。これが終わると、以下のようなコマンドプロンプトが表示されます。

![image](img/setup-cloud-shell.png)

### 2. ワークショップスクリプトのクローン

Cloudshellを開いたら、ワークショップのセットアップを自動化するスクリプトをいくつか入手する必要があります。 以下のコマンドを実行してください：

```
git clone https://github.com/dt-alliances-workshops/aws-modernization-dt-orders-setup.git
chmod +x aws-modernization-dt-orders-setup/provision-scripts/provision-workshopOLD.sh
rm -f aws-modernization-dt-orders-setup/provision-scripts/provision-workshop.sh
mv aws-modernization-dt-orders-setup/provision-scripts/provision-workshopOLD.sh aws-modernization-dt-orders-setup/provision-scripts/provision-workshop.sh
```

下の画像のように出力されます。

```
[cloudshell-user@ip-10-130-42-168 ~]$ git clone https://github.com/dt-alliances-workshops/aws-modernization-dt-orders-setup.git
Cloning into 'aws-modernization-dt-orders-setup'...
remote: Enumerating objects: 523, done.
remote: Counting objects: 100% (161/161), done.
remote: Compressing objects: 100% (89/89), done.
remote: Total 523 (delta 111), reused 94 (delta 72), pack-reused 362
Receiving objects: 100% (523/523), 19.89 MiB | 15.35 MiB/s, done.
[cloudshell-user@ip-10-130-42-168 ~]$ git clone https://github.com/dt-alliances-workshops/aws-modernization-dt-orders-setup.git
Cloning into 'aws-modernization-dt-orders-setup'...
remote: Enumerating objects: 523, done.
remote: Counting objects: 100% (161/161), done.
remote: Compressing objects: 100% (89/89), done.
remote: Total 523 (delta 111), reused 94 (delta 72), pack-reused 362
Receiving objects: 100% (523/523), 19.89 MiB | 16.28 MiB/s, done.
Resolving deltas: 100% (273/273), done.
[cloudshell-user@ip-10-130-42-168 ~]$ chmod +x aws-modernization-dt-orders-setup/provision-scripts/provision-workshopOLD.sh
[cloudshell-user@ip-10-130-42-168 ~]$ rm -f aws-modernization-dt-orders-setup/provision-scripts/provision-workshop.sh
[cloudshell-user@ip-10-130-42-168 ~]$ mv aws-modernization-dt-orders-setup/provision-scripts/provision-workshopOLD.sh aws-modernization-dt-orders-setup/provision-scripts/provision-workshop.sh
[cloudshell-user@ip-10-130-42-168 ~]$
```

## EC2インスタンスのプロビジョニング

2つのCloudFormationを実行すると以下が完了します

* `dt-orders-monolith`と`dt-orders-services`の2つのEC2インスタンスの作成
* EC2インスタンスには`Docker`と`Docker-Compose`がインストールされます
* EC2インスタンスには自動でDynatraceのテナントから`OneAgent`が設定されます
* `docker-compose up`を実行し、サンプルアプリケーションを起動します。

### 1. プロビジョニングスクリプトの取得

Dynatraceの左側のメニューから`ダッシュボード`を開きます。

ダッシュボードページで**ワークショップダッシュボード**をクリックします。

![image](img/dt-provision-dashboard-list.png)

下の画像を参考にタイル右上の▽をクリックし、**タイルの編集**を選びます。コマンドが記載されている**Markdown**をクリックします。コマンドを`すべて選択`し、`コピー`をします。

![image](img/edit-tile.png)

![image](img/copy-markdown.png)


**完了**ボタンをクリックし、編集モードから抜けます。

### 2. プロビジョニングスクリプトの実行

AWS Cloudshellに戻り、先ほどのマークダウンのコマンドを貼り付けます。以下のような画面が表示されるので`y`を押してセットアップを実行します。

```
===================================================================
About to Provision Workshop for:
https://mou612.managed-sprint.dynalabs.io/e/XXXXX
SETUP_TYPE   = all
KEYPAIR_NAME = ee-default-keypair
===================================================================
Proceed? (y/n) :
```

スクリプトの実行が完了すると、以下のような画面が表示されます。

```
-----------------------------------------------------------------------------------
Done Setting up Workshop config
End: Mon Feb 12 03:06:19 AM UTC 2024
-----------------------------------------------------------------------------------

=============================================
Provisioning workshop resources COMPLETE
End: Mon Feb 12 03:06:19 AM UTC 2024
=============================================
```

### 3. AWS CloudFormationの結果確認

CloudFormationの実行には数分かかります。その間、CloudFormationの画面から途中経過が確認できます。
AWSの管理コンソールから`CloudFormation`のページを開きます。

* <a href="https://console.aws.amazon.com/cloudformation/home" target="_blank">https://console.aws.amazon.com/cloudformation/home</a>

CloudFormation Stackが完了するとStatusが`CREATE_COMPLETE`に変更されます。

![image](img/aws-cf-complete-both.png)

### 💥 **TECHNICAL NOTE**

CloudFormationの実行にはおよそ5分程度時間が掛かります。

## Dynatrace の設定

上記のスクリプト内で以下のDynatraceの設定が自動で実行されています。

* <a href="https://www.dynatrace.com/support/help/how-to-use-dynatrace/problem-detection-and-analysis/problem-detection/detection-of-frequent-issues/" target="_blank">Frequent Issue Detection</a>の設定の無効化
* <a href="https://www.dynatrace.com/support/help/shortlink/problem-detection-sensitivity-services" target="_blank">Service Anomaly Detection</a>の調整
* <a href="https://www.dynatrace.com/support/help/how-to-use-dynatrace/management-zones/" target="_blank">Management Zones</a>の追加
* SLOとManagement Zonesの設定に必要な<a href="https://www.dynatrace.com/support/help/how-to-use-dynatrace/tags-and-metadata/" target="_blank">Auto Tagging Rules</a>の追加
* カスタムダッシュボード作成に必要な<a href="https://www.dynatrace.com/support/help/how-to-use-dynatrace/service-level-objectives/" target="_blank">SLOs</a>の追加

Dynatraceの設定変更には [Dynatrace Monitoring as Code](https://github.com/Dynatrace/dynatrace-configuration-as-code) (monaco)と [Dynatrace Configuration API](https://www.dynatrace.com/support/help/dynatrace-api/configuration-api/)を使用しています。

<!-- You can review the Monitoring as Code workshop files [in the GitHub repo](https://github.com/dt-alliances-workshops/aws-modernization-dt-orders-setup/tree/main/workshop-config) -->

## 本セクションのまとめ

このセクションでは、以下を実施しました。

✅ Dynatraceの環境にアクセスできることの確認

✅ AWSの環境にアクセスできることの確認

✅ ワークショップに必要なリソースの準備

それでは、次のセクションに進みましょう！
