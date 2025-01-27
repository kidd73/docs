---
title: Xcode 開発用の macOS ランナーに Apple 証明書をインストールする
intro: '{% data variables.product.prodname_actions %} ランナーに Apple コード署名証明書をインストールすることで、継続的インテグレーション (CI) ワークフロー内で Xcode アプリケーションに署名できます。'
redirect_from:
  - /actions/guides/installing-an-apple-certificate-on-macos-runners-for-xcode-development
  - /actions/deployment/installing-an-apple-certificate-on-macos-runners-for-xcode-development
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - CI
  - Xcode
shortTitle: Sign Xcode applications
ms.openlocfilehash: 47c534db1e16595af4735362c524f673376b53fe
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2022
ms.locfileid: '145090447'
---
{% data reusables.actions.enterprise-beta %} {% data reusables.actions.enterprise-github-hosted-runners %}

## はじめに

このガイドでは、Apple コード署名証明書とプロビジョニングプロファイルを {% data variables.product.prodname_actions %} ランナーにインストールする継続的インテグレーション (CI) ワークフローにステップを追加する方法を説明しています。 これにより、Xcode アプリケーションに署名して、Apple App Store に公開したり、テストグループに配布したりできるようになります。

## 前提条件

YAMLと{% data variables.product.prodname_actions %}の構文に馴染んでいる必要があります。 詳細については、次を参照してください。

- "[{% data variables.product.prodname_actions %} について](/actions/learn-github-actions)"
- [{% data variables.product.prodname_actions %} のワークフロー構文](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)

Xcode アプリケーションのビルドと署名について理解しておく必要があります。 詳細については、[Apple の開発者向けドキュメント](https://developer.apple.com/documentation/)を参照してください。

## 証明書とプロビジョニングプロファイルのシークレットを作成する

署名プロセスには、証明書とプロビジョニングプロファイルの保存、それらのランナーへの転送、ランナーのキーチェーンへのインポート、およびビルドでの使用が含まれます。

ランナーで証明書とプロビジョニングプロファイルを使用するには、{% data variables.product.prodname_dotcom %} シークレットを使用することを強くお勧めします。 シークレットの作成とワークフローでそれらを使用する詳細については、「[暗号化されたシークレット](/actions/reference/encrypted-secrets)」を参照してください。

次のアイテムのシークレットをリポジトリまたは Organization に作成します。

* Apple の署名証明書。

  - これは `p12` 証明書ファイルです。 Xcode から署名証明書をエクスポートする方法の詳細については、[Xcode のドキュメント](https://help.apple.com/xcode/mac/current/#/dev154b28f09)を参照してください。
  
  - 証明書をシークレットとして保存する場合は、証明書を Base64 に変換する必要があります。 この例では、シークレットの名前は `BUILD_CERTIFICATE_BASE64` です。

  - 次のコマンドを使用して、証明書を Base64 に変換し、クリップボードにコピーします。

    ```shell
    base64 <em>build_certificate</em>.p12 | pbcopy
    ```
* Apple 署名証明書のパスワード。
  - この例では、シークレットの名前は `P12_PASSWORD` です。

* Apple プロビジョニングプロファイル。

  - Xcode からプロビジョニングプロファイルをエクスポートする方法の詳細については、[Xcode のドキュメント](https://help.apple.com/xcode/mac/current/#/deva899b4fe5)を参照してください。

  - シークレットとして保存する場合は、プロビジョニングプロファイルを Base64 に変換する必要があります。 この例では、シークレットの名前は `BUILD_PROVISION_PROFILE_BASE64` です。

  - 次のコマンドを使用して、プロビジョニングプロファイルを Base64 に変換し、クリップボードにコピーします。
  
    ```shell
    base64 <em>provisioning_profile.mobileprovision</em> | pbcopy
    ```

* キーチェーンのパスワード。

  - ランナー上に新しいキーチェーンが作成されるため、新しいキーチェーンのパスワードは任意の新しいランダムな文字列にすることができます。 この例では、シークレットの名前は `KEYCHAIN_PASSWORD` です。

## ワークフローにステップを追加する

このワークフロー例には、{% data variables.product.prodname_dotcom %} シークレットから Apple 証明書とプロビジョニングプロファイルをインポートし、それらをランナーにインストールするステップが含まれています。

```yaml{:copy}
name: App build
on: push

jobs:
  build_with_signing:
    runs-on: macos-latest

    steps:
      - name: Checkout repository
        uses: {% data reusables.actions.action-checkout %}
      - name: Install the Apple certificate and provisioning profile
        env:
          BUILD_CERTIFICATE_BASE64: {% raw %}${{ secrets.BUILD_CERTIFICATE_BASE64 }}{% endraw %}
          P12_PASSWORD: {% raw %}${{ secrets.P12_PASSWORD }}{% endraw %}
          BUILD_PROVISION_PROFILE_BASE64: {% raw %}${{ secrets.BUILD_PROVISION_PROFILE_BASE64 }}{% endraw %}
          KEYCHAIN_PASSWORD: {% raw %}${{ secrets.KEYCHAIN_PASSWORD }}{% endraw %}
        run: |
          # create variables
          CERTIFICATE_PATH=$RUNNER_TEMP/build_certificate.p12
          PP_PATH=$RUNNER_TEMP/build_pp.mobileprovision
          KEYCHAIN_PATH=$RUNNER_TEMP/app-signing.keychain-db

          # import certificate and provisioning profile from secrets
          echo -n "$BUILD_CERTIFICATE_BASE64" | base64 --decode --output $CERTIFICATE_PATH
          echo -n "$BUILD_PROVISION_PROFILE_BASE64" | base64 --decode --output $PP_PATH

          # create temporary keychain
          security create-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH
          security set-keychain-settings -lut 21600 $KEYCHAIN_PATH
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN_PATH

          # import certificate to keychain
          security import $CERTIFICATE_PATH -P "$P12_PASSWORD" -A -t cert -f pkcs12 -k $KEYCHAIN_PATH
          security list-keychain -d user -s $KEYCHAIN_PATH

          # apply provisioning profile
          mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
          cp $PP_PATH ~/Library/MobileDevice/Provisioning\ Profiles
      - name: Build app
        ...
```

## セルフホストランナーに必要なクリーンアップ

{% data variables.product.prodname_dotcom %} ホストランナーは、ジョブの実行の最後に自動的に破棄される分離された仮想マシンです。 これは、ジョブ中にランナーで使用された証明書とプロビジョニングプロファイルが、ジョブの完了時にランナーとともに破棄されることを意味します。

セルフホステッド ランナーでは、ジョブの実行の最後に `$RUNNER_TEMP` ディレクトリがクリーンアップされますが、キーチェーンとプロビジョニング プロファイルがランナーにまだ存在している可能性があります。

セルフホストランナーを使用する場合は、ワークフローに最終ステップを追加して、ジョブの最後にこれらの機密ファイルが確実に削除されるようにする必要があります。 以下のワークフローステップは、これを行う方法の例です。

{% raw %}
```yaml
- name: Clean up keychain and provisioning profile
  if: ${{ always() }}
  run: |
    security delete-keychain $RUNNER_TEMP/app-signing.keychain-db
    rm ~/Library/MobileDevice/Provisioning\ Profiles/build_pp.mobileprovision
```
{% endraw %}
