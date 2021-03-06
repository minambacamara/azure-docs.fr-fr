---
title: Guide pratique pour la gestion de l’agent Azure Monitor pour conteneurs | Microsoft Docs
description: Cet article décrit la gestion des tâches de maintenance les plus courantes avec l’agent en conteneur Log Analytics utilisé par Azure Monitor pour conteneurs.
ms.topic: conceptual
ms.date: 06/15/2020
ms.openlocfilehash: fc5bc0d60cb4ef1e375a997cbb3fe4bd2aed3235
ms.sourcegitcommit: d7008edadc9993df960817ad4c5521efa69ffa9f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86107408"
---
# <a name="how-to-manage-the-azure-monitor-for-containers-agent"></a>Guide pratique pour la gestion de l’agent Azure Monitor pour conteneurs

Azure Monitor pour conteneurs utilise une version en conteneur de l’agent Log Analytics pour Linux. Après le déploiement initial, vous devrez sans doute exécuter des tâches courantes ou facultatives pendant son cycle de vie. Cet article explique comment mettre à niveau l’agent manuellement et comment désactiver la collecte des variables d’environnement à partir d’un conteneur donné. 

## <a name="how-to-upgrade-the-azure-monitor-for-containers-agent"></a>Comment mettre à niveau l’agent Azure Monitor pour conteneurs

Azure Monitor pour conteneurs utilise une version en conteneur de l’agent Log Analytics pour Linux. Lorsqu’une nouvelle version de l’agent est disponible, celui-ci est automatiquement mis à niveau sur vos clusters Kubernetes managés sur Azure Kubernetes Service (AKS) et Azure Red Hat OpenShift version 3.x. Pour un [cluster Kubernetes hybride](container-insights-hybrid-setup.md) et Azure Red Hat OpenShift version 4.x, l’agent n’est pas managé et vous devez le mettre à niveau manuellement.

Si la mise à niveau de l’agent échoue pour un cluster hébergé sur AKS ou Azure Red Hat OpenShift version 3.x, cet article décrit également le processus permettant de mettre à niveau l’agent manuellement. Pour suivre les versions publiées, consultez [Annonces des versions de l’agent](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).

### <a name="upgrade-agent-on-aks-cluster"></a>Mettre à niveau l’agent sur un cluster AKS

