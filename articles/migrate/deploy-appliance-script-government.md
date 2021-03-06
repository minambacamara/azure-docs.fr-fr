---
title: Configurer une appliance Azure Migrate dans Azure Government
description: Apprenez à configurer une appliance Azure Migrate dans Azure Government
ms.topic: article
ms.date: 04/16/2020
ms.openlocfilehash: f0ebc882646b5ff3f62ddddf91cffc85cb5e0da6
ms.sourcegitcommit: d7008edadc9993df960817ad4c5521efa69ffa9f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86109975"
---
# <a name="set-up-an-appliance-in-azure-government"></a>Configurer une appliance dans Azure Government 

Suivez cet article afin de déployer une [appliance Azure Migrate](./migrate-appliance-architecture.md) sur des machines virtuelles VMware, des machines virtuelles Hyper-V et des serveurs physiques dans un cloud Azure Government. Vous devez exécuter un script pour créer l'appliance et vérifier qu'elle peut se connecter à Azure. Si vous souhaitez configurer une appliance dans le cloud public, suivez [cet article](deploy-appliance-script.md).


> [!NOTE]
> L'option de déploiement d'une appliance à l'aide d'un modèle (pour les machines virtuelles VMware et Hyper-V) n'est pas prise en charge dans Azure Government.


## <a name="prerequisites"></a>Prérequis

Le script configure l’appliance Azure Migrate sur une machine physique ou virtuelle existante.

