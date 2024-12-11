---
title: "Introduction à OKTA Workflows : rien n’est impossible (ou presque) !"
seoTitle: "workflow101"
seoDescription: "Introduction à OKTA Workflows : rien n’est impossible (ou presque) !"
datePublished: Fri Sep 29 2023 12:23:04 GMT+0000 (Coordinated Universal Time)
cuid: cln4krlij000w08l2fmr1edug
slug: introduction-a-okta-workflows-rien-nest-impossible-ou-presque
canonical: https://medium.com/@julien.nanquette/introduction-%C3%A0-okta-workflows-rien-nest-impossible-ou-presque-56e8dda965cf
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695989981774/b1061428-1dc0-49ca-9bb7-6d11bd95a122.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1695990132005/9ac37e06-d81b-4b75-a838-8a5f472c263b.png
tags: workflow, okta, iam

---

Si vous êtes ici, c’est qu’a priori vous savez ce qu’est Okta. Mais si jamais vous ne le savez pas, personne ne vous en voudra (promis !). Okta propose une solution IAM (Identity Access Management), qui vous permet de gérer de façon centralisée et sécurisée les identités et accès de vos utilisateurs aux ressources auxquelles ils doivent pouvoir accéder.

Okta, c’est l’armoire à glace du carré VIP qui va vérifier que vous êtes bien sur la liste avant de vous laisser entrer, sauf qu’on pourra vous demander plus de preuves d’identité qu’un simple mot de passe. Et si ce que vous présentez n’est pas reconnu, vous aurez droit à une réponse à la Gandalf.

