---
title: 'Démarrage rapide : Créer un équilibreur de charge public - Azure PowerShell'
titleSuffix: Azure Load Balancer
description: Ce démarrage rapide explique comment créer un équilibreur de charge à l’aide d’Azure PowerShell.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a load balancer so that I can load balance internet traffic to VMs.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/23/2020
ms.author: allensu
ms:custom: seodec18
ms.openlocfilehash: b8a95687b1567eb6e063ccc871a4a130c5f2db69
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290352"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-azure-powershell"></a>Démarrage rapide : Créer un équilibreur de charge public pour équilibrer la charge des machines virtuelles à l’aide d’Azure PowerShell

Commencez avec Azure Load Balancer en utilisant Azure PowerShell pour créer un équilibreur de charge public et trois machines virtuelles.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell installé localement ou Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 5.4.1 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également exécuter `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) :

* Nommé **myResourceGroupLB**.
* Dans l’emplacement **eastus**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'eastus'

New-AzResourceGroup -Name $rg -Location $loc
```
---

# <a name="option-1-default-create-a-load-balancer-standard-sku"></a>[Option 1 (par défaut) : Créer un équilibreur de charge (référence SKU Standard)](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production. Pour plus d’informations sur les références SKU, consultez **[Références SKU Azure Load Balancer](skus.md)** .

## <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Pour accéder à votre application web sur Internet, vous avez besoin d’une adresse IP publique pour l’équilibreur de charge. 

Utilisez [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) pour :

* Créer une adresse IP publique redondante dans une zone standard nommée **myPublicIP**.
* Dans **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIP'
$sku = 'Standard'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku
```

Pour créer une adresse IP publique zonale dans la zone 1, utilisez la commande suivante :

```azurepowershell-interactive
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIP'
$sku = 'Standard'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku -zone 1
```

## <a name="create-standard-load-balancer"></a>Créer un équilibreur de charge standard

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de l’équilibreur de charge :

  * Un pool d’adresses IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
  * Un pool d’adresses IP principal auquel le pool frontal envoie le trafic réseau dont la charge est équilibrée.
  * Une sonde d’intégrité qui détermine l’intégrité des instances de machine virtuelle backend.
  * Une règle d’équilibreur de charge qui définit la distribution du trafic aux machines virtuelles.

### <a name="create-frontend-ip"></a>Créer une adresse IP de serveur frontal

Créez une IP front-end avec [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) :

* Nommée **myFrontEnd**.
* Attachée à l’adresse IP publique **myPublicIP**.

```azurepowershell-interactive
## Variables for the commands ##
$fe = 'myFrontEnd'
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIP'

$publicIp = 
Get-AzPublicIpAddress -Name $pubIP -ResourceGroupName $rg

$feip = 
New-AzLoadBalancerFrontendIpConfig -Name $fe -PublicIpAddress $publicIp
```

### <a name="configure-back-end-address-pool"></a>Configurer un pool d’adresses back-end

Créez un pool d’adresses back-end avec [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) : 

* Nommé **myBackEndPool**.
* Les machines virtuelles se joignent à ce pool back-end dans les étapes suivantes.

```azurepowershell-interactive
## Variable for the command ##
$be = 'myBackEndPool'

$bepool = 
New-AzLoadBalancerBackendAddressPoolConfig -Name $be
```

### <a name="create-the-health-probe"></a>Créer la sonde d’intégrité

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent transmettre du trafic réseau. 

Une machine virtuelle dont le contrôle de sonde a échoué est supprimée de l’équilibreur de charge. La machine virtuelle est rajoutée à l’équilibreur de charge une fois l’échec résolu.

Créez une sonde d’intégrité avec [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) :

* Analyse l’intégrité des machines virtuelles.
* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Surveillant le **Port 80**.

```azurepowershell-interactive
## Variables for the command ##
$hp = 'myHealthProbe'
$pro = 'http'
$port = '80'
$int = '360'
$cnt = '5'

$probe = 
New-AzLoadBalancerProbeConfig -Name $hp -Protocol $pro -Port $port -RequestPath / -IntervalInSeconds $int -ProbeCount $cnt
```

### <a name="create-the-load-balancer-rule"></a>Créer la règle d’équilibreur de charge

Une règle d’équilibreur de charge définit les éléments suivants :

* La configuration IP frontale pour le trafic entrant.
* Le pool d’adresses IP principal qui reçoit le trafic.
* Les ports source et de destination requis. 

Créez une règle d’équilibreur de charge avec [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) : 

* Nommée **myHTTPRule**.
* Écoutant le **Port 80** dans le pool frontal **myFrontEnd**.
* Envoyant le trafic réseau dont la charge est équilibrée au pool d’adresses principal **myBackEndPool** à l’aide du **Port 80**. 
* Utilisant la sonde d’intégrité **myHealthProbe**.
* Protocole **TCP**.

```azurepowershell-interactive
## Variables for the command ##
$lbr = 'myHTTPRule'
$pro = 'tcp'
$port = '80'

