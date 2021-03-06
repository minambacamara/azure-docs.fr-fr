---
title: Changer le modèle d’extraction de flux
description: Découvrir comment utiliser le modèle de tirage (pull) du flux de modification Azure Cosmos DB pour lire le flux de modification et les différences entre le modèle de tirage et le processeur de flux de modification
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/19/2020
ms.reviewer: sngun
ms.openlocfilehash: 8916f4b9824f88361fdeb9d866f84adb71e8138e
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "85563791"
---
# <a name="change-feed-pull-model-in-azure-cosmos-db"></a>Modèle de tirage (pull) du flux de modification dans Azure Cosmos DB

Avec le modèle de tirage du flux de modification, vous pouvez consommer le flux de modification Azure Cosmos DB à votre rythme. Comme le [processeur de flux de modification](change-feed-processor.md) vous le permet déjà, vous pouvez utiliser le modèle de tirage du flux de modification pour paralléliser le traitement des modifications sur plusieurs consommateurs de flux de modification.

> [!NOTE]
> Le modèle de tirage du flux de modification n’est actuellement disponible qu’en [préversion dans le SDK .NET Azure Cosmos DB](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.9.0-preview). La préversion n’est pas encore disponible pour d’autres versions du kit SDK.

## <a name="consuming-an-entire-containers-changes"></a>Consommation des modifications d’un conteneur entier

Vous pouvez créer un `FeedIterator` pour traiter le flux de modification à l’aide du modèle de tirage. Lorsque vous créez initialement un `FeedIterator`, vous pouvez spécifier un `StartTime` facultatif dans `ChangeFeedRequestOptions`. Lorsqu’il n’est pas spécifié, l’élément `StartTime` est l’heure actuelle.

Il existe deux types de `FeedIterator`. En plus des exemples ci-dessous qui retournent des objets d’entité, vous pouvez également obtenir la réponse avec la prise en charge de `Stream`. Les flux vous permettent de lire des données sans avoir à les désérialiser au préalable, en les enregistrant sur les ressources clientes.

Voici un exemple d’obtention d’un `FeedIterator` qui retourne des objets d’entité, dans ce cas un objet `User` :

```csharp
FeedIterator<User> iteratorWithPOCOS = container.GetChangeFeedIterator<User>();
```

Voici un exemple de l’obtention de `FeedIterator` qui retourne `Stream` :

```csharp
FeedIterator iteratorWithStreams = container.GetChangeFeedStreamIterator();
```

À l’aide de `FeedIterator`, vous pouvez facilement traiter l’intégralité du flux de modification d’un conteneur à votre propre rythme. Voici un exemple :

```csharp
FeedIterator<User> iteratorForTheEntireContainer= container.GetChangeFeedIterator<User>();

while (iteratorForTheEntireContainer.HasMoreResults)
{
   FeedResponse<User> users = await iteratorForTheEntireContainer.ReadNextAsync();

   foreach (User user in users)
    {
        Console.WriteLine($"Detected change for user with id {user.id}");
    }
}
```

## <a name="consuming-a-partition-keys-changes"></a>Consommation des modifications d’une clé de partition

Dans certains cas, vous souhaiterez peut-être traiter uniquement les modifications d’une clé de partition particulière. Vous pouvez obtenir `FeedIterator` pour une clé de partition spécifique, et traiter les modifications de la même façon que vous le faites pour un conteneur entier.

```csharp
FeedIterator<User> iteratorForThePartitionKey = container.GetChangeFeedIterator<User>(new PartitionKey("myPartitionKeyValueToRead"));

while (iteratorForThePartitionKey.HasMoreResults)
{
   FeedResponse<User> users = await iteratorForThePartitionKey.ReadNextAsync();

   foreach (User user in users)
    {
        Console.WriteLine($"Detected change for user with id {user.id}");
    }
}
```

## <a name="using-feedrange-for-parallelization"></a>Utilisation de FeedRange pour la parallélisation

Dans le [processeur de flux de modification](change-feed-processor.md), le travail est automatiquement réparti sur plusieurs consommateurs. Dans le modèle de tirage du flux de modification, vous pouvez utiliser `FeedRange` pour paralléliser le traitement du flux de modification. `FeedRange` représente une plage de valeurs de clé de partition.

Voici un exemple qui montre comment obtenir une liste de plages pour votre conteneur :

```csharp
IReadOnlyList<FeedRange> ranges = await container.GetFeedRangesAsync();
```

