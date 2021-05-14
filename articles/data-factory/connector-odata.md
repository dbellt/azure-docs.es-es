---
title: Copia de datos de orígenes OData mediante Azure Data Factory
description: Obtenga información sobre cómo copiar datos desde orígenes OData a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/30/2021
ms.author: jianleishen
ms.openlocfilehash: 91d1206bcaf80d054c6adde1ee5a23456083b97b
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485196"
---
# <a name="copy-data-from-an-odata-source-by-using-azure-data-factory"></a>Copia de datos desde un origen OData mediante Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-odata-connector.md)
> * [Versión actual](connector-odata.md)

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos con un origen OData como origen o destino. El artículo se basa en [Actividad de copia en Azure Data Factory](copy-activity-overview.md), en el que se ofrece información general acerca de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de OData es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de un origen OData a cualquier almacén de datos receptor admitido. Para obtener una lista de almacenes de datos que la actividad de copia admite como orígenes y receptores, consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).

En concreto, este conector OData admite las siguientes funcionalidades:

- La versión 3.0 y 4.0 de OData.
- Copiar datos mediante el uso de una de las autenticaciones siguientes: **Anónima**, **Básica**, **Windows** y **Entidad de servicio de AAD**.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que puede usar para definir entidades de Data Factory específicas del conector de OData.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con un servicio vinculado de OData:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **OData**. |Sí |
| url | Dirección URL raíz del servicio de OData. |Sí |
| authenticationType | Tipo de autenticación que se usa para conectarse al origen de OData. Los valores permitidos son **Anónima**, **Básica**, **Windows** y **AadServicePrincipal**. No se admiten usuarios basados en OAuth. Además, puede configurar encabezados de autenticación en la propiedad `authHeader`.| Sí |
| authHeaders | Encabezados de solicitud HTTP adicionales para la autenticación.<br/> Por ejemplo, para usar la autenticación de clave de API, puede seleccionar el tipo de autenticación "Anónima" y especificar la clave de API en el encabezado. | No |
| userName | Especifique **userName** si se usa la autenticación Básica o de Windows. | No |
| password | Especifique la **contraseña** de la cuenta de usuario que se especificó para **userName**. Marque este campo como de tipo **SecureString** para almacenarlo de forma segura en Data Factory. También puede [hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | No |
| servicePrincipalId | Especifique el identificador de cliente de la aplicación de Azure Active Directory. | No |
| aadServicePrincipalCredentialType | Especifique el tipo de credencial que se usará para la autenticación de entidad de servicio. Los valores permitidos son: `ServicePrincipalKey` o `ServicePrincipalCert`. | No |
| servicePrincipalKey | Especifique la clave de la aplicación de Azure Active Directory. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. | No |
| servicePrincipalEmbeddedCert | Especifique el certificado codificado en base64 de la aplicación registrada en Azure Active Directory. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. | No |
| servicePrincipalEmbeddedCertPassword | Especifique la contraseña del certificado si el certificado está protegido por una. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**.  | No|
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Para recuperarla, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | No |
| aadResourceId | Especifique el recurso de AAD para el cual solicita autorización.| No |
| azureCloudType | Para la autenticación de la entidad de servicio, especifique el tipo de entorno de nube de Azure en el que está registrada la aplicación de AAD. <br/> Los valores permitidos son **AzurePublic**, **AzureChina**, **AzureUsGovernment** y **AzureGermany**. De forma predeterminada, se usa el entorno de nube de la factoría de datos. | No |
| connectVia | Instancia de [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

**Ejemplo 1: Uso de autenticación anónima**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo 2: Uso de autenticación básica**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "Basic",
            "userName": "<user name>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo 3: Uso de la autenticación de Windows**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "Windows",
            "userName": "<domain>\\<user>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo 4: Uso de la autenticación de la clave de entidad de servicio**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "AadServicePrincipal",
            "servicePrincipalId": "<service principal id>",
            "aadServicePrincipalCredentialType": "ServicePrincipalKey",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource URL>"
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}
```

**Ejemplo 5: Uso de la autenticación de certificados de entidad de servicio**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "AadServicePrincipal",
            "servicePrincipalId": "<service principal id>",
            "aadServicePrincipalCredentialType": "ServicePrincipalCert",
            "servicePrincipalEmbeddedCert": { 
                "type": "SecureString", 
                "value": "<base64 encoded string of (.pfx) certificate data>"
            },
            "servicePrincipalEmbeddedCertPassword": { 
                "type": "SecureString", 
                "value": "<password of your certificate>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource e.g. https://tenant.sharepoint.com>"
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}
```

**Ejemplo 6: Autenticación mediante clave de API**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "Anonymous",
            "authHeader": {
                "APIKey": {
                    "type": "SecureString",
                    "value": "<API key>"
                }
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de OData.

Para ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos y servicios vinculados](concepts-datasets-linked-services.md). 

Para copiar datos desde OData, establezca la propiedad **type** del conjunto de datos en **ODataResource**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del conjunto de datos debe establecerse en **ODataResource**. | Sí |
| path | Ruta de acceso al recurso de OData. | Sí |

**Ejemplo**

```json
{
    "name": "ODataDataset",
    "properties":
    {
        "type": "ODataResource",
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<OData linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties":
        {
            "path": "Products"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admite el origen de OData.

Para ver una lista completa de las secciones y propiedades que hay disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md). 

### <a name="odata-as-source"></a>OData como origen

Para copiar datos desde OData, en la sección **source** de la actividad de copia se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia se debe establecer en **ODataSource**. | Sí |
| Query | Opciones de consulta de OData para filtrar datos. Ejemplo: `"$select=Name,Description&$top=5"`.<br/><br/>**Nota**: el conector OData copia datos de la dirección URL combinada: `[URL specified in linked service]/[path specified in dataset]?[query specified in copy activity source]`. Para más información, consulte el artículo sobre [componentes de URL de OData](https://www.odata.org/documentation/odata-version-3-0/url-conventions/). | No |
| httpRequestTimeout | El tiempo de espera (el valor **TimeSpan**) para que la solicitud HTTP obtenga una respuesta. Este valor es el tiempo de espera para obtener una respuesta, no para leer los datos de la respuesta. Si no se especifica, el valor predeterminado es **00:30:00** (30 minutos). | No |

**Ejemplo**

```json
"activities":[
    {
        "name": "CopyFromOData",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<OData input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ODataSource",
                "query": "$select=Name,Description&$top=5"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Si estaba usando un origen de tipo `RelationalSource`, todavía se admite tal cual, aunque se aconseja usar el nuevo en el futuro.

## <a name="data-type-mapping-for-odata"></a>Asignación de tipos de datos de OData

Al copiar datos desde OData, se utilizan las siguientes asignaciones entre tipos de datos de OData y tipos de datos provisionales de Azure Data Factory. Para obtener información acerca de la forma en que la actividad de copia asigna el esquema de origen y el tipo de datos al receptor, consulte [Asignación de esquemas en la actividad de copia](copy-activity-schema-and-type-mapping.md).

| Tipo de datos de OData | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| Edm.Binary | Byte[] |
| Edm.Boolean | Bool |
| Edm.Byte | Byte[] |
| Edm.DateTime | DateTime |
| Edm.Decimal | Decimal |
| Edm.Double | Double |
| Edm.Single | Single |
| Edm.Guid | Guid |
| Edm.Int16 | Int16 |
| Edm.Int32 | Int32 |
| Edm.Int64 | Int64 |
| Edm.SByte | Int16 |
| Edm.String | String |
| Edm.Time | TimeSpan |
| Edm.DateTimeOffset | DateTimeOffset |

> [!NOTE]
> No se admiten tipos de datos complejos de OData (por ejemplo, **Object**).

## <a name="copy-data-from-project-online"></a>Copia de datos de Project Online

Para copiar datos de Project Online, puede usar el conector OData y un token de acceso obtenido de herramientas como Postman.

> [!CAUTION]
> El token de acceso expira en 1 hora de manera predeterminada; debe obtener uno nuevo en cuanto expire.

1. Use **Postman** para obtener el token de acceso:

   1. Vaya a la pestaña **Authorization** (Autorización) en el sitio web de Postman.
   1. En el cuadro **Type** (Tipo), seleccione **OAuth 2.0** y, en el cuadro **Add authorization data to** (Agregar datos de autorización a), seleccione **Request Headers** (Encabezados de solicitud).
   1. Rellene la siguiente información en la página **Configure New Token** (Configurar nuevo token) para obtener un nuevo token de acceso: 
      - **Grant type** (Tipo de concesión): seleccione **Authorization Code** (Código de autorización).
      - **Callback URL** (Dirección URL de devolución de llamada): escriba `https://www.localhost.com/`. 
      - **Auth URL** (Dirección URL de autenticación): escriba `https://login.microsoftonline.com/common/oauth2/authorize?resource=https://<your tenant name>.sharepoint.com`. Reemplace `<your tenant name>` por su propio nombre de inquilino. 
      - **Access Token URL** (Dirección URL de token de acceso): escriba `https://login.microsoftonline.com/common/oauth2/token`.
      - **Client ID** (Identificador de cliente): escriba el identificador de la entidad de servicio de AAD.
      - **Client Secret** (Secreto de cliente): escriba el secreto de la entidad de servicio.
      - **Client Authentication** (Autenticación de cliente): seleccione **Send as Basic Auth header** (Enviar como encabezado de autenticación básica).
     
   1. Se le va a pedir que inicie sesión con su nombre de usuario y contraseña.
   1. Una vez que obtenga el token de acceso, cópielo y guárdelo para el siguiente paso.
   
    [![Uso de Postman para obtener el token de acceso](./media/connector-odata/odata-project-online-postman-access-token-inline.png)](./media/connector-odata/odata-project-online-postman-access-token-expanded.png#lightbox)

1. Cree el servicio vinculado de OData:
    - **URL de servicio**: escriba `https://<your tenant name>.sharepoint.com/sites/pwa/_api/Projectdata`. Reemplace `<your tenant name>` por su propio nombre de inquilino. 
    - **Tipo de autenticación**: seleccione **Anónimo**.
    - **Encabezados de autenticación**:
        - **Nombre de propiedad**: seleccione **Autorización**.
        - **Valor**: escriba el **token de acceso** copiado en el paso 1.
    - Pruebe el servicio vinculado.

    ![Creación del servicio vinculado de OData](./media/connector-odata/odata-project-online-linked-service.png)

1. Cree el conjunto de datos de OData:
    1. Cree el conjunto de datos con el servicio vinculado de OData creado en el paso 2.
    1. Obtenga una vista previa de los datos.
 
    ![Vista previa de los datos](./media/connector-odata/odata-project-online-preview-data.png)
 


## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes

Para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes y receptores en Azure Data Factory, consulte los [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).