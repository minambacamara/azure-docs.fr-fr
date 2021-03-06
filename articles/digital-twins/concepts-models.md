---
title: Modèles personnalisés
titleSuffix: Azure Digital Twins
description: Découvrez comment Azure Digital Twins utilise des modèles définis par l’utilisateur pour décrire des entités dans votre environnement.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 56ebb32e2d1c2a9bab9592da63e1ada7130bb7ff
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131631"
---
# <a name="understand-twin-models-in-azure-digital-twins"></a>Comprendre les modèles de jumeau dans Azure Digital Twins

L’une des principales caractéristiques d’Azure Digital Twins est la possibilité de définir votre propre vocabulaire et à créer votre graphique de jumeau dans les termes définis par votre entreprise. Cette fonctionnalité est fournie par le biais de **modèles** définis par l’utilisateur. Vous pouvez considérer les modèles comme des noms dans une description de votre monde. 

Un modèle est comparable à une **classe** dans un langage de programmation orienté objet, définissant une forme de données pour un concept particulier dans votre environnement de travail réel. Les modèles ont des noms (par exemple, *Room* ou *TemperatureSensor*) et contiennent des éléments tels que des propriétés, de la télémétrie ou des événements, ainsi que des commandes décrivant ce que peut faire ce type d’entité dans votre environnement. Plus tard, vous utiliserez ces modèles pour créer des [**jumeaux numériques**](concepts-twins-graph.md) représentant des entités spécifiques répondant à une description de ce type.

Les modèles sont écrits à l’aide du langage **DTDL (Digital Twin Definition Language)** basé sur JSON-LD.  

## <a name="digital-twin-definition-language-dtdl-for-writing-models"></a>Langage DTDL (Digital Twin Definition Language) pour l’écriture de modèles

Les modèles pour Azure Digital Twins sont définis à l’aide du DTDL (Digital Twin Definition Language). Le langage DTDL est basé sur JSON-LD et est indépendant du langage de programmation. Le langage DTDL n’est pas exclusif d’Azure Digital Twins. Il est également utilisé pour représenter des données d’appareil dans d’autres services IoT tels que [IoT Plug-and-Play](../iot-pnp/overview-iot-plug-and-play.md). 

