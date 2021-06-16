---
title: Creación de clústeres de HBase en Azure Virtual Network (Azure)
description: Introducción al uso de HBase en HDInsight de Azure Aprenda a crear clústeres de HBase de HDInsight en Azure Virtual Network.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/23/2019
ms.openlocfilehash: 31c6195ac83d2b1fb9cfadf30876cca4cc41a57f
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112005868"
---
# <a name="create-apache-hbase-clusters-on-hdinsight-in-azure-virtual-network"></a>Creación de clústeres de Apache HBase en HDInsight en Azure Virtual Network

Aprenda a crear clústeres de Apache HBase de Azure HDInsight en [Azure Virtual Network](https://azure.microsoft.com/services/virtual-network/).

Con la integración de Virtual Network, los clústeres de Apache HBase se pueden implementar en la misma red virtual que sus aplicaciones para que estas puedan comunicarse directamente con HBase. Entre las ventajas se incluye lo siguiente:

* Conectividad directa entre la aplicación web y los nodos del clúster de HBase, lo cual permite la comunicación mediante las API de llamada a procedimiento remoto (RPC) de Java de HBase.
* Rendimiento mejorado gracias a que el tráfico ya no tiene que examinar varias puertas de enlace y equilibradores de carga.
* La posibilidad de procesar información confidencial de una manera segura sin exponer un extremo público.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-apache-hbase-cluster-into-virtual-network"></a>Creación de un clúster de Apache HBase en Virtual Network

En esta sección, crea un clúster de Apache HBase basado en Linux con la cuenta de Azure Storage dependiente en Azure Virtual Network mediante una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md). Para conocer otros métodos de creación de clústeres y la descripción de la configuración, consulte [Creación de clústeres de Hadoop basados en Windows en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Para más información acerca de cómo utilizar una plantilla para crear clústeres de Apache Hadoop en HDInsight, consulte [Creación de clústeres de Apache Hadoop en HDInsight con plantillas de Resource Manager](../hdinsight-hadoop-create-linux-clusters-arm-templates.md).

> [!NOTE]  
> Algunas propiedades se han codificado de forma rígida en la plantilla. Por ejemplo:
>
> * **Ubicación**: este de EE. UU. 2
> * **Versión del clúster**: 3.6
> * **Número de nodos de trabajo en el clúster**: 2
> * **Nombre de la cuenta de almacenamiento predeterminada**: una cadena única
> * **Nombre de la red virtual**: CLUSTERNAME-vnet
> * **Espacio de direcciones de red virtual**: 10.0.0.0/16
> * **Nombre de subred**: subnet1
> * **Rango de direcciones de subred**: 10.0.0.0/24
>
> `CLUSTERNAME` se reemplaza por el nombre de clúster que proporcione al utilizar la plantilla.

1. Seleccione la imagen siguiente para abrir la plantilla en Azure Portal. La plantilla se encuentra en las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/hdinsight-hbase-linux-vnet/).

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.hdinsight%2Fhdinsight-hbase-linux-vnet%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-provision-vnet/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

1. En el cuadro de diálogo de **Implementación personalizada**, seleccione **Editar plantilla**.

1. En la línea 165, cambie el valor `Standard_A3` a `Standard_A4_V2`. Después, seleccione **Guardar**.

1. Complete la plantilla restante con la siguiente información:

    |Propiedad |Valor |
    |---|---|
    |Subscription|seleccione una suscripción de Azure usada para crear el clúster de HDInsight, la cuenta de Storage dependiente y Azure Virtual Network.|
    Resource group|seleccione **Crear nuevo** y especifique un nuevo nombre al grupo de recursos.|
    |Location|Seleccione una ubicación para el grupo de recursos.|
    |Cluster Name|escriba un nombre para el clúster de Hadoop que se va a crear.|
    |Nombre de usuario y contraseña de registro del clúster|El nombre de usuario predeterminado es **admin**. Indique una contraseña.|
    |Nombre de usuario y contraseña de SSH|El nombre de usuario predeterminado es **sshuser**.  Indique una contraseña.|

    Seleccione **Acepto los términos y condiciones indicados anteriormente**.

1. Seleccione **Comprar**. Tarda aproximadamente 20 minutos en crear un clúster. Una vez creado el clúster, puede seleccionarlo en el portal para abrirlo.

