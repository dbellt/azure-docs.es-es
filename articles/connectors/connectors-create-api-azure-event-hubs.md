---
title: Conexión a Azure Event Hubs
description: Conéctese al centro de eventos y agregue un desencadenador o una acción al flujo de trabajo en Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: conceptual
ms.date: 05/03/2021
tags: connectors
ms.openlocfilehash: 7f82debf0cc09d032b00de8197cf873c01801353
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108755592"
---
# <a name="connect-to-an-event-hub-from-workflows-in-azure-logic-apps"></a>Conexión a un centro de eventos desde flujos de trabajo en Azure Logic Apps

El conector de Azure Event Hubs ayuda a conectar los flujos de trabajo de aplicación lógica a centros de eventos de Azure. Luego, puede hacer que los flujos de trabajo supervisen y administren los eventos que se envían a un centro de eventos. Por ejemplo, el flujo de trabajo puede comprobar, enviar y recibir eventos desde el centro de eventos. En este artículo se proporciona una guía de introducción al uso del conector de Azure Event Hubs que muestra cómo conectarse a un centro de eventos y agregar un desencadenador o una acción de Event Hubs al flujo de trabajo.

Para más información sobre Azure Event Hubs o Azure Logic Apps, revise [¿Qué es Azure Event Hubs?](../event-hubs/event-hubs-about.md) o [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="available-operations"></a>Operaciones disponibles

Para información sobre todas las operaciones y otra información técnica, como propiedades, límites, etc., revise la [página de referencia del conector de Event Hubs](/connectors/eventhubs/).

> [!NOTE]
> En el caso de las aplicaciones lógicas hospedadas en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión ISE del conector usa los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/). 

* Un [espacio de nombres de Event Hubs y un centro de eventos](../event-hubs/event-hubs-create.md)

