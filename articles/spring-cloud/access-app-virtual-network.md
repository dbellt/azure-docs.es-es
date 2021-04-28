---
title: Acceso de Azure Spring Cloud a una aplicación en una red virtual
description: Acceda a una aplicación en Azure Spring Cloud en una red virtual.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 11/11/2020
ms.custom: devx-track-java
ms.openlocfilehash: 593065b200ab0dc98e5fa97299c137aedfd1be63
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129322"
---
# <a name="access-your-application-in-a-private-network"></a>Acceso a una aplicación en una red privada

En este documento se explica cómo acceder a un punto de conexión de la aplicación en una red privada.  Para obtener acceso, debe crear una **zona DNS privada de Azure** en su suscripción para traducir o resolver el nombre de dominio completo (FQDN) privado en su dirección IP.

Cuando **Assign Endpoint** (Asignar punto de conexión) para aplicaciones en una instancia de servicio en Azure Spring Cloud se implementa en la red virtual, el punto de conexión es un FQDN privado. Solo se puede tener acceso al dominio en la red privada. Las aplicaciones y los servicios usan el punto de conexión de la aplicación. Incluyen el **punto de conexión de prueba** descrito en [Visualización de aplicaciones e implementaciones](./how-to-staging-environment.md#view-apps-and-deployments). El **streaming de registro**, que se describe en [Streaming de registros de aplicaciones de Azure Spring Cloud en tiempo real](./how-to-log-streaming.md), también funcionan solo dentro de la red privada.

## <a name="create-a-private-dns-zone"></a>Crear una zona DNS privada

En el procedimiento siguiente se crea una zona DNS privada para una aplicación en la red privada.

1. Abra Azure Portal. En el cuadro de búsqueda, busque **Zonas DNS privadas** y seleccione **Zonas DNS privadas** en el resultado.

2. En la página **Zonas DNS privadas**, seleccione **+ Agregar**.

3. Rellene el formulario en la página **Crear zona DNS privada**. Escriba **<span>private.azuremicroservices.io</span>** como **Nombre** de la zona.

4. Seleccione **Revisar + crear**.

5. Seleccione **Crear**.

La creación de la zona puede tardar unos minutos.

## <a name="link-the-virtual-network"></a>Vincular la red virtual

Para vincular la zona DNS privada a una red virtual, debe crear un vínculo de red virtual.

1. Seleccione el recurso de zona DNS privada creado anteriormente: **<span>private.azuremicroservices.io</span>** . 

2. En el panel izquierdo, selecciona **Virtual network links** (Vínculos de red virtual).

3. Seleccione **Agregar**.

4. Escriba **azure-spring-cloud-dns-link** para **Nombre del vínculo**.

5. En **Red virtual**, seleccione la red virtual que ha creado tal como se explica en [Implementación de Azure Spring Cloud en la red virtual de Azure (inyección de VNet)](./how-to-deploy-in-azure-virtual-network.md).

    ![Agregar el vínculo de red virtual](media/spring-cloud-access-app-vnet/add-virtual-network-link.png)

6. Haga clic en **OK**.

## <a name="create-dns-record"></a>Crear un registro DNS

Para usar la zona DNS privada para traducir o resolver DNS, debe crear un registro de tipo "A" en la zona.

1. Seleccione el recurso de red virtual que ha creado tal como se explica en [Implementación de Azure Spring Cloud en la red virtual de Azure (inyección de VNet)](./how-to-deploy-in-azure-virtual-network.md).

2. En el cuadro de búsqueda **Dispositivos conectados**, escriba *kubernetes-internal*.

3. En el resultado filtrado, busque el **Dispositivo** conectado al runtime del servicio **Subred** de la instancia del servicio y copie su **Dirección IP**. En este ejemplo, la dirección IP es *10.1.0.7*.

    [ ![Creación de un registro DNS](media/spring-cloud-access-app-vnet/create-dns-record.png) ](media/spring-cloud-access-app-vnet/create-dns-record.png)

O bien, puede capturar la IP con el siguiente comando az de la CLI:

```azurecli
SPRING_CLOUD_RG= # Resource group name of your Azure Spring Cloud service instance
SPRING_CLOUD= # Name of your Azure Spring Cloud service instance

SERVICE_RUNTIME_RG=`az spring-cloud show -g $SPRING_CLOUD_RG -n $SPRING_CLOUD --query \
"properties.networkProfile.serviceRuntimeNetworkResourceGroup" -o tsv`

IP_ADDRESS=`az network lb frontend-ip list --lb-name kubernetes-internal -g \
$SERVICE_RUNTIME_RG --query "[0].privateIpAddress" -o tsv`
```

4. Seleccione el recurso de zona DNS privada creado anteriormente: **<span>private.azuremicroservices.io</span>** .

5. Seleccione **+ Conjunto de registros**.

6. En **Agregar conjunto de registros**, escriba o seleccione esta información:

    |Configuración     |Valor                                                                      |
    |------------|---------------------------------------------------------------------------|
    |Nombre        |Escriba *\**                                                                 |
    |Tipo        |Seleccione **A**.                                                               |
    |TTL         |Escriba *1*.                                                                  |
    |Unidad de TTL    |Seleccione **Horas**.                                                           |
    |Dirección IP  |Escriba la dirección IP copiada en el paso 3. En el ejemplo, escriba *10.1.0.7*.    |

    Después, seleccione **Aceptar**.

    ![Adición de registro de zona DNS privada](media/spring-cloud-access-app-vnet/private-dns-zone-add-record.png)

## <a name="assign-private-fqdn-for-your-application"></a>Asignación de un FQDN privado para la aplicación

Después de seguir el procedimiento descrito en [Creación e implementación de aplicaciones de microservicio](./how-to-deploy-in-azure-virtual-network.md), puede asignar un FQDN privado para la aplicación.

1. Seleccione la instancia del servicio de Azure Spring Cloud implementada en la red virtual y abra la pestaña **Aplicaciones** en el menú de la izquierda.

2. Seleccione la aplicación para mostrar la página **Información general**.

3. Seleccione **Assign Endpoint** (Asignar punto de conexión) para asignar un FQDN privado a la aplicación. Esta operación puede tardar unos minutos.

    ![Asignación de punto de conexión privado](media/spring-cloud-access-app-vnet/assign-private-endpoint.png)

4. Ya está disponible el FQDN privado asignado (con la etiqueta **URL**). Solo se puede tener acceso a él dentro de la red privada, pero no en Internet.

## <a name="access-application-private-fqdn"></a>Acceso al FQDN privado de la aplicación

Después de la asignación, puede tener acceso al FQDN privado de la aplicación en la red privada. Por ejemplo, puede crear una máquina jumpbox en la misma red virtual o en una red virtual del mismo nivel, y en esa máquina jumpbox el FQDN privado es accesible.

![Acceso a un punto de conexión privado en Vnet](media/spring-cloud-access-app-vnet/access-private-endpoint.png)

## <a name="next-steps"></a>Pasos siguientes

- [Exposición de las aplicaciones a Internet mediante Application Gateway y Azure Firewall](./expose-apps-gateway-azure-firewall.md)

## <a name="see-also"></a>Consulte también

- [Solución de problemas de Azure Spring Cloud en una red virtual](./troubleshooting-vnet.md)
- [Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual](./vnet-customer-responsibilities.md)