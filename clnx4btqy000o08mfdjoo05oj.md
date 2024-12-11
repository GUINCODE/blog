---
title: "Guide d'intégration de l'authentification Okta à une application Angular"
seoTitle: "guide intégration de l'authentification à Okta avec une app Angular"
datePublished: Thu Oct 19 2023 11:48:13 GMT+0000 (Coordinated Universal Time)
cuid: clnx4btqy000o08mfdjoo05oj
slug: guide-dintegration-de-lauthentification-okta-a-une-application-angular
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697228612322/d943f7a9-960d-4e30-82d6-6593df970347.jpeg
tags: angularjs, okta, application-security, identity-management

---

## Présentation de Okta

Okta est une entreprise spécialisée dans la gestion de l'identité et de l'accès, offrant une gamme complète de solutions pour l'authentification, l'autorisation et la gestion des utilisateurs. Pour notre sujet d'aujourd'hui, nous nous concentrerons sur la composante "authentification" d'Okta. Elle joue un rôle essentiel dans la sécurisation de l'accès à notre application Angular, offrant une solution robuste pour garantir la sécurité des utilisateurs.

## **Configuration et Intégration de l'authentification Okta à Angular**

### **I. Configuration de notre solution d’authentification sur Okta**

Si vous ne disposez pas d'un compte Okta, vous devez créer un compte avec votre e-mail professionnel. Pour ce faire, rendez-vous sur le [**site web d'Okta**](https://www.okta.com/) et suivez les instructions pour créer un compte. Une fois connecté, cliquez sur le bouton "**ADMINISTRATION**", puis sur l'onglet "**Applications**", ensuite sur le sous-onglet "**Applications**", vous aurez une vue comme suit :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697445350052/3ff95f50-b734-4887-a6a4-880009a08a21.png align="center")

Maintenant, nous allons créer une "**App Integration**" (intégration d'application) qui nous permettra de lier spécifiquement notre application Angular à la plateforme Okta. Pour ce faire, cliquez sur "**Create App Integration**", vous obtiendrez une vue comme celle-ci :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697288089421/7df181f9-56b8-4de9-bce0-5eaa225e5fa4.png align="center")

Lors de cette étape, nous devons spécifier deux éléments essentiels : la "**méthode d'authentification**" (sign-in method) et le "**type d'application**" à intégrer. Nous choisissons OIDC (OpenID Connect) comme méthode d'authentification en raison de ses avantages, notamment l'authentification unique (Single Sign-On), la sécurité basée sur OAuth 2.0 et l'utilisation de JWT (JSON Web Tokens). Quant au "type d'application", nous optons pour "Single-Page Application" en raison de la nature d'Angular, qui est une application de type Single-Page Application.  
Cliquez ensuite sur "Next".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697313373968/9e22b3fc-56fc-42bd-8d65-19bf76f7c904.png align="center")

Sur cette page, nous avons saisi des informations dans les champs suivants :

1. **App Integration Name** : Il s'agit du nom de votre intégration d'application.
    
2. **Grant type** : Ce champ permet de spécifier la méthode par laquelle une application peut obtenir un jeton d'accès pour interagir avec une API ou un service. Dans notre cas, nous allons utiliser le flux d'autorisation par code d'autorisation (Authorization Code), qui est considéré comme la méthode la plus sécurisée.
    
3. **Sign-in redirect URIs** : Ici, vous devez indiquer les URL de redirection une fois que l'utilisateur a été authentifié sur Okta. Dans notre cas, nous avons configuré cette URL comme [http://localhost:4200/login/callback](http://localhost:4200/login/callback). Veuillez noter que nous avons utilisé le port par défaut d'Angular, mais si vous avez modifié le port par défaut d'Angular, assurez-vous de renseigner le port correct.
    
4. **Sign-out redirect URIs** : Cette option est similaire à "Sign-in redirect URIs", mais elle concerne la déconnexion. Ici, nous avons renseigné [http://localhost:4200/login](http://localhost:4200/login) pour rediriger l'utilisateur vers la page de connexion (chemin '/login') après sa déconnexion.
    
5. **Assignements** : Dans ce champ, vous spécifiez les personnes autorisées à s'authentifier pour utiliser votre application. Dans notre cas, nous avons choisi "Limit access to selected groups" pour indiquer que seuls les utilisateurs du groupe que nous avons créé dans Okta seront autorisés à s'authentifier et à utiliser notre application. Cependant, vous pouvez choisir de spécifier les utilisateurs ou groupes ultérieurement.
    

Il est essentiel de noter que ces informations sont cruciales pour configurer correctement votre intégration d'application et définir qui peut y accéder. Une fois enregistrées, vous aurez besoin des informations suivantes pour lier cette intégration à votre application Angular :

* **issuer** : 'votre-domain/oauth2/default'
    
* **clientId** : 'votre identifiant client' (visible directement dans l'onglet général de l'intégration que vous venez de créer)
    
* **redirectUri** : 'URL' (l'URL que vous avez précédemment fournie pour le champ "Sign-in redirect URIs")
    
* **postLogoutRedirectUri** : 'URL' (l'URL que vous avez précédemment fournie pour le champ "Sign-out redirect URIs")
    

Veuillez noter que pour trouver votre domaine, vous pouvez cliquer sur votre adresse e-mail en haut de la page, où vous verrez votre domaine, par exemple, "[trial-###...###.okta.com](http://trial-3408718.okta.com)" (dans le cas d'un domaine d'essai).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697536186005/8185211b-3547-4b28-a9f4-f730d906b94a.png align="center")

Après avoir pris note de ces informations essentielles, nous pouvons désormais passer à la deuxième étape : l'intégration dans notre application Angular

### **II. Intégration de notre solution d'authentification à Angular**

Assurez-vous d'avoir créé votre application Angular. Une fois cela fait, installez les packages suivants :

```bash
npm install @okta/okta-auth-js
```

```bash
npm install @okta/okta-angular
```

Ajoutez le code suivant à votre fichier /src/environments/environment.ts, en remplaçant les valeurs par celles que vous avez notées précédemment :

* ```javascript
        export const environment = {
          production: false,
          oktaAuth :{
          issuer: 'https://trial-34##....#718.okta.com/oauth2/default',
          clientId: '08rp####....###w697',
          redirectUri: 'http://localhost:4200/login/callback',
          postLogoutRedirectUri: 'http://localhost:4200/login',
          devMode: true,
          }
        };
    ```
    

Note : *Nous avons ajouté "devMode: true" pour afficher dans la console l'objet retourné contenant les informations de l'utilisateur et le jeton. Cette option peut être utile si vous prévoyez de traiter ces informations, car elle permet de visualiser la structure de l'objet retourné*.

À présent, dans le fichier app.module.ts, nous allons importer la variable d'environnement et configurer le module OktaAuthModule.

```javascript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { OktaAuthModule, OKTA_CONFIG } from '@okta/okta-angular'; // import OktaAuthModule
import { OktaAuth } from '@okta/okta-auth-js';
import { HttpClientModule } from '@angular/common/http';
import {environment} from '../environments/environment' // import environment


// ici crée un objet de type OktaAuth, et on initialise avec 
// les données que nous avont recuperé depuis Okta 
const oktaAuth = new OktaAuth(environment.oktaAuth); 

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    OktaAuthModule.forRoot({ oktaAuth }), //ici on configure avec notre objet OktaAuth 
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

NB : Assurez-vous que les informations que vous renseignez ici correspondent à celles fournies dans la configuration sur Okta précédemment.

Maintenant, nous allons mettre en place les fonctions de connexion et de déconnexion. Pour ce faire, j'ai ajouté trois composants : Home, Navbar et Login. Le composant Home contient la page d'accueil où l'utilisateur arrive une fois authentifié. Sur cette page, un bouton de déconnexion est placé dans la Navbar (qui apparaît sur la page Home) pour permettre la déconnexion et envoyer une demande de révocation du jeton d'authentification. Le composant Login contient le bouton d'authentification.

Afin de sécuriser les routes de notre application, nous avons utilisé des guards. Vous pouvez les générer en utilisant la commande suivante :

```bash
ng generate guard gurads/OktaAuth
```

Les codes d'implémentations de ces différents composants :

***navbar.component.html***

```javascript
<nav class="navbar navbar-expand-lg navbar-light bg-light">
     <div class="container">
          <a class="navbar-brand" href="#">Gui-App</a>
          <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNav"
               aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
               <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarNav">
               <ul class="navbar-nav">
               </ul>
          </div>
          <ng-container *ngIf="(isAuthenticated$ | async) === false">
               <button class="btn btn-sm btn-outline-primary ml-auto" (click)="signIn()"> Sign in </button>
          </ng-container>
          
         
     </div>
</nav>
```

navbar.component.ts

```javascript

import { DOCUMENT } from "@angular/common";
import { Component, Inject, OnInit } from "@angular/core";
import { Router } from "@angular/router";
import { OktaAuthStateService, OKTA_AUTH } from "@okta/okta-angular";
import { AuthState, OktaAuth } from "@okta/okta-auth-js";
import { filter, map, Observable } from "rxjs";

@Component({
  selector: 'app-navbar',
  templateUrl: './navbar.component.html',
  styleUrls: ['./navbar.component.css']
})
export class NavbarComponent implements OnInit {
  public isAuthenticated$!: Observable<boolean>;
  constructor(
    private _router: Router,
    private _oktaStateService: OktaAuthStateService,
    @Inject(OKTA_AUTH) private _oktaAuth: OktaAuth
  ) {}

  public ngOnInit(): void {
    this.isAuthenticated$ = this._oktaStateService.authState$.pipe(
      filter((s: AuthState) => !!s),
      map((s: AuthState) => s.isAuthenticated ?? false)
    );
  }

  public async signIn(): Promise<void> {
    try {
      await this._oktaAuth.signInWithRedirect();
    } catch (error) {
      console.error("Error during sign-in:", error);
    }
  }


}
```

home.component.html

```javascript
<div class="">
    <app-navbar></app-navbar>
    <div class="login-page">
        <div class="login-cover"></div>
        <div class="login-content">
            <p><span class="text-primary">{{userName}} ({{userEmail}})</span> Bienvenue dans votre espace securisé</p>
        </div>
    </div>
</div>
```

home.component.ts

```javascript
import { DOCUMENT } from "@angular/common";
import { Component, Inject, OnInit } from "@angular/core";
import { Router } from "@angular/router";
import { OktaAuthStateService, OKTA_AUTH } from "@okta/okta-angular";
import { AuthState, OktaAuth } from "@okta/okta-auth-js";
import { filter, map, Observable } from "rxjs";

@Component({
  selector: "app-home",
  templateUrl: "./home.component.html",
  styleUrls: ["./home.component.css"],
})
export class HomeComponent implements OnInit {
  userName: string = "";
  userEmail: string = "";

  constructor(
    private _router: Router,
    private _oktaStateService: OktaAuthStateService,
    @Inject(OKTA_AUTH) private _oktaAuth: OktaAuth,
    @Inject(DOCUMENT) private _document: Document
  ) {}

   ngOnInit(): void {
    this._oktaStateService.authState$
      .pipe(
        filter((authState) => !!authState && !!authState.isAuthenticated), // Vérification de nullité
        map((authState) => authState?.idToken?.claims) // Récupère les informations de l'utilisateur depuis le jeton
      )
      .subscribe((userClaims) => {
        if (userClaims) {
          // Les informations de l'utilisateur, telles que le nom et le prénom, sont stockées dans les "claims" du jeton
          this.userName = `${userClaims.name}`;
          this.userEmail = `${userClaims.email}`;
        }
      });
  }
}
```

login.component.html

```javascript
<div class="login-page">
     <div class="login-cover"></div>
     <div class="login-content">
          <h1>Bienvenue sur Gui-App</h1>
          <p>Veuillez vous authentifier pour accéder à votre espace sécurisé.</p>
          <button class="  btn btn-sm btn-outline-primary ml-auto" (click)="signIn()">S'authentifier</button>
     </div>
</div>
```

login.component.ts

```javascript
import { DOCUMENT } from "@angular/common";
import { Component, Inject, OnInit } from "@angular/core";
import { Router } from "@angular/router";
import { OktaAuthStateService, OKTA_AUTH } from "@okta/okta-angular";
import { AuthState, OktaAuth } from "@okta/okta-auth-js";
import { filter, map, Observable } from "rxjs";

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css']
})
export class LoginComponent implements OnInit {

  constructor(
    private _router: Router,
    private _oktaStateService: OktaAuthStateService,
    @Inject(OKTA_AUTH) private _oktaAuth: OktaAuth,
    @Inject(DOCUMENT) private _document: Document
  ) {}

  ngOnInit(): void {
    // vavigate vers home si user connecter
    this._oktaStateService.authState$.subscribe((authState) => {
      if (authState.isAuthenticated) {
        this._router.navigate(["/home"]);
      }
    });
  }

    public async signIn(): Promise<void> {
    try {
      await this._oktaAuth.signInWithRedirect();
    } catch (error) {
      console.error("Error during sign-in:", error);
    }
  }

}
```

okta-auth.guard.ts

```javascript
import { Injectable, Inject, OnInit  } from "@angular/core";
import {
  ActivatedRouteSnapshot,
  CanActivate,
  RouterStateSnapshot,
  UrlTree,
  Router,
} from "@angular/router";
import { AuthState, OktaAuth } from "@okta/okta-auth-js";

import { OktaAuthStateService, OKTA_AUTH } from "@okta/okta-angular";
import { filter, map, Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class OktaAuthGuard implements CanActivate {

  //  public isAuthenticated$!: Observable<boolean>;

  constructor(
    private _router: Router,
     private _oktaStateService: OktaAuthStateService,
    @Inject(OKTA_AUTH) private _oktaAuth: OktaAuth
  ) {}


  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
    return this._oktaStateService.authState$.pipe(
      filter((s) => !!s),
      map((authState) => {
        if (!authState.isAuthenticated) {
          // Si l'utilisateur n'est pas authentifié, redirigez-le vers la page de connexion.
          this._router.navigate(["/login"]);
          return true;
        }
        return true;
      })
    );
  }
}
```

app-routing.module.ts

```javascript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { OktaCallbackComponent } from '@okta/okta-angular';
import { HomeComponent } from './components/home/home.component';
import { LoginComponent } from './components/login/login.component';
import { OktaAuthGuard } from './gurads/okta-auth.guard';

const routes: Routes = [
  { path: 'login/callback', component: OktaCallbackComponent }, 
  { path: 'home', component: HomeComponent, canActivate: [OktaAuthGuard]},
  { path: 'login', component: LoginComponent},
  { path: '', redirectTo: '/home', pathMatch: 'full' },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Si tout se passe bien et que vous démarrez votre application Angular, vous verrez une page qui ressemble à ceci, si vous avez ajouté les styles CSS et Bootstrap :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697321516154/f3881cdd-cd5e-41ce-a3d0-35f83df9b131.png align="center")

Lorsque vous cliquez sur le bouton "S'authentifier", vous serez redirigés vers la page d'authentification Okta, qui ressemble à ceci :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697321670756/c9ccca0f-a571-4db2-ae1a-69ee232ceb5d.png align="center")

Dans mon cas, il existe déjà un compte actif associé à l'adresse e-mail [martin.luc@test.com](mailto:martin.luc@test.com), auquel nous avons précédemment assigné, via un groupe Okta, à notre "intégration d'application" que nous avons configurée. Assurez-vous de renseigner un utilisateur actif qui est également assigné à votre intégration d'application.

Lorsque vous cliquez sur "Suivant", si c'est la première fois que l'utilisateur se connecte, il lui sera demandé de configurer Okta Verify, qui est une application d'authentification à deux facteurs (MFA) d'Okta. Okta Verify ajoute une couche de sécurité supplémentaire en générant un code de vérification temporaire à usage unique sur le smartphone de l'utilisateur. Ce code doit être saisi pour vérifier l'identité de l'utilisateur, renforçant ainsi la sécurité du processus de connexion. MFA (Multi-Factor Authentication) est une méthode efficace pour réduire les risques d'accès non autorisés en exigeant plus qu'un simple mot de passe pour l'authentification.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697322002271/50a002c6-6aef-42ce-923c-76a502fc5a80.png align="center")

