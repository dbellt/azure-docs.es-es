---
title: Implementación de Azure Spring Cloud en una red virtual
description: Implementación de Azure Spring Cloud en una red virtual (inserción de red virtual)
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 07/21/2020
ms.custom: devx-track-java, devx-track-azurecli, subject-rbac-steps
ms.openlocfilehash: 50d60621c1d6b08ded5a2fdb6f6137281e9dee78
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110462273"
---
# <a name="deploy-azure-spring-cloud-in-a-virtual-network"></a>Implementación de Azure Spring Cloud en una red virtual

**Este artículo se aplica a:** ✔️ Java ✔️ C#

En este tutorial se explica cómo implementar una instancia de Azure Spring Cloud en una red virtual. Esta implementación en ocasiones se llama inserción de red virtual.

La implementación habilita:

* El aislamiento de Internet en la red corporativa de las aplicaciones y el entorno de ejecución del servicio Azure Spring Cloud.
* La interacción de Azure Spring Cloud con los sistemas de centros de datos locales o los servicios de Azure de otras redes virtuales.
* El permiso para que los clientes controlen las comunicaciones de red entrantes y salientes para Azure Spring Cloud.

> [!Note]
> Puede seleccionar la red virtual de Azure solo cuando cree una nueva instancia de servicio de Azure Spring Cloud. No es posible cambiar para usar otra red virtual después de crear Azure Spring Cloud.

## <a name="prerequisites"></a>Requisitos previos

Registre el proveedor de recursos de Azure Spring Cloud **Microsoft.AppPlatform** y **Microsoft.ContainerService** de acuerdo con las instrucciones descritas en [Registro del proveedor de recursos en Azure Portal](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal) o mediante la ejecución del siguiente comando de la CLI de Azure:

```azurecli
az provider register --namespace Microsoft.AppPlatform
az provider register --namespace Microsoft.ContainerService
```

## <a name="virtual-network-requirements"></a>Requisitos de red virtual

La red virtual en la que se implementa la instancia de Azure Spring Cloud debe cumplir los siguientes requisitos:

* **Ubicación**: la red virtual debe residir en la misma ubicación que la instancia de Azure Spring Cloud.
* **Suscripción**: la red virtual debe estar en la misma suscripción que la instancia de Azure Spring Cloud.
* **Subredes**: la red virtual debe incluir dos subredes dedicadas para una instancia de Azure Spring Cloud:

    * Una para el entorno de ejecución del servicio.
    * Y otra para las aplicaciones de microservicios de Spring Boot.
    * Hay una relación uno a uno entre estas subredes y una instancia de Azure Spring Cloud. Use una nueva subred para cada instancia de servicio que implemente. Cada subred solo puede incluir una única instancia de servicio.
