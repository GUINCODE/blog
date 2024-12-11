---
title: "Avec Auth0 (Okta Customer Identity Cloud), passez à l'Action !"
seoTitle: "auth0-okta-customer-identity-cloud"
datePublished: Mon Oct 23 2023 12:46:12 GMT+0000 (Coordinated Universal Time)
cuid: clo2w5sg2001s09l89strd9rv
slug: avec-auth0-okta-customer-identity-cloud-passez-a-laction
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697807444543/ab818b52-895a-46b2-8916-970a9a9ca56a.png
tags: auth0, okta, ciam, auth0-actions

---

Découvrons ensemble les Actions, ces morceaux de code qui agissent comme des extensions flexibles du flux d'autorisation et d'authentification !

## Avant toute chose, Auth0, c'est quoi ?

[Auth0](https://auth0.com/) est une solution IaaS (Identity as a Service), pensée par et pour des développeurs. Acquise en 2021 par [Okta](https://www.okta.com/fr/), Auth0 devient la solution destinée au CIAM (Customer Identity Access Management), tandis que la plateforme Okta est axée WIAM (Workforce Identity Access Management).

![La page d'accueil du site d'Auth0](https://cdn.hashnode.com/res/hashnode/image/upload/v1697794389597/26fba0a3-218e-40f6-8cf9-2af8573aac15.png align="center")

Vous avez donc accès à une gestion centralisée de vos applications, serveurs d'autorisations (qui figurent sous le terme d'APIs dans un tenant Auth0), et des canaux d'authentification (copyright le boss de Point Base, Christian Fourneau, j'aime beaucoup cette terminologie qu'il emploie pour désigner les "connections" dans le tenant Auth0).

Implémenter l'authentification et l'autorisation dans une application avec Auth0 vous prendra environ 5 minutes, et quelques lignes de code. Il faut dire que [plus de 30 SDK (Software Development Kit)](https://auth0.com/docs/libraries) sont à disposition pour vous permettre d'ajouter Auth0 dans votre application, quelle que soit son langage (JS, PHP, Java, Python...) ou le framework utilisé (Vue.JS, React, Angular comme exemples pour du front, Node, Go, Laravel pour du back). Les développements IOS et Native sont aussi couverts.

En bref, Auth0, c'est la solution CIAM / IaaS qui vous facilite la vie, mais surtout vous apporte les moyens techniques de sécuriser l'accès à vos ressources via les protocoles d'autorisation robustes tels que OAuth 2.1 , OIDC (OpenID COnnect) ou SAML (Security Assertion Markup Language), ou l'ajout d'Identity Provider de type social ( Facebook, LinkedIn, Twitter...). Et je n'évoque ici qu'une petite portion des possibilités existantes de canaux d'authentification.

![Liste des Social Login possible](https://cdn.hashnode.com/res/hashnode/image/upload/v1697794272995/30212e2e-6f7c-4a4a-ac84-04e5dd47a401.png align="center")

## Ok. Et les Actions là-dedans ?

Si vous êtes ici, c'est que votre curiosité a dû être piquée. Tant mieux, car vous n'allez pas être déçus. Il y a deux types d'Actions : celles provenant du Marketplace et les custom Actions.

### Les custom Actions : codez votre logique !

Les custom Actions sont des morceaux de code écrits en Node.JS, stockés dans une librairie sur le tenant Auth0, et qui vont être injectés dans un des flux Auth0 : authentification, machine-to-machine, pré ou post-enregistrement d'un utilisateur, après changement de mot de passe, ou envoi de SMS dans le cadre MFA (Multi-Factor Authentication).

![La liste des déclencheurs](https://cdn.hashnode.com/res/hashnode/image/upload/v1697794177034/3e53f84b-3707-48da-b116-9ea7cbff7c2e.png align="center")

L'objectif de ces Actions ? Vous permettre d'ajouter de la logique personnalisée qui répond à vos besoins, probablement commune à plusieurs de vos applications, sans avoir à écrire dans chaque application le même code.

### Quelques exemples ne seraient pas de refus...

"Lecteurs, lectrices, je vous ai compris".

Imaginons que vous avez deux applications qui peuvent être amenées à dialoguer entre elle et échanger des données : vous allez avoir très probablement besoin que certaines informations figurent dans des custom claims de votre ID token ou de votre Access Token. Plutôt que d'écrire du code dans chaque application, Auth0 vous permet au moyen d'une Action d'enrichir le token ciblé d'un custom claim dont vous définissez le nom et la valeur. Vous voulez du concret ? En voilà :

![Une custom Action](https://cdn.hashnode.com/res/hashnode/image/upload/v1697797731902/d90ffa8a-52a7-4f3a-b1c1-469ca02796f5.png align="center")

Ici, quant une autorisation a lieu, l'action va rajouter quatre informations [au moyen de l'objet API](https://auth0.com/docs/customize/actions/flows-and-triggers/login-flow/api-object) :

* un claim "email" dans l'Access token (ligne 9) qui se base sur [le contexte de l'utilisateur s'authentifiant](https://auth0.com/docs/customize/actions/flows-and-triggers/login-flow/event-object)
    
* un claim "emailVerified" dans l'Access token (ligne 10), qui fait de mêm pour récupérer la valeur dans le profil Auth0 de l'attribut "email\_verified"
    
* un claim "secretIdentity" dans l'ID token avec la valeur statique "Batman" (ligne 11)
    
* et pour finir un claim "userAuth0Identities" aussi dans l'ID token, qui renvoie un tableau des différents Identity providers recensés pour l'utilisateur sur Auth0
    
    L'étape suivante ? Cliquer sur "Deploy" puis surtout, ajouter dans le flow Login cette action pour qu'elle s'exécute dès que l'utilisateur se connecte, et avant que le Token soit fourni à l'application. Enfin, cliquez sur "Apply".
    
    ![Ajout de l'Action dans le flow Login](https://cdn.hashnode.com/res/hashnode/image/upload/v1697798335026/36492e10-827b-48f0-a3a2-5909edb6e8a5.png align="center")
    

Et voilà le résultat :

![L'ID token enrichi](https://cdn.hashnode.com/res/hashnode/image/upload/v1697799624251/03599b68-6405-44de-a55b-be4f0591a825.png align="center")

![L'Access token](https://cdn.hashnode.com/res/hashnode/image/upload/v1697799674501/7937d302-f831-4e99-91fd-902724cde39a.png align="center")

A noter que vous pouvez chaîner des Actions les unes derrière les autres dans les flows.

![Enchaînement d'Actions](https://cdn.hashnode.com/res/hashnode/image/upload/v1697800046795/a80aacc5-cbe2-4e57-b26b-791ba044b119.png align="center")

Ici par exemple, une première Action Guardian MFA vérifie si l'utilisateur qui tente de se connecter à l'application est en Europe. Si c'est le cas, il lui est imposé d'utiliser l'application de MFA Guardian pour founir un code OTP ou valider une notification push. Ensuite, le token est enrichi avant d'être envoyé à l'application.

Vous voulez un autre exemple de custom Action ? Prenons un flow "Post User registration". Pour un cas où vous voudriez que dès qu'un nouvel utilisateur est enregistré sur la connexion nommée "customers", une nouvelle ligne est ajoutée à un google spreadsheet. Vous codez votre condition, et passez un appel via la librairie axios dans votre action, et le tour est joué :

![Exemple simplifié d'appel axios à la google spreadsheet api](https://cdn.hashnode.com/res/hashnode/image/upload/v1697804894355/994b0600-074d-4e24-b52a-c2ba24f972e6.png align="center")

Ça en fait des champs des possibles, pas vrai ?

### Et le deuxième type d'Actions ?

Si vous n'avez pas envie ou besoin de coder votre propre Action, vous pouvez également consulter le marketplace d'Auth0, qui propose un certain nombre d'Actions disponibles, déjà construites, et qui n'ont besoin que d'une configuration locale dans votre tenant pour s'appliquer.

![Liste non exhaustive d' Actions disponibles](https://cdn.hashnode.com/res/hashnode/image/upload/v1697805448383/63b1c652-e7de-4192-92d1-271828fcc81a.png align="center")

Au passage, j'attire votre attention sur une Action extrêmement intéressante, et qui peut compléter votre gestion CIAM par Auth0 : la possibilité de recourir à l'utilisation de workflows Okta si vous avez un tenant WIC (Workforce Identity Cloud). Vous pouvez alors faire appel à un workflow dont le déclencheur sera une carte API Endpoint, dont vous récupérez l'invoke URL et le client Token.

![L'Action Okta Workflows](https://cdn.hashnode.com/res/hashnode/image/upload/v1697805853151/f712d3b9-9871-4df0-b6ad-163ee7e62316.png align="center")

Vous pourrez passer en données envoyées au workflow certaines propriétés relevant de l'objet event :

![Les données disponibles à l'export vers le workflow](https://cdn.hashnode.com/res/hashnode/image/upload/v1697805979144/9e9b09e9-f0c7-4361-9e4d-160b7e4dc969.png align="center")

Si ce n'est pas une preuve supplémentaire de la complémentarité entre Okta et Auth0, je ne sais pas ce qu'il vous faut de plus !

Enfin, les Actions s'exécutent toutes dans l'ordre que vous leur donnerez, peu importe qu'elles soient custom ou des Actions installées depuis le Marketplace. N'hésitez donc pas à conjuguer ces deux types d'Actions pour atteindre l'objectif de customisation d'Auth0 que vous visez, en cohérence avec vos besoins.

## Ce qu'il faut retenir

* Les Actions d'Auth0 sont des morceaux de code soit que vous écrivez vous-même, soit que vous importez depuis le Marketplace Auth0
    
* Elles s'exécutent selon l'ordre que vous configurez au sein d'un des flows disponibles
    
* Dans une custom Action, vous pouvez faire appel à un package npm, définir des secrets. Vous pouvez même tester une exécution
    
* Chaque flow donne accès à des propriétés relevant des objets event et/ou api
    
* Le nombre d'Actions dépend de votre plan (Free: 3, Essential: 5, Professional: 7, Enterprise: illimité)
    
* Les Actions sont destinées à vous permettre d'apporter une logique qui est propre à vos besoins, à l'intérieur de la gestion CIAM par Auth0
    
* Les Actions, c'est top !
    

Bien entendu, je n'ai fait ici que vous présenter un tour d'horizon assez simple de toutes les possibilités qu'offrent les Actions. Pour plus de détails de configuration, et des exemples de use-cases fréquents de recours aux Actions, je vous encourage à consulter la documentation [ici](https://auth0.com/docs/customize/actions).

Des questions après cette lecture ? N'hésitez surtout pas à nous contacter !