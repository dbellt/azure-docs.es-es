---
title: Copia de datos de Office 365 con Azure Data Factory
description: Obtenga información sobre cómo copiar datos desde Office 365 a almacenes de datos receptores compatibles mediante una actividad de copia de una canalización de Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/20/2019
ms.author: jianleishen
ms.openlocfilehash: 2140eeb3302799d55a0d45c9469edea9f11ded3b
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482154"
---
# <a name="copy-data-from-office-365-into-azure-using-azure-data-factory"></a>Copia de datos de Office 365 en Azure con Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Factory se integra con la [conexión de datos de Microsoft Graph](/graph/data-connect-concept-overview), lo que permite incluir los datos importantes de la organización en el inquilino de Office 365 en Azure de una manera escalable, además de crear aplicaciones de análisis y de extraer información en función de estos importantes recursos de datos. La integración con Privileged Access Management proporciona control de acceso seguro para los datos protegidos valiosos en Office 365.  Consulte [este vínculo](/graph/data-connect-concept-overview) para obtener información general sobre la conexión de datos de Microsoft Graph y remítase a [este vínculo](/graph/data-connect-policies#licensing) para información sobre licencias.

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Office 365. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas
La conexión de datos de Microsoft Graph y del conector ADF de Office 365 habilita la ingesta a escala de diferentes tipos de conjuntos de datos desde buzones habilitados para correo electrónico de Exchange, incluidos los contactos de la libreta de direcciones, los eventos de calendario, los mensajes de correo electrónico, la información de usuario, la configuración de los buzones, etc.  Consulte [aquí](/graph/data-connect-datasets) una lista completa de los conjuntos de datos disponibles.

