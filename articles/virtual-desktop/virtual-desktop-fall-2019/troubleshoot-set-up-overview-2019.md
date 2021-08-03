---
title: 'Información general sobre la solución de problemas de Azure Virtual Desktop (clásico): Azure'
description: Información general para solucionar problemas al configurar un entorno de inquilinos de Azure Virtual Desktop (clásico).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/30/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 14449af17a39be884ae023ea23910e617048d5c4
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111753906"
---
#  <a name="azure-virtual-desktop-classic-troubleshooting-overview-feedback-and-support"></a>Información general, comentarios y soporte técnico para la solución de problemas de Azure Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop (clásico), que no admite objetos de Azure Resource Manager. Si está intentando administrar objetos de Azure Virtual Desktop para Azure Resource Manager, consulte [este artículo](../troubleshoot-set-up-overview.md).

En este artículo se proporciona información general de los problemas que pueden surgir al configurar un entorno de inquilinos de Azure Virtual Desktop y proporciona distintas maneras de resolver los problemas.

## <a name="provide-feedback"></a>Envío de comentarios

Visite la [comunidad técnica de Azure Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para analizar el servicio Azure Virtual Desktop con el equipo de producto y los miembros activos de la comunidad.

## <a name="escalation-tracks"></a>Seguimiento del escalado

Use la siguiente tabla para identificar y resolver los problemas que puedan surgir al configurar un entorno de inquilinos mediante el cliente de Escritorio remoto. Una vez que el inquilino esté configurado, puede usar el nuevo [servicio Diagnostics](diagnostics-role-service-2019.md) para identificar los problemas en escenarios comunes.

>[!NOTE]
> En el foro Tech Community puede comentar sus problemas con el equipo del producto y con los miembros activos de la comunidad. Visite el espacio de [Tech Community sobre Azure Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para iniciar una discusión.

| **Problema**                                                            | **Solución propuesta**  |
|----------------------------------------------------------------------|-------------------------------------------------|
| Creación de un inquilino de Azure Virtual Desktop                                                    | Si se produce una interrupción de Azure, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/); en caso contrario, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Azure Virtual Desktop** como servicio, el problema **Implementación** y, a continuación, el subtipo de problema **Issues creating a Azure Virtual Desktop tenant** (Problemas al crear un inquilino de Azure Virtual Desktop).|
| Acceso a las plantillas de Marketplace desde Azure Portal       | Si se produce una interrupción de Azure, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/). <br> <br> Las plantillas de Azure Virtual Desktop de Azure Marketplace están disponibles de forma gratuita.|
| Acceso a las plantillas de Azure Resource Manager desde GitHub                                  | Consulte la sección [Creación de máquinas virtuales de host de sesión de Azure Virtual Desktop](troubleshoot-set-up-issues-2019.md#creating-azure-virtual-desktop-session-host-vms) de [Creación del grupo de inquilinos y del host](troubleshoot-set-up-issues-2019.md). Si aun así el problema no se resuelve, póngase en contacto con el [equipo de soporte técnico de GitHub](https://github.com/contact). <br> <br> Si el error se produce después de acceder a la plantilla en GitHub, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/).|
| Configuración de Express Route y de Azure Virtual Network (VNET) en el grupo del host de sesión               | [Abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) y, a continuación, seleccione el servicio adecuado (en la categoría Redes). |
| Creación de una máquina virtual en el grupo del host de sesión cuando las plantilla de Azure Resource Manager que se proporcionan con Azure Virtual Desktop no se usan | [Abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) y, a continuación, seleccione el servicio **Máquina virtual que ejecuta Windows**. <br> <br> En caso de problemas con las plantillas de Azure Resource Manager que se proporcionan con Azure Virtual Desktop, consulte la sección Crear un inquilino de Azure Virtual Desktop de [Creación de los grupos de inquilinos y de host](troubleshoot-set-up-issues-2019.md). |
| Administración del entorno del host de sesión de Azure Virtual Desktop desde Azure Portal    | [Abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/). <br> <br> Para problemas de administración cuando se usa PowerShell para Servicios de Escritorio remoto/Azure Virtual Desktop, consulte [PowerShell para Azure Virtual Desktop](troubleshoot-powershell-2019.md) o [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Azure Virtual Desktop**, el tipo de problema **Configuration and Management** (Configuración y administración) y, a continuación, el subtipo de problema **Issues configuring tenant using PowerShell** (Problemas al configurar el inquilino con PowerShell). |
| Administración de la configuración de Azure Virtual Desktop asociada a los grupos de hosts y a los grupos de aplicaciones      | Consulte [PowerShell para Azure Virtual Desktop](troubleshoot-powershell-2019.md) o [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Azure Virtual Desktop** y, a continuación, el tipo de problema adecuado.|
| Implementación y administración de contenedores de perfiles de FSLogix | Consulte la [guía de solución de problemas para productos de FSLogix](/fslogix/fslogix-trouble-shooting-ht/) y, si no logra solucionar el problema, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Azure Virtual Desktop**, el tipo de problema **FSLogix** y, a continuación, el subtipo de problema adecuado. |
| Error de funcionamiento de los clientes de Escritorio remoto en el inicio                                                 | Consulte [Solución de problemas del cliente de Escritorio remoto](../troubleshoot-client.md) y, si no logra resolver el problema, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Azure Virtual Desktop** y, a continuación, el tipo de problema **Remote Desktop clients** (Clientes de Escritorio remoto).  <br> <br> Si es un problema de red, los usuarios deben ponerse en contacto con su administrador de red. |
| Está conectado, pero no hay ninguna fuente                                                                 | Para solucionar el problema, consulte la sección [El usuario se conecta, pero no se muestra nada (ninguna fuente)](troubleshoot-service-connection-2019.md#user-connects-but-nothing-is-displayed-no-feed) de [Conexiones de servicios de Azure Virtual Desktop](troubleshoot-service-connection-2019.md). <br> <br> Si sus usuarios se han asignado a un grupo de aplicaciones, [abra una solicitud de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/), seleccione el servicio **Azure Virtual Desktop** y, a continuación, el tipo de problema **Remote Desktop clients** (Clientes de Escritorio remoto). |
| Problemas de detección de fuentes debido a la red                                            | Los usuarios deben ponerse en contacto con su administrador de red. |
| Conexión de clientes                                                                    | Consulte [Conexiones de servicios de Azure Virtual Desktop](troubleshoot-service-connection-2019.md) y, si no se soluciona el problema, consulte [Configuración de máquina virtual de host de sesión](troubleshoot-vm-configuration-2019.md). |
| Capacidad de respuesta del escritorio o de las aplicaciones remotas                                      | Si los problemas están asociados a una aplicación o producto específicos, póngase en contacto con el equipo responsable de dicho producto. |
| Errores o mensajes de las licencias                                                          | Si los problemas están asociados a una aplicación o producto específicos, póngase en contacto con el equipo responsable de dicho producto. |
| Problemas al usar Azure Virtual Desktop en GitHub (plantillas de Azure Resource Manager, herramienta de diagnóstico y herramienta de administración) | Consulte [Plantillas de Azure Resource Manager para Servicios de Escritorio remoto](https://github.com/Azure/RDS-Templates/blob/master/README.md) para informar de problemas. |

## <a name="next-steps"></a>Pasos siguientes

- Para solucionar problemas durante la creación de un grupo de inquilinos y de hosts en un entorno de Azure Virtual Desktop, consulte [Creación de los grupos de inquilinos y de host](troubleshoot-set-up-issues-2019.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Azure Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).
- Para solucionar problemas con conexiones de cliente de Azure Virtual Desktop, consulte [Conexiones de servicios de Azure Virtual Desktop](troubleshoot-service-connection-2019.md).
- Para solucionar problemas con los clientes de Escritorio remoto, consulte [Solución de problemas del cliente de Escritorio remoto](../troubleshoot-client.md).
- Para solucionar problemas al usar PowerShell con Azure Virtual Desktop, consulte [PowerShell para Azure Virtual Desktop](troubleshoot-powershell-2019.md).
- Para más información sobre el servicio, consulte [Entorno de Azure Virtual Desktop](environment-setup-2019.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](../../azure-resource-manager/management/view-activity-logs.md).
- Para obtener información sobre las acciones que permiten determinar los errores que se producen durante la implementación, consulte [Visualización de operaciones de implementación](../../azure-resource-manager/templates/deployment-history.md).
