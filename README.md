# trying out terraform with atlas and github

terraformのインテグレーションをatlasとgithubで試した記録

## github

githubと連携させるための設定

```
cd trying_out_terraform_with_atlas
git init .
git add .
git commit -m '1st commit'
```

githubにリポジトリ作っておく

```
git remote add origin git@github.com:Lorentzca/trying_out_terraform_with_atlas.git
git push -u origin master
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

terraformのバックエンドをs3にしてみる

- [terraformの状態管理をatlasに指定する](https://github.com/Lorentzca/trying_out_terraform_with_atlas#terraformの状態管理をatlasに指定する)から分岐

```
terraform remote config -backend=S3 -backend-config="bucket=terraform-20160222" -backend-config="key=terraform.tfstate"
```

当然この時点でまだatrasに管理ページは作られない  
新規にenvironmentを作成する

![](./images/environment1.png)

![](./images/environment2.png)

この時点でgithubの接続と、terraformのatlasへのリンクが完了する

- `terraform remote config -backend-config "name=<atlas-uername>/trying_out_terraform_with_atlas"`した場合最初からリンクされないのは、atlasをterraformのバックエンドとしてしか指定していないから？

後は同じように環境変数を設定しておく

apply後設定を変更してみると…

- ローカル(s3参照)だと差分があるが

```
$ terraform plan
Refreshing Terraform state prior to plan...

aws_vpc.main: Refreshing state... (ID: vpc-01e89a64)

The Terraform execution plan has been generated and is shown below.
Resources are shown in alphabetical order for quick scanning. Green resources
will be created (or destroyed and then created if an existing resource
exists), yellow resources are being changed in-place, and red resources
will be destroyed.

Note: You didn't specify an "-out" parameter to save this plan, so when
"apply" is called, Terraform can't guarantee this is what will execute.

-/+ aws_vpc.main
    cidr_block:                "10.0.0.0/28" => "10.0.0.0/16" (forces new resource)
    default_network_acl_id:    "acl-f976219c" => "<computed>"
    default_security_group_id: "sg-21bb4445" => "<computed>"
    dhcp_options_id:           "dopt-48aaa22a" => "<computed>"
    enable_dns_hostnames:      "" => "<computed>"
    enable_dns_support:        "" => "<computed>"
    main_route_table_id:       "rtb-d4a2f0b1" => "<computed>"
    tags.#:                    "1" => "1"
    tags.Name:                 "trying_out_terraform_with_atlas" => "trying_out_terraform_with_atlas"


Plan: 1 to add, 0 to change, 1 to destroy.
```
- atlas上では新しく作成したように見えている(= s3の情報見えていない)

![](./images/plan1.png)
