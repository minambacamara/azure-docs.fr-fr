---
title: Se connecter à Windows Virtual Desktop, automne 2019, Windows 10 ou 7 – Azure
description: Guide pratique pour se connecter à Windows Virtual Desktop à l’aide du client Windows Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 07/10/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 78c400ffc6a84696a82c219294f133b729754259
ms.sourcegitcommit: dabd9eb9925308d3c2404c3957e5c921408089da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/11/2020
ms.locfileid: "86261434"
---
# <a name="connect-with-the-windows-desktop-client"></a>Se connecter avec le client Windows Desktop

> S’applique à : Windows 7, Windows 10 et Windows 10 IoT Entreprise

>[!IMPORTANT]
>Ce contenu s’applique à la version Automne 2019 qui ne prend pas en charge les objets Azure Resource Manager Windows Virtual Desktop. Si vous essayez de gérer les objets Azure Resource Manager Windows Virtual Desktop introduits dans la mise à jour Printemps 2020, consultez [cet article](../connect-windows-7-10.md).

Vous pouvez accéder aux ressources Windows Virtual Desktop sur des appareils Windows 7, Windows 10 ou Windows 10 IoT Entreprise à l’aide du client Bureau Windows.

>[!NOTE]
>Le client Windows passe automatiquement par défaut à la version Automne 2019 de Windows Virtual Desktop. Toutefois, si le client détecte que l’utilisateur a également des ressources Azure Resource Manager, il ajoute automatiquement les ressources ou avertit l’utilisateur qu’elles sont disponibles.

> [!IMPORTANT]
> Windows Virtual Desktop ne prend pas en charge le client RADC (Connexions RemoteApp et Bureau à distance) ni le client MSTSC (Connexion Bureau à distance).

> [!IMPORTANT]
> Windows Virtual Desktop ne prend pas actuellement en charge le client Bureau à distance à partir du Microsoft Store.

## <a name="install-the-windows-desktop-client"></a>Installer le client Windows Desktop

Choisissez le client qui correspond à votre version de Windows :

- [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2068602)
- [Windows 32 bits](https://go.microsoft.com/fwlink/?linkid=2098960)
- [Windows ARM64](https://go.microsoft.com/fwlink/?linkid=2098961)

Vous pouvez installer le client pour l’utilisateur actuel, ce qui ne nécessite pas de droits d’administrateur, ou votre administrateur peut installer et configurer le client afin que tous les utilisateurs de l’appareil puissent y accéder.

Une fois installé, le client peut être lancé à partir du menu Démarrer quand vous recherchez **Bureau à distance**.

## <a name="subscribe-to-a-workspace"></a>S'abonner à un espace de travail

Il y deux manières de vous abonner à un espace de travail. Le client peut essayer de découvrir les ressources disponibles à partir de votre compte professionnel ou scolaire, ou vous pouvez spécifier directement l’URL de vos ressources dans les cas où le client ne parvient pas à les trouver. Une fois que vous êtes abonné à un espace de travail, vous pouvez lancer des ressources avec l’une des méthodes suivantes :

- Accédez au centre de connexion, puis double-cliquez sur une ressource pour la lancer.
- Vous pouvez également accéder au menu Démarrer et rechercher un dossier avec le nom de l’espace de travail ou entrer le nom de la ressource dans la barre de recherche.

### <a name="subscribe-with-a-user-account"></a>S’abonner avec un compte d’utilisateur

1. Dans la page principale du client, sélectionnez **S’abonner**.
2. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.
3. Les ressources s’affichent dans le centre de connexion et sont regroupées par espace de travail.

### <a name="subscribe-with-a-url"></a>S’abonner avec une URL

1. Dans la page principale du client, sélectionnez **S’abonner avec une URL**.
2. Entrez l’URL de l’espace de travail ou votre adresse e-mail :
   - Si vous utilisez une **URL d’espace de travail**, utilisez celle que votre administrateur vous a donnée. Si vous accédez à des ressources à partir de Windows Virtual Desktop, vous pouvez utiliser l’une des URL suivantes :
     - Windows Virtual Desktop, automne 2019 : `https://rdweb.wvd.microsoft.com/api/feeddiscovery/webfeeddiscovery.aspx`
     - Windows Virtual Desktop, printemps 2020 : `https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery`
   - Si vous utilisez le champ **E-mail** à la place, entrez votre adresse e-mail. Cela indique au client de rechercher une URL associée à votre adresse e-mail si votre administrateur a configuré la [détection d’e-mails](/windows-server/remote/remote-desktop-services/rds-email-discovery).
3. Sélectionnez **Suivant**.
4. Connectez-vous avec votre compte d’utilisateur quand vous y êtes invité.
5. Les ressources devraient s’afficher dans le centre de connexion, regroupées par espace de travail.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du client Windows Desktop, consultez [Bien démarrer avec le client Windows Desktop](/windows-server/remote/remote-desktop-services/clients/windowsdesktop/).
