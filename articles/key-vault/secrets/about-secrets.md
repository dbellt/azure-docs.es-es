---
title: 'Acerca de los secretos de Azure Key Vault: Azure Key Vault'
description: Información general de los secretos de Azure Key Vault.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: overview
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: dc4368075e1cecb43887c9fab21e55b23208dbab
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107814935"
---
# <a name="about-azure-key-vault-secrets"></a>Acerca de los secretos de Azure Key Vault

[Key Vault](../general/overview.md) proporciona un almacenamiento seguro de secretos genéricos, como contraseñas y cadenas de conexión de base de datos.

Desde la perspectiva del desarrollador, las API de Key Vault aceptan y devuelven los valores de secreto como cadenas. Internamente, Key Vault almacena y administra secretos como secuencias de octetos (bytes de 8 bits), con un tamaño máximo de 25 000 bytes. El servicio Key Vault no proporciona semántica para los secretos. Solo acepta los datos, los cifra, los almacena y devuelve un identificador secreto ("id"). El identificador puede usarse para recuperar el secreto en un momento posterior.  

Si se trata de información muy confidencial, los clientes deben considerar las capas adicionales de protección de datos. Un ejemplo es el cifrado de datos mediante una clave de protección independiente antes de su almacenamiento en Key Vault.  

Key Vault también admite un campo contentType para secretos. Los clientes pueden especificar el tipo de contenido de un secreto para ayudar a interpretar los datos secretos cuando se recuperen. La longitud máxima de este campo es de 255 caracteres. No hay ningún valor predefinido. El uso sugerido es como sugerencia para interpretar los datos secretos. Por ejemplo, una implementación puede almacenar contraseñas y certificados como secretos y, a continuación, utilizar este campo para diferenciarlos. No hay ningún valor predefinido.  

## <a name="encryption"></a>Cifrado

Todos los secretos de Key Vault se almacenan cifrados. Este cifrado es transparente y no requiere ninguna acción por parte del usuario. El servicio Azure Key Vault cifra sus secretos cuando los agrega y los descifra automáticamente cuando los lee. La clave de cifrado es exclusiva de cada almacén de claves.

## <a name="secret-attributes"></a>Atributos del secreto

Además los datos secretos, se pueden especificar los siguientes atributos:  

