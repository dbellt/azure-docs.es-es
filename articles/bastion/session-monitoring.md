---
title: Supervisión y administración de sesiones de Azure Bastion
description: Aprenda a seleccionar una sesión en curso y forzar su desconexión o eliminación.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 05/21/2020
ms.author: cherylmc
ms.openlocfilehash: 6b0df7ae98c3718ddfadf9e13354b4f961585c98
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110540442"
---
# <a name="session-monitoring-and-management-for-azure-bastion"></a>Supervisión y administración de sesiones de Azure Bastion

Cuando el servicio Bastion se aprovisiona e implementa en la red virtual, puede usarlo para conectarse sin problemas a cualquier máquina virtual de esta red virtual. A medida que los usuarios se conectan a las cargas de trabajo, se Azure Bastion se puede usar para supervisar las sesiones remotas y realizar acciones de administración rápidas. La supervisión de sesiones de Azure Bastion le permite ver qué usuarios están conectados a las máquinas virtuales. Muestra la dirección IP desde la que se conectó el usuario, cuánto tiempo ha estado conectado y cuándo se conectaron. La experiencia de administración de sesiones le permite seleccionar una sesión en curso y forzar su desconexión o eliminar una sesión para desconectar al usuario de la sesión en curso.

## <a name="monitor-remote-sessions"></a><a name="monitor"></a>Supervisión de sesiones remotas

1. En [Azure Portal](https://portal.azure.com), navegue hasta el recurso de Azure Bastion y seleccione **Sesiones** en la página de Azure Bastion.

   ![Captura de pantalla que muestra la configuración del menú de Azure Portal con Sesiones seleccionado.](./media/session-monitoring/sessions.png)
2. En el lado derecho de la página **Sesiones**, puede ver las sesiones remotas en curso.

   ![ver sesión](./media/session-monitoring/view-session.png)
3. Seleccione **Actualizar** para ver la lista actualizada de sesiones remotas. Al seleccionar Actualizar, Azure Bastion capturará la información de supervisión más reciente y la actualizará en el portal.

   ![Captura de pantalla que muestra una sesión que usa Azure Bastion con la opción de actualización resaltada.](./media/session-monitoring/refresh.png)


## <a name="delete-or-force-disconnect-an-ongoing-remote-session"></a><a name="view"></a>Eliminar o forzar la desconexión de una sesión remota en curso

Puede seleccionar un conjunto de sesiones y forzar su desconexión. Los siguientes pasos le mostrarán cómo eliminar sesiones remotas:

1. Navegue hasta el recurso de Azure Bastion y seleccione **Sesiones** en la página de Azure Bastion.

   ![Captura de pantalla que muestra Azure Portal con Sesiones seleccionado bajo Configuración.](./media/session-monitoring/navigate.png)
2. Después de seleccionar Sesiones, verá una lista de sesiones remotas.

   ![lista de sesiones](./media/session-monitoring/list.png)
3. Seleccione una sesión remota específica, seleccione los tres puntos en el extremo derecho de la fila de sesión y, a continuación, seleccione **Eliminar**.

   ![Captura de pantalla que muestra Azure Portal con una sesión con el icono Eliminar seleccionado.](./media/session-monitoring/delete.png)
4. Al seleccionar Eliminar, la sesión remota se desconectará y el usuario recibirá un mensaje que diga "Ha sido desconectado" en la sesión remota.

   ![Captura de pantalla que muestra un mensaje que confirma que se ha desconectado.](./media/session-monitoring/disconnect.png)

## <a name="next-steps"></a>Pasos siguientes

Lea el artículo sobre [preguntas frecuentes de Bastion](bastion-faq.md).
