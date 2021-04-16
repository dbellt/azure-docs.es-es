---
title: Preguntas frecuentes sobre Iniciar VM al establecer la conexión de Windows Virtual Desktop - Azure
description: Preguntas frecuentes y procedimientos recomendados para usar la característica Iniciar VM al establecer la conexión.
author: Heidilohr
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 0a35d0541358af2f5fac5e4c7486a1be93797922
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106445591"
---
# <a name="start-vm-on-connect-faq-preview"></a>Preguntas frecuentes sobre Iniciar VM al establecer la conexión (versión preliminar)

> [!IMPORTANT]
> La característica Iniciar VM al establecer la conexión se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se tratan las preguntas frecuentes sobre la característica Iniciar máquina virtual (VM) al establecer la conexión (versión preliminar) para los grupos host de Windows Virtual Desktop.

## <a name="are-vms-automatically-deallocated-when-a-user-stops-using-them"></a>¿Se desasignan automáticamente las VM cuando un usuario deja de usarlas?

No. Tendrá que configurar directivas adicionales para cerrar la sesión de los usuarios y ejecutar scripts de Azure Automation para desasignar las VM.

Para configurar la directiva de desasignación:

1. Conéctese de forma remota a la VM para la que quiere establecer la directiva.

2. Abra el **Editor de directivas de grupo**, luego vaya a **Directiva de equipo local** > **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Servicios de escritorio remoto** > **Host de sesión de escritorio remoto** > **Límites de tiempo de sesión**.

3. Busque la directiva que indica **Establecer el límite de tiempo para las sesiones desconectadas**, a continuación, cambie su valor a **Habilitado**.

4. Una vez habilitada la directiva, seleccione **Finalizar una sesión desconectada**.

>[!NOTE]
>Asegúrese de establecer el límite de tiempo para la directiva "Finalizar una sesión desconectada" en un valor superior a cinco minutos. Un límite de tiempo bajo puede hacer que las sesiones de los usuarios finalicen si sus redes pierden la conexión durante demasiado tiempo, lo que da lugar a la pérdida de trabajo.

Al cerrar la sesión de los usuarios, no se desasignan las VM. Para obtener información sobre cómo desasignar las VM, consulte [Introducción a la característica Start/Stop VMs during off-hours](../automation/automation-solution-vm-management.md).

## <a name="can-users-turn-off-the-vm-from-their-clients"></a>¿Pueden los usuarios apagar las VM desde sus clientes?

Sí. Los usuarios pueden apagar una VM mediante el menú Inicio dentro de su sesión, tal como lo harían con una máquina física. Sin embargo, apagar una VM no la desasignará. Para obtener información sobre cómo desasignar las VM, consulte [Introducción a la característica Start/Stop VMs during off-hours](../automation/automation-solution-vm-management.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo configurar Iniciar VM al establecer la conexión, consulte [Iniciar máquina virtual al establecer la conexión(versión preliminar)](start-virtual-machine-connect.md).

Si tiene preguntas más generales sobre Windows Virtual Desktop, consulte nuestras [preguntas frecuentes](faq.md).
