---
title: "Okta API Access Management : une métaphore du Chevalier Noir"
datePublished: Tue Oct 10 2023 13:31:51 GMT+0000 (Coordinated Universal Time)
cuid: clnkd2fon000108mif9lf9dgf
slug: okta-api-access-management-une-metaphore-du-chevalier-noir
canonical: https://medium.com/@julien.nanquette/okta-api-access-management-une-m%C3%A9taphore-du-chevalier-noir-b163fadb9f60
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695992176063/4ea250cd-7a63-4386-a8bc-20877952749c.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1695992275100/1cabb714-98ed-4559-80f1-cf1cb89cbb5e.jpeg
tags: api, security, okta, api-management

---

Récemment, alors que je réfléchissais au sujet d’un nouvel article, mon regard s’est arrêté sur mon bureau. Comme souvent, un comic de Batman se trouvait à côté de mon écran droit (pour ceux qui se posent la question, il s’agit de “Batman Nocturne”). Il m’est alors venu une idée très geek : si je devais expliquer certains concepts d’Okta à un ami, j’utiliserais l’univers de Batman.

N’ayez pas peur, je ne vais pas faire un cours sur le Chevalier Noir, je suis presque sûr que tout le monde en sait un peu plus sur lui (son identité, son but, comment il a choisi de devenir Batman…). Au lieu de cela, je vais essayer d’utiliser des métaphores et des comparaisons avec son univres pour expliquer ce qu’est l’Okta API Access Management.

## Okta dans l’univers de Batman

Vous pouvez considérer l’organisation Okta WIC (Workforce Identity Cloud) comme la Batcave. Vous ne pouvez y accéder que si vous avez été ajouté en tant qu’utilisateur par un administrateur (pour l’instant, ne parlons pas des imports pilotés par les RH ou du provisionnement automatisé). Certaines ressources vous seront accessibles, d’autres non. Il se peut que certaines applications vous aient été attribuées et que vous ayez été placé dans certains groupes. De plus, pour accéder à la Batcave, certains utilisateurs devront utiliser un mot de passe ou un scanner rétinien. Oui, Batman utilise l’authentification multifactorielle (il est à la pointe de la technologie).

