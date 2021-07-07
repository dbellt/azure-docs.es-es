---
title: 'Configuración de las alertas de servicio para Azure Virtual Desktop (clásico): Azure'
description: Configuración de Azure Service Health para recibir notificaciones del servicio de Azure Virtual Desktop (clásico).
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 452dd8ce28589d473b730e1215e86c67fa490e48
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111751746"
---
# <a name="tutorial-set-up-service-alerts-for-azure-virtual-desktop-classic"></a>Tutorial: Configuración de las alertas de servicio para Azure Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop (clásico), que no admite objetos de Azure Virtual Desktop para Azure Resource Manager. Si está intentando administrar objetos de Azure Virtual Desktop para Azure Resource Manager, consulte [este artículo](../set-up-service-alerts.md).

Puede usar Azure Service Health para supervisar los problemas del servicio y los avisos de estado de Azure Virtual Desktop. Azure Service Health puede notificarlo con distintos tipos de alertas (por ejemplo, por correo electrónico o SMS), ayudarlo a entender el impacto de un problema y mantenerlo actualizado con respecto a la resolución del problema. Azure Service Health también puede ayudarlo a mitigar el tiempo de inactividad y preparar el mantenimiento planeado y los cambios que podrían afectar la disponibilidad de los recursos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear y configurar las alertas de servicio.

Para más información sobre Azure Service Health, consulte la [documentación sobre el estado de Azure](../../service-health/index.yml).

## <a name="prerequisites"></a>Prerrequisitos

- [Tutorial: Creación de un inquilino en Azure Virtual Desktop](tenant-setup-azure-active-directory.md)
- [Tutorial: Creación de entidades de servicio y asignaciones de roles con PowerShell](create-service-principal-role-powershell.md)
- [Tutorial: Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace-2019.md)

## <a name="create-service-alerts"></a>Creación de alertas de servicio

En esta sección se muestra cómo configurar Azure Service Health y cómo establecer las notificaciones a las que puede acceder en Azure Portal. Puede configurar distintos tipos de alertas y programarlas para recibir notificaciones de manera oportuna.

### <a name="recommended-service-alerts"></a>Alertas de servicio recomendadas

Se recomienda crear alertas de servicio para los tipos de eventos de estado siguientes:

- **Problema de servicio:** reciba notificaciones sobre los problemas importantes que afectan a la conectividad de los usuarios en el servicio o la capacidad de administrar el inquilino de Azure Virtual Desktop.
- **Aviso de estado:** reciba las notificaciones que requiere su atención. Estos son algunos ejemplos de este tipo de notificación:
    - Máquinas virtuales (VM) no configuradas de manera segura con el puerto 3389 abierto.
    - Funcionalidad en desuso.

### <a name="configure-service-alerts"></a>Configuración de las alertas de servicio

Para configurar las alertas de servicio:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Service Health**.
3. Use las instrucciones que aparecen en [Creación de alertas del registro de actividad en notificaciones del servicio](../../service-health/alerts-activity-log-service-notifications-portal.md?toc=%2fazure%2fservice-health%2ftoc.json) para configurar las alertas y las notificaciones.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a configurar y usar Azure Service Health para supervisar los problemas del servicio y los avisos de estado de Azure Virtual Desktop. Para más información sobre el inicio de sesión en Azure Virtual Desktop, continúe con los procedimientos de conexión a Azure Virtual Desktop.

> [!div class="nextstepaction"]
> [Conexión al cliente de Escritorio remoto en Windows 7 y Windows 10](connect-windows-7-10-2019.md)