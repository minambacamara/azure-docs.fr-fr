---
title: Superviser votre appareil Azure Data Box Gateway | Microsoft Docs
description: Décrit comment utiliser le portail Azure et l’interface utilisateur web locale pour superviser votre appareil Azure Data Box Gateway.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: how-to
ms.date: 04/15/2019
ms.author: alkohli
ms.openlocfilehash: e352c2d8141f90c4dc721fe4fc0e717dfad7b88b
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "84339040"
---
# <a name="monitor-your-azure-data-box-gateway"></a>Superviser votre appareil Azure Data Box Gateway

Cet article explique comment superviser votre appareil Azure Data Box Gateway. Pour ce faire, vous pouvez utiliser le portail Azure ou l’interface utilisateur web locale. Utilisez le portail Azure pour voir les événements de l’appareil, configurer et gérer les alertes et voir les métriques.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Afficher les événements de l’appareil et les alertes correspondantes
> * Afficher les métriques de capacité et de transaction de votre appareil
> * Configurer et gérer les alertes

## <a name="view-device-events"></a>Afficher les événements de l’appareil

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-view-device-events.md)]

## <a name="view-metrics"></a>Afficher les mesures

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-view-metrics.md)]

### <a name="metrics-on-your-device"></a>Métriques sur votre appareil

Cette section décrit les métriques de surveillance sur votre appareil. Les métriques peuvent être :

* Métriques de capacité. Les métriques de capacité sont liées à la capacité de l’appareil.

* Métriques de transaction. Les métriques de transaction sont liées aux opérations de lecture et d’écriture dans Stockage Azure.

La liste complète des métriques est présentée dans le tableau suivant :

|Métriques de capacité                     |Description  |
|-------------------------------------|-------------|
|**Capacité disponible**               | Fait référence à la taille des données qui peuvent être écrites sur l’appareil. En d’autres termes, il s’agit de la capacité qui peut être rendue disponible sur l’appareil. <br></br>Vous pouvez libérer la capacité de l’appareil en supprimant la copie locale des fichiers dont il existe une copie sur l’appareil ainsi que sur le cloud.        |
|**Capacité totale**                   | Fait référence au nombre total d’octets sur l’appareil disponibles pour l’écriture de données. On parle également de taille totale du cache local. <br></br> Vous pouvez désormais augmenter la capacité d’un appareil virtuel existant en ajoutant un disque de données. Ajoutez un disque de données par le biais de la gestion de l’hyperviseur pour la machine virtuelle, puis redémarrez celle-ci. Le pool de stockage local de l’appareil de passerelle se développe de manière à accueillir le disque de données nouvellement ajouté. <br></br>Pour plus d’informations, consultez [Ajouter un disque dur pour la machine virtuelle Hyper-V](https://www.youtube.com/watch?v=EWdqUw9tTe4). |

|Métriques de transaction              | Description         |
|-------------------------------------|---------|
|**Octets cloud chargés (appareil)**    | Somme de tous les octets chargés sur tous les partages de votre appareil        |
|**Octets cloud chargés (partage)**     | Octets chargés par partage. Il peut s’agir des ordinateurs suivants : <br></br> Moyenne, à savoir (somme de tous les octets chargés par partage/nombre de partages)  <br></br>Maximum, à savoir le nombre maximal d’octets chargés à partir d’un partage <br></br>Minimum, à savoir le nombre minimal d’octets chargés à partir d’un partage      |
|**Débit de téléchargement cloud (partage)**| Octets téléchargés par partage. Il peut s’agir des ordinateurs suivants : <br></br> Moyenne, à savoir (somme de tous les octets lus ou téléchargés sur un partage/nombre de partages) <br></br> Maximum, à savoir le nombre maximal d’octets téléchargés à partir d’un partage<br></br> Minimum, à savoir le nombre minimal d’octets téléchargés à partir d’un partage  |
|**Débit de lecture cloud**            | Somme de tous les octets lus à partir du cloud sur tous les partages de votre appareil     |
|**Débit de chargement cloud**          | Somme de tous les octets écrits dans le cloud sur tous les partages de votre appareil     |
|**Débit de chargement cloud (partage)**  | Somme de tous les octets écrits dans le cloud à partir d’un partage/nombre de partages, exprimée en valeur moyenne, maximale ou minimale par partage      |
|**Débit de lecture (réseau)**           | Inclut le débit réseau système pour tous les octets lus à partir du cloud. Cette vue peut inclure des données qui ne sont pas limitées aux partages. <br></br>En divisant, vous affichez le trafic sur toutes les cartes réseau de l’appareil. Cela inclut les cartes qui ne sont pas connectées ou activées.      |
|**Débit d’écriture (réseau)**       | Inclut le débit réseau système pour tous les octets écrits dans le cloud. Cette vue peut inclure des données qui ne sont pas limitées aux partages. <br></br>En divisant, vous affichez le trafic sur toutes les cartes réseau de l’appareil. Cela inclut les cartes qui ne sont pas connectées ou activées.          |

## <a name="manage-alerts"></a>Gérer les alertes

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-manage-alerts.md)]

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [gérer la bande passante](data-box-gateway-manage-bandwidth-schedules.md).