Por ahora, en una única actividad de copia, solo puede **copiar datos de Office 365 en [Azure Blob Storage](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) en formato JSON** (tipo setOfObjects). Si desea cargar Office 365 en otros tipos de almacenes de datos o en otros formatos, puede encadenar la primera actividad de copia con una actividad de copia posterior para cargar más datos en cualquiera de los [almacenes de destino de ADF admitidos](copy-activity-overview.md#supported-data-stores-and-formats); consulte la columna "Se admite como receptor" de la tabla "Almacenes de datos y formatos que se admiten".

>[!IMPORTANT]
>- La suscripción de Azure que contiene la factoría de datos y el almacén de datos del receptor debe estar en el mismo inquilino de Azure Active Directory (Azure AD) que Office 365.
>- Asegúrese de que la región de Azure Integration Runtime utilizada para la actividad de copia y el destino están en la misma región donde se encuentra el buzón de los usuarios del inquilino de Office 365. Haga clic [aquí](concepts-integration-runtime.md#integration-runtime-location) para obtener información sobre cómo se determina la ubicación de Azure IR. Consulte la [tabla aquí](/graph/data-connect-datasets#regions) para obtener la lista de las regiones admitidas de Office y las regiones de Azure correspondientes.
>- La autenticación de la entidad de servicio es el único mecanismo de autenticación admitido para Azure Blob Storage, Azure Data Lake Storage Gen1 y Azure Data Lake Storage Gen2 como almacenes de destino.

## <a name="prerequisites"></a>Prerrequisitos

Para copiar datos de Office 365 en Azure, es preciso completar los pasos de requisitos previos siguientes:

- El administrador de inquilinos de Office 365 debe completar las acciones de incorporación como se describe [aquí](/graph/data-connect-get-started).
- Cree y configure una aplicación web de Azure AD en Azure Active Directory.  Para obtener instrucciones, vea cómo [crear una aplicación de Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal).
- Anote los siguientes valores; los usará para definir el servicio vinculado para Office 365:
    - Identificador de inquilino. Para obtener instrucciones, vea [Obtención del identificador de inquilino](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
    - Identificador de aplicación y clave de aplicación.  Para obtener instrucciones, vea [Obtención del id. y la clave de autenticación de la aplicación](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
- Agregue la identidad del usuario que va a realizar la solicitud de acceso a los datos como propietario de la aplicación web de Azure AD (aplicación web de Azure AD > Configuración > Propietarios > Agregar propietario). 
    - La identidad del usuario debe pertenecer a la organización de Office 365 de la que obtendrá los datos. Además, no debe un usuario invitado.

## <a name="approving-new-data-access-requests"></a>Aprobar nuevas solicitudes de acceso a los datos

Si es la primera vez que solicita datos para este contexto (una combinación de los datos de tabla a los que se accede, la cuenta de destino en la que se cargan los datos y la identidad de usuario que realiza la solicitud de acceso a los datos), verá el estado de la actividad de copia como "En curso", y solo al hacer clic en el [vínculo "Detalles" en Acciones](copy-activity-overview.md#monitoring), verá el estado como "RequestingConsent".  Un miembro del grupo de aprobadores de acceso a datos debe aprobar la solicitud en Privileged Access Management antes de pasar a la extracción de datos.

Consulte [aquí](/graph/data-connect-tips#approve-pam-requests-via-office-365-admin-portal) cómo el aprobador puede aprobar la solicitud de acceso a los datos y haga clic [aquí](/graph/data-connect-pam) para obtener una explicación sobre la integración total con Privileged Access Management, incluida la forma de configurar el grupo de aprobadores de acceso a los datos.

## <a name="policy-validation"></a>Validación de directivas

Si ADF se crea como parte de una aplicación administrada y las asignaciones de directivas de Azure se realizan en los recursos dentro del grupo de recursos de administración, entonces, para cada ejecución de la actividad de copia, ADF comprobará para asegurarse de que se aplican las asignaciones de directivas. Consulte [aquí](/graph/data-connect-policies#policies) para obtener una lista de las directivas admitidas.

## <a name="getting-started"></a>Introducción

>[!TIP]
>Para ver un tutorial sobre el uso del conector de Office 365, consulte el artículo [Carga de datos de Office 365](load-office-365-data.md).

Puede crear una canalización con la actividad de copia mediante una de las siguientes herramientas o SDK. Seleccione un vínculo para acceder a un tutorial con instrucciones detalladas sobre cómo crear una canalización con una actividad de copia. 

- [Azure Portal](quickstart-create-data-factory-portal.md)
- [SDK de .NET](quickstart-create-data-factory-dot-net.md)
- [SDK de Python](quickstart-create-data-factory-python.md)
- [Azure PowerShell](quickstart-create-data-factory-powershell.md)
- [REST API](quickstart-create-data-factory-rest-api.md)
- [Plantilla de Azure Resource Manager](quickstart-create-data-factory-resource-manager-template.md) 

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Office 365.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Office 365:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Office365** | Sí |
| office365TenantId | Id. de inquilino de Azure al que pertenece la cuenta de Office 365. | Sí |
| servicePrincipalTenantId | Especifique la información del inquilino en el que reside la aplicación web de Azure AD. | Sí |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory. | Sí |
| connectVia | El entorno de ejecución de integración que se usará para conectarse al almacén de datos.  Si no se especifica, se usará Azure Integration Runtime. | No |

>[!NOTE]
> La diferencia entre **office365TenantId** y **servicePrincipalTenantId** y el valor correspondiente que se debe proporcionar:
>- Si es un desarrollador empresarial que va a desarrollar una aplicación con los datos de Office 365 para usarla en su organización, debe proporcionar el mismo identificador de inquilino para ambas propiedades, que es el identificador de inquilino de AAD de su organización.
>- Si es un ISV que va a desarrollar una aplicación para los clientes, office365TenantId será el identificador de inquilino de AAD (instalador de la aplicación) del cliente y servicePrincipalTenantId será el identificador de inquilino de AAD de la empresa.

**Ejemplo**:

```json
{
    "name": "Office365LinkedService",
    "properties": {
        "type": "Office365",
        "typeProperties": {
            "office365TenantId": "<Office 365 tenant id>",
            "servicePrincipalTenantId": "<AAD app service principal tenant id>",
            "servicePrincipalId": "<AAD app service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<AAD app service principal key>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Office 365.

Para copiar datos de Office 365, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **Office365Table** | Sí |
| tableName | Nombre del conjunto de datos para extraer de Office 365. Haga clic [aquí](/graph/data-connect-datasets#datasets) para obtener la lista de conjuntos de datos de Office 365 disponibles para la extracción. | Sí |

Si estaba configurando `dateFilterColumn`, `startTime`, `endTime` y `userScopeFilterUri` en el conjunto de datos, todavía se admite tal cual, aunque se aconseja usar en el futuro el nuevo modelo del origen de la actividad.

**Ejemplo**

```json
{
    "name": "DS_May2019_O365_Message",
    "properties": {
        "type": "Office365Table",
        "linkedServiceName": {
            "referenceName": "<Office 365 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [],
        "typeProperties": {
            "tableName": "BasicDataSet_v0.Event_v1"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de Office 365.

### <a name="office-365-as-source"></a>Office 365 como origen

Para copiar datos de Office 365, se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **Office365Source** | Sí |
| allowedGroups | Predicado de la selección de grupo.  Utilice esta propiedad para seleccionar hasta 10 grupos de usuarios para los que se recuperarán los datos.  Si no se especifica ningún grupo, se devolverán datos para toda la organización. | No |
| userScopeFilterUri | Cuando la propiedad `allowedGroups` no se especifica, puede usar una expresión de predicado que se aplica en todo el inquilino para filtrar las filas específicas que extraer de Office 365. El formato de predicado debe coincidir con el formato de consultas de las API de Microsoft Graph, por ejemplo, `https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'`. | No |
| dateFilterColumn | Nombre de la columna de filtro DateTime. Utilice esta propiedad para limitar el intervalo de tiempo para que se extraigan los datos de Office 365. | Sí, si el conjunto de datos tiene una o varias columnas DateTime. Consulte [aquí](/graph/data-connect-filtering#filtering) para obtener la lista de conjuntos de datos que requieren este filtro DateTime. |
| startTime | Valor DateTime de inicio para el filtro. | Sí, si se especifica `dateFilterColumn` |
| endTime | Valor DateTime de fin para el filtro. | Sí, si se especifica `dateFilterColumn` |
| outputColumns | Matriz de las columnas que se van a copiar en el receptor. | No |

**Ejemplo**:

```json
"activities": [
    {
        "name": "CopyFromO365ToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Office 365 input dataset name>",
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
                "type": "Office365Source",
                "dateFilterColumn": "CreatedDateTime",
                "startTime": "2019-04-28T16:00:00.000Z",
                "endTime": "2019-05-05T16:00:00.000Z",
                "userScopeFilterUri": "https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'",
                "outputColumns": [
                    {
                        "name": "Id"
                    },
                    {
                        "name": "CreatedDateTime"
                    },
                    {
                        "name": "LastModifiedDateTime"
                    },
                    {
                        "name": "ChangeKey"
                    },
                    {
                        "name": "Categories"
                    },
                    {
                        "name": "OriginalStartTimeZone"
                    },
                    {
                        "name": "OriginalEndTimeZone"
                    },
                    {
                        "name": "ResponseStatus"
                    },
                    {
                        "name": "iCalUId"
                    },
                    {
                        "name": "ReminderMinutesBeforeStart"
                    },
                    {
                        "name": "IsReminderOn"
                    },
                    {
                        "name": "HasAttachments"
                    },
                    {
                        "name": "Subject"
                    },
                    {
                        "name": "Body"
                    },
                    {
                        "name": "Importance"
                    },
                    {
                        "name": "Sensitivity"
                    },
                    {
                        "name": "Start"
                    },
                    {
                        "name": "End"
                    },
                    {
                        "name": "Location"
                    },
                    {
                        "name": "IsAllDay"
                    },
                    {
                        "name": "IsCancelled"
                    },
                    {
                        "name": "IsOrganizer"
                    },
                    {
                        "name": "Recurrence"
                    },
                    {
                        "name": "ResponseRequested"
                    },
                    {
                        "name": "ShowAs"
                    },
                    {
                        "name": "Type"
                    },
                    {
                        "name": "Attendees"
                    },
                    {
                        "name": "Organizer"
                    },
                    {
                        "name": "WebLink"
                    },
                    {
                        "name": "Attachments"
                    },
                    {
                        "name": "BodyPreview"
                    },
                    {
                        "name": "Locations"
                    },
                    {
                        "name": "OnlineMeetingUrl"
                    },
                    {
                        "name": "OriginalStart"
                    },
                    {
                        "name": "SeriesMasterId"
                    }
                ]
            },
            "sink": {
                "type": "BlobSink"
            }
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.