# 10分で作れる！EC2に構成するWordPress環境

## 概要

AWS 上に WordPress 実行環境を自動構築するポートフォリオです。  
Terraform による IaC（Infrastructure as Code）と Ansible による構成管理を組み合わせ、GitHub Actions から自動デプロイを実現しています。

インフラ構築からミドルウェア設定、WordPress 配置までをコード化し、短時間で再現可能な WordPress 環境を構築できます。

---

## 構成図

```text
GitHub
   │
   │ push / pull request
   ▼
GitHub Actions
   │
   ├── Terraform
   │      └── AWS リソース作成
   │             └── EC2
   │
   └── Ansible
          └── EC2 に対して構成管理
                 ├── Apache
                 ├── PHP
                 ├── MySQL
                 └── WordPress
```

---

# 使用技術

| 分類 | 技術 |
|---|---|
| IaC | Terraform |
| Configuration Management | Ansible |
| CI/CD | GitHub Actions |
| Cloud | AWS |
| OS | Amazon Linux 2023 |
| Web Server | Apache |
| Database | MySQL 8.0 |
| Language | PHP 8.3 |
| CMS | WordPress |
| 接続方式 | AWS Systems Manager (SSM) |

---

# アーキテクチャ

## LAMP構成

本環境は Linux / Apache / MySQL / PHP を組み合わせた、いわゆる **LAMP 構成** を採用しています。

| 項目 | 役割 |
|---|---|
| Linux | OS |
| Apache | Webサーバー |
| MySQL | データベース |
| PHP | アプリケーション実行 |

WordPress は PHP 製のアプリケーションであり、データ保存先として MySQL を利用するため、LAMP 構成と非常に相性が良いです。

---

# ディレクトリ構成

```text
.
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── ansible/
│   ├── playbook.yaml
│   └── roles/
│       ├── apache/
│       ├── php/
│       ├── mysql/
│       └── wordpress/
│
└── .github/
    └── workflows/
        └── terraform.yaml
```

---

# CI/CD フロー

## Pull Request 時

Terraform の静的チェックと plan を実行。

```yaml
terraform fmt -check -recursive
terraform validate
terraform plan
```

---

## main ブランチマージ時

Terraform Apply → Ansible 実行までを自動化。

```text
Terraform Apply
      ↓
EC2 作成
      ↓
Ansible 実行
      ↓
WordPress 構築完了
```

---

# 工夫したポイント

## 1. SSH を使わず SSM 接続

EC2 への接続は SSH ではなく AWS Systems Manager Session Manager を利用。

### メリット

- SSH ポート不要
- 鍵管理不要
- セキュリティ向上
- GitHub Actions から安全に接続可能

---

## 2. Terraform と Ansible の責務分離

| ツール | 役割 |
|---|---|
| Terraform | AWS リソース作成 |
| Ansible | OS / ミドルウェア設定 |

IaC と構成管理を分離し、保守性を高めています。

---

## 3. GitHub Actions による完全自動化

GitHub Push のみで以下を自動実行。

- Terraform init
- Terraform plan
- Terraform apply
- Ansible Playbook 実行
- WordPress 構築

---

# 実装内容

## Apache

- Apache インストール
- 自動起動設定
- HTTP(80) 開放

---

## PHP

- PHP 8.3
- php-fpm
- php-mysqlnd
- php-mbstring

### セキュリティ設定

```ini
expose_php = Off
date.timezone = Asia/Tokyo
```

---

## MySQL

- MySQL 8.0
- root パスワード変更
- test DB 削除
- anonymous user 削除
- WordPress DB 作成

---

## WordPress

- 最新版 WordPress ダウンロード
- wp-config.php 自動生成
- DB 情報自動設定
- Apache 配下へ配置

---

# 実際の画面

## Apache 動作確認

```text
It works!
```

---

## WordPress 初期画面

```text
ようこそ WordPress へ
```

---

# 今後の改善予定

- HTTPS 化（Let's Encrypt）
- Route53 + 独自ドメイン
- RDS 化
- ALB 導入
- Auto Scaling
- Docker 化
- ECS / Kubernetes 化
- WAF 導入
- CloudFront 導入

---

# 学んだこと

- Terraform と Ansible の役割分担
- GitHub Actions による CI/CD
- AWS SSM 接続
- Linux ミドルウェア構築
- LAMP 構成
- WordPress 動作原理

---

# 苦労した点

- SSM 接続待機
- MySQL 初期パスワード変更
- MySQL ポリシーエラー
- Apache / PHP-FPM 連携
- WordPress ファイル配置
- Terraform Backend 初期化

---

# まとめ

Terraform・Ansible・GitHub Actions を組み合わせることで、  
WordPress 環境を短時間で自動構築できるようになりました。

インフラ構築だけでなく、構成管理・CI/CD・セキュリティを含めた実践的な AWS 運用を学べたプロジェクトです。
