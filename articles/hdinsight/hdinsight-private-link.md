---
title: Protección y aislamiento de clústeres de Azure HDInsight con Private Link (versión preliminar)
description: Aprenda a aislar los clústeres de Azure HDInsight en una red virtual mediante Azure Private Link.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: fe3b9617db20f445d4139c006c283bbfe537d544
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110616485"
---
# <a name="secure-and-isolate-azure-hdinsight-clusters-with-private-link-preview"></a>Protección y aislamiento de clústeres de Azure HDInsight con Private Link (versión preliminar)

En la [arquitectura de red virtual predeterminada](./hdinsight-virtual-network-architecture.md) de Azure HDInsight, el proveedor de recursos (RP) de HDInsight se comunica con el clúster mediante IP públicas. Algunos escenarios requieren el aislamiento de red completo sin usar IP públicas. En este artículo, obtendrá información sobre los controles avanzados que puede usar para crear un clúster de HDInsight privado. Para información sobre cómo restringir el tráfico hacia y desde el clúster sin el aislamiento de red completo, consulte [Control del tráfico de red en Azure HDInsight](./control-network-traffic.md).

Puede crear clústeres de HDInsight privados mediante la configuración de propiedades de red específicas en una plantilla de Azure Resource Manager (ARM). Hay dos propiedades que se usan para crear clústeres de HDInsight privados:

* Establecer `resourceProviderConnection` en "saliente" para quitar las IP públicas.
* Establecer `privateLink` en "habilitado" para habilitar Azure Private Link y usar [Puntos de conexión privados](../private-link/private-endpoint-overview.md).

## <a name="remove-public-ip-addresses"></a>Eliminación de IP públicas

De manera predeterminada, el proveedor de recursos de HDInsight usa una conexión *entrante* al clúster mediante IP públicas. Cuando la propiedad de red `resourceProviderConnection` está establecida en *saliente*, invierte las conexiones al proveedor de recursos de HDInsight para que las conexiones siempre se inicien desde dentro del clúster hacia el RP. Sin una conexión entrante, no son necesarias las IP públicas ni las etiquetas de servicio de entrada.

Los equilibradores de carga básicos que se usan en la arquitectura de red virtual predeterminada proporcionan automáticamente una NAT (traducción de direcciones de red) pública para acceder a las dependencias de salida requeridas, como el RP de HDInsight. Si quiere restringir la conectividad de salida a la red pública de Internet, puede [configurar un firewall](./hdinsight-restrict-outbound-traffic.md), pero no es requisito que lo haga.

La configuración de `resourceProviderConnection` como saliente también le permite acceder a recursos específicos del clúster, como Azure Data Lake Storage Gen2 o metastores externos, mediante puntos de conexión privados. No es obligatorio usar puntos de conexión privados para estos recursos, pero si planea hacerlo, debe configurar los puntos de conexión privados y las entradas DNS `before` de crear el clúster de HDInsight. Es recomendable crear y proporcionar todas las bases de datos SQL externas necesarias, como Apache Ranger, Ambari, Oozie y metastores de Hive, durante la creación del clúster. Todos estos recursos deben ser accesibles desde dentro de la subred del clúster, ya sea a través de su propio punto de conexión privado o de cualquier otro modo.

Al conectarse a Azure Data Lake Storage Gen2 a través de un punto de conexión privado, asegúrese de que la cuenta de almacenamiento de Gen2 tenga un punto de conexión establecido para "blob" y "dfs". Para más información, consulte [Creación de un punto de conexión privado](../storage/common/storage-private-endpoints.md).

En el diagrama siguiente se muestra el aspecto que podría tener una posible arquitectura de red virtual de HDInsight cuando `resourceProviderConnection` está establecido en "saliente":

:::image type="content" source="media/hdinsight-private-link/outbound-resource-provider-connection-only.png" alt-text="Diagrama de una arquitectura de HDInsight que usa una conexión de proveedor de recursos de salida":::

Después de crear el clúster, debe configurar la resolución DNS correspondiente. El registro DNS de nombre canónico (CNAME) siguiente se crea en la zona DNS pública administrada por Azure: `azurehdinsight.net`.

```dns
<clustername>    CNAME    <clustername>-int
```

Para acceder al clúster mediante los nombres de dominio completo del clúster, puede usar directamente las IP privadas del equilibrador de carga interno o usar su propia zona DNS privada para reemplazar los puntos de conexión del clúster según corresponda a sus necesidades. Por ejemplo, puede tener una zona DNS privada, `azurehdinsight.net`, y agregar las direcciones IP privadas según sea necesario:

```dns
<clustername>        A   10.0.0.1
<clustername-ssh>    A   10.0.0.2
```

## <a name="enable-private-link"></a>Habilitación de Private Link

Private Link, que está deshabilitado de manera predeterminada, requiere un amplio conocimiento de las redes para configurar correctamente rutas definidas por el usuario (UDR) y reglas de firewall antes de crear un clúster. El uso de este valor es opcional, pero solo está disponible cuando la propiedad de red `resourceProviderConnection` está establecida en *saliente*, tal y como se ha descrito en la sección anterior.

