---
title: Cifrado de Azure Synapse Analytics
description: En este artículo se explica el cifrado en Azure Synapse Analytics.
author: nanditavalsan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: security
ms.date: 11/19/2020
ms.author: nanditav
ms.reviewer: jrasnick
ms.openlocfilehash: 71249534c6a088088213659b5a45e042229721c7
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813189"
---
# <a name="encryption-for-azure-synapse-analytics-workspaces"></a>Cifrado en las áreas de trabajo de Azure Synapse Analytics

En este artículo se describe:
* Cifrado de datos en reposo en las áreas de trabajo de Synapse Analytics.
* Configuración de las áreas de trabajo de Synapse para habilitar el cifrado con una clave administrada por el cliente.
* Administración de claves utilizadas para cifrar los datos de las áreas de trabajo.

## <a name="encryption-of-data-at-rest"></a>Cifrado de datos en reposo

Una solución de cifrado en reposo completa garantiza que los datos no se conserven nunca en un formato no cifrado. El cifrado doble de datos en reposo reduce las amenazas gracias a las dos capas de cifrado independientes que protegen frente a los peligros de una única capa. Azure Synapse Analytics ofrece una segunda capa de cifrado para los datos del área de trabajo con una clave administrada por el cliente. Esta clave está protegida en [Azure Key Vault](../../key-vault/general/overview.md), lo que permite tomar posesión de la administración y la rotación de las claves.

La primera capa de cifrado de los servicios de Azure está habilitada con claves administradas por la plataforma. De forma predeterminada, los discos de Azure y los datos de las cuentas de Azure Storage se cifran automáticamente en reposo. Aprenda más sobre cómo se usa el cifrado en Microsoft Azure en [Introducción al cifrado de Azure](../../security/fundamentals/encryption-overview.md).

## <a name="azure-synapse-encryption"></a>Cifrado de Azure Synapse

Esta sección le ayudará a comprender mejor cómo se habilita el cifrado de claves administradas por el cliente y cómo se aplica en áreas de trabajo de Synapse. Este cifrado usa claves existentes o claves nuevas generadas en Azure Key Vault. Para cifrar todos los datos de un área de trabajo, se usa una sola clave. Las áreas de trabajo de Synapse admiten claves RSA con un tamaño de 2048 y 3072 bytes.

> [!NOTE]
> No admiten el cifrado con claves de criptografía de curva elíptica (ECC).

Los datos de los siguientes componentes de Synapse se cifran con la clave administrada por el cliente configurada en el nivel de área de trabajo:
* Grupos de SQL
 * Grupos de SQL dedicados
 * Grupos de SQL sin servidor
* Grupos de Apache Spark
* Entornos de ejecución de integración, canalizaciones y conjuntos de datos de Azure Data Factory.

## <a name="workspace-encryption-configuration"></a>Configuración del cifrado de áreas de trabajo

Las áreas de trabajo se pueden configurar para habilitar el cifrado doble con una clave administrada por el cliente en el momento de su creación. Seleccione la opción "Enable double encryption using a customer-managed key" (Habilitar el cifrado doble mediante una clave administrada por el cliente) al crear el área de trabajo. Puede elegir especificar el URI de un identificador de clave o seleccionarlo en una lista de almacenes de claves en la **misma región** que el área de trabajo. La propia instancia de Key Vault debe tener **habilitada la protección de purga**.

> [!IMPORTANT]
> Después de crear el área de trabajo, no se puede cambiar el valor de configuración del cifrado doble.

:::image type="content" source="./media/workspaces-encryption/workspaces-encryption.png" alt-text="En este diagrama se muestra la opción que se debe seleccionar para permitir el cifrado doble en un área de trabajo con una clave administrada por el cliente.":::

### <a name="key-access-and-workspace-activation"></a>Acceso de clave y activación del área de trabajo

