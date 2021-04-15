---
title: puntos de conexión de servicio de red virtual - Azure Event Hubs | Microsoft Docs
description: En este artículo se proporciona información sobre cómo agregar el punto de conexión de servicio de Microsoft.EventHub a una red virtual.
ms.topic: article
ms.date: 03/29/2021
ms.openlocfilehash: f7f0f3ff480018c9bfc5d9c6f34cf7e2935f8d6a
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105959966"
---
# <a name="allow-access-to-azure-event-hubs-namespaces-from-specific-virtual-networks"></a>Permitir el acceso al espacio de nombres de Event Hubs desde redes virtuales específicas 

La integración de Event Hubs con los [puntos de conexión de servicio de red virtual (VNet)][vnet-sep] permite el acceso seguro a las funcionalidades de mensajería desde cargas de trabajo tales como máquinas virtuales que están enlazadas a redes virtuales, con una ruta de acceso del tráfico de red que está protegida en ambos extremos. 

Una vez realizada la configuración para enlazarse con al menos un punto de conexión de servicio de subred de red virtual, el espacio de nombres respectivo de Event Hubs ya solo aceptará el tráfico procedente de redes virtuales autorizadas. Desde la perspectiva de la red virtual, el enlace de un espacio de nombres de Event Hubs a un punto de conexión de servicio configura un túnel de redes aislado desde la subred de la red virtual al servicio de mensajería. 

El resultado es una relación privada y aislada entre las cargas de trabajo enlazadas a la subred y el espacio de nombres respectivo de Event Hubs, a pesar de que la dirección de red que se puede observar en el punto de conexión de servicio de mensajería esté en un intervalo IP público. Hay una excepción a este comportamiento. Al habilitar un punto de conexión de servicio, de forma predeterminada, se habilita la regla `denyall` en el [firewall de IP](event-hubs-ip-filtering.md) asociado a la red virtual. Puede agregar direcciones IP específicas en el firewall de IP para habilitar el acceso al punto de conexión público del centro de eventos. 

