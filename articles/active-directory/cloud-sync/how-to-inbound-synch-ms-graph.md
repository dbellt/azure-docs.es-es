---
title: Configuración mediante programación de la sincronización en la nube mediante MS Graph API
description: En este tema se describe cómo habilitar la sincronización de entrada con solo Graph API.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/04/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8fe220cf7b5cb8b67e5ab7ded221494e89a28aa5
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530260"
---
# <a name="how-to-programmatically-configure-cloud-sync-using-ms-graph-api"></a>Configuración mediante programación de la sincronización en la nube mediante MS Graph API

En el documento siguiente se describe cómo replicar un perfil de sincronización desde cero usando solo instancias de MS Graph API.  
La estructura de este procedimiento consta de los siguientes pasos.  Son las siguientes:

- [Configuración mediante programación de la sincronización en la nube mediante MS Graph API](#how-to-programmatically-configure-cloud-sync-using-ms-graph-api)
  - [Configuración básica](#basic-setup)
    - [Habilitación de las marcas de inquilino](#enable-tenant-flags)
  - [Creación de entidades de servicio](#create-service-principals)
  - [Creación del trabajo de sincronización](#create-sync-job)
  - [Actualización del dominio de destino](#update-targeted-domain)
  - [Habilitación de la sincronización de hashes de contraseñas en la hoja de configuración](#enable-sync-password-hashes-on-configuration-blade)
  - [Eliminaciones accidentales](#accidental-deletes)
    - [Habilitación y establecimiento del umbral](#enabling-and-setting-the-threshold)
    - [Permitir eliminaciones](#allowing-deletes)
  - [Inicio del trabajo de sincronización](#start-sync-job)
  - [Revisión del estado](#review-status)
  - [Pasos siguientes](#next-steps)

Use estos comandos del [Módulo Microsoft Azure Active Directory para Windows PowerShell](/powershell/module/msonline/) para permitir la sincronización de un inquilino de producción, un requisito previo para poder llamar al servicio web de administración de ese inquilino.

## <a name="basic-setup"></a>Configuración básica

### <a name="enable-tenant-flags"></a>Habilitación de las marcas de inquilino

 ```PowerShell
 Connect-MsolService ('-AzureEnvironment <AzureEnvironmnet>')
 Set-MsolDirSyncEnabled -EnableDirSync $true
 ```
El primero de estos dos comandos requiere las credenciales de Azure Active Directory. Estos commandlets identifican de manera implícita el inquilino y permiten que se sincronice.

## <a name="create-service-principals"></a>Creación de entidades de servicio
A continuación, es necesario crear la [aplicación o la entidad de servicio de AD2AAD](/graph/api/applicationtemplate-instantiate?view=graph-rest-beta&tabs=http&preserve-view=true).

Debe usar el identificador de aplicación 1a4721b3-e57f-4451-ae87-ef078703ec94. El valor de displayName es la dirección URL del dominio de AD, si se usa en el portal (por ejemplo, contoso.com), pero puede tener un nombre diferente.

 ```
 POST https://graph.microsoft.com/beta/applicationTemplates/1a4721b3-e57f-4451-ae87-ef078703ec94/instantiate
 Content-type: application/json
 {
    displayName: [your app name here]
 }
 ```


## <a name="create-sync-job"></a>Creación del trabajo de sincronización
La salida del comando anterior devolverá el valor de objectId de la entidad de servicio que se creó. En este ejemplo, es 614ac0e9-a59b-481f-bd8f-79a73d167e1c.  Use Microsoft Graph para agregar un trabajo de sincronización a esa entidad de servicio.  

Puede encontrar documentación sobre la creación de un trabajo de sincronización [aquí](/graph/api/synchronization-synchronizationjob-post?tabs=http&view=graph-rest-beta&preserve-view=true).

Si no registró el identificador anterior, puede encontrar la entidad de servicio mediante la ejecución de la siguiente llamada de MS Graph. Para hacer esa llamada, necesitará los permisos Directory.Read.All:
 
 `GET https://graph.microsoft.com/beta/servicePrincipals `

A continuación, busque el nombre de la aplicación en la salida.

Ejecute los dos comandos siguientes para crear dos trabajos: uno para el aprovisionamiento de usuarios o grupos y otro para la sincronización del hash de contraseñas. La solicitud es la misma dos veces, pero con distintos identificadores de plantilla.


Llame a las dos solicitudes siguientes:

 ```
 POST https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs
 Content-type: application/json
 {
 "templateId":"AD2AADProvisioning"
 } 
 ```

 ```
 POST https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs
 Content-type: application/json
 {
 "templateId":"AD2AADPasswordHash"
 }
 ```

Necesitará dos llamadas si quiere crear ambas.

Valor devuelto de ejemplo (para el aprovisionamiento):

 ```
HTTP 201/Created
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#servicePrincipals('614ac0e9-a59b-481f-bd8f-79a73d167e1c')/synchronization/jobs/$entity",
    "id": "AD2AADProvisioning.fc96887f36da47508c935c28a0c0b6da",
    "templateId": "ADDCInPassthrough",
    "schedule": {
        "expiration": null,
        "interval": "PT40M",
        "state": "Disabled"
    },
    "status": {
        "countSuccessiveCompleteFailures": 0,
        "escrowsPruned": false,
        "code": "Paused",
        "lastExecution": null,
        "lastSuccessfulExecution": null,
        "lastSuccessfulExecutionWithExports": null,
        "quarantine": null,
        "steadyStateFirstAchievedTime": "0001-01-01T00:00:00Z",
        "steadyStateLastAchievedTime": "0001-01-01T00:00:00Z",
        "troubleshootingUrl": null,
        "progress": [],
        "synchronizedEntryCountByType": []
    }
}
```

## <a name="update-targeted-domain"></a>Actualización del dominio de destino
En este inquilino, el identificador de objeto y el identificador de aplicación de la entidad de servicio son los siguientes:

ObjectId: 8895955e-2e6c-4d79-8943-4d72ca36878f AppId: 00000014-0000-0000-c000-000000000000 DisplayName: testApp

Vamos a tener que actualizar el dominio de destino de esta configuración, por lo que habrá que actualizar los secretos de este dominio.

Asegúrese de que el nombre de dominio que usa sea la misma dirección URL que estableció para el controlador de dominio local.

 ```
 PUT – https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/secrets
 ```
 Agregue el siguiente par de clave-valor en la matriz de valores siguiente, en función de lo que está intentando hacer:
 - Habilite las marcas de inquilino tanto de PHS como de sincronización {clave: "AppKey", valor: "{"appKeyScenario":"AD2AADPasswordHash"}" }
 
 - Habilite solo la marca de inquilino de sincronización (no la active en PHS) {clave: "AppKey", valor: "{"appKeyScenario":"AD2AADProvisioning"}" }
 ```
 Request body –
 {
    "value": [
               {
                 "key": "Domain",
                 "value": "{\"domain\":\"ad2aadTest.com\"}"
               }
             ]
  }
```

La respuesta esperada es: HTTP 204/Sin contenido.

Aquí, el valor de "dominio" resaltado es el nombre del dominio local Active Directory desde el que se van a aprovisionar las entradas para Azure Active Directory.

## <a name="enable-sync-password-hashes-on-configuration-blade"></a>Habilitación de la sincronización de hashes de contraseñas en la hoja de configuración

 En esta sección se trata la habilitación de la sincronización de hashes de contraseñas para una configuración determinada. Esto es diferente del secreto de AppKey que habilita la marca de características a nivel de inquilino, que es solo para un único dominio o configuración. Tendrá que establecer la clave de aplicación en la de PHS para que este funcione de un extremo a otro.

1. Tome el esquema (advertencia: es bastante grande). 
 ```
 GET –https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ [AD2AADProvisioningJobId]/schema
 ```
2. Tome esta asignación de atributos de CredentialData:
 ``` 
 {
 "defaultValue": null,
 "exportMissingReferences": false,
 "flowBehavior": "FlowWhenChanged",
 "flowType": "Always",
 "matchingPriority": 0,
 "targetAttributeName": "CredentialData",
 "source": {
 "expression": "[PasswordHash]",
 "name": "PasswordHash",
 "type": "Attribute",
 "parameters": []
 }
 ```
3. Busque las siguientes asignaciones de objeto con los siguientes nombres en el esquema.
 - Provision Active Directory users
 - Provision Active Directory inetOrgPersons

 Las asignaciones de objetos están dentro de schema.synchronizationRules[0].objectMappings (por ahora puede suponer que solo hay una regla de sincronización).

4. Tome la asignación de CredentialData del paso (2) e insértela en las asignaciones de objetos en el paso (3).

 La asignación de objeto tiene una apariencia similar a la siguiente:
 ```
 {
 "enabled": true,
 "flowTypes": "Add,Update,Delete",
 "name": "Provision Active Directory users",
 "sourceObjectName": "user",
 "targetObjectName": "User",
 "attributeMappings": [
 ...
 } 
 ```
 Copie la asignación del paso **Creación de trabajos AD2AADProvisioning y AD2AADPasswordHash** anterior y péguela en la matriz attributeMappings. 

 No importa el orden de los elementos de esta matriz (el back-end se ordenará automáticamente). Tenga cuidado al agregar esta asignación de atributos si el nombre ya existe en la matriz (por ejemplo, si ya hay un elemento en attributeMappings que tiene targetAttributeName CredentialData): puede obtener errores de conflicto, o bien las asignaciones preexistentes y nuevas se pueden combinar (normalmente no es el resultado deseado). El back-end no elimina los duplicados automáticamente. 

 Recuerde hacer esto tanto para users como para inetOrgpersons.

5. Guarde el esquema que ha creado. 
 ```
 PUT –
 https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ [AD2AADProvisioningJobId]/schema
```

 Agregue el esquema al cuerpo de la solicitud. 

## <a name="accidental-deletes"></a>Eliminaciones accidentales
En esta sección se explica cómo habilitar o deshabilitar y usar mediante programación [eliminaciones accidentales](how-to-accidental-deletes.md).


### <a name="enabling-and-setting-the-threshold"></a>Habilitación y establecimiento del umbral
Hay dos valores por trabajo que puede usar, estos son:

 - DeleteThresholdEnabled: Habilita la prevención de eliminación accidental del trabajo cuando se establece en "true". Está establecido en "true" de manera predeterminada.
 - DeleteThresholdValue: Define el número máximo de eliminaciones que se permitirán en cada ejecución del trabajo cuando la prevención de eliminaciones accidentales esté habilitada. De manera predeterminada, el valor se establece en 500.  Por lo tanto, si el valor está establecido en 500, el número máximo de eliminaciones permitidas será de 499 en cada ejecución.

Los valores de umbral de eliminación forman parte del `SyncNotificationSettings` y se pueden modificar mediante Graph. 

Vamos a tener que actualizar el valor de SyncNotificationSettings que esta configuración tiene como destino, por lo que habrá que actualizar los secretos.

 ```
 PUT – https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/secrets
 ```

 Agregue el siguiente par de clave-valor en la matriz de valores siguiente, en función de lo que está intentando hacer:

```
 Request body -
 {
   "value":[
             {
               "key":"SyncNotificationSettings",
               "value": "{\"Enabled\":true,\"Recipients\":\"foobar@xyz.com\",\"DeleteThresholdEnabled\":true,\"DeleteThresholdValue\":50}"
              }
            ]
  }


```

El valor "Habilitado" en el ejemplo anterior es para habilitar o deshabilitar los correos electrónicos de notificación cuando el trabajo se pone en cuarentena.


Actualmente, no se admiten solicitudes PATCH para secretos, por lo que tendrá que agregar todos los valores en el cuerpo de la solicitud PUT (como en el ejemplo anterior) para conservar los demás valores.

Los valores existentes para todos los secretos se pueden recuperar mediante: 

```
GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets 
```

### <a name="allowing-deletes"></a>Permitir eliminaciones
Para permitir que las eliminaciones fluyan una vez que el trabajo se ponga en cuarentena, debe emitir un reinicio con solo "ForceDeletes" como ámbito. 

```
Request:
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/restart
```

```
Request Body:
{
  "criteria": {"resetScope": "ForceDeletes"}
}
```






## <a name="start-sync-job"></a>Inicio del trabajo de sincronización
El trabajo se puede volver a recuperar con el siguiente comando:

 `GET https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ ` 

Puede encontrar documentación sobre cómo recuperar trabajos [aquí](/graph/api/synchronization-synchronizationjob-list?tabs=http&view=graph-rest-beta&preserve-view=true). 
 
Para iniciar el trabajo, emita esta solicitud con el identificador de objeto de la entidad de servicio creada en el primer paso y el identificador de trabajo devuelto por la solicitud que creó el trabajo.

Puede encontrar documentación sobre cómo iniciar un trabajo [aquí](/graph/api/synchronization-synchronizationjob-start?tabs=http&view=graph-rest-beta&preserve-view=true). 

 ```
 POST  https://graph.microsoft.com/beta/servicePrincipals/8895955e-2e6c-4d79-8943-4d72ca36878f/synchronization/jobs/AD2AADProvisioning.fc96887f36da47508c935c28a0c0b6da/start
 ```

La respuesta esperada es: HTTP 204/Sin contenido.

Otros comandos para controlar el trabajo se documentan [aquí](/graph/api/resources/synchronization-synchronizationjob?view=graph-rest-beta&preserve-view=true).
 
Para reiniciar un trabajo, usaría:

 ```
 POST  https://graph.microsoft.com/beta/servicePrincipals/8895955e-2e6c-4d79-8943-4d72ca36878f/synchronization/jobs/AD2AADProvisioning.fc96887f36da47508c935c28a0c0b6da/restart
 {
   "criteria": {
       "resetScope": "Full"
   }
 }
 ```

## <a name="review-status"></a>Revisión del estado
Recupere los estados de trabajo mediante:

 ```
 GET https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ 
 ```

Busque los detalles pertinentes en la sección "estado" del objeto devuelto.

## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es la sincronización en la nube de Azure AD Connect?](what-is-cloud-sync.md)
- [Transformaciones](how-to-transformation.md)
- [API de sincronización de Azure AD](/graph/api/resources/synchronization-overview?view=graph-rest-beta&preserve-view=true)
