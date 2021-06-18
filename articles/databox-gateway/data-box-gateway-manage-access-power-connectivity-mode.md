---
title: Acceso, encendido y modo de conectividad de los dispositivos Azure Data Box Gateway
description: Describe cómo administrar el acceso, encendido y modo de conectividad del dispositivo Azure Data Box Gateway que ayuda a transferir datos a Azure
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: how-to
ms.date: 10/14/2020
ms.author: alkohli
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bf46fb00b4326f873d0f76a2eb4cd2632e0b3cf0
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110706836"
---
# <a name="manage-access-power-and-connectivity-mode-for-your-azure-data-box-gateway"></a>Administración del acceso, encendido y modo de conectividad de Azure Data Box Gateway

En este artículo se describe cómo administrar el acceso, encendido y modo de conectividad de Azure Data Box Gateway. Estas operaciones se realizan mediante la interfaz de usuario web local o en Azure Portal. 

En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Administración del acceso al dispositivo
> * Administración del modo de conectividad
> * Administración del encendido

## <a name="manage-device-access"></a>Administración del acceso al dispositivo

El acceso al dispositivo Data Box Gateway se controla mediante una contraseña de dispositivo. Puede cambiar la contraseña mediante la interfaz de usuario web local. También puede restablecer la contraseña del dispositivo en Azure Portal.

### <a name="change-device-password"></a>Cambiar la contraseña del dispositivo

Siga estos pasos en la interfaz de usuario local para cambiar la contraseña del dispositivo.

1. En la interfaz de usuario web local, vaya a **Mantenimiento > Cambio de contraseña**.
2. Escriba la contraseña actual y, a continuación, la nueva contraseña. La contraseña proporcionada debe contener entre 8 y 16 caracteres. La contraseña debe contener tres de los siguientes caracteres: caracteres en mayúsculas, minúsculas, números y caracteres especiales. Confirme la nueva contraseña.

    ![Cambiar contraseña](media/data-box-gateway-manage-access-power-connectivity-mode/change-password-1.png)

3. Haga clic en **Cambiar contraseña**.
 
### <a name="reset-device-password"></a>Restablecimiento de la contraseña del dispositivo

El flujo de trabajo de restablecimiento no requiere que el usuario recupere la contraseña antigua y es útil cuando se pierde la contraseña. Este flujo de trabajo se realiza en Azure Portal.

1. En Azure Portal, vaya a **Información general > Restablecer la contraseña del administrador**.

    ![Restablecimiento de contraseña](media/data-box-gateway-manage-access-power-connectivity-mode/reset-password-1.png)

 
2. Escriba la nueva contraseña y confírmela. La contraseña proporcionada debe contener entre 8 y 16 caracteres. La contraseña debe contener tres de los siguientes caracteres: caracteres en mayúsculas, minúsculas, números y caracteres especiales. Haga clic en **Restablecer**.

    ![Restablecer contraseña 2](media/data-box-gateway-manage-access-power-connectivity-mode/reset-password-2.png)

## <a name="manage-resource-access"></a>Administración del acceso a recursos

Para crear un recurso de Azure Data Box Gateway, IoT Hub y Azure Storage, necesita permisos de colaborador o superiores en el nivel del grupo de recursos. También debe registrar los proveedores de recursos correspondientes. Para las operaciones que implican credenciales y claves de activación, también se necesitan permisos para la Graph API de Azure Active Directory. Estos se describen en las secciones siguientes.

### <a name="manage-microsoft-graph-api-permissions"></a>Administración de permisos para Microsoft Graph API

Cuando genere la clave de activación para el dispositivo o realice cualquier operación que requiera credenciales, necesitará permisos para Microsoft Graph API. Las operaciones que podrían necesitar credenciales son:

-  Crear un recurso compartido con una cuenta de almacenamiento asociada.
-  Crear un usuario que puede tener acceso a los recursos compartidos en el dispositivo.

Debe tener acceso de `User` en el inquilino de Active Directory para que pueda `Read all directory objects`. Un usuario invitado no tiene permiso para `Read all directory objects`. Si es un invitado, se producirá un error en operaciones como generación de una clave de activación, creación de un recurso compartido en un dispositivo y creación de un usuario.

Para más información sobre cómo proporcionar a los usuarios acceso a Microsoft Graph API, consulte [Referencia de permisos de Microsoft Graph](/graph/permissions-reference).

### <a name="register-resource-providers"></a>Registro de proveedores de recursos

