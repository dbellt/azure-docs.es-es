---
title: Omisión de la eliminación de usuarios fuera del ámbito en el aprovisionamiento de aplicaciones de Azure Active Directory
description: Aprenda a reemplazar el comportamiento predeterminado de desaprovisionamiento de los usuarios que estén fuera de ámbito en Azure Active Directory.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: how-to
ms.workload: identity
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 37196dafd094f997bc5fe2ea165cefb9099562a0
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784888"
---
# <a name="skip-deletion-of-user-accounts-that-go-out-of-scope-in-azure-active-directory"></a>Omisión de la eliminación de cuentas de usuario que están fuera de ámbito en Azure Active Directory

De forma predeterminada, el motor de aprovisionamiento de Azure AD elimina o deshabilita los usuarios que están fuera del ámbito. Sin embargo, en determinados escenarios, como el aprovisionamiento entrante de usuarios de Workday a AD, es posible que este comportamiento no sea el esperado y puede que desee reemplazarlo.  

En este artículo se describe cómo usar Microsoft Graph API y el explorador de Microsoft Graph API para establecer la marca ***SkipOutOfScopeDeletions*** que controla el procesamiento de las cuentas que quedan fuera del ámbito. 
* Si ***SkipOutOfScopeDeletions*** se establece en 0 (false), las cuentas que queden fuera del ámbito se deshabilitarán en el destino.
* Si ***SkipOutOfScopeDeletions** _ se establece en 1 (true), las cuentas que queden fuera del ámbito no se deshabilitarán en el destino. Esta marca se establece en el nivel de la _aplicación de aprovisionamiento* y se puede configurar mediante Graph API. 

Como esta configuración se utiliza profusamente con la aplicación de *aprovisionamiento de usuarios de Workday a Active Directory*, los pasos siguientes incluyen capturas de pantallas de la aplicación Workday. Sin embargo, la configuración también se puede usar con *todas las demás aplicaciones*, como ServiceNow, Salesforce y Dropbox.

## <a name="step-1-retrieve-your-provisioning-app-service-principal-id-object-id"></a>Paso 1: Recuperar el identificador de la entidad de servicio de la aplicación de aprovisionamiento (identificador de objeto)

1. Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección Propiedades de la aplicación de aprovisionamiento. Por ejemplo, si desea exportar la asignación de la *aplicación de aprovisionamiento de usuarios de Workday a AD*, vaya a la sección Propiedades de dicha aplicación. 
1. En la sección Propiedades de la aplicación de aprovisionamiento, copie el valor GUID asociado con el campo *Id. de objeto*. Este valor también se conoce como el elemento **ServicePrincipalId** de la aplicación y se usará en las operaciones del Probador de Graph.

   ![Identificador de la entidad de servicio de la aplicación de Workday](./media/skip-out-of-scope-deletions/wd_export_01.png)

## <a name="step-2-sign-into-microsoft-graph-explorer"></a>Paso 2: Iniciar sesión en el Probador de Graph de Microsoft

1. Iniciar el [Probador de Graph de Microsoft](https://developer.microsoft.com/graph/graph-explorer)
1. Haga clic en el botón "Iniciar sesión con Microsoft" e inicie sesión con las credenciales de administrador de la aplicación o de administrador global de Azure AD.

    ![Inicio de sesión en Graph](./media/skip-out-of-scope-deletions/wd_export_02.png)

1. Una vez haya iniciado sesión correctamente, verá los detalles de la cuenta de usuario en el panel izquierdo.

## <a name="step-3-get-existing-app-credentials-and-connectivity-details"></a>Paso 3: Obtener las credenciales y los detalles de conectividad de la aplicación existente

En el Probador de Graph de Microsoft, ejecute la siguiente consulta GET; para ello, reemplace [servicePrincipalId] con la propiedad **ServicePrincipalId** extraído del [paso 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/secrets
```

   ![Consulta de trabajo GET](./media/skip-out-of-scope-deletions/skip-03.png)

Copie la respuesta en un archivo de texto. Será similar al texto JSON que se muestra a continuación, y los valores resaltados son los específicos de su implementación. Agregue las líneas resaltadas en verde al final y actualice la contraseña de conexión de Workday resaltada en azul. 

   ![Respuesta del trabajo GET](./media/skip-out-of-scope-deletions/skip-04.png)

Este es el bloque JSON que se agrega a la asignación. 

```json
        {
            "key": "SkipOutOfScopeDeletions",
            "value": "True"
        }
```

## <a name="step-4-update-the-secrets-endpoint-with-the-skipoutofscopedeletions-flag"></a>Paso 4: Actualizar el punto de conexión de secretos con la marca SkipOutOfScopeDeletions

En Graph Explorer, ejecute el siguiente comando para actualizar el punto de conexión de secretos con la marca ***SkipOutOfScopeDeletions***. 

En la dirección URL siguiente, reemplace [servicePrincipalId] por el valor de **ServicePrincipalId** extraído del [paso 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id). 

```http
   PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/secrets
```
Copie el texto actualizado del paso 3 en "Request Body" y establezca el encabezado "Content-Type" en "application/json" en "Request Headers". 

   ![Solicitud PUT](./media/skip-out-of-scope-deletions/skip-05.png)

Haga clic en "Ejecutar consulta". 

Debería aparecer la siguiente salida: "Success – Status Code 204" (Correcto: código de estado 204). 

   ![Respuesta PUT](./media/skip-out-of-scope-deletions/skip-06.png)

## <a name="step-5-verify-that-out-of-scope-users-dont-get-disabled"></a>Paso 5: Comprobar que los usuarios fuera de ámbito no se deshabilitan

Para probar los resultados de esta marca en el comportamiento esperado, actualice de las reglas de ámbito para omitir un usuario específico. En el ejemplo siguiente, se excluye el empleado con el identificador 21173 (que antes estaba dentro del ámbito) agregando una nueva regla de ámbito: 

   ![Captura de pantalla que muestra la sección "Agregar filtro de ámbito" con un usuario de ejemplo resaltado.](./media/skip-out-of-scope-deletions/skip-07.png)

En el siguiente ciclo de aprovisionamiento, el servicio de aprovisionamiento de Azure AD identificará que el usuario 21173 ha quedado fuera del ámbito y, si la propiedad SkipOutOfScopeDeletions está habilitada, la regla de sincronización de dicho usuario mostrará un mensaje como el siguiente: 

   ![Ejemplo de ámbito](./media/skip-out-of-scope-deletions/skip-08.png)


