---
title: 'Guía de inicio rápido de Azure: Creación de un centro de eventos mediante Azure Portal'
description: En este inicio rápido aprenderá a crear un centro de eventos de Azure mediante Azure Portal.
ms.topic: quickstart
ms.date: 05/25/2021
ms.openlocfilehash: 32a669c08976f1626617b7d17558e2733fa784a5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110375118"
---
# <a name="quickstart-create-an-event-hub-using-azure-portal"></a>Inicio rápido: Creación de un centro de eventos mediante Azure Portal
Azure Event Hubs es una plataforma de streaming de macrodatos y servicio de ingesta de eventos capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. Los datos enviados a un centro de eventos se pueden transformar y almacenar con cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento. Para más información sobre Event Hubs, consulte [Introducción a Event Hubs](event-hubs-about.md) y [Características de Event Hubs](event-hubs-features.md).

En esta guía de inicio rápido se crea un centro de eventos mediante [Azure Portal](https://portal.azure.com).

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que tiene:

- Suscripción de Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es una recopilación lógica de recursos de Azure. Todos los recursos se implementan y administran en un grupo de recursos. Para crear un grupo de recursos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el panel de navegación izquierdo, seleccione **Grupos de recursos**. A continuación, seleccione **Agregar**.

   ![Grupos de recursos: botón Agregar](./media/event-hubs-quickstart-portal/resource-groups1.png)

1. En **Suscripción** seleccione el nombre de la suscripción de Azure en la que desea crear el grupo de recursos.
1. Escriba un **nombre único para el grupo de recursos**. El sistema comprueba de forma inmediata para ver si el nombre está disponible en la suscripción de Azure seleccionada actualmente.
1. Seleccione una **región** para el grupo de recursos.
1. Seleccione **Revisar + crear**.

   ![Grupo de recursos: Crear](./media/event-hubs-quickstart-portal/resource-groups2.png)
1. En la página **Revisar + crear**, seleccione **Crear**. 

## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs

Un espacio de nombres de Event Hubs proporciona un único contenedor donde podrá crear uno o varios centros de eventos. Para crear un espacio de nombres en el grupo de recursos mediante el portal, haga lo siguiente:

1. En Azure Portal, seleccione **Crear un recurso** en la parte superior izquierda de la pantalla.
1. Seleccione **Todos los servicios** en el menú de la izquierda y seleccione el **asterisco (`*`)** junto a **Event Hubs** en la categoría **Análisis**. Confirme que **Event Hubs** se agrega a **FAVORITOS** en el menú de navegación de la izquierda. 
    
   ![Búsqueda de instancias de Event Hubs](./media/event-hubs-quickstart-portal/select-event-hubs-menu.png)
1. Seleccione **Event Hubs** en **FAVORITOS** en el menú de navegación de la izquierda y seleccione **Agregar** en la barra de herramientas.

   ![Botón Agregar](./media/event-hubs-quickstart-portal/event-hubs-add-toolbar.png)
1. En la página **Crear espacio de nombres**, realice los pasos siguientes:  
   1. Seleccione la **suscripción** en la que desea crear el espacio de nombres.  
   1. Seleccione el **grupo de recursos** que ha creado en el paso anterior.   
   1. Escriba el **nombre** del espacio de nombres. El sistema realiza la comprobación automáticamente para ver si el nombre está disponible.  
   1. Seleccione una **ubicación** para el espacio de nombres.
   1. Elija **Básico** como **plan de tarifa**. Para conocer las diferencias entre los niveles, consulte los artículos [Cuotas y límites](event-hubs-quotas.md), [Event Hubs Prémium](event-hubs-premium-overview.md) y [Event Hubs dedicado](event-hubs-dedicated-overview.md). 
   1. Deje las **unidades de rendimiento** (para el nivel estándar) o las **unidades de procesamiento** (para el nivel Prémium) tal como están. Las unidades de procesamiento son unidades de capacidad adquiridas previamente. Para información sobre las unidades de rendimiento o las unidades de procesamiento, consulte [Escalabilidad de Event Hubs](event-hubs-scalability.md).  
   1. En la parte inferior de la página, seleccione **Revisar y crear**.
      
      ![Creación de un espacio de nombres del centro de eventos](./media/event-hubs-quickstart-portal/create-event-hub1.png)
   1. En la página **Revisar y crear**, examine la configuración y seleccione **Crear**. Espere a que la implementación se complete. 
      
      ![Página Revisar y crear](./media/event-hubs-quickstart-portal/review-create.png)
      
   1. En la página **Implementación**, seleccione **Ir al recurso** para ir a la página de su espacio de nombres. 
      
      ![Implementación completada: ir al recurso](./media/event-hubs-quickstart-portal/deployment-complete.png)  
   1. Confirme que la página **Espacio de nombres de Event Hubs** que ve es similar al ejemplo siguiente:   
      
      ![Página principal del espacio de nombres](./media/event-hubs-quickstart-portal/namespace-home-page.png)       

      > [!NOTE]
      > Azure Event Hubs proporciona un punto de conexión de Kafka. Este punto de conexión permite que el espacio de nombres de Event Hubs entienda el protocolo de mensajes y las API de [Apache Kafka](https://kafka.apache.org/intro). Con esta funcionalidad, puede comunicarse con las instancias de Event Hubs como lo haría con temas de Kafka sin cambiar los clientes de protocolo ni ejecutar sus propios clústeres. Admite [Apache Kafka versión 1.0.](https://kafka.apache.org/10/documentation.html) y versiones posteriores. Para más información, consulte [Uso de Azure Event Hubs desde aplicaciones de Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md).
    
## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Para crear un centro de eventos en el espacio de nombres, haga lo siguiente:

1. En la página Espacio de nombres de Event Hubs, seleccione **Event Hubs** en el menú de la izquierda.
1. Seleccione **+ Centro de eventos** en la parte superior de la ventana.
   
    ![Botón Agregar centro de eventos](./media/event-hubs-quickstart-portal/create-event-hub4.png)
1. Escriba el nombre del centro de eventos y seleccione **Crear**.
   
    ![Creación de un centro de eventos](./media/event-hubs-quickstart-portal/create-event-hub5.png)

    La configuración del **número de particiones** permite paralelizar el consumo entre muchos consumidores. Para más información consulte [Particiones](event-hubs-scalability.md#partitions).

    La configuración de **retención de mensajes** especifica cuánto tiempo el servicio de Event Hubs conserva los datos. Para más información, consulte [Retención de eventos](event-hubs-features.md#event-retention).
1. Puede comprobar el estado de la creación del centro de eventos en las alertas. Una vez creado el centro de eventos, puede verlo en la lista de centros de eventos.

    ![Centro de eventos creado](./media/event-hubs-quickstart-portal/event-hub-created.png)
    
## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado un grupo de recursos, un espacio de nombres de Event Hubs y un centro de eventos. En los siguientes tutoriales encontrará instrucciones paso a paso para enviar eventos a un centro de eventos y recibirlos: 

- [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
- [Java](event-hubs-java-get-started-send.md)
- [Python](event-hubs-python-get-started-send.md)
- [JavaScript](event-hubs-node-get-started-send.md)
- [Go](event-hubs-go-get-started-send.md)
- [C (solo enviar)](event-hubs-c-getstarted-send.md)
- [Apache Storm (solo recibir)](event-hubs-storm-getstarted-receive.md)


[Azure portal]: https://portal.azure.com/
[3]: ./media/event-hubs-quickstart-portal/sender1.png
[4]: ./media/event-hubs-quickstart-portal/receiver1.png
