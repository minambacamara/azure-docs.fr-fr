---
title: Définir la stratégie de cookie du lecteur immersif
titleSuffix: Azure Cognitive Services
description: Cet article vous explique comment définir la stratégie de cookie pour le lecteur immersif.
services: cognitive-services
author: pasta
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 01/06/2020
ms.author: pasta
ms.custom: devx-track-javascript
ms.openlocfilehash: 6bc55416c335f370151e54d2b09ccf00bd799fdb
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406892"
---
# <a name="how-to-set-the-cookie-policy-for-the-immersive-reader"></a>Guide pratique pour définir la stratégie de cookie pour le lecteur immersif

Par défaut, le lecteur immersif désactive l’utilisation des cookies. Si vous activez l’utilisation des cookies, le lecteur immersif peut utiliser des cookies pour gérer les préférences de l’utilisateur et suivre l’utilisation des fonctionnalités. Si vous activez l’utilisation des cookies dans le lecteur immersif, tenez compte des exigences pour la stratégie de conformité des cookies de l’Union européenne. Il incombe à l’application hôte d’obtenir les éventuels consentements nécessaires de l’utilisateur conformément à la stratégie de conformité des cookies de l’Union européenne.

La stratégie de cookie peut être définie via les [options](../reference.md#options) du lecteur immersif.

## <a name="enable-cookie-usage"></a>Activer l’utilisation des cookies

```javascript
var options = {
    'cookiePolicy': ImmersiveReader.CookiePolicy.Enable
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="disable-cookie-usage"></a>Désactiver l’utilisation des cookies

```javascript
var options = {
    'cookiePolicy': ImmersiveReader.CookiePolicy.Disable
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="next-steps"></a>Étapes suivantes

* Consulter le [guide de démarrage rapide Node.js](../quickstarts/client-libraries.md?pivots=programming-language-nodejs) pour voir ce que vous pouvez faire d’autre avec le SDK Lecteur immersif pour Node.js
* Consulter le [tutoriel Android](../tutorial-android.md) pour voir ce que vous pouvez faire d’autre avec le SDK Lecteur immersif en utilisant Java ou Kotlin pour Android
* Consulter le [tutoriel iOS](../tutorial-ios.md) pour voir ce que vous pouvez faire d’autre avec le SDK Lecteur immersif en utilisant Swift pour iOS
* Consulter le [didacticiel Python](../tutorial-python.md) pour voir ce que vous pouvez faire d’autre avec le kit SDK Lecteur immersif à l’aide de Python
* Explorer le [SDK Lecteur Immersif](https://github.com/microsoft/immersive-reader-sdk) et la [référence du SDK Lecteur immersif](../reference.md)