---
title: Configuración de Azure Attestation para el servidor lógico de Azure SQL
description: Configure Azure Attestation para Always Encrypted con enclaves seguros en Azure SQL Database.
keywords: cifrar datos, cifrado sql, cifrado de base de datos, datos confidenciales, Always Encrypted, enclaves seguros, SGX, atestación
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviwer: vanto
ms.date: 01/15/2021
ms.openlocfilehash: a51aa15e1338380d4b4179e7fb8899273750c374
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106107187"
---
# <a name="configure-azure-attestation-for-your-azure-sql-logical-server"></a>Configuración de Azure Attestation para el servidor lógico de Azure SQL

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Always Encrypted con enclaves seguros para Azure SQL Database está actualmente en **versión preliminar pública.**

[Microsoft Azure Attestation](../../attestation/overview.md) es una solución para la atestación de entornos de ejecución de confianza (TEE), incluidos los enclaves de Intel Software Guard Extensions (Intel SGX). 

Para usar Azure Attestation para la atestación de enclaves de Intel SGX usados para [Always Encrypted con enclaves seguros](/sql/relational-databases/security/encryption/always-encrypted-enclaves) en Azure SQL Database, debe:

1. Crear un [proveedor de atestación](../../attestation/basic-concepts.md#attestation-provider) y configurarlo con la directiva de atestación recomendada.

2. Conceder al servidor lógico de Azure SQL acceso al proveedor de atestación.

> [!NOTE]
> La configuración de la atestación es responsabilidad del administrador de atestación. Consulte [Roles y responsabilidades al configurar la atestación y los enclaves de SGX](always-encrypted-enclaves-plan.md#roles-and-responsibilities-when-configuring-sgx-enclaves-and-attestation).

## <a name="requirements"></a>Requisitos

El servidor lógico de Azure SQL y el proveedor de atestación deben pertenecer al mismo inquilino de Azure Active Directory. No se admiten interacciones entre inquilinos. 

El servidor lógico de Azure SQL debe tener asignada una identidad de Azure AD. Como administrador de atestación, debe obtener la identidad de Azure AD del servidor del administrador de Azure SQL Database para ese servidor. Usará la identidad para conceder al servidor acceso al proveedor de atestación. 

Para obtener instrucciones sobre cómo crear un servidor con una identidad o asignar una identidad a un servidor existente mediante PowerShell y CLI de Azure, consulte [Asignar una identidad de Azure Active Directory (Azure AD) al servidor](transparent-data-encryption-byok-configure.md#assign-an-azure-active-directory-azure-ad-identity-to-your-server).

## <a name="create-and-configure-an-attestation-provider"></a>Creación y configuración de un proveedor de atestación

Un [proveedor de atestación](../../attestation/basic-concepts.md#attestation-provider) es un recurso de Azure Attestation que evalúa [solicitudes de atestación](../../attestation/basic-concepts.md#attestation-request) con [directivas de atestación](../../attestation/basic-concepts.md#attestation-request) y emite [tokens de atestación](../../attestation/basic-concepts.md#attestation-token). 

Las directivas de atestación se especifican mediante la [gramática de reglas de notificaciones](../../attestation/claim-rule-grammar.md).

Microsoft recomienda la siguiente directiva para la atestación de enclaves de Intel SGX que se usa para Always Encrypted en Azure SQL Database:

```output
version= 1.0;
authorizationrules 
{
       [ type=="x-ms-sgx-is-debuggable", value==false ]
        && [ type=="x-ms-sgx-product-id", value==4639 ]
        && [ type=="x-ms-sgx-svn", value>= 0 ]
        && [ type=="x-ms-sgx-mrsigner", value=="e31c9e505f37a58de09335075fc8591254313eb20bb1a27e5443cc450b6e33e5"] 
    => permit();
};
```

La directiva anterior comprueba lo siguiente:

- El enclave de Azure SQL Database no admite la depuración. 
  > Los enclaves se pueden cargar con la depuración deshabilitada o habilitada. La compatibilidad con la depuración está diseñada para permitir a los desarrolladores solucionar problemas del código que se ejecuta en un enclave. En un sistema de producción, la depuración podría permitir a un administrador examinar el contenido de enclave, lo que reduciría el nivel de protección que proporciona el enclave. La directiva recomendada deshabilita la depuración para asegurarse de que si un administrador malintencionado intenta activar la compatibilidad con la depuración mediante el uso de la máquina de enclave, se producirá un error en la atestación. 
- El id. del producto del enclave coincide con el id. del producto asignado a Always Encrypted con enclaves seguros.
  > Cada enclave tiene un id. del producto único que diferencia el enclave de otros enclaves. El id. del producto asignado al enclave de Always Encrypted es 4639. 
- El número de versión de seguridad (SVN) de la biblioteca es mayor que 0.
  > SVN permite a Microsoft responder a posibles errores de seguridad identificados en el código enclave. En caso de que se detecte y corrija un problema de seguridad, Microsoft implementará una nueva versión de enclave con un nuevo SVN (incrementado). La directiva recomendada anterior se actualizará para reflejar el nuevo SVN. Cuando actualiza la directiva para que coincida con la directiva recomendada, se asegura de que si un administrador malintencionado intenta cargar una enclave anterior e insegura, se producirá un error en la atestación.
- La biblioteca del enclave se ha firmado con la clave de firma de Microsoft (el valor de la notificación x-ms-sgx-mrsigner es el hash de la clave de firma).
  > Uno de los principales objetivos de la atestación es convencer a los clientes de que el binario que se ejecuta en el enclave es el binario que se supone que se ejecuta. Las directivas de atestación proporcionan dos mecanismos para este fin. Una es la notificación de **mrenclave**, que es el hash del archivo binario que se supone que se va a ejecutar en un enclave. El problema con **mrenclave** es que el hash binario cambia incluso con cambios triviales en el código, lo que dificulta la revisión del código que se ejecuta en el enclave. Por lo tanto, se recomienda el uso de **mrsigner**, que es un hash de una clave que se usa para firmar el binario del enclave. Cuando Microsoft revierte el enclave, **mrsigner** permanece igual, siempre y cuando la clave de firma no cambie. De esta manera, es factible implementar archivos binarios actualizados sin interrumpir las aplicaciones de los clientes. 

> [!IMPORTANT]
> Se crea un proveedor de atestación con la directiva predeterminada para enclaves de Intel SGX, lo cual no valida el código que se ejecuta dentro de enclave. Microsoft recomienda encarecidamente establecer la directiva recomendada anterior y no usar la directiva predeterminada para Always Encrypted con enclaves seguros.

Para obtener instrucciones sobre cómo crear un proveedor de atestación y configurar con una directiva de atestación, consulte:

- [Inicio rápido: Configuración de Azure Attestation con Azure Portal](../../attestation/quickstart-portal.md)
    > [!IMPORTANT]
    > Cuando configure la directiva de atestación con Azure Portal, establezca el tipo de atestación en `SGX-IntelSDK`.
- [Inicio rápido: Configuración de Azure Attestation con Azure PowerShell](../../attestation/quickstart-powershell.md)
    > [!IMPORTANT]
    > Cuando configure la directiva de atestación con Azure PowerShell, establezca el parámetro `Tee` en `SgxEnclave`.
- [Inicio rápido: Configuración de Azure Attestation con la CLI de Azure](../../attestation/quickstart-azure-cli.md)
    > [!IMPORTANT]
    > Cuando configure la directiva de atestación con la CLI de Azure, establezca el parámetro `attestation-type` en `SGX-IntelSDK`.

## <a name="determine-the-attestation-url-for-your-attestation-policy"></a>Determinación de la dirección URL de atestación de la directiva de atestación

Después de configurar una directiva de atestación, debe compartir la dirección URL de la atestación que haga referencia a la directiva y los administradores de las aplicaciones que usan Always Encrypted con enclaves seguros en Azure SQL Database. Los administradores o los usuarios de las aplicaciones deberán configurar sus aplicaciones con la dirección URL de atestación, de modo que puedan ejecutar instrucciones que usen enclaves seguros.

### <a name="use-powershell-to-determine-the-attestation-url"></a>Uso de PowerShell para determinar la dirección URL de atestación

Use el siguiente script para determinar la dirección URL de atestación:

```powershell
$attestationProvider = Get-AzAttestation -Name $attestationProviderName -ResourceGroupName $attestationResourceGroupName 
$attestationUrl = $attestationProvider.AttestUri + "/attest/SgxEnclave"
Write-Host "Your attestation URL is: " $attestationUrl 
```

### <a name="use-azure-portal-to-determine-the-attestation-url"></a>Uso de Azure Portal para determinar la dirección URL de atestación

1. En el panel de información general del proveedor de atestación, copie el valor de la propiedad URI de atestación en el portapapeles. Un URI de atestación debe tener el siguiente aspecto: `https://MyAttestationProvider.us.attest.azure.net`.

2. Anexe lo siguiente al URI de atestación: `/attest/SgxEnclave`. 

La dirección URL de atestación resultante debe tener el siguiente aspecto: `https://MyAttestationProvider.us.attest.azure.net/attest/SgxEnclave`

## <a name="grant-your-azure-sql-logical-server-access-to-your-attestation-provider"></a>Concesión de acceso al proveedor de atestación para el servidor lógico de Azure SQL

Durante el flujo de trabajo de atestación, el servidor lógico de Azure SQL que contiene la base de datos llama al proveedor de atestación para enviar una solicitud de atestación. Para que el servidor lógico de Azure SQL pueda enviar solicitudes de atestación, el servidor debe tener un permiso para la acción `Microsoft.Attestation/attestationProviders/attestation/read` en el proveedor de atestación. La manera recomendada de conceder el permiso es que el administrador del proveedor de atestación asigne la identidad de Azure AD del servidor al rol de lector de atestación para el proveedor de atestación, o el grupo de recursos que lo contiene.

### <a name="use-azure-portal-to-assign-permission"></a>Uso de Azure Portal para asignar el permiso

Para asignar la identidad de un servidor de Azure SQL al rol Lector de atestación para un proveedor de atestación, siga las instrucciones generales de [Asignación de roles de Azure mediante Azure Portal](../../role-based-access-control/role-assignments-portal.md). Cuando esté en el panel **Agregar asignación de roles**:

1. En la lista desplegable **Rol**, seleccione el rol **Lector de atestación**.
1. En el campo **Seleccionar**, escriba el nombre de su servidor de Azure SQL para buscarlo.

Vea la captura de pantalla siguiente para obtener un ejemplo.

![asignación de roles del lector de atestación](./media/always-encrypted-enclaves/attestation-provider-role-assigment.png)

> [!NOTE]
> Para que un servidor se muestre en el panel **Agregar asignación de roles**, el servidor debe tener una identidad de Azure AD asignada (consulte [Requisitos](#requirements)).

### <a name="use-powershell-to-assign-permission"></a>Uso de PowerShell para asignar permisos

1. Busque el servidor lógico de Azure SQL.

```powershell
$serverResourceGroupName = "<server resource group name>"
$serverName = "<server name>" 
$server = Get-AzSqlServer -ServerName $serverName -ResourceGroupName $serverResourceGroupName 
```
 
2. Asigne al servidor el rol Lector de atestación para el grupo de recursos que contiene su proveedor de atestación.

```powershell
$attestationResourceGroupName = "<attestation provider resource group name>"
New-AzRoleAssignment -ObjectId $server.Identity.PrincipalId -RoleDefinitionName "Attestation Reader" -ResourceGroupName $attestationResourceGroupName
```

Para más información, consulte [Asignación de roles de Azure mediante Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md#assign-role-examples).

## <a name="next-steps"></a>Pasos siguientes

- [Administración de claves para Always Encrypted con enclaves seguros](/sql/relational-databases/security/encryption/always-encrypted-enclaves-manage-keys)

## <a name="see-also"></a>Consulte también

- [Tutorial: Introducción a Always Encrypted con enclaves seguros en Azure SQL Database](always-encrypted-enclaves-getting-started.md)
