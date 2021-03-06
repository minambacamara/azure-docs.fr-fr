---
title: Créer un espace de travail avec un modèle Azure Resource Manager
titleSuffix: Azure Machine Learning
description: Découvrez la manière d’utiliser un modèle Azure Resource Manager pour créer un espace de travail Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.author: larryfr
author: Blackmist
ms.date: 07/27/2020
ms.openlocfilehash: 06ab819065f96508bcc4ebd26371c743c89b9220
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/31/2020
ms.locfileid: "87487800"
---
# <a name="use-an-azure-resource-manager-template-to-create-a-workspace-for-azure-machine-learning"></a>Utiliser un modèle Azure Resource Manager pour créer un espace de travail pour Azure Machine Learning

[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]
<br>

Vous trouverez dans cet article différentes façons de créer un espace de travail Azure Machine Learning à l’aide de modèles Azure Resource Manager. Un modèle Resource Manager permet de créer des ressources sous la forme d’une seule opération coordonnée. Un modèle est un document JSON qui définit les ressources nécessaires à un déploiement. Il peut également spécifier des paramètres de déploiement. Les paramètres permettent de fournir des valeurs d’entrée lorsque vous utilisez le modèle.

Pour plus d’informations, consultez la page [Déploiement d’une application avec un modèle Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="prerequisites"></a>Prérequis

* Un **abonnement Azure**. Si vous n’en avez pas, essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree).

