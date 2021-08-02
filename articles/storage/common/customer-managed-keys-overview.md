---
title: Claves administradas por el cliente para el cifrado de cuentas
titleSuffix: Azure Storage
description: Puede usar su propia clave de cifrado para proteger los datos de la cuenta de almacenamiento. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. Las claves administradas por el cliente ofrecen más flexibilidad para administrar controles de acceso.
services: storage
author: tamram
ms.service: storage
ms.date: 06/01/2021
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: cd2a265c5d4c339fa6e50338949cbf643314a3ee
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111371328"
---
# <a name="customer-managed-keys-for-azure-storage-encryption"></a>Claves administradas por el cliente para el cifrado de Azure Storage

Puede usar su propia clave de cifrado para proteger los datos de la cuenta de almacenamiento. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. Las claves administradas por el cliente ofrecen más flexibilidad para administrar controles de acceso.

Debe usar uno de los siguientes almacenes de claves de Azure para almacenar las claves administradas por el cliente:

- [Azure Key Vault](../../key-vault/general/overview.md)
- [Módulo de seguridad de hardware (HSM) administrado por Azure Key Vault (versión preliminar)](../../key-vault/managed-hsm/overview.md)

Puede crear sus propias claves y almacenarlas en un almacén de claves o HSM administrado, o bien puede usar las API de Azure Key Vault para generarlas. La cuenta de almacenamiento y el almacén de claves o HSM administrado deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en distintas suscripciones.

> [!IMPORTANT]
>
> El cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault actualmente se encuentra en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.
>
> Azure Key Vault y HSM administrado de Azure Key Vault admiten las mismas API e interfaces de administración para la configuración.

## <a name="about-customer-managed-keys"></a>Acerca de las claves administradas por el cliente

En el siguiente diagrama se muestra cómo Azure Storage usa Azure Active Directory y un almacén de claves o HSM administrado para hacer solicitudes mediante la clave administrada por el cliente:

![Diagrama que muestra cómo funcionan las claves administradas por el cliente en Azure Storage](media/customer-managed-keys-overview/encryption-customer-managed-keys-diagram.png)

En la lista siguiente se explican los pasos numerados del diagrama:

1. Un administrador de Azure Key Vault concede permisos sobre las claves de cifrado a la identidad administrada que está asociada a la cuenta de almacenamiento.
2. Un administrador de Azure Storage configura el cifrado con una clave administrada por el cliente para la cuenta de almacenamiento.
3. Azure Storage usa la identidad administrada que está asociada a la cuenta de almacenamiento para autenticar el acceso a Azure Key Vault mediante Azure Active Directory.
4. Azure Storage encapsula la clave de cifrado de la cuenta con la clave administrada por el cliente en Azure Key Vault.
5. En operaciones de lectura y escritura, Azure Storage envía solicitudes a Azure Key Vault para desencapsular la clave de cifrado de la cuenta con el fin de realizar operaciones de cifrado y descifrado.

La identidad administrada asociada a la cuenta de almacenamiento debe tener estos permisos como mínimo para acceder a una clave administrada por el cliente en Azure Key Vault:

- *wrapkey*
- *unwrapkey*
- *get*  

