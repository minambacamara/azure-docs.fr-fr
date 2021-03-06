---
title: Bien démarrer avec SAP sur des machines virtuelles Azure | Microsoft Docs
description: Découvrir les solutions SAP qui s’exécutent sur des machines virtuelles dans Microsoft Azure
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ad8e5c75-0cf6-4564-ae62-ea1246b4e5f2
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/16/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 03d0367b3794069908b31ee7af1250fd70f1bdf7
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86525209"
---
# <a name="use-azure-to-host-and-run-sap-workload-scenarios"></a>Utiliser Azure pour héberger et exécuter des scénarios de charge de travail SAP

Quand vous utilisez Microsoft Azure, vous pouvez exécuter de façon fiable vos charges de travail et scénarios SAP critiques sur une plateforme scalable, conforme et qui a fait ses preuves en entreprise. Vous bénéficiez de la scalabilité, de la flexibilité et des économies de coût fournies par Azure. Avec le partenariat étendu entre Microsoft et SAP, vous pouvez exécuter les applications SAP dans des scénarios de développement, de test et de production dans Azure en bénéficiant d’une prise en charge complète. De SAP NetWeaver à SAP S/4HANA ou SAP BI, de Linux à Windows et de SAP HANA à SQL, nous sommes là pour vous.

En plus de l’hébergement des scénarios SAP NetWeaver avec différents SGBD sur Azure, vous pouvez héberger d’autres scénarios de charge de travail SAP, comme SAP BI sur Azure. 

Le caractère unique d’Azure pour SAP HANA est une offre qui place Azure à part. Pour permettre l’hébergement de scénarios SAP demandant davantage de ressources mémoire et processeur impliquant SAP HANA, Azure offre l’utilisation de matériel physique dédié au client. Utilisez cette solution pour exécuter des déploiements de SAP HANA nécessitant jusqu’à 24 To de mémoire (avec scale-out jusqu’à 120 To) pour S/4HANA ou d’autres charges de travail SAP HANA. 

L’hébergement de scénarios de charge de travail SAP dans Azure peut également induire des exigences en matière d’intégration des identités et d’authentification unique. Cette situation peut se produire quand vous utilisez Azure Active Directory (Azure AD) pour connecter différents composants SAP et des offres SAP SaaS ou PaaS. Une liste de ces scénarios d’intégration et d’authentification unique avec Azure AD et des entités SAP est décrite et documentée dans la section « Intégration des identités et de l’authentification unique AAD SAP ».

## <a name="changes-to-the-sap-workload-section"></a>Modifications apportées à la section charge de travail SAP
Les modifications apportées aux documents dans la section charge de travail de SAP sur Azure sont répertoriées à la fin de cet article. Les entrées dans le journal des modifications sont conservées pendant environ 180 jours.

## <a name="you-want-to-know"></a>Vous souhaitez savoir
Si vous avez des questions précises, nous allons vous orienter vers des documents ou des flux spécifiques dans cette section de la page de démarrage. Vous souhaitez savoir :

- Quelles sont les machines virtuelles Azure et les unités de grande instance HANA qui sont prises en charge pour quelles versions de logiciels SAP et quelles versions de système d’exploitation. Lisez le document [Quels sont les logiciels SAP pris en charge pour le déploiement Azure](./sap-supported-product-on-azure.md) pour obtenir des réponses et le processus de recherche des informations
- Quels sont les scénarios de déploiement SAP qui sont pris en charge avec les machines virtuelles Azure et les grandes instances HANA. Vous trouverez des informations sur les scénarios pris en charge dans les documents ci-dessous :
    - [Charge de travail SAP sur les machines virtuelles Azure - Scénarios pris en charge](./sap-planning-supported-configurations.md)
    - [Scénarios pris en charge pour des grandes instances HANA](./hana-supported-scenario.md)
