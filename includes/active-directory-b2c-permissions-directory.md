---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 04/05/2021
ms.author: mimart
ms.openlocfilehash: 5a2382146cd8b85b8eef54b924a206dda7107b0f
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382474"
---
#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/) 

1. En **Administrar**, seleccione **Permisos de API**.
1. En **Permisos configurados**, seleccione **Agregar un permiso**.
1. Seleccione la pestaña **API de Microsoft** y, a continuación, seleccione **Microsoft Graph**.
1. Seleccione **Permisos de aplicación**.
1. Expanda el grupo de permisos adecuado y active la casilla del permiso que quiera conceder a la aplicación de administración. Por ejemplo:
    * **User** > **User.ReadWrite.All**: para escenarios de migración de usuarios o administración de usuarios.
    * **Grupo**  >  de **Group.ReadWrite.All**: para crear grupos, leer y actualizar pertenencias a grupos y eliminar grupos.
    * **AuditLog** > **AuditLog.Read.All**: para leer los registros de auditoría del directorio.
    * **Policy** > **Policy.ReadWrite.TrustFramework**: para escenarios de integración continua o entrega continua (CI/CD). Por ejemplo, la implementación de directivas personalizadas con Azure Pipelines.
1. Seleccione **Agregar permisos**. Como se indicó, espere unos minutos antes de continuar con el paso siguiente.
1. Seleccione **Conceder consentimiento de administrador para (el nombre de inquilino)** .
1. Si no tiene la sesión iniciada con una cuenta de administrador global, inicie sesión con una cuenta en el inquilino de Azure AD B2C que tenga asignado al menos el rol *Administrador de aplicaciones en la nube* y, a continuación, seleccione **Conceder consentimiento de administrador para (nombre del inquilino)** .
1. Seleccione **Actualizar** y, a continuación, compruebe que aparece "Concedido para..." bajo **Estado**. Los permisos pueden tardar unos minutos en propagarse.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. En la página de información general **Aplicación registrada**, seleccione **Configuración**.
1. En **Acceso de API**, seleccione **Permisos necesarios**.
1. Seleccione **Microsoft Graph**.
1. En **Permisos de la aplicación**, active la casilla del permiso que quiere conceder a la aplicación de administración. Por ejemplo:
    * **Leer todos los datos de registro de auditoría**: seleccione este permiso para leer los registros de auditoría del directorio.
    * **Leer y escribir datos de directorio**: seleccione este permiso para escenarios de migración o administración de usuarios.
    * **Leer y escribir en las directivas del marco de confianza de su organización**: seleccione este permiso para escenarios de integración continua o entrega continua (CI/CD). Por ejemplo, la implementación de directivas personalizadas con Azure Pipelines.
1. Seleccione **Guardar**.
1. Haga clic en **Conceder permisos** y, a continuación, haga clic en **Sí**. Los permisos pueden tardar unos minutos en propagarse por completo.
