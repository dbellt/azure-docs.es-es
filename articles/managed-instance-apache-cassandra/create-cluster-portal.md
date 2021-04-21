---
title: 'Inicio rápido: Creación de un clúster de Azure Managed Instance for Apache Cassandra desde Azure Portal'
description: En este inicio rápido se muestra cómo crear un clúster de Azure Managed Instance for Apache Cassandra mediante Azure Portal.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 03/02/2021
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: e42f85bb79dcb1bfe14cacbbfda3576888b841c9
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107481335"
---
# <a name="quickstart-create-an-azure-managed-instance-for-apache-cassandra-cluster-from-the-azure-portal-preview"></a>Inicio rápido: Creación de un clúster de Azure Managed Instance for Apache Cassandra desde Azure Portal (versión preliminar)
 
Azure Managed Instance for Apache Cassandra proporciona operaciones automatizadas de implementación y escalado para centros de datos de Apache Cassandra de código abierto administrados, lo que permite acelerar los escenarios híbridos y reducir el mantenimiento continuo.

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este inicio rápido se muestra cómo usar Azure Portal para crear un clúster de Azure Managed Instance for Apache Cassandra.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-managed-instance-cluster"></a><a id="create-account"></a>Creación de un clúster de instancia administrada

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En la barra de búsqueda, escriba **Managed Instance for Apache Cassandra** y seleccione el resultado.

   :::image type="content" source="./media/create-cluster-portal/search-portal.png" alt-text="Búsqueda de Managed Instance for Apache Cassandra." lightbox="./media/create-cluster-portal/search-portal.png" border="true":::

1. Seleccione el botón **Create Managed Instance for Apache Cassandra cluster** (Crear clúster de Managed Instance for Apache Cassandra).

   :::image type="content" source="./media/create-cluster-portal/create-cluster.png" alt-text="Creación del clúster." lightbox="./media/create-cluster-portal/create-cluster.png" border="true":::

1. En el panel **Create Managed Instance for Apache Cassandra cluster** (Crear clúster de Managed Instance for Apache Cassandra), escriba los siguientes detalles:

   * **Suscripción**: en la lista desplegable, seleccione su suscripción de Azure.
   * **Grupo de recursos**: especifique si quiere crear un grupo de recursos o utilizar uno ya existente. Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. Para más información, consulte el artículo de introducción [Grupo de recursos de Azure](../azure-resource-manager/management/overview.md).
   * **Nombre del clúster**: escriba un nombre para el clúster.
   * **Ubicación**: lugar donde se implementará el clúster.
   * **SKU**: el tipo de SKU del clúster.
   * **Número de nodos**: número de nodos de un cluster. Estos nodos actúan como réplicas de los datos.
   * **Initial Cassandra admin password** (Contraseña inicial de administrador de Cassandra): la contraseña que se usa para crear el clúster.
   * **Confirm Cassandra admin password** (Confirmar contraseña de administrador de Cassandra): vuelva a escribir la contraseña.

    > [!NOTE]
    > Durante la versión preliminar pública, puede crear el clúster de instancia administrada en las regiones *Este de EE. UU., Oeste de EE. UU., Este de EE. UU. 2, Oeste de EE. UU. 2, Centro de EE. UU., Centro-sur de EE. UU., Norte de Europa, Oeste de Europa, Sudeste de Asia y Este de Australia*.

   :::image type="content" source="./media/create-cluster-portal/create-cluster-page.png" alt-text="Rellenar el formulario de creación del clúster." lightbox="./media/create-cluster-portal/create-cluster-page.png" border="true":::

1. A continuación, seleccione la pestaña **Redes**.

1. En el panel **Redes**, elija el nombre de **Red virtual** y **Subred**. Puede seleccionar una red virtual existente o crear una.

   :::image type="content" source="./media/create-cluster-portal/networking.png" alt-text="Configuración de los detalles de la red." lightbox="./media/create-cluster-portal/networking.png" border="true":::

    > [!NOTE]
    > La implementación de Azure Managed Instance for Apache Cassandra requiere acceso a Internet. En entornos con acceso limitado a Internet se produce un error de implementación. Asegúrese de no bloquear el acceso a los siguientes servicios de Azure que son esenciales para que las instancias administradas de Cassandra funcionen correctamente:
    > - Azure Storage
    > - Azure KeyVault
    > - Conjuntos de escalado de máquinas virtuales de Azure
    > - Supervisión de Azure
    > - Azure Active Directory
    > - Azure Security