* Pour utiliser un modèle d’une interface CLI, vous devez avoir [Azure PowerShell](https://docs.microsoft.com/powershell/azure/?view=azps-1.2.0) ou [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="workspace-resource-manager-template"></a>Modèle Resource Manager de l’espace de travail

Le modèle Azure Resource Manager utilisé tout au long de ce document est disponible dans le répertoire [201-machine-learning-advanced](https://github.com/Azure/azure-quickstart-templates/blob/master/201-machine-learning-advanced/azuredeploy.json) du dépôt GitHub de modèles de démarrage rapide Azure.

Ce modèle crée les services Azure suivants :

* Compte Stockage Azure
* Azure Key Vault
* Azure Application Insights
* Azure Container Registry
* Espace de travail Azure Machine Learning

Le groupe de ressources est le conteneur des services. Les différents services sont requis par l’espace de travail Azure Machine Learning.

L’exemple de modèle comprend deux paramètres **obligatoires** :

* L’emplacement (**location**) de création des ressources.

    Le modèle utilise l’emplacement que vous sélectionnez pour la plupart des ressources. L’exception est le service Application Insights, qui n’est pas disponible dans tous les emplacements où les autres services le sont. Si vous sélectionnez un emplacement où il n’est pas disponible, le service est créé à l’emplacement USA Centre Sud.

* Le nom de l’espace de travail (**workspaceName**), qui est le nom convivial de l’espace de travail Azure Machine Learning.

    > [!NOTE]
    > Le nom de l’espace de travail n’est pas sensible à la casse.

    Le nom des autres services est généré de façon aléatoire.

> [!TIP]
> Le modèle associé à ce document crée un registre de conteneurs Azure, mais vous pouvez également créer un espace de travail sans créer de registre de conteneurs. Un registre de conteneurs est créé quand vous effectuez une opération qui le nécessite. C’est par exemple ce qui se passe lors de l’entraînement ou du déploiement d’un modèle.
>
> Vous pouvez également référencer un registre de conteneurs ou un compte de stockage existant dans le modèle Azure Resource Manager au lieu d’en créer un. Toutefois, le registre de conteneurs que vous utilisez doit avoir le __compte d’administrateur__ activé. Pour plus d’informations sur l’activation du compte d’administrateur, consultez [Compte d’administrateur](/azure/container-registry/container-registry-authentication#admin-account).

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

Pour plus d’informations sur les modèles, voir les articles suivants :

* [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Déployer une application avec des modèles Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md)
* [Types de ressources Microsoft.MachineLearningServices](https://docs.microsoft.com/azure/templates/microsoft.machinelearningservices/allversions)

## <a name="deploy-template"></a>Déployer un modèle

Pour déployer votre modèle, vous devez créer un groupe de ressources.

Consultez la section du [portail Azure](#use-the-azure-portal) si vous préférez utiliser l’interface graphique utilisateur.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az group create --name "examplegroup" --location "eastus"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroup -Name "examplegroup" -Location "eastus"
```

---

Une fois votre groupe de ressources correctement créé, déployez le modèle à l’aide de la commande suivante :

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" location="eastus"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus"
```

---

Par défaut, toutes les ressources créées dans le cadre du modèle sont nouvelles. Toutefois, vous avez également la possibilité d’utiliser des ressources existantes. En fournissant des paramètres supplémentaires au modèle, vous pouvez utiliser des ressources existantes. Par exemple, si vous voulez utiliser un compte de stockage existant, définissez la valeur **storageAccountOption** sur **existing**, puis indiquez le nom de votre compte de stockage dans le paramètre **storageAccountName**.

> [!IMPORTANT]
> Si vous souhaitez utiliser un compte de stockage Azure existant, il ne doit pas s’agir d’un compte Premium (Premium_LRS ou Premium_GRS). Il ne peut pas non plus comporter d’espace de noms hiérarchique (utilisé avec Azure Data Lake Storage Gen2). Ni le stockage Premium ni l’espace de noms hiérarchique ne sont pris en charge par le compte de stockage par défaut de l’espace de travail.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      storageAccountOption="existing" \
      storageAccountName="existingstorageaccountname"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -storageAccountOption "existing" `
  -storageAccountName "existingstorageaccountname"
```

---

## <a name="deploy-an-encrypted-workspace"></a>Déployer un espace de travail chiffré

L’exemple de modèle suivant montre comment créer un espace de travail avec trois paramètres :

* Activer les paramètres de confidentialité élevée pour l’espace de travail
* Activer le chiffrement pour l’espace de travail
* Utiliser une instance Azure Key Vault existante pour récupérer les clés gérées par le client

> [!IMPORTANT]
> Une fois l'espace de travail créé, vous ne pouvez pas modifier les paramètres des données confidentielles, du chiffrement, de l’ID du coffre de clés ou des identificateurs de clés. Pour modifier ces valeurs, vous devez créer un espace de travail à l’aide des nouvelles valeurs.

Pour plus d'informations, consultez [Chiffrement au repos](concept-enterprise-security.md#encryption-at-rest).

> [!IMPORTANT]
> Pour pouvoir utiliser ce modèle, l’abonnement doit satisfaire à certaines exigences :
>
> * L’application __Azure Machine Learning__ doit être un __contributeur__ pour votre abonnement Azure.
> * Vous devez disposer d’un Azure Key Vault contenant une clé de chiffrement.
> * Vous devez disposer dans l’Azure Key Vault d’une stratégie d’accès accordant les accès __get__, __wrap__ et __unwrap__ à l’application __Azure Cosmos DB__.
> * L’Azure Key Vault doit se trouver dans la même région que celle où vous envisagez de créer l’espace de travail Azure Machine Learning.

__Pour ajouter l’application Azure Machine Learning en tant que de contributeur__, utilisez les commandes suivantes :

1. Connectez-vous à votre compte Azure, puis obtenez votre ID d’abonnement. Cet abonnement doit être le même que celui qui contient votre espace de travail Azure Machine Learning.  

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az account list --query '[].[name,id]' --output tsv
    ```

    > [!TIP]
    > Pour sélectionner un autre abonnement, utilisez la commande `az account set -s <subscription name or ID>` et spécifiez le nom ou l’ID de l’abonnement auquel vous voulez passer. Pour plus d’informations sur la sélection d’abonnements, consultez [Utiliser plusieurs abonnements Azure](https://docs.microsoft.com/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest). 

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzSubscription
    ```

    > [!TIP]
    > Pour sélectionner un autre abonnement, utilisez la commande `Az-SetContext -SubscriptionId <subscription ID>` et spécifiez le nom ou l’ID de l’abonnement auquel vous voulez passer. Pour plus d’informations sur la sélection d’abonnements, consultez [Utiliser plusieurs abonnements Azure](https://docs.microsoft.com/powershell/azure/manage-subscriptions-azureps?view=azps-4.3.0).

    ---

1. Pour obtenir l’ID d’objet de l’application Azure Machine Learning, utilisez la commande suivante. La valeur peut être différente pour chacun de vos abonnements Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az ad sp list --display-name "Azure Machine Learning" --query '[].[appDisplayName,objectId]' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzADServicePrincipal --DisplayName "Azure Machine Learning" | select-object DisplayName, Id
    ```

    ---
    Cette commande retourne l’ID d’objet qui est un GUID.

1. Pour ajouter l’ID d’objet en tant que contributeur à votre abonnement, utilisez la commande suivante. Remplacez `<object-ID>` par l’ID objet du principal de service. Remplacez `<subscription-ID>` par le nom ou l’ID de votre abonnement Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az role assignment create --role 'Contributor' --assignee-object-id <object-ID> --subscription <subscription-ID>
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    New-AzRoleAssignment --ObjectId <object-ID> --RoleDefinitionName "Contributor" -Scope /subscriptions/<subscription-ID>
    ```

    ---

1. Pour générer une clé dans un coffre de clés Azure existant, utilisez l’une des commandes suivantes. Remplacez `<keyvault-name>` par le nom du coffre de clés. Remplacez `<key-name>` par le nom à utiliser pour la clé :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az keyvault key create --vault-name <keyvault-name> --name <key-name> --protection software
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Add-AzKeyVaultKey -VaultName <keyvault-name> -Name <key-name> -Destination 'Software'
    ```
    --- 

__Pour ajouter une stratégie d’accès au coffre de clés, utilisez les commandes suivantes__ :

1. Pour récupérer l’ID d’objet de l’application Azure Cosmos DB, utilisez la commande suivante. La valeur peut être différente pour chacun de vos abonnements Azure :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az ad sp list --display-name "Azure Cosmos DB" --query '[].[appDisplayName,objectId]' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzADServicePrincipal --DisplayName "Azure Cosmos DB" | select-object DisplayName, Id
    ```
    ---

    Cette commande retourne l’ID d’objet qui est un GUID. Enregistrez-le pour plus tard.

1. Pour définit la stratégie, utilisez la commande suivante. Remplacez `<keyvault-name>` par le nom de l’Azure Key Vault existant. Remplacez `<object-ID>` par le GUID de l’étape précédente :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az keyvault set-policy --name <keyvault-name> --object-id <object-ID> --key-permissions get unwrapKey wrapKey
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)
    
    ```azurepowershell
    Set-AzKeyVaultAccessPolicy -VaultName <keyvault-name> -ObjectId <object-ID> -PermissionsToKeys get, unwrapKey, wrapKey
    ```
    ---    

__Pour obtenir les valeurs__ pour `cmk_keyvault` (ID du Key Vault) et les paramètres `resource_cmk_uri` (URI de clé) dont ce modèle a besoin, procédez comme suit :

1. Pour connaître l’ID du Key Vault, utilisez la commande suivante :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az keyvault show --name <keyvault-name> --query 'id' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzureRMKeyVault -VaultName '<keyvault-name>'
    ```
    ---

    Cette commande renvoie une valeur semblable `/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>`.

1. Pour obtenir la valeur de l’URI de la clé gérée par le client, utilisez la commande suivante :

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az keyvault key show --vault-name <keyvault-name> --name <key-name> --query 'key.kid' --output tsv
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzureKeyVaultKey -VaultName '<keyvault-name>' -KeyName '<key-name>'
    ```
    ---

    Cette commande renvoie une valeur semblable `https://mykeyvault.vault.azure.net/keys/mykey/{guid}`.

> [!IMPORTANT]
> Une fois l'espace de travail créé, vous ne pouvez pas modifier les paramètres des données confidentielles, du chiffrement, de l’ID du coffre de clés ou des identificateurs de clés. Pour modifier ces valeurs, vous devez créer un espace de travail à l’aide des nouvelles valeurs.

Après avoir effectué avec succès les étapes ci-dessus, déployez votre modèle comme vous le feriez normalement. Pour activer l’utilisation des clés gérées par le client, définissez les paramètres suivants :

* **encryption_status** sur **Enabled**.
* **cmk_keyvault** sur la valeur `cmk_keyvault` obtenue dans les étapes précédentes.
* **resource_cmk_uri** sur la valeur `resource_cmk_uri` obtenue dans les étapes précédentes.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      encryption_status="Enabled" \
      cmk_keyvault="/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>" \
      resource_cmk_uri="https://mykeyvault.vault.azure.net/keys/mykey/{guid}" \
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -encryption_status "Enabled" `
  -cmk_keyvault "/subscriptions/{subscription-guid}/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<keyvault-name>" `
  -resource_cmk_uri "https://mykeyvault.vault.azure.net/keys/mykey/{guid}"
```
---

Lors de l’utilisation d’une clé gérée par le client, Azure Machine Learning crée un groupe de ressources secondaire qui contient l’instance Cosmos DB. Pour plus d’informations, consultez [Chiffrement au repos - Cosmos DB](concept-enterprise-security.md#encryption-at-rest).

Définir le paramètre **confidential_data** sur **true** constitue une configuration supplémentaire que vous pouvez fournir pour vos données. Cette opération effectue les actions suivantes :

* Démarre le chiffrement du disque de travail local pour les clusters de calcul Azure Machine Learning, à condition que vous n’ayez créé aucun cluster dans votre abonnement avant. Si vous avez déjà créé un cluster dans l’abonnement, ouvrez un ticket de support pour que le chiffrement du disque de travail soit activé pour vos clusters de calcul.
* Nettoie le disque de travail local entre les exécutions.
* Transmet de manière sécurisée les informations d’identification pour le compte de stockage, le registre de conteneurs et le compte SSH de la couche d’exécution à vos clusters de calcul en utilisant un coffre de clés.
* Active le filtrage IP pour garantir que les pools Batch sous-jacents ne peuvent pas être appelés par des services externes autres qu’AzureMachineLearningService.

    > [!IMPORTANT]
    > Une fois l'espace de travail créé, vous ne pouvez pas modifier les paramètres des données confidentielles, du chiffrement, de l’ID du coffre de clés ou des identificateurs de clés. Pour modifier ces valeurs, vous devez créer un espace de travail à l’aide des nouvelles valeurs.

  Pour plus d’informations, consultez [Chiffrement au repos](concept-enterprise-security.md#encryption-at-rest).

## <a name="deploy-workspace-behind-a-virtual-network"></a>Déployer un espace de travail derrière un réseau virtuel

En définissant la valeur du paramètre `vnetOption` sur `new` ou `existing`, vous pouvez créer les ressources utilisées par un espace de travail derrière un réseau virtuel.

> [!IMPORTANT]
> Pour le registre de conteneurs, seule la référence SKU « Premium » est prise en charge.

> [!IMPORTANT]
> Application Insights ne prend pas en charge le déploiement derrière un réseau virtuel.

### <a name="only-deploy-workspace-behind-private-endpoint"></a>Déployer l’espace de travail uniquement derrière un point de terminaison privé

Si vos ressources associées ne sont pas derrière un réseau virtuel, vous pouvez définir le paramètre **privateEndpointType** sur `AutoAproval` ou `ManualApproval` pour déployer l’espace de travail derrière un point de terminaison privé. Vous pouvez le faire aussi bien pour les espaces de travail existants que nouveaux. Lors de la mise à jour d’un espace de travail existant, renseignez les paramètres du modèle avec les informations concernant cet espace.

> [!IMPORTANT]
> Le déploiement n’est valide que dans les régions qui prennent en charge les points de terminaison privés.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -privateEndpointType "AutoApproval"
```

---

### <a name="use-a-new-virtual-network"></a>Utiliser un nouveau réseau virtuel

Pour déployer une ressource derrière un nouveau réseau virtuel, définissez **vnetOption** sur **new** avec les paramètres de réseau virtuel de la ressource concernée. Le déploiement ci-dessous montre comment déployer un espace de travail avec la ressource de compte de stockage derrière un nouveau réseau virtuel.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      storageAccountBehindVNet="true"
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -storageAccountBehindVNet "true"
  -privateEndpointType "AutoApproval"
```

---

Vous pouvez également déployer plusieurs ou toutes les ressources dépendantes derrière un réseau virtuel.

# <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      storageAccountBehindVNet="true" \
      keyVaultBehindVNet="true" \
      containerRegistryBehindVNet="true" \
      containerRegistryOption="new" \
      containerRegistrySku="Premium"
      privateEndpointType="AutoApproval"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -storageAccountBehindVNet "true"
  -keyVaultBehindVNet "true" `
  -containerRegistryBehindVNet "true" `
  -containerRegistryOption "new" `
  -containerRegistrySku "Premium"
  -privateEndpointType "AutoApproval"
```

---

<!-- Workspaces need a private endpoint when associated resources are behind a virtual network to work properly. To set up a private endpoint for the workspace with a new virtual network:

> [!IMPORTANT]
> The deployment is only valid in regions which support private endpoints.

# [Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="new" \
      vnetName="examplevnet" \
      privateEndpointType="AutoApproval"
```

# [Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "new" `
  -vnetName "examplevnet" `
  -privateEndpointType "AutoApproval"
```

--- -->

### <a name="use-an-existing-virtual-network--resources"></a>Utiliser un réseau virtuel et des ressources existants

Pour déployer un espace de travail avec les ressources associées existantes, vous devez définir le paramètre **vnetOption** sur **existing** avec les paramètres de sous-réseau. Toutefois, vous devez créer des points de terminaison de service dans le réseau virtuel pour chacune des ressources **avant** le déploiement. Comme avec les nouveaux déploiements de réseau virtuel, vous pouvez avoir l’une de vos ressources, ou toutes, derrière un réseau virtuel.

> [!IMPORTANT]
> Le sous-réseau doit avoir un point de terminaison de service `Microsoft.Storage`

> [!IMPORTANT]
> Les sous-réseaux n’autorisent pas la création de points de terminaison privés. Désactivez le point de terminaison privé pour activer le sous-réseau.

1. Activez les points de terminaison de service pour les ressources.

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.Storage"
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.KeyVault"
    az network vnet subnet update --resource-group "examplegroup" --vnet-name "examplevnet" --name "examplesubnet" --service-endpoints "Microsoft.ContainerRegistry"
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)

    ```azurepowershell
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.Storage" | Set-AzVirtualNetwork
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
    Get-AzVirtualNetwork -ResourceGroupName "examplegroup" -Name "examplevnet" | Set-AzVirtualNetworkSubnetConfig -Name "examplesubnet" -AddressPrefix "<subnet prefix>" -ServiceEndpoint "Microsoft.ContainerRegistry" | Set-AzVirtualNetwork
    ```

    ---

1. Déployer l’espace de travail

    # <a name="azure-cli"></a>[Azure CLI](#tab/azcli)

    ```azurecli
    az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="existing" \
      vnetName="examplevnet" \
      vnetResourceGroupName="examplegroup" \
      storageAccountBehindVNet="true" \
      keyVaultBehindVNet="true" \
      containerRegistryBehindVNet="true" \
      containerRegistryOption="new" \
      containerRegistrySku="Premium" \
      subnetName="examplesubnet" \
      subnetOption="existing"
      privateEndpointType="AutoApproval"
    ```

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azpowershell)
    ```azurepowershell
    New-AzResourceGroupDeployment `
      -Name "exampledeployment" `
      -ResourceGroupName "examplegroup" `
      -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
      -workspaceName "exampleworkspace" `
      -location "eastus" `
      -vnetOption "existing" `
      -vnetName "examplevnet" `
      -vnetResourceGroupName "examplegroup" `
      -storageAccountBehindVNet "true"
      -keyVaultBehindVNet "true" `
      -containerRegistryBehindVNet "true" `
      -containerRegistryOption "new" `
      -containerRegistrySku "Premium" `
      -subnetName "examplesubnet" `
      -subnetOption "existing"
      -privateEndpointType "AutoApproval"
    ```
    ---

<!-- Workspaces need a private endpoint when associated resources are behind a virtual network to work properly. To set up a private endpoint for the workspace with an existing virtual network:

> [!IMPORTANT]
> The deployment is only valid in regions which support private endpoints.

# [Azure CLI](#tab/azcli)

```azurecli
az deployment group create \
    --name "exampledeployment" \
    --resource-group "examplegroup" \
    --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" \
    --parameters workspaceName="exampleworkspace" \
      location="eastus" \
      vnetOption="existing" \
      vnetName="examplevnet" \
      vnetResourceGroupName="rg" \
      privateEndpointType="AutoApproval" \
      subnetName="subnet" \
      subnetOption="existing"
```

# [Azure PowerShell](#tab/azpowershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name "exampledeployment" `
  -ResourceGroupName "examplegroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-machine-learning-advanced/azuredeploy.json" `
  -workspaceName "exampleworkspace" `
  -location "eastus" `
  -vnetOption "existing" `
  -vnetName "examplevnet" `
  -vnetResourceGroupName "rg"
  -privateEndpointType "AutoApproval"
  -subnetName "subnet"
  -subnetOption "existing"
```

--- -->

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Suivez la procédure indiquée dans [Déployer des ressources à partir d’un modèle personnalisé](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-portal#deploy-resources-from-custom-template). Quand vous arrivez à l’écran __Sélectionnez un modèle__, choisissez le modèle **201-machine-learning-advanced** dans la liste déroulante.
1. Sélectionnez __Sélectionner un modèle__ pour utiliser le modèle. Fournissez les informations nécessaires suivantes et tous les autres paramètres en fonction de votre scénario de déploiement.

   * Abonnement : Sélectionnez l’abonnement Azure à utiliser pour ces ressources.
   * Groupe de ressources : Sélectionnez ou créez un groupe de ressources pour contenir les services.
   * Région : sélectionnez la région Azure dans laquelle les ressources seront créées.
   * Nom de l’espace de travail : Nom à utiliser pour l’espace de travail Azure Machine Learning qui va être créé. Le nom de l'espace de travail doit contenir entre 3 et 33 caractères. Il ne peut contenir que des caractères alphanumériques et « - ».
   * Localisation : Sélectionnez l’emplacement de création des ressources.
1. Sélectionnez __Revoir + créer__.
1. Dans l’écran __Vérifier + créer__, acceptez les conditions générales mentionnées, puis sélectionnez __Créer__.

Pour plus d’informations, consultez [Déployer des ressources à partir d’un modèle personnalisé](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

## <a name="troubleshooting"></a>Dépannage

### <a name="resource-provider-errors"></a>Erreurs du fournisseur de ressources

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="azure-key-vault-access-policy-and-azure-resource-manager-templates"></a>Stratégie d’accès Azure Key Vault et modèles Azure Resource Manager

Lorsque vous utilisez plusieurs fois un modèle Azure Resource Manager pour créer l’espace de travail et les ressources associées (y compris Azure Key Vault). Vous utilisez, par exemple, le modèle plusieurs fois avec les mêmes paramètres dans le cadre d’un pipeline d’intégration et de déploiement continus.

La plupart des opérations de création de ressources via des modèles sont idempotentes, mais Key Vault efface les stratégies d’accès chaque fois que le modèle est utilisé. L’effacement des stratégies accès empêche l’accès à Key Vault de n’importe quel espace de travail existant qui l’utilise. Par exemple, les fonctions Arrêter/Créer de machine virtuelle Azure Notebooks peuvent échouer.  

Pour éviter ce problème, nous vous recommandons une des approches suivantes :

* Ne déployez pas le modèle plus d’une fois pour les mêmes paramètres. Ou supprimez les ressources existantes avant d’utiliser le modèle pour les recréer.

* Examinez les stratégies d’accès Key Vault, puis utilisez ces stratégies pour définir la propriété `accessPolicies` du modèle. Pour voir les stratégies d’accès, utilisez la commande Azure CLI suivante :

    ```azurecli
    az keyvault show --name mykeyvault --resource-group myresourcegroup --query properties.accessPolicies
    ```

    Pour plus d’informations sur l’utilisation de la section `accessPolicies` du modèle, consultez les [informations de référence sur l’objet AccessPolicyEntry](https://docs.microsoft.com/azure/templates/Microsoft.KeyVault/2018-02-14/vaults#AccessPolicyEntry).

* Vérifiez si la ressource Key Vault existe déjà. Si c’est le cas, ne la recréez pas via le modèle. Par exemple, pour utiliser la ressource Key Vault existante au lieu d’en créer une nouvelle, apportez les modifications suivantes au modèle :

    * **Ajoutez** un paramètre qui accepte l’ID d’une ressource Key Vault existante :

        ```json
        "keyVaultId":{
          "type": "string",
          "metadata": {
            "description": "Specify the existing Key Vault ID."
          }
        }
      ```

    * **Supprimez** la section qui crée une ressource Key Vault :

        ```json
        {
          "type": "Microsoft.KeyVault/vaults",
          "apiVersion": "2018-02-14",
          "name": "[variables('keyVaultName')]",
          "location": "[parameters('location')]",
          "properties": {
            "tenantId": "[variables('tenantId')]",
            "sku": {
              "name": "standard",
              "family": "A"
            },
            "accessPolicies": [
            ]
          }
        },
        ```

    * **Supprimez** la ligne `"[resourceId('Microsoft.KeyVault/vaults', variables('keyVaultName'))]",` dans la section `dependsOn` de l’espace de travail. **Modifiez** également l’entrée `keyVault` dans la section `properties` de l’espace de travail pour référencer le paramètre `keyVaultId` :

        ```json
        {
          "type": "Microsoft.MachineLearningServices/workspaces",
          "apiVersion": "2019-11-01",
          "name": "[parameters('workspaceName')]",
          "location": "[parameters('location')]",
          "dependsOn": [
            "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
            "[resourceId('Microsoft.Insights/components', variables('applicationInsightsName'))]"
          ],
          "identity": {
            "type": "systemAssigned"
          },
          "sku": {
            "tier": "[parameters('sku')]",
            "name": "[parameters('sku')]"
          },
          "properties": {
            "friendlyName": "[parameters('workspaceName')]",
            "keyVault": "[parameters('keyVaultId')]",
            "applicationInsights": "[resourceId('Microsoft.Insights/components',variables('applicationInsightsName'))]",
            "storageAccount": "[resourceId('Microsoft.Storage/storageAccounts/',variables('storageAccountName'))]"
          }
        }
        ```

    Une fois ces changements effectués, vous pouvez spécifier l’ID de la ressource Key Vault existante quand vous exécutez le modèle. Le modèle réutilisera la ressource Key Vault en affectant son ID à la propriété `keyVault` de l’espace de travail.

    Pour obtenir l’ID de la ressource Key Vault, référencez la sortie de l’exécution du modèle initial ou utilisez Azure CLI. La commande suivante est un exemple d’utilisation d’Azure CLI pour obtenir l’ID de la ressource Key Vault :

    ```azurecli
    az keyvault show --name mykeyvault --resource-group myresourcegroup --query id
    ```

    Cette commande retourne une valeur semblable au texte suivant :

    ```text
    /subscriptions/{subscription-guid}/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Déployer des ressources à l’aide de modèles Resource Manager et de l’API REST Resource Manager](../azure-resource-manager/templates/deploy-rest.md).
* [Création et déploiement de groupes de ressources Azure à l’aide de Visual Studio](../azure-resource-manager/templates/create-visual-studio-deployment-project.md).
* [Pour obtenir d’autres modèles liés à Azure Machine Learning, consultez le dépôt de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates)
