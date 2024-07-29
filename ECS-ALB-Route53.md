# ECS(Fargate)にALBを追加してRoute 53でSSL対応する手順

## 前提条件
- AWS CLIが設定されている
- ECSクラスタとサービスが作成済み
- Route 53でドメインを管理している

## 手順

### 1. ALBの作成
- [ ] AWS Management Consoleにログイン
- [ ] EC2ダッシュボードから「ロードバランサー」を選択
- [ ] 「ロードバランサーを作成」をクリック
- [ ] Application Load Balancerを選択
- [ ] 名前、スキーム（インターネット向け）、リスナー（HTTPS）、VPC、アベイラビリティゾーンを設定

### 2. SSL証明書の取得
- [ ] AWS Management ConsoleでACM（AWS Certificate Manager）に移動
- [ ] 「証明書のリクエスト」をクリック
- [ ] ドメイン名を入力し、DNS検証を選択
- [ ] ACMが提供するDNSレコードをRoute 53に追加
- [ ] 証明書の発行を確認

### 3. ALBにSSL証明書を設定
- [ ] EC2ダッシュボードで作成したALBを選択
- [ ] 「リスナー」タブをクリック
- [ ] HTTPSリスナーを編集し、ACMから取得したSSL証明書を選択

### 4. ターゲットグループの作成
- [ ] EC2ダッシュボードの「ターゲットグループ」を選択
- [ ] 「ターゲットグループを作成」をクリック
- [ ] 名前、ターゲットタイプ（ip）、プロトコル（HTTP）、ポートを設定
- [ ] 「次のステップ」をクリックし、ターゲットとしてECSタスクを追加

### 5. ECSサービスの更新
- [ ] AWS CLIを使用してECSサービスにALBを追加
    ```bash
    aws ecs update-service \
      --cluster your-cluster-name \
      --service your-service-name \
      --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:region:account-id:targetgroup/your-target-group/1234567890123456,containerName=your-container-name,containerPort=80
    ```

### 6. Route 53でDNSレコードを設定
- [ ] Route 53コンソールでホストゾーンを選択
- [ ] AレコードまたはCNAMEレコードを作成し、ALBのDNS名をターゲットに設定

### 7. セキュリティグループの設定
- [ ] ALBのセキュリティグループを設定し、ポート443（HTTPS）からのインバウンドトラフィックを許可
- [ ] ECSタスクのセキュリティグループを設定し、ALBからのトラフィックを許可

### 8. 確認とテスト
- [ ] ブラウザでドメインを開き、HTTPS接続が成功することを確認

## 参考資料
- [既存のECS(Fargate)にALBを追加する方法](https://dev.classmethod.jp/articles/add-alb-to-existing-ecs-fargate/)
- [AWS公式ドキュメント](https://docs.aws.amazon.com/)