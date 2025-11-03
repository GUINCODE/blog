---
title: "Comment choisir le bon Flux OAuth 2.0/OIDC et Serveur d’autorisation Okta pour votre Infrastructure"
seoTitle: "choix de flux d'authentification OAuth 2.0 / OIDC et serveur d'authori"
datePublished: Mon Nov 03 2025 19:09:09 GMT+0000 (Coordinated Universal Time)
cuid: cmhjikc8j000002ldg6lr6jlp
slug: flux-oauth2-oidc
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1761732858170/2a300f1e-cda3-49cd-9b15-c698c666bfa7.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1761772790275/ceb345d2-3a6b-4b9d-982e-036f8e926d6c.png
tags: security, okta, iammfaaccess-key-idsecret-access-key, identity-access-management

---

## Introduction

Dans l'écosystème du développement moderne, la sécurisation des applications repose sur deux fondamentaux : l'authentification (qui est l'utilisateur ?) et l'autorisation (que peut-il faire ?). Si OAuth 2.0 et OpenID Connect se sont imposés comme standards incontournables, leur mise en œuvre concrète soulève souvent de nombreuses questions : quel flux d'autorisation utiliser ? Comment configurer son serveur d'autorisation ? Quelle stratégie adopter selon le type d'application ?

Avec Okta, ces choix techniques deviennent encore plus cruciaux : entre les multiples flux OAuth 2.0/OIDC disponibles et les deux types de serveurs d'autorisation (Org Authorization Server vs Custom Authorization Server), il est essentiel de comprendre les implications de chaque décision pour garantir à la fois la sécurité et la maintenabilité de votre système.

Cet article vous accompagne dans cette réflexion stratégique en clarifiant :

* **Les différents flux d'authentification/autorisation** et leurs cas d'usage selon votre architecture (application web traditionnelle, SPA, application mobile, API, ou service backend)
    
* **Les critères de choix** entre l'Org Authorization Server et un Custom Authorization Server
    
* **Les bonnes pratiques** pour implémenter une solution robuste et évolutive
    

Que vous débutiez avec Okta ou que vous cherchiez à optimiser une intégration existante, ce guide vous donnera les clés pour prendre les bonnes décisions techniques.

Dans le développement moderne d’applications, l’authentification et l’autorisation sont devenues des piliers essentiels de la sécurité. Entre OAuth 2.0, OpenID Connect, et les serveurs d’autorisation d’Okta, il est parfois difficile de savoir quel flux choisir pour son application ou quel serveur utiliser.

Cet article vous guide pas à pas pour comprendre **quel flux d’authentification** correspond à votre **type d’application** (Web, SPA, mobile, ou service), et **quel serveur d’autorisation** (Org ou Custom) répond le mieux à vos besoins.

*Avant de détailler OAuth 2.0 et OpenID Connect, faisons un point sur les serveurs d’autorisation (Org et Custom)*

## **Qu’est‑ce qu’un serveur d’autorisation (vue fonctionnelle)**

Un **serveur d’autorisation** est le composant qui **contrôle l’accès** à des ressources protégées dans un domaine de sécurité. Dans OAuth 2.0 / OIDC, il **reçoit des demandes** de la part d’applications clientes, **authentifie** l’utilisateur ou le client, **recueille le consentement**, **applique des politiques d’accès** (scopes, audiences, groupes, conditions), puis **émet des jetons** signés (Access Token, ID Token, et éventuellement Refresh Token). Chaque serveur est identifié par un **issuer (URI)** et publie sa **clé publique** pour que les API puissent vérifier les jetons.

**Entrées → Traitements → Sorties**

* **Entrées** : paramètres de la demande (client\_id, redirect\_uri, scope, code\_challenge PKCE…), identifiants/preuves d’authentification (utilisateur ou client), consentement.
    
* **Traitements** : authentification, évaluation des **policies**, génération et signature des jetons, journalisation.
    
* **Sorties** : redirection avec **authorization code** (selon le flow), **Access Token**, **ID Token**, **Refresh Token** ou erreurs normalisées.
    

### Les endpoints standard d’un serveur d’autorisation Okta

Chaque serveur (Org ou Custom) expose des URLs bien définies. Voici les plus importantes :

* **Discover**
    
    * [https://yourOktaDomain.com/.well-known/openid-configuration](https://wandj-iga.okta.com/.well-known/openid-configuration) → Retourne toute la configuration OIDC (endpoints, scopes supportés, clés, etc.).
        
    * [https://](https://wandj-iga.okta.com/.well-known/oauth-authorization-server)[yourOktaDomain](https://wandj-iga.okta.com/.well-known/openid-configuration)[.com/.well-known/oauth-authorization-server](https://wandj-iga.okta.com/.well-known/oauth-authorization-server) → Version OAuth 2.0 pure (moins courante).
        
* **Autorisation**
    
    * [https://](https://wandj-iga.okta.com/oauth2/%7Bauth_server_id%7D/v1/authorize)[yourOktaDomain](https://wandj-iga.okta.com/.well-known/openid-configuration)[.com/oauth2/{auth\_server\_id}/v1/authorize](https://wandj-iga.okta.com/oauth2/%7Bauth_server_id%7D/v1/authorize) → Démarre le flux : affiche la page de login, gère le consentement, redirige avec un code.
        
* **Émission de jetons**
    
    * [https://](https://wandj-iga.okta.com/oauth2/%7Bauth_server_id%7D/v1/token)[yourOktaDomain](https://wandj-iga.okta.com/.well-known/openid-configuration)[.com/oauth2/{auth\_server\_id}/v1/token](https://wandj-iga.okta.com/oauth2/%7Bauth_server_id%7D/v1/token) → Échange un code contre des tokens (Authorization Code + PKCE), ou délivre un token via client\_credentials.
        
* **Vérification et informations**
    
    * /v1/keys → Clés publiques (JWKS) pour vérifier la signature des tokens.
        
    * /v1/userinfo → Retourne les données de l’utilisateur (email, nom, etc.) avec un Access Token valide.
        
    * /v1/introspect → Vérifie si un token est actif, expiré ou révoqué (uniquement sur Custom Server).
        
    * /v1/revoke → Révoque un token immédiatement (logout forcé, sécurité).
        

> **Note** : Remplace {auth\_server\_id} par default (Custom Server par défaut) ou ton ID personnalisé. Pour l’**Org Authorization Server**, les chemins sont en /oauth2/v1/... sans auth\_server\_id.

### **Comprendre les fondamentaux : OAuth 2.0 et OpenID Connect**

Okta prend en charge plusieurs protocoles d'authentification et d'autorisation : SAML 2.0, Secure Web Authentication (SWA), ainsi que les standards modernes OAuth 2.0 et OpenID Connect (OIDC). Ces deux derniers se sont imposés comme les piliers des intégrations d'applications web, mobiles et API.

Alors que SAML et SWA répondent encore à des cas d'usage legacy, OAuth 2.0 et OIDC offrent une approche plus flexible et standardisée. Mais avant de plonger dans leurs subtilités, posons-nous les bonnes questions : avez-vous besoin de savoir **qui** se connecte (identité), **à quoi** il peut accéder (autorisations), ou s'agit-il d'un **service qui appelle un autre service** sans utilisateur humain ? Ces distinctions orientent directement vos choix d'architecture.

**OAuth 2.0 vs OpenID Connect : deux protocoles complémentaires**

* **OAuth 2.0** contrôle et délègue l'autorisation d'accès à une ressource protégée. Il assure la sécurité des APIs via des **Access Tokens** à portée limitée (scopes), sans exposer les identifiants utilisateur.
    
* **OIDC** étend OAuth 2.0 en ajoutant l'authentification et le Single Sign-On (SSO). Il fournit un **ID Token** (JWT signé) contenant l'identité vérifiée de l'utilisateur : nom, email, rôle, photo de profil.
    

**En pratique avec Okta** La majorité des applications nécessitent les deux protocoles. Lors d'un flux complet, Okta délivre à la fois un ID Token (identité) et un Access Token (autorisation). Okta propose des modèles de déploiement qui abstraient ces protocoles, vous n'avez donc pas à les manipuler directement.

Comprendre cette complémentarité vous permet de choisir le bon flux et d'éviter les erreurs courantes : utiliser un Access Token pour l'identité, exposer un ID Token à une API tierce, ou mal dimensionner la durée de vie des tokens

### **Choisir le flux OAuth 2.0 / OpenID Connect adapté à votre type d’application**

Lors de la création d’une nouvelle intégration d’application dans Okta, comme illustré dans la capture ci-dessous, vous êtes invité à choisir une **méthode de connexion (Sign-in method)** et un **type d’application (Application type)**.

Ces deux choix déterminent directement le **flux d’authentification OAuth 2.0 / OIDC** que votre application utilisera.

Okta propose plusieurs méthodes d’intégration, chacune adaptée à un cas d’usage particulier.

* **SAML 2.0** : un standard basé sur XML, principalement utilisé dans les environnements d’entreprise pour le Single Sign-On (SSO). Il reste adapté aux applications existantes qui ne prennent pas en charge OAuth 2.0 ou OIDC.
    
* **SWA (Secure Web Authentication)** : une méthode propre à Okta, utile lorsque l’application ne supporte ni SAML ni OIDC. Elle repose sur une authentification simulée via un formulaire web.
    
* **API Services** : utilisée pour les intégrations sans utilisateur (machine-to-machine), basée sur le flux *Client Credentials* d’OAuth 2.0.
    

Cependant, pour la plupart des applications modernes – web, mobiles ou orientées API – le choix recommandé est **OIDC – OpenID Connect**, qui s’appuie sur la puissance et la flexibilité du protocole OAuth 2.0.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761767004220/0cc332f5-eb00-4ebd-9510-50b37d9f2435.png align="center")

1. La section **Sign-in method** permet de sélectionner le protocole d’authentification :
    
    * **OIDC – OpenID Connect** (recommandé pour les applications modernes basées sur OAuth 2.0),
        
    * **SAML 2.0**,
        
    * **SWA (Secure Web Authentication)**,
        
    * ou **API Services** pour les échanges machine-to-machine.
        
2. La section **Application type** définit l’environnement et le flux approprié :
    
    * **Web Application** → flux *Authorization Code* (  + PKCE : Optionnel mais fortement recommandé )
        
    * **Single-Page Application** → flux *Authorization Code with PKCE*,
        
    * **Native Application** → flux *Authorization Code with PKCE*,
        
    * **API Services** → flux *Client Credentials*.
        

En combinant ces deux paramètres, Okta détermine automatiquement **le flux OAuth 2.0 ou OIDC le plus adapté** à votre architecture — qu’il s’agisse d’une application web, d’un front-end SPA, d’une app mobile ou d’un service backend.

> ### **OIDC : le standard moderne pour l’authentification et l’autorisation**

**OpenID Connect (OIDC)** étend le protocole **OAuth 2.0** en ajoutant une couche d’authentification.

Alors qu’OAuth 2.0 se concentre sur le contrôle d’accès aux ressources (autorisation), OIDC permet d’identifier l’utilisateur de manière sécurisée.

* ## **Authorization Code Flow :**
    

Le **Authorization Code Flow** (ou Code Flow) est défini par la spécification OAuth 2.0 ([RFC 6749, section 4.1](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)). C'est le flux le plus couramment utilisé pour les applications web côté serveur et constitue la base des intégrations OAuth/OIDC modernes.

Ce flux est conçu pour offrir un haut niveau de sécurité en séparant l'authentification de l'utilisateur (qui se déroule dans le navigateur) de l'échange de tokens (qui se produit de manière sécurisée entre serveurs backend). Les tokens d'accès ne transitent jamais directement dans le navigateur, réduisant ainsi les risques d'exposition.

![Sequence diagram that displays the interactions between the resource owner, authorization server, and resource server for Authorization Code flow"](https://developer.okta.com/img/authorization/oauth-auth-code-grant-flow.png align="left")

> **Étapes du flux Authorization Code**

1. **Ton application demande un code d’autorisation** Elle redirige l’utilisateur vers Okta (/authorize).
    
2. **Okta affiche la page de connexion** L’utilisateur voit le login Okta dans son navigateur.
    
3. **L’utilisateur se connecte et donne son accord** Il entre ses identifiants, accepte les permissions demandées.
    
4. **Okta renvoie un code d’autorisation au navigateur** Le code arrive dans l’URL de redirection (ex: ?code=abc123).
    
5. **Ton application envoie le code + le client secret à Okta** Appel backend vers /token (pas dans le navigateur !).
    
6. **Okta répond avec les tokens** Tu reçois :
    
    * access\_token (pour appeler l’API)
        
    * id\_token (identité de l’utilisateur)
        
    * refresh\_token (optionnel, pour renouveler)
        
7. **Ton application appelle l’API avec l’access\_token** En-tête : Authorization: Bearer &lt;token&gt;
    
8. **L’API vérifie le token avant de répondre** Elle utilise les clés publiques d’Okta (/keys) pour valider la signature.
    

> <div data-node-type="callout">
> <div data-node-type="callout-emoji">💡</div>
> <div data-node-type="callout-text"><strong><em>Recommandation</em></strong>: Franchement, mettez PKCE partout — SPA, mobile, ou même backend</div>
> </div>
> 
> En fait, depuis 2023, il est **fortement recommandé d’utiliser le flux “Authorization Code Flow with PKCE”**, y compris pour les applications web côté serveur.
> 
> Cette approche, initialement conçue pour les applications mobiles et SPA, ajoute une couche de sécurité supplémentaire au flux classique en protégeant l’échange du code d’autorisation.
> 
> Nous reviendrons plus loin sur le fonctionnement détaillé du **PKCE (Proof Key for Code Exchange)** et sur les raisons de son adoption généralisée.
> 
> **Référence :** [Okta Developer Docs – Authorization Code Flow with PKCE](https://developer.okta.com/docs/concepts/oauth-openid/)

* ## **Authorization Code Flow with PKCE : la version renforcée du Code Flow**
    

Le **Authorization Code Flow with PKCE (Proof Key for Code Exchange)** est une évolution du flux d’autorisation classique décrite dans la **RFC 7636**.

À l’origine, ce mécanisme a été conçu pour les **applications mobiles et Single Page Applications (SPA)**, où le client\_secret ne peut pas être protégé.

Il est aujourd’hui **recommandé pour toutes les applications**, y compris celles côté serveur, car il renforce la sécurité du processus d’échange de tokens.

> **Principe de fonctionnement**

Le fonctionnement reste globalement identique au *Code Flow classique*, à une différence majeure :

l’application génère une **clé temporaire unique**, appelée code\_verifier, qu’elle transforme en un **code challenge** (haché).

Cette clé permet au serveur d’autorisation (Okta) de vérifier que le code d’autorisation échangé n’a pas été intercepté ou modifié pendant le processus.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761768916324/3a05c0cf-e069-4a72-8d3b-ccf3d933a012.png align="center")

> **Étapes du flux Authorization Code avec PKCE**

1. Génération de la clé PKCE  
    L’application commence par générer un *code verifier* (clé aléatoire) et en déduit un *code challenge* (sa version hachée). Ces deux éléments serviront à prouver l’identité de l’application lors de l’échange du code d’autorisation.
    
2. Demande d’un code d’autorisation  
    L’application redirige l’utilisateur vers le serveur d’autorisation Okta (/authorize), en incluant le code\_challenge.
    
3. Affichage de la page de connexion  
    Okta affiche la page de connexion à l’utilisateur. Celui-ci choisit une méthode d’authentification (identifiant/mot de passe, SSO, etc.).
    
4. Authentification et consentement  
    L’utilisateur s’identifie et, si nécessaire, accepte les autorisations demandées par l’application (scopes).
    
5. Redirection vers l’application  
    Une fois authentifié, Okta redirige le navigateur vers l’application avec un **code d’autorisation temporaire**.
    
6. Échange du code d’autorisation contre des tokens  
    L’application envoie ce code à Okta, accompagné du code\_verifier, pour prouver qu’elle est bien à l’origine de la demande.
    
7. Validation du code PKCE  
    Okta compare le code\_verifier reçu avec le code\_challenge initial afin de vérifier l’intégrité de la requête.
    
8. Remise des tokens  
    Si tout est correct, Okta renvoie un **Access Token**, un **ID Token**, et éventuellement un **Refresh Token**.
    
9. Accès aux ressources protégées  
    L’application utilise l’Access Token pour interroger une API (le *Resource Server*) au nom de l’utilisateur.
    
10. Réponse du serveur de ressources  
    L’API valide le token et renvoie les données demandées.
    

> Exemple de code : Authorization code flow + PKCE

```javascript
// On commence par générer deux valeurs aléatoires et uniques pour chaque connexion :
// 1. code_verifier : une chaîne longue et imprévisible (minimum 43 caractères)
//    Elle sert à prouver que c'est bien notre application qui a initié la demande.
//    On la garde en mémoire (session ou stockage local) pour la réutiliser plus tard.
const { generators } = require('openid-client');
const code_verifier = generators.codeVerifier();

// 2. state : une autre valeur aléatoire, différente à chaque fois.
//    Elle empêche les attaques CSRF : si quelqu'un essaie de forcer une redirection,
//    cette valeur ne correspondra pas à celle qu'on a envoyée.
const state = require('crypto').randomBytes(32).toString('hex');

// On calcule le code_challenge à partir du code_verifier (hash en SHA256)
const code_challenge = generators.codeChallenge(code_verifier);

// On construit l'URL complète pour rediriger l'utilisateur vers Okta
const authUrl = `https://my-domaine.okta.com/oauth2/default/v1/authorize?` +
  `client_id=0oaxxx` +
  `&response_type=code` +
  `&scope=openid profile email` +
  `&redirect_uri=http://localhost:3000/callback` +
  `&state=${state}` +              // Valeur aléatoire, vérifiée au retour
  `&code_challenge=${code_challenge}` +  // Preuve que le code vient bien de nous
  `&code_challenge_method=S256`;

// On supprime les espaces et retours à la ligne pour une URL propre
const cleanUrl = authUrl.replace(/\s/g, '');

// On stocke les deux valeurs secrètes avant de rediriger
req.session.code_verifier = code_verifier;
req.session.state = state;

res.redirect(cleanUrl); // L'utilisateur est envoyé vers la page de login Okta
```

## **Client Credentials Flow : le flux pour les communications machine-to-machine**

Le **Client Credentials Flow** est conçu pour les applications **sans utilisateur final** — autrement dit, pour les scénarios où **une application ou un service doit accéder à une ressource protégée au nom d’elle-même**.

Ce flux est typique dans les architectures de microservices ou pour les automatisations backend, où un service doit appeler une API sécurisée.

> **Principe général**

Contrairement aux autres flux OAuth 2.0, celui-ci **ne passe pas par une interaction utilisateur**.

L’application cliente s’authentifie directement auprès du serveur d’autorisation à l’aide de ses propres identifiants (généralement un *Client ID* et un *Client Secret*), et obtient en retour un **Access Token**.

Ce jeton permet ensuite d’appeler les API protégées.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1761769869654/e1d04397-da82-49d3-9a49-184422bb50af.png align="center")

> **Étapes du flux Client Credentials Flow**

* Requête de jeton d’accès  
    L’application envoie une requête au serveur d’autorisation Okta vers le point de terminaison /token, en s’authentifiant à l’aide de ses propres identifiants (client\_id et client\_secret).  
    Ces identifiants sont configurés lors de la création de l’intégration d’application (type *API Services*).
    
* Réponse avec le jeton d’accès  
    Si les informations d’identification sont valides, Okta renvoie un **Access Token** signé, limité dans le temps et contenant les *scopes* autorisés.
    
* Appel de l’API avec le jeton  
    L’application utilise ce jeton d’accès pour appeler le **Resource Server** (votre API sécurisée). Le jeton prouve que la requête vient d’une application autorisée.
    
* Validation et réponse  
    Le serveur de ressources vérifie la validité du jeton auprès d’Okta avant de fournir les données ou d’exécuter l’action demandée.
    

> <div data-node-type="callout">
> <div data-node-type="callout-emoji">💡</div>
> <div data-node-type="callout-text"><strong>Points clés à retenir</strong></div>
> </div>
> 
> * Ce flux ne fournit **pas d’ID Token**, car **aucun utilisateur n’est impliqué**.
>     
> * Les **scopes** doivent être définis avec précision afin de limiter les permissions du client.
>     
> * Les identifiants (client\_id et client\_secret) doivent être stockés **de manière sécurisée** (jamais dans le code source).
>     
> * En pratique, ce flux est presque toujours utilisé avec un Custom Authorization Server, car il permet de créer des tokens sur mesure pour vos propres APIs.
>     

> Exemple de code : Client Credentials Flow (M2M)

```javascript
const getApiToken = async () => {
  // Prépare la requête sans utilisateur
  const params = new URLSearchParams({
    grant_type: 'client_credentials',
    scope: 'api:read api:write' // -> Scopes définis dans le Custom Server
  });

  // Authentifie le service avec client_id + secret, ici on pointe sur le custom serveur
  const response = await fetch('https://my-domain.okta.com/oauth2/default/v1/token', {
    method: 'POST',
    headers: {
      'Authorization': 'Basic ' + Buffer.from('0oayyy:secret').toString('base64'),
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: params
  });

  const { access_token } = await response.json();
  return access_token; // -> Token pour appeler ton API
};

// Exemple d'appel API
const token = await getApiToken();
fetch('https://api.monservice.com/data', {
  headers: { Authorization: `Bearer ${token}` }
});
```

## **Les flux OAuth 2.0 dépréciés : Implicit et ROPG**

Bien que toujours définis dans la spécification **OAuth 2.0 (RFC 6749)**, certains flux historiques sont aujourd’hui considérés comme **obsolètes** ou **à usage restreint**.

C’est notamment le cas de l’**Implicit Flow** et du **Resource Owner Password Credentials Flow (ROPG)**.

L’**Implicit Flow**, conçu à l’origine pour les applications Single-Page (SPA) dépourvues de capacités sécurisées côté serveur, permettait de récupérer un jeton d’accès directement via l’URL après redirection.

Mais cette approche présente plusieurs failles : exposition du jeton dans le navigateur, impossibilité de vérifier sa signature, et risques de vol via des scripts tiers.

C’est pourquoi Okta, comme l’ensemble de l’écosystème OAuth, recommande désormais de le remplacer par le Authorization Code avec PKCE plus sûr et universel.

Le **Resource Owner Password Flow (ROPG)**, lui, autorise une application à collecter directement les identifiants de l’utilisateur (nom d’utilisateur et mot de passe) pour obtenir un jeton d’accès.

Ce flux contourne toutefois la page d’authentification Okta et empêche l’application de bénéficier de mécanismes avancés tels que la **MFA**, la détection d’anomalies ou l’authentification adaptative.

Okta précise dans sa documentation officielle que ce flux ne doit être utilisé **que dans les environnements de confiance** ou à des fins de compatibilité technique :

“If your app is high-trust and you own both the client and the resource it accesses, you can use the Resource Owner Password flow. However, this flow is not compatible with multifactor authentication, and should be replaced by the Authorization Code or Interaction Code flow whenever possible.”

(Source : [Okta Developer Docs – OAuth & OpenID Connect Concepts](https://developer.okta.com/docs/concepts/oauth-openid/#resource-owner-password-flow))

> <div data-node-type="callout">
> <div data-node-type="callout-emoji">💡</div>
> <div data-node-type="callout-text"><strong>Recommandation actuelle</strong></div>
> </div>
> 
> Depuis 2023, les bonnes pratiques de sécurité (Okta, IETF, Auth0) convergent :
> 
> Utilisez le Code Flow with PKCE pour toutes les applications avec utilisateur, et le Client Credentials Flow pour les échanges entre services.

## **Conclusion**

On arrive au bout de ce guide, et l’essentiel tient en une idée : **choisir le bon flux OAuth/OIDC et le bon serveur Okta, c’est protéger ton application sans te prendre la tête.**

* **Si tu as des utilisateurs** (web, SPA, mobile) → **Authorization Code avec PKCE, point final.** C’est le standard depuis 2023, ça marche partout, ça supporte la MFA, et les SDK Okta le font tout seul.
    
* **Si c’est du machine-to-machine** → **Client Credentials + Custom Authorization Server.** C’est là que tu définis tes propres audiences, scopes, claims. Tes APIs deviennent vraiment à toi.
    
* **Oublie l’Implicit et le ROPG.** Sauf si t’as une vieille app qui traîne dans un coin… et même là, planifie la migration.
    

> **Org vs Custom : le match en 3 lignes**

|  | **Org Server** | **Custom Server** |
| --- | --- | --- |
| **Pourquoi ?** | Login, SSO, apps simples | APIs perso, microservices, |
| **Souplesse** | Scopes Okta uniquement | Tu fais ce que tu veux (scopes custom) |
| **Quand l’utiliser ?** | Prototype, SaaS, interne | Controle total |

> **Mon conseil de dev à dev** : Commence avec l’**Org Server** — c’est prêt en 2 clics. Dès que tu a besoin d’un scope custom, une audience propre ou un claim métier → **passe au Custom**.