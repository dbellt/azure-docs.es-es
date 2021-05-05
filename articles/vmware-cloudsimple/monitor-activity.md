---
title: Supervisión de la actividad de nube privada
titleSuffix: Azure VMware Solution by CloudSimple
description: Describe la información disponible sobre la actividad en el entorno de Azure VMware Solution by CloudSimple, incluidas alertas, eventos, tareas y auditorías.
author: shortpatti
ms.author: v-patsho
ms.date: 08/13/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: cf07231b0883af85fb1f950cf9bb6ed9d5811211
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177321"
---
# <a name="monitor-vmware-solution-by-cloudsimple-activity"></a>Supervisión de una actividad de VMware Solution by CloudSimple

Los registros de actividad de CloudSimple proporcionan una visión general de las operaciones realizadas en el portal de CloudSimple.  La lista incluye alertas, eventos, tareas y auditorías.  Use los registros de actividad para determinar quién, cuándo y qué operaciones se realizaron.  Los registros de actividad no incluyen las operaciones de lectura completadas por un usuario.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="access-the-cloudsimple-portal"></a>Acceso al portal de CloudSimple

Acceda al [portal de CloudSimple](access-cloudsimple-portal.md).

## <a name="activity-information"></a>Información de la actividad

Para acceder a las páginas de actividad, seleccione **Actividad** en el menú lateral.

![Información general de la página Actividad](media/activity-page-overview.png)

Para ver detalles acerca de cualquiera de las actividades de la página Actividad, seleccione la actividad. Se abre un panel de detalles a la derecha. Las acciones del panel dependen del tipo de actividad. Haga clic en **X** para cerrar el panel.

Haga clic en un encabezado de columna para ordenar la visualización.  Puede filtrar las columnas para ver valores específicos.  Para descargar el informe de actividades, haga clic en el icono **Descargar como CSV**.

## <a name="alerts"></a>Alertas

Las alertas son notificaciones de cualquier actividad significativa en el entorno de CloudSimple.  Las alertas incluyen eventos que afectan a la facturación o al acceso de los usuarios.

Para confirmar las alertas y quitarlas de la lista, seleccione una o más de la lista y haga clic en **Confirmar**.

Las siguientes columnas de información están disponibles para las alertas. Haga clic en **Editar columnas** y seleccione las columnas que quiere ver.

| Columna | Descripción |
------------ | ------------- |
| Tipo de alerta | Categoría de la alerta.|
| Time | Hora en que se produjo la alerta. |
| Gravedad | Importancia de la alerta.|
| Nombre de recurso | Nombre asignado al recurso, como el nombre de la nube privada. |
| Tipo de recurso | Categoría del recurso: nube privada, bastidor de nube. |
| Id. de recurso | Identificador del recurso. |
| Descripción | Descripción de lo que desencadenó la alerta. |
| Confirmado | Indicación de si se ha confirmado la alerta. |

## <a name="events"></a>Events

Los eventos muestran la actividad del usuario y del sistema en el portal de CloudSimple. La página Eventos muestra la actividad asociada a un recurso específico y la gravedad del impacto.

Las siguientes columnas de información están disponibles para las alertas. Haga clic en **Editar columnas** y seleccione las columnas que quiere ver.

| Columna | Descripción |
------------ | ------------- |
| Time | Fecha y hora en que se produjo el evento. |
| Tipo de evento | Código numérico que identifica el evento. |
| Gravedad | Gravedad del evento.|
| Nombre de recurso | Nombre asignado al recurso, como el nombre de la nube privada. |
| Tipo de recurso | Categoría del recurso: nube privada, bastidor de nube. |
| Descripción | Descripción de lo que desencadenó la alerta. |

## <a name="tasks"></a>Tareas

Las tareas son actividades de la nube privada que se espera tarden 30 segundos o más en completarse. (Las actividades que se espera que tarden menos de 30 segundos solo se muestran como eventos). Abra la página Tareas para hacer un seguimiento del progreso de las tareas de la nube privada.

Las siguientes columnas de información están disponibles para las alertas. Haga clic en **Editar columnas** y seleccione las columnas que quiere ver.

| Columna | Descripción |
------------ | ------------- |
| Identificador de tarea | Identificador único para la tarea. |
| Operación | Acción que realiza la tarea. |
| Usuario | Usuario asignado para completar la tarea. |
| Nombre de recurso | Nombre asignado al recurso. |
| Tipo de recurso | Categoría del recurso: nube privada, bastidor de nube. |
| Id. de recurso | Identificador del recurso. |
| Start | Hora de inicio de la tarea. |
| End | Hora de finalización de la tarea. |
| Estado | Estado actual de la tarea. |
| Tiempo transcurrido | Tiempo en que tardó en completarse la tarea (si se completa) o que está tomando (si está en curso). |
| Descripción | Descripción de la tarea. |

## <a name="audit"></a>Auditoría

Los registros de auditoría hacen un seguimiento de la actividad del usuario. Puede usar los registros de auditoría para supervisar la actividad de usuario de todos los usuarios.

Las siguientes columnas de información están disponibles para las alertas. Haga clic en **Editar columnas** y seleccione las columnas que quiere ver.

| Columna | Descripción |
------------ | ------------- |
| Time | Hora de la entrada de auditoría. |
| Operación | Acción que realiza la tarea. |
| Usuario | Usuario asignado a la tarea. |
| Nombre de recurso | Nombre asignado al recurso. |
| Tipo de recurso | Categoría del recurso: nube privada, bastidor de nube. |
| Id. de recurso | Identificador del recurso. |
| Resultado | Resultado de la actividad, como **Correcto**. |
| Tiempo empleado | Tiempo para completar la tarea. |
| Descripción | Descripción de la acción. |

## <a name="next-steps"></a>Pasos siguientes

* [Uso de máquinas virtuales de VMware en Azure](quickstart-create-vmware-virtual-machine.md)
* Más información sobre las [nubes privadas](cloudsimple-private-cloud.md)
