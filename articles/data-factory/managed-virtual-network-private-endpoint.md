---
title: Red virtual administrada y puntos de conexión privados administrados
description: Obtenga información sobre la red virtual administrada y los puntos de conexión privados administrados en Azure Data Factory.
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.topic: conceptual
ms.custom:
- seo-lt-2019
- references_regions
ms.date: 07/15/2020
ms.openlocfilehash: 34211feb34b3e2ec21b9bc0e6a180df0febd8c76
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987945"
---
# <a name="azure-data-factory-managed-virtual-network-preview"></a>Red virtual administrada de Azure Data Factory (versión preliminar)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica la red virtual administrada y los puntos de conexión privados administrados en Azure Data Factory.


## <a name="managed-virtual-network"></a>Red virtual administrada

Al crear una instancia de Azure Integration Runtime (IR) en la red virtual (VNET) administrada de Azure Data Factory, el entorno de ejecución de integración se aprovisionará con la red virtual administrada y aprovechará los puntos de conexión privados para conectarse de forma segura a almacenes de datos compatibles. 

La creación de una instancia de Azure IR dentro de una red virtual administrada garantiza que el proceso de integración de datos esté aislado y protegido. 

Ventajas del uso de una red virtual administrada:

- Con una red virtual administrada puede dejar que Azure Data Factory se ocupe de la pesada tarea de administrar la red virtual. No es necesario crear una subred para Azure Integration Runtime que podría usar muchas direcciones IP privadas de la red virtual y requeriría una planeación previa de la infraestructura de la red. 
- No se necesita un conocimiento profundo de las redes de Azure para realizar integraciones de datos de forma segura. En su lugar, empezar a utilizar ETL seguro es mucho más sencillo para los ingenieros de datos. 
- La red virtual administrada, junto con los puntos de conexión privados administrados, sirven de protección contra la filtración de datos. 

> [!IMPORTANT]
>Actualmente, la red virtual administrada solo se admite en la misma región que la región de Azure Data Factory.
 

![Arquitectura de la red virtual de Azure Data Factory](./media/managed-vnet/managed-vnet-architecture-diagram.png)

## <a name="managed-private-endpoints"></a>Puntos de conexión privados administrados

Los puntos de conexión privados administrados se crean en la red virtual administrada de Azure Data Factory y establecen un vínculo privado a recursos de Azure. Azure Data Factory administra estos puntos de conexión privados en su nombre. 

![Nuevo punto de conexión privado administrado](./media/tutorial-copy-data-portal-private/new-managed-private-endpoint.png)

Azure Data Factory admite vínculos privados. El vínculo privado le permite acceder a servicios de Azure (PaaS) (como Azure Storage, Azure Cosmos DB, Azure Synapse Analytics).

Cuando se usa un vínculo privado, el tráfico entre los almacenes de datos y la red virtual administrada atraviesa completamente la red troncal de Microsoft. Private Link protege frente a los riesgos de la filtración de datos. El vínculo privado a un recurso se establece mediante la creación de un punto de conexión privado.

El punto de conexión privado usa una dirección IP privada en la red virtual administrada para incorporar el servicio de manera eficaz a la red virtual. Los puntos de conexión privados se asignan a un recurso específico de Azure, no a todo el servicio. Los clientes pueden limitar la conectividad a un recurso específico aprobado por su organización. Más información sobre [vínculos privados y puntos de conexión privados](../private-link/index.yml).

> [!NOTE]
> Se recomienda crear puntos de conexión privados administrados para conectarse a todos los orígenes de datos de Azure. 
 
> [!WARNING]
> Si un almacén de datos de PaaS (Blob, ADLS Gen2 o Azure Synapse Analytics) tiene un punto de conexión privado que ya se ha creado, aunque permita el acceso desde todas las redes, ADF solo podrá acceder a dicho almacén mediante un punto de conexión privado administrado. Asegúrese de crear un punto de conexión privado en estos escenarios. 

Una conexión de punto de conexión privado se crea en un estado "pendiente" cuando se crea un punto de conexión privado administrado en Azure Data Factory. Se inicia un flujo de trabajo de aprobación. El propietario del recurso de vínculo privado es responsable de aprobar o rechazar la conexión.

![Administración de un punto de conexión privado](./media/tutorial-copy-data-portal-private/manage-private-endpoint.png)

Si el propietario aprueba la conexión, se establece el vínculo privado. De lo contrario, no se establece. En cualquier caso, el punto de conexión privado administrado se actualizará con el estado de la conexión.

![Punto de conexión privado administrado aprobado](./media/tutorial-copy-data-portal-private/approve-private-endpoint.png)

Solo un punto de conexión privado administrado en un estado aprobado puede enviar tráfico a un recurso de vínculo privado determinado.

## <a name="interactive-authoring"></a>Creación interactiva
Entre las funcionalidades de la creación interactiva se incluyen probar la conexión, examinar la lista de carpetas y la lista de tablas, obtener esquemas y obtener una vista previa de los datos. Puede habilitar la creación interactiva al crear o editar una instancia de Azure Integration Runtime que se encuentre en una red virtual administrada por ADF. El servicio de back-end asignará previamente el proceso para las funcionalidades de creación interactiva. De lo contrario, el proceso se asignará cada vez que se realice cualquier operación interactiva, lo que tardará más tiempo. El período de vida (TTL) para la creación interactiva es de 60 minutos, lo que significa que se deshabilitará automáticamente después de 60 minutos de la última operación de creación interactiva.

