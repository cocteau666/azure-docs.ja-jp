---
title: 'P2S VPN と証明書認証を使用して VNet に接続する: ポータル'
titleSuffix: Azure VPN Gateway
description: P2S と自己署名証明書または CA によって発行された証明書を使用して、Windows、macOS、Linux の各クライアントを Azure 仮想ネットワークに安全に接続します。 この記事では Azure Portal を使用します。
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/09/2020
ms.author: cherylmc
ms.openlocfilehash: 0b2fa06bc04bdb584367312b1e89939ed386b4f2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/20/2020
ms.locfileid: "94952850"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-native-azure-certificate-authentication-azure-portal"></a>ネイティブ Azure 証明書認証を使用した VNet へのポイント対サイト VPN 接続の構成:Azure portal

この記事では、Windows、Linux、または macOS が実行されている個々のクライアントを Azure VNet に安全に接続する方法を紹介します。 ポイント対サイト VPN 接続は、自宅や会議室でのテレワークなど、リモートの場所から VNet に接続する場合に便利です。 VNet への接続を必要とするクライアントがごく少ない場合は、サイト対サイト VPN の代わりに P2S を使用することもできます。 ポイント対サイト接続に、VPN デバイスや公開 IP アドレスは必要ありません。 P2S により、SSTP (Secure Socket トンネリング プロトコル) または IKEv2 経由の VPN 接続が作成されます。 ポイント対サイト VPN について詳しくは、「[ポイント対サイト VPN について](point-to-site-about.md)」を参照してください。

:::image type="content" source="./media\vpn-gateway-howto-point-to-site-resource-manager-portal/point-to-site-diagram.png" alt-text="コンピューターから Azure VNet への接続 - ポイント対サイト接続の図":::

ポイント対サイト VPN の詳細については、「[ポイント対サイト VPN について](point-to-site-about.md)」を参照してください。 Azure PowerShell を使用してこの構成を作成するには、[Azure PowerShell を使用したポイント対サイト VPN の構成](vpn-gateway-howto-point-to-site-rm-ps.md)に関する記事を参照してください。

[!INCLUDE [P2S basic architecture](../../includes/vpn-gateway-p2s-architecture.md)]

## <a name="prerequisites"></a>前提条件

