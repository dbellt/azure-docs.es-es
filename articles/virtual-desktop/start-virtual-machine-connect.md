---
title: Iniciar máquina virtual al establecer la conexión - Azure
description: Cómo configurar la característica Iniciar máquina virtual al establecer la conexión.
author: Heidilohr
ms.topic: how-to
ms.date: 05/21/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 7e4ca9a6cfc87844bf74131b145c19aecd964554
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111752142"
---
# <a name="start-virtual-machine-on-connect-preview"></a>Iniciar máquina virtual al establecer la conexión (versión preliminar)

> [!IMPORTANT]
> La característica Iniciar VM al establecer la conexión se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La característica Iniciar máquina virtual (VM) al establecer conexión (versión preliminar) permite ahorrar costos al permitir que los usuarios finales activen sus máquinas virtuales solo cuando las necesiten. A continuación, puede desactivar las máquinas virtuales cuando no se necesiten.

>[!NOTE]
>Tenga en cuenta que Azure Virtual Desktop (clásico) no admite esta característica.

## <a name="requirements-and-limitations"></a>Limitaciones y requisitos

Puede habilitar la característica Iniciar máquina virtual al establecer la conexión para grupos de hosts personales o agrupados mediante PowerShell y Azure Portal.

Los siguientes clientes de Escritorio remoto admiten la característica Iniciar VM al establecer la conexión:

- [El cliente web](connect-web.md)
- [El cliente de Windows (versión 1.2748 o posterior)](connect-windows-7-10.md)
- [El cliente de Android (versión 10.0.10 o posterior)](connect-android.md)
- [El cliente de macOS (versión 10.6.4 o posterior)](connect-macos.md)

Puede buscar anuncios sobre actualizaciones y asistencia al cliente en el [foro Tech Community](https://aka.ms/wvdtc).

## <a name="create-a-custom-role-for-start-vm-on-connect"></a>Creación de un rol personalizado para Iniciar VM al establecer la conexión

Antes de poder configurar la característica Iniciar VM al establecer la conexión, deberá asignar a la VM un rol de RBAC (control de acceso basado en roles) personalizado. Este rol permitirá que Azure Virtual Desktop administre las VM de su suscripción. También puede usar este rol para activar VM, comprobar su estado y notificar información de diagnóstico. Si quiere saber más sobre lo que hace cada rol, eche un vistazo a [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Para usar Azure Portal para asignar un rol personalizado para Iniciar VM al establecer la conexión:

1. Abra Azure Portal y vaya a **Suscripciones**.

2. Vaya al **Control de acceso (IAM)** y seleccione **Agregar rol personalizado**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de un menú desplegable del botón Agregar en Control de acceso (IAM). "Agregar rol personalizado" está resaltado en rojo.](media/add-custom-role.png)

3. A continuación, asigne un nombre al rol personalizado y agregue una descripción. Le recomendamos que le asigne el nombre "start VM on connect".

4. En la pestaña **Permisos**, agregue los siguientes permisos a la suscripción a la que va a asignar el rol: 
 
   - Microsoft.Compute/virtualMachines/start/action
   - Microsoft.Compute/virtualMachines/read

5. Cuando haya finalizado, seleccione **Aceptar**.

Después de esto, deberá asignar el rol para conceder acceso a Azure Virtual Desktop.

Para asignar el rol personalizado:

1. En la pestaña **Control de acceso (IAM)** , seleccione **Agregar asignación de roles**.

2. Seleccione el rol que acaba de crear.

3. En la barra de búsqueda, escriba y seleccione **Azure Virtual Desktop**.

      >[!NOTE]
      >Es posible que vea dos aplicaciones si ha implementado Azure Virtual Desktop (clásico). Asigne el rol a las dos aplicaciones que vea.
      >
      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de la pestaña Control de acceso (IAM). En la barra de búsqueda, tanto Azure Virtual Desktop como Azure Virtual Desktop (clásico) están resaltados en rojo.](media/add-role-assignment.png)

### <a name="create-a-custom-role-with-a-json-file-template"></a>Creación de un rol personalizado con una plantilla de archivo JSON

Si usa un archivo JSON para crear el rol personalizado, en el ejemplo siguiente encontrará una plantilla básica que puede usar. Asegúrese de reemplazar el valor de identificador de suscripción por el identificador de suscripción al que quiere asignar el rol.

