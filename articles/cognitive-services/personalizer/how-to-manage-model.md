---
title: Gérer les paramètres de modèle et d’apprentissage – Personalizer
description: Les paramètres d’apprentissage et de modèle d’apprentissage automatique peuvent être exportés à des fins de sauvegarde dans votre propre système de contrôle de code source.
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: 59af5a64ea5752ec03fb44df91440e8d395e3782
ms.sourcegitcommit: 61d850bc7f01c6fafee85bda726d89ab2ee733ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/03/2020
ms.locfileid: "84344354"
---
# <a name="how-to-manage-model-and-learning-settings"></a>Comment gérer les paramètres de modèle et d’apprentissage

Les paramètres d’apprentissage et de modèle d’apprentissage automatique peuvent être exportés à des fins de sauvegarde dans votre propre système de contrôle de code source.

## <a name="export-the-personalizer-model"></a>Exporter le modèle de Personalizer

Dans la section **Paramètres de modèle et d’apprentissage** de Gestion des ressources, examinez la date de création et de dernière mise à jour du modèle, puis exportez le modèle actuel. Vous pouvez utiliser le portail Azure ou les API Personalizer pour exporter un fichier de modèle à des fins d’archivage.

![Exporter le modèle actuel de Personalizer](media/settings/export-current-personalizer-model.png)

## <a name="clear-data-for-your-learning-loop"></a>Effacer les données de votre boucle d'apprentissage

1. Sur le portail Azure, pour votre ressource Personalizer, dans la page **Paramètres de modèle et d’apprentissage**, sélectionnez **Effacer les données**.
1. Pour effacer toutes les données et rétablir l'état d'origine de la boucle d'apprentissage, cochez les 3 cases.

    ![Sur le portail Azure, effacez les données de la ressource Personalizer.](./media/settings/clear-data-from-personalizer-resource.png)

    |Valeur|Objectif|
    |--|--|
    |Données de personnalisation et de récompense consignées.|Ces données de journalisation sont utilisées lors des évaluations hors connexion. Effacez les données si vous réinitialisez votre ressource.|
    |Réinitialisez le modèle Personalizer.|Ce modèle change à chaque nouvelle formation. Cette fréquence de formation est spécifiée dans la section **Fréquence du modèle de chargement** de la page **Configuration**. |
    |Définissez la stratégie d'apprentissage sur le paramètre par défaut.|Si vous avez modifié la stratégie d'apprentissage dans le cadre d'une évaluation hors connexion, la stratégie d'apprentissage d'origine est rétablie.|

1. Sélectionnez **Effacer les données sélectionnées** pour lancer le processus de suppression. L'état est signalé dans les notifications Azure, en haut à droite.

## <a name="import-a-new-learning-policy"></a>Importer une nouvelle stratégie d’apprentissage

Les paramètres de [stratégie d’apprentissage](concept-active-learning.md#understand-learning-policy-settings) déterminent les _hyperparamètres_ de la formation des modèles. Effectuez une [évaluation hors connexion](how-to-offline-evaluation.md) pour rechercher une nouvelle stratégie d’apprentissage.

1. Ouvrez le [portail Azure](https://portal.azure.com) et sélectionnez votre ressource Personalizer.
1. Sélectionnez **Paramètres de modèle et d’apprentissage**  dans la section **Gestion des ressources**.
1. Pour **Importer les paramètres d’apprentissage** sélectionnez le fichier que vous avez créé au format JSON spécifié ci-dessus, puis sélectionnez le bouton **Charger**.

    Attendez la notification qui indique que la stratégie d’apprentissage a été chargée avec succès.

## <a name="export-a-learning-policy"></a>Exporter une stratégie d’apprentissage

1. Ouvrez le [portail Azure](https://portal.azure.com) et sélectionnez votre ressource Personalizer.
1. Sélectionnez **Paramètres de modèle et d’apprentissage**  dans la section **Gestion des ressources**.
1. Pour **Importer les paramètres d’apprentissage**, sélectionnez le bouton **Exporter les paramètres d’apprentissage**. Cette opération enregistre le fichier `json` sur votre ordinateur local.

## <a name="next-steps"></a>Étapes suivantes

[Découvrir comment gérer une stratégie d’apprentissage](how-to-manage-model.md)
