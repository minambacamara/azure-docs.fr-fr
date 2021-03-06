---
title: Découvrir, évaluer et migrer des machines virtuelles Amazon Web Services (AWS) vers Azure
description: Cet article explique comment migrer des machines virtuelles AWS vers Azure avec Azure Migrate.
ms.topic: tutorial
ms.date: 06/16/2020
ms.custom: MVC
ms.openlocfilehash: 5d697c2146144ca7f4b9a8739b6863ba31845f4e
ms.sourcegitcommit: 1e6c13dc1917f85983772812a3c62c265150d1e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2020
ms.locfileid: "86165428"
---
# <a name="discover-assess-and-migrate-amazon-web-services-aws-vms-to-azure"></a>Découvrir, évaluer et migrer des machines virtuelles Amazon Web Services (AWS) vers Azure

Ce tutoriel vous montre comment découvrir, évaluer et migrer des machines virtuelles Amazon Web Services (AWS) vers des machines virtuelles Azure à l’aide des outils Azure Migrate : Server Assessment et Server Migration.

> [!NOTE]
> Quand vous migrez vos machines virtuelles AWS vers Azure, elles sont traitées comme s’il s’agissait de serveurs physiques. Vous allez utiliser le workflow Server Migration pour la migration des machines physiques, pour migrer vos machines virtuelles AWS vers Azure.

Dans ce didacticiel, vous apprendrez à :
> [!div class="checklist"]
> * Vérifier les prérequis pour la migration.
> * Préparer les ressources Azure avec Azure Migrate : Server Migration. Configurez des autorisations pour permettre à votre compte et vos ressources Azure de fonctionner avec Azure Migrate.
> * Préparer les instances AWS EC2 pour la migration.
> * Ajouter l’outil Azure Migrate : Server Migration au hub Azure Migrate.
> * Configurer l’appliance de réplication et déployer le serveur de configuration.
> * Installer le service Mobility sur chaque machine virtuelle AWS à migrer.
> * Activez la réplication des machines virtuelles.
> * Suivre et superviser l’état de la réplication. 
> * Exécuter une migration de test pour vérifier que tout fonctionne comme prévu.
> * Exécuter une migration complète vers Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.

## <a name="discover-and-assess-aws-vms"></a>Découvrir et évaluer des machines virtuelles AWS  

Avant de migrer vers Azure, nous vous recommandons d’effectuer une évaluation de la découverte et de la migration des machines virtuelles. Cette évaluation permet de dimensionner de façon appropriée vos machines virtuelles AWS pour la migration vers Azure et d’estimer les coûts d’exécution potentiels d’Azure.

Configurez une évaluation comme suit :

1. Une évaluation peut être effectuée en traitant vos machines virtuelles AWS comme des machines physiques dans le cadre d’une évaluation à l’aide de l’outil Azure Migrate : Server Assessment. Suivez le [tutoriel](./tutorial-prepare-physical.md) pour configurer Azure et préparer vos machines virtuelles AWS à une évaluation.
2. Ensuite, suivez ce [tutoriel](./tutorial-assess-physical.md) pour configurer un projet et une appliance Azure Migrate afin de découvrir et d’évaluer vos machines virtuelles AWS.

Même si nous vous conseillons d’effectuer une évaluation, celle-ci n’est pas une étape obligatoire pour pouvoir migrer des machines virtuelles.

## <a name="migrate-aws-vms"></a>Migrer des machines virtuelles AWS   

## <a name="1-prerequisites-for-migration"></a>1. Prérequis pour la migration

