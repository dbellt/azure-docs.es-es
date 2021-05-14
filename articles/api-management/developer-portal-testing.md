---
title: Prueba del portal para desarrolladores autohospedado
titleSuffix: Azure API Management
description: Obtenga información sobre cómo configurar pruebas unitarias y pruebas integrales para el portal de API Management autohospedado.
author: dlepow
ms.author: apimpm
ms.date: 03/25/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: ab6a7aa8fc4f11c34126415379294ef1e48fa286
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741380"
---
# <a name="test-the-self-hosted-developer-portal"></a>Prueba del portal para desarrolladores autohospedado

En este artículo se explica cómo configurar pruebas unitarias y pruebas integrales para el [portal autohospedado](developer-portal-self-host.md).

## <a name="unit-tests"></a>Pruebas unitarias

Una prueba unitaria es un enfoque para validar pequeños fragmentos de funcionalidad. Se hace de forma aislada de otras partes de la aplicación.

### <a name="example-scenario"></a>Escenario de ejemplo

En este escenario, está probando un control de entrada de contraseñas. Solo acepta contraseñas que contengan al menos:

- Una letra

- Un número

- Un carácter especial
 
Por lo tanto, la prueba para validar estos requisitos tiene el aspecto siguiente:

```typescript
const passwordInput = new PasswordInput();

passwordInput.value = "";
expect(passwordInput.isValid).to.equal(false);

passwordInput.value = "password";
expect(passwordInput.isValid).to.equal(false);

passwordInput.value = "p@ssw0rd";
expect(passwordInput.isValid.to.equal(true);
```
 
### <a name="project-structure"></a>Estructura del proyecto

Es habitual mantener una prueba unitaria junto al componente que se supone debe validar.

```console
component.ts
component.spec.ts
```

### <a name="mock-http-requests"></a>Simulación de solicitudes HTTP

Hay casos en los que espera que un componente haga solicitudes HTTP. El componente tiene que reaccionar correctamente a diferentes tipos de respuestas. Para simular respuestas HTTP específicas, use `MockHttpClient`. Implementa la interfaz `HttpClient` usada por muchos otros componentes del proyecto.

```typescript
const httpClient = new MockHttpClient();

httpClient.mock()
    .get("/users/jane")
    .reply(200, {
        firstName: "Jane",
        lastName: "Doe"
    });
```

## <a name="end-to-end-tests"></a>Prueba integrales

Una prueba integral ejecuta un escenario de usuario determinado que sigue los pasos exactos que espera que lleve a cabo el usuario. En una aplicación web como el portal para desarrolladores de Azure API Management, el usuario se desplaza por el contenido y selecciona opciones para lograr determinados resultados. 