- *exp*: tipo IntDate, opcional, el valor predeterminado es **forever** (indefinidamente). El atributo *exp* (hora de expiración) identifica la hora de expiración después de la cual NO SE DEBEN recuperar los datos secretos, excepto en las [situaciones particulares](#date-time-controlled-operations). Este campo tiene fines **informativo** únicamente, que informa a los usuarios del servicio del almacén de claves que no se puede usar un determinado secreto. Su valor debe ser un número que contenga un valor IntDate.   
- *nbf*: tipo IntDate, opcional, el valor predeterminado es **now** (ahora). El atributo *nbf* (no antes de) identifica la hora antes de la cual NO SE DEBEN recuperar los datos secretos, excepto en las [situaciones particulares](#date-time-controlled-operations). Este campo tiene fines **informativos** únicamente. Su valor debe ser un número que contenga un valor IntDate. 
- *enabled*: booleano, opcional, el valor predeterminado es **true**. Este atributo especifica si se pueden recuperar los datos secretos. El atributo enabled se usa junto con *nbf* y *exp* y cuando se produce una operación entre *nbf* y *exp*, solo se permitirá si enabled se establece en **true**. Las operaciones fuera de la franja entre *nbf* y *exp* se deniegan automáticamente, excepto en [situaciones particulares](#date-time-controlled-operations).  

Existen atributos de solo lectura adicionales que se incluyen en cualquier respuesta que incluya atributos de secreto:  

- *created*: IntDate, opcional. El atributo created indica cuándo se creó esta versión del secreto. Este valor es NULL para los secretos creados antes de agregar este atributo. Su valor debe ser un número que contenga un valor IntDate.  
- *updated*: IntDate, opcional. El atributo updated indica cuándo se modificó esta versión del secreto. Este valor es NULL para los secretos modificados por última vez antes de agregar este atributo. Su valor debe ser un número que contenga un valor IntDate.

Para obtener información sobre los atributos comunes de cada tipo de objeto del almacén de claves, consulte [Introducción a las claves, secretos y certificados de Azure Key Vault](../general/about-keys-secrets-certificates.md)

### <a name="date-time-controlled-operations"></a>Operaciones controladas de fecha y hora

Una operación **get** de un secreto funcionará para los secretos todavía no válidos y los expirados, fuera de la franja *nbf* / *exp*. La llamada a la operación **get** de un secreto, para un secreto todavía no válido, se puede usar con fines de prueba. La obtención (**get**) de un secreto caducado se puede utilizar para operaciones de recuperación.

## <a name="secret-access-control"></a>Control de acceso al secreto

El control de acceso para los secretos administrados por Key Vault se proporciona en el nivel de la instancia de Key Vault que contiene esos secretos. La directiva de control de acceso para los secretos es distinta de la directiva de control de acceso para las claves en la misma instancia de Key Vault. Los usuarios pueden crear uno o varios almacenes para almacenar los secretos y están obligados a mantener la segmentación adecuada del escenario y la administración de los secretos.   

Los siguientes permisos se pueden utilizar, en una base por entidad, en la entrada de control de acceso de secretos en un almacén y reflejan fielmente las operaciones permitidas en un objeto de secreto:  

- Permisos para las operaciones de administración de secretos
  - *get*: leer un secreto  
  - *list*: enumerar los secretos o las versiones de un secreto almacenado en un almacén de claves  
  - *set*: Crear un secreto  
  - *delete*: eliminar un secreto  
  - *recover*: recuperar un servidor eliminado
  - *backup*: copia de seguridad de un secreto de un almacén de claves
  - *restore*: restaurar una copia de seguridad de un secreto a un almacén de claves

- Permisos para las operaciones con privilegios
  - *purge*: purgar (eliminar permanentemente) un secreto eliminado

Para más información sobre cómo trabajar con secretos, consulte las [operaciones con secretos en la referencia de la API REST de Key Vault](/rest/api/keyvault). Para obtener información sobre cómo establecer permisos, vea [Almacenes: crear o actualizar](/rest/api/keyvault/vaults/createorupdate) y [Almacenes: actualizar directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy). 

Guías de procedimientos para controlar el acceso en Key Vault:
- [Asignación de una directiva de acceso de Key Vault mediante la CLI](../general/assign-access-policy-cli.md)
- [Asignación de una directiva de acceso de Key Vault mediante PowerShell](../general/assign-access-policy-powershell.md)
- [Asignación de una directiva de acceso de Key Vault mediante Azure Portal](../general/assign-access-policy-portal.md)
- [Acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure](../general/rbac-guide.md)

## <a name="secret-tags"></a>Etiquetas del secreto  
Puede especificar metadatos específicos de la aplicación adicionales en forma de etiquetas. Key Vault admite hasta 15 etiquetas, cada una de las cuales puede tener un nombre de 256 caracteres y un valor de 256 caracteres.  

>[!Note]
>El autor de llamada puede leer las etiquetas si tienen el permiso *list* o *get*.

## <a name="azure-storage-account-key-management"></a>Administración de claves de la cuenta de almacenamiento de Azure

Key Vault puede administrar las claves de una [cuenta de almacenamiento Azure](../../storage/common/storage-account-overview.md):

- Internamente, Key Vault puede enumerar (sincronizar) las claves con una cuenta de almacenamiento de Azure. 
- Key Vault vuelve a generar (rotar) las claves periódicamente.
- Los valores de clave nunca se devuelven como respuesta al autor de la llamada.
- Key Vault administra las claves de las cuentas de almacenamiento y de las cuentas de almacenamiento clásicas.

Para más información, consulte:
- [Claves de acceso de cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md)
- [Administración de claves de una cuenta de almacenamiento en Azure Key Vault](../secrets/overview-storage-keys.md))


## <a name="storage-account-access-control"></a>Control de acceso a la cuenta de almacenamiento

Los siguientes permisos pueden usarse al autorizar a una entidad de seguridad de aplicación o usuario para realizar operaciones en una cuenta de almacenamiento administrada:  

- Permisos para la cuenta de almacenamiento administrada y operaciones de definiciones de SAS
  - *get*: obtener información sobre una cuenta de almacenamiento 
  - *list*: enumerar las cuentas de almacenamiento administradas por Key Vault
  - *update*: crear una cuenta de almacenamiento
  - *delete*: Eliminar una cuenta de almacenamiento  
  - *recover*: recuperar una cuenta de almacenamiento eliminada
  - *backup*: copia de seguridad de una cuenta de almacenamiento
  - *restore*: restaurar una copia de seguridad de una cuenta de almacenamiento en un almacén de claves
  - *set*: crear o actualizar una cuenta de almacenamiento
  - *regeneratekey*: volver a generar un valor de clave específico para una cuenta de almacenamiento
  - *getsas*: obtener información sobre una definición de SAS para una cuenta de almacenamiento
  - *listsas*: enumerar las definiciones de SAS de almacenamiento para una cuenta de almacenamiento
  - *deletesas*: eliminar una definición de SAS de una cuenta de almacenamiento
  - *setsas*: crear o actualizar definiciones o atributos de SAS nuevos para una cuenta de almacenamiento

- Permisos para las operaciones con privilegios
  - *purge*: purgar (eliminar permanentemente) una cuenta de almacenamiento administrada

Para más información, vea las [operaciones para cuentas de almacenamiento en la referencia de la API REST de Key Vault](/rest/api/keyvault). Para obtener información sobre cómo establecer permisos, vea [Almacenes: crear o actualizar](/rest/api/keyvault/vaults/createorupdate) y [Almacenes: actualizar directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy).

Guías de procedimientos para controlar el acceso en Key Vault:
- [Asignación de una directiva de acceso de Key Vault mediante la CLI](../general/assign-access-policy-cli.md)
- [Asignación de una directiva de acceso de Key Vault mediante PowerShell](../general/assign-access-policy-powershell.md)
- [Asignación de una directiva de acceso de Key Vault mediante Azure Portal](../general/assign-access-policy-portal.md)
- [Acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure](../general/rbac-guide.md)


## <a name="next-steps"></a>Pasos siguientes

- [Acerca de Key Vault](../general/overview.md)
- [Información acerca de claves, secretos y certificados](../general/about-keys-secrets-certificates.md)
- [Información acerca de las claves](../keys/about-keys.md)
- [Información acerca de los certificados](../certificates/about-certificates.md)
- [Protección del acceso a un almacén de claves](../general/security-features.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)