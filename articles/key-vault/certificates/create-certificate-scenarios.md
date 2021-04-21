---
title: Supervisión y administración de la creación de certificados
description: Escenarios que muestran una variedad de opciones para la creación, la supervisión y la interacción con el proceso de creación de certificados con Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: 8e4acb5195497dd31f466829b1cde301ba9696b3
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751101"
---
# <a name="monitor-and-manage-certificate-creation"></a>Supervisión y administración de la creación de certificados
Se aplica a: Azure

escenarios y operaciones que se describen en este artículo son:

- Solicitar un certificado KV con un emisor admitido
- Obtener una solicitud pendiente, el estado de la solicitud es "inProgress"
- Obtener una solicitud pendiente, el estado de la solicitud es "completed"
- Obtener una solicitud pendiente, el estado de la solicitud pendiente es "canceled" o "failed"
- Obtener una solicitud pendiente, el estado de la solicitud pendiente es "deleted" u "overwritten"
- Crear (o importar) cuando exista una solicitud pendiente, el estado es "inProgress"
- Combinar cuando se cree una solicitud pendiente con un emisor (por ejemplo, DigiCert)
- Solicitar una cancelación mientras el estado de la solicitud pendiente es "inProgress"
- Eliminar un objeto de solicitud pendiente
- Crear un certificado KV manualmente
- Combinar cuando se crea una solicitud pendiente, creación manual de certificados

## <a name="request-a-kv-certificate-with-a-supported-issuer"></a>Solicitar un certificado KV con un emisor admitido 

|Método|URI de solicitud|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

Los siguientes ejemplos requieren un objeto denominado "mydigicert" que ya esté disponible en el almacén de claves con el proveedor emisor DigiCert. El emisor de certificados es una entidad que se representa en Azure Key Vault (KV) como un recurso CertificateIssuer. Se utiliza para proporcionar información sobre el origen de un certificado de KV; el nombre del emisor, el proveedor, las credenciales y otros detalles administrativos.

### <a name="request"></a>Solicitud

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "mydigicert",
      "cty": "OV-SSL",
    }
  }
}
```

### <a name="response"></a>Response

```
StatusCode: 202, ReasonPhrase: 'Accepted'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "mydigicert"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "InProgress",
  "status_details": "Pending certificate created. Certificate request is in progress. This may take some time based on the issuer provider. Please check again later",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="get-pending-request---request-status-is-inprogress"></a>Obtener una solicitud pendiente, el estado de la solicitud es "inProgress"

|Método|URI de solicitud|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Solicitud
GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

O BIEN

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

> [!NOTE]
> Si se especifica *request_id* en la consulta, sirve como filtro. Si el valor de *request_id* en la consulta y en el objeto pendiente son diferentes, se devuelve un código de estado HTTP de 404.

### <a name="response"></a>Response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "inProgress",
  "status_details": "…",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="get-pending-request---request-status-is-complete"></a>Obtener una solicitud pendiente, el estado de la solicitud es "completed"

### <a name="request"></a>Solicitud

|Método|URI de solicitud|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

O BIEN

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "completed",
  "request_id": "a76827a18b63421c917da80f28e9913d",
  "target": “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"
}

```

## <a name="get-pending-request---pending-request-status-is-canceled-or-failed"></a>Obtener una solicitud pendiente, el estado de la solicitud pendiente es "canceled" o "failed"

### <a name="request"></a>Solicitud

|Método|URI de solicitud|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

O BIEN

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "failed",
  "status_details": "",
  "request_id": "a76827a18b63421c917da80f28e9913d",
  "error": {
    "code": "<errorcode>",
    "message": "<message>"
  }
}

```

> [!NOTE]
> El valor de *errorcode* puede ser un error del emisor del certificado o de solicitud rechazada, según si se trata de un error del emisor o del usuario, respectivamente.

## <a name="get-pending-request---pending-request-status-is-deleted-or-overwritten"></a>Obtener una solicitud pendiente, el estado de la solicitud pendiente es "deleted" u "overwritten"
Un objeto pendiente se puede eliminar o sobrescribir mediante una operación de creación o importación cuando su estado no es "inProgress".

|Método|URI de solicitud|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Solicitud
GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

O BIEN

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Response

```
StatusCode: 404, ReasonPhrase: 'Not Found'
{
  "error": {
    "code": "PendingCertificateNotFound",
    "message": "…"
  }
}

```

## <a name="create-or-import-when-pending-request-exists---status-is-inprogress"></a>Crear (o importar) cuando exista una solicitud pendiente, el estado es "inProgress"
Un objeto pendiente tiene cuatro estados posibles: "inprogress", "canceled", "failed" o "completed".

Cuando el estado de una solicitud pendiente es "inprogress", las operaciones de creación (e importación) generan un error con un código de estado HTTP 409 (conflicto).

Para corregir un conflicto:

- Si el certificado se crea manualmente, puede completar el certificado KV mediante la realización de una combinación o eliminación sobre el objeto pendiente.

- Si el certificado se va a crear con un emisor, puede esperar a que el certificado se complete, genere un error o se cancele. Como alternativa, puede eliminar el objeto pendiente.