![](https://cdn-images-1.medium.com/max/800/0*qEhK0oEJQVR7vWz4.jpeg align="left")

*La Batcave*

Par conséquent, considérez Batman comme un super administrateur. Il a des droits sur tout et s’occupe de la gestion de l’organisation. Il peut accorder à d’autres utilisateurs des rôles d’administrateur standard, leur attribuer des applications ou même créer des rôles d’administrateur personnalisés pour leur attribuer des droits très spécifiques sur certaines ressources.

En tant que super administrateur, Batman accorderait probablement des droits d’administration à Nightwing, Robin et autres acolytes, mais pas nécessairement les mêmes.

Imaginez les applications d’Okta comme les véhicules de Batman : Robin n’est pas autorisé à monter sur la moto de Nightwing, mais peut prendre place dans la Batmobile. Vous voyez ce que je veux dire ?

Avant d’aborder le sujet principal de la gestion de l’accès à l’API Okta, je vais faire une autre métaphore : imaginez les API comme toutes les ressources de données que Batman possède. Ses fichiers et dossiers sur tous les criminels de Gotham ne sont pas à la libre disposition de tout le monde. Vos API ne sont pas non plus sans protection, il faut être authentifié et autorisé pour accéder à vos ressources.

## Okta API Access Management

L’Okta API Access Management (APIAM) est une fonctionnalité payante qui vous permet de construire des serveurs d’autorisation personnalisés, que vous utilisez ensuite pour protéger vos propres API endpoints. Notez que l’APIAM est inclus dans les organisations de développeurs Okta (gratuites), car l’objectif est de permettre aux développeurs de découvrir pleinement son utilité.

### Les serveurs d’autorisation

Un serveur d’autorisation est un serveur dont l’objectif est de fournir des Access Tokens(jetons d’accès), qui sont la représentation d’un accès spécifique accordé à un utilisateur ou à une autre machine à certaines ressources spécifiques. Il utilise OAuth 2.0, qui est un protocole standard pour l’autorisation, et l’overlay OpenID Connect (OIDC) pour générer un jeton d’identification en parallèle.

Dans l’univers de Batman, un serveur d’autorisation est comme le Bat-ordinateur.

![](https://cdn-images-1.medium.com/max/800/0*Db1ooSomZDoZBe9q.jpg align="left")

*Sympa le serveur d’autorisation…*

En ce qui concerne l’utilisateur, le Bat-ordinateur autorise ou refuse l’accès aux ressources demandées. La demande peut être faite par un utilisateur, comme Robin, qui veut connaître tous les repaires connus du Sphinx par exemple. Mais la demande peut également provenir d’une autre organisation Okta, comme si Oracle (Barbara Gordon, à la retraite en tant que Batgirl, après s’être faite tirer dessus par le Joker, et restée paralysée) utilise son propre ordinateur depuis sa tour de guet pour accéder aux données.

Dans chaque cas d’utilisation, une demande est adressée au serveur d’autorisation, sur la base du jeton d’accès de l’utilisateur ou de la machine qui fait la demande. Le serveur d’autorisation vérifie les scopes de l’Access Token, les claims, les informations sur l’émetteur et la durée de vie du jeton avant de prendre une décision.

### Access Token, scopes et claims

Dans l’API Okta, les Access Tokens se présentent sous la forme de Json Web Token (JWT). Gardez à l’esprit qu’ils contiennent des claims, qui sont des informations sur l’entité qui a demandé le jeton (“sub” — pour subject), l’émetteur du jeton (“iss”), un horodatage du moment où il a été émis (“iat”), sa durée de vie (“exp”), l’audience attendue (“aud” — un clientId ici), et d’autres informations personnalisées.

Voyons ce que serait un Access Token délivré à Red Hood, l’un des alliés de Batman :

```plaintext
{
    "sub": "1234567890",
    "name": "Red Hood",
    "iss": "9xg45XDe34",
    "aud": "0oa9rt9d91hy1iS8Z1d7",
    "iat": 1693487184,
    "exp": 1693490784,
    "jti": "ID.fVwsJy_qCrcp23etC_1EtK6iKLby2kPdZp-75TYU47hAk",
    "amr": [
       "pwd"
    ],
    "admin": true,
    "role": "ally",
    "true_Id": "Jason Todd"
}
```

*Dans ce Token, le claim “true\_Id” est personnalisé, tandis que “name” ou “amr” sont des exemples de registered claims (standard).*

#### Les scopes

  
Les scopes sont les permissions accordées aux utilisateurs : ils définissent ce que les utilisateurs peuvent faire ou ne pas faire. Contrairement aux claims qui sont reçues dans un Access Token, les scopes sont envoyés avec la demande d’autorisation. Dans un tenant Okta, vous pouvez avoir des scopes par défaut et des scopes custom (personnalisés).

Si Red Hood avait demandé un Access Token avec le flux de code d’autorisation, cela pourrait ressembler à ceci :

```plaintext
https://batcomputer.com/oauth2/v1/authorize?client_id=0op8xwtmarwTfDzDT427&redirect_uri=https%3A%2F%2Foidcdebugger.com%2Fdebug&scope=openid%20profile%20email%20address&response_type=code&response_mode=form_post&state=4a8wvmqbbea&nonce=96vc3gdc6jd
```

Les scopes par défaut renvoient des informations standard sur l’utilisateur (comme “email”, “address”), à l’exception du scope “openid” : celui-ci est utilisé pour indiquer que la demande d’autorisation est faite avec le protocole OIDC. Dans ce cas, un ID Token est reçu avec l’Access Token. Le scope “profile” renvoie vos propres informations de profil.

#### Scopes de l’API Okta et scopes custom

En ce qui concerne les scopes de l’API Okta, ils ne peuvent être utilisés qu’avec le serveur d’autorisation Org, et dans le seul cas d’utilisation des requêtes à l’API Okta. Comme il s’agit d’un serveur d’autorisation prédéfini qui n’est pas destiné à être modifié, le serveur d’autorisation Org ne peut pas être vu sur le tenant Okta. Ces scopes ont une désignation claire qui indique ce que leur permission concerne. Par exemple, “[okta.users.read](http://okta.users.read)” permet uniquement de lire les informations de l’utilisateur, tandis que “okta.users.manage” donne la possibilité de lire et de modifier les informations du profil de l’utilisateur. Pour plus de détails, vous pouvez consulter la liste des scopes.

D’autre part, l’Okta APIAM vous offre la possibilité de construire et d’utiliser des serveurs d’autorisation personnalisés, et de définir vos propres scopes et claims. Contrairement aux scopes de l’API Okta, il n’y a pas de liste préexistante de custom scopes : c’est à vous de les définir, pour répondre au mieux à vos besoins, en demandant une API sur mesure. Si vous avez payé pour APIAM, vous aurez un serveur d’autorisation personnalisé par défaut, dont le nom est “default” (sans suspense).

![](https://cdn-images-1.medium.com/max/800/0*Pp845IMOzf316N7Q.png align="left")

*Les serveurs d’autorisation custom*

#### Les politiques d’accès (Access policies)

Vous savez donc maintenant qu’avec l’APIAM, vous disposez d’un serveur d’autorisation par défaut et de serveurs d’autorisation personnalisés. Vous pouvez définir des scopes et des custom claims.

![](https://cdn-images-1.medium.com/max/800/0*WT9GtnNcjn0MY7hO.png align="left")

*Une vue d’ensemble d’un serveur d’autorisation custom*

Cependant, comme vous l’avez probablement deviné, ce n’est pas tout ce que vous offre l’APIAM. Les access policies constituent une autre partie intéressante : elles vous permettent de déterminer une réponse personnalisée du serveur d’autorisation au niveau de la configuration de l’Access Token, pour une application OIDC spécifique. Pour chaque politique d’accès, vous pouvez définir différentes règles pour répondre à des besoins spécifiques. Pour accéder à cette fonctionnalité, vous devez suivre le chemin de menu “Security” &gt; “API” et ensuite, sur votre serveur d’autorisation personnalisé ciblé, cliquer sur l’icône du stylo.

![](https://cdn-images-1.medium.com/max/800/0*MZrdg-XZrH1X4Bo9.png align="left")

*Par ici !*

![](https://cdn-images-1.medium.com/max/800/0*4EMjelqxMeV8nu2-.png align="left")

*Politique d’accès et règle pour un client OIDC nommé “Julien\_Tests”*

Dans chaque règle, vous pouvez définir :

* quel grant type peut être utilisé
    
* si la règle doit être appliquée à n’importe quel utilisateur ou à un utilisateur appartenant à un groupe spécifique
    
* si un scope spécifique doit être demandé
    
* l’utilisation d’un inline hook si nécessaire
    
* la durée de vie du jeton
    
* la durée de vie du refresh token, y compris un délai d’expiration si le jeton n’a pas été utilisé
    

![](https://cdn-images-1.medium.com/max/800/0*U_f8B77qVX_voRrT.png align="left")

*Une configuration très spécifique peut être réalisée*

Très utile lorsque vous souhaitez une configuration personnalisée et spécifique pour une application OIDC, n’est-ce pas ? Vous devez maintenant vous demander comment vous pouvez vous assurer que votre règle est valide et qu’elle s’appliquera correctement comme vous le souhaitez. L’onglet “Token Preview” vous y aidera.

#### Un cas d’utilisation simple

  
Pour cet exemple, j’ai créé un scope custom nommée “userType” dont la valeur retourne “user.title”, et ensuite un custom claim nommé “Role”, pour laquelle la valeur de Red Hood retournera “ally”. Ce custom claim est inclus dans mon custom scope. Je définis une politique d’accès qui fixe la durée de vie du jeton d’accès à 8 heures, si mon scope”userType” est demandé.

![](https://cdn-images-1.medium.com/max/800/0*z6LSWxY3zoqmqd1f.png align="left")

*La règle créée pour notre exemple*

Ici vous pouvez constater ce que la preview du token montre lorsqu’on simule un requête pour un Access Token par Red Hood avec le scope “userType”, pour l’application OIDC “Bat-computer”.

![](https://cdn-images-1.medium.com/max/800/0*e4ADRyQlhPBix4Bi.png align="left")

En bas, vous pouvez voir le claim “role” qui renvoie la valeur de l’attribut “title” du profil de Red Hood : “allié”

Tout a fonctionné jusqu’à présent, car la demande était conforme à la politique d’accès et à sa règle. Si Red Hood avait essayé la demande sans le scope “userType” et avait utilisé “openid” et “profile” à la place, elle aurait échoué.

![](https://cdn-images-1.medium.com/max/800/0*pI7gR8-rs5NMoXrt.png align="left")

*L’access policy a dit “Merci, mais non”.*

## Conclusion

  
Je pense que vous voyez maintenant plus clairement l’ensemble du tableau. Voici donc un petit récapitulatif de ce que vous obtenez avec Okta APIAM :

* Vous avez un contrôle complet et précis de qui peut accéder à vos API, et comment il peut le faire.
    
* Vous pouvez définir plusieurs règles et politiques d’accès, chacune répondant à un besoin spécifique.
    
* Vous pouvez créer des serveurs d’autorisation personnalisés et les utiliser à des fins spécifiques.
    
* Les politiques d’accès vous permettent de compléter la protection des routes API dans le code.
    
* Vous couvrez ainsi toutes les situations possibles. Comme Batman, vous ne laissez aucune place à la chance, tout est sous contrôle.
    

N’oubliez pas que cette fonctionnalité est incluse dans les tenants développeurs d’Okta, mais que vous devez payer pour cette fonctionnalité dans les organisations Okta de preview et de production.

Maintenant, si vous voulez bien m’excuser, je vois quelque chose à travers ma fenêtre, je dois y aller.

![](https://cdn-images-1.medium.com/max/800/0*-PdIo0VqB6jxEaL3.jpeg align="left")

*Je ne dis pas que je suis Batman, je dis juste qu’on ne nous a jamais vu ensemble.*