---
title: Tutorial para configurar Saviynt con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Tutorial para configurar Azure Active Directory B2C con Saviynt para la integración entre aplicaciones a fin de optimizar la modernización de TI y promover una mejor seguridad, gobernanza y cumplimiento. 
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/16/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: fe5a2114a99b7867ac5e51b6bf9ec542ad47ca13
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257898"
---
# <a name="tutorial-for-configuring-saviynt-with-azure-active-directory-b2c"></a>Tutorial para configurar Saviynt con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure Active Directory (AD) B2C con [Saviynt](https://saviynt.com/integrations/azure-ad/for-b2c/). La plataforma de administración de seguridad de Saviynt proporciona la visibilidad, la seguridad y la gobernanza que las empresas actuales necesitan, en una única plataforma unificada. Saviynt incorpora el riesgo y la gobernanza de aplicaciones, la administración de la infraestructura, la administración de cuentas con privilegios y el análisis de riesgos del cliente.

En este tutorial de ejemplo, configurará Saviynt para proporcionar una administración pormenorizada delegada, basada en el control de acceso para usuarios de Azure AD B2C. Saviynt realiza las siguientes comprobaciones para determinar si un usuario está autorizado para administrar usuarios de Azure AD B2C.

- Seguridad de nivel de característica para determinar si un usuario puede realizar una operación específica. Por ejemplo, Crear usuario, Actualizar usuario, Restablecer la contraseña de usuario, etc.

- Seguridad de nivel de campo para determinar si un usuario puede leer o escribir un atributo específico de otro usuario durante las operaciones de administración de usuarios. Por ejemplo, el agente del departamento de soporte técnico solo puede actualizar el número de teléfono y todos los demás atributos son de solo lectura.

- Seguridad de nivel de datos para determinar si un usuario puede realizar una operación específica en un usuario concreto. Por ejemplo, el administrador del departamento de soporte técnico de la región del Reino Unido solo puede administrar usuarios del Reino Unido.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md). El inquilino está vinculado a la suscripción de Azure.

- Una [suscripción](https://saviynt.com/contact-us/) de Saviynt.

## <a name="scenario-description"></a>Descripción del escenario

La integración de Saviynt incluye los siguientes componentes:

- [Azure AD B2C](https://azure.microsoft.com/services/active-directory/external-identities/b2c/): la identidad de negocio a cliente como servicio que permite el control personalizado de la forma en que los clientes se registran, inician sesión y administran sus perfiles.

- [Saviynt](https://saviynt.com/integrations/azure-ad/for-b2c/): la plataforma de gobernanza de identidades que proporciona una administración delegada pormenorizada para la administración del ciclo de vida de los usuarios y la gobernanza del acceso de los usuarios de Azure AD B2C.  

- [Microsoft Graph API](/graph/use-the-api): esta API proporciona las interfaces de Saviynt para administrar los usuarios de Azure AD B2C y su acceso en Azure AD B2C.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Imagen que muestra el diagrama de la arquitectura de Saviynt](./media/partner-saviynt/saviynt-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | Un administrador delegado inicia una operación de administración de un usuario de Azure AD B2C a través de Saviynt.
| 2. | Saviynt comprueba con su motor de autorización si el administrador delegado puede realizar la operación específica.
| 3. | El motor de autorización de Saviynt envía una respuesta de autorización correcta o incorrecta.
| 4. | Saviynt permite que el administrador delegado realice la operación requerida.
| 5. | Saviynt invoca Microsoft Graph API junto con los atributos de usuario para administrar el usuario en Azure AD B2C.
| 6. | A su vez, Microsoft Graph API creará, actualizará o eliminará el usuario en Azure AD B2C.
| 7. | Azure AD B2C enviará una respuesta de éxito o error.
| 8. | A continuación, Microsoft Graph API devolverá la respuesta a Saviynt.

## <a name="onboard-with-saviynt"></a>Incorporación con Saviynt

1. Para crear una cuenta de Saviynt, póngase en contacto con [Saviynt](https://saviynt.com/contact-us/).

2. Cree directivas de administración delegada y asigne usuarios como administradores delegados con varios roles.

## <a name="configure-azure-ad-b2c-with-saviynt"></a>Configuración de Azure AD B2C con Saviynt

### <a name="create-an-azure-ad-application-for-saviynt"></a>Creación de una aplicación de Azure AD para Saviynt

1. Inicie sesión en [Azure Portal](https://portal.azure.com/#home).

2. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

3. En Azure Portal, busque y seleccione **Azure AD B2C**.

4. Seleccione **Registros de aplicaciones** > **Nuevo registro**.

5. Escriba un Nombre para la aplicación. Por ejemplo, Saviynt y seleccione **Crear**.

6. Vaya a **Permisos de API** y seleccione **+ Agregar un permiso**.

7. Aparece la página Solicitud de permisos de API. Seleccione la pestaña **API de Microsoft** y seleccione **Microsoft Graph** como las API de Microsoft que se usan habitualmente.

8. Vaya a la página siguiente y seleccione **Permisos de la aplicación**.

9. Seleccione **Directorio** y seleccione las casillas **Directory.Read.All** y **Directory.ReadWrite.All**.

10. Seleccione **Agregar permisos**. Revise los permisos agregados.

11. Seleccione **Grant admin consent for Default Directory** (Conceder consentimiento de administrador para el directorio predeterminado) > **Guardar**.

12. Vaya a **Certificados y secretos** y seleccione **+ Add Client Secret** (+ Agregar secreto de cliente). Escriba la descripción del secreto de cliente, seleccione la opción de expiración y luego seleccione **Agregar**.

13. La clave de secreto se genera y se muestra en la sección de secreto de cliente.

    >[!NOTE]
    > Necesitará el secreto de cliente más adelante.

14. Vaya a **Información general** y obtenga el **Id. de cliente** y el **Id. de inquilino**.

15. Se necesitará el id. de inquilino, el id. de cliente y el secreto de cliente para completar la configuración en Saviynt.

### <a name="enable-saviynt-to-delete-users"></a>Habilitación de Saviynt para eliminar usuarios

En los pasos siguientes se explica cómo habilitar Saviynt para realizar operaciones de eliminación de usuario en Azure AD B2C.

>[!NOTE]
>[Evalúe el riesgo antes de conceder a los roles de administrador el acceso a una entidad de servicio.](../active-directory/develop/app-objects-and-service-principals.md)

1. Instale la versión más reciente del módulo de PowerShell de MSOnline en un servidor o estación de trabajo de Windows.

2. Conéctese al módulo AzureAD de PowerShell, use los siguientes comandos:

```powershell
Connect-msolservice #Enter Admin credentials of the Azure portal
$webApp = Get-MsolServicePrincipal –AppPrincipalId “<ClientId of Azure AD Application>”
Add-MsolRoleMember -RoleName "Company Administrator" -RoleMemberType ServicePrincipal -RoleMemberObjectId $webApp.ObjectId
```

## <a name="test-the-solution"></a>Probar la solución

Vaya al inquilino de la aplicación Saviynt y pruebe el caso de uso de administración del ciclo de vida de usuarios y la gobernanza de acceso.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)

- [Creación de una aplicación de API web](./add-web-api-application.md)