## $feip and $bePool are the variables from previous steps. ##

$rule = 
New-AzLoadBalancerRuleConfig -Name $lbr -Protocol $pro -Probe $probe -FrontendPort $port -BackendPort $port -FrontendIpConfiguration $feip -BackendAddressPool $bePool -DisableOutboundSNAT
```

### <a name="create-load-balancer-resource"></a>Créer la ressource d’équilibreur de charge

Créez l’équilibreur de charge public avec [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) :

* Nommé **myLoadBalancer**.
* Dans l’emplacement **eastus**.
* Dans le groupe de ressources **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for the command ##
$lbn = 'myLoadBalancer'
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$sku = 'Standard'

## $feip, $bepool, $probe, $rule are variables with configuration information from previous steps. ##

$lb = 
New-AzLoadBalancer -ResourceGroupName $rg -Name $lbn -SKU $sku -Location $loc -FrontendIpConfiguration $feip -BackendAddressPool $bepool -Probe $probe -LoadBalancingRule $rule
```

## <a name="configure-virtual-network"></a>Configurer un réseau virtuel

Avant de déployer des machines virtuelles et de tester votre équilibreur de charge, créez les ressources de réseau virtuel associées.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) :

* Nommé **myVNet**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Sous-réseau nommé **myBackendSubnet**.
* Réseau virtuel **10.0.0.0/16**.
* Sous-réseau **10.0.0.0/24**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$sub = 'myBackendSubnet'
$spfx = '10.0.0.0/24'
$vnm = 'myVNet'
$vpfx = '10.0.0.0/16'

## Create subnet config ##
$subnetConfig = 
New-AzVirtualNetworkSubnetConfig -Name $sub -AddressPrefix $spfx

## Create the virtual network ##
$vnet = 
New-AzVirtualNetwork -ResourceGroupName $rg -Location $loc -Name $vnm -AddressPrefix $vpfx -Subnet $subnetConfig
```
### <a name="create-public-ip-addresses-for-the-vms"></a>Créer des adresses IP publiques pour les machines virtuelles

Pour accéder à vos machines virtuelles à l’aide d’une connexion RDP, vous avez besoin d’adresses IP publiques pour les machines virtuelles. 

Utilisez [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) pour créer des adresses IP publiques standard pour :

#### <a name="vm1"></a>MV1

* Nommée **myVMPubIP1**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Référence **Standard**.
* Allocation **statique** pour l’adresse IP.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$ip1 = 'myVMPubIP1'
$sku = 'Standard'
$all = 'static'

$RdpPubIP1 = 
New-AzPublicIpAddress -Name $ip1 -ResourceGroupName $rg -Location $loc -SKU $sku -AllocationMethod $all
```

#### <a name="vm2"></a>MV2

* Nommée **myVMPubIP2**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Référence **Standard**.
* Allocation **statique** pour l’adresse IP. 

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$ip2 = 'myVMPubIP2'
$sku = 'Standard'
$all = 'static'

$RdpPubIP2 = 
New-AzPublicIpAddress -Name $ip2 -ResourceGroupName $rg -Location $loc -SKU $sku -AllocationMethod $all
```

#### <a name="vm3"></a>MV3

* Nommée **myVMPubIP3**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Référence **Standard**.
* Allocation **statique** pour l’adresse IP. 

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$ip3 = 'myVMPubIP3'
$sku = 'Standard'
$all = 'static'

$RdpPubIP3 = 
New-AzPublicIpAddress -Name $ip3 -ResourceGroupName $rg -Location $loc -SKU $sku -AllocationMethod $all
```

### <a name="create-network-security-group"></a>Création d’un groupe de sécurité réseau
Créez un groupe de sécurité réseau pour définir les connexions entrantes vers votre réseau virtuel.

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>Créer une règle de groupe de sécurité réseau pour le port 3389

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) :

* Nommé **myNSGRuleRDP**.
* Description de **Autoriser RDP**.
* Accès **Autoriser**.
* Protocole **TCP**.
* Direction **entrante**.
* Priorité **1000**.
* Source : **Internet**.
* Plage de ports sources de **(*)** .
* Préfixe d’adresse de destination de **(*)** .
* Port de destination **3389**.

```azurepowershell-interactive
## Variables for command ##
$rnm = 'myNSGRuleRDP'
$des = 'Allow RDP'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '1000'
$spfx = 'Internet'
$spr = '*'
$dpfx = '*'
$dpr = '3389'

$rule1 = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $des -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix $spfx -SourcePortRange $spr -DestinationAddressPrefix $dpfx -DestinationPortRange $dpr
```

