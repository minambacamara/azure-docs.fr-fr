---
title: Expédition autogérée Microsoft Azure Data Box | Microsoft Docs sur les données
description: Décrit le flux de travail d’expédition autogérée pour les appareils Azure Data Box
services: databox
author: priestlg
ms.service: databox
ms.subservice: disk
ms.topic: how-to
ms.date: 05/20/2020
ms.author: v-grpr
ms.openlocfilehash: 99c7021744467484656c52cf72378c6f553c3913
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "84608585"
---
# <a name="use-self-managed-shipping-for-azure-data-box-in-the-azure-portal"></a>Utilisez l’expédition autogérée pour Azure Data Box dans le Portail Azure

Cet article décrit les tâches d’expédition autogérées pour la commande, l’enlèvement et le dépôt d’une appareil Azure Data Box. Vous pouvez gérer l’appareil Data Box à l’aide du Portail Azure.

## <a name="prerequisites"></a>Prérequis

L’expédition autogérée est disponible comme option lorsque vous [commandez Azure Data Box](data-box-deploy-ordered.md). L’expédition autogérée est disponible uniquement dans les régions suivantes :

* Gouvernement américain
* Europe de l’Ouest
* Japon
* Singapour
* Corée du Sud

## <a name="use-self-managed-shipping"></a>Utiliser l’expédition autogérée

Quand vous passez une commande de Data Box, vous pouvez choisir l’option d’expédition autogérée.

1. Dans votre commande Azure Data Box, sous **Détails du contact**, sélectionnez **+ Ajouter une adresse de livraison**.
   ![Expédition autogérée](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-1.png)

2. Lorsque vous choisissez le type d’expédition, sélectionnez l’option **expédition autogérée**. Cette option est disponible uniquement si vous êtes dans une des régions prises en charge, comme cela est décrit dans les prérequis.

3. Une fois que vous avez fourni votre adresse de livraison, vous devez la valider et terminer votre commande.
   ![Expédition autogérée](media\data-box-portal-customer-managed-shipping\choose-self-managed-shipping-2.png)

4. Quand l’appareil a été préparé et que vous en avez été informé par e-mail, vous pouvez planifier un prélèvement.

   Dans votre commande Azure Data Box, accédez à **Vue d’ensemble** puis sélectionnez **Planifier l’enlèvement**.

   ![Commande d’un appareil Data Box à enlever](media\data-box-portal-customer-managed-shipping\data-box-portal-schedule-pickup-01.png)

5. Suivez les instructions dans **Planifier la collecte pour Azure**.

   Avant de pouvoir obtenir votre code d’autorisation, vous devez envoyer un courrier électronique à [adbops@microsoft.com](mailto:adbops@microsoft.com) pour planifier l’enlèvement de l’appareil à partir du centre de données de votre région.

   ![Planifier l’enlèvement](media\data-box-portal-customer-managed-shipping\data-box-portal-schedule-pickup-email-01.png)

6. Une fois que vous avez planifié l’enlèvement de votre appareil, vous pouvez afficher le code d’autorisation de votre appareil dans le volet **Planifier l’enlèvement pour Azure**.

   ![Affichage de votre code d’autorisation](media\data-box-portal-customer-managed-shipping\data-box-portal-auth-01b.png)

   Notez ce **code d’autorisation**. Conformément aux exigences de sécurité, au moment de la planification de l’enlèvement, il est nécessaire de donner le nom de la personne qui sera présente pour l’enlèvement.

   Vous devez également fournir des détails sur les personnes qui se rendront au centre de données pour l’enlèvement. Vous ou le point de contact devez posséder une pièce d’identité avec photo approuvée par le gouvernement qui sera validé par le centre de donnée.

   En outre, la personne qui viendra enlever l’appareil doit également avoir le **code d’autorisation**. Le code d’autorisation est validé par le centre de données au moment de l’enlèvement.

7. Votre commande passe automatiquement à l’état **Récupérée** une fois l’appareil enlevé au centre de données.

    ![Picked up (Récupérée)](media\data-box-portal-customer-managed-shipping\data-box-portal-picked-up-boxed-01.png)

8. Après avoir enlevé l’appareil, copiez les données vers le ou les Data Box sur votre site. Une fois la copie des données terminée, vous pouvez préparer l’expédition du Data Box. Pour plus d’informations, consultez [Préparer l’expédition](data-box-deploy-picked-up.md#prepare-to-ship).

   L’étape **Préparer l’expédition** doit être exécutée sans erreur critique, dans le cas contraire, vous devrez réexécuter cette étape après avoir effectué les corrections nécessaires. Une fois l’étape Préparer l’expédition terminée, vous pouvez afficher le code d’autorisation pour le dépôt sur l’interface utilisateur locale de l’appareil.

   > [!NOTE]
   > Ne partagez pas le code d’autorisation par e-mail. Il doit uniquement être vérifié au centre de données lors du dépôt.

9. Si vous avez reçu une date de rendez-vous pour le dépôt, la commande doit se trouver à l’état **Prêt à être reçu au centre de données Azure** sur le Portail Azure. Suivez les instructions sous **Planifier le dépôt** pour renvoyer l’appareil.

   ![Affichage de votre code d’autorisation](media\data-box-portal-customer-managed-shipping\data-box-portal-received-complete-02b.png)

10. Une fois que votre pièce d’identité et votre code d’autorisation ont été vérifiés et que vous avez déposé l’appareil dans le centre de données, l’état de la commande passe à **Reçu**.

    ![Réception terminée](media\data-box-portal-customer-managed-shipping\data-box-portal-received-complete-01.png)

11. Une fois l’appareil reçu, la copie des données se poursuit. Lorsque la copie est finie, la commande est terminée.

## <a name="next-steps"></a>Étapes suivantes

* [Bien démarrer avec Azure Data Box](data-box-quickstart-portal.md)
