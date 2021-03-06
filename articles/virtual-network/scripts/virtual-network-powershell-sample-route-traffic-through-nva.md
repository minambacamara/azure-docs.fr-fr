---
title: Router le trafic via une appliance virtuelle réseau - Exemple de script Azure PowerShell
description: Exemple de script Azure PowerShell - Acheminer le trafic via une appliance virtuelle réseau de pare-feu.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.openlocfilehash: 6fb16b9839a5f4078347e51647b1f1b916941409
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068587"
---
# <a name="route-traffic-through-a-network-virtual-appliance-script-sample"></a>Exemple de script pour acheminer le trafic via une appliance virtuelle réseau

Cet exemple de script permet de créer un réseau virtuel avec des sous-réseaux frontaux et principaux. Il crée également une machine virtuelle sur laquelle le transfert IP est activé pour acheminer le trafic entre les deux sous-réseaux. Après avoir exécuté le script, vous pouvez déployer un logiciel réseau, telle qu’une application de pare-feu, sur la machine virtuelle.

Vous pouvez exécuter le script à partir d’Azure [Cloud Shell](https://shell.azure.com/powershell) ou à partir d’une installation PowerShell locale. Si vous utilisez PowerShell en local, vous devez exécuter le module Az PowerShell version 5.4.1 ou ultérieure pour les besoins de ce script. Pour trouver la version installée, exécutez `Get-Module -ListAvailable Az`. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/route-traffic-through-nva/route-traffic-through-nva.ps1 "Route traffic through a network virtual appliance")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Exécutez la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées :

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour créer un groupe de ressources, un réseau virtuel et les groupes de sécurité réseau. Chaque commande du tableau suivant renvoie à une documentation spécifique :

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)  | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crée un réseau virtuel et un sous-réseau frontal Azure. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crée des sous-réseaux principaux et DMZ. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique pour accéder à la machine virtuelle à partir d’Internet. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Crée une interface réseau virtuelle et active le transfert IP pour celle-ci. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crée un groupe de sécurité réseau (NSG). |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) | Crée des règles NSG qui autorisent des ports HTTP et HTTPS entrants sur la machine virtuelle. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig)| Associe les NSG et les tables de routage aux sous-réseaux. |
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable)| Crée une table de routage pour tous les itinéraires. |
| [New-AzRouteConfig](/powershell/module/az.network/new-azrouteconfig)| Créer des itinéraires pour acheminer le trafic entre les sous-réseaux et Internet via la machine virtuelle. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Crée une machine virtuelle et lui associe la carte d’interface réseau. Cette commande spécifie également l’image de machine virtuelle à utiliser ainsi que les informations d’identification d’administration. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup)  | Supprime un groupe de ressources, ainsi que toutes ses ressources. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

Vous trouverez des exemples supplémentaires de scripts PowerShell pour réseau virtuel dans [Exemples PowerShell pour réseau virtuel](../powershell-samples.md).