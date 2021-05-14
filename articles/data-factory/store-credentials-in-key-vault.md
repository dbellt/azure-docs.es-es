---
title: Almacenamiento de credenciales en Azure Key Vault
description: Aprenda a almacenar credenciales para almacenes de datos usados en una instancia de Azure Key Vault que Azure Data Factory pueda recuperar automáticamente en tiempo de ejecución.
author: nabhishek
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/13/2020
ms.author: abnarain
ms.openlocfilehash: b9c9d04e3667e95e9bcc02711755efa74559bc52
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482514"
---
# <a name="store-credential-in-azure-key-vault"></a>Almacenamiento de credenciales en Azure Key Vault

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Puede almacenar las credenciales de los almacenes de datos y los procesos en una instancia de [Azure Key Vault](../key-vault/general/overview.md). Azure Data Factory las recupera al ejecutar una actividad que usa el almacén de datos o el proceso.

Actualmente, todos los tipos de actividad, excepto la actividad personalizada, admiten esta característica. En cuanto a la configuración de conectores concretos, revise la sección "Propiedades del servicio vinculado" de [cada tema de conector](copy-activity-overview.md#supported-data-stores-and-formats) para obtener más información.

## <a name="prerequisites"></a>Prerrequisitos

Esta característica se basa en la identidad administrada de Data Factory. Aprenda cómo funciona en [Identidad administrada de Data Factory](data-factory-service-identity.md) y asegúrese de que su instancia de Data Factory tenga una asociada.

## <a name="steps"></a>Pasos

Para hacer referencia a una credencial almacenada en Azure Key Vault, deberá seguir estos pasos:

1. **Recuperación de la identidad administrada de Data Factory**: copie el valor de "Id. del objeto de identidad administrada" que se generó junto con la factoría. Si usa la interfaz de usuario de creación de ADF, el identificador del objeto de identidad administrada se mostrará en la ventana de creación del servicio vinculado de Azure Key Vault; también puede recuperarlo de Azure Portal, consulte la sección [Recuperación de la identidad administrada de Data Factory](data-factory-service-identity.md#retrieve-managed-identity).
2. **Concesión del acceso de identidad administrada a Azure Key Vault.** En el almacén de claves -> Directivas de acceso -> Agregar directiva de acceso, busque esta identidad administrada para conceder el permiso **Get** en el menú desplegable Permisos de secretos. De esta forma, la factoría designada puede acceder al secreto del almacén de claves.
3. **Creación de un servicio vinculado que apunte a Azure Key Vault.** Consulte [Servicio vinculado de Azure Key Vault](#azure-key-vault-linked-service).
4. **Creación de un servicio vinculado a un almacén de datos en el que se hace referencia al secreto correspondiente guardado en el almacén de claves.** Consulte [Secreto de referencia almacenado en el almacén de claves](#reference-secret-stored-in-key-vault).

## <a name="azure-key-vault-linked-service"></a>Servicio vinculado de Azure Key Vault

Las siguientes propiedades son compatibles con el servicio vinculado de Azure Key Vault:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AzureKeyVault**. | Sí |
| baseUrl | Especifique la dirección URL de Azure Key Vault. | Sí |

**Uso de la IU de creación:**

Seleccione **Conexiones** -> **Servicios vinculados** -> **Nuevo**. En el nuevo servicio vinculado, busque y seleccione "Azure Key Vault":

![Busque Azure Key Vault](media/store-credentials-in-key-vault/search-akv.png)

Seleccione el almacén de Azure Key Vault aprovisionado en el que se almacenan las credenciales. Puede hacer una **conexión de prueba** para asegurarse de que su conexión de AKV es válida. 

![Configuración de Azure Key Vault](media/store-credentials-in-key-vault/configure-akv.png)

**Ejemplo JSON:**

```json
{
    "name": "AzureKeyVaultLinkedService",
    "properties": {
        "type": "AzureKeyVault",
        "typeProperties": {
            "baseUrl": "https://<azureKeyVaultName>.vault.azure.net"
        }
    }
}
```

## <a name="reference-secret-stored-in-key-vault"></a>Secreto de referencia almacenado en el almacén de claves

Al configurar un campo en un servicio vinculado que hace referencia a un secreto del almacén de claves, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del campo debe establecerse en: **AzureKeyVaultSecret**. | Sí |
| secretName | Nombre del secreto en Azure Key Vault. | Sí |
| secretVersion | Versión del secreto en Azure Key Vault.<br/>Si no se especifica, siempre se usa la versión más reciente del secreto.<br/>Si se especifica, se usa la versión dada.| No |
| store | Hace referencia a un servicio vinculado de Azure Key Vault que se usa para almacenar las credenciales. | Sí |

**Uso de la IU de creación:**

Seleccione **Azure Key Vault** para campos secretos al crear la conexión al almacén de datos/proceso. Seleccione el servicio vinculado de Azure Key Vault aprovisionado y proporcione el **nombre del secreto**. También puede proporcionar una versión del secreto. 

>[!TIP]
>Para los conectores que usan cadena de conexión en el servicio vinculado, como SQL Server, Blob Storage, etc., puede elegir almacenar solo el campo secreto, por ejemplo, la contraseña, en AKV o la cadena de conexión completa. Puede encontrar ambas opciones en la interfaz de usuario.

![Configuración del secreto de Azure Key Vault](media/store-credentials-in-key-vault/configure-akv-secret.png)

**Ejemplo de JSON: (consulte la sección "Contraseña")**

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "typeProperties": {
            "deploymentType": "<>",
            "organizationName": "<>",
            "authenticationType": "<>",
            "username": "<>",
            "password": {
                "type": "AzureKeyVaultSecret",
                "secretName": "<secret name in AKV>",
                "store":{
                    "referenceName": "<Azure Key Vault linked service>",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