#### <a name="create-a-network-security-group-rule-for-port-80"></a>Créer une règle de groupe de sécurité réseau pour le port 80
Créez une règle de groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) :

* Nommée **myNSGRuleHTTP**.
* Description **Autoriser HTTP**.
* Accès **Autoriser**.
* Protocole **TCP**.
* Direction **entrante**.
* Priorité **2000**.
* Source **Internet**.
* Plage de ports sources de **(*)** .
* Préfixe d’adresse de destination de **(*)** .
* Port de destination **80**.

```azurepowershell-interactive
## Variables for command ##
$rnm = 'myNSGRuleHTTP'
$des = 'Allow HTTP'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '2000'
$spfx = 'Internet'
$spr = '*'
$dpfx = '*'
$dpr = '80'

$rule2 = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $des -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix $spfx -SourcePortRange $spr -DestinationAddressPrefix $dpfx -DestinationPortRange $dpr
```

#### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) :

* Nommé **myNSG**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Avec les règles de sécurité créées dans les étapes précédentes stockées dans une variable.

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nmn = 'myNSG'

## $rule1 and $rule2 are variables with configuration information from the previous steps. ##
$nsg = 
New-AzNetworkSecurityGroup -ResourceGroupName $rg -Location $loc -Name $nmn -SecurityRules $rule1,$rule2
```

### <a name="create-network-interfaces"></a>Création d’interfaces réseau

Créez trois interfaces réseau à l’aide de [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) :

#### <a name="vm-1"></a>Machine virtuelle 1

* Nommée **myNicVM1**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.
* Attachée à l’équilibreur de charge **myLoadBalancer** dans **myBackEndPool**.

```azurepowershell-interactive
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nic1 = 'myNicVM1'
$vnt = 'myVNet'
$ip1 = 'myVMPubIP1'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get public ip address for VM1 ##
$pub1 = 
Get-AzPublicIPAddress -Name $ip1 -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM1 ##
$nicVM1 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic1 -PublicIpAddress $pub1 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-2"></a>Machine virtuelle 2

* Nommée **myNicVM2**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.
* Attachée à l’équilibreur de charge **myLoadBalancer** dans **myBackEndPool**.

```azurepowershell-interactive
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nic2 = 'myNicVM2'
$vnt = 'myVNet'
$ip2 = 'myVMPubIP2'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get public ip address for VM2 ##
$pub2 = 
Get-AzPublicIPAddress -Name $ip2 -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM2 ##
$nicVM2 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic2 -PublicIpAddress $pub2 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-3"></a>Machine virtuelle 3

* Nommée **myNicVM3**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.
* Attachée à l’équilibreur de charge **myLoadBalancer** dans **myBackEndPool**.

```azurepowershell-interactive
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nic3 = 'myNicVM3'
$vnt = 'myVNet'
$ip3 = 'myVMPubIP3'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get public ip address for VM3 ##
$pub3 = 
Get-AzPublicIPAddress -Name $ip3 -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM3 ##
$nicVM3 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic3 -PublicIpAddress $pub3 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```
### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Définissez un nom d’utilisateur administrateur et un mot de passe pour les machines virtuelles avec [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) :

```azurepowershell
$cred = Get-Credential
```

Créez les machines virtuelles avec :

* [New-AzVM](/powershell/module/az.compute/new-azvm)
* [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
* [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
* [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
* [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)


#### <a name="vm1"></a>MV1

* Nommée **myVM1**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Attachée à l’interface réseau **myNicVM1**.
* Attachée à l’équilibreur de charge **myLoadBalancer**.
* Dans **Zone 1**.
* Dans l’emplacement **eastus**.

```azurepowershell-interactive
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM1'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$zn = '1'
$loc = 'eastus'

## Create a virtual machine configuration. $cred and $nicVM1 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM1.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Zone $zn -Location $loc -VM $vmConfig
```


#### <a name="vm2"></a>MV2

* Nommée **myVM2**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Attachée à l’interface réseau **myNicVM2**.
* Attachée à l’équilibreur de charge **myLoadBalancer**.
* Dans **Zone 2**.
* Dans l’emplacement **eastus**.

```azurepowershell-interactive
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM2'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$zn = '2'
$loc = 'eastus'

## Create a virtual machine configuration. $cred and $nicVM2 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM2.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Zone $zn -Location $loc -VM $vmConfig
```

#### <a name="vm3"></a>MV3

* Nommée **myVM3**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Attachée à l’interface réseau **myNicVM3**.
* Attachée à l’équilibreur de charge **myLoadBalancer**.
* Dans **Zone 3**.
* Dans l’emplacement **eastus**.

```azurepowershell-interactive
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM3'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$zn = '3'
$loc = 'eastus'

