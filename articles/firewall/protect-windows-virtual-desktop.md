---
title: Uso de Azure Firewall para proteger Windows Virtual Desktop
description: Aprenda a usar Azure Firewall para proteger las implementaciones de Windows Virtual Desktop
author: vhorne
ms.service: firewall
services: firewall
ms.topic: how-to
ms.date: 05/06/2020
ms.author: victorh
ms.openlocfilehash: 7b9de22a3209a75cec680ae3ea04d2e1f54c956c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110453273"
---
# <a name="use-azure-firewall-to-protect-window-virtual-desktop-deployments"></a>Uso de Azure Firewall para proteger las implementaciones de Windows Virtual Desktop

Windows Virtual Desktop es un servicio de virtualización de aplicaciones y de escritorio que se ejecuta en Azure. Cuando un usuario final se conecta a un entorno de Windows Virtual Desktop, un grupo de hosts ejecuta su sesión. Un grupo de hosts es una colección de máquinas virtuales de Azure que se registran en Windows Virtual Desktop como hosts de sesión. Estas máquinas virtuales se ejecutan en la red virtual y están sujetas a sus controles de seguridad. Necesitan acceso saliente a Internet para que el servicio Windows Virtual Desktop funcione correctamente y puede que también requieran acceso a Internet para los usuarios finales. Azure Firewall puede ayudarle a bloquear su entorno y a filtrar el tráfico saliente.

[ ![Arquitectura de Windows Virtual Desktop](media/protect-windows-virtual-desktop/windows-virtual-desktop-architecture-diagram.png) ](media/protect-windows-virtual-desktop/windows-virtual-desktop-architecture-diagram.png#lightbox)

Siga las instrucciones de este artículo para proporcionar protección adicional para el grupo de hosts de Windows Virtual Desktop mediante Azure Firewall.

## <a name="prerequisites"></a>Prerrequisitos


 - Un entorno de Windows Virtual Desktop implementado y un grupo de hosts.

   Para más información, consulte el [Tutorial: Creación de un grupo de hosts con Azure Marketplace](../virtual-desktop/create-host-pools-azure-marketplace.md) y [Creación de un grupo de hosts con una plantilla de Azure Resource Manager](../virtual-desktop/virtual-desktop-fall-2019/create-host-pools-arm-template.md).

Para más información sobre los entornos de Windows Virtual Desktop, consulte [Entorno de Windows Virtual Desktop](../virtual-desktop/environment-setup.md).

## <a name="host-pool-outbound-access-to-windows-virtual-desktop"></a>Acceso de salida del grupo de hosts a Windows Virtual Desktop

Las máquinas virtuales de Azure que cree para Windows Virtual Desktop deben tener acceso a varios nombres de dominio completos (FQDN) para que funcionen correctamente. Azure Firewall proporciona una etiqueta FQDN de Windows Virtual Desktop para simplificar esta configuración. Siga estos pasos para permitir el tráfico saliente de la plataforma Windows Virtual Desktop:

- Implemente Azure Firewall y configure la ruta definida por el usuario (UDR) de la subred del grupo de hosts de Windows Virtual Desktop para redirigir el tráfico predeterminado (0.0.0.0/0) mediante Azure Firewall. La ruta predeterminada apunta ahora al firewall.
- Cree una colección de reglas de aplicación y agregue una regla para habilitar la etiqueta FQDN *WindowsVirtualDesktop*. El intervalo de direcciones IP de origen es la red virtual del grupo de hosts, el protocolo es **https** y el destino es **WindowsVirtualDesktop**.

- El conjunto de cuentas de Service Bus y de Storage necesarias para el grupo de hosts de Windows Virtual Desktop es específico de la implementación, por lo que aún no se ha capturado en la etiqueta FQDN WindowsVirtualDesktop. Puede abordar esto de una de las formas siguientes:

   - Permita el acceso HTTPS desde la subred del grupo de hosts a *xt.blob.core.windows.net y *eh.servicebus.windows.net. Estos FQDN con caracteres comodín permiten el acceso requerido, pero son menos restrictivos.
   - Use la siguiente consulta de análisis de registros para enumerar los FQDN necesarios exactos después de la implementación del grupo de hosts de WVD y, a continuación, permitirlos explícitamente en las reglas de aplicación del firewall:
   ```
   AzureDiagnostics
   | where Category == "AzureFirewallApplicationRule"
   | search "Deny"
   | search "gsm*eh.servicebus.windows.net" or "gsm*xt.blob.core.windows.net"
   | parse msg_s with Protocol " request from " SourceIP ":" SourcePort:int " to " FQDN ":" *
   | project TimeGenerated,Protocol,FQDN
   ```

- Cree una colección de reglas de red y agregue las siguientes:

   - Permitir DNS: para permitir el tráfico desde su dirección IP privada de ADDS a * para los puertos TCP y UDP 53.
   - Permitir KMS: para permitir el tráfico desde las máquinas virtuales con Windows Virtual Desktop al puerto TCP 1688 de Windows Activation Service. Para más información acerca de las direcciones IP de destino, consulte [Error de activación de Windows en el escenario de tunelización forzada](/troubleshoot/azure/virtual-machines/custom-routes-enable-kms-activation#solution).

> [!NOTE]
> Es posible que algunas implementaciones no necesiten reglas DNS. Por ejemplo, los controladores de dominio de Azure Active Directory reenvían las consultas de DNS a Azure DNS en 168.63.129.16.

## <a name="host-pool-outbound-access-to-the-internet"></a>Acceso de salida del grupo de hosts a Internet

En función de las necesidades de su organización, es posible que desee habilitar el acceso de salida a Internet seguro para los usuarios finales. En los casos en los que la lista de destinos permitidos está bien definida (por ejemplo, en el [acceso a Microsoft 365](/microsoft-365/enterprise/microsoft-365-ip-web-service)), puede usar reglas de red y de aplicación de Azure Firewall para configurar el acceso necesario. De esta forma, enruta el tráfico del usuario final directamente a Internet para obtener el mejor rendimiento.

Si quiere filtrar el tráfico saliente de Internet de los usuarios mediante una puerta de enlace web segura local existente, puede configurar exploradores web u otras aplicaciones que se ejecuten en el grupo de hosts de Windows Virtual Desktop con una configuración de proxy explícita. Por ejemplo, consulte [Cómo usar las opciones de línea de comandos de Microsoft Edge para establecer la configuración de proxy](/deployedge/edge-learnmore-cmdline-options-proxy-settings). Esta configuración de proxy solo afecta al acceso a Internet del usuario final, de modo que permite el tráfico saliente de la plataforma Windows Virtual Desktop directamente mediante Azure Firewall.

## <a name="additional-considerations"></a>Consideraciones adicionales

Es posible que tenga que configurar reglas de firewall adicionales, en función de sus requisitos:

- Acceso al servidor NTP

   De forma predeterminada, las máquinas virtuales que ejecutan Windows se conectan a time.windows.com a través del puerto UDP 123 para la sincronización de hora. Cree una regla de red para permitir este acceso o para un servidor horario que use en su entorno.


## <a name="next-steps"></a>Pasos siguientes

- Más información sobre Windows Virtual Desktop: [¿Qué es Windows Virtual Desktop?](../virtual-desktop/overview.md)
