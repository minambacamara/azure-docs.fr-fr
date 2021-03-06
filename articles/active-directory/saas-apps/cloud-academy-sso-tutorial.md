---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Cloud Academy - SSO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Cloud Academy - SSO.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: 560c86fb-e25c-4428-a1dd-a5711cfece5c
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 07/16/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: bb118e4d2fa811bf88ff13db84848ebb230ed209
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293404"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cloud-academy---sso"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Cloud Academy - SSO

Dans ce tutoriel, vous allez découvrir comment intégrer Cloud Academy - SSO à Azure Active Directory (Azure AD). Quand vous intégrez Cloud Academy - SSO à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Cloud Academy - SSO.
* Permettre à vos utilisateurs de se connecter automatiquement à Cloud Academy - SSO avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Cloud Academy - SSO pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Cloud Academy - SSO prend en charge l’authentification unique lancée par le **fournisseur de services**

* Après avoir configuré Cloud Academy - SSO, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-cloud-academy---sso-from-the-gallery"></a>Ajout de Cloud Academy - SSO à partir de la galerie

Pour configurer l’intégration de Cloud Academy - SSO à Azure AD, vous devez ajouter Cloud Academy - SSO à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Cloud Academy - SSO** dans la zone de recherche.
1. Sélectionnez **Cloud Academy - SSO** dans le panneau des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-cloud-academy---sso"></a>Configurer et tester l’authentification unique Azure AD pour Cloud Academy - SSO

Configurez et testez l’authentification unique Azure AD avec Cloud Academy - SSO à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Cloud Academy - SSO associé.

Pour configurer et tester l’authentification unique Azure AD avec Cloud Academy - SSO, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Cloud Academy - SSO](#configure-cloud-academy-sso-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Cloud Academy - SSO](#create-cloud-academy-sso-test-user)** pour avoir, dans Cloud Academy - SSO, un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Cloud Academy - SSO**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de connexion**, tapez l’URL : `https://cloudacademy.com/login/enterprise/`

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Cloud Academy - SSO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Cloud Academy - SSO**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-cloud-academy-sso-sso"></a>Configurer l’authentification unique Cloud Academy - SSO

1. Dans une autre fenêtre de navigateur, connectez-vous à votre site d’entreprise Cloud Academy - SSO en tant qu’administrateur.

1. Cliquez sur le nom de la société et sélectionnez **Settings & Integrations** (Paramètres et intégrations) dans le menu.

    ![Configuration ](./media/cloud-academy-sso-tutorial/config-1.PNG)

1. Dans la page **Settings & Integrations**, accédez à l’onglet **Integrations** et cliquez sur la carte **SSO**.

    ![Configuration ](./media/cloud-academy-sso-tutorial/config-2.PNG)

1. Effectuez les étapes ci-après dans la page suivante :

    ![Configuration ](./media/cloud-academy-sso-tutorial/config-3.PNG)

    a. Dans la zone de texte **Entity ID URL**, collez la valeur **ID d’entité** que vous avez copiée dans le portail Azure.

    b. Dans la zone de texte **URL SSO**, collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    c. Ouvrez le **Certificat (en base64)** téléchargé à partir du portail Azure dans le Bloc-notes, puis collez le contenu dans la zone de texte **Certificat**.

    d. Dans la zone de texte **Name ID Format**, la valeur par défaut est `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`.

1. Cliquez sur le bouton **Enregistrer**.

    > [!NOTE]
    > Pour plus d’informations sur la configuration de Cloud Academy - SSO, consultez cet [article de support](https://support.cloudacademy.com/hc/articles/360043908452-Setting-Up-Single-Sign-On).

### <a name="create-cloud-academy-sso-test-user"></a>Créer un utilisateur de test Cloud Academy - SSO

1. Connectez-vous à **Cloud Academy - SSO**.

1. Cliquez sur le nom de la société et sélectionnez **Members** dans le menu.

    ![ Créer un utilisateur de test ](./media/cloud-academy-sso-tutorial/create-user.PNG)

1. Cliquez sur **Invite Members** et sélectionnez **Invite a Single Member**.

    ![ Créer un utilisateur de test ](./media/cloud-academy-sso-tutorial/create-user-1.PNG)

1. Entrez les champs requis, puis cliquez sur **Invite**.

    ![ Créer un utilisateur de test ](./media/cloud-academy-sso-tutorial/create-user-2.PNG)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Cloud Academy - SSO dans le volet d’accès, vous devez être connecté automatiquement à l’application Cloud Academy - SSO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Essayer Cloud Academy - SSO avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Cloud Academy - SSO avec une visibilité et des contrôles avancés](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)