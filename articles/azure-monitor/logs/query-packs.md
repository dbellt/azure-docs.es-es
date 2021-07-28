---
title: Paquetes de consultas en Azure Monitor
description: Los paquetes de consultas en Azure Monitor proporcionan una manera de compartir colecciones de consultas de registro en varias áreas de trabajo de Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/20/2021
ms.openlocfilehash: 59df6fa8624a19b611515551839076a511fe717f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110482963"
---
# <a name="query-packs-in-azure-monitor-logs-preview"></a>Paquetes de consultas en registros de Azure Monitor (versión preliminar)
Un paquete de consultas es un objeto de Resource Manager que actúa como contenedor para las consultas de registro en Azure Monitor y proporciona un método para guardar las consultas de registro y compartirlas entre varias áreas de trabajo y otros contextos en Log Analytics. 

## <a name="view-query-packs"></a>Visualización de paquetes de consultas
Puede ver y administrar los paquetes de consultas en Azure Portal desde el menú **Paquetes de consultas de Log Analytics**. Seleccione un paquete de consultas para ver y editar sus permisos. Consulte a continuación para obtener más información sobre cómo crear un paquete de consultas mediante la API.

[![Visualización de paquetes de consultas](media/query-packs/view-query-pack.png)](media/query-packs/view-query-pack.png#lightbox)

## <a name="permissions"></a>Permisos
Puede establecer los permisos en un paquete de consultas cuando lo vea en Azure Portal. Los usuarios requieren los siguientes permisos para usar paquetes de consultas:

- **Lector**: el usuario puede ver y ejecutar todas las consultas del paquete de consultas.
- **Colaborador**: el usuario puede modificar las consultas existentes y agregar nuevas al paquete de consultas.

## <a name="default-query-pack"></a>Paquete de consultas predeterminado
Un paquete de consultas, denominado **DefaultQueryPack**, se crea automáticamente en cada suscripción de un grupo de recursos denominado **LogAnalyticsDefaultResources** cuando se guarda la primera consulta. Puede crear consultas en este paquete de consultas o crear paquetes de consultas adicionales en función de sus requisitos.

## <a name="using-multiple-query-packs"></a>Uso de varios paquetes de consultas
El paquete de consultas predeterminado único será suficiente para que la mayoría de los usuarios guarde y reutilice las consultas. Sin embargo, hay motivos por los que quizá desee crear varios paquetes de consultas para los usuarios de su organización, incluida la carga de diferentes conjuntos de consultas en diferentes sesiones de Log Analytics y la especificación de permisos diferentes para diferentes colecciones de consultas. 

Al crear un nuevo paquete de consultas mediante la API, puede agregar etiquetas que clasifiquen las consultas según sus requisitos empresariales. Por ejemplo, podría etiquetar un paquete de consultas para relacionarlo con un departamento específico de la organización o con el nivel de gravedad de los problemas que las consultas incluidas deben solucionar. Esto le permite crear diferentes conjuntos de consultas destinados a diferentes conjuntos de usuarios y a distintas situaciones.

## <a name="query-pack-definition"></a>Definición del paquete de consultas
Cada paquete de consultas se define en un archivo JSON que incluye la definición de una o varias consultas. Cada consulta se representa mediante un bloque como se muestra a continuación:

```json
{
    "properties":
       {
        "displayName": "Query name that will be displayed in the UI",
        "description": "Query description that will be displayed in the UI",
        "body": "<<query text, standard KQL code>>",
        "related": {
            "categories": [
                "workloads"
            ],
            "resourceTypes": [
                "microsoft.insights/components"
            ],
            "solutions": [
                "logmanagement"
            ]
        },
        "tags": {
            "Tag1": [
                "Value1",
                "Value2"
            ]
        },
   }
}
```


## <a name="query-properties"></a>Propiedades de la consulta
Cada consulta del paquete de consultas tiene las siguientes propiedades.


| Propiedad | Descripción |
|:---|:---|
| DisplayName | Nombre para mostrar que aparece en Log Analytics para cada consulta. | 
| description | Descripción de la consulta que se muestra en Log Analytics para cada consulta. |
| body        | Consulta escrita en KQL. |
| cliente     | Categorías relacionadas, tipos de recursos y soluciones para la consulta. Se usa para que el usuario agrupe y filtre en Log Analytics a fin de localizar su consulta. Cada consulta puede tener hasta diez de cada tipo. Recupere los valores permitidos de https://api.loganalytics.io/v1/metadata?select=resourceTypes,solutions,categories. |
| etiquetas        | Etiquetas adicionales usadas por el usuario para ordenar y filtrar en Log Analytics. Cada etiqueta se agregará a Category, Resource Type y Solution al [agrupar y filtrar consultas](queries.md#finding-and-filtering-queries). |




## <a name="create-a-query-pack"></a>Creación de un paquete de consultas
La única manera de instalar actualmente un paquete de consultas es con la API REST. 

### <a name="create-token"></a>Creación del token
Necesita un token para la autenticación de la solicitud de API. Hay varios métodos para obtener un token, incluido el uso de **armclient**.

Primero, conéctese a Azure mediante el comando siguiente:

```
armclient login
```

A continuación, cree el token con el siguiente comando. El token se copia automáticamente en el Portapapeles para que pueda pegarlo en otra herramienta.

```
armclient token
```

### <a name="create-payload"></a>Creación de una carga
La carga de la solicitud es el JSON que define una o varias consultas y la ubicación donde se debe almacenar el paquete de consultas. El nombre del paquete de consultas se especifica en la solicitud de API que se describe en la sección siguiente.

```json
{
    "location": "eastus",
    "properties":
    {
        "displayName": "Query name that will be displayed in the UI",
        "description": "Query description that will be displayed in the UI",
        "body": "<<query text, standard KQL code>>",
        "related": {
            "categories": [
                "workloads"
            ],
            "resourceTypes": [
                "microsoft.insights/components"
            ],
            "solutions": [
                "logmanagement"
            ]
        },
        "tags": {
            "Tag1": [
                "Value1",
                "Value2"
            ]
        }
    }
}
```

### <a name="create-request"></a>Crear solicitud
Use la siguiente solicitud para crear un paquete de consultas mediante la API REST. La solicitud debe utilizar la autorización del token de portador. El tipo de contenido debe ser application/json.

```rest
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Insights/querypacks/my-query-pack?api-version=2019-09-01-preview
```

Use una herramienta que pueda enviar una solicitud de API REST, como Fiddler o Postman, para enviar la solicitud con la carga descrita en la sección anterior. El identificador de consulta se generará y devolverá en la carga útil. 

## <a name="update-a-query-pack"></a>Actualización de un paquete de consultas
Para actualizar un paquete de consultas, envíe la siguiente solicitud con una carga actualizada. Este comando requiere el identificador del paquete de consultas.

```rest
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-resource-group/providers/Microsoft.Insights/querypacks/my-query-pack/queries/query-id/?api-version=2019-09-01-preview
```

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Uso de consultas en Azure Monitor Log Analytics](queries.md) para ver cómo interactúan los usuarios con los paquetes de consultas en Log Analytics.