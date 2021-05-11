---
title: 'Personalización de las propiedades de RDP con PowerShell: Azure'
description: Cómo personalizar las propiedades de EDP para Windows Virtual Desktop con cmdlets de PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 10/09/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 06409903e77923176ac5fab4dd27bf918ea0d9d9
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108331809"
---
# <a name="customize-remote-desktop-protocol-rdp-properties-for-a-host-pool"></a>Personalización de las propiedades de Protocolo de escritorio remoto (RDP) para un grupo de hosts

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/customize-rdp-properties-2019.md).

La personalización de las propiedades de Protocolo de escritorio remoto (RDP) de un grupo de hosts, como el uso de varios monitores y la redirección de audio, permite ofrecer una experiencia óptima a los usuarios en función de sus necesidades. Las propiedades de RDP se pueden personalizar en Windows Virtual Desktop desde Azure Portal o mediante el parámetro *-CustomRdpProperty* del cmdlet **Update-AzWvdHostPool**.

En [Configuración admitida del archivo RDP](/windows-server/remote/remote-desktop-services/clients/rdp-files?context=%2fazure%2fvirtual-desktop%2fcontext%2fcontext) encontrará una lista completa de las propiedades admitidas y sus valores predeterminados.

## <a name="default-rdp-file-properties"></a>Propiedades predeterminadas del archivo RDP

Los archivos RDP tienen las siguientes propiedades de forma predeterminada:

|Propiedad de RDP|Para Escritorio y RemoteApp|
|---|---|
|Modo multimonitor|Disabled|
|Redireccionamiento de unidad habilitado|Unidades, portapapeles, impresoras, puertos COM, tarjetas inteligentes, dispositivos y usbdevicestore|
|Modo de audio remoto|Reproducción local|
|VideoPlayback|habilitado|
|EnableCredssp|habilitado|

>[!NOTE]
>El modo de supervisión múltiple solo es aplicable a los grupos de aplicaciones de escritorio y se omitirá para los grupos de aplicaciones de RemoteApp.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, siga las instrucciones que se indican en [Configuración del módulo de PowerShell para Windows Virtual Desktop](powershell-module.md) para configurar el módulo de PowerShell e iniciar sesión en Azure.

## <a name="configure-rdp-properties-in-the-azure-portal"></a>Configuración de propiedades de RDP en Azure Portal

Para configurar las propiedades de RDP en Azure Portal:

1. Inicie sesión en Azure en <https://portal.azure.com>.
2. Escriba **Windows Virtual Desktop** en la barra de búsqueda.
3. En Servicios, seleccione **Windows Virtual Desktop**.
4. En la página Windows Virtual Desktop, seleccione **grupos de hosts** en el menú del lado izquierdo de la pantalla.
5. Seleccione **el nombre del grupo de hosts** que quiera actualizar.
6. Seleccione **RDP Properties** (Propiedades de RDP) en el menú del lado izquierdo de la pantalla.
7. Establezca la propiedad que quiera.
   - También puede abrir la pestaña **Avanzado** y agregar las propiedades de RDP en un formato separado por punto y coma, como en los ejemplos de PowerShell de las secciones siguientes.
8. Cuando haya terminado, haga clic en **Guardar** para guardar los cambios.

En las secciones siguientes se explica cómo editar las propiedades personalizadas de RDP manualmente en PowerShell.

## <a name="add-or-edit-a-single-custom-rdp-property"></a>Adición y edición de una sola propiedad de RDP personalizada

Para agregar o editar una sola propiedad de RDP personalizada, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty <property>
```

Para comprobar si el cmdlet que acaba de ejecutar ha actualizado la propiedad, ejecute este cmdlet:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <customRDPpropertystring>
```

Por ejemplo, si estaba comprobando la propiedad "audiocapturemode" en un grupo de hosts denominado 0301HP, debería escribir este cmdlet:

```powershell
Get-AzWvdHostPool -ResourceGroupName 0301rg -Name 0301hp | format-list Name, CustomRdpProperty

Name              : 0301HP
CustomRdpProperty : audiocapturemode:i:1;
```

## <a name="add-or-edit-multiple-custom-rdp-properties"></a>Adición y edición de varias propiedades de RDP personalizadas

Para agregar o modificar varias propiedades de RDP personalizadas, ejecute los siguientes cmdlets de PowerShell, pero utilice las propiedades de RDP personalizadas como una cadena separada por punto y coma:

```powershell
$properties="<property1>;<property2>;<property3>"
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty $properties
```

Para comprobar que se ha agregado la propiedad de RDP, ejecute el siguiente cmdlet:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <customRDPpropertystring>
```

Según el ejemplo de cmdlet anterior, si configuró varias propiedades de RDP en el grupo de hosts de 0301HP, el cmdlet tendría el siguiente aspecto:

```powershell
Get-AzWvdHostPool -ResourceGroupName 0301rg -Name 0301hp | format-list Name, CustomRdpProperty

Name              : 0301HP
CustomRdpProperty : audiocapturemode:i:1;audiomode:i:0;
```

## <a name="reset-all-custom-rdp-properties"></a>Restablecimiento de todas las propiedades de RDP personalizadas

Las propiedades de RDP personalizadas se pueden restablecer a sus valores predeterminados. Para ello, solo hay que seguir las instrucciones de [Adición o edición de una sola propiedad de RDP personalizada](#add-or-edit-a-single-custom-rdp-property), o bien se pueden restablecer todas las propiedades de RDP personalizadas de un grupo de hosts ejecutando el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty ""
```

Para asegurarse de que ha quitado correctamente la configuración, escriba este cmdlet:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <CustomRDPpropertystring>
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha personalizado las propiedades de RDP grupo de hosts especificado, puede iniciar sesión en un cliente de Windows Virtual Desktop para realizar pruebas como parte de una sesión de usuario. En estas guías de procedimientos se indica cómo conectarse a una sesión mediante el cliente que elija:

- [Conexión con el cliente de Escritorio de Windows](connect-windows-7-10.md)
- [Conexión con el cliente web](connect-web.md)
- [Conexión con el cliente de Android](connect-android.md)
- [Conexión con el cliente macOS](connect-macos.md)
- [Conexión con el cliente iOS](connect-ios.md)