> [!NOTE]
> Es posible que la eliminación de un objeto pendiente no cancele la solicitud del certificado X.509 con el proveedor.

|Método|URI de solicitud|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

### <a name="request"></a>Solicitud

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "mydigicert"
    }
  }
}
```

### <a name="response"></a>Response

```
StatusCode: 409, ReasonPhrase: 'Conflict'
{
  "error": {
    "code": "Forbidden",
    "message": "A new key vault certificate can not be created or imported while a pending key vault certificate's status is inProgress."
  }
}

```

## <a name="merge-when-pending-request-is-created-with-an-issuer"></a>Combinar cuando se cree una solicitud pendiente con un emisor
La combinación no se permite cuando un objeto pendiente se crea con un emisor, pero se permite cuando su estado es "inProgress". 

Si la solicitud para crear el certificado X.509 genera error o se cancela por algún motivo, y si un certificado X.509 se puede recuperar por medios fuera de banda, se puede realizar una operación de combinación para completar el certificado KV.

|Método|URI de solicitud|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|

### <a name="request"></a>Solicitud

```json
{
  "x5c": [ "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8=" ]
}

```

### <a name="response"></a>Response

```json
StatusCode: 403, ReasonPhrase: 'Forbidden'
{
  "error": {
    "code": "Forbidden",
    "message": "Merge is forbidden on pending object created with issuer : <issuer-name> while it is in progess."
  }
}

```

## <a name="request-a-cancellation-while-the-pending-request-status-is-inprogress"></a>Solicitar una cancelación mientras el estado de la solicitud pendiente es "inProgress"
Solo se puede solicitar una cancelación. Una solicitud puede ser o no cancelada. Si una solicitud no está "inProgress", se devuelve un estado HTTP 400 (solicitud incorrecta).

|Método|URI de solicitud|
|------------|-----------------|
|PATCH|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Solicitud
PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

O BIEN

PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

```json
{
  "cancellation_requested": true
}

```

### <a name="response"></a>Response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": true,
  "status": "inProgress",
  "status_details": "…",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}
```

## <a name="delete-a-pending-request-object"></a>Eliminar un objeto de solicitud pendiente

> [!NOTE]
> Es posible que la eliminación de un objeto pendiente cancele o no la solicitud del certificado X.509 con el proveedor.

|Método|URI de solicitud|
|------------|-----------------|
|Delete|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Solicitud
DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

O BIEN

DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>Response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "inProgress",
  "request_id": "a76827a18b63421c917da80f28e9913d",
}
```

## <a name="create-a-kv-certificate-manually"></a>Crear un certificado KV manualmente
Puede crear un certificado emitido con una entidad de certificación de su elección mediante un proceso de creación manual. Establezca el nombre del emisor como "Unknown" o no especifique el campo del emisor.

|Método|URI de solicitud|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

### <a name="request"></a>Solicitud

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "Unknown"
    }
  }
}

```

### <a name="response"></a>Response

```
StatusCode: 202, ReasonPhrase: 'Accepted'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "Unknown"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "status": "inProgress",
  "status_details": "Pending certificate created. Please Perform Merge to complete the request.",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="merge-when-a-pending-request-is-created---manual-certificate-creation"></a>Combinar cuando se crea una solicitud pendiente, creación manual de certificados

|Método|URI de solicitud|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|

### <a name="request"></a>Solicitud

```json
{
  "x5c": [ "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8=" ]
}

```

|Nombre del elemento|Obligatorio|Tipo|Versión|Descripción|
|------------------|--------------|----------|-------------|-----------------|
|x5c|Sí|array|\<introducing version>|Cadena de certificados X.509 como matriz de cadenas de base 64.|

### <a name="response"></a>Response

```
StatusCode: 201, ReasonPhrase: 'Created'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"
{
    "id": "https mykeyvault.vault.azure.net/certificates/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "kid": "https:// mykeyvault.vault.azure.net/keys/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "sid": " mykeyvault.vault.azure.net/secrets/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "cer": "……de34534……",
    "x5t": "n14q2wbvyXr71Pcb58NivuiwJKk",
    "attributes": {
        "enabled": true,
        "exp": 1530394215,
        "nbf": 1435699215,
        "created": 1435699919,
        "updated": 1435699919
    },
    "pending": {
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/pending"
    },
    "policy": {
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/policy",
        "key_props": {
            "exportable": false,
            "kty": "RSA",
            "key_size": 2048,
            "reuse_key": false
        },
        "secret_props": {
            "contentType": "application/x-pkcs12"
        },
        "x509_props": {
            "subject": "CN=Mycert1",
            "ekus": ["1.3.6.1.5.5.7.3.1", "1.3.6.1.5.5.7.3.2"],
            "validity_months": 12
        },
        "lifetime_actions": [{
            "trigger": {
                "lifetime_percentage": 80
            },
            "action": {
                "action_type": "EmailContacts"
            }
        }],
        "issuer": {
            "name": "Unknown"
        },
        "attributes": {
            "enabled": true,
            "created": 1435699811,
            "updated": 1435699811
        }
    }
}

```