Cuando termine el artículo, puede eliminar el clúster. Con HDInsight, los datos se almacenan en Azure Storage, por lo que puede eliminar un clúster de forma segura cuando no se esté usando. También se le cobrará por un clúster de HDInsight aunque no se esté usando. Como en muchas ocasiones los cargos por el clúster son mucho más elevados que los cargos por el almacenamiento, desde el punto de vista económico tiene sentido eliminar clústeres cuando no se estén usando. Para ver instrucciones sobre cómo eliminar un clúster, consulte [Administración de clústeres de Apache Hadoop en HDInsight mediante Azure Portal](../hdinsight-administer-use-portal-linux.md#delete-clusters).

Para comenzar a trabajar con el nuevo clúster de HBase, utilice los procedimientos que encontrará en [Introducción al uso de Apache HBase con Apache Hadoop en HDInsight](./apache-hbase-tutorial-get-started-linux.md).

## <a name="connect-to-the-apache-hbase-cluster-using-apache-hbase-java-rpc-apis"></a>Conexión al clúster de Apache HBase mediante las API de RPC de Java de Apache HBase

### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree una máquina virtual de infraestructura como servicio (IaaS) en la misma red virtual de Azure y la misma subred. Para obtener instrucciones sobre cómo crear una nueva máquina virtual de IaaS consulte el tutorial sobre la [creación de una máquina virtual que ejecuta Windows Server](../../virtual-machines/windows/quick-create-portal.md). Al seguir los pasos descritos en este documento, tiene que usar los valores siguientes para la configuración de red:

* **Red virtual**: CLUSTERNAME-vnet
* **Subred**: subnet1

> [!IMPORTANT]  
> Reemplace `CLUSTERNAME` por el nombre que utilizó al crear el clúster de HDInsight en los pasos anteriores.

Con estos valores, se colocará la máquina virtual en la misma red virtual y subred que el clúster de HDInsight. Esta configuración les permitirá comunicarse directamente entre sí. No hay una manera de crear un clúster de HDInsight con un nodo perimetral vacío. El nodo perimetral puede usarse para administrar el clúster.  Para obtener más información, consulte [Uso de nodos perimetrales vacíos en HDInsight](../hdinsight-apps-use-edge-node.md).

### <a name="obtain-fully-qualified-domain-name"></a>Obtención del nombre de dominio completo

Cuando use una aplicación Java para conectarse a HBase en modo remoto, debe usar el nombre de dominio completo (FQDN). Para determinarlo, debe obtener el sufijo DNS específico de la conexión del clúster de HBase. Para ello, use uno de los siguientes métodos:

* Utilice un explorador web para realizar una llamada [Apache Ambari](https://ambari.apache.org/):

    Vaya a `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts?minimal_response=true`. Devuelve un archivo JSON con los sufijos DNS.

* Utilice el sitio web de Ambari:

    1. Vaya a `https://CLUSTERNAME.azurehdinsight.net`.
    2. Seleccione **Hosts** en el menú superior.

* Use Curl para realizar llamadas REST:

    ```bash
    curl -u <username>:<password> -k https://CLUSTERNAME.azurehdinsight.net/ambari/api/v1/clusters/CLUSTERNAME.azurehdinsight.net/services/hbase/components/hbrest
    ```

En los datos de notación de objetos JavaScript (JSON) devueltos, busque la entrada "host_name". Contiene el nombre de dominio completo (FQDN) de los nodos del clúster. Por ejemplo:

```json
"host_name" : "hn*.hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net"
```

La parte del nombre de dominio que comienza con el nombre del clúster es el sufijo DNS. Por ejemplo, `hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net`.

### <a name="verify-communication-inside-virtual-network"></a>Comprobar la comunicación dentro de la red virtual

Para comprobar que la máquina virtual puede comunicarse con el clúster de HBase, use el comando `ping headnode0.<dns suffix>` desde la máquina virtual. Por ejemplo, `ping hn*.hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net`.

Para usar esta información en una aplicación Java, puede seguir los pasos que se indican en [Uso de Apache Maven para crear aplicaciones Java que utilicen Apache HBase con HDInsight (Hadoop)](./apache-hbase-build-java-maven-linux.md) para crear una aplicación. Para que la aplicación se conecte a un servidor HBase remoto, modifique el archivo **hbase-site.xml** de este ejemplo para que use el FQDN de ZooKeeper. Por ejemplo:

```xml
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>zookeeper0.<dns suffix>,zookeeper1.<dns suffix>,zookeeper2.<dns suffix></value>
</property>
```

> [!NOTE]  
> Para más información sobre la resolución de nombres en redes virtuales de Azure, incluido el uso de su propio servidor DNS, vea [Resolución de nombres (DNS)](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear un clúster de Apache HBase. Para obtener más información, consulte:

* [Introducción a HDInsight](../hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Uso de nodos perimetrales vacíos en HDInsight](../hdinsight-apps-use-edge-node.md)
* [Configuración de la replicación de Apache HBase en HDInsight](apache-hbase-replication.md)
* [Creación de clústeres de Apache Hadoop en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
* [Introducción al uso de Apache HBase con Apache Hadoop en HDInsight](./apache-hbase-tutorial-get-started-linux.md)
* [Información general sobre redes virtuales](../../virtual-network/virtual-networks-overview.md)
