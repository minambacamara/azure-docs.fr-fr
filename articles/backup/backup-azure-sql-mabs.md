---
title: Sauvegarder des bases de données SQL Server à l'aide du serveur de sauvegarde Azure
description: Cet article présente les étapes de configuration de la sauvegarde des bases de données SQL Server à l'aide du serveur de sauvegarde Microsoft Azure.
ms.topic: conceptual
ms.date: 03/24/2017
ms.openlocfilehash: 2bb172ca36f3f932fdaaf5b71e8fa183c04d1510
ms.sourcegitcommit: 1f48ad3c83467a6ffac4e23093ef288fea592eb5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/29/2020
ms.locfileid: "84194181"
---
# <a name="back-up-sql-server-to-azure-by-using-azure-backup-server"></a>Sauvegarder des bases de données SQL Server sur Azure à l'aide du serveur de sauvegarde Azure

Cet article explique comment configurer des sauvegardes de bases de données SQL Server à l'aide du serveur de sauvegarde Microsoft Azure.

Pour sauvegarder une base de données SQL Server et la récupérer à partir d'Azure :

1. Créez une stratégie de sauvegarde pour protéger les bases de données SQL Server dans Azure.
1. Créez des copies de sauvegarde à la demande dans Azure.
1. Récupérez la base de données dans Azure.

## <a name="before-you-start"></a>Avant de commencer

Avant de commencer, assurez-vous d'avoir [installé et préparé le serveur de sauvegarde Azure](backup-azure-microsoft-azure-backup.md).

## <a name="create-a-backup-policy"></a>Créer une stratégie de sauvegarde

Pour protéger des bases de données SQL Server dans Azure, commencez par créer une stratégie de sauvegarde :

1. Sur le serveur de sauvegarde Azure, sélectionnez l'espace de travail **Protection**.
1. Sélectionnez **Nouveau** pour créer un groupe de protection.

    ![Créer un groupe de protection sur le serveur de sauvegarde Azure](./media/backup-azure-backup-sql/protection-group.png)
1. Sur la page de démarrage, passez en revue les instructions relatives à la création d'un groupe de protection. Sélectionnez ensuite **Suivant**.
1. Pour le type de groupe de protection, sélectionnez **Serveurs**.

    ![Sélectionner le type de groupe de protection Serveurs](./media/backup-azure-backup-sql/pg-servers.png)
1. Développez l'instance de SQL Server sur laquelle se trouvent les bases de données que vous souhaitez sauvegarder. Vous accédez aux sources de données qui peuvent être sauvegardées à partir de ce serveur. Développez **Tous les partages SQL**, puis sélectionnez les bases de données que vous souhaitez sauvegarder. Dans cet exemple, nous sélectionnons ReportServer$MSDPM2012 et ReportServer$MSDPM2012TempDB. Sélectionnez **Suivant**.

    ![Sélectionner une base de données SQL Server](./media/backup-azure-backup-sql/pg-databases.png)
1. Nommez le groupe de protection, puis sélectionnez **Je souhaite une protection en ligne**.

    ![Choisir une méthode de protection des données : protection sur disque à court terme ou protection Azure en ligne](./media/backup-azure-backup-sql/pg-name.png)
1. Sur la page **Spécifier les objectifs à court terme**, incluez les entrées nécessaires à la création de points de sauvegarde sur le disque.

    Dans cet exemple, la **Période de rétention** est définie sur *5 jours*. La **Fréquence de synchronisation** de la sauvegarde est définie sur *15 minutes*. La **Sauvegarde complète rapide** est définie sur *20h00*.

    ![Définir des objectifs à court terme pour la protection de la sauvegarde](./media/backup-azure-backup-sql/pg-shortterm.png)

   > [!NOTE]
   > Dans cet exemple, un point de sauvegarde est créé à 20h00 quotidiennement. Les données qui ont été modifiées depuis le point de sauvegarde de la veille à 20h00 sont transférées. Ce processus est appelé **Sauvegarde expresse rapide**. Bien que les journaux des transactions soient synchronisés toutes les 15 minutes, si nous devons récupérer la base de données à 21h00, le point est créé en relisant les journaux à partir du dernier point de sauvegarde complète rapide, à savoir 20h00 dans cet exemple.
   >
   >