![Creación interactiva](./media/managed-vnet/interactive-authoring.png)

## <a name="create-managed-virtual-network-via-azure-powershell"></a>Creación de una red virtual administrada mediante Azure PowerShell
```powershell
$subscriptionId = ""
$resourceGroupName = ""
$factoryName = ""
$managedPrivateEndpointName = ""
$integrationRuntimeName = ""
$apiVersion = "2018-06-01"
$privateLinkResourceId = ""

$vnetResourceId = "subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.DataFactory/factories/${factoryName}/managedVirtualNetworks/default"
$privateEndpointResourceId = "subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.DataFactory/factories/${factoryName}/managedVirtualNetworks/default/managedprivateendpoints/${managedPrivateEndpointName}"
$integrationRuntimeResourceId = "subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.DataFactory/factories/${factoryName}/integrationRuntimes/${integrationRuntimeName}"

# Create managed Virtual Network resource
New-AzResource -ApiVersion "${apiVersion}" -ResourceId "${vnetResourceId}"

# Create managed private endpoint resource
New-AzResource -ApiVersion "${apiVersion}" -ResourceId "${privateEndpointResourceId}" -Properties @{
        privateLinkResourceId = "${privateLinkResourceId}"
        groupId = "blob"
    }

# Create integration runtime resource enabled with VNET
New-AzResource -ApiVersion "${apiVersion}" -ResourceId "${integrationRuntimeResourceId}" -Properties @{
        type = "Managed"
        typeProperties = @{
            computeProperties = @{
                location = "AutoResolve"
                dataFlowProperties = @{
                    computeType = "General"
                    coreCount = 8
                    timeToLive = 0
                }
            }
        }
        managedVirtualNetwork = @{
            type = "ManagedVirtualNetworkReference"
            referenceName = "default"
        }
    }

```

## <a name="limitations-and-known-issues"></a>Limitaciones y problemas conocidos
### <a name="supported-data-sources"></a>Orígenes de datos compatibles
Los orígenes de datos siguientes pueden establecer una conexión a través de un vínculo privado desde la red virtual administrada de ADF.
- Azure Blob Storage (sin incluir la cuenta de almacenamiento V1)
- Azure Table Storage (sin incluir la cuenta de almacenamiento V1)
- Azure Files (sin incluir la cuenta de almacenamiento V1)
- Azure Data Lake Gen2
- Azure SQL Database (sin incluir Azure SQL Managed Instance)
- Azure Synapse Analytics
- Azure CosmosDB SQL
- Azure Key Vault
- Servicio Azure Private Link
- Azure Search
- Azure Database for MySQL
- Azure Database for PostgreSQL
- Azure Database for MariaDB

### <a name="azure-data-factory-managed-virtual-network-is-available-in-the-following-azure-regions"></a>La característica Managed Virtual Network de Azure Data Factory está disponible en las siguientes regiones de Azure:
- Este de Australia
- Sudeste de Australia
- Sur de Brasil
- Centro de Canadá
- Este de Canadá
- Centro de la India
- Centro de EE. UU.
- Este de EE. UU.
- Este de EE. UU. 2
- Centro de Francia
- Japón Oriental
- Japón Occidental
- Centro de Corea del Sur
- Norte de Europa
- Este de Noruega
- Norte de Sudáfrica
- Centro-sur de EE. UU.
- Sudeste de Asia
- Norte de Suiza
- Norte de Emiratos Árabes Unidos
- Sur de Reino Unido
- Oeste de Reino Unido
- Centro-Oeste de EE. UU.
- Oeste de Europa
- Oeste de EE. UU.
- Oeste de EE. UU. 2


### <a name="outbound-communications-through-public-endpoint-from-adf-managed-virtual-network"></a>Comunicaciones salientes a través del punto de conexión público desde la red virtual administrada de ADF
- Se abren todos los puertos para las comunicaciones salientes.
- Azure Storage y Azure Data Lake Gen2 no pueden establecer una conexión a través del punto de conexión público desde la red virtual administrada de ADF.

### <a name="linked-service-creation-of-azure-key-vault"></a>Creación de un servicio vinculado para Azure Key Vault 
- Cuando se crea un servicio vinculado para Azure Key Vault, no existe ninguna referencia a Azure Integration Runtime. Por este motivo, no se puede generar un punto de conexión privado cuando se crea un servicio vinculado para Azure Key Vault. Sin embargo, si se crea un servicio vinculado para almacenes de datos que hace referencia a un servicio vinculado de Azure Key Vault y dicho servicio vinculado hace referencia a Azure Integration Runtime con la característica Managed Virtual Network habilitada, puede crear un punto de conexión privado para el servicio vinculado de Azure Key Vault durante la creación. 
- La operación **Prueba de conexión** del servicio vinculado de Azure Key Vault solo valida el formato de la dirección URL, pero no realiza ninguna operación de red.
- La columna **Using private endpoint** (Uso de punto de conexión privado) siempre se muestra en blanco, incluso si crea un punto de conexión privado para Azure Key Vault.
![Punto de conexión privado para AKV](./media/managed-vnet/akv-pe.png)

## <a name="next-steps"></a>Pasos siguientes

- Tutorial: [Creación de una canalización de copia mediante la red virtual administrada y los puntos de conexión privados](tutorial-copy-data-portal-private.md) 
- Tutorial: [Creación de una canalización de flujo de datos de asignación mediante la red virtual administrada y los puntos de conexión privados](tutorial-data-flow-private.md)