Para replicar la navegación del usuario, puede usar bibliotecas auxiliares de manipulación del explorador, como [Puppeteer](https://github.com/puppeteer/puppeteer). Esto le permite simular acciones del usuario y automatizar escenarios supuestos. Puppeteer también toma automáticamente capturas de pantalla de páginas o componentes en cualquier fase de la prueba. Más adelante, compare los resultados con los anteriores para detectar desviaciones y posibles regresiones.

### <a name="example-scenario"></a>Escenario de ejemplo

En este escenario, tiene que validar un flujo de inicio de sesión de usuario. Para este escenario, se deberían seguir los pasos a continuación:

1. Abra el explorador y vaya a la página de inicio de sesión.

1. Escriba la dirección de correo electrónico.

1. Escriba la contraseña.

1. Seleccione **Iniciar sesión**.

1. Compruebe que se redirija al usuario a la página principal.

1. Compruebe que la página incluye el elemento de menú **Perfil**. Es uno de los posibles indicadores de que ha iniciado sesión correctamente.

Para ejecutar la prueba automáticamente, cree un script con exactamente los mismos pasos:

```typescript
// 1. Open browser and navigate to the sign-in page.
const page = await browser.newPage();
await page.goto("https://contoso.com/signin");

// 2. Enter email.
await this.page.type("#email", "john.doe@contoso.com");

// 3. Enter password.
await this.page.type("#password", "p@s$w0rd");

// 4. Click Sign-in.
await this.page.click("#signin");

// 5. Verify that user got redirected to Home page.
expect(page.url()).to.equal("https://contoso.com");

// 6. Verify that the page includes the Profile menu item.
const profileMenuItem = await this.page.$("#profile");
expect(profileMenuItem).not.equals(null);
```

> [!NOTE]
> Cadenas como "#email", "#password" y "#signin" son selectores de tipo CSS que identifican elementos HTML en la página. Consulte la especificación [Selectors Level 3](https://www.w3.org/TR/selectors-3/) de W3C para obtener más información.

### <a name="ui-component-maps"></a>Asignación de componentes de UI

Los flujos de usuario suelen pasar por las mismas páginas o componentes. Un buen ejemplo es el menú principal del sitio web, que está presente en cada página. 

Cree una asignación de componentes de la interfaz de usuario para evitar configurar y actualizar los mismos selectores para cada prueba. Por ejemplo, podría reemplazar los pasos del 2 al 6 del ejemplo anterior por solo dos líneas:

```typescript
const signInWidget = new SigninBasicWidget(page);
await signInWidget.signInWithBasic({ email: "...", password: "..." });
```

### <a name="test-configuration"></a>Configuración de prueba

Algunos escenarios pueden requerir datos o configuraciones creados previamente. Por ejemplo, es posible que tenga que automatizar el inicio de sesión de los usuarios con cuentas de redes sociales. Es difícil crear esos datos de forma rápida o sencilla.

Para ello, podría agregar un archivo de configuración especial al escenario de prueba. Los scripts de prueba pueden obtener del archivo los datos necesarios. En función de la canalización de compilación y prueba, las pruebas pueden extraer los secretos de un almacén seguro con nombre.

Este es un ejemplo de un archivo `validate.config.json` que se almacenaría en la carpeta `src` del proyecto.

```json
{
    "environment": "validation",
    "urls": {
        "home": "https://contoso.com",
        "signin": "https://contoso.com/signin",
        "signup": "https://contoso.com/signup/"
    },
    "signin": {
        "firstName": "John",
        "lastName": "Doe",
        "credentials": {
            "basic": {
                "email": "johndoe@contoso.com",
                "password": "< password >"
            },
            "aadB2C": {
                "email": "johndoe@contoso.com",
                "password": "< password >"
            }
        }
    },
    "signup": {
        "firstName": "John",
        "lastName": "Doe",
        "credentials": {
            "basic": {
                "email": "johndoe@contoso.com",
                "password": "< password >"
            }
        }
    }
}

```

### <a name="headless-vs-normal-tests"></a>Pruebas desatendidas frente a pruebas normales

Los exploradores modernos, como Chrome o Microsoft Edge, permiten ejecutar la automatización en modo desatendido y en modo normal. El explorador funciona sin una interfaz gráfica de usuario en el modo desatendido. Todavía lleva a cabo las mismas manipulaciones de página y Document Object Model (DOM). Normalmente, la interfaz de usuario del explorador no es necesaria en las canalizaciones de entrega. En ese caso, la ejecución de pruebas en modo desatendido es una excelente opción.

Al desarrollar un script de prueba, resulta útil ver lo que sucede exactamente en el explorador. Ese es un buen momento para usar el modo normal.

Para cambiar entre modos, cambie la opción `headless` en el archivo `constants.ts`. Se encuentra en la carpeta `tests` del proyecto:

```typescript
export const LaunchOptions = {
    headless: false
};
```

Otra opción útil es `slowMo`. Pausa la ejecución de la prueba entre cada acción:

```typescript
export const LaunchOptions = {
    slowMo: 200 // milliseconds
};
```

## <a name="run-tests"></a>Ejecutar pruebas

Hay dos maneras integradas de ejecutar pruebas en este proyecto:

**Comando npm**

```console
npm run test
```

**Explorador de pruebas**

La extensión Explorador de pruebas para VS Code (por ejemplo, [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter)) tiene una interfaz de usuario cómoda y una opción para ejecutar pruebas automáticamente en cada cambio del código fuente:

:::image type="content" source="media/developer-portal-testing/visual-studio-code-test-explorer.png" alt-text="Captura de pantalla del Explorador de pruebas de Visual Studio Code":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)

- [Autohospedaje del portal para desarrolladores](developer-portal-self-host.md)
