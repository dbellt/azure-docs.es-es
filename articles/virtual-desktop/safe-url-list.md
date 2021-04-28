---
title: 'Lista de direcciones URL requeridas de Windows Virtual Desktop: Azure'
description: Una lista de direcciones URL que debe desbloquear para garantizar que su implementación de Windows Virtual Desktop funcione según lo previsto.
author: Heidilohr
ms.topic: conceptual
ms.date: 12/04/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 183b956c0ec38101f7875fe01dd7c6ef9c8c8c8f
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107891687"
---
# <a name="required-url-list"></a>Lista de direcciones URL requeridas

Para implementar y usar Windows Virtual Desktop, debe desbloquear determinadas direcciones URL para que las máquinas virtuales (VM) puedan acceder a ellas en cualquier momento. En este artículo se enumeran las direcciones URL requeridas que se deben desbloquear para que Windows Virtual Desktop funcione correctamente. 

>[!IMPORTANT]
>Windows Virtual Desktop no admite implementaciones que bloqueen las direcciones URL que se muestran en este artículo.

## <a name="required-url-check-tool"></a>Herramienta de comprobación de direcciones URL requeridas

La herramienta de comprobación de direcciones URL requeridas examinará las URL e indicará si están accesibles las URL que la máquina virtual necesita para funcionar. En caso negativo, la herramienta indicará que las direcciones URL son inaccesibles para que pueda desbloquearlas si es necesario.

Es importante tener en cuenta los siguientes aspectos:

- La herramienta de comprobación de direcciones URL requeridas únicamente se puede usar en implementaciones de nubes comerciales.
- La herramienta de comprobación de direcciones URL requeridas no pueden examinar las URL que tienen caracteres comodín. Por tanto, hay que asegurarse de desbloquear esas URL en primer lugar.

### <a name="requirements"></a>Requisitos

Para usar la herramienta comprobación de direcciones URL requeridas, hay que cumplir los siguientes requisitos:

- La máquina virtual debe tener .NET Framework 4.6.2.
- La versión de RDAgent debe ser 1.0.2944.400 o superior.
- El archivo WVDAgentUrlTool.exe debe estar en la misma carpeta que WVDAgentUrlTool.config.

### <a name="how-to-use-the-required-url-check-tool"></a>Uso de la herramienta comprobación de direcciones URL requeridas

Para usar la herramienta, siga estos pasos:

1. Abra un símbolo del sistema como administrador en la máquina virtual.
2. Ejecute el siguiente comando para cambiar el directorio a la misma carpeta que el agente de compilación:

    ```console
    cd C:\Program Files\Microsoft RDInfra\RDAgent_1.0.2944.1200
    ```

3. Ejecute el siguiente comando:

    ```console
    WVDAgentUrlTool.exe
    ```
 
4. Una vez que ejecute el archivo, verá una lista de direcciones URL accesibles e inaccesibles.

    Por ejemplo, en la captura de pantalla siguiente se muestra un escenario en el que tendría que desbloquear dos direcciones URL requeridas que no tengan caracteres comodín:

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la salida de las direcciones URL que no están accesibles.](media/noaccess.png)
    
    Este es el aspecto que debería tener la salida una vez que haya desbloqueado todas las URL requeridas sin caracteres comodín:

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la salida de direcciones URL que son accesibles.](media/access.png)

## <a name="virtual-machines"></a>Máquinas virtuales

Las máquinas virtuales de Azure que cree para Windows Virtual Desktop deben tener acceso a las siguientes direcciones URL en la nube comercial de Azure:

