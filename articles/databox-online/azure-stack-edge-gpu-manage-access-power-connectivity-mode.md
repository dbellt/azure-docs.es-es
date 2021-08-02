---
title: Acceso, encendido y modo de conectividad en el dispositivo de Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo administrar el acceso, el encendido y el modo de conectividad del dispositivo de Azure Stack Edge Pro con GPU que ayuda a transferir datos a Azure
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 21cb737cf6168f44c1e71815d90139a04ba1d37b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697084"
---
# <a name="manage-access-power-and-connectivity-mode-for-your-azure-stack-edge-pro-gpu"></a>Administración del acceso, el encendido y el modo de conectividad de Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe cómo administrar el acceso, el encendido y el modo de conectividad del dispositivo de Azure Stack Edge Pro con GPU. Estas operaciones se realizan mediante la interfaz de usuario web local o en Azure Portal.

Este artículo se aplica a los dispositivos de Azure Stack Edge Pro GPU, Azure Stack Edge Pro R y Azure Stack Edge Mini R.


En este artículo aprenderá a:

> [!div class="checklist"]
> * Administración del acceso al dispositivo
> * Administración del acceso a recursos
> * Administración del modo de conectividad
> * Administración del encendido


## <a name="manage-device-access"></a>Administración del acceso al dispositivo

El acceso al dispositivo de Azure Stack Edge Pro se controla mediante una contraseña del dispositivo. Puede cambiar la contraseña mediante la interfaz de usuario web local. También puede restablecer la contraseña del dispositivo en Azure Portal.

El acceso a los datos en los discos del dispositivo también se controla mediante las claves de cifrado en reposo. 

Para acceder al dispositivo, abra una sesión remota de PowerShell a través de HTTP o HTTPS desde la interfaz de usuario web local del dispositivo.

### <a name="change-device-password"></a>Cambiar la contraseña del dispositivo

Siga estos pasos en la interfaz de usuario local para cambiar la contraseña del dispositivo.

1. En la interfaz de usuario web local, vaya a **Mantenimiento > Contraseña**.
2. Escriba la contraseña actual y, a continuación, la nueva contraseña. La contraseña proporcionada debe contener entre 8 y 16 caracteres. La contraseña debe contener tres de los siguientes caracteres: caracteres en mayúsculas, minúsculas, números y caracteres especiales. Confirme la nueva contraseña.

    ![Cambiar contraseña](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/change-password-1.png)

3. Seleccione **Cambiar contraseña**.
 
### <a name="reset-device-password"></a>Restablecimiento de la contraseña del dispositivo

El flujo de trabajo de restablecimiento no requiere que el usuario recupere la contraseña antigua y es útil cuando se pierde la contraseña. Este flujo de trabajo se realiza en Azure Portal.

1. En Azure Portal, vaya a **Información general > Restablecer la contraseña del administrador**.

    ![Captura de pantalla que muestra el dispositivo con la contraseña de restablecimiento del dispositivo seleccionada.](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/reset-password-1.png)


2. Escriba la nueva contraseña y confírmela. La contraseña proporcionada debe contener entre 8 y 16 caracteres. La contraseña debe contener tres de los siguientes caracteres: caracteres en mayúsculas, minúsculas, números y caracteres especiales. Seleccione **Restablecer**.

    ![Restablecer contraseña 2](media/azure-stack-edge-manage-access-power-connectivity-mode/reset-password-2.png)

### <a name="manage-access-to-device-data"></a>Administración del acceso a los datos del dispositivo

En el caso de los dispositivos Azure Stack Edge Pro R y Azure Stack Edge Mini R, el acceso a los datos del dispositivo se controla mediante el uso de claves de cifrado en reposo en las unidades del dispositivo. Después de haber configurado correctamente el dispositivo para realizar el cifrado en reposo, la opción para rotar las claves de cifrado en reposo estará disponible en la interfaz de usuario local del dispositivo. 

Esta operación le permite cambiar las claves de los volúmenes de BitLocker `HcsData` y `HcsInternal` y todas las unidades de cifrado automático del dispositivo.

Siga estos pasos para rotar las claves de cifrado en reposo.

1. En la interfaz de usuario web local del dispositivo, vaya a la página **Comenzar**. En el icono **Seguridad**, seleccione la opción **Cifrado en reposo: rotar claves**. Esta opción solo está disponible después de haber configurado correctamente las claves de cifrado en reposo.

    ![Seleccione la opción Rotar claves para el cifrado en reposo en la página Comenzar.](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/rotate-encryption-keys-1.png)