Cuando `privateLink` está establecido en *habilitado*, se crean [equilibradores de carga estándar](../load-balancer/load-balancer-overview.md) (SLB) y se aprovisiona un servicio Azure Private Link para cada uno de ellos. El servicio Private Link es lo que permite acceder al clúster de HDInsight desde puntos de conexión privados.

Los equilibradores de carga estándar no proporcionan automáticamente la [NAT saliente pública](../load-balancer/load-balancer-outbound-connections.md) como sí lo hacen los equilibradores de carga básicos. Debe proporcionar su propia solución NAT, como [Virtual Network NAT](../virtual-network/nat-overview.md) o un [firewall](./hdinsight-restrict-outbound-traffic.md), para las dependencias de salida. El clúster de HDInsight de todos modos necesita acceso a sus dependencias de salida. Si no se permiten estas dependencias de salida, puede producirse un error en la creación del clúster.

### <a name="prepare-your-environment"></a>Preparación del entorno

Para crear servicios de vínculo privado de forma correcta, debe [deshabilitar explícitamente las directivas de red para el servicio de vínculo privado](../private-link/disable-private-link-service-network-policy.md).

En el diagrama siguiente se muestra un ejemplo de la configuración de red necesaria antes de crear un clúster. En este ejemplo, todo el tráfico de salida es [forzado](../firewall/forced-tunneling.md) a Azure Firewall mediante UDR y se deben "permitir" las dependencias de salida requeridas en el firewall antes de crear un clúster. En el caso de los clústeres de Enterprise Security Package, el emparejamiento de red virtual puede proporcionar la conectividad de red a Azure Active Directory Domain Services.

:::image type="content" source="media/hdinsight-private-link/before-cluster-creation.png" alt-text="Diagrama del entorno de Private Link antes de la creación del clúster":::

Una vez que configura las redes, puede crear un clúster con la conexión del proveedor de recursos saliente y Private Link habilitado, tal como se muestra en la ilustración siguiente. En esta configuración, no hay IP públicas y el servicio Private Link se aprovisiona para cada equilibrador de carga estándar.

:::image type="content" source="media/hdinsight-private-link/after-cluster-creation.png" alt-text="Diagrama del entorno de Private Link después de la creación del clúster":::

### <a name="access-a-private-cluster"></a>Acceso a un clúster privado

Para acceder a los clústeres privados, puede usar directamente las IP privadas del equilibrador de carga interno, o bien puede usar las extensiones DNS de Private Link y puntos de conexión privados. Cuando el valor de `privateLink` está establecido en "habilitado", puede crear sus propios puntos de conexión privados y configurar la resolución DNS a través de zonas DNS privadas.

Las entradas de Private Link creadas en la zona DNS pública `azurehdinsight.net` administrada por Azure son las siguientes:

```dns
<clustername>        CNAME    <clustername>.privatelink
<clustername>-int    CNAME    <clustername>-int.privatelink
<clustername>-ssh    CNAME    <clustername>-ssh.privatelink
```

En la imagen siguiente se muestra un ejemplo de las entradas DNS privadas que se necesitan para acceder al clúster desde una red virtual que no está emparejada o no tiene una línea de visión directa respecto de los equilibradores de carga del clúster. Puede usar Azure Private Zone para reemplazar los nombres de dominio completo `*.privatelink.azurehdinsight.net` y resolver las direcciones IP de sus propios puntos de conexión privados.

:::image type="content" source="media/hdinsight-private-link/access-private-clusters.png" alt-text="Diagrama de la arquitectura de Private Link":::

## <a name="how-to-create-clusters"></a>¿Cómo se crean clústeres?
### <a name="use-arm-template-properties"></a>Uso de propiedades de plantilla de ARM

El fragmento de código JSON siguiente incluye las dos propiedades de red que tiene que configurar en una plantilla de Resource Manager para crear un clúster de HDInsight privado.

```json
networkProperties: {
    "resourceProviderConnection": "Inbound" | "Outbound",
    "privateLink": "Enabled" | "Disabled"
}
```

Para una plantilla completa con muchas de las características de Enterprise Security de HDInsight, incluido Private Link, consulte el artículo sobre la [plantilla de Enterprise Security de HDInsight](https://github.com/Azure-Samples/hdinsight-enterprise-security/tree/main/ESP-HIB-PL-Template).

### <a name="use-azure-powershell"></a>Uso de Azure PowerShell

Para usar PowerShell, consulte el ejemplo [aquí](/powershell/module/az.hdinsight/new-azhdinsightcluster#example-4--create-an-azure-hdinsight-cluster-with-relay-outbound-and-private-link-feature).

### <a name="use-azure-cli"></a>Uso de CLI de Azure
Para usar la CLI de Azure, consulte el ejemplo [aquí](/cli/azure/hdinsight#az_hdinsight_create-examples).

## <a name="next-steps"></a>Pasos siguientes

* [Enterprise Security Package para Azure HDInsight](enterprise-security-package.md)
* [Directrices generales e información de seguridad de la empresa en Azure HDInsight](./domain-joined/general-guidelines.md)