Para aprovisionar un recurso de Azure (en el modelo de Azure Resource Manager), necesita un proveedor de recursos que sea compatible con la creación de ese recurso. Por ejemplo, para aprovisionar una máquina virtual, debe tener un proveedor de recursos de "Microsoft.Compute" disponible en la suscripción.
 
Los proveedores de recursos se registran en el nivel de la suscripción. De forma predeterminada, cualquier nueva suscripción a Azure se registra previamente con una lista de proveedores de recursos usados con frecuencia. El proveedor de recursos de "Microsoft.DataBoxEdge" no está incluido en esta lista.

No necesita conceder permisos de acceso en el nivel de suscripción para que los usuarios puedan crear recursos como "Microsoft.DataBoxEdge" dentro de los grupos de recursos en los que tienen derechos de propietario, siempre y cuando los proveedores de estos recursos ya estén registrados.

Antes de que intente crear cualquier recurso, asegúrese de que el proveedor de recursos esté registrado en la suscripción. Si el proveedor de recursos no está registrado, deberá asegurarse de que el usuario que crea el nuevo recurso tenga derechos suficientes para registrar al proveedor de recursos necesario en el nivel de suscripción. Si aún no lo ha hecho, verá el siguiente error:

*La suscripción \<Subscription name> no tiene permisos para registrar los siguientes proveedores de recursos: Microsoft.DataBoxEdge.*


Para obtener una lista de los proveedores de recursos registrados en la suscripción actual, ejecute el siguiente comando:

```PowerShell
Get-AzResourceProvider -ListAvailable |where {$_.Registrationstate -eq "Registered"}
```

Para un dispositivo de Data Box Gateway, se debe registrar `Microsoft.DataBoxEdge`. Para registrar `Microsoft.DataBoxEdge`, el administrador de suscripciones debe ejecutar el siguiente comando:

```PowerShell
Register-AzResourceProvider -ProviderNamespace Microsoft.DataBoxEdge
```

Para obtener más información sobre cómo registrar un proveedor de recursos, consulte [Resolución de errores del registro del proveedor de recursos](../azure-resource-manager/templates/error-register-resource-provider.md).

## <a name="manage-connectivity-mode"></a>Administración del modo de conectividad

Aparte del modo normal predeterminado, el dispositivo también se puede ejecutar en modo parcialmente desconectado o en modo desconectado:

- **Parcialmente desconectado**: en este modo, el dispositivo no puede cargar datos en los recursos compartidos. Sin embargo, se puede administrar mediante Azure Portal.

    Este modo se suele usar para minimizar el consumo de ancho de banda de red cuando se encuentra en una red de satélite de uso medido. Todavía puede producirse un consumo de red mínimo por las operaciones de supervisión del dispositivo.

- **Desconectado**: en este modo, el dispositivo está completamente desconectado de la nube, y las cargas y descargas de la nube están deshabilitadas. Solo se puede administrar el dispositivo mediante la interfaz de usuario web local.

    Este modo se suele usar cuando desea desconectar el dispositivo.

Para cambiar el modo del dispositivo, siga estos pasos:

1. En la interfaz de usuario web local del dispositivo, vaya a **Configuración > Configuración de la nube**.
2. Deshabilite la opción **Carga y descarga de la nube**.
3. Para ejecutar el dispositivo en modo parcialmente desconectado, habilite **Administración en Azure Portal**.

    ![Modo de conectividad](media/data-box-gateway-manage-access-power-connectivity-mode/connectivity-mode-1.png)
 
4. Para ejecutar el dispositivo en modo desconectado, deshabilite **Administración en Azure Portal**. Ahora solo se puede administrar el dispositivo mediante la interfaz de usuario web local.

    ![Modo de conectividad 2](media/data-box-gateway-manage-access-power-connectivity-mode/connectivity-mode-2.png)

## <a name="manage-power"></a>Administración del encendido

Puede apagar o reiniciar el dispositivo virtual mediante la interfaz de usuario web local. Se recomienda que, antes de reiniciar el dispositivo, desconecte los recursos compartidos del host y, luego, el dispositivo. Esta acción minimizará la posibilidad de daños en los datos.

1. En la interfaz de usuario web local, vaya a **Mantenimiento > Configuración de encendido**.
2. Haga clic en **Apagar** o **Reiniciar** según lo que se desee.

    ![Configuración de encendido](media/data-box-gateway-manage-access-power-connectivity-mode/shut-down-restart-1.png)

3. Cuando se le pida confirmación, haga clic en **Sí** para continuar.

> [!NOTE]
> Si apaga el dispositivo virtual, deberá iniciar el dispositivo mediante la administración del hipervisor.
