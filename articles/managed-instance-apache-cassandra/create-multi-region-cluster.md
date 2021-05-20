---
title: 'Inicio rápido: Configuración de un clúster de varias regiones con Azure Managed Instance for Apache Cassandra'
description: En este inicio rápido se muestra cómo configurar un clúster de varias regiones con Azure Managed Instance for Apache Cassandra.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 05/05/2021
ms.openlocfilehash: d2319ee86c356dfc3f145bd7031efe2ff01befa5
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109520582"
---
# <a name="quickstart-create-a-multi-region-cluster-with-azure-managed-instance-for-apache-cassandra-preview"></a>Inicio rápido: Configuración de un clúster de varias regiones con Azure Managed Instance for Apache Cassandra (versión preliminar)

Azure Managed Instance for Apache Cassandra proporciona operaciones de implementación y escalado automatizadas para los centros de datos administrados de código abierto de Apache Cassandra. Este servicio le ayuda a acelerar los escenarios híbridos y a reducir el mantenimiento continuo.

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este inicio rápido se muestra cómo usar los comandos de la CLI de Azure para configurar un clúster de varias regiones en Azure.  

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

* Este artículo requiere la CLI de Azure 2.12.1 o una versión posterior. Si usa Azure Cloud Shell, la versión más reciente ya está instalada.

* [Azure Virtual Network](../virtual-network/virtual-networks-overview.md) con conectividad a su entorno autohospedado o local. Para obtener más información sobre cómo conectar entornos locales a Azure, consulte el artículo [Conexión de una red local a Azure](/azure/architecture/reference-architectures/hybrid-networking/).

## <a name="setting-up-the-network-environment"></a><a id="create-account"></a>Configuración del entorno de red

Como todos los centros de datos aprovisionados con este servicio deben implementarse en subredes dedicadas mediante la inserción de red virtual, es necesario configurar el emparejamiento de red adecuado antes de la implementación para que el clúster de varias regiones funcione correctamente. Vamos a crear un clúster con dos centros de datos en distintas regiones: Este de EE. UU. y Este de EE. UU. 2. En primer lugar, es necesario crear las redes virtuales de cada región. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Cree un grupo de recursos llamado "cassandra-mi-multi-region".

    ```azurecli-interactive
        az group create -l eastus2 -n cassandra-mi-multi-region
    ```

1. Cree la primera red virtual en Este de EE. UU. 2 con una subred dedicada:

    ```azurecli-interactive
        az network vnet create -n vnetEastUs2 -l eastus2 -g cassandra-mi-multi-region --address-prefix 10.0.0.0/16 --subnet-name dedicated-subnet
    ```

1. Ahora, cree la segunda red virtual en Este de EE. UU., también con una subred dedicada:

    ```azurecli-interactive
        az network vnet create -n vnetEastUs -l eastus -g cassandra-mi-multi-region --address-prefix 192.168.0.0/16 --subnet-name dedicated-subnet
    ```

   > [!NOTE]
   > Agregaremos explícitamente distintos intervalos de direcciones IP para garantizar que no se produzcan errores durante el emparejamiento. 

1. Ahora es necesario emparejar la primera red virtual con la segunda:

    ```azurecli-interactive
        az network vnet peering create -g cassandra-mi-multi-region -n MyVnet1ToMyVnet2 --vnet-name vnetEastUs2 \
            --remote-vnet vnetEastUs --allow-vnet-access --allow-forwarded-traffic
    ```

1. Para conectar las dos redes virtuales, cree otro emparejamiento entre la segunda red virtual y la primera:

    ```azurecli-interactive
        az network vnet peering create -g cassandra-mi-multi-region -n MyVnet2ToMyVnet1 --vnet-name vnetEastUs \
            --remote-vnet vnetEastUs2 --allow-vnet-access --allow-forwarded-traffic  
    ```

   > [!NOTE]
   > Si agrega más regiones, será necesario emparejar cada red virtual a todas las demás redes virtuales y viceversa. 

1. Compruebe la salida del comando anterior y asegúrese de que el valor de "peeringState" es ahora "Conectado". Otra manera de comprobarlo es ejecutar el siguiente comando:

    ```azurecli-interactive
        az network vnet peering show \
          --name MyVnet1ToMyVnet2 \
          --resource-group cassandra-mi-multi-region \
          --vnet-name vnetEastUs2 \
          --query peeringState
    ``` 

1. A continuación, aplique algunos permisos especiales a ambas redes virtuales, que se requieren en Azure Managed Instance for Apache Cassandra. Ejecute el siguiente código y asegúrese de reemplazar `<Subscription ID>` por su identificador de suscripción:

    ```azurecli-interactive
        az role assignment create --assignee a232010e-820c-4083-83bb-3ace5fc29d0b --role 4d97b98b-1d4f-4787-a291-c67834d212e7 --scope /subscriptions/<Subscription ID>/resourceGroups/cassandra-mi-multi-region/providers/Microsoft.Network/virtualNetworks/vnetEastUs2
        az role assignment create --assignee a232010e-820c-4083-83bb-3ace5fc29d0b --role 4d97b98b-1d4f-4787-a291-c67834d212e7 --scope /subscriptions/<Subscription ID>/resourceGroups/cassandra-mi-multi-region/providers/Microsoft.Network/virtualNetworks/vnetEastUs
    ```
   > [!NOTE]
   > Los valores `assignee` y `role` del comando anterior son valores fijos. Escriba estos valores exactamente como se mencionó en el comando. Si no lo hace, se producirán errores al crear el clúster. Si se producen errores al ejecutar este comando, es posible que no tenga permisos para ejecutarlo. Para obtenerlos, póngase en contacto con el administrador.

