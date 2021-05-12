---
title: Cifrado de credenciales en Azure Data Factory
description: Aprenda a cifrar y almacenar credenciales para sus almacenes de datos locales en una máquina con Integration Runtime autohospedado.
author: lrtoyou1223
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: lle
ms.openlocfilehash: 57b31c77943b41d76b265a8cea76c9882cf1fbf3
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109486726"
---
# <a name="encrypt-credentials-for-on-premises-data-stores-in-azure-data-factory"></a>Cifrado de credenciales de almacenes de datos locales en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Puede cifrar y almacenar las credenciales de los almacenes de datos locales (servicios vinculados con información confidencial) en una máquina con Integration Runtime autohospedado. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Se pasa un archivo de definición JSON con credenciales al cmdlet <br/>cmdlet [**New-AzDataFactoryV2LinkedServiceEncryptedCredential**](/powershell/module/az.datafactory/New-AzDataFactoryV2LinkedServiceEncryptedCredential) para generar un archivo de definición JSON de salida con las credenciales cifradas. A continuación, se usa la definición de JSON actualizada para crear los servicios vinculados.

## <a name="author-sql-server-linked-service"></a>Creación del servicio vinculado de SQL Server
Cree un archivo JSON denominado **SqlServerLinkedService.json** en cualquier carpeta con el siguiente contenido:  

Reemplace `<servername>`, `<databasename>`, `<username>` y `<password>` por los valores de SQL Server antes de guardar el archivo. Y, reemplace `<integration runtime name>` por el nombre de su instancia de Integration Runtime. 

```json
{
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Server=<servername>;Database=<databasename>;User ID=<username>;Password=<password>;Timeout=60"
        },
        "connectVia": {
            "type": "integrationRuntimeReference",
            "referenceName": "<integration runtime name>"
        },
        "name": "SqlServerLinkedService"
    }
}
```

## <a name="encrypt-credentials"></a>Cifrado de las credenciales
Para cifrar los datos confidenciales de la carga de JSON en un entorno de ejecución de integración autohospedado, ejecute **New-AzDataFactoryV2LinkedServiceEncryptedCredential** y pase la carga de JSON. Este cmdlet garantiza que las credenciales se cifran utilizando DPAPI y se almacenan en el nodo de Integration Runtime autohospedado de manera local. La carga de salida que contiene la referencia cifrada a la credencial se puede redirigir a otro archivo JSON (en este caso, "encryptedLinkedService.json").

```powershell
New-AzDataFactoryV2LinkedServiceEncryptedCredential -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "SqlServerLinkedService" -DefinitionFile ".\SQLServerLinkedService.json" > encryptedSQLServerLinkedService.json
```

## <a name="use-the-json-with-encrypted-credentials"></a>Uso de JSON con credenciales cifradas
Ahora, use el archivo JSON de salida del comando anterior que contiene la credencial cifrada para configurar **SqlServerLinkedService**.

```powershell
Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "EncryptedSqlServerLinkedService" -DefinitionFile ".\encryptedSqlServerLinkedService.json" 
```

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las consideraciones de seguridad en el movimiento de datos, consulte [Consideraciones sobre la seguridad del movimiento de datos](data-movement-security-considerations.md).