![](https://miro.medium.com/v2/resize:fit:593/1*XU_BR7uP-558PsMXj1p_lQ.jpeg align="center")

Vous ne passerez pas !

Plus d’informations par ici : [Workforce Identity Cloud | Okta](https://www.okta.com/fr/workforce-identity/)

Aujourd’hui, je vais vous parler d’une des fonctionnalités de la solution Workforce Identity Cloud d’Okta, et non des moindres, les workflows.

# Les workflows ? Quézaco ?

Les workflows d’Okta sont une fonctionnalité low-code qui vous permet d’automatiser et d’enchaîner des actions, qui peuvent toucher des cibles différentes (utilisateurs, groupes, applications externes ou intégrées à votre organisation Okta, etc…).

Pour exécuter des workflows, vous aurez besoins d’une ou plusieurs “connections”.

# Les “connections”

Ce sont des connexions aux applications cibles du workflows, qui vont gérer les flux d’autorisations pour garantir la bonne exécution des requêtes.

Pour agir sur un tenant Okta (pas forcément le vôtre !), vous devrez saisir les client id et client secret du tenant cible, ce qui fera utiliser le flow client credentials pour les actions.

Pour d’autres applications, vous pourrez utiliser des flux OAuth 2.0, ou Basic, ou même Custom.

Un workflow démarre toujours par un évènement déclencheur, qui peut être :

1° Interne à votre organisation (tenant) — quelques exemples :

* l’ajout d’un utilisateur à un groupe
    
* l’assignation d’une application à un utilisateur ou un groupe
    
* la mise à jour d’une valeur d’un attribut du profil d’un utilisateur
    
* un déclenchement programmé tous les jours à une heure précise
    

2° Un évènement dans une application tierce — quelques exemples :

* un mail reçu dans Gmail
    
* une requête créé dans Jira
    
* un nouveau canal créé dans Slack
    
* une nouvelle entrée dans Salesforce
    

Les évènements déclencheurs possibles sont nombreux, et je n’ai même pas encore parlé de déclenchement par web hook ou appel d’endpoint API.

Une fois le déclencheur configuré, vous allez pouvoir enchaîner des actions, chacune représentée par une carte, qui pourra, pour certaines, en retour renvoyer des données (profil d’un utilisateur par exemple), et qui représente les appels API qui seront réalisés.

Les données récupérées peuvent être alors liées comme des variables qui seront utilisées sur d’autres cartes, l’exemple le plus courant est l’ID d’un user.

# Un exemple très simpliste

Dans la console admin Okta, à chaque ajout d’un utilisateur dans un groupe, on veut que la valeur de l’attribut “isMemberOfGroup” du profil de l’utilisateur passe à “true”.

![](https://miro.medium.com/v2/resize:fit:875/1*aQMOOY89-R1Jeq-IEzEFHw.png align="center")

Le workflow en question: un déclencheur, deux actions

Le déclencheur est la carte “User Added to Group”, les deux cartes suivantes sont les actions qui suivent : on lit le profil de l’utilisateur en se basant sur son ID (étape facultative, mise ici pour illustrer l’enchaînement), puis on fait un update partiel du profil de l’utilisateur, en passant la valeur de l’attribut “isMemberOfGroup” à “True”.

![](https://miro.medium.com/v2/resize:fit:875/1*SFiixMr0w8tbFFi4_DHolQ.png align="left")

L’attribut ‘IsMemberOfGroup” passe de “null” dans la carte de lecture du profil à “True” dans la carte d’update juste après

# Les applications connectées

Dans le même principe de l’Okta Integration Network (OIN), le catalogue des applications tierces pour lesquels des connecteurs à Okta existent déjà et sont disponibles, certaines applications ont des connecteurs dans les workflows.

![](https://miro.medium.com/v2/resize:fit:770/1*oYFY6rU9jVyZ1cqpcSKgSA.png align="center")

Extrait de la liste des connecteurs d’applications disponibles

Des cartes d’actions vous permettent facilement de programmer la tâche à réaliser dans ce cas : par exemple, avec Gmail, envoyer un mail dont vous définirez le contenu, le(s) destinataire(s), ou encore, ajouter un utilisateur dans une organisation GitHub.

![](https://miro.medium.com/v2/resize:fit:765/1*EMISDyUfe5CaDQ2aEqeQEg.png align="center")

Exemples d’actions disponibles pour le connecteur Google Workspace

“Et si l’application sur laquelle on veut agir n’est pas déjà référencée, on fait quoi ?” me direz-vous. Et bien, il vous reste toujours la possibilité d’effectuer des opérations via des appels API vers votre cible, les cartes “API Connector”.

Un exemple concret réalisé récemment pour un client : à chaque onboarding d’un nouveau collaborateur, on va aller stocker un mot de passe temporaire dans le coffre-fort 1Password du manager de ce nouvel employé. 1Password n’ayant pas de connecteur établi, on va alors passer un appel d’API en POST vers le coffre-fort du Manager et lui envoyer un objet JSON contenant le mot de passe.

![](https://miro.medium.com/v2/resize:fit:844/1*BcOzx_6nLyKAZQlmAMajyA.png align="center")

Et hop, un mot de passe aléatoire va dans un coffre-fort 1Password.

# L’historique des exécutions

Pour finir cette introduction aux workflows, il me faut vous parler de l’historique, que vous pouvez choisir de sauvegarder ou non.

Il est en tout cas très utile de conserver ces données, notamment lors de la phase de construction d’un workflow, car chaque exécution vous apportera carte par carte les données résultantes : en cas d’erreur, vous savez précisément où et pourquoi une action n’a pas marché comme prévu.

![](https://miro.medium.com/v2/resize:fit:875/1*mCq-NEARISkJhq6Q37BjTQ.png align="center")

Une exécution où tout a fonctionné

# Ce qu’il faut retenir

* Les workflows permettent d’automatiser des enchaînements d’actions via des appels API sur votre tenant Okta ou sur des applications tierces
    
* Le déclenchement peut être programmé à une fréquence donnée, ou répondre à un évènement précis, voire même dépendre d’un web hook
    
* Des connecteurs pour certaines applications existent et proposent certaines actions déjà établies sous formes de cartes
    
* Pour les autres applications tierces, vous pouvez toujours passer par des cartes d’appels API custom, où vous définissez l’endpoint à atteindre, la méthode, les headers, et le flow d’autorisation
    
* Vous pouvez sauvegarder tout l’historique d’exécution d’un worfklow
    
* Gains de temps et de productivité, des tâches répétitives en moins pour vos collaborateurs
    

Les points abordés ici ne sont pas exhaustifs, puisque je n’ai pas parlé de toutes les cartes de fonctions, qui apportent de multiples fonctionnalités complémentaires : traitement et modifications de données (texte, objet, JSON), opérations de branching avec des if/else par exemple, etc…

Je reviendrai plus en détails sur ces fonctions complémentaires dans le prochain article.