Lorsque vous obtenez la liste des FeedRanges pour votre conteneur, vous récupérez un `FeedRange` par [partition physique](partition-data.md#physical-partitions).

À l’aide d’un `FeedRange`, vous pouvez ensuite créer un `FeedIterator` pour paralléliser le traitement du flux de modification sur plusieurs machines ou threads. Contrairement à l’exemple précédent qui illustrait comment obtenir un `FeedIterator` unique pour le conteneur entier, vous pouvez utiliser `FeedRange` pour obtenir plusieurs FeedIterators qui peuvent traiter le flux de modification en parallèle.

Si vous souhaitez utiliser FeedRanges, vous devez disposer d’un processus d’orchestrateur qui obtient FeedRanges et les distribue à ces machines. Cette distribution peut être :

* L’utilisation de `FeedRange.ToJsonString` et la distribution de cette valeur de chaîne. Les consommateurs peuvent utiliser cette valeur avec `FeedRange.FromJsonString`
* Le passage de la référence d’objet `FeedRange` si la distribution est en cours.

Voici un exemple montrant comment lire à partir du début du flux de modification du conteneur, au moyen de deux machines hypothétiques distinctes qui lisent en parallèle :

Machine 1 :

```csharp
FeedIterator<User> iteratorA = container.GetChangeFeedIterator<User>(ranges[0], new ChangeFeedRequestOptions{StartTime = DateTime.MinValue});
while (iteratorA.HasMoreResults)
{
   FeedResponse<User> users = await iteratorA.ReadNextAsync();

   foreach (User user in users)
    {
        Console.WriteLine($"Detected change for user with id {user.id}");
    }
}
```

Machine 2 :

```csharp
FeedIterator<User> iteratorB = container.GetChangeFeedIterator<User>(ranges[1], new ChangeFeedRequestOptions{StartTime = DateTime.MinValue});
while (iteratorB.HasMoreResults)
{
   FeedResponse<User> users = await iteratorB.ReadNextAsync();

   foreach (User user in users)
    {
        Console.WriteLine($"Detected change for user with id {user.id}");
    }
}
```

## <a name="saving-continuation-tokens"></a>Enregistrement des jetons de continuation

Vous pouvez enregistrer la position de votre `FeedIterator` en créant un jeton de continuation. Un jeton de continuation est une valeur de chaîne qui assure le suivi des dernières modifications traitées de votre FeedIterator. Le `FeedIterator` peut ainsi reprendre à cet endroit, ultérieurement. Le code suivant lit le flux de modification depuis la création du conteneur. Lorsque plus aucune modification n’est disponible, un jeton de continuation est conservé pour que la consommation du flux de modification puisse être reprise plus tard.

```csharp
FeedIterator<User> iterator = container.GetChangeFeedIterator<User>(ranges[0], new ChangeFeedRequestOptions{StartTime = DateTime.MinValue});

string continuation = null;

while (iterator.HasMoreResults)
{
   FeedResponse<User> users = await iterator.ReadNextAsync();
   continuation = users.ContinuationToken;

   foreach (User user in users)
    {
        Console.WriteLine($"Detected change for user with id {user.id}");
    }
}

// Some time later
FeedIterator<User> iteratorThatResumesFromLastPoint = container.GetChangeFeedIterator<User>(continuation);
```

Tant que le conteneur Cosmos existe, le jeton de continuation d’un FeedIterator n’expire pas.

## <a name="comparing-with-change-feed-processor"></a>Comparaison avec le processeur de flux de modification

De nombreux scénarios peuvent traiter le flux de modification à l’aide du [processeur de flux de modification](change-feed-processor.md) ou du modèle de tirage (pull). Les jetons de continuation du modèle de tirage, comme le conteneur de bail du processeur de flux de modification, sont des « signets » pour le dernier élément (ou lot d’éléments) traité dans le flux de modification.
Toutefois, vous ne pouvez pas convertir les jetons de continuation en conteneur de bail (ou vice versa).

Envisagez plutôt d’utiliser le modèle de tirage dans les scénarios suivants :

- Lecture des modifications à partir d’une clé de partition particulière
- Contrôle de la vitesse à laquelle votre client reçoit les modifications à traiter
- Lecture unique des données existantes dans le flux de modification (par exemple, pour effectuer une migration de données)

Voici quelques différences capitales entre le processeur de flux de modification et le modèle de tirage :

|Fonctionnalité  | Processeur de flux de modification| Modèle de tirage (pull) |
| --- | --- | --- |
| Suivi du point actuel dans le traitement du flux de modification | Bail (stocké dans un conteneur Azure Cosmos DB) | Jeton de continuation (stocké en mémoire ou rendu persistant manuellement) |
| Possibilité de relire les modifications passées | Oui, avec le modèle d’envoi (push) | Oui, avec le modèle de tirage (pull)|
| Interrogation des modifications à venir | Recherche automatiquement les modifications en fonction de `WithPollInterval` spécifié par l’utilisateur | Manuel |
| Traitement des modifications depuis l’ensemble du conteneur | Oui, et automatiquement mis en parallèle sur plusieurs threads/machines consommant à partir du même conteneur| Oui, et mis en parallèle manuellement à l’aide de FeedTokens |
| Traitement des modifications à partir d’une seule clé de partition | Non pris en charge | Oui|
| Niveau de support | Mise à la disposition générale | PRÉVERSION |

## <a name="next-steps"></a>Étapes suivantes

* [Présentation du flux de modification](change-feed.md)
* [Utilisation du processeur de flux de modification](change-feed-processor.md)
* [Déclencher Azure Functions](change-feed-functions.md)