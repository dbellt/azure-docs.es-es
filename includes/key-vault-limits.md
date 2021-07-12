---
author: amitbapat
ms.service: key-vault
ms.topic: include
ms.date: 05/28/2021
ms.author: ambapat
ms.openlocfilehash: 3539e222407fee63a829e80db986f2b0eed02e24
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111761415"
---
El servicio Azure Key Vault admite dos tipos de recursos: almacenes y HSM administrados. En las dos secciones siguientes se describen los límites de servicio para cada uno de ellos, respectivamente.

### <a name="resource-type-vault"></a>Tipo de recurso: almacén

En esta sección se describen los límites de servicio para el tipo de recurso `vaults`.

#### <a name="key-transactions-maximum-transactions-allowed-in-10-seconds-per-vault-per-regionsup1sup"></a>Transacciones clave (n.º máximo de transacciones permitidas en 10 segundos, por almacén y región<sup>1</sup>):

|Tipo de clave|Clave HSM<br>Clave CREATE|Clave HSM<br>Las restantes transacciones|Clave de software<br>Clave CREATE|Clave de software<br>Las restantes transacciones|
|:---|---:|---:|---:|---:|
|2048 bits de RSA|5|1,000|10|2\.000|
|3072 bits de RSA|5|250|10|500|
|4096 bits de RSA|5|125|10|250|
|ECC P-256|5|1,000|10|2\.000|
|ECC P-384|5|1,000|10|2\.000|
|ECC P-521|5|1,000|10|2\.000|
|ECC SECP256K1|5|1,000|10|2\.000|
||||||

> [!NOTE]
> En la tabla anterior, vemos que para las claves de software de 2048 bits RSA, se permiten 2000 transacciones GET cada 10 segundos. Para las claves HSM de 2048 bits RSA, se permiten 1000 transacciones GET cada 10 segundos.
>
> Los umbrales de limitación se ponderan y el cumplimiento se basa en su suma. Por ejemplo, como se ha mostrado en la tabla anterior, al realizar operaciones GET en las claves HSM RSA, resulta ocho veces más costoso usar claves de 4096 bits en comparación con las claves de 2048 bits. Eso es porque 1000/125 = 8.
>
> En un intervalo determinado de 10 segundos, un cliente de Azure Key Vault *solo* puede realizar una de las siguientes operaciones antes de encontrar un código de estado HTTP de limitación `429`:
> - 2000 transacciones GET de clave de software de 2048 bits RSA
> - 1000 transacciones GET de clave HSM de 2048 bits RSA
> - 125 transacciones GET de clave HSM de 4096 bits RSA
> - 124 transacciones GET de clave HSM de 4096 bits y 8 transacciones GET de clave HSM de 2048 bits RSA

#### <a name="secrets-managed-storage-account-keys-and-vault-transactions"></a>Secretos, claves de cuentas de almacenamiento administradas y transacciones de almacén:

| Tipo de transacciones | N.º máximo de transacciones permitidas en 10 segundos, por almacén y región<sup>1</sup> |
| --- | --- |
| Todas las transacciones |2\.000 |

Vea la [Guía de las limitaciones de Azure Key Vault](../articles/key-vault/general/overview-throttling.md) para obtener información sobre cómo controlar la limitación cuando se superan estos límites.

<sup>1</sup> Un límite global para la suscripción para todos los tipos de transacciones es cinco veces el límite del almacén de claves. Por ejemplo, en las otras transacciones HSM por suscripción, el límite es de 5000 transacciones en 10 segundos por suscripción.

#### <a name="backup-keys-secrets-certificates"></a>Copia de seguridad de claves, secretos y certificados

Al realizar una copia de seguridad de un objeto almacenado en el almacén de claves (secreto, clave o certificado), la operación de copia de seguridad descargará el objeto como un blob cifrado. Este blob no se puede descifrar fuera de Azure. Para obtener datos que se puedan usar de este blob, debe restaurar el blob en un almacén de claves dentro de la misma suscripción de Azure y zona geográfica de Azure.

| Tipo de transacciones | Número máximo de versiones de objetos de almacén de claves que se permiten |
| --- | --- |
| Copia de seguridad de clave, secreto o certificado individual |500 |

> [!NOTE]
> Si se intenta hacer una copia de seguridad de un objeto de clave, secreto o certificado con más versiones que el límite anterior, se producirá un error. No es posible eliminar versiones anteriores de una clave, un secreto o un certificado. 

### <a name="limits-on-count-of-keys-secrets-and-certificates"></a>Límites en el recuento de claves, secretos y certificados:

Key Vault no restringe el número de claves, secretos o certificados que se pueden almacenar en un almacén. Los límites de transacción en el almacén deben tenerse en cuenta para asegurarse de que las operaciones no estén limitadas.

Key Vault no restringe el número de versiones de un secreto, una clave o un certificado, pero el almacenamiento de un gran número de versiones (más de 500) puede afectar al rendimiento de las operaciones de copia de seguridad. Consulte [Copia de seguridad de Azure Key Vault](../articles/key-vault/general/backup.md).

