---
title: Entidades de mensajería de suspensión de Azure Service Bus
description: En este artículo se explica cómo suspender y reactivar temporalmente entidades de mensaje de Azure Service Bus (colas, temas y suscripciones).
ms.topic: article
ms.date: 09/29/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: abdf5f36fd47ab50f784ee494e7ce4891a6a5747
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671467"
---
# <a name="suspend-and-reactivate-messaging-entities-disable"></a>Suspensión y reactivación de entidades de mensajería (deshabilitar)

Las colas, los temas y las suscripciones se pueden suspender temporalmente. La suspensión coloca la entidad en un estado deshabilitado, en el que todos los mensajes se mantienen en el almacenamiento. Sin embargo, no se quitan ni se agregan mensajes, y las operaciones del protocolo respectivo producen error.

Es posible que quiera suspender una entidad por razones administrativas urgentes. Por ejemplo, cuando un receptor defectuoso saca mensajes de la cola, no realiza el procesamiento y encima completa incorrectamente los mensajes y los quita. En este caso, puede que quiera deshabilitar la cola de recepción hasta que corrija e implemente el código. 

El usuario o el sistema pueden realizar la suspensión o la reactivación. El sistema solo suspende las entidades por motivos administrativos graves como haber alcanzado el límite de gasto de suscripción. El usuario no puede reactivar las entidades deshabilitadas por el sistema, pero se restauran cuando se ha solucionado la causa de la suspensión.

## <a name="queue-status"></a>Estado de la cola 
Los estados que se pueden establecer para una **cola** son:

-   **Activa**: la cola está activa. Puede enviar y recibir mensajes de la cola. 
-   **Disabled**: la cola se ha suspendido. Es equivalente a establecer **SendDisabled** y **ReceiveDisabled**. 
-   **SendDisabled**: no puede enviar mensajes a la cola, pero puede recibirlos de ella. Si intenta enviar mensajes a la cola, recibirá una excepción. 
-   **ReceiveDisabled**: puede enviar mensajes a la cola, pero no puede recibirlos de ella. Si intenta recibir mensajes de la cola, obtendrá una excepción.


### <a name="change-the-queue-status-in-the-azure-portal"></a>Cambie el estado de la cola en Azure Portal: 

1. En Azure Portal, vaya al espacio de nombres de Service Bus. 
1. Seleccione la cola para la que desea cambiar el estado. Verá las colas en el panel inferior, en la parte central. 
1. En la página **Cola de Service Bus**, vea el estado actual de la cola como un hipervínculo. Si no está seleccionada la opción de **información general** en el menú de la izquierda, selecciónela para ver el estado de la cola. Seleccione el estado actual de la cola para cambiarlo. 

    :::image type="content" source="./media/entity-suspend/select-state.png" alt-text="Selección del estado de la cola":::
4. Seleccione el nuevo estado de la cola y luego **Aceptar**. 

    :::image type="content" source="./media/entity-suspend/entity-state-change.png" alt-text="Establecimiento del estado de la cola":::
    
También puede deshabilitar las operaciones de envío y recepción mediante las API [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) de Service Bus en el SDK de .NET, o con una plantilla de Azure Resource Manager mediante la CLI de Azure o Azure PowerShell.

### <a name="change-the-queue-status-using-azure-powershell"></a>Cambio del estado de la cola mediante Azure PowerShell
El comando de PowerShell para deshabilitar una cola se muestra en el ejemplo siguiente. El comando de reactivación es equivalente a establecer `Status` en **Activo**.

```powershell
$q = Get-AzServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue

$q.Status = "Disabled"

Set-AzServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue -QueueObj $q
```

## <a name="topic-status"></a>Estado del tema
Puede cambiar el estado del tema en Azure Portal. Seleccione el estado actual del tema para ver la siguiente página, que le permite cambiar el estado. 

:::image type="content" source="./media/entity-suspend/topic-state-change.png" alt-text="Cambio del estado de tema":::

Los estados que se pueden establecer para un **tema** son:
- **Activa**: El tema está activo. Puede enviar mensajes al tema. 
- **Disabled**: El tema se ha suspendido. No puede enviar mensajes al tema. 
- **SendDisabled**: El mismo efecto que **Disabled**. No puede enviar mensajes al tema. Si intenta enviar mensajes al tema, recibirá una excepción. 

## <a name="subscription-status"></a>Estado de la suscripción
Puede cambiar el estado de la suscripción en Azure Portal. Seleccione el estado actual de la suscripción para ver la siguiente página, que le permite cambiar el estado. 

:::image type="content" source="./media/entity-suspend/subscription-state-change.png" alt-text="Cambio del estado de la suscripción":::

Los estados que se pueden establecer para una **suscripción** son:
- **Activa**: Que la suscripción está activa. Puede recibir mensajes de la suscripción.
- **Disabled**: la suscripción está suspendida. No puede recibir mensajes de la suscripción. 
- **ReceiveDisabled**: El mismo efecto que **Disabled**. No puede recibir mensajes de la suscripción. Si intenta recibir mensajes de la suscripción, obtendrá una excepción.

| Estado del tema | Estado de la suscripción | Comportamiento | 
| ------------ | ------------------- | -------- | 
| Activo | Activo | Puede enviar mensajes al tema y recibirlos de la suscripción. | 
| Activo | Deshabilitada o recepción deshabilitada | Puede enviar mensajes al tema, pero no puede recibirlos de la suscripción. | 
| Deshabilitada o envío deshabilitado | Activo | No puede enviar mensajes al tema, pero puede recibir los que ya estén en la suscripción. | 
| Deshabilitada o envío deshabilitado | Deshabilitada o recepción deshabilitada | No puede enviar mensajes al tema ni tampoco recibirlos de la suscripción. | 

## <a name="other-statuses"></a>Otros estados
La enumeración [EntityStatus](/dotnet/api/microsoft.servicebus.messaging.entitystatus) también define un conjunto de estados de transición que solo puede establecer el sistema. 


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[1]: ./media/entity-suspend/entity-state-change.png