## Create a virtual machine configuration. $cred and $nicVM3 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM3.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Zone $zn -Location $loc -VM $vmConfig
```

La création et la configuration des trois machines virtuelles prennent quelques minutes.

## <a name="create-outbound-rule-configuration"></a>Créer une configuration de règle de trafic sortant
Les règles de trafic sortant de l’équilibreur de charge configurent la traduction d’adresses réseau source (SNAT) sortante pour les machines virtuelles dans le pool principal. 

Pour plus d’informations sur les connexions sortantes, consultez [Connexions sortantes dans Azure](load-balancer-outbound-connections.md).

### <a name="create-outbound-public-ip-address"></a>Créer une adresse IP publique sortante

Utilisez [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) pour :

* Créer une adresse IP publique redondante dans une zone standard nommée **myPublicIPmyPublicIPOutbound**.
* Dans **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIPOutbound'
$sku = 'Standard'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku
```

Pour créer une adresse IP publique zonale dans la zone 1, utilisez la commande suivante :

```azurepowershell-interactive
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIPOutbound'
$sku = 'Standard'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku -zone 1
```
### <a name="create-outbound-frontend-ip-configuration"></a>Créer une configuration IP frontale sortante

Créez une configuration IP frontale avec [Add-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/add-azloadbalancerfrontendipconfig) :

* Nommée **myFrontEndOutbound**.
* Associée à une adresse IP publique **myPublicIPOutbound**.

```azurepowershell-interactive
## Variables for the command ##
$fen = 'myFrontEndOutbound'

## Get the load balancer configuration  and apply the frontend config##
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg | Add-AzLoadBalancerFrontendIPConfig -Name $fen -PublicIpAddress $publicIP | Set-AzLoadBalancer
```

### <a name="create-outbound-pool"></a>Créer un pool sortant

Créez un pool sortant avec [Add-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/add-azloadbalancerbackendaddresspoolconfig). 

Appliquez le pool et l’adresse IP frontale à l’équilibreur de charge avec [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer) :

* Nommé **myBackendPoolOutbound**.

```azurepowershell-interactive
## Variables for the command ##
$ben = 'myBackEndPoolOutbound'
$lbn = 'myLoadBalancer'
$rg = 'myResourceGroupLB'

## Get the load balancer configuration and create the outbound backend address pool##
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg | Add-AzLoadBalancerBackendAddressPoolConfig -Name $ben | Set-AzLoadBalancer
```
### <a name="create-outbound-rule-and-apply-to-load-balancer"></a>Créer une règle de trafic sortant et l’appliquer à l’équilibreur de charge

Créez une règle de trafic sortant pour le pool principal sortant avec [Add-AzLoadBalancerOutboundRuleConfig](/powershell/module/az.network/new-azloadbalanceroutboundruleconfig). 

Appliquez la règle à l’équilibreur de charge avec [New-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer) :

* Nommée **myOutboundRule**.
* Associée avec l’équilibreur de charge **myLoadBalancer**.
* Associée au serveur frontal **myFrontEndOutbound**.
* Protocole **Tous**.
* Délai d’inactivité de **15**.
* **10000** ports sortants.
* Associée au pool principal **myBackEndPoolOutbound**.
* Dans le groupe de ressources **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$brn = 'myOutboundRule'
$pro = 'All'
$idl = '15'
$por = '10000'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg 

## Apply the outbound rule configuration to the load balancer. ##
$lb | Add-AzLoadBalancerOutBoundRuleConfig -Name $brn -FrontendIPConfiguration $lb.FrontendIpConfigurations[1] -BackendAddressPool $lb.BackendAddressPools[1] -Protocol $pro -IdleTimeoutInMinutes $idl -AllocatedOutboundPort $por | Set-AzLoadBalancer
```

### <a name="add-virtual-machines-to-outbound-pool"></a>Ajouter des machines virtuelles au pool sortant

Ajoutez les interfaces réseau des machines virtuelles au pool sortant de l’équilibreur de charge avec [Add-AzNetworkInterfaceIpConfig](/powershell/module/az.network/add-aznetworkinterfaceipconfig) :


#### <a name="vm1"></a>MV1
* Dans le pool d’adresses principal **myBackEndPoolOutbound**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Associée à l’interface réseau **myNicVM1** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurepowershell-interactive
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$bep = 'myBackEndPoolOutbound'
$nic1 = 'myNicVM1'
$ipc = 'ipconfig1'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg

## Get the network interface configuration ##
$nic = 
Get-AzNetworkInterface -Name $nic1 -ResourceGroupName $rg

## Apply the backend to the network interface ##
$nic | Set-AzNetworkInterfaceIpConfig -Name $ipc -LoadBalancerBackendAddressPoolId $lb.BackendAddressPools[1].id | Set-AzNetworkInterface
```