1. Puede usar sus propias claves de BitLocker o usar las claves que haya generado el sistema.  

    Si quiere usar su propia clave, escriba una cadena codificada de tipo Base-64 de 32 caracteres. La entrada es similar a la que se proporcionaría al configurar el cifrado en reposo por primera vez.

    ![Use su propia clave de cifrado en reposo](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/rotate-encryption-keys-2.png)

    También puede optar por usar una clave que haya generado el sistema.

    ![Uso de la clave de cifrado en reposo generada por el sistema](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/rotate-encryption-keys-3.png)

1. Seleccione **Aplicar**. Verá cómo rotan los protectores de clave.

    ![Aplicación de la nueva clave de cifrado en reposo](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/rotate-encryption-keys-4.png)

1. Cuando se le pida que descargue y guarde el archivo de la clave, seleccione **Descargar y continuar**. 

    ![Descarga del archivo de clave y continuación del proceso](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/rotate-encryption-keys-5.png)

    Guarde el archivo de clave `.json` en una ubicación segura. Recuerde que puede usar este archivo para que le sea más fácil recuperar los datos del dispositivo en un futuro.

    ![Captura de pantalla que muestra el cuadro de diálogo de restablecimiento de la contraseña del dispositivo.](media/azure-stack-edge-manage-access-power-connectivity-mode/reset-password-2.png)

## <a name="enable-device-access-via-remote-powershell-over-http"></a>Habilitación del acceso a dispositivos mediante PowerShell remoto a través de HTTP

Puede abrir una sesión remota de PowerShell en el dispositivo a través de HTTP o HTTPS. De forma predeterminada, puede acceder al dispositivo mediante una sesión de PowerShell a través de HTTPS. Sin embargo, en redes de confianza, es aceptable habilitar PowerShell remoto a través de HTTP.

Siga estos pasos en la interfaz de usuario local para habilitar PowerShell remoto a través de HTTP:

1. En la interfaz de usuario local del dispositivo, vaya a **Configuración** en la esquina superior derecha de la página.
1. Seleccione **Habilitar** para poder abrir una sesión remota de PowerShell para el dispositivo a través de HTTP. Esta configuración solo debe habilitarse en redes de confianza.

    ![Captura de pantalla que muestra la opción de habilitar PowerShell remoto a través de HTTP.](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/enable-remote-powershell-http-1.png)

1. Seleccione **Aplicar**.

Ahora puede conectarse a la interfaz de PowerShell del dispositivo a través de HTTP. Para más información, consulte [Conexión a la interfaz de PowerShell del dispositivo](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).

## <a name="manage-resource-access"></a>Administración del acceso a recursos

Para crear el recurso de Azure Stack Edge/Data Box Gateway, IoT Hub y Azure Storage, necesita permisos de colaborador o superiores en el nivel del grupo de recursos. También debe registrar los proveedores de recursos correspondientes. Para las operaciones que implican credenciales y claves de activación, también se necesitan permisos para Microsoft Graph API. Estos se describen en las secciones siguientes. 

### <a name="manage-microsoft-graph-api-permissions"></a>Administración de permisos para Microsoft Graph API

Cuando genere la clave de activación para el dispositivo de Azure Stack Edge Pro o realice cualquier operación que requiera credenciales, necesitará permisos para Graph API de Azure Active Directory. Las operaciones que podrían necesitar credenciales son:

-  Crear un recurso compartido con una cuenta de almacenamiento asociada.
-  Crear un usuario que puede tener acceso a los recursos compartidos en el dispositivo.

Debe tener acceso de `User` en el inquilino de Active Directory, ya que necesita la capacidad de `Read all directory objects`. No puede ser un usuario invitado, ya que estos no tienen permisos para `Read all directory objects`. Si es un invitado, se producirán errores en operaciones como la generación de una clave de activación, la creación de un recurso compartido en el dispositivo de Azure Stack Edge Pro, la creación de un usuario, la configuración de un rol de proceso perimetral o el restablecimiento de la contraseña del dispositivo.

Para más información sobre cómo proporcionar a los usuarios acceso a Microsoft Graph API, consulte [Referencia de permisos de Microsoft Graph](/graph/permissions-reference).

### <a name="register-resource-providers"></a>Registro de proveedores de recursos

Para aprovisionar un recurso de Azure (en el modelo de Azure Resource Manager), necesita un proveedor de recursos que sea compatible con la creación de ese recurso. Por ejemplo, para aprovisionar una máquina virtual, debe tener un proveedor de recursos de "Microsoft.Compute" disponible en la suscripción.
 