#### <a name="azure-private-link-integration"></a>Integración de Azure Private Link

> [!NOTE]
> El número de almacenes de claves con puntos de conexión privados habilitados por suscripción es un límite ajustable. El límite que se muestra a continuación es el límite predeterminado. Si desea solicitar un aumento del límite para el servicio, cree una solicitud de soporte técnico, que se evaluará de manera individualizada.

| Resource | Límite |
| -------- | -----:|
| Puntos de conexión privados por almacén de claves | 64 |
| Almacenes de claves con puntos de conexión privados por suscripción | 400 |

### <a name="resource-type-managed-hsm"></a>Tipo de recurso: HSM administrado

En esta sección se describen los límites de servicio para el tipo de recurso `managed HSM`.

#### <a name="object-limits"></a>Límites de objetos

|Elemento|Límites|
|----|------:|
Número de instancias de HSM por suscripción por región|1 
Número de claves por grupo de HSM|5000
Número de versiones por clave|100
Número de definiciones de roles personalizados por HSM|50
Número de asignaciones de roles en el ámbito de HSM|50
Número de asignación de roles en cada ámbito de clave individual|10
|||

#### <a name="transaction-limits-for-administrative-operations-number-of-operations-per-second-per-hsm-instance"></a>Límites de transacciones para operaciones administrativas (número de operaciones por segundo por instancia de HSM)
|Operación |Número de operaciones por segundo.|
|----|------:|
Todas las operaciones de RBAC<br/>(incluye todas las operaciones CRUD para las definiciones de roles y las asignaciones de roles)|5
Copia de seguridad/restauración completa de HSM<br/>(solo se admite una operación simultánea de copia de seguridad o restauración por instancia de HSM)|1

#### <a name="transaction-limits-for-cryptographic-operations-number-of-operations-per-second-per-hsm-instance"></a>Límites de transacciones para operaciones criptográficas (número de operaciones por segundo por instancia de HSM)

- Cada instancia de HSM administrada constituye tres particiones de HSM con equilibrio de carga. Los límites de rendimiento son una función de la capacidad de hardware subyacente que se asigna a cada partición. Las tablas siguientes muestran el rendimiento máximo con al menos una partición disponible. El rendimiento real puede ser hasta tres veces superior si están disponibles las tres particiones.
- Los límites de rendimiento indicados suponen que se utiliza una sola clave para lograr el máximo rendimiento. Por ejemplo, si se usa una única clave RSA-2048, el rendimiento máximo será de 1100 operaciones de firma. Si utilizan 1100 claves diferentes con una transacción por segundo, no podrán lograr el mismo rendimiento.

##### <a name="rsa-key-operations-number-of-operations-per-second-per-hsm-instance"></a>Operaciones de clave RSA (número de operaciones por segundo por instancia HSM)

|Operación|2048 bits|3072 bits|4096 bits|
|--|--:|--:|--:|
Crear clave|1| 1| 1
Eliminación de clave (eliminación temporal)|10|10|10 
Purgado de clave|10|10|10 
Copia de seguridad de clave|10|10|10 
Restauración de clave|10|10|10 
Obtención de información clave|1100|1100|1100
Cifrado|10000|10000|6000
Descifrado|1100|360|160
Encapsulado|10000|10000|6000
Desencapsulado|1100|360|160
Firma|1100|360|160
Comprobar|10000|10000|6000
|||||

##### <a name="ec-key-operations-number-of-operations-per-second-per-hsm-instance"></a>Operaciones de clave EC (número de operaciones por segundo por instancia de HSM)

En esta tabla se describe el número de operaciones por segundo para cada tipo de curva.

|Operación|P-256|P-256K|P-384|P-521|
|---|---:|---:|---:|---:|
Crear clave| 1| 1| 1| 1
Eliminación de clave (eliminación temporal)|10|10|10|10
Purgado de clave|10|10|10|10
Copia de seguridad de clave|10|10|10|10
Restauración de clave|10|10|10|10
Obtención de información clave|1100|1100|1100|1100
Firma|260|260|165|56
Comprobar|130|130|82|28
||||||


##### <a name="aes-key-operations-number-of-operations-per-second-per-hsm-instance"></a>Operaciones de clave AES (número de operaciones por segundo por instancia de HSM)
- Las operaciones de cifrado y descifrado suponen un tamaño de paquete de 4 KB.
- Los límites de rendimiento para cifrar y descifrar se aplican a los algoritmos AES-CBC y AES-GCM.
- Los límites de rendimiento para encapsular y desencapsular se aplican al algoritmo AES-KW.

|Operación|128 bits|192 bits|256 bits|
|--|--:|--:|--:|
Crear clave|1| 1| 1
Eliminación de clave (eliminación temporal)|10|10|10
Purgado de clave|10|10|10
Copia de seguridad de clave|10|10|10
Restauración de clave|10|10|10
Obtención de información clave|1100|1100|1100
Cifrado|8000|8000 |8000 
Descifrado|8000|8000|8000
Encapsulado|9000|9000|9000
Desencapsulado|9000|9000|9000