#### <a name="vm2"></a>MV2
* Dans le pool d’adresses principal **myBackEndPoolOutbound**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Associée à l’interface réseau **myNicVM2** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurepowershell-interactive
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$bep = 'myBackEndPoolOutbound'
$nic2 = 'myNicVM2'
$ipc = 'ipconfig1'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg

## Get the network interface configuration ##
$nic = 
Get-AzNetworkInterface -Name $nic2 -ResourceGroupName $rg

## Apply the backend to the network interface ##
$nic | Set-AzNetworkInterfaceIpConfig -Name $ipc -LoadBalancerBackendAddressPoolId $lb.BackendAddressPools[1].id | Set-AzNetworkInterface
```

#### <a name="vm3"></a>MV3
* Dans le pool d’adresses principal **myBackEndPoolOutbound**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Associée à l’interface réseau **myNicVM3** et **ipconfig1**.
* Associée à l’équilibreur de charge **myLoadBalancer**.

```azurepowershell-interactive
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$bep = 'myBackEndPoolOutbound'
$nic3 = 'myNicVM3'
$ipc = 'ipconfig1'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg

## Get the network interface configuration ##
$nic = 
Get-AzNetworkInterface -Name $nic3 -ResourceGroupName $rg

## Apply the backend to the network interface ##
$nic | Set-AzNetworkInterfaceIpConfig -Name $ipc -LoadBalancerBackendAddressPoolId $lb.BackendAddressPools[1].id | Set-AzNetworkInterface

```

# <a name="option-2-create-a-load-balancer-basic-sku"></a>[Option n°2 : Créer un équilibreur de charge (référence SKU De base)](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production. Pour plus d’informations sur les références SKU, consultez **[Références SKU Azure Load Balancer](skus.md)** .

## <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Pour accéder à votre application web sur Internet, vous avez besoin d’une adresse IP publique pour l’équilibreur de charge. 

Utilisez [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) pour :

* Créer une adresse IP publique redondante dans une zone standard nommée **myPublicIP**.
* Dans **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIP'
$sku = 'Basic'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku
```

## <a name="create-basic-load-balancer"></a>Créer un équilibreur de charge de base

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de l’équilibreur de charge :

  * Un pool d’adresses IP frontal qui reçoit le trafic réseau entrant sur l’équilibreur de charge.
  * Un pool d’adresses IP principal auquel le pool frontal envoie le trafic réseau dont la charge est équilibrée.
  * Une sonde d’intégrité qui détermine l’intégrité des instances de machine virtuelle backend.
  * Une règle d’équilibreur de charge qui définit la distribution du trafic aux machines virtuelles.

### <a name="create-frontend-ip"></a>Créer une adresse IP de serveur frontal

Créez une adresse IP front-end avec [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) :

* Nommée **myFrontEnd**.
* Attachée à l’adresse IP publique **myPublicIP**.

```azurepowershell-interactive
## Variables for the commands ##
$fe = 'myFrontEnd'
$rg = 'MyResourceGroupLB'
$loc = 'eastus'
$pubIP = 'myPublicIP'

$publicIp = 
Get-AzPublicIpAddress -Name $pubIP -ResourceGroupName $rg

$feip = 
New-AzLoadBalancerFrontendIpConfig -Name $fe -PublicIpAddress $publicIp
```

### <a name="configure-back-end-address-pool"></a>Configurer un pool d’adresses back-end

Créez un pool d’adresses back-end avec [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) : 

* Nommé **myBackEndPool**.
* Les machines virtuelles se joignent à ce pool back-end dans les étapes suivantes.

```azurepowershell-interactive
## Variable for the command ##
$be = 'myBackEndPool'

$bepool = 
New-AzLoadBalancerBackendAddressPoolConfig -Name $be
```

### <a name="create-the-health-probe"></a>Créer la sonde d’intégrité

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent transmettre du trafic réseau. 

Une machine virtuelle dont le contrôle de sonde a échoué est supprimée de l’équilibreur de charge. La machine virtuelle est rajoutée à l’équilibreur de charge une fois l’échec résolu.

Créez une sonde d’intégrité avec [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) :

* Analyse l’intégrité des machines virtuelles.
* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Surveillant le **Port 80**.

```azurepowershell-interactive
## Variables for the command ##
$hp = 'myHealthProbe'
$pro = 'http'
$port = '80'
$int = '360'
$cnt = '5'

$probe = 
New-AzLoadBalancerProbeConfig -Name $hp -Protocol $pro -Port $port -RequestPath / -IntervalInSeconds $int -ProbeCount $cnt
```

### <a name="create-the-load-balancer-rule"></a>Créer la règle d’équilibreur de charge

Une règle d’équilibreur de charge définit les éléments suivants :