Los proveedores de recursos se registran en el nivel de la suscripción. De forma predeterminada, cualquier nueva suscripción a Azure se registra previamente con una lista de proveedores de recursos usados con frecuencia. El proveedor de recursos de "Microsoft.DataBoxEdge" no está incluido en esta lista.

No necesita conceder permisos de acceso en el nivel de suscripción para que los usuarios puedan crear recursos como "Microsoft.DataBoxEdge" dentro de los grupos de recursos en los que tienen derechos de propietario, siempre y cuando los proveedores de estos recursos ya estén registrados.

Antes de que intente crear cualquier recurso, asegúrese de que el proveedor de recursos está registrado en la suscripción. Si el proveedor de recursos no está registrado, deberá asegurarse de que el usuario que crea el nuevo recurso tiene derechos suficientes para registrar al proveedor de recursos necesarios en el nivel de suscripción. Si aún no lo ha hecho, verá el siguiente error:

*La suscripción \<Subscription name> no tiene permisos para registrar los siguientes proveedores de recursos: Microsoft.DataBoxEdge.*


Para obtener una lista de los proveedores de recursos registrados en la suscripción actual, ejecute el siguiente comando:

```PowerShell
Get-AzResourceProvider -ListAvailable |where {$_.Registrationstate -eq "Registered"}
```

Es necesario que `Microsoft.DataBoxEdge` esté registrado en el dispositivo de Azure Stack Edge Pro. Para registrar `Microsoft.DataBoxEdge`, el administrador de suscripciones debe ejecutar el siguiente comando:

```PowerShell
Register-AzResourceProvider -ProviderNamespace Microsoft.DataBoxEdge
```

Para obtener más información sobre cómo registrar un proveedor de recursos, consulte [Resolución de errores del registro del proveedor de recursos](../azure-resource-manager/templates/error-register-resource-provider.md).

## <a name="manage-connectivity-mode"></a>Administración del modo de conectividad

Aparte del modo predeterminado totalmente conectado, el dispositivo también se puede ejecutar en modo parcialmente conectado o en modo totalmente desconectado. A continuación se describe cada uno de estos modos:

- **Totalmente conectado**: este es el modo normal predeterminado en el que opera el dispositivo. En este modo están habilitadas la carga y descarga de datos en la nube. Puede usar Azure Portal o la interfaz de usuario web local para administrar el dispositivo.

- **Parcialmente desconectado**: en este modo, el dispositivo no puede cargar ni descargar datos de recurso compartido; sin embargo, se pueden administrar mediante Azure Portal.

    Este modo se suele usar cuando se encuentra en una red de satélite de uso medido y el objetivo es minimizar el consumo de ancho de banda de red. Todavía puede producirse un consumo de red mínimo por las operaciones de supervisión del dispositivo.

- **Desconectado**: en este modo, el dispositivo está completamente desconectado de la nube y las cargas y descargas de la nube están deshabilitadas. Solo se puede administrar el dispositivo mediante la interfaz de usuario web local.

    Este modo se suele usar cuando desea desconectar el dispositivo.

Para cambiar el modo del dispositivo, siga estos pasos:

1. En la interfaz de usuario web local del dispositivo, vaya a **Configuración > Nube**.
2. En la lista desplegable, seleccione el modo con el que quiere operar el dispositivo. Puede seleccionar entre las opciones **totalmente conectado**, **parcialmente conectado**, y **totalmente desconectado**. Para ejecutar el dispositivo en modo parcialmente desconectado, habilite **Administración en Azure Portal**.

 
## <a name="manage-power"></a>Administración del encendido

Puede apagar o reiniciar el dispositivo físico mediante la interfaz de usuario web local. Se recomienda que antes de reiniciarlo, desconecte los recursos compartidos del servidor de datos y, luego, el dispositivo. Esta acción minimizará la posibilidad de daños en los datos.

1. En la interfaz de usuario web local, vaya a **Mantenimiento > Potencia**.
2. Seleccione **Apagar** o **Reiniciar** según lo que se quiera hacer.

    ![Configuración de encendido](media/azure-stack-edge-gpu-manage-access-power-connectivity-mode/shut-down-restart-1.png)

3. Cuando se le pida confirmación, seleccione **Sí** para continuar.

> [!NOTE]
> Si apaga el dispositivo físico, deberá presionar el botón de encendido en el dispositivo para encenderlo.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [administrar los recursos compartidos](azure-stack-edge-manage-shares.md).
