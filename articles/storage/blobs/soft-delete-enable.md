---
title: Activer et gérer la suppression réversible pour les blobs
titleSuffix: Azure Storage
description: Activez la suppression réversible pour les blobs afin de récupérer plus facilement vos données en cas de modification ou de suppression malencontreuses.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/15/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli
ms.openlocfilehash: 182679c89482b269319a4d0747b6d83bb19b9c5c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498172"
---
# <a name="enable-and-manage-soft-delete-for-blobs"></a>Activer et gérer la suppression réversible pour les blobs

La suppression réversible protège les données de l’objet BLOB contre les modifications ou suppressions accidentelles ou erronées. Lorsque la suppression réversible est activée pour un compte de stockage, les objets BLOB, les versions d’objets BLOB (préversion) et les captures instantanées de ce compte de stockage peuvent être récupérés après leur suppression, au cours d’une période de rétention que vous spécifiez.

S’il existe une possibilité de modification ou de suppression accidentelles de vos données par une application ou un autre utilisateur du compte de stockage, Microsoft vous recommande d’activer la suppression réversible.

Cet article explique comment prendre en main la suppression réversible.

## <a name="enable-soft-delete"></a>Activer la suppression réversible

# <a name="portal"></a>[Portail](#tab/azure-portal)

Activez la suppression réversible pour blob sur votre compte de stockage à l’aide du Portail Azure :

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre compte de stockage. 

2. Accédez à l’option **Protection des données** sous **Service blob**.

3. Cliquez sur **Activé** sous **Suppression réversible blob**

4. Entrez le nombre de jours que vous souhaitez *conserver* sous **Stratégies de rétention**

5. Choisissez le bouton **Enregistrer** pour confirmer vos paramètres de protection des données

![Capture d’écran du portail Azure avec le service Blob de protection des données sélectionné.](media/soft-delete-enable/storage-blob-soft-delete-portal-configuration.png)

Pour afficher les objets blob supprimés de manière réversible, activez la case à cocher **Afficher les objets blob supprimés**.

![Capture d’écran de la page du service Blob de protection des données avec l’option Afficher les objets blob supprimés mise en évidence.](media/soft-delete-enable/storage-blob-soft-delete-portal-view-soft-deleted.png)

Pour afficher les instantanés d’objets blob supprimés de manière réversible pour un objet blob donné, sélectionnez celui-ci, puis cliquez sur **Afficher les instantanés**.

![Capture d’écran de la page du service Blob de protection des données avec l’option Afficher les instantanés mise en évidence.](media/soft-delete-enable/storage-blob-soft-delete-portal-view-soft-deleted-snapshots.png)

Assurez-vous que la case à cocher **Afficher les instantanés supprimés** est activée.

![Capture d’écran de la page Afficher les instantanés avec l’option Afficher les objets blob supprimés mise en évidence.](media/soft-delete-enable/storage-blob-soft-delete-portal-view-soft-deleted-snapshots-check.png)

Lorsque vous cliquez sur un blob ou un instantané d’objet blob supprimés de manière réversible, notez les nouvelles propriétés de l’objet blob. Elles indiquent quand l’objet a été supprimé, et le nombre de jours restants avant l’expiration définitive de l’objet blob ou de l’instantané d’objet blob. Si l’objet supprimé de manière réversible n’est pas un instantané, vous avez également la possibilité d’annuler sa suppression.

![Capture d’écran des détails d’un objet supprimé de manière réversible.](media/soft-delete-enable/storage-blob-soft-delete-portal-properties.png)

N’oubliez pas que l’annulation de la suppression d’un objet blob a également pour effet d’annuler la suppression de tous les instantanés associés. Pour annuler la suppression des instantanés d’objets blob supprimés de manière réversible pour un objet blob actif, cliquez sur celui-ci, puis sélectionnez **Annuler la suppression de tous les instantanés**.

![Capture d’écran des détails d’un objet blob supprimé de manière réversible.](media/soft-delete-enable/storage-blob-soft-delete-portal-undelete-all-snapshots.png)

Après avoir annulé la suppression d’instantanés d’un objet blob, vous pouvez cliquer sur **Promouvoir** pour copier un instantané sur la racine du blob, ce qui a pour effet de restaurer l’objet blob à son instantané.