Azure サブスクリプションを持っていることを確認します。 Azure サブスクリプションをまだお持ちでない場合は、[MSDN サブスクライバーの特典](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details)を有効にするか、[無料アカウント](https://azure.microsoft.com/pricing/free-trial)にサインアップしてください。

### <a name="example-values"></a><a name="example"></a>値の例

次の値を使用して、テスト環境を作成できます。また、この値を参考にしながら、この記事の例を確認していくこともできます。

* **VNet 名:** VNet1
* **[アドレス空間]:** 10.1.0.0/16<br>この例では、1 つのアドレス空間のみを使用します。 VNet には、複数のアドレス空間を使用することができます。
* **サブネット名:** FrontEnd
* **サブネットのアドレス範囲:** 10.1.0.0/24
* **サブスクリプション:** サブスクリプションが複数ある場合は、適切なサブスクリプションを使用していることを確認します。
* **[リソース グループ]:** TestRG1
* **[場所]:** 米国東部
* **GatewaySubnet:** 10.1.255.0/27<br>
* **仮想ネットワーク ゲートウェイ名:** VNet1GW
* **ゲートウェイの種類:** VPN
* **VPN の種類:** ルート ベース
* **パブリック IP アドレス名:** VNet1GWpip
* **[接続の種類]** : ポイント対サイト
* **クライアント アドレス プール:** 172.16.201.0/24<br>このポイント対サイト接続を利用して VNet に接続する VPN クライアントは、クライアント アドレス プールから IP アドレスを受け取ります。

## <a name="1-create-a-virtual-network"></a><a name="createvnet"></a>1.仮想ネットワークの作成

最初に Azure サブスクリプションを持っていることを確認します。 Azure サブスクリプションをまだお持ちでない場合は、[MSDN サブスクライバーの特典](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details)を有効にするか、[無料アカウント](https://azure.microsoft.com/pricing/free-trial)にサインアップしてください。

[!INCLUDE [About cross-premises addresses](../../includes/vpn-gateway-cross-premises.md)]

[!INCLUDE [Basic Point-to-Site VNet](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="2-create-a-virtual-network-gateway"></a><a name="creategw"></a>2.仮想ネットワーク ゲートウェイの作成

この手順では、VNet の仮想ネットワーク ゲートウェイを作成します。 選択したゲートウェイ SKU によっては、ゲートウェイの作成に 45 分以上かかる場合も少なくありません。

>[!NOTE]
>Basic ゲートウェイ SKU では、IKEv2 と RADIUS 認証はサポートされません。 Mac クライアントを仮想ネットワークに接続する予定がある場合は、Basic SKU を使用しないでください。
>

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

[!INCLUDE [Create a gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## <a name="3-generate-certificates"></a><a name="generatecert"></a>3.証明書の生成

証明書は、ポイント対サイト VPN 接続を介して VNet に接続するクライアントを認証するために、Azure によって使用されます。 ルート証明書を取得したら、公開キー情報を Azure に[アップロード](#uploadfile)します。 ルート証明書は、Azure によって "信頼された" と見なされ、P2S 経由での仮想ネットワークへの接続に使用されます。 また、信頼されたルート証明書からクライアント証明書を生成し、それを各クライアント コンピューターにインストールします。 クライアント証明書は、クライアントで VNet への接続を開始するときに、そのクライアントを認証するために使用されます。 

### <a name="generate-a-root-certificate"></a><a name="getcer"></a>ルート証明書を生成する

[!INCLUDE [root-certificate](../../includes/vpn-gateway-p2s-rootcert-include.md)]

### <a name="generate-client-certificates"></a><a name="generateclientcert"></a>クライアント証明書を生成する

[!INCLUDE [generate-client-cert](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="4-add-the-client-address-pool"></a><a name="addresspool"></a>4.クライアント アドレス プールの追加

クライアント アドレス プールとは、指定するプライベート IP アドレスの範囲です。 ポイント対サイト VPN 経由で接続するクライアントは、この範囲内の IP アドレスを動的に受け取ります。 接続元であるオンプレミスの場所、または接続先とする VNet と重複しないプライベート IP アドレス範囲を使用してください。 複数のプロトコルを構成するとき、SSTP がプロトコルの 1 つの場合、構成後のアドレス プールは構成されるプロトコル間で均等に分割されます。

1. 仮想ネットワーク ゲートウェイが作成されたら、[仮想ネットワーク ゲートウェイ] ページの **[設定]** セクションに移動します。 **[設定]** で、 **[ポイント対サイトの構成]** を選択します。 **[今すぐ構成]** を選択して、構成ページを開きます。

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/configure-now.png" alt-text="[ポイント対サイトの構成] ページ" lightbox="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/configure-now.png":::
1. **[ポイント対サイトの構成]** ページでは、さまざまな設定を構成できます。 このページにトンネルの種類または認証の種類が表示されない場合、ゲートウェイでは Basic SKU が使用されています。 Basic SKU では、IKEv2 と RADIUS 認証はサポートされません。 これらの設定を使用する場合は、ゲートウェイを削除し、別のゲートウェイ SKU を使って再作成する必要があります。

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/address-pool.png" alt-text="アドレス プールを指定する" lightbox="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/address-pool.png":::
1. **[アドレス プール]** ボックスに、使用するプライベート IP アドレス範囲を追加します。 VPN クライアントには、指定した範囲から動的に IP アドレスが割り当てられます。 最小のサブネット マスクは、アクティブ/パッシブ構成の場合は 29 ビット、アクティブ/アクティブ構成の場合は 28 ビットです。
1. 次のセクションに進み、トンネルの種類を構成します。

## <a name="5-configure-tunnel-type"></a><a name="tunneltype"></a>5.トンネルの種類の構成

トンネルの種類を選択します。 トンネルのオプションには、OpenVPN、SSTP、IKEv2 があります。

* Android と Linux の strongSwan クライアントおよび iOS と OSX のネイティブ IKEv2 VPN クライアントでは、接続に IKEv2 トンネルのみを使用します。
* Windows クライアントでは最初に IKEv2 を試し、接続できなかった場合に SSTP にフォールバックします。
* OpenVPN クライアントを使用して、OpenVPN トンネルの種類に接続することができます。

:::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/tunnel-ike.png" alt-text="トンネルの種類":::

## <a name="6-configure-authentication-type"></a><a name="authenticationtype"></a>6.認証の種類の構成

**[認証の種類]** で **[Azure 証明書]** を選択します。

:::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/azure-certificate.png" alt-text="認証の種類":::

## <a name="7-upload-the-root-certificate-public-certificate-data"></a><a name="uploadfile"></a>7.ルート証明書の公開証明書データのアップロード

その後は、最大で合計 20 個まで、信頼されたルート証明書を追加でアップロードできます。 公開証明書データがアップロードされたら、Azure でそれを使用し、信頼されたルート証明書から生成されたクライアント証明書がインストールされているクライアントを認証できます。 ルート証明書の公開キー情報を Azure にアップロードします。

1. 証明書は、 **[ルート証明書]** セクションの **[ポイント対サイトの構成]** ページで追加します。
1. ルート証明書を Base 64 でエンコードされた X.509 (.cer) ファイルとしてエクスポートしたことを確認してください。 証明書をテキスト エディターで開くことができるように、この形式でエクスポートする必要があります。
1. 証明書をメモ帳などのテキスト エディターで開きます。 証明書データをコピーするときはに、必ず、テキストを復帰や改行のない 1 つの連続した行としてコピーしてください。 復帰や改行を確認するには、テキスト エディターのビューを "記号を表示する/すべての文字を表示する" ように変更することが必要になる場合があります。 次のセクションのみを 1 つの連続した行としてコピーします。

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/notepadroot.png" alt-text="証明書データ" border="false":::
1. **[公開証明書データ]** フィールドに証明書データを貼り付けます。 証明書に **名前を付けて**、 **[保存]** を選択します。 最大 20 個の信頼されたルート証明書を追加できます。

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/uploaded.png" alt-text="証明書データを貼り付ける" border="false":::
1. ページの上部にある **[保存]** を選択して、構成設定をすべて保存します。

   :::image type="content" source="./media/vpn-gateway-howto-point-to-site-resource-manager-portal/save.png" alt-text="構成を保存する" border="false":::

## <a name="8-install-an-exported-client-certificate"></a><a name="installclientcert"></a>8.エクスポートしたクライアント証明書のインストール

クライアント証明書の生成に使用したクライアント コンピューター以外から P2S 接続を作成する場合は、クライアント証明書をインストールする必要があります。 クライアント証明書をインストールするときに、クライアント証明書のエクスポート時に作成されたパスワードが必要になります。

クライアント証明書が証明書チェーン全体と一緒に .pfx としてエクスポートされている (既定値) ことを確認します。 そうでないと、ルート証明書情報がクライアント コンピューターに存在せず、クライアントは正しく認証されません。

インストールの手順については、[クライアント証明書のインストール](point-to-site-how-to-vpn-client-install-azure-cert.md)に関するページを参照してください。

## <a name="9-generate-and-install-the-vpn-client-configuration-package"></a><a name="clientconfig"></a>9.VPN クライアント構成パッケージの生成とインストール

VPN クライアント構成ファイルには、P2S 接続を使って VNet に接続できるようにデバイスを構成するための設定が含まれています。 VPN クライアント構成ファイルの生成とインストールに関する手順については、「[ネイティブ Azure 証明書認証の P2S 構成のための VPN クライアント構成ファイルを作成およびインストールする](point-to-site-vpn-client-configuration-azure-cert.md)」を参照してください。

## <a name="10-connect-to-azure"></a><a name="connect"></a>10.Azure に接続する

### <a name="to-connect-from-a-windows-vpn-client"></a>Windows VPN クライアントから接続するには

[!INCLUDE [Connect from a Windows client](../../includes/vpn-gateway-p2s-connect-windows-client.md)]

[!INCLUDE [verifies client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="to-connect-from-a-mac-vpn-client"></a>Mac の VPN クライアントから接続するには

[ネットワーク] ダイアログ ボックスで、使用するクライアント プロファイルを検索し、[VpnSettings.xml](point-to-site-vpn-client-configuration-azure-cert.md#installmac) の設定を指定して、 **[接続]** を選択します。

手順の詳細については、[Mac (OS X) のインストール](./point-to-site-vpn-client-configuration-azure-cert.md#installmac)に関するセクションを参照してください。 接続に問題がある場合は、仮想ネットワーク ゲートウェイが Basic SKU を使用していないことを確認します。 Basic SKU は Mac クライアントではサポートされていません。

:::image type="content" source="./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png" alt-text="Mac VPN クライアント接続" border="false":::

## <a name="to-verify-your-connection"></a><a name="verify"></a>接続を確認するには

ここで紹介する手順は、Windows クライアントに適用されます。

1. VPN 接続がアクティブであることを確認するには、管理者特権でのコマンド プロンプトを開いて、 *ipconfig/all* を実行します。
2. 結果を表示します。 受信した IP アドレスが、構成に指定したポイント対サイト VPN クライアント アドレス プール内のアドレスのいずれかであることに注意してください。 結果は次の例のようになります。

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>仮想マシンに接続するには

ここで紹介する手順は、Windows クライアントに適用されます。

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* VNet に対して DNS サーバーの IP アドレスが指定された後に VPN クライアント構成パッケージが生成されたことを確認します。 DNS サーバーの IP アドレスを更新した場合は、新しい VPN クライアント構成パッケージを生成してインストールしてください。

* 接続元のコンピューターのイーサネット アダプターに割り当てられている IPv4 アドレスを "ipconfig" でチェックします。 その IP アドレスが、接続先の VNet のアドレス範囲内または VPNClientAddressPool のアドレス範囲内にある場合、これを "アドレス空間の重複" といいます。 アドレス空間がこのように重複していると、ネットワーク トラフィックが Azure に到達せずローカル ネットワーク上に留まることになります。

## <a name="to-add-or-remove-trusted-root-certificates"></a><a name="add"></a>信頼されたルート証明書を追加または削除するには

信頼されたルート証明書を Azure に追加したり、Azure から削除したりできます。 ルート証明書を削除すると、そのルートから証明書を生成したクライアントは認証が無効となり、接続できなくなります。 クライアントの認証と接続を正常に実行できるようにするには、Azure に信頼されている (Azure にアップロードされている) ルート証明書から生成した新しいクライアント証明書をインストールする必要があります。

### <a name="to-add-a-trusted-root-certificate"></a>信頼されたルート証明書を追加するには

信頼されたルート証明書 .cer ファイルを最大 20 個まで Azure に追加できます。 手順については、この記事の[信頼されたルート証明書のアップロード](#uploadfile)に関するセクションを参照してください。

### <a name="to-remove-a-trusted-root-certificate"></a>信頼されたルート証明書を削除するには

1. 信頼されたルート証明書を削除するには、仮想ネットワーク ゲートウェイの **[ポイント対サイトの構成]** ページに移動します。
1. ページの **[ルート証明書]** セクションで、削除する証明書を見つけます。
1. 証明書の横にある省略記号を選び、[削除] を選択します。

## <a name="to-revoke-a-client-certificate"></a><a name="revokeclient"></a>クライアント証明書を失効させるには

クライアント証明書は失効させることができます。 証明書失効リストを使用すると、個々のクライアント証明書に基づくポイント対サイト接続を選択して拒否することができます。 これは、信頼されたルート証明書を削除することとは異なります。 信頼されたルート証明書 .cer を Azure から削除すると、失効したルート証明書によって生成または署名されたすべてのクライアント証明書のアクセス権が取り消されます。 ルート証明書ではなくクライアント証明書を失効させることで、ルート証明書から生成されたその他の証明書は、その後も認証に使用できます。

一般的な方法としては、ルート証明書を使用してチームまたは組織レベルでアクセスを管理し、失効したクライアント証明書を使用して、個々のユーザーの細かいアクセス制御を構成します。

### <a name="revoke-a-client-certificate"></a>クライアント証明書の失効

失効リストに拇印を追加することで、クライアント証明書を失効させることができます。

1. クライアント証明書の拇印を取得します。 詳細については、「[方法: 証明書のサムプリントを取得する](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate)」を参照してください。
1. 情報をテキスト エディターにコピーし、文字列が 1 つにつながるようにスペースをすべて削除します。
1. 仮想ネットワーク ゲートウェイの **[ポイント対サイトの構成]** ページに移動します。 このページは、[信頼されたルート証明書のアップロード](#uploadfile)に使用したものです。
1. **[失効した証明書]** セクションで、証明書のフレンドリ名を入力します (証明書 CN にする必要はありません)。
1. 拇印の文字列をコピーして **[拇印]** フィールドに貼り付けます。
1. 拇印が検証され、自動的に失効リストに追加されます。 リストが更新されていることを示すメッセージが画面に表示されます。 
1. 更新が完了した後は、証明書を接続に使用することができなくなります。 この証明書を使用して接続を試みたクライアントには、証明書が無効になっていることを示すメッセージが表示されます。

## <a name="point-to-site-faq"></a><a name="faq"></a>ポイント対サイトに関する FAQ

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="next-steps"></a>次のステップ
接続が完成したら、仮想ネットワークに仮想マシンを追加することができます。 詳細については、[Virtual Machines](../index.yml) に関するページを参照してください。 ネットワークと仮想マシンの詳細については、「[Azure と Linux の VM ネットワークの概要](../virtual-machines/network-overview.md)」を参照してください。

P2S のトラブルシューティング情報については、[Azure ポイント対サイト接続のトラブルシューティング](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)に関するページを参照してください。