1. Si ha creado una red virtual en el último paso, vaya al paso 8. Si seleccionó una red virtual existente, antes de crear el clúster, debe aplicar algunos permisos especiales a la red virtual y a la subred. Para ello, use el comando `az role assignment create` y reemplace `<subscription ID>`, `<resource group name>` y `<VNet name>` con los valores adecuados:

   ```azurecli-interactive
   az role assignment create --assignee a232010e-820c-4083-83bb-3ace5fc29d0b --role 4d97b98b-1d4f-4787-a291-c67834d212e7 --scope /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.Network/virtualNetworks/<VNet name>
   ```

   > [!NOTE]
   > Los valores `assignee` y `role` del comando anterior son valores fijos. Escriba estos valores exactamente como se mencionó en el comando. Si no lo hace, se producirán errores al crear el clúster. Si se producen errores al ejecutar este comando, es posible que no tenga permisos para ejecutarlo. Para obtenerlos, póngase en contacto con el administrador.

1. Ahora que ha terminado con la red, haga clic en **Revisar y crear** > **Crear**.

    > [!NOTE]
    > El clúster puede tardar hasta 15 minutos en crearse.

   :::image type="content" source="./media/create-cluster-portal/review-create.png" alt-text="Revisión del resumen para crear el clúster." lightbox="./media/create-cluster-portal/review-create.png" border="true":::


1. Una vez finalizada la implementación, compruebe el grupo de recursos para ver el clúster de instancia administrada recién creado:

   :::image type="content" source="./media/create-cluster-portal/managed-instance.png" alt-text="Página de información general una vez creado el clúster." lightbox="./media/create-cluster-portal/managed-instance.png" border="true":::

1. Para examinar los nodos del clúster, vaya al panel Red virtual que ha usado para crear el clúster y abra el panel **Información general** para verlos:

   :::image type="content" source="./media/create-cluster-portal/resources.png" alt-text="Visualización de los recursos del clúster." lightbox="./media/create-cluster-portal/resources.png" border="true":::


## <a name="connecting-to-your-cluster"></a>Conectarse a los clúster

Azure Managed Instance for Apache Cassandra no crea nodos con direcciones IP públicas, así que para conectarse al clúster de Cassandra recién creado, deberá crear otro recurso dentro de la red virtual. Puede ser una aplicación o una máquina virtual con la herramienta de consulta de código abierto de Apache, [CQLSH](https://cassandra.apache.org/doc/latest/tools/cqlsh.html), instalada. Puede usar una [plantilla](https://azure.microsoft.com/resources/templates/101-vm-simple-linux/) para implementar una máquina virtual de Ubuntu. Cuando se implemente, use SSH para conectarse a la máquina e instale CQLSH con los siguientes comandos:

```bash
# Install default-jre and default-jdk
sudo apt update
sudo apt install openjdk-8-jdk openjdk-8-jre

# Install the Cassandra libraries in order to get CQLSH:
echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra

# Export the SSL variables:
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false

# Connect to CQLSH (replace <IP> with the private IP addresses of the nodes in your Datacenter):
host=("<IP>" "<IP>" "<IP>")
cqlsh $host 9042 -u cassandra -p cassandra --ssl
```

## <a name="troubleshooting"></a>Solución de problemas

Si se produce un error al aplicar permisos a la red virtual, por ejemplo, se indica que *no se encuentra el usuario o la entidad de servicio en la base de datos de grafos para "e5007d2c-4b13-4a74-9b6a-605d99f03501"* , puede aplicar el mismo permiso manualmente desde Azure Portal. Para aplicar permisos desde el portal, vaya al panel **Control de acceso (IAM)** de la red virtual existente y agregue una asignación de roles para "Azure Cosmos DB" al rol "Administrador de red". Si aparecen dos entradas al buscar "Azure Cosmos DB", agregue las dos entradas, tal como se muestra en la siguiente imagen: 

   :::image type="content" source="./media/create-cluster-cli/apply-permissions.png" alt-text="Permisos de aplicación" lightbox="./media/create-cluster-cli/apply-permissions.png" border="true":::

> [!NOTE] 
> La asignación de roles de Azure Cosmos DB se usa solo con fines de implementación. Azure Managed Instance for Apache Cassandra no tiene dependencias de back-end en Azure Cosmos DB.   

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando este clúster de instancia administrada, elimínelo mediante los siguientes pasos:

1. En el menú de la izquierda de Azure Portal, seleccione **Grupos de recursos**.
1. En la lista, seleccione el grupo de recursos que creó para este inicio rápido.
1. En el panel **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
1. En la ventana siguiente, escriba el nombre del grupo de recursos que desea eliminar y, después, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear un clúster de Azure Managed Instance for Apache Cassandra mediante Azure Portal. Ahora puede empezar a trabajar con el clúster:

> [!div class="nextstepaction"]
> [Implementación de un clúster de Apache Spark administrado con Azure Databricks](deploy-cluster-databricks.md)