Azure Digital Twins utilise le langage DTDL *version 2*. Pour plus d’informations sur cette version de DTDL, consultez sa documentation spec dans GitHub : [*DTDL (Digital Twins Definition Language) – version 2*](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

> [!TIP] 
> Les services qui utilisent le langage DTDL n’implémentent pas tous exactement les mêmes fonctionnalités de ce langage. Par exemple, la technologie IoT Plug-and-Play n’utilise pas les fonctionnalités DTDL destinées aux graphiques, tandis qu’Azure Digital Twins n’implémente pas actuellement de commandes DTDL. Pour plus d’informations sur les fonctionnalités DTDL spécifiques d’Azure Digital Twins, consultez, plus loin dans cet article, la section concernant les [spécificités de l’implémentation du langage DTDL dans Azure Digital Twins](#azure-digital-twins-dtdl-implementation-specifics).

## <a name="elements-of-a-model"></a>Éléments d’un modèle

Dans une définition de modèle, l’élément de code de niveau supérieur est une **interface**. Celle-ci encapsule le modèle entier, le reste du modèle étant défini dans l’interface. 

Une interface de modèle DTDL peut contenir de zéro à un grand nombre des champs suivants :
* **Propriété** : certaines propriétés sont des champs de données représentant l’état d’une entité ; c’est par exemple le cas des propriétés dans de nombreux langages de programmation orientés objet. Contrairement à la télémétrie, qui est un événement de données lié au temps, les propriétés ont un stockage de sauvegarde et peuvent être lues à tout moment.
* **Télémétrie** : les champs de télémétrie représentent des mesures ou des événements, et sont souvent utilisés pour décrire des lectures de capteur d’appareil. La télémétrie n’est pas stockée sur un jumeau numérique, mais ressemble davantage à un flux d’événements de données prêts à être envoyés quelque part. 
* **Composant** : les composants vous permettent, le cas échéant, de créer votre interface de modèle en tant qu’assembly d’autres interfaces. Un exemple de composant est une interface *frontCamera*, ainsi qu’une autre interface de composant *backCamera*, utilisées pour la définition d’un modèle de *téléphone*. Vous devez commencer par définir une interface pour le composant *frontCamera*, comme s’il s’agissait de son propre modèle, puis la référencer lors de la définition de *téléphone*.

    Utilisez un composant pour décrire un élément qui fait partie intégrante de votre solution mais n’a pas besoin d’identité séparée, et qu’il n’est pas nécessaire de créer, supprimer ou réorganiser de manière indépendante dans le graphique de jumeaux. Si vous souhaitez que les entités aient des existences indépendantes dans le graphique de jumeaux, représentez-les en tant que jumeaux numériques séparés de différents modèles connectés par des *relations* (voir la puce suivante).
    
    >[!TIP] 
    >Des composants peuvent également être utilisés pour une organisation, afin de regrouper des ensembles de propriétés connexes à l’intérieur d’une interface de modèle. Dans ce cas, vous pouvez considérer chaque composant comme un espace de noms, ou « dossier », à l’intérieur de l’interface.
* **Relation** : les relations vous permettent de représenter la manière dont un jumeau numérique peut être impliqué avec d’autres jumeaux numériques. Elles peuvent avoir différents sens sémantiques. Par exemple *contains* (« floor contains room »), *cools* (« hvac cools room »), *isBilledTo* (« compressor is billed to user »), etc. Les relations permettent à la solution de fournir un graphique d’entités associées.

> [!NOTE]
> La spec pour le langage DTDL définit également des **commandes**, c’est-à-dire des méthodes qui peuvent être exécutées sur un jumeau numérique (comme une commande de réinitialisation ou d’allumage/extinction d’un ventilateur). Cependant, les *commandes ne sont actuellement pas prises en charge dans Azure Digital Twins*.

### <a name="azure-digital-twins-dtdl-implementation-specifics"></a>Spécificités de l’implémentation du langage DTDL dans Azure Digital Twins

Pour qu’un modèle DTDL soit compatible avec Azure Digital Twins, il doit répondre à ces exigences.

* Tous les éléments DTDL de niveau supérieur dans un modèle doivent être de type *interface*. Cela est dû au fait que les API de modèle Azure Digital Twins peuvent recevoir des objets JSON représentant une interface ou un tableau d’interfaces. Par conséquent, aucun autre type d’élément DTDL n’est autorisé au niveau supérieur.
* Le langage DTDL pour Azure Digital Twins ne doit pas définir de *commandes*.
* Azure Digital Twins n’autorise qu’un seul niveau d’imbrication de composant. Cela signifie qu’une interface utilisée en tant que composant ne peut pas avoir de composants. 
* Les interfaces ne peuvent pas être définies comme incluses dans d’autres interfaces DTDL. Elles doivent être définies en tant qu’entités de niveau supérieur distinctes avec leurs propres ID. Ensuite, si une autre interface souhaite inclure cette interface en tant que composant ou par héritage, elle peut référencer son ID.

## <a name="example-model-code"></a>Exemple de code de modèle

Des modèles de type de jumeau peuvent être écrits dans n’importe quel éditeur de texte. Le langage DTDL suivant la syntaxe JSON, vous devez donc stocker les modèles avec l’extension *.json*. L’utilisation de l’extension JSON permet à de nombreux éditeurs de texte de programmation de fournir une vérification et une mise en évidence de syntaxe de base pour vos documents DTDL. Il existe également une [extension DTDL](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) disponible pour [Visual Studio Code](https://code.visualstudio.com/).

Cette section contient un exemple de modèle classique, écrit sous la forme d’une interface DTDL. Le modèle décrit des **planètes**, chacune avec un nom, une masse et une température.
 
Sachez que les planètes peuvent également interagir avec des **lunes** qui sont leurs satellites et qu’elles peuvent contenir des **cratères**. Dans l’exemple ci-dessous, le modèle de `Planet` exprime les connexions à ces autres entités en référençant deux modèles externes, `Moon` et `Crater`. Ces modèles sont également définis dans l’exemple de code ci-dessous, mais ils sont très simples pour ne pas porter atteinte à l’exemple de `Planet` principal.

```json
[
  {
    "@id": "dtmi:com:contoso:Planet;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "displayName": "Planet",
    "contents": [
      {
        "@type": "Property",
        "name": "name",
        "schema": "string"
      },
      {
        "@type": "Property",
        "name": "mass",
        "schema": "double"
      },
      {
        "@type": "Telemetry",
        "name": "Temperature",
        "schema": "double"
      },
      {
        "@type": "Relationship",
        "name": "satellites",
        "target": "dtmi:com:contoso:Moon;1"
      },
      {
        "@type": "Component",
        "name": "deepestCrater",
        "schema": "dtmi:com:contoso:Crater;1"
      }
    ]
  },
  {
    "@id": "dtmi:com:contoso:Crater;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2"
  },
  {
    "@id": "dtmi:com:contoso:Moon;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2"
  }
]
```

Les champs du modèle sont les suivants :

| Champ | Description |
| --- | --- |
| `@id` | Identificateur pour le modèle. Doit être au format `dtmi:<domain>:<unique model identifier>;<model version number>`. |
| `@type` | Identifie le type des informations décrites. Pour une interface, le type est *interface*. |
| `@context` | Définit le [contexte](https://niem.github.io/json/reference/json-ld/context/) pour le document JSON. Les modèles doivent utiliser `dtmi:dtdl:context;2`. |
| `displayName` | [facultatif] Vous permet, le cas échéant, d’attribuer au modèle un nom convivial. |
| `contents` | Toutes les données d’interface restantes sont placées ici, sous la forme d’un tableau de définitions d’attributs. Chaque attribut doit fournir un `@type` (*Property*, *Telemetry*, *Command*, *Relationship* ou *Component*) pour identifier le type d’informations d’interface qu’il décrit, puis un ensemble de propriétés qui définissent l’attribut réel (par exemple, `name` et `schema` pour définir une *Propriété*). |

> [!NOTE]
> Notez que l’interface de composant (*Crater* dans cet exemple) est définie dans le même tableau que l’interface qui l’utilise (*Planet*). Les composants doivent être définis de cette façon dans les appels d’API, afin que l’interface soit trouvée.

### <a name="possible-schemas"></a>Schémas possibles

Comme pour langage DTDL, le schéma pour les attributs *Property* et *Telemetry* peuvent être composés de types primitifs standard (`integer`, `double`, `string` et `Boolean`) et d’autres types tels que `DateTime` et `Duration`. 

En plus des types primitifs, les champs *Property* et *Telemetry* peuvent avoir les types complexes suivants :
* `Object`
* `Map`
* `Enum`

Les champs *Telemetry* prennent également en charge `Array`.

### <a name="model-inheritance"></a>Héritage de modèle

Il est parfois nécessaire de spécialiser un modèle. Par exemple, il peut être utile de disposer d’un modèle générique, *Room*, et de variantes spécialisées, *ConferenceRoom* et *Gym*. Pour exprimer la spécialisation, le langage DTDL prend en charge l’héritage : des interfaces peuvent hériter d’une ou plusieurs autres interfaces. 

L’exemple suivant revisite le modèle *Planet* de l’exemple de DTDL précédent en tant que sous-type d’un modèle *CelestialBody* plus grand. Une fois le modèle « parent » défini, il est utilisé pour élaborer le modèle « enfant » à l’aide du champ `extends`.

```json
[
  {
    "@id": "dtmi:com:contoso:CelestialBody;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "displayName": "Celestial body",
    "contents": [
      {
        "@type": "Property",
        "name": "name",
        "schema": "string"
      },
      {
        "@type": "Property",
        "name": "mass",
        "schema": "double"
      },
      {
        "@type": "Telemetry",
        "name": "temperature",
        "schema": "double"
      }
    ]
  },
  {
    "@id": "dtmi:com:contoso:Planet;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "displayName": "Planet",
    "extends": "dtmi:com:contoso:CelestialBody;1",
    "contents": [
      {
        "@type": "Relationship",
        "name": "satellites",
        "target": "dtmi:com:contoso:Moon;1"
      },
      {
        "@type": "Component",
        "name": "deepestCrater",
        "schema": "dtmi:com:contoso:Crater;1"
      }
    ]
  },
  {
    "@id": "dtmi:com:contoso:Crater;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2"
  }
]
```

Dans cet exemple, le modèle *CelestialBody* fournit un nom, une masse et une température au modèle *Planet*. La section `extends` est un nom d’interface ou un tableau de noms d’interface (permettant, le cas échéant, à l’interface d’extension d’hériter de plusieurs modèles parents).

Une fois l’héritage appliqué, l’interface d’extension expose toutes les propriétés de la chaîne d’héritage entière.

L’interface d’extension ne peut pas modifier les définitions des interfaces parentes, mais simplement y faire des ajouts. Elle ne peut pas non plus redéfinir une fonctionnalité déjà définie dans ses interfaces parentes (même si les fonctionnalités sont définies pour être identiques). Par exemple, si une interface parente définit une propriété `double` *mass*, l’interface d’extension ne peut pas contenir une déclaration de *mass*, même s’il s’agit également d’un `double`.

## <a name="validating-models"></a>Validation des modèles

> [!TIP]
> Il est recommandé de valider les modèles hors connexion avant de les charger sur une instance Azure Digital Twins.

Il existe un exemple de validation de documents de modèle indépendant du langage permettant de vérifier que le langage DTDL est correct. Il se trouve ici : [**Exemple de validateur DTDL**](https://docs.microsoft.com/samples/azure-samples/dtdl-validator/dtdl-validator).

L’exemple de validateur DTDL s’appuie sur une bibliothèque d’analyseur DTDL .NET, disponible sur NuGet comme bibliothèque côté client : [**Microsoft.Azure.DigitalTwins.Parser**](https://nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser/). Vous pouvez également vous servir directement de la bibliothèque pour concevoir votre propre solution de validation. Dans ce cas, veillez à utiliser une version compatible avec celle d’Azure Digital Twins. Pendant la préversion, il s’agit de la version *3.7.0*.

Pour plus d’informations sur la bibliothèque d’analyseur, et notamment des exemples d’utilisation, consultez [*Guide pratique pour analyser et valider les modèles*](how-to-use-parser.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment gérer les modèles avec les API de DigitalTwinsModels :
* [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md)

Vous pouvez également découvrir comment des jumeaux numériques sont créés à partir de modèles :
* [*Concepts : Jumeaux numériques et graphe de jumeaux*](concepts-twins-graph.md)