|Dirección|Puerto TCP de salida|Propósito|Etiqueta de servicio|
|---|---|---|---|
|*.wvd.microsoft.com|443|Tráfico de servicio|WindowsVirtualDesktop|
|gcs.prod.monitoring.core.windows.net|443|Tráfico de agente|AzureCloud|
|production.diagnostics.monitoring.core.windows.net|443|Tráfico de agente|AzureCloud|
|*xt.blob.core.windows.net|443|Tráfico de agente|AzureCloud|
|*eh.servicebus.windows.net|443|Tráfico de agente|AzureCloud|
|*xt.table.core.windows.net|443|Tráfico de agente|AzureCloud|
|*xt.queue.core.windows.net|443|Tráfico de agente|AzureCloud|
|catalogartifact.azureedge.net|443|Azure Marketplace|AzureCloud|
|kms.core.windows.net|1688|Activación de Windows|Internet|
|mrsglobalsteus2prod.blob.core.windows.net|443|Actualizaciones de la pila de agentes y SXS|AzureCloud|
|wvdportalstorageblob.blob.core.windows.net|443|Soporte técnico de Azure Portal|AzureCloud|
| 169.254.169.254 | 80 | [Punto de conexión de servicio de metadatos de instancias de Azure](../virtual-machines/windows/instance-metadata-service.md) | N/D |
| 168.63.129.16 | 80 | [Seguimiento de estado del host de sesión](../virtual-network/network-security-groups-overview.md#azure-platform-considerations) | N/D |

>[!IMPORTANT]
>Windows Virtual Desktop ahora admite la etiqueta FQDN. Para más información, consulte [Uso de Azure Firewall para proteger las implementaciones de Windows Virtual Desktop](../firewall/protect-windows-virtual-desktop.md).
>
>Se recomienda usar etiquetas FQDN o etiquetas de servicio, en lugar de las direcciones URL, para evitar problemas de servicio. Las direcciones URL y etiquetas de la lista solo corresponden a sitios y recursos de Windows Virtual Desktop. No incluyen direcciones URL para otros servicios como Azure Active Directory.

Las máquinas virtuales de Azure que cree para Windows Virtual Desktop deben tener acceso a las siguientes direcciones URL en la nube de Azure Government:

|Dirección|Puerto TCP de salida|Propósito|Etiqueta de servicio|
|---|---|---|---|
|*.wvd.azure.us|443|Tráfico de servicio|WindowsVirtualDesktop|
|gcs.monitoring.core.usgovcloudapi.net|443|Tráfico de agente|AzureCloud|
|monitoring.core.usgovcloudapi.net|443|Tráfico de agente|AzureCloud|
|fairfax.warmpath.usgovcloudapi.net|443|Tráfico de agente|AzureCloud|
|*xt.blob.core.usgovcloudapi.net|443|Tráfico de agente|AzureCloud|
|*.servicebus.usgovcloudapi.net|443|Tráfico de agente|AzureCloud|
|*xt.table.core.usgovcloudapi.net|443|Tráfico de agente|AzureCloud|
|Kms.core.usgovcloudapi.net|1688|Activación de Windows|Internet|
|mrsglobalstugviffx.blob.core.usgovcloudapi.net|443|Actualizaciones de la pila de agentes y SXS|AzureCloud|
|wvdportalstorageblob.blob.core.usgovcloudapi.net|443|Soporte técnico de Azure Portal|AzureCloud|
| 169.254.169.254 | 80 | [Punto de conexión de servicio de metadatos de instancias de Azure](../virtual-machines/windows/instance-metadata-service.md) | N/D |
| 168.63.129.16 | 80 | [Seguimiento de estado del host de sesión](../virtual-network/network-security-groups-overview.md#azure-platform-considerations) | N/D |

En la tabla siguiente se enumeran las direcciones URL opcionales a las que pueden tener acceso las máquinas virtuales de Azure:

|Dirección|Puerto TCP de salida|Propósito|Azure Gov|
|---|---|---|---|
|*.microsoftonline.com|443|Autenticación en Microsoft Online Services|login.microsoftonline.us|
|*.events.data.microsoft.com|443|Servicio de telemetría|None|
|www.msftconnecttest.com|443|Detecta si el sistema operativo está conectado a Internet.|None|
|*.prod.do.dsp.mp.microsoft.com|443|Windows Update|None|
|login.windows.net|443|Inicio de sesión en Microsoft Online Services, Microsoft 365|login.microsoftonline.us|
|*.sfx.ms|443|Actualizaciones del software cliente de OneDrive|oneclient.sfx.ms|
|*.digicert.com|443|Comprobación de revocación de certificados|None|
|*.azure-dns.com|443|Resolución de Azure DNS|None|
|*.azure-dns.net|443|Resolución de Azure DNS|None|

>[!NOTE]
>Windows Virtual Desktop actualmente no tiene una lista de intervalos de direcciones IP que puede desbloquear para permitir el tráfico de red. En este momento, solo se pueden desbloquear direcciones URL específicas.
>
>Si usa un firewall de última generación (NGFW), tendrá que usar una lista dinámica específica creada específicamente para las direcciones IP, a fin de asegurarse de que puede conectarse.
>
>Para obtener una lista de direcciones URL seguras relacionadas con Office, incluidas las direcciones URL requeridas relacionadas con Azure Active Directory, consulte [Direcciones URL e intervalos de direcciones IP de Office 365](/office365/enterprise/urls-and-ip-address-ranges).
>
>Debe usar el carácter comodín (*) para las direcciones URL que impliquen tráfico de servicio. Si prefiere no usar el carácter comodín (*) para el tráfico relacionado con el agente, aquí se muestra cómo buscar las direcciones URL sin estos caracteres:
>
>1. Registre las máquinas virtuales en el grupo de hosts de Windows Virtual Desktop.
>2. Abra el **Visor de eventos** y, después, diríjase a **Registros de Windows** > **Aplicación** > **WVD-Agent** y busque el evento con el identificador 3701.
>3. Desbloquee las direcciones URL que se encuentran en el identificador de evento 3701. Las direcciones URL del identificador de evento 3701 son específicas de una región. Deberá repetir el proceso de desbloqueo con las direcciones URL pertinentes de cada región en la que quiera implementar las máquinas virtuales.

## <a name="remote-desktop-clients"></a>Clientes de Escritorio remoto

Todos los clientes de Escritorio remoto que use deben tener acceso a las siguientes direcciones URL:

|Dirección|Puerto TCP de salida|Propósito|Cliente(s)|Azure Gov|
|---|---|---|---|---|
|*.wvd.microsoft.com|443|Tráfico de servicio|All|*.wvd.microsoft.us|
|*.servicebus.windows.net|443|Solución de problemas de los datos|All|*.servicebus.usgovcloudapi.net|
|go.microsoft.com|443|Microsoft FWLinks|All|None|
|aka.ms|443|Acortador de direcciones URL de Microsoft|All|None|
|docs.microsoft.com|443|Documentación|All|None|
|privacy.microsoft.com|443|Declaración de privacidad|All|None|
|query.prod.cms.rt.microsoft.com|443|Actualizaciones de clientes|Escritorio de Windows|Ninguno|

>[!IMPORTANT]
>Para conseguir una experiencia de cliente confiable, es esencial abrir estas direcciones URL. No se admite el bloqueo del acceso a estas direcciones URL y afectará a la funcionalidad del servicio.
>
>Estas direcciones URL solo se corresponden con los sitios de cliente y los recursos. No incluyen direcciones URL para otros servicios como Azure Active Directory. Las direcciones URL de Azure Active Directory se pueden encontrar en el id. 56 en las [direcciones URL e intervalos de direcciones IP de Office 365](/office365/enterprise/urls-and-ip-address-ranges#microsoft-365-common-and-office-online).