1. Sélectionnez **Suivant**. Le serveur de sauvegarde Microsoft Azure affiche l'espace de stockage global disponible. Il présente également l'utilisation potentielle de l'espace disque.

    ![Configurer l'allocation de disque dans le serveur de sauvegarde Microsoft Azure](./media/backup-azure-backup-sql/pg-storage.png)

    Par défaut, le serveur de sauvegarde Microsoft Azure crée un volume par source de données (base de données SQL Server). Le volume est utilisé pour la copie de sauvegarde initiale. Dans cette configuration, le Gestionnaire de disque logique limite la protection du serveur de sauvegarde Microsoft Azure à 300 sources de données (bases de données SQL Server). Pour contourner cette limitation, sélectionnez **Colocaliser les données dans le pool de stockage DPM**. Si vous utilisez cette option, le serveur de sauvegarde Microsoft Azure utilise un seul volume pour plusieurs sources de données. Cette configuration permet au serveur de sauvegarde Microsoft Azure de protéger jusqu'à 2 000 bases de données SQL Server.

    Si vous sélectionnez l'option **Augmenter automatiquement les volumes**, le serveur de sauvegarde Microsoft Azure peut gérer l'augmentation du volume de sauvegarde à mesure que les données de production augmentent. Si vous ne sélectionnez pas l'option **Augmenter automatiquement les volumes**, le serveur de sauvegarde Microsoft Azure limite le stockage de sauvegarde aux sources de données figurant dans le groupe de protection.
1. Si vous êtes un administrateur, vous pouvez décider de transférer cette sauvegarde initiale **Automatiquement sur le réseau** et choisir l'heure de transfert. Vous pouvez également choisir de transférer **Manuellement** la sauvegarde. Sélectionnez ensuite **Suivant**.

    ![Choisir une méthode de création de réplica dans le serveur de sauvegarde Microsoft Azure](./media/backup-azure-backup-sql/pg-manual.png)

    La copie de sauvegarde initiale nécessite le transfert de la totalité de la source de données (base de données SQL Server). Les données de sauvegarde sont déplacées du serveur de production (ordinateur SQL Server) vers le serveur de sauvegarde Microsoft Azure. Si cette sauvegarde est volumineuse, le transfert des données sur le réseau peut entraîner une saturation de la bande passante. Pour cette raison, les administrateurs peuvent choisir d'utiliser des supports amovibles pour transférer la sauvegarde initiale **Manuellement**. Ils peuvent également transférer les données **Automatiquement sur le réseau** à une heure spécifiée.

    Une fois la sauvegarde initiale terminée, les sauvegardes se poursuivent de façon incrémentielle sur la copie de sauvegarde initiale. Les sauvegardes incrémentielles sont en général très limitées et sont faciles à transférer sur le réseau.
1. Choisissez quand exécuter une vérification de cohérence. Sélectionnez ensuite **Suivant**.

    ![Choisir quand exécuter une vérification de cohérence](./media/backup-azure-backup-sql/pg-consistent.png)

    Le serveur de sauvegarde Microsoft Azure peut exécuter une vérification de cohérence sur l'intégrité du point de sauvegarde. Il calcule la somme de contrôle du fichier de sauvegarde sur le serveur de production (ordinateur SQL Server dans cet exemple) et les données sauvegardées pour ce fichier sur le serveur de sauvegarde Microsoft Azure. Si la vérification détecte un conflit, le fichier sauvegardé sur le serveur de sauvegarde Microsoft Azure est supposé endommagé. Le serveur de sauvegarde Microsoft Azure corrige les données sauvegardées en envoyant les blocs correspondant à l'incohérence de somme contrôle. La vérification de cohérence étant une opération exigeante en matière de performances, les administrateurs peuvent la planifier ou l'exécuter automatiquement.
1. Sélectionnez les sources de données à protéger dans Azure. Sélectionnez ensuite **Suivant**.

    ![Sélectionner les sources de données à protéger dans Azure](./media/backup-azure-backup-sql/pg-sqldatabases.png)
1. Si vous êtes un administrateur, vous pouvez choisir des planifications de sauvegarde et des stratégies de rétention correspondant aux stratégies de votre organisation.

    ![Choisir des planifications et des stratégies de rétention](./media/backup-azure-backup-sql/pg-schedule.png)

    Dans cet exemple, les sauvegardes sont effectuées quotidiennement à 12h00 et 20h00.

    > [!TIP]
    > Pour une récupération rapide, conservez quelques points de récupération à court terme sur votre disque. Ces points de récupération sont utilisés pour la restauration opérationnelle. Azure constitue un bon emplacement hors site, adossé à des contrats de niveau de service plus avantageux et dont la disponibilité est garantie.
    >
    > Utilisez Data Protection Manager (DPM) pour planifier les sauvegardes Azure après la fin des sauvegardes sur disque local. Lorsque vous suivez cette pratique, la dernière sauvegarde sur disque est copiée sur Azure.
    >

1. Cliquez sur la planification de stratégie de rétention. Pour plus d'informations sur le fonctionnement de la stratégie de rétention, consultez [Utilisation de Sauvegarde Azure pour remplacer votre infrastructure sur bande](backup-azure-backup-cloud-as-tape.md).

    ![Choisir une stratégie de rétention sur le serveur de sauvegarde Microsoft Azure](./media/backup-azure-backup-sql/pg-retentionschedule.png)

    Dans cet exemple :

    * les sauvegardes sont effectuées quotidiennement à 12h00 et 20h00. Elles sont conservées pendant 180 jours.
    * La sauvegarde du samedi à 12h00 est conservée pendant 104 semaines.
    * La sauvegarde du dernier samedi du mois à 12h00 est conservée pendant 60 mois.
    * La sauvegarde du dernier samedi de mars à 12h00 est conservée pendant 10 ans.

    Après avoir choisi une stratégie de rétention, sélectionnez **Suivant**.
1. Choisissez comment transférer la copie de sauvegarde initiale vers Azure.

    * L'option **Automatiquement sur le réseau** suit votre planification de sauvegarde pour transférer les données vers Azure.
    * Pour plus d'informations sur la **sauvegarde en mode hors connexion**, consultez [Vue d'ensemble de la sauvegarde hors connexion](offline-backup-overview.md).

    Après avoir choisi un mécanisme de transfert, sélectionnez **Suivant**.
