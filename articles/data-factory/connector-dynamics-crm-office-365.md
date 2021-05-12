---
title: Copia de datos en Dynamics (Common Data Service)
description: Aprenda a copiar datos de Microsoft Dynamics CRM o Microsoft Dynamics 365 (Common Data Service/Microsoft Dataverse) en almacenes de datos receptores compatibles, o bien de almacenes de datos de origen compatibles en Dynamics CRM o Dynamics 365 mediante una actividad de copia en una canalización de factoría de datos.
ms.service: data-factory
ms.topic: conceptual
ms.author: jianleishen
author: jianleishen
ms.custom: seo-lt-2019
ms.date: 03/17/2021
ms.openlocfilehash: c949ed8d0ecb35df0a2c31bb90514c18cf3a3755
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109484332"
---
# <a name="copy-data-from-and-to-dynamics-365-common-data-servicemicrosoft-dataverse-or-dynamics-crm-by-using-azure-data-factory"></a>Copia de datos desde y hacia Dynamics 365 (Common Data Service/Microsoft Dataverse) o Dynamics CRM mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde y hacia Microsoft Dynamics 365 y Microsoft Dynamics CRM. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que proporciona información general de esta actividad.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen y receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de Dynamics 365 (Common Data Service/Microsoft Dataverse) o Dynamics CRM en cualquier almacén de datos receptor compatible. También puede copiar datos de cualquier almacén de datos de origen compatible en Dynamics 365 (Common Data Service) o Dynamics CRM. Consulte la tabla de [almacenes de datos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes y receptores.

Este conector de Dynamics es compatible con las versiones 7 a 9.x de Dynamics, tanto en línea como en el entorno local. Más concretamente:

- La versión 7 se asigna a Dynamics CRM 2015.
- La versión 8 se asigna a Dynamics CRM 2016 y a la versión anterior de Dynamics 365.
- La versión 9 se asigna a la versión más reciente de Dynamics 365.

Consulte en la tabla siguiente los tipos y configuraciones de autenticación compatibles para las versiones o productos de Dynamics.