## <a name="important-points"></a>Observaciones importantes
- Esta característica se admite en los niveles tanto **estándar** como **dedicado**. No se admiten en el nivel **básico**.
- La activación de las redes virtuales en el espacio de nombres de Event Hubs bloquea las solicitudes entrantes de manera predeterminada, a menos que las solicitudes se originen en un servicio que funciona desde redes virtuales permitidas. Las solicitudes que bloquean incluyen aquellas de otros servicios de Azure, desde Azure Portal, desde los servicios de registro y de métricas, etc. Como excepción, puede permitir el acceso a los recursos de Event Hubs desde determinados **servicios de confianza**, incluso cuando las redes virtuales están habilitadas. Para ver una lista de servicios de confianza, consulte [Servicios de confianza](#trusted-microsoft-services).
- Especifique **al menos una regla de IP o una regla de red virtual** para que el espacio de nombres permita el tráfico solo desde las direcciones IP o la subred especificadas de una red virtual. Si no hay ninguna regla de red virtual y de IP, se puede acceder al espacio de nombres a través de la red pública de Internet (mediante la clave de acceso).  

## <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>Escenarios de seguridad avanzados que habilita la integración de VNet 

En el caso de soluciones que requieren una estricta seguridad compartimentalizada y en las que las subredes de redes virtuales proporcionan la segmentación entre los servicios compartimentalizados, estas siguen necesitando rutas de comunicación entre los servicios que residen en esos compartimentos.

Cualquier ruta IP inmediata entre compartimientos, incluidos aquellos que transportan HTTPS a través de TCP/IP, tienen el riesgo de sufrir ataques por puntos vulnerables en la capa de red o en capas superiores. Los servicios de mensajería proporcionan rutas de comunicación aisladas, donde los mensajes incluso se escriben en discos cuando se trasladan entre entidades. Las cargas de trabajo de dos redes virtuales distintas enlazadas a la misma instancia de Event Hubs pueden comunicarse de forma eficaz y fiable mediante mensajes, al tiempo que se preserva la integridad de los respectivos límites de aislamiento de red.
 
Esto significa que sus soluciones confidenciales en la nube no solo obtienen acceso a las funcionalidades de mensajería asíncrona de Azure (líderes del sector en fiabilidad y escalabilidad), sino que también pueden usar la mensajería para crear rutas de comunicación entre compartimentos seguros de la solución, que son intrínsecamente más seguros de lo que se puede lograr con cualquier otro modo de comunicación entre iguales, incluyendo en protocolo HTTPS y los protocolos de socket protegidos por TLS.

## <a name="bind-event-hubs-to-virtual-networks"></a>Enlace de centros de eventos a redes virtuales

Las **reglas de red virtual** son una característica de firewall que controla si el espacio de nombres de Azure Event Hubs acepta las conexiones de una subred determinada de una red virtual.

Enlazar un espacio de nombres de Event Hubs a una red virtual es un proceso de dos pasos. Primero debe crear un **punto de conexión de servicio de red virtual** en una subred de red virtual y habilitarlo para **Microsoft.EventHub**, tal como se explica en el artículo de [introducción a los puntos de conexión de servicio][vnet-sep]. Una vez que haya agregado el punto de conexión de servicio, enlácelo con el espacio de nombres de Event Hubs con una **regla de red virtual**.

La regla de red virtual es una asociación del espacio de nombres de Event Hubs con una subred de red virtual. Mientras exista la regla, se les concederá acceso a todas las cargas de trabajo que estén enlazadas a la subred, al espacio de nombres de Event Hubs. Event Hubs no establece nunca por sí mismo conexiones de salida, no necesita obtener acceso y, por tanto, nunca se le concede acceso a la subred habilitando esta regla.

## <a name="use-azure-portal"></a>Usar Azure Portal
En esta sección se muestra cómo usar Azure Portal para agregar un punto de conexión de servicio de red virtual. Para limitar el acceso, debe integrar el punto de conexión de servicio de red virtual para este espacio de nombres de Event Hubs.

1. Vaya a su **espacio de nombres de Event Hubs** en [Azure Portal](https://portal.azure.com).
4. Seleccione **Redes** en **Configuración** en el menú de la izquierda. La pestaña **Redes** solo se muestra para espacios de nombres **estándar** o **dedicados**. 

    > [!WARNING]
    > Si selecciona la opción **Redes seleccionadas** y no agrega al menos una regla de firewall de IP o una red virtual en esta página, se podrá acceder al espacio de nombres desde la **red pública de Internet** (mediante la clave de acceso). 

    :::image type="content" source="./media/event-hubs-firewall/selected-networks.png" alt-text="Pestaña Redes: opción redes seleccionadas" lightbox="./media/event-hubs-firewall/selected-networks.png":::    

    Si selecciona la opción **Todas las redes**, el centro de eventos aceptará conexiones procedentes de cualquier dirección IP (mediante la tecla de acceso). Esta configuración equivale a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0. 

    ![Firewall: opción Todas las redes seleccionada](./media/event-hubs-firewall/firewall-all-networks-selected.png)
1. Para restringir el acceso a redes específicas, seleccione la opción **Selected Networks** (Redes seleccionadas) en la parte superior de la página si aún no está seleccionada.
2. En la sección **Red virtual** de la página, seleccione **+Agregar red virtual existente** _. Seleccione _ *+ Crear una red virtual nueva** si quiere crear una red virtual nueva. 

    ![adición de una red virtual existente](./media/event-hubs-tutorial-vnet-and-firewalls/add-vnet-menu.png)

    >[!WARNING]
    > Si selecciona la opción **Redes seleccionadas** y no agrega al menos una regla de firewall de IP o una red virtual en esta página, se podrá acceder al espacio de nombres desde la red pública de Internet (mediante la clave de acceso).
3. En la lista de redes virtuales, seleccione la red virtual y después elija la **subred**. Debe habilitar el punto de conexión de servicio antes de agregar la red virtual a la lista. Si no está habilitado el punto de conexión de servicio, el portal le pedirá que lo habilite.
   
   ![selección de una subred](./media/event-hubs-tutorial-vnet-and-firewalls/select-subnet.png)

4. Debería ver el siguiente mensaje correcto después de habilitar el punto de conexión de servicio para la subred para **Microsoft.EventHub**. Seleccione **Agregar** en la parte inferior de la página para agregar la red. 

    ![selección de una subred y habilitación de punto de conexión](./media/event-hubs-tutorial-vnet-and-firewalls/subnet-service-endpoint-enabled.png)

    > [!NOTE]
    > Si no puede habilitar el punto de conexión de servicio, puede ignorar el punto de conexión de servicio de red virtual que falta mediante la plantilla de Resource Manager. Esta funcionalidad no está disponible en Azure Portal.
5. Especifique si quiere **permitir que los servicios de confianza de Microsoft omitan este firewall**. Consulte [Servicios de Microsoft de confianza](#trusted-microsoft-services) para más información. 
6. Seleccione **Guardar** en la barra de herramientas para guardar la configuración. Espere unos minutos hasta que la confirmación se muestre en las notificaciones de Azure Portal.

    ![Guardar red](./media/event-hubs-tutorial-vnet-and-firewalls/save-vnet.png)

    > [!NOTE]
    > Para restringir el acceso a intervalos o direcciones concretos, consulte [Permitir el acceso desde intervalos o direcciones IP específicos](event-hubs-ip-filtering.md).

[!INCLUDE [event-hubs-trusted-services](../../includes/event-hubs-trusted-services.md)]

## <a name="use-resource-manager-template"></a>Uso de plantillas de Resource Manager
La siguiente plantilla de Resource Manager de ejemplo agrega una regla de red virtual a un espacio de nombres de Event Hubs. Para la regla de red, especifica el identificador de una subred en una red virtual. 

El identificador es una ruta de acceso de Resource Manager completa para la subred de la red virtual. Por ejemplo, `/subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/default` para la subred predeterminada de una red virtual.

Al agregar reglas de red virtual o de firewalls, establezca el valor de `defaultAction` en `Deny`.


```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "eventhubNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Event Hubs namespace"
        }
      },
      "virtualNetworkName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Virtual Network Rule"
        }
      },
      "subnetName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Virtual Network Sub Net"
        }
      },
      "location": {
        "type": "string",
        "metadata": {
          "description": "Location for Namespace"
        }
      }
    },
    "variables": {
      "namespaceNetworkRuleSetName": "[concat(parameters('eventhubNamespaceName'), concat('/', 'default'))]",
      "subNetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets/', parameters('virtualNetworkName'), parameters('subnetName'))]"
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('eventhubNamespaceName')]",
        "type": "Microsoft.EventHub/namespaces",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
        },
        "properties": { }
      },
      {
        "apiVersion": "2017-09-01",
        "name": "[parameters('virtualNetworkName')]",
        "location": "[parameters('location')]",
        "type": "Microsoft.Network/virtualNetworks",
        "properties": {
          "addressSpace": {
            "addressPrefixes": [
              "10.0.0.0/23"
            ]
          },
          "subnets": [
            {
              "name": "[parameters('subnetName')]",
              "properties": {
                "addressPrefix": "10.0.0.0/23",
                "serviceEndpoints": [
                  {
                    "service": "Microsoft.EventHub"
                  }
                ]
              }
            }
          ]
        }
      },
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[variables('namespaceNetworkRuleSetName')]",
        "type": "Microsoft.EventHub/namespaces/networkruleset",
        "dependsOn": [
          "[concat('Microsoft.EventHub/namespaces/', parameters('eventhubNamespaceName'))]"
        ],
        "properties": {
          "virtualNetworkRules": 
          [
            {
              "subnet": {
                "id": "[variables('subNetId')]"
              },
              "ignoreMissingVnetServiceEndpoint": false
            }
          ],
          "ipRules":[<YOUR EXISTING IP RULES>],
          "trustedServiceAccessEnabled": false,
          "defaultAction": "Deny"
        }
      }
    ],
    "outputs": { }
  }
```

Para implementar la plantilla, siga las instrucciones para [Azure Resource Manager][lnk-deploy].

> [!IMPORTANT]
> Si no hay ninguna regla de red virtual y de IP, todo el tráfico fluye al espacio de nombres, aunque establezca `defaultAction` en `deny`.  Se puede acceder al espacio de nombres a través de la red pública de Internet (mediante la clave de acceso). Especifique al menos una regla de IP o una regla de red virtual para que el espacio de nombres permita el tráfico solo desde las direcciones IP o la subred especificadas de una red virtual.  

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de las redes virtuales, consulte los siguientes vínculos:

- [Punto de conexión de servicio de red virtual de Azure][vnet-sep]
- [Filtrado de IP de Azure Event Hubs][ip-filtering]

[vnet-sep]: ../virtual-network/virtual-network-service-endpoints-overview.md
[lnk-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[ip-filtering]: event-hubs-ip-filtering.md