```json
{
    "properties": {
        "roleName": "start VM on connect",
        "description": "Friendly description.",
        "assignableScopes": [
            "/subscriptions/<SubscriptionID>"
        ],
        "permissions": [
            {
                "actions": [
                    "Microsoft.Compute/virtualMachines/start/action",
                    "Microsoft.Compute/virtualMachines/read"
                ],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ]
    }
}
```

## <a name="configure-the-start-vm-on-connect-feature"></a>Configuración de la característica Iniciar VM al establecer la conexión

Ahora que ha asignado el rol a la suscripción, es momento de configurar la característica Iniciar VM al establecer la conexión.

### <a name="deployment-considerations"></a>Consideraciones de la implementación 

Iniciar VM al establecer la conexión es una configuración de grupo de hosts. Si solo quiere que un cierto grupo de usuarios use esta característica, asegúrese de solo asignar el rol necesario a los usuarios que quiere agregar.

En el caso de los escritorios personales, la característica solo activará una máquina virtual existente que el servicio ya haya asignado o que asignará a un usuario. En un escenario de grupo de hosts agrupados, el servicio solo activará una máquina virtual cuando no haya ninguna activada. La característica solo activará máquinas virtuales adicionales cuando la primera máquina virtual alcance el límite de sesión.

>[!IMPORTANT]
> Solo puede configurar esta característica en grupos de hosts existentes. Esta característica no está disponible cuando se crea un nuevo grupo de hosts.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Para usar Azure Portal para configurar Start VM on Connect (Iniciar VM al conectar):

1. Abra el explorador y vaya a [Azure Portal](https://portal.azure.com).

2. Inicie sesión en Azure Portal y vaya a **Azure Virtual Desktop**.

3. Seleccione **Grupos de hosts** y, a continuación, vaya al grupo de hosts donde desea habilitar la configuración.

4. En el grupo de hosts, seleccione **Propiedades**. En **Start VM on connect** (Iniciar VM al conectar), seleccione **Sí** y, a continuación, seleccione **Guardar** para aplicar la configuración al instante.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la ventana Propiedades. La opción Start VM on connect (Iniciar VM al conectar) está resaltada en rojo.](media/properties-start-vm-on-connect.png)

### <a name="use-powershell"></a>Uso de PowerShell

Para configurar este valor con PowerShell, tiene que asegurarse de contar con los nombres del grupo de recursos y grupos de hosts que quiere configurar. También deberá instalar el [módulo de Azure PowerShell (versión 2.1.0 o posterior)](https://www.powershellgallery.com/packages/Az.DesktopVirtualization/2.1.0).

Para configurar Iniciar VM al establecer la conexión con PowerShell:

1. Abra una ventana de comando de PowerShell.

2. Ejecute el siguiente cmdlet para habilitar Iniciar VM al establecer la conexión:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -StartVMOnConnect:$true
    ```

3. Ejecute el siguiente cmdlet para deshabilitar Iniciar VM al establecer la conexión:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -StartVMOnConnect:$false
    ```

## <a name="user-experience"></a>Experiencia del usuario

En las sesiones típicas, el tiempo necesario para que un usuario se conecte a una VM desasignada aumenta debido a que la VM necesita tiempo para volver a activarse, como sucede al encender un equipo físico. El cliente de Escritorio remoto tiene un indicador que permite al usuario saber que el equipo se está encendiendo mientras se está conectando.

## <a name="troubleshooting"></a>Solución de problemas

Si la característica tiene algún problema, se recomienda usar la [característica de diagnóstico](diagnostics-log-analytics.md) de Azure Virtual Desktop para comprobar si hay errores. Si recibe un mensaje de error, asegúrese de prestar mucha atención al contenido del mensaje y de copiar el nombre del error para tenerlo como referencia.

También puede usar [Azure Monitor para Azure Virtual Desktop](azure-monitor.md) para obtener sugerencias sobre cómo resolver los problemas.

Si la máquina virtual no se activa, deberá comprobar el estado de la máquina virtual que intentó activar antes de cualquier otra acción.

## <a name="next-steps"></a>Pasos siguientes

Si tiene algún problema que no pueda resolver con la documentación de solución de problemas ni con la característica de diagnóstico, consulte las [Preguntas frecuentes sobre Iniciar VM al establecer la conexión](start-virtual-machine-connect-faq.md).
