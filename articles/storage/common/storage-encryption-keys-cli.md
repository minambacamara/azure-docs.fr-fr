---
title: Utiliser Azure CLI pour configurer des clés gérées par le client
titleSuffix: Azure Storage
description: Découvrez comment utiliser Azure CLI afin de configurer des clés gérées par le client avec Azure Key Vault pour le chiffrement du stockage Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 4d54a8bf9fb5a1f31a29e41ecea545b43dbe58cf
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/28/2020
ms.locfileid: "87276684"
---
# <a name="configure-customer-managed-keys-with-azure-key-vault-by-using-azure-cli"></a>Configurer des clés gérées par le client avec Azure Key Vault à l’aide d’Azure CLI

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

Cet article explique comment configurer un coffre de clés Azure avec des clés gérées par le client à l’aide d’Azure CLI. Pour en savoir plus sur la création d’un coffre de clés à l’aide d’Azure CLI, consultez [Démarrage rapide : Définir et récupérer un secret depuis Azure Key Vault à l’aide d’Azure CLI](../../key-vault/secrets/quick-create-cli.md).

## <a name="assign-an-identity-to-the-storage-account"></a>Affecter une identité au compte de stockage

Pour activer des clés gérées par le client pour votre compte de stockage, affectez tout d’abord à votre compte de stockage une identité managée affectée au système. Cette identité managée vous sera utile pour autoriser le compte de stockage à accéder au coffre de clés.

Pour attribuer une identité managée à l’aide d’Azure CLI, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
az login
az account set --subscription <subscription-id>

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

Pour plus d’informations sur la configuration d’identités managées affectées à un système avec Azure CLI, consultez [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md).

## <a name="create-a-new-key-vault"></a>Créer un coffre de clés

Le coffre de clés que vous utilisez pour stocker des clés gérées par le client pour le chiffrement du stockage Azure doit disposer de deux paramètres de protection de clés, **Suppression réversible** et **Ne pas vider**. Pour créer un coffre de clés à l’aide de PowerShell ou Azure CLI avec ces paramètres sont activés, exécutez les commandes suivantes. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

Pour créer un coffre de clés avec Azure CLI, appelez [az keyvault create](/cli/azure/keyvault#az-keyvault-create). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-soft-delete \
    --enable-purge-protection
```

Pour savoir comment activer les options **Suppression réversible** et les **Ne pas vider** sur un coffre de clés existant avec Azure CLI, consultez les sections intitulées **Activation de la suppression réversible** et **Activation de la protection contre le vidage** dans [Guide pratique pour utiliser la suppression réversible avec CLI](../../key-vault/general/soft-delete-cli.md).

## <a name="configure-the-key-vault-access-policy"></a>Configurer la stratégie d’accès au coffre de clés

Configurez ensuite la stratégie d’accès au coffre de clés de sorte que le compte de stockage dispose des autorisations nécessaires pour y accéder. À cette étape, vous utiliserez l’identité managée que vous avez précédemment affectée au compte de stockage.

Pour définir la stratégie d’accès au coffre de clés, appelez [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

## <a name="create-a-new-key"></a>Créer une clé

Créez ensuite une clé dans le coffre de clés. Pour créer une clé, appelez [az keyvault key create](/cli/azure/keyvault/key#az-keyvault-key-create). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
az keyvault key create \
    --name <key> \
    --vault-name <key-vault>
```

Le chiffrement du stockage Azure prend en charge les clés RSA et RSA-HSM dans les tailles 2048, 3072 et 4096. Pour plus d’informations sur les clés, consultez **Clés Key Vault** dans [À propos des clés, des secrets et des certificats Azure Key Vault](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys).

## <a name="configure-encryption-with-customer-managed-keys"></a>Configurer le chiffrement avec les clés gérées par le client

Par défaut, le chiffrement du stockage Azure utilise des clés gérées par Microsoft. Dans cette étape, configurez votre compte de stockage Azure pour utiliser des clés gérées par le client à l’aide d’Azure Key Vault, puis spécifiez la clé à lui associer.

Lorsque vous configurez le chiffrement avec des clés gérées par le client, vous pouvez choisir de faire pivoter automatiquement la clé utilisée pour le chiffrement lorsque la version change dans le coffre de clés associé. Vous pouvez également spécifier explicitement une version de clé à utiliser pour le chiffrement jusqu’à ce que la version de clé soit mise à jour manuellement.

### <a name="configure-encryption-for-automatic-rotation-of-customer-managed-keys"></a>Configurer le chiffrement pour la rotation automatique des clés gérées par le client

Afin de configurer le chiffrement avec rotation automatique des clés gérées par le client, installez [Azure CLI version 2.4.0](/cli/azure/release-notes-azure-cli#april-21-2020) ou une version ultérieure. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

Pour faire pivoter automatiquement les clés gérées par le client, omettez la version de clé lorsque vous configurez des clés gérées par le client pour le compte de stockage. Pour mettre à jour les paramètres de chiffrement du compte de stockage, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update), comme illustré dans l’exemple suivant. Incluez le paramètre `--encryption-key-source` et définissez-le sur `Microsoft.Keyvault` pour activer les clés gérées par le client pour le compte. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

### <a name="configure-encryption-for-manual-rotation-of-key-versions"></a>Configurer le chiffrement pour la rotation manuelle des versions de clé

Pour spécifier explicitement une version de clé à utiliser pour le chiffrement, fournissez la version de clé lorsque vous configurez le chiffrement avec les clés gérées par le client pour le compte de stockage. Pour mettre à jour les paramètres de chiffrement du compte de stockage, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update), comme illustré dans l’exemple suivant. Incluez le paramètre `--encryption-key-source` et définissez-le sur `Microsoft.Keyvault` pour activer les clés gérées par le client pour le compte. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [-1].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

Lorsque vous faites pivoter manuellement la version de clé, vous devez mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version. Tout d’abord, lancez une requête pour l’URI du coffre de clés en appelant [az keyvault show](/cli/azure/keyvault#az-keyvault-show)et pour la version de la clé en appelant [az keyvault key list-versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Appelez ensuite [az storage account update](/cli/azure/storage/account#az-storage-account-update) pour mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version de la clé, comme indiqué dans l’exemple précédent.

## <a name="use-a-different-key"></a>Utiliser une autre clé

Pour modifier la clé utilisée pour le chiffrement de Stockage Azure, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update) comme indiqué dans [Configurer des clés de chiffrement gérées par le client](#configure-encryption-with-customer-managed-keys) et indiquez le nom et la version de la nouvelle clé. Si la nouvelle clé se trouve dans un coffre de clés différent, mettez également à jour l’URI du coffre de clés.

## <a name="revoke-customer-managed-keys"></a>Révoquer des clés gérées par le client

Vous pouvez révoquer des clés gérées par le client en supprimant la stratégie d’accès du coffre de clés. Pour révoquer une clé gérée par le client, appelez la commande [az keyvault delete-policy](/cli/azure/keyvault#az-keyvault-delete-policy), comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```azurecli-interactive
az keyvault delete-policy \
    --name <key-vault> \
    --object-id $storage_account_principal
```

## <a name="disable-customer-managed-keys"></a>Désactiver les clés gérées par le client

Quand vous désactivez les clés gérées par le client, votre compte de stockage est de nouveau chiffré avec des clés managées par Microsoft. Pour désactiver les clés gérées par le client, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update) et définissez le `--encryption-key-source parameter` sur `Microsoft.Storage`, comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-source Microsoft.Storage
```

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md) 
- [Qu’est-ce qu’Azure Key Vault ?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
