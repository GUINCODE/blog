---
title: "ISPM & ITDR OKTA - Kezako 🤔?"
seoTitle: "ISPM ITDR OKTA"
seoDescription: "Identity Security and Posture Management
Identity Threat Detection and Response"
datePublished: Wed Feb 19 2025 12:08:29 GMT+0000 (Coordinated Universal Time)
cuid: cm7bvdfoc000809lac8ao2j4g
slug: ispm-and-itdr-okta-kezako
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1739966161769/19f19a5d-0620-4063-b0ab-1b40cf02c696.png
tags: identity-management

---

## Introduction

Malgré l’adoption quasi-généralisée de modèles de sécurité « **Principe du moindre privilège** » et/ou « **ZeroTrust** », plus de 60% des cyberattaques en 2024 étaient encore liées à des usurpations d’identité et/ou à des droits d’accès excessifs. Les solutions IDP (IDentity Provider), telles Okta ou EntraID, ont vocation à réduire considérablement ce type de risques par la mise en place de politiques d’authentification et d’accès centralisées et sécurisées (SSO, MFA contextuel, gouvernance des identités…).

En théorie, avec ce type de solutions, on peut dormir tranquille, surtout si l’on utilise des facteurs dits « phishing résistants » ...**Sauf que, ce n’est pas nécessairement aussi simple !**

Si le fait de mettre en place un IDP moderne réduit considérablement la surface d’attaque, il reste encore quelques angles morts ! En effet, les IDP modernes sont très efficaces pour gérer l’authentification, les politiques de droits d’accès ou encore le cycle de vie des identités, mais jusqu’à présent, ils répondaient de manière encore trop imparfaite à deux enjeux importants :

* **La gestion de la posture de sécurité des identités**
    
* **La détection et la réponse aux menaces liées aux tentatives d’accès**
    

Fort de ce constat, Okta a récemment lancé 2 nouveaux modules intégrés à sa solution Okta Identity Platform, qui apportent des réponses très concrètes : **ISPM et ITDR**.

L’objet de cet article est de mieux comprendre à quoi servent les modules **ISPM (Identity Security Posture Management)** et **ITDR (Identity Threat Detection & Response)**.

## I. **Gestion de la Posture de Sécurité des Identités (ISPM)**

Commençons par une définition claire : l'ISPM sert à identifier et corriger, en amont de toute tentative d’accès, les vulnérabilités potentielles sur les identités.

Comme sa définition l’indique, son rôle est donc proactif (en amont). **On n’attend pas que quelqu’un essaie de se connecter pour identifier et corriger les vulnérabilités potentielles**.

Donnons quelques exemples de vulnérabilités sur les identités :

* Attribution excessive de droits/privilèges
    
* Mots de passe faibles ou inchangés
    
* Incohérence dans les parcours d’authentification (notamment les conditions de déclenchement du MFA ou encore les facteurs MFA eux-mêmes)
    
* Utilisation incorrecte des comptes de service (utilisation des comptes de service comme comptes humains et réciproquement)
    
* Gestion incomplète du cycle de vie des identités (par exemple, lors d’une suppression de compte, Okta est mis à jour mais pas l’AD ou réciproquement)
    
* Utilisation de comptes partagés
    

Ces exemples permettent de bien comprendre le rôle du module ISPM d’Okta : offrir une vue centralisée et complète de toutes les vulnérabilités potentielles à travers l’ensemble de l’écosystème lié à l’identité, afin de les corriger en amont, avant toute tentative d’authentification par un utilisateur !

Un de nos clients a résumé l’intérêt d’ISPM de la façon suivante, et je trouve sa formulation assez juste : **« c’est un peu comme une sentinelle qui vérifierait en continue si l’on a bien fait les choses dans notre gestion des identités, et si ce n’est pas le cas, il nous invite à les corriger ».**

## II. **Détection et Réponse aux Menaces sur les Identités (ITDR)**

Définition : l'ITDR d'Okta se concentre sur la surveillance **continue** et **la réponse** aux menaces ciblant les identités.

En première lecture, cela ressemble à ce que fait l’ISPM, à l’exception près qu’il s’agît d’opérations réalisées après l’authentification. Il n’est plus question de qualifier et améliorer la posture de sécurité des identités mais de réagir et remédier en temps réel à une possible compromission.

**Exemples concrets :**

* Si un attaquant tente d’accéder aux comptes d’utilisateurs par force brute (il essaie des milliers de paires de login/password différents), l’adresse IP de cet attaquant sera immédiatement bloquée.
    
* Une fuite de données sur le darkweb révèle les identifiants et mots de passe d’utilisateurs. Les comptes seront alors mis en quarantaine (« suspendus ») en attendant que les utilisateurs concernés réinitialisent leur mot de passe via une procédure sécurisée n’utilisant pas leurs credentials actuels
    
* Détection d’un comportement anormal : un utilisateur se connecte depuis un lieu et à une heure très inhabituels. ITDR va alors identifier une activité suspecte et un deuxième facteur (par exemple) sera déclenché
    
* Détection et réaction automatique aux attaques de Phishing (Hameçonnage). Si un employé clique par mégarde sur un lien malveillant et fournit ses identifiants, ITDR va alors identifier qu’un utilisateur utilisant les mêmes credentials essaie de se connecter depuis un lieux différent ou encore une machine différente. La réponse configurable d’ITDR peut alors être : autre facteur d’authentification requis (car finalement un utilisateur légitime peut aussi prendre l’avion et utiliser un autre ordinateur) ou encore impossibilité de se connecter depuis un autre terminal.
    
* Universal Logout : Mécanisme qui permet de fermer toutes les sessions actives de l’utilisateur. Pas uniquement la session Okta, mais aussi (et peut-être surtout) les sessions des applications et services fédérés par Okta. Ainsi, si un attaquant récupère un token de session Okta (stocké sous forme d’un cookie sécurisé) et essaie de le jouer depuis un autre poste, ITDR le détectera et déclenchera l’universal logout sur toutes les sessions Okta actives et sur l’ensemble des sessions des applications fédérées.
    

Les exemples donnés visent à mieux comprendre les finalités respectives d’ISPM et ITDR, le premier permettant une surveillance continue du respect des bonnes pratiques de sécurité des identités en amont (la posture), le deuxième étant une surveillance et une réponse en continue aux tentatives réelles d’authentification.

Ci-après un schéma récapitulatif simplifié:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739965426905/f73668e4-4917-4a37-893d-65129f55583d.png align="center")

## **CONCLUSION**

Avec l'adoption croissante des environnements multiclouds et SaaS, les équipes de sécurité rencontrent des défis pour maintenir la visibilité et le contrôle sur leur paysage d'identités. Les solutions ISPM et ITDR ont pour objet d’une part de diminuer la surface d'attaque en détectant les vulnérabilités potentielles liées aux identités, et d’autre part détecter les menaces effectives pour une remédiation rapide. En combinant ces deux approches, les organisations peuvent renforcer leur posture de sécurité globale et se protéger plus efficacement encore contre les menaces basées sur les identités.

De manière plus générale, Okta prenant une place de plus en plus centrale dans le SI de nombreuses organisations, l’idée d’aider ses clients à Monitorer, Surveiller, Piloter leurs identités et plus généralement réduire toute exposition au risque d’accès frauduleux est clairement une direction attendue par le marché.

Il y avait déjà « Okta ThreatInsight », et dorénavant, les clients d’Okta disposent de deux armes supplémentaires : **ISPM** et **ITDR**.