## <a name="create-a-multi-region-cluster"></a><a id="create-account"></a>Creación de un clúster de varias regiones

1. Ahora que tenemos las redes adecuadas, estamos listos para implementar el recurso de clúster (reemplace `<Subscription ID>` por su identificador de suscripción). La instalación puede tardar entre 5 y 10 minutos.

    ```azurecli-interactive
        resourceGroupName='cassandra-mi-multi-region'
        clusterName='test-multi-region'
        location='eastus2'
        delegatedManagementSubnetId='/subscriptions/<Subscription ID>/resourceGroups/cassandra-mi-multi-region/providers/Microsoft.Network/virtualNetworks/vnetEastUs2/subnets/dedicated-subnet'
        initialCassandraAdminPassword='myPassword'
        
        az managed-cassandra cluster create \
           --cluster-name $clusterName \
           --resource-group $resourceGroupName \
           --location $location \
           --delegated-management-subnet-id $delegatedManagementSubnetId \
           --initial-cassandra-admin-password $initialCassandraAdminPassword \
           --debug
    ```

1. Después de crear el recurso de clúster, está listo para crear un centro de datos. En primer lugar, cree un centro de datos en Este de EE. UU. 2 (reemplace `<Subscription ID>` por su identificador de suscripción). Esta operación puede tardar hasta 10 minutos.

    ```azurecli-interactive
        resourceGroupName='cassandra-mi-multi-region'
        clusterName='test-multi-region'
        dataCenterName='dc-eastus2'
        dataCenterLocation='eastus2'
        delegatedManagementSubnetId='/subscriptions/<Subscription ID>/resourceGroups/cassandra-mi-multi-region/providers/Microsoft.Network/virtualNetworks/vnetEastUs2/subnets/dedicated-subnet'
        
        az managed-cassandra datacenter create \
           --resource-group $resourceGroupName \
           --cluster-name $clusterName \
           --data-center-name $dataCenterName \
           --data-center-location $dataCenterLocation \
           --delegated-subnet-id $delegatedManagementSubnetId \
           --node-count 3
    ```

1. Luego, cree un centro de datos en Este de EE. UU. (reemplace `<Subscription ID>` por su identificador de suscripción).

    ```azurecli-interactive
        resourceGroupName='cassandra-mi-multi-region'
        clusterName='test-multi-region'
        dataCenterName='dc-eastus'
        dataCenterLocation='eastus'
        delegatedManagementSubnetId='/subscriptions/<Subscription ID>/resourceGroups/cassandra-mi-multi-region/providers/Microsoft.Network/virtualNetworks/vnetEastUs/subnets/dedicated-subnet'
        
        az managed-cassandra datacenter create \
           --resource-group $resourceGroupName \
           --cluster-name $clusterName \
           --data-center-name $dataCenterName \
           --data-center-location $dataCenterLocation \
           --delegated-subnet-id $delegatedManagementSubnetId \
           --node-count 3 
    ```

1. Una vez creado el segundo centro de datos, obtenga el estado del nodo para comprobar que todos los nodos de Cassandra funcionan correctamente:

    ```azurecli-interactive
    resourceGroupName='cassandra-mi-multi-region'
    clusterName='test-multi-region'
    
    az managed-cassandra cluster node-status \
        --cluster-name $clusterName \
        --resource-group $resourceGroupName
    ```


1. Por último, [conecte el clúster](create-cluster-cli.md#connect-to-your-cluster) mediante CQLSH y use la siguiente consulta de CQL para actualizar la estrategia de replicación de cada espacio de claves a fin de incluir todos los centros de datos del clúster:

   ```bash
   ALTER KEYSPACE "ks" WITH REPLICATION = {'class': 'NetworkTopologyStrategy', 'dc-eastus2': 3, 'dc-eastus': 3};
   ```
   También debe actualizar las tablas de contraseñas:

   ```bash
    ALTER KEYSPACE "system_auth" WITH REPLICATION = {'class': 'NetworkTopologyStrategy', 'dc-eastus2': 3, 'dc-eastus': 3}
   ```

## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error al aplicar permisos a la red virtual, por ejemplo, uno que dice que *no se encuentra el usuario o la entidad de servicio en la base de datos de grafos para "e5007d2c-4b13-4a74-9b6a-605d99f03501"* , puede aplicar el mismo permiso manualmente desde Azure Portal. 

Para aplicar permisos desde Azure Portal, vaya al panel **Control de acceso (IAM)** de la red virtual existente y agregue una asignación de roles de "Azure Cosmos DB" al rol "Administrador de red". Si aparecen dos entradas al buscar "Azure Cosmos DB", agregue las dos entradas, tal como se muestra en la siguiente imagen: 

   :::image type="content" source="./media/create-cluster-cli/apply-permissions.png" alt-text="Permisos de aplicación" lightbox="./media/create-cluster-cli/apply-permissions.png" border="true":::

> [!NOTE] 
> La asignación de roles de Azure Cosmos DB se usa solo con fines de implementación. Azure Managed Instance for Apache Cassandra no tiene dependencias de back-end en Azure Cosmos DB.  

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando este clúster de instancia administrada, elimínelo mediante los siguientes pasos:

1. En el menú de la izquierda de Azure Portal, seleccione **Grupos de recursos**.
1. En la lista, seleccione el grupo de recursos que creó para este inicio rápido.
1. En el panel **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
3. En la ventana siguiente, escriba el nombre del grupo de recursos que desea eliminar y, después, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear un clúster de varias regiones con la CLI de Azure y Azure Managed Instance for Apache Cassandra. Ahora puede empezar a trabajar con el clúster.

> [!div class="nextstepaction"]
> [Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure](manage-resources-cli.md)
