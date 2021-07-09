---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 05/26/2021
ms.author: mimart
ms.openlocfilehash: b4bcb2da8efdf4943c758aaa2ad739e220cfcc8d
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110574604"
---
Para registrar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](../articles/active-directory-b2c/app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba un **Nombre** para la aplicación. Por ejemplo, *nativeapp1*.
1. En **Tipos de cuenta compatibles**, seleccione **Cuentas de cualquier directorio de la organización o cualquier proveedor de identidades**.
1. En **URI de redirección**, use la lista desplegable para seleccionar **Cliente público o nativo (móvil y escritorio)** .
1. Escriba un URI de redirección con un esquema único. Por ejemplo, `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`. Hay dos consideraciones importantes que se deben tener en cuenta al elegir un URI de redirección:
    * **Desarrollo** Para que se pueda usar para el desarrollo y las **aplicaciones de escritorio**, puede establecer el identificador URI de redireccionamiento en `http://localhost` y Azure AD B2C respetará todos los puertos de la solicitud. Si el URI registrado contiene un puerto, Azure AD B2C usará solo ese. Por ejemplo, si el URI de redirección registrado es `http://localhost`, el URI de redirección de la solicitud puede ser `http://localhost:<randomport>`. Si el URI de redirección registrado es `http://localhost:8080`, el URI de redirección de la solicitud debe ser `http://localhost:8080`.
    * **Único**: el esquema del identificador URI de redirección debe ser único para todas las aplicaciones. En el ejemplo `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`, `com.onmicrosoft.contosob2c.exampleapp` es el esquema. Se debe seguir este patrón. Si dos aplicaciones comparten el mismo esquema, el usuario tiene opción de elegir una aplicación. Si el usuario elige incorrectamente, se produce un error en el inicio de sesión.
    * **Completo**: el URI de redirección debe tener un esquema y una ruta de acceso. La ruta de acceso debe contener al menos una barra diagonal después del dominio. Por ejemplo, `//oauth/` sirve y `//oauth` produce un error. No incluya caracteres especiales en el URI, por ejemplo, caracteres de subrayado.
1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
2. Seleccione **Registrar**.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Aplicaciones (heredado)** y, a continuación, **Agregar**.
1. Escriba un nombre para la aplicación. Por ejemplo, *nativeapp1*.
1. En **Cliente nativo**, seleccione **Sí**.
1. Escriba un **URI de redirección personalizado** con un esquema único. Por ejemplo, `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`. Hay dos aspectos importantes que se deben tener en cuenta al elegir un URI de redirección:
    * **Único**: el esquema del identificador URI de redirección debe ser único para todas las aplicaciones. En el ejemplo `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`, `com.onmicrosoft.contosob2c.exampleapp` es el esquema. Se debe seguir este patrón. Si dos aplicaciones comparten el mismo esquema, el usuario tiene opción de elegir una aplicación. Si el usuario elige incorrectamente, se produce un error en el inicio de sesión.
    * **Completo**: el URI de redirección debe tener un esquema y una ruta de acceso. La ruta de acceso debe contener al menos una barra diagonal después del dominio. Por ejemplo, `//oauth/` sirve y `//oauth` produce un error. No incluya caracteres especiales en el URI, por ejemplo, caracteres de subrayado.
1. Seleccione **Crear**.