* El flujo de trabajo de la aplicación lógica desde donde quiere acceder al centro de eventos.

  Para iniciar un flujo de trabajo con un desencadenador de Event Hubs, necesita un flujo de trabajo vacío. Si no está familiarizado con [Azure Logic Apps](../logic-apps/logic-apps-overview.md), pruebe este [inicio rápido para crear un flujo de trabajo de aplicación lógica de ejemplo](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="permissions-connection-string"></a>

## <a name="check-permissions-and-get-connection-string"></a>Comprobación de los permisos y obtención de la cadena de conexión

Para que el flujo de trabajo pueda acceder al centro de eventos, compruebe sus permisos y, luego, obtenga la cadena de conexión para el espacio de nombres del centro de eventos.

1. En [Azure Portal](https://portal.azure.com), vaya al *espacio de nombres* de Event Hubs, no a un centro de eventos específico.

1. En el menú del espacio de nombres, en **Configuración**, seleccione **Directivas de acceso compartido**. En la columna **Notificaciones**, compruebe que tiene al menos permisos para **Administrar** en ese espacio de nombres.

   ![Captura de pantalla que muestra Azure Portal, el espacio de nombres de Event Hubs y los permisos para "Administrar" que aparecen en la columna "Notificaciones".](./media/connectors-create-api-azure-event-hubs/event-hubs-namespace.png)

1. Si más adelante quiere especificar manualmente la información de conexión, obtenga la cadena de conexión del espacio de nombres del centro de eventos.

   1. En la columna **Directiva**, seleccione **RootManageSharedAccessKey**.

   1. Busque la cadena de conexión de clave principal. Copie y guarde la cadena de conexión para usarla más adelante.

      ![Captura de pantalla que muestra la cadena de conexión de la clave principal con el botón Copiar seleccionado.](media/connectors-create-api-azure-event-hubs/find-event-hub-namespace-connection-string.png)

      > [!TIP]
      > Para confirmar si la cadena de conexión está asociada al espacio de nombres de Event Hubs o a un centro de eventos específico, asegúrese de que la cadena de conexión no tiene el parámetro `EntityPath`. Si encuentra este parámetro, la cadena de conexión es para una "entidad" de Event Hubs específica y no es la cadena correcta para utilizar con el flujo de trabajo.

<a name="create-connection"></a>

## <a name="create-an-event-hub-connection"></a>Creación de una conexión al centro de eventos

Al agregar por primera vez un desencadenador o una acción de Event Hubs, se le pedirá que cree una conexión al centro de eventos.

1. Cuando esto suceda, elija una de las siguientes opciones:

   * Proporcione la siguiente información de conexión:

     | Propiedad | Obligatorio | Value | Descripción |
     |----------|----------|-------|-------------|
     | **Nombre de la conexión** | Sí | <*connection-name*> | El nombre que se va a crear para su conexión |
     | **Espacio de nombres de Event Hubs** | Sí | <*event-hubs-namespace*> | Seleccione el espacio de nombres de Event Hubs que quiere usar. |
     |||||

   * Para escribir de forma manual la cadena de conexión guardada anteriormente, seleccione **Especificar la información de conexión manualmente**. Obtenga información acerca de [cómo buscar la cadena de conexión](#permissions-connection-string).

1. Seleccione la directiva de Event Hubs que se usará, si no está ya seleccionada, y elija **Crear**.

   ![Captura de pantalla que muestra la información de conexión proporcionada con la opción "Crear" seleccionada.](./media/connectors-create-api-azure-event-hubs/create-event-hubs-connection-2.png)

1. Después de crear la conexión, continúe con [Adición de un desencadenador de Event Hubs](#add-trigger) o [Adición de una acción de Event Hubs](#add-action).

<a name="add-trigger"></a>

## <a name="add-event-hubs-trigger"></a>Adición de un desencadenador de Event Hubs

En Azure Logic Apps, cada flujo de trabajo debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando se cumple una condición determinada. Cada vez que se activa el desencadenador, el servicio Logic Apps crea una instancia del flujo de trabajo y comienza a ejecutar los pasos de este.

En los pasos siguientes se describe la manera general de agregar un desencadenador, por ejemplo, **When events are available in Event Hub** (Cuando los eventos estén disponibles en el centro de eventos). En este ejemplo se muestra cómo agregar un desencadenador que comprueba si hay nuevos eventos en el centro de eventos e inicia una ejecución del flujo de trabajo cuando existen.

1. En el Diseñador de aplicaciones lógicas, abra el flujo de trabajo de aplicación lógica en blanco, si aún no está abierto.

1. En el cuadro de búsqueda de operaciones, escriba `event hubs`. En la lista de desencadenadores, seleccione el desencadenador llamado **When events are available in Event Hub** (Cuando los eventos estén disponibles en el centro de eventos).

   ![Seleccionar un desencadenador](./media/connectors-create-api-azure-event-hubs/find-event-hubs-trigger.png)

1. Si se le pide que cree una conexión al centro de eventos, [proporcione la información de conexión solicitada](#create-connection).

1. En el desencadenador, proporcione información sobre el centro de eventos que quiere supervisar, por ejemplo:

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre del centro de eventos** | Sí | Nombre del centro de eventos que quiere supervisar. |
   | **Tipo de contenido** | No | Tipo de contenido del evento. El valor predeterminado es `application/octet-stream`. |
   | **Nombre del grupo de consumidores** | No | [Nombre del grupo de consumidores de Event Hubs](../event-hubs/event-hubs-features.md#consumer-groups) que se usará para leer los eventos. Si no se especifica, se utiliza el grupo de consumidores predeterminado. |
   | **Número máximo de eventos**. | No | Número máximo de eventos. El desencadenador devuelve entre uno y el número de eventos que especifica esta propiedad. |
   | **Intervalo** | Sí | Entero positivo que describe la frecuencia con la que se ejecuta el flujo de trabajo. |
   | **Frecuencia** | Sí | Unidad de tiempo que se usa para la periodicidad. |
   ||||

   Para ver más propiedades, abra la lista **Agregar nuevo parámetro**. Si selecciona un parámetro, se agrega esa propiedad al desencadenador, por ejemplo:

   ![Propiedades de desencadenador](./media/connectors-create-api-azure-event-hubs/event-hubs-trigger.png)

   **Más propiedades**

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Esquema de contenido** | No | Esquema de contenido JSON de los eventos que se leerán desde el centro de eventos. Por ejemplo, si especifica el esquema de contenido, puede desencadenar el flujo de trabajo solo para los eventos que coincidan con el esquema. |
   | **Clave de partición mínima** | No | Escriba el identificador de la [partición](../event-hubs/event-hubs-features.md#partitions) mínima que va a leer. De forma predeterminada, se leen todas las particiones. |
   | **Clave de partición máxima** | No | Escriba el identificador de la [partición](../event-hubs/event-hubs-features.md#partitions) máxima que va a leer. De forma predeterminada, se leen todas las particiones. |
   | **Zona horaria** | No | Solo se aplica cuando se especifica una hora de inicio porque este desencadenador no acepta diferencia horaria con UTC. Seleccione la zona horaria que desea aplicar. <p>Para obtener más información, consulte [Creación y ejecución de tareas y flujos de trabajo repetitivos con Azure Logic Apps](../connectors/connectors-native-recurrence.md). |
   | **Hora de inicio** | No | Proporcione una hora de inicio con este formato: <p>AAAA-MM-DDThh:mm:ss si selecciona una zona horaria<p>O bien<p>AAAA-MM-DDThh:mm:ssZ si no selecciona una zona horaria<p>Para obtener más información, consulte [Creación y ejecución de tareas y flujos de trabajo repetitivos con Azure Logic Apps](../connectors/connectors-native-recurrence.md). |
   ||||

1. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

1. Ahora continúe agregando una o varias acciones para que pueda realizar otras tareas mediante las salidas del desencadenador.

   Por ejemplo, para filtrar eventos basándose en un valor específico, (como una categoría), puede agregar una condición para que la acción **Send event** (Enviar evento) envíe solo los eventos que cumplan la condición. 

## <a name="trigger-polling-behavior"></a>Desencadenamiento del comportamiento de sondeo

Todos los desencadenadores de Event Hubs son de *sondeo largo*, lo que significa que el desencadenador procesa todos los eventos y espera 30 segundos por partición a que aparezcan más eventos en el centro de eventos. 

Por ejemplo, si el desencadenador está configurado con cuatro particiones, este retraso podría ser de hasta dos minutos antes de que termine de sondearlas todas. Si no se recibe ningún evento dentro de este retraso, se omite la ejecución del desencadenador. De lo contrario, el desencadenador sigue leyendo eventos hasta que el centro de eventos está vacío. El siguiente sondeo del desencadenador ocurre en función del intervalo de periodicidad especificado en las propiedades del desencadenador.

## <a name="trigger-checkpoint-behavior"></a>Desencadenamiento del comportamiento de punto de control

Cuando un desencadenador de Event Hubs lee eventos de cada partición de un centro de eventos, utiliza su propio estado para mantener información sobre el desplazamiento de secuencia (la posición del evento en una partición) y las particiones desde donde el desencadenador lee los eventos.

Cada vez que se ejecuta el flujo de trabajo, el desencadenador lee eventos de una partición, empezando por el desplazamiento de secuencia que mantiene el estado del desencadenador. En modo round-robin, el desencadenador recorre en iteración cada partición del centro de eventos y lee los eventos en las sucesivas ejecuciones. Una sola ejecución obtiene eventos de una sola partición cada vez.

El desencadenador no usa esta funcionalidad de punto de control en el almacenamiento, por lo que no se generan costos adicionales. Sin embargo, el aspecto clave es que al actualizar el desencadenador de Event Hubs se restablece el estado del desencadenador, lo que podría hacer que este leyese eventos al principio de la secuencia.

<a name="add-action"></a>

## <a name="add-event-hubs-action"></a>Adición de una acción de Event Hubs

En Azure Logic Apps, una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) sigue al desencadenador o a otra acción y realiza alguna operación en el flujo de trabajo. En los pasos siguientes se describe la manera general de agregar una acción, por ejemplo, **Send event** (Enviar evento). En este ejemplo, el flujo de trabajo comienza con un desencadenador de Event Hubs que comprueba si hay nuevos eventos en su centro de eventos.

1. En el Diseñador de aplicaciones lógicas, abra el flujo de trabajo de la aplicación lógica, si aún no está abierto.

1. En el desencadenador u otra acción, agregue un nuevo paso.

   Para agregar un paso entre los ya existentes, mueva el mouse sobre la flecha. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda de operaciones, escriba `event hubs`. En la lista de acciones, seleccione la acción **Send event** (Enviar evento).

   ![Seleccione la acción "Enviar evento"](./media/connectors-create-api-azure-event-hubs/find-event-hubs-action.png)

1. Si se le pide que cree una conexión al centro de eventos, [proporcione la información de conexión solicitada](#create-connection).

1. En la acción, proporcione información sobre los eventos que quiere enviar.

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre del centro de eventos** | Sí | Centro de eventos donde quiere enviar el evento. |
   | **Contenido** | No | El contenido del evento que quiere enviar |
   | **Propiedades** | No | Las propiedades de la aplicación y los valores para enviar |
   | **Clave de partición** | No | Identificador de la [partición](../event-hubs/event-hubs-features.md#partitions) a la que se va a enviar el evento. |
   ||||

   Para ver más propiedades, abra la lista **Agregar nuevo parámetro**. Si selecciona un parámetro, se agrega esa propiedad a la acción, por ejemplo:

   ![Seleccione el nombre del centro de eventos y proporcione contenido de eventos.](./media/connectors-create-api-azure-event-hubs/event-hubs-send-event-action.png)

   Por ejemplo, puede enviar la salida desde el desencadenador de Event Hubs hasta otro centro de eventos:

   ![Ejemplo de evento de envío](./media/connectors-create-api-azure-event-hubs/event-hubs-send-event-action-example.png)

1. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

## <a name="connector-reference"></a>Referencia de conectores

Para información sobre todas las operaciones y otra información técnica, como propiedades, límites, etc., revise la [página de referencia del conector de Event Hubs](/connectors/eventhubs/).

> [!NOTE]
> En el caso de las aplicaciones lógicas hospedadas en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión ISE del conector usa los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)