---
title: Dépréciation de la fonctionnalité de chiffrement de données Azure Site Recovery | Microsoft Docs
description: Détails concernant la fonctionnalité de chiffrement de données Azure Site Recovery
services: site-recovery
author: rajani-janaki-ram
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/15/2019
ms.author: rajanaki
ms.openlocfilehash: b9d333467864f67bc357314830ff885af4232ba0
ms.sourcegitcommit: e995f770a0182a93c4e664e60c025e5ba66d6a45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86133194"
---
# <a name="deprecation-of-site-recovery-data-encryption-feature"></a>Dépréciation de la fonctionnalité de chiffrement de données Site Recovery

Ce document décrit les détails de la dépréciation et la mesure de correction que vous devez prendre si vous utilisez la fonctionnalité de chiffrement de données Site Recovery pendant que vous configurez la reprise d’activité après sinistre de machines virtuelles Hyper-V sur Azure. 

## <a name="deprecation-information"></a>Informations sur la dépréciation


La fonctionnalité de chiffrement de données Site Recovery était accessible aux clients qui protégeaient leurs machines virtuelles Hyper-V de façon à assurer la protection des données répliquées contre les menaces de sécurité. Cette fonctionnalité sera dépréciée d’ici le **30 décembre 2019**. Elle sera remplacée par une fonctionnalité plus avancée, le [Chiffrement au repos](https://azure.microsoft.com/blog/azure-site-recovery-encryption-at-rest/), qui utilise [Storage Service Encryption](../storage/common/storage-service-encryption.md) (SSE). Avec SSE, les données sont chiffrées avant d’être consignées dans le stockage et déchiffrées au moment de la récupération. Par ailleurs, lors du basculement vers Azure, vos machines virtuelles s’exécutent à partir des comptes de stockage chiffrés, ce qui permet d’améliorer l’objectif de délai de récupération (RTO).

Notez que si vous êtes un client existant et que vous utilisez cette fonctionnalité, vous devez avoir reçu une note avec des détails concernant la dépréciation et les mesures de correction à prendre. 


## <a name="what-are-the-implications"></a>Quelles sont les conséquences ?

Après le **30 décembre 2019**, toutes les machines virtuelles qui continuent d’utiliser la fonctionnalité de chiffrement retirée ne seront pas autorisées à effectuer le basculement. 

## <a name="required-action"></a>Action requise
Pour poursuivre avec succès les opérations de basculement et les réplications, suivez les étapes indiquées ci-dessous :

Suivez ces étapes pour chaque machine virtuelle : 
1.  [Désactivez la réplication](./site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario).
2.  [Créez une nouvelle stratégie de réplication](./hyper-v-azure-tutorial.md#set-up-a-replication-policy).
3.  [Activez la réplication](./hyper-v-vmm-azure-tutorial.md#enable-replication) et sélectionnez un compte de stockage sur lequel SSE est activé.

Une fois que vous aurez effectué la réplication initiale sur les comptes de stockage avec SSE activé, vos machines virtuelles utiliseront le Chiffrement au repos avec Azure Site Recovery.


## <a name="next-steps"></a>Étapes suivantes
Planifiez l’exécution des étapes de correction et exécutez-les dans les plus brefs délais. Si vous avez des questions au sujet de cette dépréciation, contactez le Support Microsoft. Pour en savoir plus sur le scénario Hyper-V vers Azure, consultez [cette page](hyper-v-vmm-architecture.md).

