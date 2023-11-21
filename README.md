## 概要
CloudFormation をより深掘りして学んでいく為、AWS無料枠内でシンプルなアーキテクチャWordPressを構築していきます。  
※WordPress用のDBユーザー、DBは今後インストールしていきます。

- EC2 インスタンスを起動時に ElasticIPでIPアドレスの固定をしないと、毎起動時に Public IPアドレスが変わってしまいます。 ですが、ElasticIPは1日16円ほどの料金が発生する為、今回は検証、アウトプットを含めた構築なので利用しません。
- AWSの無料枠では、t2.microインスタンス1か月あたり 750時間まで無料です。
- **Amazon Linux 2では`yum`を使ってパッケージ管理していましたが、Amazon Linux 2023から`dnf`コマンドに切り替わっています。**  
参照[Amazon Linux 2023 での WordPress ブログのホスト](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/hosting-wordpress-aml-2023.html)

- 以前にAWS マネコンを利用してWordPress環境構築をしたことがありますが、以下のデメリットを感じてCloudFormationで構築することにしました。
  - セキュリティグループ設定、ACLネットワーク設定、ルーティング設定が複雑になってくると関連性が分からなくなってくる
  - 何度もSSH接続をしなければならない
  - 一度リソースを削除すると、再度構築する場合、手間と時間がかかる
  - 丁寧に手打ちしても、ミスをするリスクがある


## アーキテクチャ図
![simple-wordpress drawio](https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/c054dc3c-9ca2-4b2b-bb37-3f514d31e958)
- EC2へSSH接続ができるようにして、実際のWordPressへのアクセスはALB経由HTTP通信する。  
- **※セキュリティ面に関して問題はありますが、無料枠での構築のためご承知ください。**  
- ALBは今後、Multi-AZ構成を組むと想定した場合に必要になるので設置しています。


## 前提条件
- AWS IAMにて適切なユーザーを作成済み
- EC2へSSH接続するためのキーペアを事前に準備

### CloudFormationからスタックを作成  
７分ほど待つと、リソースの作成が以下のように完了しています。  
<img width="556" alt="スクリーンショット 2023-11-21 10 56 17" src="https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/3c4c1c69-0367-4335-8e95-60e46e2916d2">  


Parametersは以下のようになっております。  
<img width="459" alt="スクリーンショット 2023-11-21 11 23 11" src="https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/58b341ca-23e7-48a4-af75-4cf6fca2431a">

### 動作確認
AWS マネコン > EC2サービスにアクセス > ロードバランサーを選択します。  
DNS名をコピーして、実際にApache、PHP、WordPressがインストールされているか動作確認をします。  
<img width="687" alt="スクリーンショット 2023-11-21 10 53 20" src="https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/429489cf-f471-45d9-9c23-95ea889d2cee">  

ブラウザにDNS名を貼り付けて確認します。  
<img width="536" alt="スクリーンショット 2023-11-21 11 34 20" src="https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/521006a1-0cd1-4241-a7a5-01fd0563e454">

WordPressの画面が表示され、その後、画面の指示に従いWordPressのインストールが完了しました。  

----------


### EC2へSSH接続をして動作確認
問題なくSSH接続が出来ているようです。    
![スクリーンショット 2023-11-21 11 39 34](https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/cb3101d1-9b86-4a23-9b8f-8f2445274158)

`$ sudo systemctl status httpd.service`でも確認してみます。  
![スクリーンショット 2023-11-21 11 44 42](https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/01646455-f100-4b52-b9b5-15eb29154db4)  

`cd /var/www/html/`で/var/www/html/ディレクトリに移動し、中身を確認します。  
`ll`で中身を拝見・・・  
![スクリーンショット 2023-11-21 11 46 46](https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/aaca504b-a3c5-41f3-9984-2742a16e7677)  
CloudFormationテンプレの中にある、UserDataで指定したスクリプトが上手くいっているようです。

最後に、`$ exit`でSSH接続解除します。お疲れさまでした。

### つまづいた部分
- 今回はAmazon Linux 2023を使用しており、MySQLバージョンが合っておらず、最新のバージョンを確認して解決  
 ![スクリーンショット 2023-11-20 16 38 53](https://github.com/Kana-Karin/aws-linux2023-wordpress-cloudformation/assets/84316229/97d05b7d-b5e0-493d-96bc-11827abb7781)
- CloudFormationを作る際の情報がAmazon Linux 2のものだったり、スクリプトが古く、パッケージ自体間違っていたりして何度も詰みましたが、公式リファレンスで解決できました

### テンプレ化や自動化の便利さについて
改めて、便利だなぁと思いました。勿論、マネコンで構築するのも素敵ですが、CloudFormationは再利用性やyml形式で見易く、  
CloudFormationのリファレンスさえ理解してしまえばとても便利なものだと思います。  

### 参照させていただいたサイト
- [Amazon Linux 2023 での WordPress ブログのホスト](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/hosting-wordpress-aml-2023.html)
- [Amazon Linux 2023 – 長期サポート付きのクラウド最適化 Linux ディストリビューション](https://aws.amazon.com/jp/blogs/news/amazon-linux-2023-a-cloud-optimized-linux-distribution-with-long-term-support/)
- [Qiita - 5分で理解するAmazon Linux 2023](https://qiita.com/Brutus/items/a2ac164297677edaa6e2)