| Versiones de Dynamics | Tipos de autenticación | Ejemplos de servicios vinculados |
|:--- |:--- |:--- |
| Common Data Service <br/><br/> Dynamics 365 Online <br/><br/> Dynamics CRM Online | Entidad de servicio de Azure Active Directory (Azure AD) <br/><br/> Office 365 | [Autenticación de Office 365 o entidad de servicio de Azure AD y Dynamics en línea](#dynamics-365-and-dynamics-crm-online) |
| Dynamics 365 en el entorno local con implementación con conexión a Internet (IFD) <br/><br/> Dynamics CRM 2016 local con IFD <br/><br/> Dynamics CRM 2015 local con IFD | IFD | [Dynamics en el entorno local con IFD y la autenticación de IFD](#dynamics-365-and-dynamics-crm-on-premises-with-ifd) |

Para Dynamics 365 en concreto, se admiten los siguientes tipos de aplicación:

- Dynamics 365 for Sales
- Dynamics 365 for Customer Service
- Dynamics 365 for Field Service
- Dynamics 365 for Project Service Automation
- Dynamics 365 for Marketing

Este conector no admite otros tipos de aplicaciones como Finance, Operations o Talent.

>[!TIP]
>Para copiar datos desde Dynamics 365 Finance and Operations, puede usar el [conector de Dynamics AX](connector-dynamics-ax.md).

Este conector de Dynamics se basa en las [herramientas de Dynamics XRM](/dynamics365/customer-engagement/developer/build-windows-client-applications-xrm-tools).

## <a name="prerequisites"></a>Requisitos previos

Para usar este conector con la autenticación de la entidad de servicio de Azure AD, debe configurar la autenticación de servidor a servidor (S2S) en Common Data Service o Dynamics. Consulte [este artículo](/powerapps/developer/common-data-service/build-web-applications-server-server-s2s-authentication) para conocer los pasos detallados.

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de Dynamics.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Dynamics.

### <a name="dynamics-365-and-dynamics-crm-online"></a>Dynamics 365 y Dynamics CRM en línea

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en "Dynamics", "DynamicsCrm" o "CommonDataServiceForApps". | Sí |
| deploymentType | El tipo de implementación de la instancia de Dynamics. El valor debe ser "Online" para Dynamics en línea. | Sí |
| serviceUri | Dirección URL de servicio de la instancia de Dynamics, la misma a la que se accede desde el explorador. "https://\<organization-name>.crm[x].dynamics.com" es un ejemplo. | Sí |
| authenticationType | Tipo de autenticación para conectarse a un servidor de Dynamics. Los valores válidos son "AADServicePrincipal" y "Office365". | Sí |
| servicePrincipalId | Identificador de cliente de la aplicación de Azure AD. | Sí, cuando la autenticación es "AADServicePrincipal" |
| servicePrincipalCredentialType | Tipo de credencial que se usará para la autenticación de entidad de servicio. Los valores válidos son "ServicePrincipalKey" y "ServicePrincipalCert". | Sí, cuando la autenticación es "AADServicePrincipal" |
| servicePrincipalCredential | Credencial de entidad de seguridad de servicio. <br/><br/>Cuando se usa "ServicePrincipalKey" como tipo de credencial, `servicePrincipalCredential` puede ser una cadena que Azure Data Factory cifra en la implementación del servicio vinculado. O bien, puede ser una referencia a un secreto en Azure Key Vault. <br/><br/>Cuando se usa "ServicePrincipalCert" como credencial, `servicePrincipalCredential` debe ser una referencia a un certificado en Azure Key Vault. | Sí, cuando la autenticación es "AADServicePrincipal" |
| username | Nombre de usuario para conectarse a Dynamics. | Sí, cuando la autenticación es "Office365" |
| password | Contraseña de la cuenta de usuario que especificó como nombre de usuario. Marque este campo con "SecureString" para almacenarlo de forma segura en Data Factory, o bien [haga referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | Sí, cuando la autenticación es "Office365" |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si no se especifica ningún valor, la propiedad usa el entorno de ejecución de integración de Azure predeterminado. | No |

>[!NOTE]
>El conector de Dynamics usaba antes la propiedad opcional **organizationName** para identificar la instancia de Dynamics CRM o Dynamics 365 en línea. Mientras la propiedad siga funcionando, es preferible especificar la nueva propiedad **serviceUri** en su lugar para lograr un mejor rendimiento de detección de la instancia.

#### <a name="example-dynamics-online-using-azure-ad-service-principal-and-key-authentication"></a>Ejemplo: Dynamics en línea con la entidad de servicio de Azure AD y autenticación de claves

```json
{  
    "name": "DynamicsLinkedService",  
    "properties": {  
        "type": "Dynamics",  
        "typeProperties": {  
            "deploymentType": "Online",  
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com",  
            "authenticationType": "AADServicePrincipal",  
            "servicePrincipalId": "<service principal id>",  
            "servicePrincipalCredentialType": "ServicePrincipalKey",  
            "servicePrincipalCredential": "<service principal key>"
        },  
        "connectVia": {  
            "referenceName": "<name of Integration Runtime>",  
            "type": "IntegrationRuntimeReference"  
        }  
    }  
}  
```
#### <a name="example-dynamics-online-using-azure-ad-service-principal-and-certificate-authentication"></a>Ejemplo: Dynamics en línea con la entidad de servicio de Azure AD y la autenticación de certificados

```json
{ 
    "name": "DynamicsLinkedService", 
    "properties": { 
        "type": "Dynamics", 
        "typeProperties": { 
            "deploymentType": "Online", 
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com", 
            "authenticationType": "AADServicePrincipal", 
            "servicePrincipalId": "<service principal id>", 
            "servicePrincipalCredentialType": "ServicePrincipalCert", 
            "servicePrincipalCredential": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<AKV reference>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<certificate name in AKV>" 
            } 
        }, 
        "connectVia": { 
            "referenceName": "<name of Integration Runtime>", 
            "type": "IntegrationRuntimeReference" 
        } 
    } 
} 
```

#### <a name="example-dynamics-online-using-office-365-authentication"></a>Ejemplo: Dynamics en línea mediante la autenticación de Office 365

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "typeProperties": {
            "deploymentType": "Online",
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com",
            "authenticationType": "Office365",
            "username": "test@contoso.onmicrosoft.com",
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

### <a name="dynamics-365-and-dynamics-crm-on-premises-with-ifd"></a>Dynamics 365 y Dynamics CRM local con IFD

Las propiedades adicionales, en comparación con Dynamics en línea, son **hostName** y **port**.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en "Dynamics", "DynamicsCrm" o "CommonDataServiceForApps". | Sí. |
| deploymentType | El tipo de implementación de la instancia de Dynamics. Debe ser "OnPremisesWithIfd" para Dynamics en el entorno local con IFD.| Sí. |
| hostName | El nombre de host del servidor local de Dynamics. | Sí. |
| port | El puerto del servidor local de Dynamics. | No. El valor predeterminado es 443. |
| organizationName | El nombre de la organización de la instancia de Dynamics. | Sí. |
| authenticationType | Tipo de autenticación para conectarse al servidor de Dynamics. Especifique "Ifd" para Dynamics en el entorno local con IFD. | Sí. |
| username | Nombre de usuario para conectarse a Dynamics. | Sí. |
| password | Contraseña de la cuenta de usuario especificada para el nombre de usuario. Este campo se puede marcar como "SecureString" para almacenarlo de forma segura en Data Factory. También puede almacenar una contraseña en Key Vault y permitir que la actividad de copia se extraiga desde allí cuando realice la copia de datos. Más información sobre [Almacenamiento de credenciales en Key Vault](store-credentials-in-key-vault.md). | Sí. |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si no se especifica ningún valor, la propiedad usa el entorno de ejecución de integración de Azure predeterminado. | No |

#### <a name="example-dynamics-on-premises-with-ifd-using-ifd-authentication"></a>Ejemplo: Dynamics en el entorno local con IFD mediante la autenticación de IFD

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics on-premises with IFD linked service using IFD authentication",
        "typeProperties": {
            "deploymentType": "OnPremisesWithIFD",
            "hostName": "contosodynamicsserver.contoso.com",
            "port": 443,
            "organizationName": "admsDynamicsTest",
            "authenticationType": "Ifd",
            "username": "test@contoso.onmicrosoft.com",
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

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Dynamics.

Para copiar datos con Dynamics como origen o destino, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en "DynamicsEntity", "DynamicsCrmEntity" o "CommonDataServiceForAppsEntity". |Sí |
| entityName | El nombre lógico de la entidad que se va a recuperar. | No para el origen si se especifica "query" en el origen de la actividad y sí para el receptor |

#### <a name="example"></a>Ejemplo

```json
{
    "name": "DynamicsDataset",
    "properties": {
        "type": "DynamicsEntity",
        "schema": [],
        "typeProperties": {
            "entityName": "account"
        },
        "linkedServiceName": {
            "referenceName": "<Dynamics linked service name>",
            "type": "linkedservicereference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el tipo de origen y el tipo de receptor de Dynamics.

### <a name="dynamics-as-a-source-type"></a>Dynamics como tipo de origen

Para copiar datos desde Dynamics, en la sección **source** de la actividad de copia se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en "DynamicsSource", "DynamicsCrmSource" o "CommonDataServiceForAppsSource". | Sí |
| Query | FetchXML es un lenguaje de consulta patentado que se usa en Dynamics en línea y en el entorno local. Consulte el ejemplo siguiente. Para más información, consulte [Build queries with FetchXML](/previous-versions/dynamicscrm-2016/developers-guide/gg328332(v=crm.8)) (Creación de consultas con FetchXML). | No, si se especifica `entityName` en el conjunto de datos |

>[!NOTE]
>La columna PK siempre se copiará incluso si la proyección de columna que se configura en la consulta de FetchXML no la contiene.

> [!IMPORTANT]
>- Al copiar datos desde Dynamics, la asignación de columnas explícita de Dynamics al receptor es opcional. Sin embargo, se recomienda encarecidamente la asignación para garantizar un resultado de copia determinista.
>- Cuando Data Factory importa un esquema en la interfaz de usuario de creación, lo infiere. Para ello, muestrea las primeras filas del resultado de la consulta de Dynamics a fin de inicializar la lista de columnas de origen. En ese caso, se omiten las columnas sin valores en las filas superiores. El mismo comportamiento se aplica a las ejecuciones de copia si no existe una asignación explícita. Puede revisar y agregar más columnas a la asignación, que se respetarán durante el tiempo de ejecución de la copia.

#### <a name="example"></a>Ejemplo

```json
"activities":[
    {
        "name": "CopyFromDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DynamicsSource",
                "query": "<FetchXML Query>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="sample-fetchxml-query"></a>Consulta FetchXML de ejemplo

```xml
<fetch>
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="marketingonly" />
    <attribute name="modifiedon" />
    <order attribute="modifiedon" descending="false" />
    <filter type="and">
      <condition attribute ="modifiedon" operator="between">
        <value>2017-03-10 18:40:00z</value>
        <value>2017-03-12 20:40:00z</value>
      </condition>
    </filter>
  </entity>
</fetch>
```

### <a name="dynamics-as-a-sink-type"></a>Dynamics como tipo de receptor

Para copiar datos a Dynamics, la sección **sink** de la actividad de copia admite las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del receptor de la actividad de copia debe establecerse en "DynamicsSink", "DynamicsCrmSink" o "CommonDataServiceForAppsSink". | Sí. |
| writeBehavior | El comportamiento de escritura de la operación. El valor debe ser "Upsert". | Sí |
| alternateKeyName | Nombre de clave alternativo definido en la entidad para realizar una operación de actualización e inserción. | No. |
| writeBatchSize | El recuento de filas de datos escritos en Dynamics en cada lote. | No. El valor predeterminado es 10. |
| ignoreNullValues | Si durante la operación de escritura se omiten los valores nulos de los datos de entrada excepto los campos de clave.<br/><br/>Los valores válidos son **TRUE** y **FALSE**:<ul><li>**TRUE**: deje los datos del objeto de destino sin cambiar cuando realice una operación upsert o update. Inserta un valor predeterminado definido al realizar una operación insert.</li><li>**FALSE**: actualice los datos del objeto de destino a un valor NULL cuando realice una operación de actualización e inserción, o bien de actualización. Inserte un valor NULL al realizar una operación de inserción.</li></ul> | No. El valor predeterminado es **FALSE**. |
| maxConcurrentConnections |Número máximo de conexiones simultáneas establecidas en el almacén de datos durante la ejecución de la actividad. Especifique un valor solo cuando quiera limitar las conexiones simultáneas.| No |

>[!NOTE]
>El valor predeterminado tanto del receptor **writeBatchSize** como de la actividad de copia **[parallelCopies](copy-activity-performance-features.md#parallel-copy)** del receptor de Dynamics es 10. Por lo tanto, se envían 100 registros simultáneamente de forma predeterminada a Dynamics.

Para Dynamics 365 en línea, hay un límite de [dos llamadas simultáneas por lotes por organización](/previous-versions/dynamicscrm-2016/developers-guide/jj863631(v=crm.8)#Run-time%20limitations). Si se supera ese límite, se produce un error de "Servidor ocupado" antes incluso de que se ejecute la primera solicitud. Mantenga **writeBatchSize** en 10 o menos para evitar la limitación de llamadas simultáneas.

La combinación óptima de **writeBatchSize** y **parallelCopies** depende del esquema de la entidad. Los elementos del esquema incluyen el número de columnas, tamaño de las filas y número de complementos, flujos de trabajo o actividades de flujo de trabajo enlazadas a esas llamadas. La configuración predeterminada de **writeBatchSize** (10) &times; **parallelCopies** (10) es la recomendación de acuerdo con el servicio Dynamics. Este valor funciona con la mayoría de las entidades de Dynamics, aunque es posible que no ofrezca el mejor rendimiento. Para optimizar el rendimiento, ajuste la combinación en su configuración de la actividad de copia.

#### <a name="example"></a>Ejemplo

```json
"activities":[
    {
        "name": "CopyToDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Dynamics output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DynamicsSink",
                "writeBehavior": "Upsert",
                "writeBatchSize": 10,
                "ignoreNullValues": true
            }
        }
    }
]
```

## <a name="retrieving-data-from-views"></a>Recuperación de datos de las vistas

Para recuperar datos de las vistas de Dynamics, debe obtener la consulta guardada de la vista y usar la consulta para obtener los datos.

Hay dos entidades que almacenan distintos tipos de vista: "consulta guardada" almacena la vista del sistema "consulta de usuario" almacena la vista del usuario. Para obtener información de las vistas, consulte la siguiente consulta FetchXML y reemplace "TARGETENTITY" por `savedquery` o `userquery`. Cada tipo de entidad tiene más atributos disponibles que puede agregar a la consulta en función de sus necesidades. Obtenga más información sobre la [entidad savedquery](/dynamics365/customer-engagement/web-api/savedquery) y la [entidad userquery](/dynamics365/customer-engagement/web-api/userquery).

```xml
<fetch top="5000" >
  <entity name="<TARGETENTITY>">
    <attribute name="name" />
    <attribute name="fetchxml" />
    <attribute name="returnedtypecode" />
    <attribute name="querytype" />
  </entity>
</fetch>
```

También puede agregar filtros para filtrar las vistas. Por ejemplo, agregue el siguiente filtro para obtener una vista denominada "Mis cuentas activas" en entidad de cuenta.

```xml
<filter type="and" >
    <condition attribute="returnedtypecode" operator="eq" value="1" />
    <condition attribute="name" operator="eq" value="My Active Accounts" />
</filter>
```

## <a name="data-type-mapping-for-dynamics"></a>Asignación de tipos datos de Dynamics

Al copiar los datos desde Dynamics, la tabla siguiente muestra las asignaciones de tipos de datos de Dynamics a los tipos de datos provisionales de Data Factory. Para obtener información acerca de la forma en que la actividad de copia asigna el esquema de origen y el tipo de datos a un receptor, consulte [Asignación de esquemas en la actividad de copia](copy-activity-schema-and-type-mapping.md).

Configure el tipo de datos de Data Factory correspondiente en la estructura del conjunto de datos en función del tipo de datos de Dynamics de origen mediante la siguiente tabla de asignación:

| Tipo de datos de Dynamics | Tipo de datos provisionales de Data Factory | Se admite como origen | Se admite como receptor |
|:--- |:--- |:--- |:--- |
| AttributeTypeCode.BigInt | long | ✓ | ✓ |
| AttributeTypeCode.Boolean | Boolean | ✓ | ✓ |
| AttributeType.Customer | GUID | ✓ | ✓ (Consulte [guía](#writing-data-to-a-lookup-field)) |
| AttributeType.DateTime | Datetime | ✓ | ✓ |
| AttributeType.Decimal | Decimal | ✓ | ✓ |
| AttributeType.Double | Double | ✓ | ✓ |
| AttributeType.EntityName | String | ✓ | ✓ |
| AttributeType.Integer | Int32 | ✓ | ✓ |
| AttributeType.Lookup | GUID | ✓ | ✓ (Consulte [guía](#writing-data-to-a-lookup-field)) |
| AttributeType.ManagedProperty | Boolean | ✓ | |
| AttributeType.Memo | String | ✓ | ✓ |
| AttributeType.Money | Decimal | ✓ | ✓ |
| AttributeType.Owner | GUID | ✓ | ✓ (Consulte [guía](#writing-data-to-a-lookup-field)) |
| AttributeType.Picklist | Int32 | ✓ | ✓ |
| AttributeType.Uniqueidentifier | GUID | ✓ | ✓ |
| AttributeType.String | String | ✓ | ✓ |
| AttributeType.State | Int32 | ✓ | ✓ |
| AttributeType.Status | Int32 | ✓ | ✓ |

> [!NOTE]
> Los tipos de datos **AttributeType.CalendarRules**, **AttributeType.MultiSelectPicklist** y **AttributeType.PartyList** no se admiten.

## <a name="writing-data-to-a-lookup-field"></a>Escritura de datos en un campo de búsqueda

Para escribir datos en un campo de búsqueda con varios destinos como Customer y Owner, siga estas instrucciones y el ejemplo:

1. Haga que el origen contenga tanto el valor del campo como el nombre de la entidad de destino correspondiente.
   - Si todos los registros se asignan a la misma entidad de destino, asegúrese una de las siguientes condiciones:
      - Los datos de origen tienen una columna que almacena el nombre de la entidad de destino.
      - Ha agregado una columna adicional en el origen de la actividad de copia para definir la entidad de destino.
   - Si hay registros diferentes asignados a diferentes entidades de destino, asegúrese de que los datos de origen tengan una columna que almacene el nombre de la entidad de destino correspondiente.

1. Asigne tanto las columnas de valor y de referencia de entidad del origen al receptor. La columna de referencia de entidad debe asignarse a una columna virtual con el patrón de nomenclatura especial `{lookup_field_name}@EntityReference`. La columna no existe realmente en Dynamics. Se usa para indicar que es la columna de metadatos del campo de búsqueda de múltiples destinos especificado.

Por ejemplo, suponga que el origen tiene estas dos columnas:

- **CustomerField** columna de tipo **GUID**, que es el valor de clave principal de la entidad de destino en Dynamics.
- **Target** columna de tipo **String**, que es el nombre lógico de la entidad de destino.

Supongamos también que desea copiar estos datos al campo de entidad de Dynamics receptor **CustomerField** de tipo **Customer**.

En la asignación de columnas de copia y actividad, asigne las dos columnas como se indica a continuación:

- **CustomerField** a **CustomerField**. Se trata de la asignación de campos normal.
- **Target** a **CustomerField\@EntityReference**. La columna de receptor es una columna virtual que representa la referencia de entidad. Escriba estos nombres de campo en una asignación, ya que no se mostrarán mediante la importación de esquemas.

![Asignación de columnas de campo de búsqueda de Dynamics](./media/connector-dynamics-crm-office-365/connector-dynamics-lookup-field-column-mapping.png)

Si todos los registros de origen se asignan a la misma entidad de destino y los datos de origen no contienen el nombre de la entidad de destino, este es un acceso directo: en el origen de la actividad de copia, agregue una columna adicional. Asigne un nombre a la nueva columna mediante el patrón `{lookup_field_name}@EntityReference`, establezca el valor en el nombre de la entidad de destino y, después, continúe con la asignación de columnas como de costumbre. Si los nombres de las columnas del origen y el receptor son idénticos, también puede omitir la asignación de columnas explícita, ya que la actividad de copia asigna las columnas por nombre de forma predeterminada.

![Campo de búsqueda de Dynamics que agrega una columna de referencia de entidad](./media/connector-dynamics-crm-office-365/connector-dynamics-add-entity-reference-column.png)

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Para consultar una lista de los almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores, consulte [Almacenes de datos admitidos](copy-activity-overview.md#supported-data-stores-and-formats).