- Quels sont les services Azure, les types de machines virtuelles Azure et les services de stockage Azure disponibles dans les différentes régions Azure. Consultez le site [Produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/). 
- Outre Windows et le Pacemaker, les frameworks de haute disponibilité tiers sont-ils pris en charge ? Consultez la deuxième moitié de la [note de support SAP #1928533](https://launchpad.support.sap.com/#/notes/1928533).

 
## <a name="sap-hana-on-azure-large-instances"></a>SAP HANA sur Azure (grandes instances)

Une série de documents vous guident dans l’utilisation de SAP HANA sur Azure (grandes instances), qui est abrégé en « HANA - Grandes instances ». Pour plus d’informations sur les grandes instances HANA, commencez par le document [Présentation et architecture de SAP HANA sur Azure (grandes instances)](./hana-overview-architecture.md) et consultez la documentation associée dans la section Grande instance SAP HANA



## <a name="sap-hana-on-azure-virtual-machines"></a>SAP HANA sur des machines virtuelles Azure
Cette partie de la documentation couvre différents aspects de SAP HANA. Vous devez avoir une bonne connaissance des principaux services d’Azure qui fournissent les services de base de la plateforme IaaS Azure. Vous devez donc connaître les ressources de calcul, de stockage et de réseau Azure. Bon nombre de ces sujets sont abordés dans le [Guide de planification Azure](./planning-guide.md) relatif à SAP NetWeaver. 

 

## <a name="sap-netweaver-deployed-on-azure-virtual-machines"></a>SAP NetWeaver déployé sur des machines virtuelles Azure
Cette section liste la documentation sur la planification et le déploiement de SAP NetWeaver et de Business One sur Azure. La documentation se concentre sur les principes de base et l’utilisation de bases de données non-HANA avec une charge de travail SAP sur Azure. Les documents et les articles relatifs à la haute disponibilité s’appliquent également à la haute disponibilité HANA dans Azure, comme :

- [Guide de planification Azure](./planning-guide.md). 
- [SAP Business One sur des machines virtuelles Azure](./business-one-azure.md)
- [Protéger un déploiement d’applications SAP NetWeaver multiniveau à l’aide de Site Recovery](../../../site-recovery/site-recovery-sap.md)
- [Connecteur SAP LaMa pour Azure](./lama-installation.md)

Pour plus d’informations sur les bases de données non-HANA sous une charge de travail SAP sur Azure, consultez :

- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](./dbms_guide_general.md)
- [Déploiement SGBD de machines virtuelles SQL Server Azure pour SAP NetWeaver](./dbms_guide_sqlserver.md)
- [Déploiement SGBD de machines virtuelles Oracle Azure pour charge de travail SAP](./dbms_guide_oracle.md)
- [Déploiement SGBD de machines virtuelles IBM DB2 Azure pour charge de travail SAP](./dbms_guide_ibm.md)
- [Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP](./dbms_guide_sapase.md)
- [Déploiement de Content Server, LiveCache et SAP maxDB sur des machines virtuelles Azure](./dbms_guide_maxdb.md)

Pour plus d’informations sur les bases de données SAP HANA sur Azure, consultez la section « SAP HANA sur des machines virtuelles Azure ».

Pour plus d’informations sur la haute disponibilité d’une charge de travail SAP sur Azure, consultez :

- [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver](./sap-high-availability-guide-start.md)

Ce document pointe vers différents autres documents relatifs à l’architecture et aux scénarios. Des documents sur des scénarios ultérieurs incluent des liens vers des documents techniques détaillés qui décrivent le déploiement et la configuration des différentes méthodes de haute disponibilité. Les différents documents montrant comment établir et configurer la haute disponibilité pour une charge de travail SAP NetWeaver s’appliquent à la fois aux systèmes d’exploitation Linux et Windows.


Pour plus d’informations sur l’intégration entre Azure Active Directory (Azure AD), les services SAP et l’authentification unique, consultez :

