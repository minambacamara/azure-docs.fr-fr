---
title: 'Préversion : Créer une version d’image chiffrée avec vos propres clés'
description: Créez une version d’image dans une galerie d’images partagées à l’aide de clés de chiffrement gérées par le client.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/06/2020
ms.author: cynthn
ms.openlocfilehash: 469e225a1cc40dc2ecc45339d9355484e87c4af2
ms.sourcegitcommit: f844603f2f7900a64291c2253f79b6d65fcbbb0c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/10/2020
ms.locfileid: "86223582"
---
# <a name="preview-use-customer-managed-keys-for-encrypting-images"></a>Aperçu : Utiliser des clés gérées par le client pour le chiffrement d’images

Les images de la galerie sont stockées en tant que disques managés, de sorte qu’elles sont automatiquement chiffrées à l’aide du chiffrement côté serveur. Le chiffrement côté serveur utilise le [chiffrement AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256 bits, l’un des chiffrements par blocs les plus puissants actuellement disponibles, et est conforme à la norme FIPS 140-2. Pour plus d’informations sur les modules cryptographiques des disque managés Azure, consultez [API de chiffrement : nouvelle génération](/windows/desktop/seccng/cng-portal)

Vous pouvez vous appuyer sur des clés gérées par la plateforme pour le chiffrement de vos images, ou vous pouvez gérer le chiffrement en utilisant vos propres clés. Si vous choisissez de gérer le chiffrement avec vos propres clés, vous pouvez spécifier une *clé gérée par le client* à utiliser pour chiffrer et déchiffrer tous les disques de vos images. 

Le chiffrement côté serveur à l’aide de clés gérées par le client utilise Azure Key Vault. Vous pouvez importer [vos clés RSA](../key-vault/keys/hsm-protected-keys.md) vers votre Key Vault ou générer de nouvelles clés RSA dans Azure Key Vault.

Afin d’utiliser des clés gérées par le client pour les images, vous avez d’abord besoin d’un coffre de clé Azure. Vous créez ensuite un jeu de chiffrement de disque. Le jeu de chiffrement de disque est ensuite utilisé lors de la création de vos versions d’image.

Pour plus d’informations sur la création et l’utilisation de jeux de chiffrement de disque, consultez la rubrique [Clés gérées par le client](./windows/disk-encryption.md#customer-managed-keys).

## <a name="limitations"></a>Limites

L’utilisation de clés gérées par le client pour le chiffrement d’images de la galerie d’images partagées présente plusieurs inconvénients :  

- Les jeux de clés de chiffrement doivent être se trouver dans les mêmes région et abonnement que votre image.

- Vous ne pouvez pas partager des images qui utilisent des clés gérées par le client. 

- Vous ne pouvez pas répliquer des images qui utilisent des clés gérées par le client dans d’autres régions.

- Une fois que vous avez utilisé vos propres clés pour chiffrer un disque ou une image, vous ne pouvez pas revenir à l’utilisation de clés gérées par la plateforme pour chiffrer ces disques ou images.


> [!IMPORTANT]
> Le chiffrement à l’aide de clés gérées par le client est actuellement disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="powershell"></a>PowerShell

Pour la préversion publique, vous devez d’abord vous inscrire à la fonctionnalité.

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

L’inscription peut prendre quelques minutes. Utilisez Get-AzProviderFeature pour vérifier l’état de l’inscription à la fonctionnalité.

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

Quand RegistrationState retourne la valeur Registered, vous pouvez passer à l’étape suivante.

Vérifiez l’inscription de votre fournisseur. Assurez-vous qu’elle retourne `Registered`.

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState
```

Si elle ne retourne pas `Registered`, utilisez la commande suivante pour inscrire les fournisseurs :

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

Afin de spécifier un jeu de chiffrement de disque pour une version d’image, utilisez [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion) avec le paramètre `-TargetRegion`. 

```azurepowershell-interactive

$sourceId = <ID of the image version source>

$osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet'}

$dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet1';Lun=1}

$dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet2';Lun=2}

$dataDiskImageEncryptions = @($dataDiskImageEncryption1,$dataDiskImageEncryption2)

$encryption1 = @{OSDiskImage=$osDiskImageEncryption;DataDiskImages=$dataDiskImageEncryptions}

$region1 = @{Name='West US';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption1}

$targetRegion = @{$region1}


# Create the image
New-AzGalleryImageVersion `
   -ResourceGroupName $rgname `
   -GalleryName $galleryName `
   -GalleryImageDefinitionName $imageDefinitionName `
   -Name $versionName -Location $location `
   -SourceImageId $sourceId `
   -ReplicaCount 2 `
   -StorageAccountType Standard_LRS `
   -PublishingProfileEndOfLifeDate '2020-12-01' `
   -TargetRegion $targetRegion
```

### <a name="create-a-vm"></a>Créer une machine virtuelle

Vous pouvez créer une machine virtuelle à partir d’une galerie d’images partagées et utiliser des clés gérées par le client pour chiffrer les disques. La syntaxe est la même que pour créer une machine virtuelle [généralisée](vm-generalized-image-version-powershell.md) ou [spécialisée](vm-specialized-image-version-powershell.md) à partir d’une image : vous devez utiliser le jeu de paramètres étendus et ajouter `Set-AzVMOSDisk -Name $($vmName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage` à la configuration de la machine virtuelle.

Pour les disques de données, vous devez ajouter le paramètre `-DiskEncryptionSetId $setID` lorsque vous utilisez [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk).


## <a name="cli"></a>Interface de ligne de commande 

Pour la préversion publique, vous devez d’abord vous inscrire à la fonctionnalité.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name SIGEncryption
```

Vérifiez l’état d’inscription de la fonctionnalité.

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name SIGEncryption | grep state
```

Lorsque l’inscription retourne `"state": "Registered"`, vous pouvez passer à l’étape suivante.

Vérifiez votre inscription.

```azurecli-interactive
az provider show -n Microsoft.Compute | grep registrationState
```

Si elle ne retourne pas Registered, exécutez la commande suivante :

```azurecli-interactive
az provider register -n Microsoft.Compute
```


Afin de spécifier un jeu de chiffrement de disque pour une version d’image, utilisez [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create) avec le paramètre `--target-region-encryption`. Le format pour `--target-region-encryption` est une liste de clés, séparée par des espaces, pour le chiffrement des disques de système d’exploitation et de données. Ce que vous obtenez doit ressembler à ceci : `<encryption set for the OS disk>,<Lun number of the data disk>, <encryption set for the data disk>, <Lun number for the second data disk>, <encryption set for the second data disk>`. 

Si la source du disque du système d’exploitation est un disque managé ou une machine virtuelle, utilisez `--managed-image` pour spécifier la source de la version de l’image. Dans cet exemple, la source est une image managée qui possède un disque de système d’exploitation et un disque de données au numéro d’unité logique 0. Le disque du système d’exploitation sera chiffré avec DiskEncryptionSet1 et le disque de données avec DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --target-regions westus=2=standard_lrs \
   --target-region-encryption DiskEncryptionSet1,0,DiskEncryptionSet2 \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage \
   --managed-image "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
```

Si la source du disque du système d’exploitation est un instantané, utilisez `--os-snapshot` pour spécifier le disque du système d’exploitation. Si des instantanés de disque de données doivent également faire partie de la version de l’image, ajoutez-les à l’aide de `--data-snapshot-luns` pour spécifier le numéro d’unité logique et `--data-snapshots` pour spécifier les instantanés.

Dans cet exemple, les sources sont des instantanés de disque. Le numéro d’unité logique 0 comporte un disque de système d’exploitation et un disque de données. Le disque du système d’exploitation sera chiffré avec DiskEncryptionSet1 et le disque de données avec DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --target-regions westus=2=standard_lrs \
   --target-region-encryption DiskEncryptionSet1,0,DiskEncryptionSet2 \
   --os-snapshot "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myOSSnapshot"
   --data-snapshot-luns 0
   --data-snapshots "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myDDSnapshot"
   --gallery-name MyGallery \
   --gallery-image-definition MyImage 
   
```

### <a name="create-the-vm"></a>Création de la machine virtuelle

Vous pouvez créer une machine virtuelle à partir d’une galerie d’images partagées et utiliser des clés gérées par le client pour chiffrer les disques. La syntaxe est la même que pour créer une machine virtuelle [généralisée](vm-generalized-image-version-cli.md) ou [spécialisée](vm-specialized-image-version-cli.md) à partir d’une image. Vous devez simplement ajouter le paramètre `--os-disk-encryption-set` avec l’ID du jeu de chiffrement. Pour les disques de données, ajoutez `--data-disk-encryption-sets` avec une liste des jeux de chiffrement de disque, délimitée par des espaces.


## <a name="portal"></a>Portail

Lorsque vous créez votre version d’image dans le portail, vous pouvez utiliser l’onglet **Chiffrement** pour entrer les informations relatives à vos jeux de chiffrement de stockage.

1. Dans la page **Créer une version d’image**, sélectionnez l’onglet **Chiffrement**.
2. Dans **Type de chiffrement**, sélectionnez **Chiffrement au repos avec une clé gérée par le client**. 
3. Pour chaque disque de l’image, sélectionnez le **jeu de chiffrement de disque** à utiliser dans la liste déroulante. 

### <a name="create-the-vm"></a>Création de la machine virtuelle

Vous pouvez créer une machine virtuelle à partir d’une galerie d’images partagées et utiliser des clés gérées par le client pour chiffrer les disques. Lorsque vous créez la machine virtuelle dans le portail, sous l’onglet **Disques**, sélectionnez **Chiffrement au repos avec des clés gérées par le client** pour le **type de chiffrement**. Vous pouvez ensuite sélectionner le jeu de chiffrement dans la liste déroulante.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le [chiffrement de disque côté serveur](./windows/disk-encryption.md).

Pour savoir comment fournir des informations sur le plan d'achat, consultez [Supply Azure Marketplace purchase plan information when creating images](marketplace-images.md) (Donner des informations sur le plan d'achat de la place de marché Azure lors de la création d’images).