Para obtener más información sobre los permisos de clave, consulte [Tipos de claves, algoritmos y operaciones](../../key-vault/keys/about-keys-details.md#key-access-control).

Azure Policy proporciona una directiva integrada para requerir que las cuentas de almacenamiento usen claves administradas por el cliente para cargas de trabajo de Blob Storage y Azure Files. Para obtener más información, consulte la sección **Almacenamiento** de las [definiciones de directivas integradas de Azure Policy](../../governance/policy/samples/built-in-policies.md#storage).

## <a name="customer-managed-keys-for-queues-and-tables"></a>Claves administradas por el cliente para colas y tablas

Los datos almacenados en Queue y Table Storage no se protegen automáticamente mediante una clave administrada por el cliente cuando se habilitan las claves administradas por el cliente para la cuenta de almacenamiento. Opcionalmente, puede configurar estos servicios para que se incluyan en esta protección en el momento de crear la cuenta de almacenamiento.

Para más información sobre cómo crear una cuenta de almacenamiento que admita el uso de claves administradas por el cliente para colas y tablas, consulte [Creación de una cuenta que admita las claves administradas por el cliente para tablas y colas](account-encryption-key-create.md).

Los datos en Blob Storage y Azure Files siempre están protegidos por claves administradas por el cliente cuando se han configurado las claves administradas por el cliente para la cuenta de almacenamiento.

## <a name="enable-customer-managed-keys-for-a-storage-account"></a>Habilitación de claves administradas por el cliente para una cuenta de almacenamiento

Al configurar una clave administrada por el cliente, Azure Storage encapsula la clave de cifrado de la cuenta con la clave administrada por el cliente en el almacén de claves o HSM administrado asociado. La habilitación de claves administradas por el cliente no afecta al rendimiento y surte efecto de forma inmediata.

Al habilitar o deshabilitar las claves administradas por el cliente, o al modificar la clave o la versión de la clave, la protección de la clave de cifrado raíz cambia, pero no es necesario volver a cifrar los datos de la cuenta de Azure Storage.

Las claves administradas por el cliente solo se pueden habilitar en cuentas de almacenamiento existentes. El almacén de claves o HSM administrado se deben configurar para conceder permisos a la identidad administrada que está asociada a la cuenta de almacenamiento. La identidad administrada solo está disponible después de crear la cuenta de almacenamiento.

Puede cambiar entre las claves administradas por el cliente y las claves administradas por Microsoft en cualquier momento. Para obtener más información sobre las claves administradas por Microsoft, vea [Información sobre la administración de claves de cifrado](storage-service-encryption.md#about-encryption-key-management).

Para obtener información sobre cómo configurar el cifrado de Azure Storage con las claves administradas por el cliente en un almacén de claves, consulte [Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault](customer-managed-keys-configure-key-vault.md). Para configurar las claves administradas por el cliente en un HSM administrado, consulte [Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)](customer-managed-keys-configure-key-vault-hsm.md).

> [!IMPORTANT]
> Las claves administradas por el cliente dependen de identidades administradas para los recursos de Azure, una característica de Azure AD. Las identidades administradas no admiten actualmente escenarios entre directorios. Al configurar las claves administradas por el cliente en Azure Portal, se asigna automáticamente una identidad administrada a la cuenta de almacenamiento en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o la cuenta de almacenamiento de un directorio de Azure AD a otro, la identidad administrada asociada a la cuenta de almacenamiento no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para más información, vea **Transferencia de una suscripción entre directorios de Azure AD** en [Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

El cifrado de almacenamiento de Azure admite claves RSA y RSA-HSM de los tamaños 2048, 3072 y 4096. Para obtener más información sobre las claves, consulte [Acerca de las claves](../../key-vault/keys/about-keys.md).

El uso de un almacén de claves o HSM administrado tiene costos asociados. Para más información, vea [Precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="update-the-key-version"></a>Actualización de la versión de la clave

Al configurar el cifrado con claves administradas por el cliente, tiene dos opciones para actualizar la versión de la clave:

- **Actualización automática de la versión de la clave:** para actualizar automáticamente una clave administrada por el cliente a una nueva versión disponible, omita la versión de la clave al habilitar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Si se omite la versión de la clave, Azure Storage comprueba el almacén de claves o HSM administrado diariamente para obtener una nueva versión de una clave administrada por el cliente. Azure Storage usa automáticamente la versión más reciente de la clave.
- **Actualización manual de la versión de la clave:** para usar una versión determinada de una clave para el cifrado de Azure Storage, especifique la versión de la clave al habilitar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Si especifica la versión de la clave, Azure Storage usará esa versión para el cifrado hasta que actualice manualmente la versión de la clave.

    Cuando se especifica la versión de la clave de manera explícita, debe actualizar manualmente la cuenta de almacenamiento para usar el nuevo URI de la versión de la clave cuando se crea una nueva versión. Para obtener más información sobre cómo actualizar la cuenta de almacenamiento para usar una nueva versión de la clave, consulte [Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault](customer-managed-keys-configure-key-vault.md) o [Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)](customer-managed-keys-configure-key-vault-hsm.md).

Al actualizar la versión de la clave, la protección de la clave de cifrado raíz cambia, pero los datos de la cuenta de Azure Storage no se vuelven a cifrar. No es preciso que el usuario realice ninguna otra acción.

> [!NOTE]
> Para rotar una clave, cree una nueva versión de la clave en el almacén de claves o HSM administrado, según las directivas de cumplimiento. Puede rotar la clave manualmente o crear una función para rotarla según una planificación.

## <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Puede revocar el acceso de la cuenta de almacenamiento a la clave administrada por el cliente en cualquier momento. Después de revocar el acceso a las claves administradas por el cliente, o después de que se haya deshabilitado o eliminado la clave, los clientes no pueden llamar a las operaciones que leen o escriben en un blob o en sus metadatos. Los intentos de llamar a cualquiera de las siguientes operaciones producirán un error con el código de error 403 (prohibido) para todos los usuarios:

- [List Blobs](/rest/api/storageservices/list-blobs), cuando se llama con el parámetro `include=metadata` en el URI de solicitud
- [Get Blob](/rest/api/storageservices/get-blob)
- [Get Blob Properties](/rest/api/storageservices/get-blob-properties)
- [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Snapshot Blob](/rest/api/storageservices/snapshot-blob), cuando se llama con el encabezado de solicitud `x-ms-meta-name`
- [Copy Blob](/rest/api/storageservices/copy-blob)
- [Copy Blob From URL](/rest/api/storageservices/copy-blob-from-url)
- [Set Blob Tier](/rest/api/storageservices/set-blob-tier)
- [Put Block](/rest/api/storageservices/put-block)
- [Put Block From URL](/rest/api/storageservices/put-block-from-url)
- [Append Block](/rest/api/storageservices/append-block)
- [Append Block From URL](/rest/api/storageservices/append-block-from-url)
- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Page](/rest/api/storageservices/put-page)
- [Put Page From URL](/rest/api/storageservices/put-page-from-url)
- [Incremental Copy Blob](/rest/api/storageservices/incremental-copy-blob) (Copia incremental del blob)

Para volver a llamar a estas operaciones, restaure el acceso a la clave administrada por el cliente.

Todas las operaciones de datos que no aparecen en esta sección pueden continuar después de que se hayan revocado las claves administradas por el cliente o se haya deshabilitado o eliminado una clave.

Para revocar el acceso a las claves administradas por el cliente, use [PowerShell](./customer-managed-keys-configure-key-vault.md#revoke-customer-managed-keys) o la [CLI de Azure](./customer-managed-keys-configure-key-vault.md#revoke-customer-managed-keys).

## <a name="customer-managed-keys-for-azure-managed-disks"></a>Claves administradas por el cliente para discos administrados por Azure

Las claves administradas por el cliente también están disponibles para administrar el cifrado de discos administrados por Azure. Las claves administradas por el cliente se comportan de forma diferente en los discos administrados que en los recursos de Azure Storage. Para más información, consulte [Cifrado del lado servidor de Azure Managed Disks](../../virtual-machines/disk-encryption.md) para Windows o [Cifrado del lado servidor de Azure Managed Disks](../../virtual-machines/disk-encryption.md) para Linux.

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [Configuración del cifrado con claves administradas por el cliente almacenadas en Azure Key Vault](customer-managed-keys-configure-key-vault.md)
- [Configuración del cifrado con claves administradas por el cliente almacenadas en HSM administrado de Azure Key Vault (versión preliminar)](customer-managed-keys-configure-key-vault-hsm.md)