1. Sur la page **Résumé**, examinez les détails de la stratégie. Sélectionnez ensuite **Créer un groupe**. Vous pouvez sélectionner **Fermer** et surveiller la progression du travail dans l'espace de travail **Surveillance**.

    ![Progression de la création du groupe de protection](./media/backup-azure-backup-sql/pg-summary.png)

## <a name="create-on-demand-backup-copies-of-a-sql-server-database"></a>Créer des copies de sauvegarde à la demande d'une base de données SQL Server

Un point de récupération est créé lors de la première sauvegarde. Au lieu d'attendre l'exécution de la planification, vous pouvez déclencher manuellement la création d'un point de récupération :

1. Dans le groupe de protection, assurez-vous que l'état de la base de données est **OK**.

    ![Groupe de protection présentant l'état de la base de données](./media/backup-azure-backup-sql/sqlbackup-recoverypoint.png)
1. Cliquez avec le bouton droit sur la base de données, puis sélectionnez **Créer un point de récupération**.

    ![Choisir de créer un point de récupération en ligne](./media/backup-azure-backup-sql/sqlbackup-createrp.png)
1. Dans le menu déroulant, sélectionnez **Protection en ligne**. Sélectionnez ensuite **OK** pour démarrer la création d'un point de récupération dans Azure.

    ![Démarrer la création d'un point de récupération dans Azure](./media/backup-azure-backup-sql/sqlbackup-azure.png)
1. Vous pouvez voir la progression du travail dans l'espace de travail **Surveillance**.

    ![Afficher la progression du travail sur la console Surveillance](./media/backup-azure-backup-sql/sqlbackup-monitoring.png)

## <a name="recover-a-sql-server-database-from-azure"></a>Récupération d'une base de données SQL Server à partir d'Azure

Pour récupérer une entité protégée, telle qu'une base de données SQL Server, à partir d'Azure :

1. Ouvrez la Console de gestion du serveur DPM. Accédez à l'espace de travail **Récupération** pour afficher les serveurs que DPM sauvegarde. Sélectionnez la base de données (dans cet exemple, ReportServer$MSDPM2012). Sélectionnez une **Heure de récupération** qui se termine par **En ligne**.

    ![Sélectionner un point de récupération](./media/backup-azure-backup-sql/sqlbackup-restorepoint.png)
1. Cliquez avec le bouton droit sur le nom de la base de données, puis sélectionnez **Récupérer**.

    ![Récupérer une base de données à partir d'Azure](./media/backup-azure-backup-sql/sqlbackup-recover.png)
1. DPM affiche les détails du point de récupération. Sélectionnez **Suivant**. Pour remplacer la base de données, sélectionnez le type de récupération **Récupérer l’instance d’origine de SQL Server**. Sélectionnez ensuite **Suivant**.

    ![Récupérer une base de données à son emplacement d'origine](./media/backup-azure-backup-sql/sqlbackup-recoveroriginal.png)

    Dans cet exemple, DPM permet la récupération de la base de données sur une autre instance de SQL Server ou dans un dossier réseau autonome.
1. Sur la page **Spécifier les options de récupération**, vous pouvez sélectionner les options de récupération. Par exemple, vous pouvez choisir **Limitation de l'utilisation de la bande passante réseau** pour limiter la bande passante que la récupération utilise. Sélectionnez ensuite **Suivant**.
1. Sur la page **Résumé**, vous voyez la configuration actuelle de la récupération. Sélectionnez **Récupérer**.

    L'état de récupération indique la base de données en cours de récupération. Vous pouvez sélectionner **Fermer** pour fermer l'Assistant et afficher la progression dans l'espace de travail **Surveillance**.

    ![Démarrer le processus de récupération](./media/backup-azure-backup-sql/sqlbackup-recoverying.png)

    Une fois la récupération terminée, la base de données restaurée est cohérente avec l'application.

### <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez [Forum aux questions sur Sauvegarde Azure](backup-azure-backup-faq.md).
