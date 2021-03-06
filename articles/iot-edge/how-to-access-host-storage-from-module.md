---
title: Utiliser le stockage local d’un appareil IoT Edge à partir d’un module - Azure IoT Edge | Microsoft Docs
description: Utilisez des variables d’environnement et créez des options pour permettre à un module d’accéder au stockage local d’un appareil IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/18/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 079d5845917e63fadcf0466e5a744ed637d704ca
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75434519"
---
# <a name="give-modules-access-to-a-devices-local-storage"></a>Fournir à des modules l’accès au stockage local d’un appareil

En plus de stocker des données à l’aide des services de stockage Azure ou dans le stockage de conteneur de votre appareil, vous pouvez également dédier un stockage sur l’appareil hôte IoT Edge lui-même pour améliorer la fiabilité, en particulier lors d’une utilisation hors connexion.

## <a name="link-module-storage-to-device-storage"></a>Lier le stockage du module au stockage de l’appareil

Pour activer un lien entre le stockage du module et le stockage sur le système hôte, créez une variable d’environnement pour votre module qui pointe vers un dossier de stockage dans le conteneur. Utilisez ensuite les options de création pour lier ce dossier de stockage à un dossier situé sur la machine hôte.

Par exemple, si vous souhaitez permettre au hub IoT Edge de stocker des messages dans le stockage local de votre appareil et de les récupérer ultérieurement, vous pouvez configurer les variables d’environnement et les options de création dans le portail Azure, dans la section **Paramètres du runtime**.

1. Pour le hub IoT Edge et l’agent IoT Edge, ajoutez une variable d’environnement appelée **storageFolder** qui pointe vers un répertoire dans le module.
1. Pour le hub IoT Edge et l’agent IoT Edge, ajoutez des liaisons pour connecter un répertoire local sur la machine hôte à un répertoire du module. Par exemple :

   ![Ajouter des options de création et des variables d’environnement pour le stockage local](./media/how-to-access-host-storage-from-module/offline-storage.png)

Vous pouvez aussi configurer le stockage local directement dans le manifeste de déploiement. Par exemple :

```json
"systemModules": {
    "edgeAgent": {
        "settings": {
            "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
            "createOptions": {
                "HostConfig": {
                    "Binds":["<HostStoragePath>:<ModuleStoragePath>"]
                }
            }
        },
        "type": "docker",
        "env": {
            "storageFolder": {
                "value": "<ModuleStoragePath>"
            }
        }
    },
    "edgeHub": {
        "settings": {
            "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
            "createOptions": {
                "HostConfig": {
                    "Binds":["<HostStoragePath>:<ModuleStoragePath>"],
                    "PortBindings":{"5671/tcp":[{"HostPort":"5671"}],"8883/tcp":[{"HostPort":"8883"}],"443/tcp":[{"HostPort":"443"}]}}}
        },
        "type": "docker",
        "env": {
            "storageFolder": {
                "value": "<ModuleStoragePath>"
            }
        },
        "status": "running",
        "restartPolicy": "always"
    }
}
```

Remplacez `<HostStoragePath>` et `<ModuleStoragePath>` par le chemin de stockage de votre hôte et de votre module. Les deux valeurs doivent être des chemins absolus.

Par exemple, sur un système Linux `"Binds":["/etc/iotedge/storage/:/iotedge/storage/"]` signifie que le répertoire **/etc/iotedge/storage** sur votre système hôte est mappé au répertoire **/iotedge/stockage/** sur le conteneur. Sur un système Windows, pour prendre un autre exemple, `"Binds":["C:\\temp:C:\\contemp"]` signifie que le répertoire **C:\\temp** sur votre système hôte est mappé au répertoire **C:\\contemp** sur le conteneur.

Par ailleurs, sur les appareils Linux, assurez-vous que le profil utilisateur pour votre module dispose des autorisations de lecture, d’écriture et d’exécution nécessaires sur le répertoire système de l’ordinateur hôte. Pour revenir à l’exemple précédent permettant au hub IoT Edge de stocker des messages dans le stockage local de votre appareil, vous devez accorder des autorisations à son profil utilisateur, UID 1000. (L’agent IoT Edge fonctionne comme root, donc il n’a pas besoin d’autorisations supplémentaires.) Il existe plusieurs façons de gérer les autorisations de répertoire sur les systèmes Linux , notamment l’utilisation de `chown` pour modifier le propriétaire du répertoire, puis `chmod` pour modifier les autorisations, comme :

```bash
sudo chown 1000 <HostStoragePath>
sudo chmod 700 <HostStoragePath>
```

Vous trouverez plus de détails sur les options de création dans la [documentation de docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate).

## <a name="next-steps"></a>Étapes suivantes

Pour un autre exemple d’accès à l’hôte de stockage à partir d’un module, consultez [Stocker des données en périphérie avec le Stockage Blob Azure sur IoT Edge](how-to-store-data-blob.md).
