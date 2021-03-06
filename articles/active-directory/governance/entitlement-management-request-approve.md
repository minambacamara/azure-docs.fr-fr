---
title: Approuver ou refuser des demandes d'accès - Gestion des droits d'utilisation Azure AD
description: Découvrez comment utiliser le portail Mon Accès pour approuver ou refuser les demandes d’accès à un package d’accès dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: msaburnley
manager: daveba
editor: mamtakumar
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mamkumar
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78c3c177bfcd5ee969e1430306c7294f0a14b658
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "85078092"
---
# <a name="approve-or-deny-access-requests-in-azure-ad-entitlement-management"></a>Approuver ou refuser des demandes d’accès dans la gestion des droits d’utilisation Azure AD

Avec la gestion des droits d’utilisation Azure AD, vous pouvez configurer des stratégies pour exiger l’approbation des packages d’accès et choisir un ou plusieurs approbateurs. Cet article décrit comment les approbateurs désignés peuvent approuver ou refuser les demandes pour les packages d’accès.

## <a name="open-request"></a>Ouvrir la demande

La première étape pour approuver ou refuser les demandes d’accès consiste à rechercher et ouvrir la demande d’accès en attente d’approbation. Il existe deux manières d’ouvrir la demande d’accès.

**Rôle prérequis :** Approbateur

1. Recherchez un e-mail de Microsoft Azure vous demandant d’approuver ou de refuser une demande. Voici un exemple d’e-mail :

    ![E-mail concernant l’approbation de la demande pour un package d’accès](./media/entitlement-management-shared/approver-request-email.png)

1. Cliquez sur le lien **Approuver ou refuser la demande** pour ouvrir la demande d’accès.

1. Connectez-vous au portail Mon Accès.

Si vous n’avez pas reçu l’e-mail, vous pouvez localiser les demandes d’accès en attente d’approbation comme suit.

1. Connectez-vous au portail Mon Accès sur [https://myaccess.microsoft.com](https://myaccess.microsoft.com).  (Pour le gouvernement des États-Unis, le domaine figurant dans le lien du portail Mon Accès sera `myaccess.microsoft.us`.)

1. Dans le menu de gauche, cliquez sur **Approbations** pour afficher la liste des demandes d’accès en attente d’approbation.

1. Dans l’onglet **En attente**, recherchez la demande.

## <a name="approve-or-deny-request"></a>Approuver ou refuser la demande

Une fois que vous avez ouvert une demande d’accès en attente d’approbation, vous pouvez consulter les détails qui vous aideront à prendre la décision.

**Rôle prérequis :** Approbateur

1. Cliquez sur le lien **Afficher** pour ouvrir le volet Demande d’accès.

1. Cliquez sur **Détails** pour afficher les détails de la demande d’accès.

    Les détails incluent le nom d’utilisateur, l’organisation, la date de début et de fin le cas échéant, la justification professionnelle, la date de soumission de la demande et la date d’expiration de la demande.

1. Cliquez sur **Approuver** ou **Refuser**.

1. Si nécessaire, indiquez un motif.

    ![Portail Mon Accès - Demande d’accès](./media/entitlement-management-request-approve/my-access-approve-request.png)

1. Cliquez sur **Soumettre** pour soumettre votre décision.

    Si une stratégie est configurée avec plusieurs approbateurs, un seul approbateur doit prendre une décision concernant l’approbation en attente. Une fois qu’un approbateur a rendu sa décision concernant la demande d’accès, celle-ci est terminée et n’est plus disponible pour les autres approbateurs. Les autres approbateurs peuvent voir la décision et l’approbateur qui est intervenu dans leur portail Mon Accès. À ce stade, seule l’approbation en une étape est prise en charge.

    Si aucun des approbateurs configurés ne peut approuver ou refuser la demande d’accès, celle-ci expire après la période définie. L’utilisateur est averti que sa demande d’accès a expiré et qu’il doit la soumettre à nouveau.

## <a name="next-steps"></a>Étapes suivantes

- [Demander l’accès à un package d’accès](entitlement-management-request-access.md)
- [Processus de requête et notifications par e-mail](entitlement-management-process.md)
