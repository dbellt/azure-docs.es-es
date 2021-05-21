---
title: 'Tutorial: Creación de una aplicación de Angular que usa la Plataforma de identidad de Microsoft para la autenticación mediante el flujo de código de autenticación | Azure'
titleSuffix: Microsoft identity platform
description: En este tutorial creará una aplicación de página única (SPA) de Angular que utiliza el flujo de código de autenticación y la Plataforma de identidad de Microsoft para iniciar sesión a los usuarios y obtener un token de acceso para llamar a Microsoft Graph API en su nombre.
services: active-directory
author: joarroyo
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 04/14/2021
ms.author: joarroyo
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: e9b99ed26276acd8c606e7d19f490867cb42b297
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109635332"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-an-angular-single-page-application-spa-using-auth-code-flow"></a>Tutorial: Inicio de sesión de usuarios y llamada a Microsoft Graph API desde una aplicación de página única (SPA) de Angular mediante el uso del flujo de código de autenticación

En este tutorial, creará una aplicación de página única (SPA) de Angular que inicia la sesión de los usuarios y llama a Microsoft Graph API mediante el flujo de código de autorización con PKCE. La aplicación de página única que cree usa la biblioteca de autenticación de Microsoft (MSAL) para Angular v2.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un proyecto de Angular con `npm`
> * Registrar la aplicación en Azure Portal
> * Agregar código para admitir el inicio y el cierre de sesión de usuario
> * Agregar código para llamar a Microsoft Graph API
> * Prueba de la aplicación

MSAL Angular v2 mejora MSALAngular v1 al ser compatible con el flujo de código de autorización en el explorador, en lugar de utilizar el flujo de concesión implícita. MSAL Angular v2 **no** admite el flujo implícito.

## <a name="prerequisites"></a>Prerrequisitos

