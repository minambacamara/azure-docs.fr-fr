---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/26/2019
ms.author: glenga
ms.openlocfilehash: 4ace70abe0112e0fe27d177c02bcb697746c92cc
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/26/2020
ms.locfileid: "78261948"
---
### <a name="set-the-storage-account-connection"></a>Définir la connexion de compte de stockage

Ouvrez le fichier local.settings.json et copiez la valeur d’`AzureWebJobsStorage`, qui est la chaîne de connexion de compte de stockage. Définissez la variable d’environnement `AZURE_STORAGE_CONNECTION_STRING` sur la chaîne de connexion à l’aide de la commande Bash suivante :

```bash
AZURE_STORAGE_CONNECTION_STRING="<STORAGE_CONNECTION_STRING>"
```

Lorsque vous définissez la chaîne de connexion dans la variable d’environnement `AZURE_STORAGE_CONNECTION_STRING`, vous pouvez accéder à votre compte de stockage sans devoir vous authentifier à chaque fois.