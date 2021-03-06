---
title: Interroger des fichiers Parquet à l’aide de SQL à la demande (préversion)
description: Cet article vous explique comment interroger des fichiers Parquet à l’aide de SQL à la demande (préversion).
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: dd1e387727b0a80781b1103ddfb40afcbce8fce8
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386620"
---
# <a name="query-parquet-files-using-sql-on-demand-preview-in-azure-synapse-analytics"></a>Interroger des fichiers Parquet à l’aide de SQL à la demande (préversion) dans Azure Synapse Analytics

Cet article vous explique comment écrire une requête à l’aide de SQL à la demande (préversion) pour lire des fichiers Parquet.

## <a name="quickstart-example"></a>Exemple de démarrage rapide

La fonction `OPENROWSET` vous permet de lire le contenu d’un fichier Parquet en fournissant l’URL de votre fichier.

### <a name="reading-parquet-file"></a>Lecture de fichier Parquet

Le moyen le plus simple d’afficher le contenu de votre fichier `PARQUET` consiste à fournir l’URL du fichier à la fonction `OPENROWSET` et à spécifier le `FORMAT` Parquet. Si le fichier est disponible publiquement ou si votre identité Azure AD peut y accéder, vous devriez pouvoir voir le contenu du fichier à l’aide d’une requête comme celle montrée dans l’exemple suivant :

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
    format = 'parquet') as rows
```

Assurez-vous que vous pouvez accéder à ce fichier. Si votre fichier est protégé par une clé SAP ou une identité Azure personnalisée, vous devez configurer les [informations d’identification au niveau du serveur pour la connexion SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

### <a name="using-data-source"></a>Utilisation d’une source de données

L’exemple précédent utilise le chemin complet du fichier. Vous pouvez également créer une source de données externe avec l’emplacement qui pointe vers le dossier racine du stockage et utiliser cette source de données et le chemin relatif du fichier dans la fonction `OPENROWSET` :

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) as rows
```

Si une source de données est protégée par une clé SAP ou une identité personnalisée, vous pouvez configurer la [source de données avec des informations d’identification dans l’étendue de la base de données](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Spécifier explicitement le schéma

`OPENROWSET` vous permet de spécifier explicitement les colonnes que vous souhaitez lire à partir du fichier à l’aide de la clause `WITH` :

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) with ( date_rep date, cases int, geo_id varchar(6) ) as rows
```

Dans les sections suivantes, vous pouvez voir comment interroger différents types de fichiers Parquet.

## <a name="prerequisites"></a>Prérequis

La première étape consiste à **créer une base de données** avec une source de données qui fait référence au compte de stockage [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Ensuite, initialisez les objets en exécutant le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur cette base de données. Ce script d’installation crée les sources de données, les informations d’identification délimitées à la base de données et les formats de fichiers externes utilisés dans ces exemples.

## <a name="dataset"></a>Dataset

Le jeu de données [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) est utilisé dans cet exemple. Vous pouvez interroger les fichiers Parquet de la même façon que vous [lisez des fichiers CSV](query-parquet-files.md). La seule différence réside dans le fait que le paramètre `FILEFORMAT` doit être défini sur `PARQUET`. Les exemples de cet article montrent les spécificités de lecture des fichiers Parquet.

## <a name="query-set-of-parquet-files"></a>Ensemble de requêtes de fichiers Parquet

Vous pouvez spécifier uniquement les colonnes intéressantes lorsque vous interrogez des fichiers Parquet.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) WITH (
        tpepPickupDateTime DATETIME2,
        passengerCount INT
    ) AS nyc
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="automatic-schema-inference"></a>Inférence de schéma automatique

Vous n’avez pas besoin d'utiliser la clause OPENROWSET WITH lors de la lecture de fichiers Parquet. Les noms de colonnes et les types de données sont automatiquement lus à partir des fichiers Parquet.

L’exemple ci-dessous montre les fonctionnalités d’inférence automatique du schéma des fichiers Parquet. Il renvoie le nombre de lignes en septembre 2017 sans spécifier de schéma.

> [!NOTE]
> Vous n’avez pas besoin de spécifier de colonnes dans la clause OPENROWSET WITH lors de la lecture de fichiers Parquet. Dans ce cas, le service de requête SQL à la demande utilise les métadonnées dans le fichier Parquet et lie les colonnes par nom.

```sql
SELECT TOP 10 *
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) AS nyc
```

### <a name="query-partitioned-data"></a>Interroger des données partitionnées

Le jeu de données fourni dans cet exemple est divisé (partitionné) en sous-dossiers distincts. Utilisez la fonction filepath pour cibler des partitions spécifiques. Cet exemple montre les montants par année, par mois et par type de paiement pour les trois premiers mois de 2017.

> [!NOTE]
> La requête SQL à la demande est compatible avec le schéma de partitionnement Hive/Hadoop.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=*/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) nyc
WHERE
    nyc.filepath(1) = 2017
    AND nyc.filepath(2) IN (1, 2, 3)
    AND tpepPickupDateTime BETWEEN CAST('1/1/2017' AS datetime) AND CAST('3/31/2017' AS datetime)
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="type-mapping"></a>Mappage des types

Les fichiers Parquet contiennent des descriptions de type pour chaque colonne. Le tableau suivant explique comment les types Parquet sont mappés aux types SQL natifs.

| Type Parquet | Type logique Parquet (annotation) | Type de données SQL |
| --- | --- | --- |
| BOOLEAN | | bit |
| BINARY / BYTE_ARRAY | | varbinary |
| DOUBLE | | float |
| FLOAT | | real |
| INT32 | | int |
| INT64 | | bigint |
| INT96 | |datetime2 |
| FIXED_LEN_BYTE_ARRAY | |binary |
| BINARY |UTF8 |varchar \*(classement UTF8) |
| BINARY |STRING |varchar \*(classement UTF8) |
| BINARY |ENUM|varchar \*(classement UTF8) |
| BINARY |UUID |UNIQUEIDENTIFIER |
| BINARY |DECIMAL |Décimal |
| BINARY |JSON |varchar (max) \*(classement UTF8) |
| BINARY |BSON |varbinary(max) |
| FIXED_LEN_BYTE_ARRAY |DECIMAL |Décimal |
| BYTE_ARRAY |INTERVAL |varchar (max), sérialisé au format standardisé |
| INT32 |INT(8, true) |SMALLINT |
| INT32 |INT(16, true) |SMALLINT |
| INT32 |INT(32, true) |int |
| INT32 |INT(8, false) |TINYINT |
| INT32 |INT(16, false) |int |
| INT32 |INT(32, false) |bigint |
| INT32 |DATE |Date |
| INT32 |DECIMAL |Décimal |
| INT32 |TIME (MILLIS )|time |
| INT64 |INT(64, true) |bigint |
| INT64 |INT(64, false ) |decimal(20,0) |
| INT64 |DECIMAL |Décimal |
| INT64 |TIME (MICROS / NANOS) |time |
|INT64 |TIMESTAMP (MILLIS / MICROS / NANOS) |datetime2 |
|[Type complexe](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#lists) |Liste |varchar(max), sérialisé en JSON |
|[Type complexe](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#maps)|MAP|varchar(max), sérialisé en JSON |

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour savoir comment [interroger des types imbriqués Parquet](query-parquet-nested-types.md).