* [Node.js](https://nodejs.org/en/download/) para ejecutar un servidor web local.
* [Visual Studio Code](https://code.visualstudio.com/download) u otro editor para modificar archivos de proyecto.

## <a name="how-the-sample-app-works"></a>Funcionamiento de la aplicación de ejemplo

:::image type="content" source="media/tutorial-v2-javascript-auth-code/diagram-01-auth-code-flow.png" alt-text="Diagrama que muestra el flujo de código de autorización para una aplicación de página única":::

La aplicación de ejemplo que se crea con este tutorial permite que una aplicación de página única de Angular haga consultas a Microsoft Graph API o a una API web que acepte los tokens que emite la Plataforma de identidad de Microsoft. Usa la Biblioteca de autenticación de Microsoft (MSAL) para Angular v2, un contenedor de la biblioteca principal de MSAL.js v2. MSAL Angular permite que las aplicaciones de Angular 9 y versiones posteriores autentiquen a usuarios empresariales mediante Azure Active Directory, a usuarios de cuentas de Microsoft y a usuarios de identidades de redes sociales como Facebook, Google y LinkedIn. La biblioteca también permite que las aplicaciones obtengan acceso a los servicios en la nube de Microsoft y a Microsoft Graph.

En este escenario, después de que un usuario inicia sesión, se solicita un token de acceso y se agrega a las solicitudes HTTP mediante el encabezado de autorización. MSAL controla la adquisición y la renovación de tokens.

### <a name="libraries"></a>Bibliotecas

En este tutorial se usan las siguientes bibliotecas:

|Biblioteca|Descripción|
|---|---|
|[MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular)|Biblioteca de autenticación de Microsoft para el contenedor de Angular de JavaScript|
|[Explorador MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)|Biblioteca de autenticación de Microsoft para el paquete del explorador de JavaScript v2 |

Puede encontrar el código fuente de todas las bibliotecas MSAL.js en el repositorio [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) de GitHub.

## <a name="create-your-project"></a>Creación del proyecto

Una vez que tenga [Node.js](https://nodejs.org/en/download/) instalado, abra una ventana de terminal y ejecute los siguientes comandos para generar una nueva aplicación de Angular:

```bash
npm install -g @angular/cli                         # Install the Angular CLI
ng new msal-angular-tutorial --routing=true --style=css --strict=false    # Generate a new Angular app
cd msal-angular-tutorial                            # Change to the app directory
npm install @angular/material @angular/cdk          # Install the Angular Material component library (optional, for UI)
npm install @azure/msal-browser @azure/msal-angular # Install MSAL Browser and MSAL Angular in your application
ng generate component home                          # To add a home page
ng generate component profile                       # To add a profile page
```

## <a name="register-your-application"></a>Registrar su aplicación

Siga las [instrucciones para registrar una aplicación de página única](./scenario-spa-app-registration.md) en Azure Portal.

En la página de **información general** del registro de la aplicación, anote el valor de **Id. de aplicación (cliente)** para su uso posterior.

Registre el valor de **URI de redirección** como **http://localhost:4200/** y escriba como 'SPA'.

## <a name="configure-the-application"></a>Configuración de la aplicación

1. En la carpeta *src/app*, edite *app.module.ts* y agregue `MsalModule` y `MsalInterceptor` a `imports`, así como la constante `isIE`: El código debe ser similar al siguiente:

    ```javascript
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';

    import { MsalModule } from '@azure/msal-angular';
    import { PublicClientApplication } from '@azure/msal-browser';

    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

    @NgModule({
      declarations: [
        AppComponent,
        HomeComponent,
        ProfileComponent
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        MsalModule.forRoot( new PublicClientApplication({
          auth: {
            clientId: 'Enter_the_Application_Id_here', // This is your client ID
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here', // This is your tenant ID
            redirectUri: 'Enter_the_Redirect_Uri_Here'// This is your redirect URI
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
          }
        }), null, null)
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```

    Reemplace estos valores:

    |Nombre del valor|Información|
    |---------|---------|
    |Enter_the_Application_Id_Here|En la página de **información general** del registro de la aplicación, corresponde al valor de **Id. de aplicación (cliente)** . |
    |Enter_the_Cloud_Instance_Id_Here|Esta es la instancia de la nube de Azure. En el caso de la nube principal o global de Azure, escriba **https://login.microsoftonline.com** . Para las nubes nacionales (por ejemplo, China) consulte [Nubes nacionales](./authentication-national-cloud.md).|
    |Enter_the_Tenant_Info_Here| Seleccione una de las siguientes opciones: Si la aplicación admite las *cuentas de este directorio organizativo*, reemplace este valor por el identificador del directorio (inquilino) o por el nombre del inquilino (por ejemplo, **contoso.microsoft.com**). Si la aplicación admite *cuentas en cualquier directorio organizativo*, reemplace este valor por **organizaciones**. Si la aplicación admite *cuentas en cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por **común**. Para restringir la compatibilidad a *Personal Microsoft accounts only* (Solo cuentas Microsoft personales), reemplace este valor por **consumidores**. |
    |Enter_the_Redirect_Uri_Here|Reemplace este valor por **http://localhost:4200** .|

    Para más información acerca de las opciones configurables disponibles, consulte [Inicializar aplicaciones cliente](msal-js-initializing-client-applications.md).

2. Agregue rutas a los componentes principales y de perfil en *src/app/app-routing.module.ts*. El código debe tener un aspecto parecido al siguiente:

    ```javascript
    import { NgModule } from '@angular/core';
    import { Routes, RouterModule } from '@angular/router';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';

    const routes: Routes = [
      {
        path: 'profile',
        component: ProfileComponent,
      },
      {
        path: '',
        component: HomeComponent
      },
    ];

    const isIframe = window !== window.parent && !window.opener;

    @NgModule({
      imports: [RouterModule.forRoot(routes, {
        initialNavigation: !isIframe ? 'enabled' : 'disabled' // Don't perform initial navigation in iframes
      })],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

## <a name="replace-base-ui"></a>Reemplazo de la interfaz de usuario base

1. Reemplace el código de marcador de posición de *src/app/app.component.html* por lo siguiente:

    ```HTML
    <mat-toolbar color="primary">
      <a class="title" href="/">{{ title }}</a>

      <div class="toolbar-spacer"></div>

      <a mat-button [routerLink]="['profile']">Profile</a>

      <button mat-raised-button *ngIf="!loginDisplay" (click)="login()">Login</button>

    </mat-toolbar>
    <div class="container">
      <!--This is to avoid reload during acquireTokenSilent() because of hidden iframe -->
      <router-outlet *ngIf="!isIframe"></router-outlet>
    </div>
    ```

2. Agregue módulos de material a *src/app/app.module.ts*. `AppModule` ahora debería presentar un aspecto similar a este:

    ```javascript
    import { BrowserModule } from '@angular/platform-browser';
    import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
    import { NgModule } from '@angular/core';

    import { MatButtonModule } from '@angular/material/button';
    import { MatToolbarModule } from '@angular/material/toolbar';
    import { MatListModule } from '@angular/material/list';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';

    import { MsalModule } from '@azure/msal-angular';
    import { PublicClientApplication } from '@azure/msal-browser';

    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

    @NgModule({
      declarations: [
        AppComponent,
        HomeComponent,
        ProfileComponent
      ],
      imports: [
        BrowserModule,
        BrowserAnimationsModule,
        AppRoutingModule,
        MatButtonModule,
        MatToolbarModule,
        MatListModule,
        MsalModule.forRoot( new PublicClientApplication({
          auth: {
            clientId: 'Enter_the_Application_Id_here',
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here',
            redirectUri: 'Enter_the_Redirect_Uri_Here'
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE, 
          }
        }), null, null)
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```

3. (OPCIONAL) Agregue CSS a *src/style.css*:

    ```css
    @import '~@angular/material/prebuilt-themes/deeppurple-amber.css';

    html, body { height: 100%; }
    body { margin: 0; font-family: Roboto, "Helvetica Neue", sans-serif; }
    .container { margin: 1%; }
    ```

4. (OPCIONAL) Agregue CSS a *src/app/app.component.css*:

    ```css
    .toolbar-spacer {
        flex: 1 1 auto;
      }
      
      a.title {
        color: white;
      }
    ```

## <a name="sign-in-a-user"></a>Inicio de sesión de un usuario

Agregue el código de las secciones siguientes para invocar el inicio de sesión mediante una ventana emergente o un redireccionamiento de marco completo: 

### <a name="sign-in-using-popups"></a>Inicie sesión mediante elementos emergentes

1. Cambie el código de *src/app/app.component.ts* a lo siguiente para iniciar sesión en un usuario mediante una ventana emergente:

    ```javascript
    import { MsalService } from '@azure/msal-angular';
    import { Component, OnInit } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      title = 'msal-angular-tutorial';
      isIframe = false;
      loginDisplay = false;

      constructor(private authService: MsalService) { }

      ngOnInit() {
        this.isIframe = window !== window.parent && !window.opener;
      }

      login() {
        this.authService.loginPopup()
          .subscribe({
            next: (result) => {
              console.log(result);
              this.setLoginDisplay();
            },
            error: (error) => console.log(error)
          });
      }

      setLoginDisplay() {
        this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
      }
    }
    ```

> [!NOTE]
> En el resto del tutorial se usa el método `loginRedirect` con Microsoft Internet Explorer, debido a un[problema conocido](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/internet-explorer.md) relacionado con la manera en que Internet Explorer trata las ventanas emergentes.

### <a name="sign-in-using-redirects"></a>Inicio de sesión mediante redireccionamientos

1. Actualice *src/app/app.module.ts* para arrancar `MsalRedirectComponent`. Se trata de un componente de redireccionamiento dedicado que controlará las redirecciones. Ahora el código debería ser similar a este:

    ```javascript
    import { BrowserModule } from '@angular/platform-browser';
    import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
    import { NgModule } from '@angular/core';

    import { MatButtonModule } from '@angular/material/button';
    import { MatToolbarModule } from '@angular/material/toolbar';
    import { MatListModule } from '@angular/material/list';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';

    import { MsalModule, MsalRedirectComponent } from '@azure/msal-angular'; // Updated import
    import { PublicClientApplication } from '@azure/msal-browser';

    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

    @NgModule({
      declarations: [
        AppComponent,
        HomeComponent,
        ProfileComponent
      ],
      imports: [
        BrowserModule,
        BrowserAnimationsModule,
        AppRoutingModule,
        MatButtonModule,
        MatToolbarModule,
        MatListModule,
        MsalModule.forRoot( new PublicClientApplication({
          auth: {
            clientId: 'Enter_the_Application_Id_here',
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here', 
            redirectUri: 'Enter_the_Redirect_Uri_Here'
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE,
          }
        }), null, null)
      ],
      providers: [],
      bootstrap: [AppComponent, MsalRedirectComponent] // MsalRedirectComponent bootstrapped here
    })
    export class AppModule { }
    ```

2. Agregue el selector `<app-redirect>` a *src/index.html*. Este selector lo usa `MsalRedirectComponent`. El archivo *src/index.html* debe ser similar a lo siguiente:

    ```HTML
    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <title>msal-angular-tutorial</title>
      <base href="/">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <link rel="icon" type="image/x-icon" href="favicon.ico">
    </head>
    <body>
      <app-root></app-root>
      <app-redirect></app-redirect>
    </body>
    </html>
    ```

3. Reemplace el código de *src/app/app.component.ts* por lo siguiente para iniciar sesión en un usuario mediante una redirección de marco completo:

    ```javascript
    import { MsalService } from '@azure/msal-angular';
    import { Component, OnInit } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      title = 'msal-angular-tutorial';
      isIframe = false;
      loginDisplay = false;

      constructor(private authService: MsalService) { }

      ngOnInit() {
        this.isIframe = window !== window.parent && !window.opener;
      }

      login() {
        this.authService.loginRedirect();
      }

      setLoginDisplay() {
        this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
      }
    }
    ```

4. Reemplace el código existente en *src/app/home/home.component.ts* para suscribirse al evento `LOGIN_SUCCESS`. Esto le permitirá acceder al resultado del inicio de sesión correcto con redireccionamiento. El código debe ser similar al siguiente:

    ```javascript
    import { Component, OnInit } from '@angular/core';
    import { MsalBroadcastService, MsalService } from '@azure/msal-angular';
    import { EventMessage, EventType, InteractionStatus } from '@azure/msal-browser';
    import { filter } from 'rxjs/operators';

    @Component({
      selector: 'app-home',
      templateUrl: './home.component.html',
      styleUrls: ['./home.component.css']
    })
    export class HomeComponent implements OnInit {
      constructor(private authService: MsalService, private msalBroadcastService: MsalBroadcastService) { }

      ngOnInit(): void {
        this.msalBroadcastService.msalSubject$
          .pipe(
            filter((msg: EventMessage) => msg.eventType === EventType.LOGIN_SUCCESS),
          )
          .subscribe((result: EventMessage) => {
            console.log(result);
          });
      }
    }
    ```

## <a name="conditional-rendering"></a>Representación condicional

Para representar cierta interfaz de usuario solo para los usuarios autenticados, los componentes tienen que suscribirse a `MsalBroadcastService` para ver si los usuarios han iniciado sesión y la interacción se ha completado.

1. Agregue `MsalBroadcastService` a *src/app/app.component.ts* y suscríbase al objeto observable `inProgress$` para comprobar si la interacción se ha completado y una cuenta ha iniciado sesión antes de representar la interfaz de usuario. Ahora el código debería ser similar a este:

    ```javascript
    import { Component, OnInit } from '@angular/core';
    import { MsalService, MsalBroadcastService } from '@azure/msal-angular';
    import { InteractionStatus } from '@azure/msal-browser';
    import { Subject } from 'rxjs';
    import { filter, takeUntil } from 'rxjs/operators';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      title = 'msal-angular-tutorial';
      isIframe = false;
      loginDisplay = false;
      private readonly _destroying$ = new Subject<void>();

      constructor(private broadcastService: MsalBroadcastService, private authService: MsalService) { }

      ngOnInit() {
        this.isIframe = window !== window.parent && !window.opener;

        this.broadcastService.inProgress$
        .pipe(
          filter((status: InteractionStatus) => status === InteractionStatus.None),
          takeUntil(this._destroying$)
        )
        .subscribe(() => {
          this.setLoginDisplay();
        })
      }

      login() {
        this.authService.loginRedirect();
      }

      setLoginDisplay() {
        this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
      }

      ngOnDestroy(): void {
        this._destroying$.next(undefined);
        this._destroying$.complete();
      }
    }
    ```

2. Actualice el código de *src/app/home/home.component.ts* para comprobar también si la interacción se completará antes de actualizar la interfaz de usuario. Ahora el código debería ser similar a este:

    ```javascript
    import { Component, OnInit } from '@angular/core';
    import { MsalBroadcastService, MsalService } from '@azure/msal-angular';
    import { EventMessage, EventType, InteractionStatus } from '@azure/msal-browser';
    import { filter } from 'rxjs/operators';

    @Component({
      selector: 'app-home',
      templateUrl: './home.component.html',
      styleUrls: ['./home.component.css']
    })
    export class HomeComponent implements OnInit {
      loginDisplay = false;

      constructor(private authService: MsalService, private msalBroadcastService: MsalBroadcastService) { }

      ngOnInit(): void {
        this.msalBroadcastService.msalSubject$
          .pipe(
            filter((msg: EventMessage) => msg.eventType === EventType.LOGIN_SUCCESS),
          )
          .subscribe((result: EventMessage) => {
            console.log(result);
          });
        
        this.msalBroadcastService.inProgress$
          .pipe(
            filter((status: InteractionStatus) => status === InteractionStatus.None)
          )
          .subscribe(() => {
            this.setLoginDisplay();
          })
      }
      
      setLoginDisplay() {
        this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
      }
    }
    ```

3. Reemplace el código de *src/app/home/home.component.html* por las siguientes pantallas condicionales:

    ```HTML
    <div *ngIf="!loginDisplay">
        <p>Please sign-in to see your profile information.</p>
    </div>

    <div *ngIf="loginDisplay">
        <p>Login successful!</p>
        <p>Request your profile information by clicking Profile above.</p>
    </div>
    ```

## <a name="guarding-routes"></a>Protección de rutas

### <a name="angular-guard"></a>Angular Guard

MSAL Angular proporciona `MsalGuard`, una clase que puede usar para proteger las rutas y requerir autenticación antes de acceder a la ruta protegida. Los pasos siguientes agregan `MsalGuard` a la ruta `Profile`. Proteger la ruta `Profile` significa que, incluso si un usuario no inicia sesión con el botón `Login`, si intenta acceder a la ruta `Profile` o hace clic en el botón `Profile`, `MsalGuard` solicitará al usuario que se autentique a través de un elemento emergente o un redireccionamiento antes de mostrar la página `Profile`.

`MsalGuard` es una clase de utilidad que puede usar para mejorar la experiencia del usuario, pero no debe basarse en ella para la seguridad. Los atacantes pueden moverse por las protecciones del lado cliente y debe asegurarse de que el servidor no devuelva ningún dato al que el usuario no deba acceder.

1. Agregue la clase `MsalGuard` como proveedor en la aplicación en *src/app/app.module.tsy* agregue las configuraciones de `MsalGuard`. Los ámbitos necesarios para adquirir tokens más adelante se pueden proporcionar en `authRequest` y el tipo de interacción para la protección se puede establecer en `Redirect` o `Popup`. El código debe tener un aspecto parecido al siguiente:

    ```javascript
    import { BrowserModule } from '@angular/platform-browser';
    import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
    import { NgModule } from '@angular/core';

    import { MatButtonModule } from '@angular/material/button';
    import { MatToolbarModule } from '@angular/material/toolbar';
    import { MatListModule } from '@angular/material/list';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';

    import { MsalModule, MsalRedirectComponent, MsalGuard } from '@azure/msal-angular'; // MsalGuard added to imports
    import { PublicClientApplication, InteractionType } from '@azure/msal-browser'; // InteractionType added to imports

    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

    @NgModule({
      declarations: [
        AppComponent,
        HomeComponent,
        ProfileComponent
      ],
      imports: [
        BrowserModule,
        BrowserAnimationsModule,
        AppRoutingModule,
        MatButtonModule,
        MatToolbarModule,
        MatListModule,
        MsalModule.forRoot( new PublicClientApplication({
          auth: {
            clientId: 'Enter_the_Application_Id_here',
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here', 
            redirectUri: 'Enter_the_Redirect_Uri_Here'
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE,
          }
        }), {
            interactionType: InteractionType.Redirect, // MSAL Guard Configuration
            authRequest: {
              scopes: ['user.read']
            }
        }, null)
      ],
      providers: [
        MsalGuard // MsalGuard added as provider here
      ],
      bootstrap: [AppComponent, MsalRedirectComponent]
    })
    export class AppModule { }
    ```

2. Establezca `MsalGuard` las rutas que desea proteger en *src/app/app-routing.module.ts*:

    ```javascript
    import { NgModule } from '@angular/core';
    import { Routes, RouterModule } from '@angular/router';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';
    import { MsalGuard } from '@azure/msal-angular';

    const routes: Routes = [
      {
        path: 'profile',
        component: ProfileComponent,
        canActivate: [MsalGuard]
      },
      {
        path: '',
        component: HomeComponent
      },
    ];

    const isIframe = window !== window.parent && !window.opener;

    @NgModule({
      imports: [RouterModule.forRoot(routes, {
        initialNavigation: !isIframe ? 'enabled' : 'disabled' // Don't perform initial navigation in iframes
      })],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

3. Ajuste las llamadas de inicio de sesión en *src/app/app.component.ts* para tener en cuenta el conjunto de `authRequest` en las configuraciones de protección. El código debe tener un aspecto parecido al siguiente.

    ```javascript
    import { Component, OnInit, Inject } from '@angular/core';
    import { MsalService, MsalBroadcastService, MSAL_GUARD_CONFIG, MsalGuardConfiguration } from '@azure/msal-angular';
    import { InteractionStatus, RedirectRequest } from '@azure/msal-browser';
    import { Subject } from 'rxjs';
    import { filter, takeUntil } from 'rxjs/operators';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      title = 'msal-angular-tutorial';
      isIframe = false;
      loginDisplay = false;
      private readonly _destroying$ = new Subject<void>();

      constructor(@Inject(MSAL_GUARD_CONFIG) private msalGuardConfig: MsalGuardConfiguration, private broadcastService: MsalBroadcastService, private authService: MsalService) { }

      ngOnInit() {
        this.isIframe = window !== window.parent && !window.opener;

        this.broadcastService.inProgress$
        .pipe(
          filter((status: InteractionStatus) => status === InteractionStatus.None),
          takeUntil(this._destroying$)
        )
        .subscribe(() => {
          this.setLoginDisplay();
        })
      }

      login() {
        if (this.msalGuardConfig.authRequest){
          this.authService.loginRedirect({...this.msalGuardConfig.authRequest} as RedirectRequest);
        } else {
          this.authService.loginRedirect();
        }
      }

      setLoginDisplay() {
        this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
      }

      ngOnDestroy(): void {
        this._destroying$.next(undefined);
        this._destroying$.complete();
      }
    }
    ```

## <a name="acquire-a-token"></a>Adquisición de un token

### <a name="angular-interceptor"></a>Interceptor de Angular

MSAL Angular proporciona una clase `Interceptor` que adquiere automáticamente tokens para las solicitudes salientes que usan el cliente `http` de Angular con los recursos protegidos conocidos.

1. Agregue la clase `Interceptor` como proveedor a la aplicación en *src/app/app.module.ts*, con sus configuraciones. Ahora el código debería ser similar a este:

    ```javascript
    import { BrowserModule } from '@angular/platform-browser';
    import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
    import { NgModule } from '@angular/core';
    import { HTTP_INTERCEPTORS, HttpClientModule } from "@angular/common/http"; // Import 

    import { MatButtonModule } from '@angular/material/button';
    import { MatToolbarModule } from '@angular/material/toolbar';
    import { MatListModule } from '@angular/material/list';

    import { AppRoutingModule } from './app-routing.module';
    import { AppComponent } from './app.component';
    import { HomeComponent } from './home/home.component';
    import { ProfileComponent } from './profile/profile.component';

    import { MsalModule, MsalRedirectComponent, MsalGuard, MsalInterceptor } from '@azure/msal-angular'; // Import MsalInterceptor
    import { InteractionType, PublicClientApplication } from '@azure/msal-browser';

    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

    @NgModule({
      declarations: [
        AppComponent,
        HomeComponent,
        ProfileComponent
      ],
      imports: [
        BrowserModule,
        BrowserAnimationsModule,
        AppRoutingModule,
        MatButtonModule,
        MatToolbarModule,
        MatListModule,
        HttpClientModule,
        MsalModule.forRoot( new PublicClientApplication({
          auth: {
            clientId: 'Enter_the_Application_Id_Here',
            authority: 'Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Info_Here',
            redirectUri: 'Enter_the_Redirect_Uri_Here',
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE,
          }
        }), {
          interactionType: InteractionType.Redirect,
          authRequest: {
            scopes: ['user.read']
            }
        }, {
          interactionType: InteractionType.Redirect, // MSAL Interceptor Configuration
          protectedResourceMap: new Map([ 
              ['Enter_the_Graph_Endpoint_Here/v1.0/me', ['user.read']]
          ])
        })
      ],
      providers: [
        {
          provide: HTTP_INTERCEPTORS,
          useClass: MsalInterceptor,
          multi: true
        },
        MsalGuard
      ],
      bootstrap: [AppComponent, MsalRedirectComponent]
    })
    export class AppModule { }

    ```

    Los recursos protegidos se proporcionan como `protectedResourceMap`. Las direcciones URL que proporciona en la colección `protectedResourceMap` distinguen mayúsculas de minúsculas. Para cada recurso, agregue los ámbitos cuya devolución se solicita en el token de acceso.

    Por ejemplo:

    * `["user.read"]` para Microsoft Graph
    * `["<Application ID URL>/scope"]` para las API web personalizadas (es decir, `api://<Application ID>/access_as_user`)
    
    Modifique los valores en la sección `protectedResourceMap`, como se describe aquí:

    |Nombre del valor| Información|
    |----------|------|
    |`Enter_the_Graph_Endpoint_Here`| La instancia de Microsoft Graph API con la que la aplicación debe comunicarse. En el caso del punto de conexión de Microsoft Graph API **global**, reemplace ambas instancias de esta cadena por `https://graph.microsoft.com`. Para obtener información sobre los puntos de conexión en las implementaciones de nubes **nacionales**, consulte [Implementaciones de nube nacionales](/graph/deployments) en la documentación de Microsoft Graph.|

2. Reemplace el código de *src/app/profile/profile.component.ts* para recuperar el perfil de un usuario por una solicitud HTTP:

    ```JavaScript
    import { Component, OnInit } from '@angular/core';
    import { HttpClient } from '@angular/common/http';

    const GRAPH_ENDPOINT = 'Enter_the_Graph_Endpoint_Here/v1.0/me';

    type ProfileType = {
      givenName?: string,
      surname?: string,
      userPrincipalName?: string,
      id?: string
    };

    @Component({
      selector: 'app-profile',
      templateUrl: './profile.component.html',
      styleUrls: ['./profile.component.css']
    })
    export class ProfileComponent implements OnInit {
      profile!: ProfileType;

      constructor(
        private http: HttpClient
      ) { }

      ngOnInit() {
        this.getProfile();
      }

      getProfile() {
        this.http.get(GRAPH_ENDPOINT)
          .subscribe(profile => {
            this.profile = profile;
          });
      }
    }
    ```

3. Reemplace la interfaz de usuario de *src/app/profile/profile.component.html* para mostrar la información del perfil: 

    ```HTML
    <div>
        <p><strong>First Name: </strong> {{profile?.givenName}}</p>
        <p><strong>Last Name: </strong> {{profile?.surname}}</p>
        <p><strong>Email: </strong> {{profile?.userPrincipalName}}</p>
        <p><strong>Id: </strong> {{profile?.id}}</p>
    </div>
    ```

## <a name="sign-out"></a>Cerrar sesión

Actualice el código de *src/app/app.component.html* para mostrar condicionalmente un botón `Logout`:

```HTML
<mat-toolbar color="primary">
  <a class="title" href="/">{{ title }}</a>

  <div class="toolbar-spacer"></div>

  <a mat-button [routerLink]="['profile']">Profile</a>

  <button mat-raised-button *ngIf="!loginDisplay" (click)="login()">Login</button>
  <button mat-raised-button *ngIf="loginDisplay" (click)="logout()">Logout</button>

</mat-toolbar>
<div class="container">
  <!--This is to avoid reload during acquireTokenSilent() because of hidden iframe -->
  <router-outlet *ngIf="!isIframe"></router-outlet>
</div>
```

### <a name="sign-out-using-redirects"></a>Cierre de sesión mediante redireccionamientos

Actualice el código de *src/app/app.component.ts* para cerrar la sesión de un usuario mediante redirecciones:

```javascript
import { Component, OnInit, Inject } from '@angular/core';
import { MsalService, MsalBroadcastService, MSAL_GUARD_CONFIG, MsalGuardConfiguration } from '@azure/msal-angular';
import { InteractionStatus, RedirectRequest } from '@azure/msal-browser';
import { Subject } from 'rxjs';
import { filter, takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  title = 'msal-angular-tutorial';
  isIframe = false;
  loginDisplay = false;
  private readonly _destroying$ = new Subject<void>();

  constructor(@Inject(MSAL_GUARD_CONFIG) private msalGuardConfig: MsalGuardConfiguration, private broadcastService: MsalBroadcastService, private authService: MsalService) { }

  ngOnInit() {
    this.isIframe = window !== window.parent && !window.opener;

    this.broadcastService.inProgress$
    .pipe(
      filter((status: InteractionStatus) => status === InteractionStatus.None),
      takeUntil(this._destroying$)
    )
    .subscribe(() => {
      this.setLoginDisplay();
    })
  }

  login() {
    if (this.msalGuardConfig.authRequest){
      this.authService.loginRedirect({...this.msalGuardConfig.authRequest} as RedirectRequest);
    } else {
      this.authService.loginRedirect();
    }
  }

  logout() { // Add log out function here
    this.authService.logoutRedirect({
      postLogoutRedirectUri: 'http://localhost:4200'
    });
  }

  setLoginDisplay() {
    this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
  }

  ngOnDestroy(): void {
    this._destroying$.next(undefined);
    this._destroying$.complete();
  }
}
```

### <a name="sign-out-using-popups"></a>Cierre de sesión con elementos emergentes

Actualice el código de *src/app/app.component.ts* para cerrar la sesión de un usuario mediante elementos emergentes:

```javascript
import { Component, OnInit, Inject } from '@angular/core';
import { MsalService, MsalBroadcastService, MSAL_GUARD_CONFIG, MsalGuardConfiguration } from '@azure/msal-angular';
import { InteractionStatus, PopupRequest } from '@azure/msal-browser';
import { Subject } from 'rxjs';
import { filter, takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  title = 'msal-angular-tutorial';
  isIframe = false;
  loginDisplay = false;
  private readonly _destroying$ = new Subject<void>();

  constructor(@Inject(MSAL_GUARD_CONFIG) private msalGuardConfig: MsalGuardConfiguration, private broadcastService: MsalBroadcastService, private authService: MsalService) { }

  ngOnInit() {
    this.isIframe = window !== window.parent && !window.opener;

    this.broadcastService.inProgress$
    .pipe(
      filter((status: InteractionStatus) => status === InteractionStatus.None),
      takeUntil(this._destroying$)
    )
    .subscribe(() => {
      this.setLoginDisplay();
    })
  }

  login() {
    if (this.msalGuardConfig.authRequest){
      this.authService.loginPopup({...this.msalGuardConfig.authRequest} as PopupRequest)
        .subscribe({
          next: (result) => {
            console.log(result);
            this.setLoginDisplay();
          },
          error: (error) => console.log(error)
        });
    } else {
      this.authService.loginPopup()
        .subscribe({
          next: (result) => {
            console.log(result);
            this.setLoginDisplay();
          },
          error: (error) => console.log(error)
        });
    }
  }

  logout() { // Add log out function here
    this.authService.logoutPopup({
      mainWindowRedirectUri: "/"
    });
  }

  setLoginDisplay() {
    this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
  }

  ngOnDestroy(): void {
    this._destroying$.next(undefined);
    this._destroying$.complete();
  }
}
```

## <a name="test-your-code"></a>Prueba del código

1.  Inicie el servidor web para que escuche el puerto; para ello, ejecute los siguientes comandos desde la carpeta de la aplicación en un símbolo del sistema de la línea de comandos:

    ```bash
    npm install
    npm start
    ```
1. En el explorador, escriba **http://localhost:4200** o **http://localhost:{port}** , donde *port* es el puerto en el que escucha el servidor web. Verá una página con el siguiente aspecto.

    :::image type="content" source="media/tutorial-v2-angular-auth-code/angular-01-not-signed-in.png" alt-text="Explorador web que muestra el cuadro de diálogo de inicio de sesión":::


### <a name="provide-consent-for-application-access"></a>Consentimiento para el acceso a la aplicación

La primera vez que inicie sesión en la aplicación, se le pedirá que le conceda acceso a su perfil y que le permita iniciar sesión:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-02-consent-dialog.png" alt-text="Cuadro de diálogo de contenido que se muestra en el explorador web":::

Si da su consentimiento a los permisos solicitados, la aplicación web muestra una página que indica que el inicio de sesión se ha realizado correctamente:

:::image type="content" source="media/tutorial-v2-angular-auth-code/angular-02-signed-in.png" alt-text="Resultados de un inicio de sesión correcto en el explorador web":::

### <a name="call-the-graph-api"></a>Llamada a Graph API

Después de iniciar sesión, seleccione **Perfil** para ver la información de perfil de usuario devuelta en la respuesta de la llamada a Microsoft Graph API:

:::image type="content" source="media/tutorial-v2-angular-auth-code/angular-03-profile-data.png" alt-text="Información de perfil de Microsoft Graph que se muestra en el explorador":::

## <a name="add-scopes-and-delegated-permissions"></a>Incorporación de ámbitos y permisos delegados

Microsoft Graph API requiere el ámbito _User.Read_ para leer el perfil de un usuario. El ámbito _User.Read_ se agrega automáticamente a cada registro de aplicación que cree en Azure Portal. Otras API de Microsoft Graph, así como las API personalizadas para el servidor back-end, pueden requerir ámbitos adicionales. Por ejemplo, Microsoft Graph API requiere el ámbito _Mail.Read_ para mostrar el correo electrónico del usuario.

A medida que se agregan ámbitos, puede que se pida a los usuarios que proporcionen consentimiento adicional para los ámbitos agregados.

>[!NOTE]
>Es posible que se pida al usuario algún consentimiento adicional a medida que aumente el número de ámbitos.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Puede profundizar más en el desarrollo de aplicaciones de página única (SPA) en la plataforma de identidad de Microsoft, en nuestra serie de artículos de varias partes.

> [!div class="nextstepaction"]
> [Escenario: Aplicación de una sola página](scenario-spa-overview.md)