Une fois la configuration d'Okta Verify terminée, l'utilisateur peut désormais accéder de manière sécurisée à votre application. Il doit saisir son mot de passe une seule fois, lors de la première connexion. Cela démontre l'avantage du SSO (Single Sign-On), car lors de ses futures connexions, l'utilisateur sera simplement confirmé via l'application Okta Verify sur son téléphone. Il sera ensuite redirigé vers la page d'accueil de votre application, comme illustré ci-dessous  :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697322348993/93b2fa32-525d-4cd1-8fe8-5fe24ebb2cb2.png align="center")

Pour tester la déconnexion, vous pouvez cliquer sur le bouton "Se déconnecter". Le jeton d'authentification fourni par Okta sera révoqué, et vous serez déconnecté, puis redirigé vers la page de connexion.

## Conclusion 

En conclusion, l'intégration de l'authentification Okta dans votre application Angular constitue une étape cruciale pour garantir un accès sécurisé à vos utilisateurs. Avec Okta, vous bénéficiez d'une solution de gestion de l'identité et de l'accès qui gère la sécurité, l'authentification et l'autorisation (nous aborderons la partie autorisation dans un prochain article), vous permettant ainsi de vous concentrer sur le développement des fonctionnalités essentielles de votre application. Cela vous offre l'opportunité de fournir une expérience utilisateur fluide tout en renforçant la protection de vos ressources, tout en réduisant la charge liée à la gestion des identités. En exploitant les capacités d'Okta, vous pouvez garantir un accès sécurisé et offrir une tranquillité d'esprit à vos utilisateurs, tout en restant axé sur l'innovation et la croissance de votre application Angular.