- La machine faisant office d’appliance doit exécuter Windows Server 2016, avec 32 Go de mémoire, huit processeurs virtuels, environ 80 Go de stockage sur disque et un commutateur virtuel externe. Elle a besoin d’une adresse IP statique ou dynamique et d’un accès à Internet.
- Avant de déployer l’appliance, examinez en détail la configuration requise pour les [machines virtuelles VMware](migrate-appliance.md#appliance---vmware), les [machines virtuelles Hyper-V](migrate-appliance.md#appliance---hyper-v) et les [serveurs physiques](migrate-appliance.md#appliance---physical).
- N’exécutez pas le script sur une appliance de Azure Migrate existante.

## <a name="set-up-the-appliance-for-vmware"></a>Configurer l'appliance pour VMware

Afin de configurer l'appliance pour VMware, téléchargez un fichier zip à partir du portail Azure et extrayez son contenu. Exécutez le script PowerShell pour lancer l'application web de l'appliance. Installez l'appliance et configurez-la pour la première fois. Inscrivez ensuite l'appliance auprès du projet Azure Migrate.

### <a name="download-the-script"></a>Télécharger le script

1.  Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, cliquez sur **Découvrir**.
2.  Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec l’hyperviseur vSphere VMWare**.
3.  Cliquez sur **Télécharger** pour télécharger le fichier zip. 


### <a name="verify-file-security"></a>Vérifier la sécurité du fichier

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier compressé
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple : ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-VMWare-USGov.zip SHA256```

3. Vérifiez la dernière version de l’appliance et les valeurs de hachage :

    **Algorithme** | **Télécharger** | **SHA256**
    --- | --- | ---
    VMware (63,1 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2120300&clcid=0x409 ) | 3d5822038646b81f458d89d706832c0a2c0e827bfa9b0a55cc478eaf2757a4de


### <a name="run-the-script"></a>Exécuter le script

Le script effectue les opérations suivantes :

- Installe les agents et une application web.
- Installe des rôles Windows, à savoir le service d’activation Windows (WAS), IIS et PowerShell ISE.
- Télécharge et installe un module réinscriptible IIS. [Plus d’informations](https://www.microsoft.com/download/details.aspx?id=7435)
- Met à jour une clé de Registre (HKLM) avec des paramètres persistants pour Azure Migrate.
- Crée les fichiers journaux et de configuration comme suit :
    - **Fichiers config** : %ProgramData%\Microsoft Azure\Config
    - **Fichiers journaux** : %ProgramData%\Microsoft Azure\Logs

Pour exécuter le script :

1. Extrayez le fichier zip dans un dossier sur la machine qui doit héberger l’appliance. Veillez à ne pas exécuter le script sur une machine d'une appliance Azure Migrate existante.
2. Lancez PowerShell sur la machine avec un privilège administrateur (élevé).
3. Modifiez le répertoire PowerShell en sélectionnant le dossier contenant l’extraction du fichier zip téléchargé.
4. Exécutez le script **AzureMigrateInstaller.ps1** comme suit : ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-VMWare-USGov>AzureMigrateInstaller.ps1 ```
5. Une fois correctement exécuté, le script lance l’application web de l’appliance afin que vous puissiez configurer celle-ci. En cas de problèmes, consultez les journaux du script sous C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Vérifier l'accès

Vérifiez que l'appliance peut se connecter aux URL Azure des [clouds du secteur public](migrate-appliance.md#government-cloud-urls).


## <a name="set-up-the-appliance-for-hyper-v"></a>Configurer l'appliance pour Hyper-V

Afin de configurer l'appliance pour Hyper-V, téléchargez un fichier zip à partir du portail Azure et extrayez son contenu. Exécutez le script PowerShell pour lancer l'application web de l'appliance. Installez l'appliance et configurez-la pour la première fois. Inscrivez ensuite l'appliance auprès du projet Azure Migrate.

### <a name="download-the-script"></a>Télécharger le script

1.  Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, cliquez sur **Découvrir**.
2.  Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec Hyper-V**.
3.  Cliquez sur **Télécharger** pour télécharger le fichier zip. 


### <a name="verify-file-security"></a>Vérifier la sécurité du fichier

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier compressé
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple : ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-HyperV-USGov.zip SHA256```

3. Vérifiez la dernière version de l’appliance et la valeur de hachage :

    **Scénario** | **Télécharger** | **SHA256**
    --- | --- | ---
    Hyper-V (63,1 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2120200&clcid=0x409) |  2c5e73a1e5525d4fae468934408e43ab55ff397b7da200b92121972e683f9aa3

          

### <a name="run-the-script"></a>Exécuter le script

Le script effectue les opérations suivantes :

- Installe les agents et une application web.
- Installe des rôles Windows, à savoir le service d’activation Windows (WAS), IIS et PowerShell ISE.
- Télécharge et installe un module réinscriptible IIS. [Plus d’informations](https://www.microsoft.com/download/details.aspx?id=7435)
- Met à jour une clé de Registre (HKLM) avec des paramètres persistants pour Azure Migrate.
- Crée les fichiers journaux et de configuration comme suit :
    - **Fichiers config** : %ProgramData%\Microsoft Azure\Config
    - **Fichiers journaux** : %ProgramData%\Microsoft Azure\Logs

Pour exécuter le script :

1. Extrayez le fichier zip dans un dossier sur la machine qui doit héberger l’appliance. Veillez à ne pas exécuter le script sur une machine d'une appliance Azure Migrate existante.
2. Lancez PowerShell sur la machine avec un privilège administrateur (élevé).
3. Modifiez le répertoire PowerShell en sélectionnant le dossier contenant l’extraction du fichier zip téléchargé.
4. Exécutez le script **AzureMigrateInstaller.ps1** comme suit : ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-HyperV-USGov>AzureMigrateInstaller.ps1 ``` 
5. Une fois correctement exécuté, le script lance l’application web de l’appliance afin que vous puissiez configurer celle-ci. En cas de problèmes, consultez les journaux du script sous C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Vérifier l'accès

Vérifiez que l'appliance peut se connecter aux URL Azure des [clouds du secteur public](migrate-appliance.md#government-cloud-urls).


## <a name="set-up-the-appliance-for-physical-servers"></a>Configurer l'appliance pour des serveurs physiques

Afin de configurer l'appliance pour VMware, téléchargez un fichier zip à partir du portail Azure et extrayez son contenu. Exécutez le script PowerShell pour lancer l'application web de l'appliance. Installez l'appliance et configurez-la pour la première fois. Inscrivez ensuite l'appliance auprès du projet Azure Migrate.

### <a name="download-the-script"></a>Télécharger le script

1.  Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, cliquez sur **Découvrir**.
2.  Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Non virtualisé/Autre**.
3.  Cliquez sur **Télécharger** pour télécharger le fichier zip. 


### <a name="verify-file-security"></a>Vérifier la sécurité du fichier

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier compressé
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple : ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip SHA256```

3. Vérifiez la dernière version de l’appliance et la valeur de hachage :

    **Scénario** | **Télécharger*** | **Valeur de hachage**
    --- | --- | ---
    Physique (63,1 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2120100&clcid=0x409) | 93dfef131026e70acdfad2769cd208ff745ab96a96f013cdf3f9e1e61c9b37e1
          

### <a name="run-the-script"></a>Exécuter le script

Le script effectue les opérations suivantes :

- Installe les agents et une application web.
- Installe des rôles Windows, à savoir le service d’activation Windows (WAS), IIS et PowerShell ISE.
- Télécharge et installe un module réinscriptible IIS. [Plus d’informations](https://www.microsoft.com/download/details.aspx?id=7435)
- Met à jour une clé de Registre (HKLM) avec des paramètres persistants pour Azure Migrate.
- Crée les fichiers journaux et de configuration comme suit :
    - **Fichiers config** : %ProgramData%\Microsoft Azure\Config
    - **Fichiers journaux** : %ProgramData%\Microsoft Azure\Logs

Pour exécuter le script :

1. Extrayez le fichier zip dans un dossier sur la machine qui doit héberger l’appliance. Veillez à ne pas exécuter le script sur une machine d'une appliance Azure Migrate existante.
2. Lancez PowerShell sur la machine avec un privilège administrateur (élevé).
3. Modifiez le répertoire PowerShell en sélectionnant le dossier contenant l’extraction du fichier zip téléchargé.
4. Exécutez le script **AzureMigrateInstaller.ps1** comme suit : ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>AzureMigrateInstaller.ps1 ```
5. Une fois correctement exécuté, le script lance l’application web de l’appliance afin que vous puissiez configurer celle-ci. En cas de problèmes, consultez les journaux du script sous C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Vérifier l'accès

Vérifiez que l'appliance peut se connecter aux URL Azure des [clouds du secteur public](migrate-appliance.md#government-cloud-urls).

## <a name="next-steps"></a>Étapes suivantes

Après avoir déployé l'appliance, vous devez la configurer pour la première fois et l'inscrire auprès du projet Azure Migrate.

- Configurez l’appliance pour [VMware](how-to-set-up-appliance-vmware.md#configure-the-appliance).
- Configurez l’appliance pour [Hyper-V](how-to-set-up-appliance-hyper-v.md#configure-the-appliance).
- Configurez l’appliance pour les [serveurs physiques](how-to-set-up-appliance-physical.md).