Le processus de mise à niveau de l’agent sur des clusters AKS se compose de deux étapes simples. La première étape consiste à désactiver la surveillance avec Azure Monitor pour conteneurs à l’aide de l'interface de ligne de commande Azure. Suivez les étapes décrites dans l'article [Désactiver la surveillance](container-insights-optout.md?#azure-cli). L'interface de ligne de commande Azure permet de supprimer l’agent des nœuds du cluster, sans incidence sur la solution et les données correspondantes stockées dans l’espace de travail. 

>[!NOTE]
>Lors de cette activité de maintenance, les nœuds du cluster ne transfèrent pas les données collectées et les vues de performances n'affichent pas les données entre le moment où vous supprimez l'agent et celui où vous installez la nouvelle version. 
>

Afin d’installer la nouvelle version de l’agent, suivez les étapes décrites dans l’article [Activer la surveillance de l’intégration à l’aide de l’interface de ligne de commande Azure](container-insights-enable-new-cluster.md#enable-using-azure-cli) pour terminer ce processus.  

Après l’activation de la surveillance, un délai d’environ 15 minutes peut être nécessaire pour afficher les métriques d’intégrité mises à jour pour le cluster. Pour vérifier que l’agent a bien été mis à niveau, exécutez la commande : `kubectl logs omsagent-484hw --namespace=kube-system`

L’état doit ressembler à l’exemple suivant, où la valeur *omi* et *omsagent* doit correspondre à la dernière version spécifiée dans [l’historique des versions de l’agent](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).  

```console
User@aksuser:~$ kubectl logs omsagent-484hw --namespace=kube-system
:
:
instance of Container_HostInventory
{
    [Key] InstanceID=3a4407a5-d840-4c59-b2f0-8d42e07298c2
    Computer=aks-nodepool1-39773055-0
    DockerVersion=1.13.1
    OperatingSystem=Ubuntu 16.04.3 LTS
    Volume=local
    Network=bridge host macvlan null overlay
    NodeRole=Not Orchestrated
    OrchestratorType=Kubernetes
}
Primary Workspace: b438b4f6-912a-46d5-9cb1-b44069212abc
Status: Onboarded(OMSAgent Running)
omi 1.4.2.5
omsagent 1.6.0-163
docker-cimprov 1.0.0.31
```

### <a name="upgrade-agent-on-hybrid-kubernetes-cluster"></a>Mettre à niveau l’agent sur un cluster Kubernetes hybride

Procédez comme suit pour mettre à niveau l’agent sur un cluster Kubernetes s’exécutant sur :

* Des clusters Kubernetes automanagés hébergés sur Azure à l’aide du moteur AKS.
* Des clusters Kubernetes automanagés hébergés sur Azure Stack ou localement à l’aide du moteur AKS.
* Red Hat OpenShift version 4.x.

Si l’espace de travail Log Analytics se trouve dans un cloud commercial Azure, exécutez la commande suivante :

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<my_prod_cluster> incubator/azuremonitor-containers
```

Si l’espace de travail Log Analytics se trouve dans Azure China 21Vianet, exécutez la commande suivante :

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
```

Si l’espace de travail Log Analytics se trouve dans Azure US Government, exécutez la commande suivante :

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.domain=opinsights.azure.us,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
```

### <a name="upgrade-agent-on-azure-red-hat-openshift-v4"></a>Mettre à niveau l’agent sur Azure Red Hat OpenShift v4

Procédez comme suit pour mettre à niveau l’agent sur un cluster Kubernetes s’exécutant sur Azure Red Hat OpenShift version 4.x. 

>[!NOTE]
>Azure Red Hat OpenShift version 4.x prend uniquement en charge l’exécution dans le cloud commercial Azure.
>

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterId=<azureAroV4ResourceId> incubator/azuremonitor-containers
```

### <a name="upgrade-agent-on-azure-arc-enabled-kubernetes"></a>Mettre à niveau un agent sur Azure Arc pour Kubernetes

Exécutez la commande suivante pour mettre à niveau l’agent sur un cluster Kubernetes Azure Arc activé sans point de terminaison proxy.

```console
$ helm upgrade --install azmon-containers-release-1  –set omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterId=<resourceIdOfAzureArcK8sCluster>
```

Exécutez la commande suivante pour mettre à niveau l’agent lorsqu’un point de terminaison de proxy est spécifié. Pour plus d’informations sur le point de terminaison proxy, consultez [Configurer le point de terminaison proxy](container-insights-enable-arc-enabled-clusters.md#configure-proxy-endpoint) plus loin dans cet article.

```console
$ helm upgrade –name azmon-containers-release-1 –set omsagent.proxy=<proxyEndpoint>,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterId=<resourceIdOfAzureArcK8sCluster>
```

## <a name="how-to-disable-environment-variable-collection-on-a-container"></a>Comment désactiver la collecte des variables d’environnement sur un conteneur

Azure Monitor pour conteneurs collecte des variables d’environnement à partir des conteneurs en cours d’exécution dans un pod, puis les affiche dans le volet des propriétés du conteneur sélectionné dans la vue **Conteneurs**. Vous pouvez contrôler ce comportement en désactivant la collecte pour un conteneur spécifique, soit pendant le déploiement du cluster Kubernetes, soit après le déploiement en définissant la variable d’environnement *AZMON_COLLECT_ENV*. Cette fonctionnalité est disponible avec l’agent ciprod11292018 et versions ultérieures.  

Pour désactiver la collecte des variables d’environnement sur un nouveau conteneur ou sur un conteneur existant, définissez la variable **AZMON_COLLECT_ENV** avec la valeur **False** dans votre fichier de configuration yaml pour le déploiement de Kubernetes. 

```yaml
- name: AZMON_COLLECT_ENV  
  value: "False"  
```

Pour appliquer la modification aux clusters Kubernetes autres qu’Azure Red Hat OpenShift, exécutez cette commande : `kubectl apply -f  <path to yaml file>`. Pour modifier ConfigMap et appliquer cette modification aux clusters Azure Red Hat OpenShift, exécutez la commande suivante :

```bash
oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging
```

Votre éditeur de texte par défaut s’ouvre. Définissez la variable, puis enregistrez le fichier dans l’éditeur.

Pour vérifier que le changement de configuration est effectif, sélectionnez un conteneur dans la vue **Conteneurs** d’Azure Monitor pour conteneurs puis, dans le volet des propriétés, développez **Variables d’environnement**.  La section doit afficher uniquement la variable créée précédemment - **AZMON_COLLECT_ENV = FALSE**. Pour tous les autres conteneurs, la section des variables d’environnement doit répertorier toutes les variables d’environnement détectées.

Pour réactiver la découverte des variables d’environnement, suivez le même processus et remplacez la valeur **False** par **True**, avant de réexécuter la commande `kubectl` pour mettre à jour le conteneur.  

```yaml
- name: AZMON_COLLECT_ENV  
  value: "True"  
```  

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes lors de la mise à niveau de l'agent, consultez le [guide de résolution des problèmes](container-insights-troubleshoot.md).