* La configuration IP frontale pour le trafic entrant.
* Le pool d’adresses IP principal qui reçoit le trafic.
* Les ports source et de destination requis. 

Créez une règle d’équilibreur de charge avec [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) : 

* Nommée **myHTTPRule**.
* Écoutant le **Port 80** dans le pool frontal **myFrontEnd**.
* Envoyant le trafic réseau dont la charge est équilibrée au pool d’adresses principal **myBackEndPool** à l’aide du **Port 80**. 
* Utilisant la sonde d’intégrité **myHealthProbe**.
* Protocole **TCP**.
* Activez la traduction d’adresses réseau source (SNAT) sortante à l’aide de l’adresse IP frontale.


```azurepowershell-interactive
## Variables for the command ##
$lbr = 'myHTTPRule'
$pro = 'tcp'
$port = '80'

## $feip and $bePool are the variables from previous steps. ##

$rule = 
New-AzLoadBalancerRuleConfig -Name $lbr -Protocol $pro -Probe $probe -FrontendPort $port -BackendPort $port -FrontendIpConfiguration $feip -BackendAddressPool $bePool
```

### <a name="create-load-balancer-resource"></a>Créer des ressources d’équilibreur de charge

Créez un équilibreur de charge public avec [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) :

* Nommé **myLoadBalancer**.
* Dans l’emplacement **eastus**.
* Dans le groupe de ressources **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for the command ##
$lbn = 'myLoadBalancer'
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$sku = 'Basic'

## $feip, $bepool, $probe, $rule are variables with configuration information from previous steps. ##

$lb = 
New-AzLoadBalancer -ResourceGroupName $rg -Name $lbn -SKU $sku -Location $loc -FrontendIpConfiguration $feip -BackendAddressPool $bepool -Probe $probe -LoadBalancingRule $rule
```

## <a name="configure-virtual-network"></a>Configurer un réseau virtuel

Avant de déployer des machines virtuelles et de tester votre équilibreur de charge, créez les ressources de réseau virtuel associées.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) :

* Nommé **myVNet**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Sous-réseau nommé **myBackendSubnet**.
* Réseau virtuel **10.0.0.0/16**.
* Sous-réseau **10.0.0.0/24**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$sub = 'myBackendSubnet'
$spfx = '10.0.0.0/24'
$vnm = 'myVNet'
$vpfx = '10.0.0.0/16'

## Create subnet config ##
$subnetConfig = 
New-AzVirtualNetworkSubnetConfig -Name $sub -AddressPrefix $spfx

## Create the virtual network ##
$vnet = 
New-AzVirtualNetwork -ResourceGroupName $rg -Location $loc -Name $vnm -AddressPrefix $vpfx -Subnet $subnetConfig
```
### <a name="create-public-ip-addresses-for-the-vms"></a>Créer des adresses IP publiques pour les machines virtuelles

Pour accéder à vos machines virtuelles à l’aide d’une connexion RDP, vous avez besoin d’adresses IP publiques pour les machines virtuelles. 

Utilisez [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) pour créer des adresses IP publiques standard pour :

#### <a name="vm1"></a>MV1

* Nommée **myVMPubIP1**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Référence **Standard**.
* Allocation **statique** pour l’adresse IP.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$ip1 = 'myVMPubIP1'
$sku = 'Basic'
$all = 'static'

$RdpPubIP1 = 
New-AzPublicIpAddress -Name $ip1 -ResourceGroupName $rg -Location $loc -SKU $sku -AllocationMethod $all
```

#### <a name="vm2"></a>MV2

* Nommée **myVMPubIP2**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Référence **Standard**.
* Allocation **statique** pour l’adresse IP. 

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$ip2 = 'myVMPubIP2'
$sku = 'Basic'
$all = 'static'

$RdpPubIP2 = 
New-AzPublicIpAddress -Name $ip2 -ResourceGroupName $rg -Location $loc -SKU $sku -AllocationMethod $all
```

#### <a name="vm3"></a>MV3

* Nommée **myVMPubIP3**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Référence **Standard**.
* Allocation **statique** pour l’adresse IP. 

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$ip3 = 'myVMPubIP3'
$sku = 'Basic'
$all = 'static'

$RdpPubIP2 = 
New-AzPublicIpAddress -Name $ip3 -ResourceGroupName $rg -Location $loc -SKU $sku -AllocationMethod $all
```

### <a name="create-network-security-group"></a>Création d’un groupe de sécurité réseau
Créez un groupe de sécurité réseau pour définir les connexions entrantes vers votre réseau virtuel.

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>Créer une règle de groupe de sécurité réseau pour le port 3389

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) :

* Nommé **myNSGRuleRDP**.
* Description de **Autoriser RDP**.
* Accès **Autoriser**.
* Protocole **TCP**.
* Direction **entrante**.
* Priorité **1000**.
* Source **Internet**.
* Plage de ports sources de **(*)** .
* Préfixe d’adresse de destination de **(*)** .
* Port de destination **3389**.

```azurepowershell-interactive
## Variables for command ##
$rnm = 'myNSGRuleRDP'
$des = 'Allow RDP'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '1000'
$spfx = 'Internet'
$spr = '*'
$dpfx = '*'
$dpr = '3389'