![Capture d’écran de la page Afficher les instantanés avec l’option Promouvoir mise en évidence.](media/soft-delete-enable/storage-blob-soft-delete-portal-promote-snapshot.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour activer la suppression réversible, mettez à jour les propriétés du service du client d’un objet blob. L’exemple suivant active la suppression réversible pour un sous-ensemble de comptes dans un abonnement :

```powershell
Set-AzContext -Subscription "<subscription-name>"
$MatchingAccounts = Get-AzStorageAccount | where-object{$_.StorageAccountName -match "<matching-regex>"}
$MatchingAccounts | Enable-AzStorageDeleteRetentionPolicy -RetentionDays 7
```

Vous pouvez vérifier que la suppression réversible a été activée à l’aide de la commande suivante :

```powershell
$MatchingAccounts | $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context | Select-Object -ExpandProperty DeleteRetentionPolicy
```

Pour récupérer des objets blob supprimés accidentellement, appelez la commande Undelete sur ceux-ci. N’oubliez pas que l’appel de la commande **Undelete Blob** sur les objets blob tant actifs que supprimés de manière réversible, a pour effet de restaurer comme actifs tous les instantanés d’objets blob supprimés de manière réversible. L’exemple suivant appelle la commande Undelete sur tous les objets blob, tant actifs que supprimés de manière réversible, figurant dans un conteneur :

```powershell
# Create a context by specifying storage account name and key
$ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

# Get the blobs in a given container and show their properties
$Blobs = Get-AzStorageBlob -Container $StorageContainerName -Context $ctx -IncludeDeleted
$Blobs.ICloudBlob.Properties

# Undelete the blobs
$Blobs.ICloudBlob.Undelete()
```
Pour rechercher la stratégie actuelle de conservation de suppression réversible, utilisez la commande suivante :

```azurepowershell-interactive
   $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-CLI)

Pour activer la suppression réversible, mettez à jour les propriétés du service du client d’un objet blob :

```azurecli-interactive
az storage blob service-properties delete-policy update --days-retained 7  --account-name mystorageaccount --enable true
```

Pour vérifier si la suppression réversible est activée, utilisez la commande suivante : 

```azurecli-interactive
az storage blob service-properties delete-policy show --account-name mystorageaccount 
```

# <a name="python"></a>[Python](#tab/python)

Pour activer la suppression réversible, mettez à jour les propriétés du service du client d’un objet blob :

```python
# Make the requisite imports
from azure.storage.blob import BlockBlobService
from azure.storage.common.models import DeleteRetentionPolicy

# Initialize a block blob service
block_blob_service = BlockBlobService(
    account_name='<enter your storage account name>', account_key='<enter your storage account key>')

# Set the blob client's service property settings to enable soft delete
block_blob_service.set_blob_service_properties(
    delete_retention_policy=DeleteRetentionPolicy(enabled=True, days=7))
```

# <a name="net-v12-sdk"></a>[Kit de développement logiciel (SDK) .NET v12](#tab/dotnet)

Pour activer la suppression réversible, mettez à jour les propriétés du service du client d’un objet blob :

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_EnableSoftDelete":::

Pour récupérer des objets blob supprimés accidentellement, appelez la commande Undelete sur ceux-ci. N’oubliez pas que l’appel de la commande **Undelete** sur les blobs tant actifs que supprimés de manière réversible a pour effet de restaurer comme actifs tous les instantanés supprimés de manière réversible qui sont associés. L’exemple suivant appelle la commande Undelete sur tous les objets blob, tant actifs que supprimés de manière réversible, figurant dans un conteneur :

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverDeletedBlobs":::

Pour récupérer une version spécifique d’un objet blob, commencez par appeler la commande Undelete sur un objet blob, puis copiez l’instantané souhaité sur l’objet blob. L’exemple suivant récupère un objet blob de blocs en restaurant son instantané généré le plus récemment :

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverSpecificBlobVersion":::

# <a name="net-v11-sdk"></a>[Kit de développement logiciel (SDK) .NET v11](#tab/dotnet11)

Pour activer la suppression réversible, mettez à jour les propriétés du service du client d’un objet blob :

```csharp
// Get the blob client's service property settings
ServiceProperties serviceProperties = blobClient.GetServiceProperties();

// Configure soft delete
serviceProperties.DeleteRetentionPolicy.Enabled = true;
serviceProperties.DeleteRetentionPolicy.RetentionDays = RetentionDays;

// Set the blob client's service property settings
blobClient.SetServiceProperties(serviceProperties);
```

Pour récupérer des objets blob supprimés accidentellement, appelez la commande Undelete sur ceux-ci. N’oubliez pas que l’appel de la commande **Undelete** sur les blobs tant actifs que supprimés de manière réversible a pour effet de restaurer comme actifs tous les instantanés supprimés de manière réversible qui sont associés. L’exemple suivant appelle la commande Undelete sur tous les objets blob, tant actifs que supprimés de manière réversible, figurant dans un conteneur :

```csharp
// Recover all blobs in a container
foreach (CloudBlob blob in container.ListBlobs(useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Deleted))
{
       await blob.UndeleteAsync();
}
```

Pour récupérer une version spécifique d’un objet blob, commencez par appeler la commande Undelete sur un objet blob, puis copiez l’instantané souhaité sur l’objet blob. L’exemple suivant récupère un objet blob de blocs en restaurant son instantané généré le plus récemment :

```csharp
// Undelete
await blockBlob.UndeleteAsync();

// List all blobs and snapshots in the container prefixed by the blob name
IEnumerable<IListBlobItem> allBlobVersions = container.ListBlobs(
    prefix: blockBlob.Name, useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Snapshots);

// Restore the most recently generated snapshot to the active blob    
CloudBlockBlob copySource = allBlobVersions.First(version => ((CloudBlockBlob)version).IsSnapshot &&
    ((CloudBlockBlob)version).Name == blockBlob.Name) as CloudBlockBlob;
blockBlob.StartCopy(copySource);
```  

---

## <a name="next-steps"></a>Étapes suivantes

- [Suppression réversible pour le stockage de blobs](soft-delete-overview.md)
- [Contrôle de version des objets blob (préversion)](versioning-overview.md)
