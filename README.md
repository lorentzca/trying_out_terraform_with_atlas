# trying out terraform with atlas

terraformのインテグレーションをatlasで試す

## git

```
cd trying_out_terraform_with_atlas
git init .
git add .
git commit -m '1st commit'
```

## atlasトークンを取得する

<https://atlas.hashicorp.com/settings/tokens>

![](./images/create_atlas_token.png)


## 必要な環境変数の設定

- ATLAS_TOKENは必須
- ほかはterraformで使うプロバイダによって適宜変える

```bash
export ATLAS_TOKEN="1a2b3c4d"
export AWS_ACCESS_KEY_ID="1234abcd"
export AWS_SECRET_ACCESS_KEY="1234ABCD"
export AWS_DEFAULT_REGION="ap-northeast-1"
```

## terraformの状態管理をatlasに指定する

```bash
terraform remote config -backend-config "name=<atlas-uername>/trying_out_terraform_with_atlas"
```

atlasに管理ページが作成される

![](./images/created_page1.png)

![](./images/created_page2.png)

## integrationsの設定をする

### atlas側の設定をする

<https://atlas.hashicorp.com/lorentzca/environments/trying_out_terraform_with_atlas/settings>

- terraformのplan結果に何かしら機密情報が含ま得れる場合もあるかもしれない(RDSのパスワードとか)のでプライベートにしておいたほうが良さそう(デフォルトでプライベートになっている)
- terraformのバージョン指定もしておく

![](./images/integrations1.png)

- 案内にある通りatlasの環境とterraformをリンクさせる(上の画像の続きにある)

![](./images/integrations2.png)

```
terraform push -name <atlas-uername>/trying_out_terraform_with_atlas
```

### githubのリポジトリに接続する

<https://atlas.hashicorp.com/lorentzca/environments/trying_out_terraform_with_atlas/integrations>

- terraform directoryはtfファイルがリポジトリのroot以下に無い場合(`terraform/main.tf`とか)指定する

![](./images/integrations3.png)


## 適当に何かコミットする

atlasでplanが実行される

![](./images/pr1.png)

環境変数が足りなくて失敗！

![](./images/pr2.png)

ステータスがPRに反映される

![](./images/pr3.png)

### Atlasに環境変数の設定

variablesの設定から行う

![](./images/variables1.png)

諸々必要な環境変数を埋めた

![](./images/variables2.png)

terraform.tfvarsを使っている場合はterraform pushしたら自動で変数が使えるようになるっぽい

### 再度コミット

plan成功

![](./images/pr4.png)

PRのステータスもグリーンになった

![](./images/pr5.png)

## メモ

### terraformの状態管理をatlas以外にした場合
