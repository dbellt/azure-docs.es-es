---
title: Registros de auditoría de Azure Attestation
description: Describe los registros de auditoría completos que se recopilan para Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 11/23/2020
ms.author: mbaldwin
ms.openlocfilehash: 97aa0fd842318bae088f25d49200feb971e49b5c
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111748380"
---
# <a name="audit-logs-for-azure-attestation"></a>Registros de auditoría para Azure Attestation

Los registros de auditoría son documentos inmutables, seguros y con marca de tiempo de eventos discretos que se produjeron a lo largo del tiempo. Estos registros capturan eventos importantes que pueden afectar a la funcionalidad de la instancia de atestación.

Azure Attestation administra las instancias de atestación y las directivas asociadas a ellas. Las acciones asociadas a la administración de instancias y los cambios de directivas se auditan y registran.

Este artículo contiene información sobre los eventos registrados, la información recopilada y la ubicación de estos registros.

## <a name="about-audit-logs"></a>Acerca de los registros de auditoría

Azure Attestation usa código para generar registros de auditoría para los eventos que afectan a la manera en que se realiza la atestación. Normalmente, esto se reduce a cómo y cuándo se realizan los cambios de directiva en la instancia de atestación, así como algunas acciones de administración.

### <a name="auditable-events"></a>Eventos que se pueden auditar
Estos son algunos de los registros de auditoría que recopilamos:

|     Evento/API                              |     Descripción del evento                                                                         |
|--------------------------------------------|-----------------------------------------------------------------------------------------------|
|     Create Instance                        |     Crea una nueva instancia del servicio de atestación. |
|     Destroy Instance                       |     Destruye una instancia del servicio de atestación. |
|     Add Policy Certificate                 |     Adición de un certificado al conjunto actual de certificados de administración de directivas. |
|     Remove Policy Certificate              |     Eliminación de un certificado del conjunto actual de certificados de administración de directivas. |
|     Set Current Policy                     |     Establece la directiva de atestación para un tipo de TEE determinado. |
|     Reset Attestation Policy               |     Restablece la directiva de atestación para un tipo de TEE determinado. |
|     Prepare to Update Policy               |     Realice los preparativos para actualizar la directiva de atestación para un tipo de TEE determinado. |
|     Rehydrate Tenants After Disaster       |     Vuelve a sellar todos los inquilinos de atestación en esta instancia del servicio de atestación. Esto solo lo pueden realizar los administradores del servicio de atestación. |

### <a name="collected--information"></a>Información recopilada
Para cada uno de estos eventos, Azure Attestation recopila la siguiente información:

- Nombre de la operación
- Operación correcta
- Autor de llamada de la operación, que podría ser cualquiera de los siguientes:
    - UPN de Azure AD
    - Id. de objeto
    - Certificado
    - Identificador de inquilino de Azure AD
- Destino de la operación, que podría ser cualquiera de los siguientes:
    - Entorno
    - Región del servicio
    - Rol del servicio
    - Instancia del rol del servicio
    - Id. de recurso
    - Región del recurso

### <a name="sample-audit-log"></a>Registro de auditoría de ejemplo

Los registros de auditoría se proporcionan en formato JSON. Este es un ejemplo de cómo podría ser un registro de auditoría.

```json
{
    "operationName": "SetCurrentPolicy",
    "resultType": "Success",
    "resultDescription": null,
    "auditEventCategory": [
        "ApplicationManagement"
    ],
    "nCloud": null,
    "requestId": null,
    "callerIpAddress": null,
    "callerDisplayName": null,
    "callerIdentities": [
        {
            "callerIdentityType": "ObjectID",
            "callerIdentity": "<some object ID>"
        },
        {
            "callerIdentityType": "TenantId",
            "callerIdentity": "<some tenant ID>"
        }
    ],
    "targetResources": [
        {
            "targetResourceType": "Environment",
            "targetResourceName": "PublicCloud"
        },
        {
            "targetResourceType": "ServiceRegion",
            "targetResourceName": "EastUS2"
        },
        {
            "targetResourceType": "ServiceRole",
            "targetResourceName": "AttestationRpType"
        },
        {
            "targetResourceType": "ServiceRoleInstance",
            "targetResourceName": "<some service role instance>"
        },
        {
            "targetResourceType": "ResourceId",
            "targetResourceName": "/subscriptions/<some subscription ID>/resourceGroups/<some resource group name>/providers/Microsoft.Attestation/attestationProviders/<some instance name>"
        },
        {
            "targetResourceType": "ResourceRegion",
            "targetResourceName": "EastUS2"
        }
    ],
    "ifxAuditFormat": "Json",
    "env_ver": "2.1",
    "env_name": "#Ifx.AuditSchema",
    "env_time": "2020-11-23T18:23:29.9427158Z",
    "env_epoch": "MKZ6G",
    "env_seqNum": 1277,
    "env_popSample": 0.0,
    "env_iKey": null,
    "env_flags": 257,
    "env_cv": "##00000000-0000-0000-0000-000000000000_00000000-0000-0000-0000-000000000000_00000000-0000-0000-0000-000000000000",
    "env_os": null,
    "env_osVer": null,
    "env_appId": null,
    "env_appVer": null,
    "env_cloud_ver": "1.0",
    "env_cloud_name": null,
    "env_cloud_role": null,
    "env_cloud_roleVer": null,
    "env_cloud_roleInstance": null,
    "env_cloud_environment": null,
    "env_cloud_location": null,
    "env_cloud_deploymentUnit": null
}
```

## <a name="access-audit-logs"></a>Acceso a registros de auditoría

Estos registros se almacenan en Azure y no se puede acceder a ellos directamente. Si necesita acceder a estos registros, envíe una incidencia de soporte técnico. Para obtener más información, consulte [Ponerse en contacto con el soporte técnico de Microsoft](https://azure.microsoft.com/support/options/). 

Una vez que se haya enviado la incidencia de Soporte técnico, Microsoft descargará estos registros y le proporcionará acceso a ellos.