- Vérifiez que les machines virtuelles AWS que vous voulez migrer exécutent une version du système d’exploitation prise en charge. Les machines virtuelles AWS sont traitées comme des machines physiques dans le cadre de la migration. Passez en revue les [systèmes d’exploitation pris en charge](../site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines) pour le workflow de la migration de serveurs physiques. Nous vous recommandons d’effectuer un test de migration (test de basculement) pour vérifier si la machine virtuelle fonctionne comme prévu avant de procéder à la migration réelle.
- Vérifiez que vos machines virtuelles AWS sont conformes aux [configurations prises en charge](./migrate-support-matrix-physical-migration.md#physical-server-requirements) pour la migration vers Azure.
- Vérifiez que les machines virtuelles AWS que vous répliquez sur Azure respectent les [exigences des machines virtuelles Azure](./migrate-support-matrix-physical-migration.md#azure-vm-requirements).
- Certaines modifications doivent être apportées aux machines virtuelles pour pouvoir les migrer vers Azure.
    - Pour certains systèmes d’exploitation, Azure Migrate effectue ces modifications automatiquement.
    - Il est important d’effectuer ces modifications avant de commencer la migration. Si vous migrez la machine virtuelle avant d’effectuer la modification, elle risque de ne pas démarrer dans Azure.
Examinez les modifications à apporter aux machines [Windows](prepare-for-migration.md#windows-machines) et [Linux](prepare-for-migration.md#linux-machines).

## <a name="2-prepare-azure-resources-for-migration"></a>2. Préparer les ressources Azure à la migration

Préparez Azure à la migration avec l’outil Server Migration d’Azure Migrate.

**Tâche** | **Détails**
--- | ---
**Créer un projet Azure Migrate** | Votre compte Azure doit disposer d’autorisations Contributeur ou Propriétaire pour créer un projet.
**Vérifier les autorisations pour votre compte Azure** | Votre compte Azure a besoin d’autorisations pour créer une machine virtuelle et écrire sur un disque managé Azure.

### <a name="assign-permissions-to-create-project"></a>Attribuer des autorisations pour créer un projet

1. Dans le portail Azure, ouvrez l’abonnement, puis sélectionnez **Contrôle d’accès (IAM)** .
2. Dans **Vérifier l’accès**, recherchez le compte approprié, puis cliquez dessus pour voir les autorisations correspondantes.
3. Vous devez disposer des autorisations de **Contributeur** ou de **Propriétaire**.
    - Si vous venez de créer un compte Azure gratuit, vous êtes le propriétaire de votre abonnement.
    - Si vous n’êtes pas le propriétaire de l’abonnement, demandez au propriétaire de vous attribuer le rôle.

### <a name="assign-azure-account-permissions"></a>Affecter des autorisations de compte Azure

Affectez le rôle Contributeur de machines virtuelles au compte Azure. Cela permet d’effectuer les opérations suivantes :

- Créer une machine virtuelle dans le groupe de ressources sélectionné
- Créer une machine virtuelle dans le réseau virtuel sélectionné
- Écrire sur un disque managé Azure. 

### <a name="create-an-azure-network"></a>Créer un réseau Azure

[Configurez](../virtual-network/manage-virtual-network.md#create-a-virtual-network) un réseau virtuel Azure. Quand vous effectuez une réplication sur Azure, les machines virtuelles Azure créées sont jointes au réseau virtuel Azure que vous avez spécifié lors de la configuration de la migration.

## <a name="3-prepare-aws-instances-for-migration"></a>3. Préparer les instances AWS à la migration

Pour préparer AWS à la migration Azure, vous devez préparer et déployer une appliance de réplication pour la migration.

### <a name="prepare-a-machine-for-the-replication-appliance"></a>Préparer une machine pour l’appliance de réplication

Azure Migrate : Server Migration utilise une appliance de réplication pour répliquer les machines sur Azure. L’appliance de réplication exécute les composants suivants :

- **Serveur de configuration** : Le serveur de configuration coordonne la communication entre l’environnement AWS et Azure, puis gère la réplication des données.
- **Serveur de traitement**: Le serveur de processus fait office de passerelle de réplication. Il reçoit les données de réplication, les optimise grâce à une mise en cache, une compression et un chiffrement, puis les envoie à un compte de stockage de cache dans Azure.

Préparez le déploiement de l’appliance comme suit :

- Configurez une machine virtuelle EC2 distincte pour héberger l’appliance de réplication. Cette instance doit exécuter Windows Server 2012 R2 ou Windows Server 2016. [Passez en revue](./migrate-replication-appliance.md#appliance-requirements) la configuration matérielle, logicielle et réseau nécessaire pour l’appliance.
- L’appliance ne doit pas être installée sur une machine virtuelle source que vous voulez répliquer. Elle doit plutôt être déployée sur une autre machine virtuelle.
- Les machines virtuelles AWS sources à migrer doivent avoir l’appliance de réplication en ligne de mire. Configurez les règles de groupe de sécurité nécessaires à cette fin. Il est recommandé de déployer l’appliance de réplication sur le même VPC que les machines virtuelles sources à migrer. Si l’appliance de réplication a besoin de se trouver dans un autre VPC, les VPC doivent être connectés par peering VPC.
- Les machines virtuelles AWS sources communiquent avec l’appliance de réplication sur les ports HTTPS 443 (orchestration du canal de contrôle) et TCP 9443 (transport de données) entrants pour la gestion de la réplication et le transfert des données de réplication. L’appliance de réplication à son tour orchestre et envoie les données de réplication à Azure sur le port HTTPS 443 sortant. Pour configurer ces règles, modifiez les règles de trafic entrant/sortant du groupe de sécurité avec les ports et les informations d’adressage IP source appropriés.

   ![Groupes de sécurité AWS ](./media/tutorial-migrate-aws-virtual-machines/aws-security-groups.png)
     
 
   ![Modifier les paramètres de sécurité ](./media/tutorial-migrate-aws-virtual-machines/edit-security-settings.png)

- L’appliance de réplication utilise MySQL. Passez en revue les [options](migrate-replication-appliance.md#mysql-installation) d’installation de MySQL sur l’appliance.
- Passez en revue les URL Azure nécessaires à l’appliance de réplication pour accéder aux clouds [publics](migrate-replication-appliance.md#url-access) et du [secteur public](migrate-replication-appliance.md#azure-government-url-access).

## <a name="4-add-the-server-migration-tool"></a>4. Ajouter l’outil Migration de serveur

Configurez un projet Azure Migrate, puis ajoutez-y l’outil Migration de serveur.

1. Dans le portail Azure, sélectionnez **Tous les services**, puis recherchez **Azure Migrate**.
2. Sous **Services**, sélectionnez **Azure Migrate**.
3. Dans **Vue d’ensemble**, cliquez sur **Évaluer et migrer des serveurs**.
4. Sous **Découvrir, évaluer et migrer des serveurs**, cliquez sur **Évaluer et migrer des serveurs**.

    ![Découvrir et évaluer des serveurs](./media/tutorial-migrate-physical-virtual-machines/assess-migrate.png)

5. Dans **Découvrir, évaluer et migrer des serveurs**, cliquez sur **Ajouter des outils**.
6. Dans **Projet de migration**, sélectionnez votre abonnement Azure, puis créez un groupe de ressources si vous n’en avez pas.
7. Dans **Détails du projet**, spécifiez le nom du projet et la géographie dans laquelle vous souhaitez créer le projet, puis cliquez sur **Suivant**. Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et [gouvernementaux](migrate-support-matrix.md#supported-geographies-azure-government).
    - La géographie du projet sert uniquement à stocker les métadonnées rassemblées à partir des machines AWS.
    - Vous pouvez sélectionner n’importe quelle région cible quand vous exécutez une migration.

    ![Créer un projet Azure Migrate](./media/tutorial-migrate-physical-virtual-machines/migrate-project.png)

8. Dans **Sélectionner un outil d’évaluation**, sélectionnez **Ignorer l’ajout d’un outil d’évaluation pour l’instant** > **Suivant**.
9. Dans **Sélectionner un outil de migration**, sélectionnez **Azure Migrate : Server Migration** > **Suivant**.
10. Dans **Vérifier + ajouter des outils**, passez en revue les paramètres, puis cliquez sur **Ajouter des outils**.
11. L’outil ajouté apparaît alors dans le projet Azure Migrate > **Serveurs** > **Outils de migration**.

## <a name="5-set-up-the-replication-appliance"></a>5. Configurer l’appliance de réplication

La première étape de la migration consiste à configurer l’appliance de réplication. Pour configurer l’appliance pour la migration des machines virtuelles AWS, vous devez télécharger le fichier d’installation de l’appliance, puis l’exécuter sur la [machine virtuelle que vous avez préparée](#prepare-a-machine-for-the-replication-appliance).

### <a name="download-the-replication-appliance-installer"></a>Télécharger le programme d’installation de l’appliance de réplication

1. Dans le projet Azure Migrate > **Serveurs**, dans **Azure Migrate : Server Migration**, cliquez sur **Découvrir**.

    ![Détection des machines virtuelles](./media/tutorial-migrate-physical-virtual-machines/migrate-discover.png)

2. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , cliquez sur **Non virtualisé/autre**.
3. Dans **Région cible**, sélectionnez la région Azure vers laquelle vous souhaitez migrer les machines.
4. Sélectionnez **Confirmer que la région cible pour la migration est <nom_région>** .
5. Cliquez sur **Créer des ressources**. Un coffre Azure Site Recovery est créé en arrière-plan.
    - Si vous avez déjà configuré la migration avec Azure Migrate Server Migration, l’option Cible ne peut pas être configurée, car les ressources ont déjà été configurées.
    - Après avoir cliqué sur ce bouton, vous ne pouvez plus changer la région cible de ce projet.
    - Pour migrer vos machines virtuelles vers une autre région, vous devez créer un projet Azure Migrate différent.

6. Dans **Voulez-vous installer une nouvelle appliance de réplication ?** , sélectionnez **Installer une appliance de réplication**.
7. Dans **Télécharger et installer le logiciel de l’appliance de réplication**, téléchargez le programme d’installation de l’appliance et la clé d’inscription. Vous avez besoin de la clé pour inscrire l’appliance. Une fois téléchargée, la clé reste valide pendant 5 jours.

    ![Télécharger le fournisseur](media/tutorial-migrate-physical-virtual-machines/download-provider.png)

8. Copiez le fichier de configuration et le fichier de clé de l’appliance sur la machine virtuelle AWS Windows Server 2016 ou Windows Server 2012 que vous avez créée pour l’appliance de réplication.
9. Exécutez le fichier de configuration de l’appliance de réplication en suivant la procédure ci-dessous.  
    9.1. Dans **Avant de commencer**, sélectionnez **Installer le serveur de configuration et le serveur de processus**, puis sélectionnez **Suivant**.   
    9.2 Dans **Licence de logiciel tiers**, sélectionnez **J’accepte le contrat de licence tiers**, puis **Suivant**.   
    9.3 Dans **Inscription**, sélectionnez **Parcourir** et accédez à l’endroit où vous avez placé le fichier de clé d’inscription du coffre. Sélectionnez **Suivant**.  
    9.4 Dans **Paramètres Internet**, sélectionnez **Se connecter directement à Azure Site Recovery sans serveur proxy**, puis cliquez sur **Suivant**.  
    9.5 La page **Vérification de la configuration requise** effectue des vérifications pour plusieurs éléments. Quand cela est terminé, cliquez sur **Suivant**.  
    9.6 Dans **Configuration MySQL**, indiquez un mot de passe pour la base de données MySQL, puis sélectionnez **Suivant**.  
    9.7 Dans la page **Détails de l’environnement**, sélectionnez **Non**. Vous n’avez pas besoin de protéger vos machines virtuelles. Ensuite, sélectionnez **Suivant**.  
    9.8 Dans **Emplacement d’installation**, cliquez sur **Suivant** pour accepter la valeur par défaut.  
    9.9 Dans **Sélection du réseau**, cliquez sur **Suivant** pour accepter la valeur par défaut.  
    9.10 Dans **Résumé**, sélectionnez **Installer**.   
    9.11 **Progression de l’installation** vous montre des informations sur le processus d’installation. Quand cela est terminé, cliquez sur **Terminer**. Une fenêtre affiche un message concernant un redémarrage. Sélectionnez **OK**.   
    9.12 Ensuite, une fenêtre affiche un message concernant la phrase secrète de la connexion du serveur de configuration. Copiez la phrase secrète dans le Presse-papiers et enregistrez-la dans un fichier texte temporaire sur les machines virtuelles sources. Vous aurez besoin de cette phrase secrète plus tard, pendant le processus d’installation du service Mobility.
10. Une fois l’installation terminée, l’Assistant Configuration de l’appliance démarre automatiquement (vous pouvez également le lancer manuellement à l’aide du raccourci cspsconfigtool créé sur le Bureau de l’appliance). Utilisez l’onglet Gérer les comptes de l’Assistant pour ajouter les détails du compte à utiliser pour l’installation push du service Mobilité. Dans ce tutoriel, nous allons installer manuellement le service Mobility sur les machines virtuelles sources à répliquer. Avant de continuer, vous devez donc créer un compte factice.
11. Une fois l’appliance installée et redémarrée, dans **Découvrir des machines**, sélectionnez la nouvelle appliance dans **Sélectionner un serveur de configuration**, puis cliquez sur **Finaliser l’inscription**. L’option Finaliser l’inscription exécute quelques tâches finales pour préparer l’appliance de réplication.

    ![Finaliser l’inscription](./media/tutorial-migrate-physical-virtual-machines/finalize-registration.png)

## <a name="6-install-the-mobility-service"></a>6. Installer le service Mobilité

Un agent du service Mobility doit être installé sur les machines virtuelles AWS sources à migrer. Les programmes d’installation de l’agent sont disponibles dans l’appliance de réplication. Recherchez le programme d’installation qui convient, puis installez l’agent sur chaque machine devant faire l’objet d’une migration. Effectuez les étapes suivantes :

1. Connectez-vous à l’appliance de réplication.
2. Accédez au dossier **%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository**.
3. Recherchez le programme d’installation correspondant à la version du système d’exploitation des machines virtuelles AWS. Vérifiez quels sont les [systèmes d’exploitation pris en charge](../site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines).
4. Copiez le fichier du programme d’installation sur la machine virtuelle AWS qui doit faire l’objet d’une migration.
5. Vérifiez que vous avez bien le fichier texte de la phrase secrète créé lors de l’installation de l’appliance de réplication.
    - Si vous avez oublié d’enregistrer cette phrase secrète, vous pouvez l’afficher sur l’appliance de réplication en suivant cette étape. Sur la ligne de commande, exécutez **C:\ProgramData\ASR\home\svsystems\bin\genpassphrase.exe -v** pour afficher la phrase secrète.
    - Maintenant, copiez cette phrase secrète dans le Presse-papiers et enregistrez-la dans un fichier texte temporaire sur les machines virtuelles sources.

### <a name="installation-guide-for-windows-aws-vms"></a>Guide d’installation pour les machines virtuelles AWS Windows

1. Procédez de la façon suivante pour extraire le contenu du fichier du programme d’installation dans un dossier local (par exemple, C:\Temp) de la machine virtuelle AWS :

    ```
    ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
    MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
    cd C:\Temp\Extracted
    ```  

2. Exécutez le programme d’installation de Mobility Service :
    ```
   UnifiedAgent.exe /Role "MS" /Silent
    ```  

3. Inscrivez l’agent auprès de l’appliance de réplication :
    ```
    cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
    UnifiedAgentConfigurator.exe  /CSEndPoint <replication appliance IP address> /PassphraseFilePath <Passphrase File Path>
    ```

### <a name="installation-guide-for-linux-aws-vms"></a>Guide d’installation pour les machines virtuelles AWS Linux

1. Procédez de la façon suivante pour extraire le contenu du tarball du programme d’installation dans un dossier local (par exemple, /tmp/MobSvcInstaller) de la machine virtuelle AWS :
    ```
    mkdir /tmp/MobSvcInstaller
    tar -C /tmp/MobSvcInstaller -xvf <Installer tarball>
    cd /tmp/MobSvcInstaller
    ```  

2. Exécutez le script du programme d’installation :
    ```
    sudo ./install -r MS -q
    ```  

3. Inscrivez l’agent auprès de l’appliance de réplication :
    ```
    /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <replication appliance IP address> -P <Passphrase File Path>
    ```

## <a name="7-enable-replication-for-aws-vms"></a>7. Activer la réplication pour les machines virtuelles AWS

> [!NOTE]
> Par le biais du portail, vous pouvez ajouter jusqu’à 10 machines virtuelles à répliquer en même temps. Pour en répliquer davantage simultanément, vous pouvez les ajouter par lots de 10.

1. Dans le projet Azure Migrate > **Serveurs**, **Azure Migrate : Server Migration**, cliquez sur **Répliquer**.

    ![Répliquer des machines virtuelles](./media/tutorial-migrate-physical-virtual-machines/select-replicate.png)

2. Dans **Répliquer** > **Paramètres de la source** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Non virtualisé/autre**.
3. Dans **Appliance locale**, sélectionnez le nom de l’appliance Azure Migrate que vous avez configurée.
4. Dans **Process Server**, sélectionnez le nom de l’appliance de réplication.
6. Dans **Informations d’identification de l’invité**, spécifiez un compte factice qui sera utilisé pour installer manuellement le service Mobility (l’installation push n’est pas prise en charge). Cliquez ensuite sur **Suivant : Machines virtuelles**.

    ![Répliquer des machines virtuelles](./media/tutorial-migrate-physical-virtual-machines/source-settings.png)

7. Dans **Machines virtuelles**, dans **Importer les paramètres de migration à partir d’une évaluation ?** , conservez le paramètre par défaut **Non, je vais spécifier les paramètres de migration manuellement**.
8. Cochez chaque machine virtuelle devant faire l’objet d’une migration. Cliquez ensuite sur **Suivant : Paramètres de la cible**.

    ![Sélectionner les machines virtuelles](./media/tutorial-migrate-physical-virtual-machines/select-vms.png)

9. Dans **Paramètres de la cible**, sélectionnez l’abonnement et la région cible vers laquelle vous allez migrer, puis spécifiez le groupe de ressources dans lequel les machines virtuelles Azure résideront après la migration.
10. Dans **Réseau virtuel**, sélectionnez le réseau virtuel/sous-réseau Azure auquel les machines virtuelles Azure seront jointes après la migration.
11. Dans **Azure Hybrid Benefit** :
    - Sélectionnez **Non** si vous ne souhaitez pas appliquer Azure Hybrid Benefit. Cliquez ensuite sur **Suivant**.
    - Sélectionnez **Oui** si vous avez des machines Windows Server couvertes par des abonnements Software Assurance ou Windows Server actifs et que vous souhaitez appliquer l’avantage aux machines que vous migrez. Cliquez ensuite sur **Suivant**.

    ![Paramètres de la cible](./media/tutorial-migrate-physical-virtual-machines/target-settings.png)

12. Dans **Capacité de calcul**, vérifiez le nom, la taille, le type de disque du système d’exploitation et le groupe à haute disponibilité de la machine virtuelle. Les machines virtuelles doivent satisfaire aux [exigences d’Azure](migrate-support-matrix-physical-migration.md#azure-vm-requirements).

    - **Taille de la machine virtuelle** : par défaut, Azure Migrate Server Migration choisit une taille qui correspond à la taille la plus proche dans l’abonnement Azure. Vous pouvez également choisir une taille manuelle dans **Taille de la machine virtuelle Azure**.
    - **Disque de système d’exploitation** : spécifiez le disque du système d’exploitation (démarrage) pour la machine virtuelle. Le disque du système d’exploitation est le disque qui contient le chargeur de démarrage et le programme d’installation du système d’exploitation. 
    - **Groupe à haute disponibilité** : si la machine virtuelle doit se trouver dans un groupe à haute disponibilité Azure après la migration, spécifiez-le ici. Ce groupe doit figurer dans le groupe de ressources cible que vous spécifiez pour la migration.

    ![Paramètres de capacité de calcul](./media/tutorial-migrate-physical-virtual-machines/compute-settings.png)

13. Dans **Disques**, indiquez si les disques de machine virtuelle doivent être répliqués sur Azure, puis sélectionnez le type de disque (SSD/HDD standard ou disques managés Premium) dans Azure. Cliquez ensuite sur **Suivant**.
    - Vous pouvez exclure des disques de la réplication.
    - Si vous excluez des disques, ils ne seront pas présents sur la machine virtuelle Azure après la migration. 

    ![Paramètres des disques](./media/tutorial-migrate-physical-virtual-machines/disks.png)

14. Dans **Passer en revue et démarrer la réplication**, passez en revue les paramètres, puis cliquez sur **Répliquer** pour démarrer la réplication initiale pour les serveurs.

> [!NOTE]
> Vous pouvez mettre à jour les paramètres de réplication à tout moment avant le démarrage de la réplication (**Gérer** > **Réplication des machines**). Vous ne pouvez pas changer les paramètres après le démarrage de la réplication.

## <a name="8-track-and-monitor-replication-status"></a>8. Suivre et superviser l’état de la réplication

- Quand vous cliquez sur **Répliquer**, un travail Démarrer la réplication commence.
- Une fois le travail Démarrer la réplication terminé, les machines virtuelles commencent leur réplication initiale vers Azure.
- Au terme de la réplication initiale, la réplication différentielle commence. Les modifications incrémentielles apportées aux disques des machines virtuelles AWS sont répliquées régulièrement sur les disques de réplica dans Azure.

Vous pouvez suivre l’état du travail dans les notifications du portail.

Vous pouvez superviser l’état de la réplication en cliquant sur **Réplication de serveurs** dans **Azure Migrate : Server Migration**.  

![Superviser la réplication](./media/tutorial-migrate-physical-virtual-machines/replicating-servers.png)

## <a name="9-run-a-test-migration"></a>9. Exécuter un test de migration

Quand la réplication différentielle commence, vous pouvez exécuter une migration de test avec les machines virtuelles avant d’exécuter une migration complète vers Azure. Un test de migration est fortement recommandé car il offre la possibilité de détecter tous les problèmes éventuels et de les résoudre avant de procéder à la migration réelle. Il est recommandé de faire ce test au moins une fois pour chaque machine virtuelle avant de la migrer.

- L’exécution d’un test de migration permet de vérifier que la migration fonctionne comme prévu, sans impact sur les machines virtuelles AWS qui restent opérationnelles et poursuivent la réplication.
- Une migration de test simule la migration en créant une machine virtuelle Azure à l’aide de données répliquées (il s’agit en général d’une migration vers un réseau virtuel hors production dans votre abonnement Azure).
- Vous pouvez utiliser la machine virtuelle Azure de test répliquée pour valider la migration, effectuer des tests d’applications et résoudre les éventuels problèmes avant la migration complète.

Effectuez une migration de test de la façon suivante :

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Tester les serveurs migrés**.

     ![Tester les serveurs migrés](./media/tutorial-migrate-physical-virtual-machines/test-migrated-servers.png)

2. Cliquez avec le bouton droit sur la machine virtuelle à tester, puis cliquez sur **Migration de test**.

    ![Migration de test](./media/tutorial-migrate-physical-virtual-machines/test-migrate.png)

3. Dans **Migration de test**, sélectionnez le réseau virtuel Azure dans lequel la machine virtuelle Azure se trouvera après la migration. Nous vous recommandons d’utiliser un réseau virtuel hors production.
4. Le travail **Migration de test** démarre. Supervisez le travail dans les notifications du portail.
5. Une fois la migration terminée, affichez la machine virtuelle Azure migrée dans **Machines virtuelles** dans le portail Azure. Le nom de la machine a le suffixe **-Test**.
6. Une fois le test terminé, cliquez avec le bouton droit sur la machine virtuelle Azure dans **Réplication des machines**, puis cliquez sur **Nettoyer la migration de test**.

    ![Nettoyer la migration](./media/tutorial-migrate-physical-virtual-machines/clean-up.png)


## <a name="10-migrate-aws-vms"></a>10. Migrer des machines virtuelles AWS

Après avoir vérifié que le test de migration fonctionne comme prévu, vous pouvez migrer les machines virtuelles AWS.

1. Dans le projet Azure Migrate > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Réplication de serveurs**.

    ![Réplication de serveurs](./media/tutorial-migrate-physical-virtual-machines/replicate-servers.png)

2. Dans **Réplication des machines**, cliquez avec le bouton droit sur la machine virtuelle > **Migrer**.
3. Dans **Migrer** > **Arrêter les machines virtuelles et effectuer une migration planifiée sans perte de données**, sélectionnez **Oui** > **OK**.
    - Si vous ne souhaitez pas arrêter la machine virtuelle, sélectionnez **Non**.
4. Un travail de migration démarre pour la machine virtuelle. Vous pouvez afficher l’état du travail en cliquant sur l’icône de notification en forme de cloche située en haut à droite de la page du portail ou en accédant à la page des travaux de l’outil Server Migration (cliquez sur Vue d’ensemble dans la vignette de l’outil > sélectionnez Travaux dans le menu de gauche).
5. Une fois le travail terminé, vous pouvez afficher et gérer la machine virtuelle à partir de la page Machines virtuelles.

### <a name="complete-the-migration"></a>Effectuer la migration

1. Une fois la migration terminée, cliquez avec le bouton droit sur la machine virtuelle > **Arrêter la migration**. Cette opération effectue les actions suivantes :
    - Arrête la réplication de la machine virtuelle AWS.
    - Supprime la machine virtuelle AWS du nombre **Réplication de serveurs** dans Azure Migrate : Server Migration.
    - Nettoie les informations d’état de réplication pour la machine virtuelle.
2. Installez l’agent [Linux](../virtual-machines/extensions/agent-linux.md) sur les machines migrées. L’agent Windows de machine virtuelle Azure est préinstallé au cours du processus de migration.
3. Effectuez les éventuels ajustements post-migration de l’application, comme la mise à jour des chaînes de connexion de base de données et les configurations du serveur web.
4. Effectuez les tests finaux de réception de l’application et de la migration sur l’application migrée qui s’exécute maintenant dans Azure.
5. Réduisez le trafic vers l’instance de machine virtuelle Azure migrée.
6. Mettez à jour la documentation interne en y mentionnant le nouvel emplacement et la nouvelle adresse IP des machines virtuelles Azure. 

## <a name="post-migration-best-practices"></a>Bonnes pratiques après la migration

- Pour une meilleure résilience :
    - Sécurisez les données en sauvegardant les machines virtuelles Azure avec le service Sauvegarde Azure. [Plus d’informations](../backup/quick-backup-vm-portal.md)
    - Conservez les charges de travail en cours d’exécution et disponibles en continu en répliquant des machines virtuelles Azure vers une région secondaire avec Site Recovery. [Plus d’informations](../site-recovery/azure-to-azure-tutorial-enable-replication.md)
- Pour renforcer la sécurité :
    - Verrouillez et limitez l’accès du trafic entrant avec l’[administration juste-à-temps d’Azure Security Center](../security-center/security-center-just-in-time.md).
    - Limitez le trafic réseau vers les points de terminaison de gestion avec des [groupes de sécurité réseau](../virtual-network/security-overview.md).
    - Déployez [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) pour sécuriser les disques, et protégez les données contre le vol et les accès non autorisés.
    - Découvrez plus d’informations sur la [sécurisation des ressources IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) et visitez [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Pour la surveillance et la gestion :
    - Envisagez de déployer [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) pour surveiller l’utilisation et les coûts des ressources.

## <a name="next-steps"></a>Étapes suivantes

Examinez le [parcours de migration cloud](/azure/architecture/cloud-adoption/getting-started/migrate) dans le framework d’adoption du cloud Azure.

## <a name="troubleshooting--tips"></a>Résolution des problèmes/Conseils

**Question :** Je ne vois pas ma machine virtuelle AWS dans la liste des serveurs découverts pour la migration.   
**Réponse :** Vérifiez si votre appliance de réplication respecte les exigences. Vérifiez que l’agent Mobility est installé sur la machine virtuelle source à migrer et qu’il est inscrit sur le serveur de configuration. Vérifiez les paramètres réseau et les règles de pare-feu pour autoriser un chemin réseau entre l’appliance de réplication et les machines virtuelles AWS sources.  

**Question :** Comment puis-je savoir si la migration de ma machine virtuelle s’est correctement effectuée ?   
**Réponse :** Après la migration, vous pouvez afficher et gérer la machine virtuelle à partir de la page Machines virtuelles. Connectez-vous à la machine virtuelle migrée pour le vérifier.  

**Question :** Je ne parviens pas à importer des machines virtuelles à des fins de migration à partir de mes résultats Server Assessment précédemment créés.   
**Réponse :** Actuellement, nous ne prenons pas en charge l’importation de l’évaluation pour ce workflow. En guise de solution de contournement, vous pouvez exporter l’évaluation, puis sélectionner manuellement la recommandation pendant l’étape Activer la réplication.
  
**Question :** J’obtiens l’erreur « Impossible de récupérer le GUID du BIOS » quand j’essaie de découvrir mes machines virtuelles AWS.   
**Réponse :** Vérifiez quels sont les systèmes d’exploitation pris en charge pour les machines virtuelles AWS.  

**Question :** Mon état de réplication ne progresse pas.    
**Réponse :** Vérifiez si votre appliance de réplication respecte les exigences. Vérifiez que vous avez activé les ports nécessaires sur votre appliance de réplication : port TCP 9443 et HTTPS 443 pour le transport de données. Vérifiez qu’il n’existe aucune version en double obsolète de l’appliance de réplication connectée au même projet.   

**Question :** Je ne parviens pas à découvrir des instances AWS à l’aide d’Azure Migrate en raison du code d’état HTTP 504 du service de gestion Windows distant.    
**Réponse :** Consultez les exigences applicables à l’appliance Azure Migrate et les besoins d’accès URL. Vérifiez qu’aucun paramètre de proxy ne bloque l’inscription de l’appliance.   
