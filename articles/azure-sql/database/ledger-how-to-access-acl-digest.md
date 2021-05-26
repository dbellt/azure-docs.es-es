---
title: Cómo obtener acceso a los resúmenes almacenados en Azure Confidential Ledger (ACL)
description: Cómo obtener acceso a los resúmenes almacenados en Azure Confidential Ledger (ACL) con el libro de contabilidad de Azure SQL Database
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: how-to
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: e5baf12de52543280d294fc68f326f53e045f70d
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388762"
---
# <a name="how-to-access-the-digests-stored-in-acl"></a>Cómo obtener acceso a los resúmenes almacenados en ACL

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

En este artículo se muestra cómo acceder a un resumen de [libro de contabilidad de Azure SQL Database](ledger-overview.md) almacenado en [Azure Confidential Ledger (ACL)](/azure/confidential-ledger/) para obtener garantías de integridad y seguridad de un extremo a otro. En este artículo se explica cómo obtener acceso a la información almacenada y comprobar su integridad.

## <a name="prerequisites"></a>Requisitos previos

- Python 2.7, 3.5.3 o versiones posteriores
- Tener una instancia de Azure SQL Database con el libro de contabilidad habilitado. Consulte [Inicio rápido: Creación de una instancia de Azure SQL Database con el libro de contabilidad habilitado](ledger-create-a-single-database-with-ledger-enabled.md) si aún no ha creado una.
- [Biblioteca cliente de Azure Confidential Ledger para Python](https://github.com/Azure/azure-sdk-for-python/blob/b42651ae4791aca8c9fbe282832b81badf798aa9/sdk/confidentialledger/azure-confidentialledger/README.md#create-a-client)
- Una instancia en ejecución de [Azure Confidential Ledger](/azure/confidential-ledger/).

## <a name="how-does-the-integration-work"></a>¿Cómo funciona la integración?

El servidor de Azure SQL calcula periódicamente los resúmenes de las [bases de datos del libro de contabilidad](ledger-overview.md#ledger-database) y los almacena en Azure Confidential Ledger. Un usuario puede validar en cualquier momento la integridad de los datos descargando los resúmenes de Azure Confidential Ledger y comparándolos con los resúmenes almacenados en el libro de contabilidad de Azure SQL Database. En los pasos siguientes se explica el proceso.

## <a name="step-1---find-the-digest-location"></a>Paso 1: buscar la ubicación del resumen

> [!NOTE]
> La consulta devolverá más de una fila si se usaron varias instancias de Azure Confidential Ledger para almacenar el resumen. Para cada fila, repita los pasos del 2 al 6 para descargar los resúmenes de todas las instancias de Azure Confidential Ledger.

Ejecute la siguiente consulta mediante [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms). El resultado muestra el punto de conexión de la instancia de Azure Confidential Ledger donde se almacenan los resúmenes.

```sql
SELECT * FROM sys.database_ledger_digest_locations WHERE path like '%.confidential-ledger.azure.com%
```

## <a name="step-2---determine-the-subledgerid"></a>Paso 2: determinar el valor Subledgerid

Nos interesa el valor de la columna de ruta de acceso de la salida de la consulta. Consta de dos partes, es decir, `host name` y `subledgerid`. Por ejemplo, en la dirección URL `https://contoso-ledger.confidential-ledger.azure.com/sqldbledgerdigests/ledgersvr2/ledgerdb/2021-04-13T21:20:51.0000000`, `host name` es `https://contoso-ledger.confidential-ledger.azure.com` y `subledgerid` es `sqldbledgerdigests/ledgersvr2/ledgerdb/2021-04-13T21:20:51.0000000`. Usaremos esto en el paso 4 para descargar los resúmenes.

## <a name="step-3---obtain-an-azure-ad-token"></a>Paso 3: obtención de un token de Azure AD

La API de Azure Confidential Ledger acepta un token de portador de Azure Active Directory (Azure AD) como identidad del autor de la llamada. Esta identidad necesita obtener acceso a ACL a través de Azure Resource Manager durante el aprovisionamiento. El usuario que había habilitado el libro de contabilidad en SQL Database recibe automáticamente acceso de administrador a Azure Confidential Ledger. Para obtener un token, el usuario debe autenticarse mediante la [CLI de Azure](/cli/azure/install-azure-cli) con la misma cuenta que se usó con Azure Portal. Una vez que el usuario se ha autenticado, puede usar [DefaultAzureCredentials()](/dotnet/api/azure.identity.defaultazurecredential) para recuperar un token de portador y llamar a la API de Azure Confidential Ledger.

Inicie sesión en Azure AD mediante la identidad con acceso a ACL.

```azure-cli
az login
```

Recupere el token de portador.

```python
from azure.identity import DefaultAzureCredential
credential = DefaultAzureCredential()
```

## <a name="step-4---download-the-digests-from-azure-confidential-ledger"></a>Paso 4: descargar los resúmenes de Azure Confidential Ledger

El siguiente script de Python descarga los resúmenes de Azure Confidential Ledger. El script usa la [biblioteca cliente de Azure Confidential Ledger para Python](https://github.com/Azure/azure-sdk-for-python/blob/b42651ae4791aca8c9fbe282832b81badf798aa9/sdk/confidentialledger/azure-confidentialledger/README.md#create-a-client).

```python
from azure.identity import DefaultAzureCredential
from azure.confidentialledger import ConfidentialLedgerClient
from azure.confidentialledger.identity_service import ConfidentialLedgerIdentityServiceClient

ledger_id = "contoso-ledger"
identity_server_url = "https://identity.confidential-ledger.core.azure.com"
sub_ledger_id = "sqldbledgerdigests/ledgersvr2/ledgerdb/2021-04-13T21:20:51.0000000"
ledger_host_url = f"https://{ledger_id}.confidential-ledger.azure.com"
initial_path = f"/app/transactions?api-version=0.1-preview&subLedgerId={sub_ledger_id}"

identity_client = ConfidentialLedgerIdentityServiceClient(identity_server_url)
network_identity = identity_client.get_ledger_identity(
    ledger_id=ledger_id
)

ledger_tls_cert_file_name = f"{ledger_id}_certificate.pem"
with open(ledger_tls_cert_file_name, "w") as cert_file:
    cert_file.write(network_identity.ledger_tls_certificate)

credential = DefaultAzureCredential()
ledger_client = ConfidentialLedgerClient(
    endpoint=ledger_host_url, 
    credential=credential,
    ledger_certificate_path=ledger_tls_cert_file_name
)

ranged_result = ledger_client.get_ledger_entries(
    sub_ledger_id=sub_ledger_id
)

entries = 0

for entry in ranged_result:
    entries += 1
    print(f"\nTransaction id {entry.transaction_id} contents: {entry.contents}")

if entries == 0:
    print("\n***No digests are found for the supplied SubledgerID.")
else:
    print("\n***No more digests were found for the supplied SubledgerID.")
```

## <a name="step-5---download-the-digests-from-the-sql-server"></a>Paso 5: descargar los resúmenes de SQL Server

> [!NOTE]
> Esta es una manera de confirmar que los hash almacenados en el libro de contabilidad de Azure SQL Database no han cambiado con el tiempo. Para obtener una auditoría completa de la integridad del libro de contabilidad de Azure SQL Database, consulte [Comprobación de una tabla de libro de contabilidad para detectar alteraciones](ledger-verify-database.md).

Con [SSMS](/sql/ssms/download-sql-server-management-studio-ssms), ejecute la siguiente consulta. La consulta devuelve los resúmenes de los bloques de Genesis.

```sql
SELECT * FROM sys.database_ledger_blocks
```

## <a name="step-6---comparison"></a>Paso 6: comparación

Compare el resumen recuperado de Azure Confidential Ledger con el resumen devuelto de la base de datos SQL con `block_id` como clave. Por ejemplo, el resumen de `block_id` = `1` es el valor de la columna `previous_block_hash` de la fila `block_id`= `2`. De forma similar, `block_id` = `3` es el valor de la columna `previous_block_id` de la fila `block_id` = `4`. Una discrepancia en el valor de hash es un indicador de una posible alteración de los datos.

Si sospecha que se han alterado los datos, consulte [Cómo comprobar una tabla de libro de contabilidad para detectar una alteración](ledger-verify-database.md) para realizar una auditoría completa del libro de contabilidad de Azure SQL Database.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al libro de contabilidad de Azure SQL Database](ledger-overview.md)
- [Libro de contabilidad de base de datos](ledger-database-ledger.md)
- [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md)
- [Tablas de libro de contabilidad de solo anexión](ledger-append-only-ledger-tables.md)
- [Tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md)
- [Comprobación de una tabla de libro de contabilidad para detectar alteraciones](ledger-verify-database.md)