---
title: Tableau de prise en charge de sauvegarde SAP HANA
description: Dans cet article,découvrez les scénarios et limitations pris en charge lorsque vous utilisez Sauvegarde Azure pour sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure.
ms.topic: conceptual
ms.date: 11/7/2019
ms.openlocfilehash: 19d62fac6a7072018508808b1c9695eb8b260b15
ms.sourcegitcommit: 1e6c13dc1917f85983772812a3c62c265150d1e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2020
ms.locfileid: "86170613"
---
# <a name="support-matrix-for-backup-of-sap-hana-databases-on-azure-vms"></a>Matrice de prise en charge pour la sauvegarde des bases de données SAP HANA sur des machines virtuelles Azure

Le service Sauvegarde Azure prend en charge la sauvegarde des bases de données SAP HANA sur Azure. Cet article résume les scénarios pris en charge et les limitations qui se présentent lorsque vous utilisez Sauvegarde Azure pour sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure.

> [!NOTE]
> La fréquence de sauvegarde de fichier journal peut désormais être définie sur un minimum de 15 minutes. Les sauvegardes de fichiers journaux ne commencent à s’effectuer qu’en cas de réussite d’une sauvegarde complète de la base de données.

## <a name="scenario-support"></a>Prise en charge du scénario

| **Scénario**               | **Configurations prises en charge**                                | **Configurations non prises en charge**                              |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Topologie**               | SAP HANA s’exécutant sur des machines virtuelles Azure Linux uniquement                    | Grandes instances HANA (HLI)                                   |
| **Régions**                   | **Disponibilité générale :**<br> **Amériques**  – USA Centre, USA Est 2, USA Est, USA Centre Nord, USA Centre Sud, USA Ouest 2, USA Centre-Ouest, USA Ouest, Canada Centre, Canada Est, Brésil Sud <br> **Asie-Pacifique** – Australie Centre, Australie Centre 2, Australie Est, Australie Sud-Est, Japon Est, Japon Ouest, Corée Centre, Corée Sud, Asie Est, Asie Sud-Est, Inde Centre, Inde Sud, Inde Ouest, Chine Est, Chine Nord, Chine Est 2, Chine Nord 2 <br> **Europe** – Europe Ouest, Europe Nord, France Centre, Royaume-Uni Sud, Royaume-Uni Ouest, Allemagne Nord, Allemagne Centre-Ouest, Suisse Nord, Suisse Ouest, Suisse Centre Nord, Norvège Est, Norvège Ouest <br> **Moyen-Orient et Afrique** – Afrique du Sud Nord, Afrique du Sud Ouest, Émirats arabes unis Nord, Émirats arabes unis Centre  <BR>  **Régions Azure Government** | France Sud, Allemagne Centre, Allemagne Nord-Est, US Gov IOWA |
| **Versions du système d’exploitation**            | SLES 12 avec SP2, SP3 et SP4 ; SLES 15 avec SP0 et SP1 <br><br>   **Préversion** - RHEL 7.4, 7.6, 7.7 et 8.1  <br>     [Bien démarrer](https://docs.microsoft.com/azure/backup/tutorial-backup-sap-hana-db) avec la sauvegarde SAP HANA en préversion pour RHEL (7.4, 7.6, 7.7 et 8.1). Si vous avez d’autres questions, écrivez-nous à l’adresse [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com).                |                                             |
| **Versions HANA**          | SDC sur HANA 1.x, MDC sur HANA 2.x <= SPS04 Rev 48, SPS05 (en attente de validation pour les scénarios de chiffrement activés)      |                                                            |
| **Déploiements HANA**       | SAP HANA sur une machine virtuelle Azure unique - Scale-up uniquement. <br><br> Pour les déploiements à haute disponibilité, les deux nœuds présents sur les deux ordinateurs sont traités individuellement avec des chaînes de données distinctes.               | Montée en charge <br><br> Dans les déploiements à haute disponibilité, la sauvegarde ne bascule pas automatiquement vers le nœud secondaire. La configuration de la sauvegarde doit être effectuée séparément pour chaque nœud.                                           |
| **Instances HANA**         | Une seule SAP HANA instance sur une seule machine virtuelle Azure - montée en puissance uniquement | Plusieurs instances de SAP HANA sur une seule machine virtuelle                  |
| **Types de bases de données HANA**    | Conteneur de base de données unique (SDC) sur 1.x, conteneur de bases de données multiples (MDC) sur 2.x | MDC dans HANA 1. x                                              |
| **Taille de la base de données HANA**     | Bases de données HANA de taille <= 2 To (il ne s’agit pas de la taille de la mémoire du système HANA)               |                                                              |
| **Types de sauvegarde**           | Sauvegardes complètes, différentielles et de fichier journal                          | Incrémentielle, instantanés                                       |
| **Types de restaurations**          | Reportez-vous à la note SAP HANA [1642148](https://launchpad.support.sap.com/#/notes/1642148) pour connaître les types de restauration pris en charge |                                                              |
| **Limites de sauvegarde**          | Jusqu’à 2 To de taille de sauvegarde complète par instance de SAP HANA         |                                                              |
| **Configurations spéciales** |                                                              | SAP HANA + Dynamic Tiering <br>  Clonage via LaMa        |

------

>[!NOTE]
>Sauvegarde Azure ne s’ajuste pas automatiquement au changement d’heure lorsque vous sauvegardez une base de données SAP HANA qui s’exécute dans une machine virtuelle Azure.
>
>Modifiez la stratégie manuellement en fonction des besoins.


> [!NOTE]
> Vous pouvez maintenant [surveiller la sauvegarde et restaurer](https://docs.microsoft.com/azure/backup/sap-hana-db-manage#monitor-manual-backup-jobs-in-the-portal) (sur la même machine) les travaux déclenchés à partir de clients natifs HANA (SAP HANA Studio/Cockpit/DBA Cockpit) dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [Sauvegarder des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](https://docs.microsoft.com/azure/backup/backup-azure-sap-hana-database)
* Découvrez comment [Restaurer des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](https://docs.microsoft.com/azure/backup/sap-hana-db-restore)
* Découvrez comment [gérer des bases de données SAP HANA sauvegardées à l’aide de Sauvegarde Azure](sap-hana-db-manage.md)
* Découvrez comment [résoudre les problèmes courants lors de la sauvegarde de bases de données SAP HANA](https://docs.microsoft.com/azure/backup/backup-azure-sap-hana-database-troubleshoot)