- [Tutoriel : Intégration d’Azure Active Directory avec SAP Cloud for Customer](../../../active-directory/saas-apps/sap-customer-cloud-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutoriel : Intégration d’Azure Active Directory avec SAP Cloud Platform Identity Authentication](../../../active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutoriel : Intégration d’Azure Active Directory avec SAP Cloud Platform](../../../active-directory/saas-apps/sap-hana-cloud-platform-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutoriel : Intégration d’Azure Active Directory avec SAP NetWeaver](../../../active-directory/saas-apps/sap-netweaver-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutoriel : Intégration d’Azure Active Directory avec SAP Business ByDesign](../../../active-directory/saas-apps/sapbusinessbydesign-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Tutoriel : Intégration d’Azure Active Directory avec SAP HANA](../../../active-directory/saas-apps/saphana-tutorial.md?toc=%2fazure%2fvirtual-machines%2fworkloads%2fsap%2ftoc.json)
- [Your S/4HANA environment: Fiori Launchpad SAML single sign-on with Azure AD](https://blogs.sap.com/2017/02/20/your-s4hana-environment-part-7-fiori-launchpad-saml-single-sing-on-with-azure-ad/)

Pour plus d’informations sur l’intégration de services Azure dans des composants SAP, consultez :

- [Utiliser SAP HANA dans Power BI Desktop](/power-bi/desktop-sap-hana)
- [DirectQuery et SAP HANA](/power-bi/desktop-directquery-sap-hana)
- [Utiliser le connecteur SAP BW dans Power BI Desktop](/power-bi/desktop-sap-bw-connector) 
- [Azure Data Factory offre l’intégration des données SAP HANA et Business Warehouse](https://azure.microsoft.com/blog/azure-data-factory-offer-sap-hana-and-business-warehouse-data-integration)


## <a name="change-log"></a>Journal des modifications

- 16/07/2020 : Description de l’utilisation d’Azure PowerShell afin d’installer une nouvelle extension de machine virtuelle pour SAP dans le [guide de déploiement](deployment-guide.md)
- 04/07/2020 : Sortie de [Azure Monitor pour solutions SAP (préversion)](./azure-monitor-overview.md)
- 01/07/2020 : Suggestion d’une configuration de stockage moins coûteuse basée sur la fonctionnalité de stockage en rafale du Stockage Premium Azure dans le document [Configurations de stockage des machines virtuelles Azure SAP HANA](./hana-vm-operations-storage.md) 
- 24/06/2020 : Modification de [Configuration de Pacemaker sur SLES dans Azure](./high-availability-guide-suse-pacemaker.md) de façon à publier un nouvel agent Azure Fence amélioré et une configuration STONITH plus résiliente pour les appareils, à partir de l’agent Azure Fence 
- 24/06/2020 : Modification de [Configuration de Pacemaker sur RHEL dans Azure](./high-availability-guide-rhel-pacemaker.md) de façon à publier une configuration STONITH plus résiliente
- 23/06/2020 : Modification du guide [Planification et implémentation d’Azure Virtual Machines pour SAP NetWeaver](./planning-guide.md) et du guide [Types de Stockage Azure pour la charge de travail SAP](./planning-guide-storage.md)
- 22/06/2020 : Ajout des étapes d’installation de la nouvelle extension de machine virtuelle pour SAP au [Guide de déploiement](deployment-guide.md)
- 16/06/2020 : Modification de [Connectivité du point de terminaison public pour les machines virtuelles utilisant l’équilibreur de charge interne standard Azure dans les scénarios de haute disponibilité SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) avec ajout d’un lien vers la documentation Infrastructure de cloud public SUSE 101 
- 10/06/2020 : Ajout de nouvelles références SKU HLI dans [Références SKU disponibles pour HLI](./hana-available-skus.md) et [Architecture de stockage SAP HANA (Grandes instances)](./hana-storage-architecture.md)
- 21/05/2020 : Modification de la [Configuration de Pacemaker sur SLES dans Azure](./high-availability-guide-suse-pacemaker.md) et de la [Configuration de Pacemaker sur RHEL dans Azure](./high-availability-guide-rhel-pacemaker.md) pour ajouter un lien à la [Connectivité du point de terminaison public pour les machines virtuelles utilisant Azure Standard ILB dans les scénarios SAP HA](./high-availability-guide-standard-load-balancer-outbound-connections.md)  
- 19/05/2020 : Ajout d’un message important signifiant de ne pas utiliser un groupe de volumes racine lors de l’utilisation de LVM pour les volumes associés à HANA dans [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md)
- 19/05/2020 : Ajout d’un système d’exploitation pris en charge pour HANA – Grandes instances Type II dans [Systèmes d’exploitation compatibles pour HANA – Grandes instances](/- azure/virtual-machines/workloads/sap/os-compatibility-matrix-hana-large-instance)
- 12/05/2020 : Modification de [Connectivité du point de terminaison public pour les machines virtuelles utilisant Azure Standard ILB dans les scénarios SAP HA](./high-availability-guide-standard-load-balancer-outbound-connections.md) pour mettre à jour des liens et ajouter des informations pour la configuration de pare-feu tiers
- 11/05/2020 : Modification de [Haute disponibilité de SAP HANA sur des machines virtuelles Azure sur SLES](./sap-hana-high-availability.md) pour définir l’adhérence de ressource sur 0 pour la ressource netcat, car cela simplifie le basculement 
- 05/05/2020 : Modifications apportées à [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md) pour exprimer la disponibilité des déploiements Gen2 pour la famille de machines virtuelles Mv1
- 24/04/2020 : Modifications apportées à [Scale-out SAP HANA avec le nœud de secours sur des machines virtuelles Azure à l’aide d’ANF sur SLES](./sap-hana-scale-out-standby-netapp-files-suse.md), à [Scale-out SAP HANA avec le nœud de secours sur des machines virtuelles Azure à l’aide d’ANF sur RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md), à [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur SLES avec ANF](./high-availability-guide-suse-netapp-files.md) et à [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur RHEL avec ANF](./high-availability-guide-rhel-netapp-files.md) pour préciser que les adresses IP des volumes ANF sont attribuées automatiquement
- 22/04/2020 : Modification apportée à [Haute disponibilité de SAP HANA sur des machines virtuelles Azure sur SLES](./sap-hana-high-availability.md) pour supprimer l’attribut méta `is-managed` des instructions, en raison d’un conflit avec le placement du cluster en mode de maintenance ou sa sortie de ce mode
- 21/04/2020 : Ajout de SQL Azure DB en tant que système SGBD pris en charge pour SAP (Hybris) Commerce Platform 1811 et versions ultérieures dans les articles [Logiciels SAP dont le déploiement est pris en charge sur Azure](./sap-supported-product-on-azure.md) et [Certifications et configurations SAP en cours sur Microsoft Azure](./sap-certifications.md)
- 16/04/2020 : Ajout de SAP HANA en tant que système SGBD pris en charge pour SAP (Hybris) Commerce Platform dans les articles [Logiciels SAP dont le déploiement est pris en charge sur Azure](./sap-supported-product-on-azure.md) et [Certifications et configurations SAP en cours sur Microsoft Azure](./sap-certifications.md)
- 13/04/2020 : Correction des numéros de version SAP ASE dans [Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP](./dbms_guide_sapase.md)
- 07/04/2020 : Modification apportée à [Configuration de Pacemaker sur SLES dans Azure](./high-availability-guide-suse-pacemaker.md) afin de clarifier les instructions cloud-netconfig-Azure
- 06/04/2020 : Modifications apportées à [Scale-out de SAP HANA avec le nœud en attente sur les machines virtuelles Azure avec Azure NetApp Files sur SLES](./sap-hana-scale-out-standby-netapp-files-suse.md) et à [Scale-out de SAP HANA avec le nœud en attente sur les machines virtuelles Azure avec Azure NetApp Files sur RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md) pour supprimer les références à NetApp [TR-4435](https://www.netapp.com/us/media/tr-4746.pdf) (remplacé pr [TR-4746](https://www.netapp.com/us/media/tr-4746.pdf))
- 31/03/2020 : Modifications apportées à [Haute disponibilité de SAP HANA sur des machines virtuelles Azure sur SLES](./sap-hana-high-availability.md) et [Haute disponibilité de SAP HANA sur les machines virtuelles Azure qui exécutent RHEL](./sap-hana-high-availability-rhel.md) pour ajouter des instructions sur la façon de spécifier la taille des bandes lors de la création de volumes agrégés par bandes
- 27/03/2020 : Modification apportée à [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md) afin d’aligner les options de montage du système de fichiers sur NetApp TR-4746 (suppression de l’option de montage de synchronisation)
- 26/03/2020 : Modification apportée à [Guide de haute disponibilité multi-SID pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse-multi-sid.md) pour ajouter une référence à NetApp TR-4746
- 26/03/2020 : Modification apportée à [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md), [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md), [Haute disponibilité pour NFS sur des machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md), [Guide de haute disponibilité multi-SID pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse-multi-sid.md), [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Red Hat Enterprise Linux pour les applications SAP](./high-availability-guide-rhel.md) et [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Red Hat Enterprise Linux avec Azure NetApp Files pour les applications SAP](./high-availability-guide-rhel-netapp-files.md) pour modifier les diagrammes et clarifier les instructions pour la création d’un pool principal Azure Load Balancer
- 19/03/2020 : Révision majeure du document intitulé [Guide de démarrage rapide : installation manuelle de SAP HANA à une instance sur des machines virtuelles Azure](./hana-get-started.md) pour le remplacer par [installation de SAP HANA sur des machines virtuelles Azure](./hana-get-started.md)
- 17/03/2020 : Modification de [Configuration de Pacemaker sur SUSE Linux Enterprise Server dans Azure](./high-availability-guide-suse-pacemaker.md) pour supprimer le paramètre de configuration SBD qui n’est plus nécessaire
- 16/03/2020 : Clarification du scénario de certification de colonne dans la plateforme certifiée IaaS SAP HANA dans [Logiciels SAP pris en charge pour les déploiements Azure](./sap-supported-product-on-azure.md)
- 11/03/2020 : Modification de [Charge de travail SAP sur les machines virtuelles Azure - Scénarios pris en charge](./sap-planning-supported-configurations.md) pour clarifier la prise en charge de plusieurs bases de données par instance DBMS
- 11/03/2020 : Modification de [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md) pour décrire les machines virtuelles de première et deuxième génération
- 10/03/2020 : Modification de [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md) pour clarifier les limites de débit existantes d'ANF
- 09/03/2020 : Modification de [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md), [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md), [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md), [Configuration de Pacemaker sur SUSE Linux Enterprise Server dans Azure](./high-availability-guide-suse-pacemaker.md), [Haute disponibilité d’IBM Db2 LUW sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Pacemaker](./dbms-guide-ha-ibm.md), [Haute disponibilité de SAP HANA sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./sap-hana-high-availability.md) et [Guide multi-SID de haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](./high-availability-guide-suse-multi-sid.md) pour mettre à jour les ressources du cluster avec l’agent de ressources azure-lb 
- 05/03/2020 : Modifications de structure et de contenu pour les régions Azure et les machines virtuelles Azure dans [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md)
- 03/03/2020 : Modification apportée à [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md) pour améliorer la disposition des volumes ANF
- 01/03/2020 : Remaniement du [Guide de sauvegarde pour SAP HANA sur les machines virtuelles Azure](./sap-hana-backup-guide.md) pour inclure le service de Sauvegarde Azure. Contenu de [Sauvegarde SAP HANA sur Azure au niveau fichier](./sap-hana-backup-file-level.md) réduit et condensé et suppression d’un troisième document traitant de la sauvegarde avec capture instantanée de disque. Ce contenu est traité dans le Guide de sauvegarde pour SAP HANA sur les machines virtuelles Azure. 
- 27 février 2020 : Modification apportée à [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md), [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md) et [Guide de haute disponibilité multi-SID pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse-multi-sid.md) pour ajuster le paramètre de cluster « on-fail »
- 26 février 2020 : Modification de [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md) pour clarifier le choix du système de fichiers pour HANA sur Azure
- 26 février 2020 : Modification apportée à [Scénarios et architecture de haute disponibilité pour SAP](./sap-high-availability-architecture-scenarios.md) pour inclure le lien vers le Guide multi-SID de haute disponibilité des machines virtuelles Azure pour SAP NW sur RHEL
- 26 février 2020 : Modification apportée à [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md), [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md), [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux](./high-availability-guide-rhel.md) et [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files](./high-availability-guide-rhel-netapp-files.md) pour supprimer la remarque indiquant que le cluster multi-SID ASCS/ERS n’est pas pris en charge
- 26 février 2020 : Version du [Guide multi-SID de haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur Red Hat Enterprise Linux](./high-availability-guide-rhel-multi-sid.md) avec ajout d’un lien vers le guide du cluster multi-SID SUSE
- 25/02/2020 : Modification apportée à [Scénarios et architecture de haute disponibilité pour SAP NetWeaver](./sap-high-availability-architecture-scenarios.md) afin d’ajouter des liens vers des articles plus récents sur la haute disponibilité
- 25 février 2020 : Modification apportée à [Haute disponibilité d’IBM Db2 LUW sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Pacemaker](./dbms-guide-ha-ibm.md) pour pointer vers un document décrivant l’accès au point de terminaison public avec Azure Load Balancer standard
- 21 février 2020 : Révision complète de l’article [Déploiement SGBD de machines virtuelles SAP ASE Azure pour charge de travail SAP](./dbms_guide_sapase.md)
- 21 février 2020 : Modification apportée à [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md) pour représenter une nouvelle recommandation dans la taille de bande pour /hana/data et ajout du paramètre du planificateur d’E/S
- 21 février 2020 : Modifications apportées aux documents sur les grandes instances HANA pour représenter les références SKU S224 et S224m nouvellement certifiées
- 21 février 2020 : Modification apportée à [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux](./high-availability-guide-rhel.md) et à [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files pour les applications SAP](./high-availability-guide-rhel-netapp-files.md) afin d’ajuster les contraintes de cluster pour l’architecture de réplication du serveur d’empilage 2 (ENSA2)
- 20 février 2020 : Modification apportée au [Guide de haute disponibilité multi-SID pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse-multi-sid.md) pour ajouter un lien au guide de clusters multi-SID SUSE
- 13 février 2020 : Modifications apportées à [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver](./planning-guide.md) pour implémenter des liens vers de nouveaux documents
- 13 février 2020 : Ajout d’un nouveau document [Charge de travail SAP sur les machines virtuelles Azure - Scénarios pris en charge](./sap-planning-supported-configurations.md)
- 13 février 2020 : Ajout d’un nouveau document [Quels sont les logiciels SAP pris en charge pour le déploiement Azure](./sap-supported-product-on-azure.md)
- 13 février 2020 : Modification apportée à [Haute disponibilité d’IBM Db2 LUW sur les machines virtuelles Azure sur Red Hat Enterprise Linux Server](./high-availability-guide-rhel-ibm-db2-luw.md) pour pointer vers un document décrivant l’accès au point de terminaison public avec Azure Load Balancer standard
- 13 février 2020 : Ajout des nouveaux types de machines virtuelles à [Certifications et configurations SAP en cours sur Microsoft Azure](./sap-certifications.md)
- 13 février 2020 : Ajout de nouvelles notes de support SAP [Check-list relative à la planification et au déploiement de la charge de travail SAP sur Azure](./sap-deployment-checklist.md)
- 13 février 2020 : Modification de la [haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur RHEL](./high-availability-guide-rhel.md) et de la [haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur RHEL avec Azure NetApp Files](./high-availability-guide-rhel-netapp-files.md) pour aligner les délais d’attente des ressources de cluster sur ceux suggérés pour Red Hat
- 11 février 2020 : Publication de [Migration de grandes instances SAP HANA sur Azure vers des machines virtuelles Azure](./hana-large-instance-virtual-machine-migration.md)
- 7 février 2020 : Modification de la [connectivité du point de terminaison public pour les machines virtuelles utilisant Azure Standard ILB dans les scénarios SAP HA](./high-availability-guide-standard-load-balancer-outbound-connections.md) afin de mettre à jour l'exemple de capture d'écran NSG
- 3 février 2020 : Modification dans [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP](./high-availability-guide-suse.md) et [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md) pour supprimer l’avertissement relatif à l’utilisation d’un tiret dans les noms d’hôtes des nœuds de cluster sur SLES
- 28 janvier 2020 : Modification de la [haute disponibilité de SAP HANA sur les machines virtuelles Azure sous RHEL](./sap-hana-high-availability-rhel.md) pour aligner les délais d’expiration des ressources de cluster SAP HANA sur les recommandations de Red Hat en matière de délai d’attente
- 17 janvier 2020 : Modifications apportées à la rubrique [Groupes de placements de proximité Azure pour une latence réseau optimale avec les applications SAP](./sap-proximity-placement-scenarios.md) afin de modifier la section concernant le déplacement des machines virtuelles existantes vers un groupe de placements de proximité
- 17 janvier 2020 : Modifications apportées à la rubrique [Configurations de la charge de travail SAP avec des zones de disponibilité Azure](./sap-ha-availability-zones.md) afin de diriger le lecteur vers une procédure qui automatise les mesures de latence entre les zones de disponibilité
- 16 janvier 2020 : Modifications apportées à la rubrique [Guide pratique d’installation et de configuration de SAP HANA (grandes instances) sur Azure](./hana-installation.md) afin d’adapter les versions de système d’exploitation au répertoire du matériel IaaS HANA
- 16 janvier 2020 : Modifications apportées à la rubrique [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure](./high-availability-guide-suse-multi-sid.md) afin d’ajouter des instructions concernant les systèmes SAP et l’utilisation de l’architecture ENSA2 (Enqueue Server 2)
- 10 janvier 2020 : Modifications apportées à [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur SLES](./sap-hana-scale-out-standby-netapp-files-suse.md) et à [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md) pour ajouter des instructions sur la façon de rendre les modifications `nfs4_disable_idmapping` permanentes.
- 10 janvier 2020 : Modifications apportées à [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur SLES avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md) et à [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur RHEL avec Azure NetApp Files pour les applications SAP](./high-availability-guide-rhel-netapp-files.md) pour ajouter des instructions sur le montage des volumes NFSv4 d’Azure NetApp Files.
- 23 décembre 2019 : Publication du [Guide de haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur SLES multi-SID](./high-availability-guide-suse-multi-sid.md)
- 18 décembre 2019 : Publication de [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur RHEL](./sap-hana-scale-out-standby-netapp-files-rhel.md)
- 21 novembre 2019 : Modifications apportées dans [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md) pour simplifier la configuration du mappage d’ID NFS et changer l’interface réseau principale recommandée afin de simplifier le routage.
- 15 novembre 2019 : Modifications mineures apportées dans [Haute disponibilité pour SAP NetWeaver sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](high-availability-guide-suse-netapp-files.md) et [Haute disponibilité pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files pour les applications SAP](high-availability-guide-rhel-netapp-files.md) pour clarifier les restrictions de taille du pool de capacité et supprimer la déclaration stipulant que seule la version NFSv3 est prise en charge.
- 12 novembre 2019 : Publication de [Haute disponibilité pour SAP NetWeaver sur Windows avec Azure NetApp Files (SMB)](high-availability-guide-windows-netapp-files-smb.md)
- 8 novembre 2019 : Modifications apportées à [Haute disponibilité de SAP HANA sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](sap-hana-high-availability.md), [Configurer la réplication système SAP HANA sur les machines virtuelles Azure](sap-hana-high-availability-rhel.md), [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur SUSE Linux Enterprise Server pour les applications SAP](high-availability-guide-suse.md), [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur SUSE Linux Enterprise Server avec Azure NetApp Files](high-availability-guide-suse-netapp-files.md), [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux](high-availability-guide-rhel.md), [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files](high-availability-guide-rhel-netapp-files.md), [Haute disponibilité pour NFS sur les machines virtuelles Azure sur SUSE Linux Enterprise Server](high-availability-guide-suse-nfs.md), [GlusterFS sur les machines virtuelles Azure sur Red Hat Enterprise Linux pour SAP NetWeaver](high-availability-guide-rhel-glusterfs.md) pour recommander l’équilibreur de charge standard Azure  
- 8 novembre 2019 : Modifications apportées à [Check-list relative à la planification et au déploiement de la charge de travail SAP sur Azure](sap-deployment-checklist.md) pour clarifier la recommandation de chiffrement  
- 4 novembre 2019 : Modifications apportées à la [configuration de Pacemaker sur SUSE Linux Enterprise Server dans Azure](high-availability-guide-suse-pacemaker.md) pour créer le cluster directement avec la configuration de monodiffusion  