* **Espacio de direcciones**: CIDR bloquea hasta */28* tanto para la subred del entorno de ejecución del servicio como para la subred de aplicaciones de microservicios de Spring Boot.
* **Tabla de rutas**: de forma predeterminada, las subredes no necesitan las tablas de rutas existentes asociadas. Puede [traer su propia tabla de rutas](#bring-your-own-route-table).

Los procedimientos siguientes describen la configuración de la red virtual para que contenga la instancia de Azure Spring Cloud.

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Si ya tiene una red virtual para hospedar la instancia de Azure Spring Cloud, omita los pasos 1, 2 y 3. Puede comenzar en el paso 4 para preparar las subredes de la red virtual.

1. En el menú de Azure Portal, seleccione **Crear un recurso**. En Azure Marketplace, seleccione **Redes** > **Red virtual**.

1. En el cuadro de diálogo **Crear red virtual**, escriba o seleccione la siguiente información:

    |Configuración          |Valor                                             |
    |-----------------|--------------------------------------------------|
    |Suscripción     |Seleccione su suscripción.                         |
    |Resource group   |Seleccione el grupo de recursos o cree uno nuevo.  |
    |Nombre             |Escriba **azure-spring-cloud-vnet**.                 |
    |Location         |Seleccione **Este de EE. UU**.                               |

1. Seleccione **Siguiente: Direcciones IP**.

1. En el espacio de direcciones IPv4, escriba **10.1.0.0/16**.

1. Haga clic en **Agregar subred**. Luego, escriba **service-runtime-subnet** en **Nombre de subred** y **10.1.0.0/28** en **Intervalo de direcciones de subred**. A continuación, seleccione **Agregar**.

1. Vuelva a seleccionar **Agregar subred** y, después, escriba los valores de **Nombre de subred** e **Intervalo de direcciones de subred**. Por ejemplo, escriba **apps-subnet** y **10.1.1.0/28**. A continuación, seleccione **Agregar**.

1. Seleccione **Revisar + crear**. Deje el resto tal como está y seleccione **Crear**.

## <a name="grant-service-permission-to-the-virtual-network"></a>Concesión de permisos de servicio a la red virtual
Azure Spring Cloud requiere permisos de **propietario** en la red virtual, con el fin de conceder permisos a una entidad de servicio dinámica y dedicada en la red virtual para realizar más tareas de implementación y mantenimiento.

Seleccione la red virtual **azure-spring-cloud-vnet** que creó anteriormente.

1. Seleccione **Control de acceso (IAM)** y, después, seleccione **Agregar** >  **asignación de roles**.

    ![Captura de pantalla que muestra la pantalla Access control (Control de acceso).](./media/spring-cloud-v-net-injection/access-control.png)

1. Asigne el rol [Propietario](../role-based-access-control/built-in-roles.md#owner) al [usuario | grupo | entidad de servicio | identidad administrada] en el ámbito de [grupo de administración | suscripción | grupo de recursos | recurso].

    Para acceder a los pasos detallados, vea [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md).

Para realizar este paso también puede ejecutar el siguiente comando de la CLI de Azure:

```azurecli
VIRTUAL_NETWORK_RESOURCE_ID=`az network vnet show \
    --name ${NAME_OF_VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP_OF_VIRTUAL_NETWORK} \
    --query "id" \
    --output tsv`

az role assignment create \
    --role "Owner" \
    --scope ${VIRTUAL_NETWORK_RESOURCE_ID} \
    --assignee e8de9221-a19c-4c81-b814-fd37c6caf9d2
```

## <a name="deploy-an-azure-spring-cloud-instance"></a>Implementación de una instancia de Azure Spring Cloud

Para implementar una instancia de Azure Spring Cloud en la red virtual:

1. Abra [Azure Portal](https://portal.azure.com).

1. En el cuadro de búsqueda superior, busque **Azure Spring Cloud**. Seleccione **Azure Spring Cloud** en el resultado.

1. En la página **Azure Spring Cloud**, seleccione **+ Add** (+ Agregar).

1. Rellene el formulario en la página **Crear** de Azure Spring Cloud.

1. Seleccione el mismo grupo de recursos y la misma región que la red virtual.

1. En el campo **Name** (Nombre) situado en **Service Details** (Detalles de servicio), seleccione **azure-spring-cloud-vnet**.

1. Seleccione la pestaña **Redes** y, seguidamente, seleccione los siguientes valores:

    |Configuración                                |Valor                                             |
    |---------------------------------------|--------------------------------------------------|
    |Deploy in your own virtual network (Implementar en su propia red virtual)     |Seleccione **Sí**.                                   |
    |Virtual network                        |Seleccione **azure-spring-cloud-vnet**.               |
    |Service runtime subnet (Subred del entorno de ejecución del servicio)                 |Seleccione **service-runtime-subnet**.                |
    |Spring Boot microservice apps subnet (Subred de aplicaciones de microservicios de Spring Boot)   |Seleccione **apps-subnet**.                           |

    ![Captura de pantalla que muestra la pestaña Redes en la página de creación de Azure Spring Cloud.](./media/spring-cloud-v-net-injection/creation-blade-networking-tab.png)

1. Seleccione **Revisar y crear**.

1. Compruebe sus especificaciones y seleccione **Crear**.

    ![Captura de pantalla que muestra cómo se comprueban las especificaciones.](./media/spring-cloud-v-net-injection/verify-specifications.png)

Después de la implementación, se crearán dos grupos de recursos adicionales en la suscripción para hospedar los recursos de red de la instancia de Azure Spring Cloud. Vaya a **Inicio** y seleccione **Grupos de recursos** en los elementos del menú superior para buscar los siguientes nuevos grupos de recursos.

El grupo de recursos llamado **ap-svc-rt_{nombre de la instancia del servicio}_{región de la instancia del servicio}** contiene los recursos de red del runtime de servicio de la instancia de servicio.

  ![Captura de pantalla que muestra el runtime del servicio.](./media/spring-cloud-v-net-injection/service-runtime-resource-group.png)

El grupo de recursos llamado **ap-app_{nombre de la instancia de servicio}_{región de la instancia de servicio}** contiene los recursos de red de las aplicaciones de microservicios de Spring Boot de la instancia de servicio.

  ![Captura de pantalla que muestra un grupo de recursos de aplicaciones.](./media/spring-cloud-v-net-injection/apps-resource-group.png)

Esos recursos de red están conectados a la red virtual creada en la imagen anterior.

  ![Captura de pantalla que muestra la red virtual con dispositivos conectados.](./media/spring-cloud-v-net-injection/vnet-with-connected-device.png)

   > [!Important]
   > El servicio Azure Spring Cloud administra completamente los grupos de recursos. *No* elimine ni modifique manualmente ningún recurso de ellos.

## <a name="using-smaller-subnet-ranges"></a>Uso de intervalos de subred más pequeños

En esta tabla se muestra el número máximo de instancias de aplicación que admite Azure Spring Cloud con intervalos de subred más pequeños.

| CIDR de la subred de la aplicación | Número total de direcciones IP | Direcciones IP disponibles | Número máximo de instancias de aplicación                                        |
| --------------- | --------- | ------------- | ------------------------------------------------------------ |
| /28             | 16        | 8             | <p> Aplicación con 1 núcleo:  96 <br/> Aplicación con 2 núcleos: 48<br/>  Aplicación con 3 núcleos: 32 <br/> Aplicación con 4 núcleos: 24 </p> |
| /27             | 32        | 24            | <p> Aplicación con 1 núcleo:  228<br/> Aplicación con 2 núcleos: 144<br/>  Aplicación con 3 núcleos: 96 <br/>  Aplicación con 4 núcleos: 72</p> |
| /26             | 64        | 56            | <p> Aplicación con 1 núcleo:  500<br/> Aplicación con 2 núcleos: 336<br/>  Aplicación con 3 núcleos: 224<br/>  Aplicación con 4 núcleos: 168</p> |
| /25             | 128       | 120           | <p> Aplicación con 1 núcleo:  500<br> Aplicación con 2 núcleos:  500<br>  Aplicación con 3 núcleos:  480<br>  Aplicación con 4 núcleos: 360</p> |
| /24             | 256       | 248           | <p> Aplicación con 1 núcleo:  500<br/> Aplicación con 2 núcleos:  500<br/>  Aplicación con 3 núcleos: 500<br/>  Aplicación con 4 núcleos: 500</p> |

En las subredes, Azure reserva cinco direcciones IP y Azure Spring Cloud requiere al menos cuatro direcciones. Se requieren al menos nueve direcciones IP, por lo que /29 y /30 no son operativos.

En el caso de una subred del runtime del servicio, el tamaño mínimo es /28. Este tamaño no tiene ninguna relación con el número de instancias de la aplicación.

## <a name="bring-your-own-route-table"></a>Traer su propia tabla de rutas

Azure Spring Cloud admite el uso de las subredes y las tablas de rutas existentes.

Si las subredes personalizadas no contienen tablas de rutas, Azure Spring Cloud las crea para cada una de las subredes y les agrega reglas a lo largo del ciclo de vida de la instancia. Si las subredes personalizadas contienen tablas de rutas, Azure Spring Cloud confirma las tablas de rutas existentes durante las operaciones de instancia y agrega o actualiza las reglas correspondientes a las operaciones.

> [!Warning] 
> Se pueden agregar y actualizar reglas personalizadas en las tablas de rutas personalizadas. Sin embargo, Azure Spring Cloud agrega reglas y estas no se deben actualizar ni quitar. Las reglas como 0.0.0.0/0 deben existir siempre en una tabla de rutas dada y asignarse al destino de la puerta de enlace de Internet, como una NVA u otra puerta de enlace de salida. Tenga cuidado al actualizar las reglas cuando solo se modifiquen las reglas personalizadas.


### <a name="route-table-requirements"></a>Requisitos de la tabla de rutas

Las tablas de rutas a las que está asociada la red virtual personalizada deben cumplir los siguientes requisitos:

* Puede asociar las tablas de rutas de Azure a la red virtual solo cuando cree una instancia de servicio de Azure Spring Cloud. Una vez que se ha creado Azure Spring Cloud, no puede cambiar y usar otra tabla de rutas.
* Tanto la subred de aplicación de microservicio como la subred de tiempo de ejecución del servicio deben asociarse a distintas tablas de rutas o a ninguna de ellas.
* Los permisos se deben asignar antes de la creación de la instancia. Asegúrese de conceder permisos de *propietario de Azure Spring Cloud* a las tablas de rutas.
* El recurso de la tabla de rutas asociado no se puede actualizar después de crear el clúster. Aunque no se puede actualizar el recurso de la tabla de rutas, se pueden modificar las reglas personalizadas en la tabla de rutas.
* No se puede reutilizar una tabla de rutas con varios clústeres debido al posible conflicto entre reglas de enrutamiento.

## <a name="next-steps"></a>Pasos siguientes

[Implementación de la aplicación en Azure Spring Cloud en la red virtual](https://github.com/microsoft/vnet-in-azure-spring-cloud/blob/master/02-deploy-application-to-azure-spring-cloud-in-your-vnet.md)

## <a name="see-also"></a>Consulte también

- [Solución de problemas de Azure Spring Cloud en una red virtual](https://github.com/microsoft/vnet-in-azure-spring-cloud/blob/master/05-troubleshooting-azure-spring-cloud-in-vnet.md)
- [Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual](https://github.com/microsoft/vnet-in-azure-spring-cloud/blob/master/06-customer-responsibilities-for-running-azure-spring-cloud-in-vnet.md)
