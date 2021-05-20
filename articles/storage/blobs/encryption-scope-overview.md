---
title: Ámbitos de cifrado para Blob Storage
description: Los ámbitos de cifrado ofrecen la posibilidad de administrar el cifrado en el nivel del contenedor o de un blob individual. Se pueden usar ámbitos de cifrado para crear límites seguros entre los datos que residen en la misma cuenta de almacenamiento, pero que pertenecen a clientes distintos.
services: storage
author: tamram
ms.service: storage
ms.date: 05/10/2021
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: a583af2811facf1cf8037fa0dd3fb9982fba60c4
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685682"
---
# <a name="encryption-scopes-for-blob-storage"></a>Ámbitos de cifrado para Blob Storage

Los ámbitos de cifrado permiten administrar el cifrado con una clave cuyo ámbito es un contenedor o un blob individual. Se pueden usar ámbitos de cifrado para crear límites seguros entre los datos que residen en la misma cuenta de almacenamiento, pero que pertenecen a clientes distintos.

Para obtener más información acerca de cómo trabajar con ámbitos de cifrado, consulte [Creación y administración de ámbitos de cifrado](encryption-scope-manage.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="how-encryption-scopes-work"></a>Funcionamiento de los ámbitos de cifrado

De forma predeterminada, una cuenta de almacenamiento está cifrada con una clave cuyo ámbito es toda la cuenta de almacenamiento. Al definir un ámbito de cifrado, se especifica una clave que puede estar en el ámbito de un contenedor o un blob individual. Cuando el ámbito de cifrado se aplica a un blob, el blob se cifra con esa clave. Cuando el ámbito de cifrado se aplica a un contenedor, actúa como el ámbito predeterminado para los blobs de ese contenedor, de modo que todos los blobs que se cargan en ese contenedor pueden cifrarse con la misma clave. El contenedor puede configurarse para aplicar el ámbito de cifrado predeterminado para todos los blobs del contenedor, o para permitir que se cargue un blob individual en el contenedor con un ámbito de cifrado distinto del predeterminado.

Las operaciones de lectura en un blob que pertenece a un ámbito de cifrado se producen de forma transparente, siempre y cuando el ámbito de cifrado no esté deshabilitado.

### <a name="key-management"></a>Administración de claves

Cuando se define un ámbito de cifrado, se especifica si el ámbito está protegido con una clave administrada por Microsoft o con una clave administrada por el cliente almacenada en Azure Key Vault. Distintos ámbitos de cifrado de una misma cuenta de almacenamiento pueden usar claves administradas por Microsoft o por el cliente. También puede cambiar el tipo de clave que se usa para proteger un ámbito de cifrado de clave administrada por el cliente a clave administrada por Microsoft, o viceversa, en cualquier momento. Para más información sobre las claves administradas por el cliente, consulte [Claves administradas por el cliente para el cifrado de Azure Storage](../common/customer-managed-keys-overview.md). Para obtener más información sobre las claves administradas por Microsoft, vea [Información sobre la administración de claves de cifrado](../common/storage-service-encryption.md#about-encryption-key-management).

Si define un ámbito de cifrado con una clave administrada por el cliente, puede optar por actualizar la versión de la clave de forma automática o manual. Si elige actualizar automáticamente la versión de la clave, Azure Storage comprobará el almacén de claves o el HSM administrado diariamente para obtener una nueva versión de la clave administrada por el cliente y actualizará automáticamente la clave a la versión más reciente. Para obtener más información sobre cómo actualizar la versión de una clave administrada por el cliente, consulte [Actualización de la versión de la clave](../common/customer-managed-keys-overview.md#update-the-key-version).

Una cuenta de almacenamiento puede tener hasta 10 000 ámbitos de cifrado que están protegidos con claves administradas por el cliente para las que la versión de la clave se actualiza automáticamente. Si la cuenta de almacenamiento ya tiene 10 000 ámbitos de cifrado que están protegidos con claves administradas por el cliente que actualizan automáticamente, la versión de la clave debe actualizarse manualmente para cualquier ámbito de cifrado adicional que esté protegido con claves administradas por el cliente.  

### <a name="infrastructure-encryption"></a>Cifrado de infraestructura

El cifrado de infraestructura Azure Storage permite el cifrado doble de datos. Cuando se habilita el cifrado de infraestructura, los datos se cifran dos veces, &mdash; una vez en el nivel de servicio y otra en el nivel de infraestructura, &mdash; con dos algoritmos de cifrado y dos claves diferentes.

El cifrado de infraestructura se admite para un ámbito de cifrado, así como en el nivel de la cuenta de almacenamiento. Si el cifrado de infraestructura está habilitado para una cuenta, cualquier ámbito de cifrado creado en esa cuenta usa automáticamente el cifrado de infraestructura. Si el cifrado de infraestructura no está habilitado en el nivel de cuenta, tiene la opción de habilitarlo para un ámbito de cifrado en el momento de crear el ámbito. La configuración de cifrado de infraestructura para un ámbito de cifrado no se puede cambiar después de crear el ámbito.

Para obtener más información sobre el cifrado de infraestructura, consulte [Creación de una cuenta de almacenamiento con el cifrado de infraestructura para realizar el cifrado doble de datos](../common/infrastructure-encryption-enable.md).

### <a name="encryption-scopes-for-containers-and-blobs"></a>Ámbitos de cifrado para contenedores y blobs

Al crear un contenedor, puede especificar un ámbito de cifrado predeterminado para los blobs que se cargan posteriormente en dicho contenedor. Cuando se especifica un ámbito de cifrado predeterminado para un contenedor, puede decidir cómo se aplica el ámbito de cifrado predeterminado:

- Puede requerir que todos los blobs cargados en el contenedor utilicen el ámbito de cifrado predeterminado. En este caso, todos los blobs del contenedor se cifran con la misma clave.
- Puede permitir que un cliente invalide el ámbito de cifrado predeterminado para el contenedor, de modo que se pueda cargar un blob con un ámbito de cifrado distinto del ámbito predeterminado. En este caso, los blobs del contenedor se pueden cifrar con claves diferentes.

En la tabla siguiente se resume el comportamiento de una operación de carga de blobs, en función de cómo se configure el ámbito de cifrado predeterminado para el contenedor:

| El ámbito de cifrado definido en el contenedor es… | La carga de un blob con el ámbito de cifrado predeterminado… | La carga de un blob con un ámbito de cifrado distinto del ámbito predeterminado… |
|--|--|--|
| Un ámbito de cifrado predeterminado con invalidaciones permitidas | Se realiza correctamente | Se realiza correctamente |
| Un ámbito de cifrado predeterminado con invalidaciones prohibidas | Se realiza correctamente | Errores |

Se debe especificar un ámbito de cifrado predeterminado para un contenedor en el momento de su creación.

Si no se especifica ningún ámbito de cifrado predeterminado para el contenedor, puede cargar un blob mediante cualquier ámbito de cifrado que haya definido para la cuenta de almacenamiento. El ámbito de cifrado debe especificarse en el momento en que se carga el blob.

## <a name="disabling-an-encryption-scope"></a>Deshabilitación de un ámbito de cifrado

Al deshabilitar un ámbito de cifrado, las operaciones de lectura o escritura posteriores realizadas con el ámbito de cifrado producirán un error con el código de error HTTP 403 (prohibido). Si vuelve a habilitar el ámbito de cifrado, las operaciones de lectura y escritura continuarán con normalidad.

Cuando se deshabilita un ámbito de cifrado, ya no se le facturará. Deshabilite los ámbitos de cifrado que no sean necesarios para evitar cargos innecesarios.

Si el ámbito de cifrado está protegido con una clave administrada por el cliente y elimina la clave en el almacén de claves, los datos dejarán de estar accesibles. Asegúrese de deshabilitar también el ámbito de cifrado para evitar que se apliquen cargos.

Tenga en cuenta que las claves administradas por el cliente están protegidas por la protección de eliminación y purga temporal en el almacén de claves, y que una clave eliminada está sujeta al comportamiento definido por esas propiedades. Para más información, consulte uno de los siguientes temas en la documentación de Azure Key Vault:

- [Uso de la eliminación temporal con PowerShell](../../key-vault/general/key-vault-recovery.md)
- [Uso de la eliminación temporal con la CLI](../../key-vault/general/key-vault-recovery.md).

> [!IMPORTANT]
> No es posible eliminar un ámbito de cifrado.

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](../common/storage-service-encryption.md)
- [Creación y administración de ámbitos de cifrado](encryption-scope-manage.md)
- [Claves administradas por el cliente para el cifrado de Azure Storage](../common/customer-managed-keys-overview.md)
- [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)