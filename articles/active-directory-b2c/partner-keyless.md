---
title: Tutorial para configurar Keyless con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Tutorial para configurar Keyless con Azure Active Directory B2C para la autenticación sin contraseña
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 1/17/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 006bcc6c0cffe6f7c08b16057325a499e44f06be
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257915"
---
# <a name="tutorial-configure-keyless-with-azure-active-directory-b2c"></a>Tutorial: Configuración de Keyless con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure Active Directory (AD) B2C con [Keyless](https://keyless.io/). Con Azure AD B2C como proveedor de identidades, puede integrar Keyless con cualquiera de las aplicaciones de sus clientes para proporcionar una verdadera autenticación sin contraseña para los usuarios.

La solución **Keyless Zero-Knowledge Biometric (ZKB™)** de Keyless ofrece una autenticación multifactor sin contraseña que elimina los fraudes, la suplantación de identidad (phishing) y la reutilización de credenciales, todo ello a la vez que mejora la experiencia del cliente y protege su privacidad.

## <a name="pre-requisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md). El inquilino debe estar vinculado a la suscripción a Azure.

- Un inquilino en la nube de Keyless; obtenga una [cuenta de prueba](https://keyless.io/go) gratuita.

- La aplicación Keyless Authenticator instalada en el dispositivo del usuario.

## <a name="scenario-description"></a>Descripción del escenario

La integración de Keyless incluye los siguientes componentes:

- Azure AD B2C: el servidor de autorización, responsable de comprobar las credenciales del usuario, al que también se le conoce como proveedor de identidades.

- Aplicaciones web y móviles: las aplicaciones web o móviles que decida proteger con Keyless y Azure AD B2C.

- Aplicación móvil de Keyless: esta se usará para la autenticación en las aplicaciones habilitadas para Azure AD B2C.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Imagen que muestra el diagrama de la arquitectura de Keyless](./media/partner-keyless/keyless-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. Los usuarios inician sesión o se registran y escriben su nombre de usuario.
| 2. | La aplicación envía los atributos de usuario a Azure AD B2C para la verificación de identificación.
| 3. | Azure AD B2C recopila los atributos de usuario y los envía a Keyless para autenticar al usuario a través de la aplicación móvil de Keyless.
| 4. | Keyless envía una notificación push al dispositivo móvil registrado del usuario para una autenticación que protege la privacidad en forma de un examen biométrico facial.
| 5. | Una vez que el usuario responda a la notificación push, se le concederá o denegará el acceso a la aplicación del cliente en función de los resultados de la comprobación.

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

### <a name="add-a-new-identity-provider"></a>Incorporación de un nuevo proveedor de identidades

Para agregar un nuevo proveedor de identidades, siga estos pasos:

1. Inicie sesión en **[Azure Portal](https://portal.azure.com/#home)** como administrador global del inquilino de Azure AD B2C.

2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.

3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.

4. Vaya a **Panel** > **Azure Active Directory B2C** >  **Proveedores de identidad**.

5. Seleccione **Proveedores de identidades**.

6. Seleccione **Agregar**.

### <a name="configure-an-identity-provider"></a>Configuración de un proveedor de identidades

Para configurar un proveedor de identidades, siga estos pasos:

1. Seleccione **Tipo de proveedor de identidades** > **OpenID Connect (versión preliminar)** .
2. Rellene el formulario para configurar el proveedor de identidades:

   |Propiedad | Value |
   |:-----| :-----------|
   | Nombre   | Keyless |
   | URL de metadatos | Inserte el URI de la aplicación de autenticación de Keyless, seguido de la ruta de acceso específica, como "https://keyless.auth/.well-known/openid-configuration". |
   | Secreto del cliente | Secreto asociado a la instancia de autenticación de Keyless, no es el mismo que el configurado anteriormente. Inserte una cadena compleja de su elección. Este secreto se usará más adelante en la configuración del contenedor de Keyless.|
   | Id. de cliente | Id. del cliente. Este id. se usará más adelante en la configuración del contenedor de Keyless.|
   | Ámbito | openid |
   | Tipo de respuesta | ID_token |
   | Modo de respuesta | form_post|

3. Seleccione **Aceptar**.

4. Seleccione **Asignar las notificaciones de este proveedor de identidades**.

5. Rellene el formulario para asignar el proveedor de identidades:

   |Propiedad | Value |
   |:-----| :-----------|
   | UserID    | Desde la suscripción. |
   | Nombre para mostrar | Desde la suscripción. |
   | Modo de respuesta | Desde la suscripción. |

6. Seleccione **Guardar** para completar la configuración del nuevo proveedor de identidades de Open ID Connect (OIDC).

### <a name="create-a-user-flow-policy"></a>Creación de una directiva de flujo de usuario

Ahora debería ver Keyless como un nuevo proveedor de identidades de Open ID Connect enumerado entre los proveedores de identidades de B2C.

1. En el inquilino de Azure AD B2C, en **Directivas**, seleccione **Flujos de usuario**.

2. Seleccione **Nuevo** flujo de usuario.

3. Seleccione **Registrarse e iniciar sesión**, seleccione una **versión** y, a continuación, seleccione **Crear**.

4. Escriba un **nombre** para la directiva.

5. En la sección Proveedores de identidades seleccione el proveedor de identidades de Keyless recién creado.

6. Configure los parámetros del flujo de usuario. Inserte un nombre y seleccione el proveedor de identidades que creó. También puede agregar una dirección de correo electrónico. En este caso, Azure no redirigirá el procedimiento de inicio de sesión directamente a Keyless, sino que mostrará una pantalla en la que el usuario podrá elegir la opción que quiere usar.

7. Deje el campo **Autenticación multifactor** con el valor que tiene.

8. Seleccione **Aplicar directivas de acceso condicional**.

9. En **atributos de usuario y notificaciones de token**, seleccione **Dirección de correo electrónico** en la opción Recopilar atributo. Puede agregar todos los atributos que Azure Active Directory puede recopilar sobre el usuario junto con las notificaciones que Azure AD B2C puede devolver a la aplicación cliente.

10. Seleccione **Crear**.

11. Tras un proceso de creación correcto, seleccione el nuevo **Flujo de usuario**.

12. En el panel izquierdo, seleccione **Notificaciones de aplicación**. En opciones, marque la casilla **correo electrónico** y seleccione **Guardar**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione Identity Experience Framework.

2. Seleccione el valor de SignUpSignIn que creó anteriormente.

3. Seleccione Ejecutar flujo de usuario y elija la configuración:

   a. Aplicación: seleccione la aplicación registrada (el ejemplo es JWT).

   b. URL de respuesta: seleccione la dirección URL de redireccionamiento.

   c. Seleccione Ejecutar flujo de usuario.

4. Recorra el flujo de registro y cree una cuenta.

5. Se llamará a Keyless durante el flujo, una vez creado el atributo de usuario. Si el flujo está incompleto, compruebe que el usuario no esté guardado en el directorio.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)