El modelo de cifrado de Azure Synapse con claves administradas por el cliente supone el acceso del área de trabajo a las claves de Azure Key Vault de cifrado y descifrado, según sea necesario. El área de trabajo puede acceder a las claves mediante una directiva de acceso o el [acceso RBAC de Azure Key Vault](../../key-vault/general/rbac-guide.md). Al conceder permisos a través de una directiva de acceso de Azure Key Vault, elija la opción ["Application-only"](../../key-vault/general/security-features.md#key-vault-authentication-options) (Solo aplicación) durante la creación de la directiva (seleccione la identidad administrada del área de trabajo y no la agregue como aplicación autorizada).

 Para poder activar el área de trabajo, a su identidad administrada se le deben conceder los permisos necesarios en el almacén de claves. Este enfoque por fases para la activación del área de trabajo garantiza que los datos del área de trabajo se cifren con la clave administrada por el cliente. Tenga en cuenta que el cifrado puede habilitarse o deshabilitarse para los grupos de SQL dedicados; de forma predeterminada, cada grupo no está habilitado para el cifrado.

#### <a name="permissions"></a>Permisos

Para cifrar o descifrar los datos en reposo, la identidad administrada del área de trabajo debe tener los permisos siguientes:
* WrapKey (para insertar una clave en Key Vault al crear una clave).
* UnwrapKey (para obtener la clave de descifrado).
* Get (para leer la parte pública de una clave)

#### <a name="workspace-activation"></a>Activación del área de trabajo

Después de crear el área de trabajo (con el cifrado doble habilitado), permanecerá en estado "pendiente" hasta que la activación se realice correctamente. El área de trabajo debe estar activada para poder usar completamente toda la funcionalidad. Por ejemplo, solo puede crear un grupo de SQL dedicado una vez que se logra la activación. Conceda a la identidad administrada del área de trabajo acceso al almacén de claves y haga clic en el vínculo de activación en el mensaje emergente de Azure Portal del área de trabajo. Una vez que la activación finaliza correctamente, el área de trabajo está lista para usarse con la garantía de que todos los datos que contiene están protegidos con la clave administrada por el cliente. Como se indicó anteriormente, el almacén de claves debe tener habilitada la protección de purga para que la activación se lleve a cabo correctamente.

:::image type="content" source="./media/workspaces-encryption/workspace-activation.png" alt-text="En este diagrama se muestra el mensaje emergente con el vínculo de activación del área de trabajo.":::


### <a name="manage-the-workspace-customer-managed-key"></a>Administración de la clave administrada por el cliente del área de trabajo 

Puede cambiar la clave administrada por el cliente que se usa para cifrar los datos en la página **Cifrado** de Azure Portal. Aquí también puede elegir una nueva clave con un identificador de clave o seleccionar entre los almacenes de claves a los que tiene acceso en la misma región que el área de trabajo. Si elige una clave de un almacén de claves diferente de los usados anteriormente, conceda a la identidad administrada del área de trabajo los permisos "Obtener", "Encapsular" y "Desencapsular" en el nuevo almacén de claves. El área de trabajo validará su acceso al nuevo almacén de claves y todos los datos del área de trabajo se volverán a cifrar con la nueva clave.

:::image type="content" source="./media/workspaces-encryption/workspace-encryption-management.png" alt-text="En este diagrama se muestra la sección Cifrado del área de trabajo en Azure Portal.":::

>[!IMPORTANT]
>Al cambiar la clave de cifrado de un área de trabajo, conserve la clave hasta que la reemplace en el área de trabajo por una nueva. Esto se hace para permitir el descifrado de los datos con la clave antigua antes de cifrarlos de nuevo con la nueva.

Las directivas de Azure Key Vault para la rotación periódica y automática de claves o las acciones sobre las claves pueden dar lugar a la creación de otras versiones de claves. Puede optar por volver a cifrar todos los datos del área de trabajo con la versión más reciente de la clave activa. Para realizar un nuevo cifrado, cambie la clave en Azure Portal por una clave temporal y, luego, vuelva a cambiar a la clave que quiera usar para el cifrado. Por ejemplo, para actualizar el cifrado de datos con la versión más reciente de la clave activa, Key1, cambie la clave administrada por el cliente del área de trabajo por la clave temporal, Key2. Espere a que finalice el cifrado con Key2. Luego, cambie la clave administrada por el cliente del área de trabajo por Key1; los datos del área de trabajo se volverán a cifrar con la versión más reciente de Key1.

> [!NOTE]
> Azure Synapse Analytics trabaja activamente en la incorporación de una funcionalidad para volver a cifrar los datos automáticamente cuando se creen otras versiones de clave. Hasta que esta funcionalidad esté disponible, fuerce el cifrado repetido de los datos mediante el proceso detallado anteriormente a fin de garantizar la coherencia en el área de trabajo.

#### <a name="sql-transparent-data-encryption-with-service-managed-keys"></a>Cifrado de datos transparente de SQL con claves administradas por el servicio

El Cifrado de datos transparente (TDE) de SQL está disponible para los grupos de SQL dedicados de áreas de trabajo *no* habilitadas para el cifrado doble. En este tipo de área de trabajo, se usa una clave administrada por el servicio para proporcionar cifrado doble para los datos de los grupos de SQL dedicados. TDE con la clave administrada por el servicio puede habilitarse o deshabilitarse para los grupos de SQL dedicados de forma individual.

## <a name="next-steps"></a>Pasos siguientes

[Uso de directivas de Azure integradas para implementar la protección de cifrado en áreas de trabajo de Synapse](../policy-reference.md)