$rule1 = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $des -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix $spfx -SourcePortRange $spr -DestinationAddressPrefix $dpfx -DestinationPortRange $dpr
```

#### <a name="create-a-network-security-group-rule-for-port-80"></a>Créer une règle de groupe de sécurité réseau pour le port 80
Créez une règle de groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) :

* Nommée **myNSGRuleHTTP**.
* Description de **Autoriser HTTP**.
* Accès **Autoriser**.
* Protocole **TCP**.
* Direction **entrante**.
* Priorité **2000**.
* Source **Internet**.
* Plage de ports sources de **(*)** .
* Préfixe d’adresse de destination de **(*)** .
* Port de destination **80**.

```azurepowershell-interactive
## Variables for command ##
$rnm = 'myNSGRuleHTTP'
$des = 'Allow HTTP'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '2000'
$spfx = 'Internet'
$spr = '*'
$dpfx = '*'
$dpr = '80'

$rule2 = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $des -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix $spfx -SourcePortRange $spr -DestinationAddressPrefix $dpfx -DestinationPortRange $dpr
```

#### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) :

* Nommé **myNSG**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Avec les règles de sécurité créées dans les étapes précédentes stockées dans une variable.

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nmn = 'myNSG'

## $rule1 and $rule2 are variables with configuration information from the previous steps. ##
$nsg = 
New-AzNetworkSecurityGroup -ResourceGroupName $rg -Location $loc -Name $nmn -SecurityRules $rule1,$rule2
```

### <a name="create-network-interfaces"></a>Création d’interfaces réseau

Créez trois interfaces réseau à l’aide de [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) :

#### <a name="vm-1"></a>Machine virtuelle 1

* Nommée **myNicVM1**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.
* Attachée à l’équilibreur de charge **myLoadBalancer** dans **myBackEndPool**.

```azurepowershell-interactive
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nic1 = 'myNicVM1'
$vnt = 'myVNet'
$ip1 = 'myVMPubIP1'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get public ip address for VM1 ##
$pub1 = 
Get-AzPublicIPAddress -Name $ip1 -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM1 ##
$nicVM1 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic1 -PublicIpAddress $pub1 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-2"></a>Machine virtuelle 2

* Nommée **myNicVM2**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.
* Attachée à l’équilibreur de charge **myLoadBalancer** dans **myBackEndPool**.

```azurepowershell-interactive
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nic2 = 'myNicVM2'
$vnt = 'myVNet'
$ip2 = 'myVMPubIP2'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get public ip address for VM2 ##
$pub2 = 
Get-AzPublicIPAddress -Name $ip2 -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM2 ##
$nicVM2 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic2 -PublicIpAddress $pub2 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-3"></a>Machine virtuelle 3

* Nommée **myNicVM3**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Dans le groupe de sécurité réseau **myNSG**.
* Attachée à l’équilibreur de charge **myLoadBalancer** dans **myBackEndPool**.

```azurepowershell-interactive
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'eastus'
$nic3 = 'myNicVM3'
$vnt = 'myVNet'
$ip3 = 'myVMPubIP3'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get public ip address for VM3 ##
$pub3 = 
Get-AzPublicIPAddress -Name $ip3 -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM3 ##
$nicVM3 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic3 -PublicIpAddress $pub3 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

### <a name="create-availability-set-for-virtual-machines"></a>Créer un groupe à haute disponibilité pour des machines virtuelles

Utilisez [New-AzAvailabilitySet](/powershell/module/az.compute/new-azvm) pour créer un groupe à haute disponibilité pour les machines virtuelles :

* Nommé **myAvSet**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Dans l’emplacement **eastus**.

```azurepowershell-interactive
## Variables used for the command. ##
$rg = 'myResourceGroupLB'
$avs = 'myAvSet'
$loc = 'eastus'

New-AzAvailabilitySet -ResourceGroupName $rg -Name $avs -Location $loc
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Définissez un nom d’utilisateur administrateur et un mot de passe pour les machines virtuelles avec [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) :

```azurepowershell
$cred = Get-Credential
```

Créez les machines virtuelles avec :

