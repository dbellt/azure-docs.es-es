---
title: 'Inicio rápido: implementación de un clúster de Apache Spark administrado con Azure Databricks'
description: En este inicio rápido se muestra cómo implementar un clúster de Apache Spark administrado con Azure Databricks mediante Azure Portal.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 06/02/2021
ms.openlocfilehash: 257055110bffac10c92d87820bcfadff2b208b3b
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111538042"
---
# <a name="quickstart-deploy-a-managed-apache-spark-cluster-preview-with-azure-databricks"></a>Inicio rápido: implementación de un clúster de Apache Spark administrado (versión preliminar) con Azure Databricks

Azure Managed Instance for Apache Cassandra proporciona operaciones automatizadas de implementación y escalado para centros de datos de Apache Cassandra de código abierto administrados, lo que permite acelerar los escenarios híbridos y reducir el mantenimiento continuo.

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este inicio rápido se muestra cómo usar Azure Portal para crear un clúster de Apache Spark totalmente administrado en Azure Virtual Network para el clúster de Azure Managed Instance for Apache Cassandra. Creará el clúster de Spark en Azure Databricks. Posteriormente, puede crear o adjuntar cuadernos al clúster, leer datos de diferentes orígenes de datos y analizar la información.

Puede obtener más información con instrucciones detalladas sobre la [implementación de Azure Databricks en Azure Virtual Network (inserción de red virtual)](/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject).

## <a name="create-an-azure-databricks-cluster"></a>Crear un clúster de Azure Databricks

Siga estos pasos para crear un clúster de Azure Databricks en una red virtual que tenga Azure Managed Instance for Apache Cassandra:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En el panel de navegación izquierdo, busque **Grupos de recursos** y navegue hasta el grupo de recursos que contiene la red virtual donde se implementa la instancia administrada.

1. Abra el recurso **Red virtual** y tome nota del **espacio de direcciones**:

    :::image type="content" source="./media/deploy-cluster-databricks/virtual-network-address-space.png" alt-text="Obtenga el espacio de direcciones de su red virtual." border="true":::

1. En el grupo de recursos, seleccione **Agregar** y busque **Azure Databricks** en el campo de búsqueda:

    :::image type="content" source="./media/deploy-cluster-databricks/databricks.png" alt-text="Busque Azure Databricks." border="true":::

1. Seleccione **Crear** para crear una cuenta de Azure Databricks:

    :::image type="content" source="./media/deploy-cluster-databricks/databricks-create.png" alt-text="Cree una cuenta de Azure Databricks." border="true":::

1. Rellene los siguientes valores:

   * **Nombre de área de trabajo**: proporcione un nombre para el área de trabajo de Databricks.
   * **Región** : asegúrese de seleccionar la misma región que su red virtual.
   * **Plan de tarifa**: elija entre Estándar, Premium o Evaluación gratuita. Para más información sobre estos planes, consulte la [página de precios de Databricks](https://azure.microsoft.com/pricing/details/databricks/).

    :::image type="content" source="./media/deploy-cluster-databricks/select-name.png" alt-text="Especifique el nombre del área de trabajo, la región y el plan de tarifa para la cuenta de Databricks." border="true":::

1. A continuación, seleccione la pestaña **Redes** y rellene la información siguiente:

   * **Implementar área de trabajo de Azure Databricks en una red virtual (VNet) propia**: seleccione **Sí**.
   * **Red virtual** : en la lista desplegable, elija la red virtual donde se encuentra la instancia administrada.
   * **Nombre de subred pública**: escriba un nombre para la subred pública.
   * **Intervalo de CIDR de subred pública**: escriba un intervalo IP para la subred pública.
   * **Nombre de subred privada** : escriba un nombre para la subred privada.
   * **Intervalo de CIDR de subred privada**: escriba un intervalo IP para la subred privada.

   Para evitar conflictos de intervalos, asegúrese de seleccionar intervalos superiores. Si es necesario, use una [calculadora de subred visual](https://www.fryguy.net/wp-content/tools/subnets.html) para dividir los intervalos:

   :::image type="content" source="./media/deploy-cluster-databricks/subnet-calculator.png" alt-text="Uso de la calculadora de subred de la red virtual." border="true":::

   En la captura de pantalla siguiente se muestran detalles de ejemplo en el panel de redes:

   :::image type="content" source="./media/deploy-cluster-databricks/subnets.png" alt-text="Especificación de los nombres de las subredes pública y privada." border="true":::

1. Seleccione **Revisar y crear** y, a continuación, **Crear** para implementar el área de trabajo.

1. Una vez que se ha creado, seleccione **Iniciar área de trabajo**.

1. Se le redirigirá al portal de Azure Databricks. En el portal, seleccione **Nuevo clúster**.

1. En el panel **Nuevo clúster**, acepte los valores predeterminados para todos los campos distintos a los campos siguientes:

   * **Nombre del clúster**: escriba un nombre para el clúster.
   * **Versión de Databricks Runtime:** se recomienda seleccionar Databricks Runtime versión 7.5 o posterior, para la compatibilidad con Spark 3.x. 

    :::image type="content" source="../cosmos-db/media/cassandra-migrate-cosmos-db-databricks/databricks-runtime.png" alt-text="Seleccione la versión de Databricks Runtime y el clúster de Spark." border="true":::

1. Expanda **Opciones avanzadas** y agregue la configuración siguiente. Asegúrese de reemplazar las direcciones IP y las credenciales del nodo:

    ```java
    spark.cassandra.connection.host <node1 IP>,<node 2 IP>, <node IP>
    spark.cassandra.auth.password cassandra
    spark.cassandra.connection.port 9042
    spark.cassandra.auth.username cassandra
    spark.cassandra.connection.ssl.enabled true
    ```

1. Agregue la biblioteca de conectores de Cassandra de Apache Spark a su clúster para conectarse a los puntos de conexión nativos y de Cassandra de Azure Cosmos DB. En el clúster, seleccione **Libraries** > **Install New** > **Maven** (Bibliotecas > Instalar nueva > Maven) y, después, agregue `com.datastax.spark:spark-cassandra-connector-assembly_2.12:3.0.0` en las coordenadas de Maven.

:::image type="content" source="../cosmos-db/media/cassandra-migrate-cosmos-db-databricks/databricks-search-packages.png" alt-text="Captura de pantalla que muestra la búsqueda de paquetes Maven en Databricks":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando este clúster de instancia administrada, elimínelo mediante los siguientes pasos:

1. En el menú de la izquierda de Azure Portal, seleccione **Grupos de recursos**.
1. En la lista, seleccione el grupo de recursos que creó para este inicio rápido.
1. En el panel **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
3. En la ventana siguiente, escriba el nombre del grupo de recursos que desea eliminar y, después, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido se muestra cómo usar Azure Portal para crear un clúster de Apache Spark totalmente administrado en la red virtual del clúster de Azure Managed Instance for Apache Cassandra. A continuación, obtenga información sobre cómo administrar los recursos del centro de datos y el clúster: 

> [!div class="nextstepaction"]
> [Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure](manage-resources-cli.md)

