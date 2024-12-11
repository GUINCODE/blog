---
title: "OKTA Workflows : le meilleur reste à venir"
datePublished: Fri Sep 29 2023 12:50:06 GMT+0000 (Coordinated Universal Time)
cuid: cln4lqd5s000808l05yuv10yc
slug: okta-workflows-le-meilleur-reste-a-venir
canonical: https://medium.com/@julien.nanquette/okta-workflows-le-meilleur-reste-%C3%A0-venir-ef347416d84e
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695991448408/41ab19bc-a7c6-465d-b60e-c94b47494a96.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1695991585899/15300d26-604d-49b9-9a35-4675e18193a8.png
tags: workflow, okta, identity-management, workflow-automation

---

[Dans mon article précédent,](https://hashnode.com/post/cln4krlij000w08l2fmr1edug) je vous présentais les workflows Okta de façon sommaire, afin de susciter votre curiosité quant au potentiel de cet outil. A priori, si vous êtes ici, c’est que c’est le cas ( si vous avez vu de la lumière et que vous êtes entré(e), c’est bien aussi !).

Aujourd’hui, je vais pousser un peu loin la présentation des workflows, et vais vous parler des nombreuses fonctionnalités que je n’avais pas ou peu évoqué jusqu’à présent. Le sujet sera les cartes fonctions, et les possibilités qu’elles apportent en plus des cartes applications.

## **Les cartes dans les workflows Okta, c’est quoi déjà ?**

![](https://miro.medium.com/v2/resize:fit:574/1*ftkxcoo3xrD80JjgKc5Q7Q.jpeg align="left")

*Rassurez-vous, rien à voir avec le FreeCell de Windows (moi, j’aimais bien)…*

Les cartes de la section applications vous permettent de chaîner des actions, qui concernent soit un tenant Okta , soit une application tierce, au moyen d’une “connection” (connecteur existant ou custom), si vous vous souvenez bien. Quelques exemples pour rappel : ajouter un utilisateur Okta à un groupe, lui assigner une application, ou encore envoyer un mail via Gmail, créer une requête dans Jira, etc.

![](https://miro.medium.com/v2/resize:fit:700/1*aQMOOY89-R1Jeq-IEzEFHw.png align="left")

*Des cartes applications (actions dans Okta en l’occurence)*

Vous suivez toujours ? Ok, maintenant, parlons des cartes fonctions !

## **Les cartes de la section fonctions**

Les cartes fonctions mettent à votre disposition un large panel d’opérations qui sont réparties en quatre sections : logic, manipulation, elements, et advanced.

![](https://miro.medium.com/v2/resize:fit:700/1*SResE5sfbpX4pLoXLGLNEg.png align="left")

*Le menu est fourni, et ce ne sont là que les sous-sections.*

## **Les fonctions de Logic**

Elles permettent de faire du branching, de la gestion d’erreur, et d’agir sur l’exécution d’un flow. Un exemple de branching simple est que vous pouvez déterminer avec des conditions la suite des actions à exécuter : si dans telle donnée on peut trouver la valeur définie, alors le workflow continuera une suite d’actions, et laissera de côté les autres branches. Ou décider que le workflow ne continuera à agir que si une condition est remplie.

![](https://miro.medium.com/v2/resize:fit:700/1*4QMT0Q7fhI9-X3_2UKOr8g.png align="left")

*Comparaison d’un ID de user avec une valeur attendue : pas de concordance, le workflow n’est pas allé plus loin.*

Sans surprise, les fonctions d’erreurs ont trait à la gestion d’erreurs, je ne m’étale pas dessus. Les fonctions de flow control, elles, ont pour vocation de gérer l’exécution du workflow, en permettant de le mettre en pause, de faire appel à un autre workflow pour récupérer des données ( de façon synchrone ou asynchrone ), ou encore de mettre fin au flow et de retourner des données.

![](https://miro.medium.com/v2/resize:fit:495/1*TrmpUcUyo7ldRwND2kxsCA.png align="left")

*Pause de trente secondes avant de passer un appel API à un 1Password SCIM Bridge pour s’assurer que les opérations précédentes ont bien été exécutées.*

Vous avez des cas d’usage différents en fonction de certains attributs de votre utilisateur, ou de ses permissions ? C’est là que vous trouverez l’intérêt du branching et des conditions. Il y a au final un grand nombre de fonctions de comparaisons, similaires à celles utilisées lors d’un développement classique.

![](https://miro.medium.com/v2/resize:fit:267/1*7yf26ibg3NWMWvT8IbpekQ.png align="left")

*Les opérateurs de comparaison*

## **Les fonctions de manipulation**

Ce sont toutes les fonctions qui vont vous permettre de traiter de la donnée dans un état initial pour la transformer, la réduire, l’étendre, la changer de format… Je pourrais passer une journée à vous décrire toutes les possibilités tellement elles sont nombreuses. Il y a de la manipulation de booléen, datetime, number, text, object, list (array) et même de fichier.

Prenons plutôt un exemple concret. Suite à un appel API, vous récupérez une liste (array) des utilisateurs qui font partie d’un groupe. Vous avez besoin de récupérer tous les user Id, pour ensuite les mettre dans un array sur lequel vous ferez tourner un autre workflow qui modifiera un attribut du profil de l’utilisateur.

![](https://miro.medium.com/v2/resize:fit:700/1*jO7L2mTQD-u4Lrz_OzFPyg.png align="left")

*Comment récupérer tous les userId depuis le résultat de List Group Members ?*

La solution, c’est une carte fonction qui l’apporte, avec la fonction “Pluck” qui va créer un nouvel array à partir du résultat, mais en n’utilisant qu’une valeur déterminée. Ici, on va donc choisir l’array Users, pour lequel on va demander la key ID, et hop, le tour est joué.

![](https://miro.medium.com/v2/resize:fit:700/1*soXhkNHJlr_ODHcdfMrJ0A.png align="left")

Et on finit par un For Each qui fera appel à un deuxième flow d’update de profil, en se basant sur chaque item de l’array “values”, qui est le résultat de la carte “Pluck”. RIen de sorcier, pas vrai ?

Mais le deuxième workflow qui prend la relève, il est lancé comment ? Et bien, c’est un “helper flow” : il est appelé par le premier, et lors de cet appel, on lui passe dans le contexte d’exécution les données dont il a besoin.

![](https://miro.medium.com/v2/resize:fit:533/1*gcXqEV-samq-eL-8FA7phQ.png align="left")

*Le userId est passé au contexte lors de la carte “Helper Flow” qui fait s’exécuter ce workflow dès que le premier l’appelle.*

Deuxième exemple concret, moins long (promis !) : vous voulez passer un appel d’API à Google, mais vous avez besoin de passer dans le body de la requête un objet JSON dépend de la composition d’un Id Token reçu. Vous récupérez le token, vous le décodez puisque c’est un JWT avec la carte “Decode”, et derrière, vous allez pouvoir construire un body en objet JSON à partir des valeurs que vous souhaitez, que vous enverrez ensuite dans un appel API.

![](https://miro.medium.com/v2/resize:fit:700/1*mPQI8FGAm2bt4SpX7UPfeQ.png align="left")

*Ici, on construit un body qui nous servira à être échangé contre un access de l’API Security Token Service de Google.*

## **Les fonctions elements**

Elles permettent d’exporter en fichier JSON des flows, des dossiers de workflows, ou de créer / manipuler un tableau csv. Je ne pense pas qu’il est nécessaire d’illustrer par un exemple ces fonctions, elles parlent suffisamment d’elles-mêmes.

## **Les fonctions advanced**

Elles touchent principalement à de la gestion de format : URL, JSON, JWT, XML, encryption. Vous trouverez également ici les cartes pour passer des appels API qui ne relèvent pas d’un connecteur tiers existant. A noter que la carte “Raw Request” vous donne une totale liberté de contrôle d’un appel API : vous définissez les headers, la méthode, le contenu, l’url de destination, et une query si besoin.

## **La lisibilité des workflows**

Vous pouvez visualiser de façon globale le déroulé d’un flow en cliquant sur “flow chart” qui vous montre un diagramme de l’exécution.

![](https://miro.medium.com/v2/resize:fit:700/1*nyTOw5U0NgrlRW6lxTkxpQ.png align="left")

*Le déroulé de l’exemple précédent*

En termes de vision globale, l’aperçu des flows existant vous montre aussi les types de cartes utilisées.

![](https://miro.medium.com/v2/resize:fit:700/1*wdnKVRRm0JM06zaaqBy1iw.png align="left")

*Pour le 1er flow : de l’APIP endpoint, du JWT, de l’object et du branching*

Récemment encore, Okta a ajouté de nouvelles fonctionnalités pour faciliter la gestion des workflows.

## **Comme dirait mon boucher, “Il y a un peu plus, je vous le mets ?”**

Okta Workflows vous met également à disposition des templates ! Parce que comme certains cas d’usages sont clairement communs à de nombreux clients, des exemples de réalisations vous sont fournis.

![](https://miro.medium.com/v2/resize:fit:700/1*BjxZv5X1IOs-7dEnsbkYxQ.png align="left")

## **Vous voulez plus d’informations, regarder une démo ?**

C’est [ici](https://www.okta.com/platform/workflows/) que ça se passe !

## **Conclusion**

Les workflows, vous l’aurez compris, ont un potentiel incroyable en ce qu’ils mettent à votre disposition un panel d’actions et de fonctions qui vous permettent d’automatiser, maintenir, modifier à volonté une suite d’actions à travers une interface low-code.

Si la gestion des cartes d’appels API demande quelques connaissances et compétences sur le sujet, c’est surtout de logique dont vous aurez besoin pour mettre en oeuvre vos workflows.

Pour rappel, si vous contractez une offre Okta Workforce Identity Cloud, vous aurez toujours cinq workflows gratuitement à disposition.