* [New-AzVM](/powershell/module/az.compute/new-azvm)
* [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
* [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
* [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
* [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)


#### <a name="vm1"></a>MV1

* Nommée **myVM1**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Attachée à l’interface réseau **myNicVM1**.
* Attachée à l’équilibreur de charge **myLoadBalancer**.
* Dans l’emplacement **eastus**.
* Dans le groupe à haute disponibilité **myAvSet**.

```azurepowershell-interactive
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM1'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'eastus'
$avs = 'myAvSet'

## Create a virtual machine configuration. $cred and $nicVM1 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM1.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig -AvailabilitySetName $avs
```


#### <a name="vm2"></a>MV2

* Nommée **myVM2**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Attachée à l’interface réseau **myNicVM2**.
* Attachée à l’équilibreur de charge **myLoadBalancer**.
* Dans l’emplacement **eastus**.
* Dans le groupe à haute disponibilité **myAvSet**.

```azurepowershell-interactive
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM2'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'eastus'
$avs = 'myAvSet'

## Create a virtual machine configuration. $cred and $nicVM2 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM2.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig -AvailabilitySetName $avs
```

#### <a name="vm3"></a>MV3

* Nommée **myVM3**.
* Dans le groupe de ressources **myResourceGroupLB**.
* Attachée à l’interface réseau **myNicVM3**.
* Attachée à l’équilibreur de charge **myLoadBalancer**.
* Dans l’emplacement **eastus**.
* Dans le groupe à haute disponibilité **myAvSet**.

```azurepowershell-interactive
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM3'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'eastus'
$avs = 'myAvSet'

## Create a virtual machine configuration. $cred and $nicVM3 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM3.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig -AvailabilitySetName $avs
```

La création et la configuration des trois machines virtuelles prennent quelques minutes.

---

### <a name="install-iis-with-a-custom-web-page"></a>Installer IIS avec une page web personnalisée

Installez IIS avec une page web personnalisée sur les deux machines virtuelles back-end comme suit :

1. Obtenez les adresses IP publiques des trois machines virtuelles avec [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress).

   ```azurepowershell-interactive
   ## Variables for commands. ##
   $rg = 'myResourceGroupLB'
   $ip1 = 'myVMPubIP1'
   $ip2 = 'myVMPubIP2'
   $ip3 = 'myVMPubIP3'

   ## VM1 ##
   (Get-AzPublicIPAddress -ResourceGroupName $rg -Name $ip1).IpAddress

   ## VM2 ##
   (Get-AzPublicIPAddress -ResourceGroupName $rg -Name $ip2).IpAddress

   ## VM3 ##
   (Get-AzPublicIPAddress -ResourceGroupName $rg -Name $ip3).IpAddress
   ```
   
2. Créez des connexions Bureau à distance avec **myVM1**, **myVM2** et **myVM3** en utilisant les adresses IP publiques des machines virtuelles.

3. Entrez les informations d’identification de chaque machine virtuelle pour démarrer la session RDP.

4. Lancez Windows PowerShell sur chaque machine virtuelle et utilisez les commandes suivantes pour installer le serveur IIS et mettre à jour le fichier htm par défaut.

   ```powershell
   # Install IIS
   Install-WindowsFeature -name Web-Server -IncludeManagementTools

   # Remove default htm file
   remove-item  C:\inetpub\wwwroot\iisstart.htm
    
   #Add custom htm file
   Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from host " + $env:computername)
   ```

5. Fermez les connexions RDP à **myVM1**, **myVM2** et **myVM3**.


## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge
Pour obtenir l’adresse IP publique de l’équilibreur de charge, utilisez [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) :

* Nommée **myPublicIP**
* Dans le groupe de ressources **myResourceGroupLB**.

```azurepowershell-interactive
## Variables for command. ##
$rg = 'myResourceGroupLB'
$ipn = 'myPublicIP'

Get-AzPublicIPAddress -ResourceGroupName $rg -Name $ipn | select IpAddress
```

Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur.

![Tester l’équilibreur de charge](media/quickstart-create-basic-load-balancer-powershell/load-balancer-test.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez utiliser la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources, l’équilibreur de charge et toutes les ressources associées. 

```azurepowershell-interactive
## Variable for command. ##
$rg = 'myResourceGroupLB'

Remove-AzResourceGroup -Name $rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide

* Vous avez créé un équilibreur de charge public standard ou de base
* Vous avez attaché des machines virtuelles. 
* Vous avez configuré la règle de trafic et la sonde d’intégrité de l’équilibreur de charge.
* Vous avez testé l’équilibreur de charge.

Pour en savoir plus sur Azure Load Balancer, consultez [Qu’est-ce qu’Azure Load Balancer ?](load-balancer-overview.md) et [Questions fréquentes sur Load Balancer](load-balancer-faqs.md).

Découvrez-en plus sur les [équilibreurs de charge et les zones de disponibilité](load-balancer-standard-availability-zones.md).
