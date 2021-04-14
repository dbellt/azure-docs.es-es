---
author: memildin
ms.service: security-center
ms.topic: include
ms.date: 03/21/2021
ms.author: memildin
ms.custom: generated
ms.openlocfilehash: 5fc36e327a9530105182f0a23b3ef22ab324e01c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "104803267"
---
- Se debe restringir el acceso a las cuentas de almacenamiento con configuraciones de red virtual y firewall
- Las variables de cuenta de automatización deben cifrarse
- Azure Cache for Redis debe residir en una red virtual
- Las cuentas de Azure Cosmos DB deben usar claves administradas por el cliente para cifrar los datos en reposo
- Las áreas de trabajo de Azure Machine Learning deben cifrarse con una clave administrada por el cliente (CMK).
- Azure Spring Cloud debe usar la inserción de red
- Las cuentas de Cognitive Services deben habilitar el cifrado de datos con una clave administrada por el cliente (CMK)
- Debe aplicar los límites de CPU y memoria de los contenedores.
- Las imágenes de contenedor solo deben implementarse desde registros de confianza
- Las instancias de Container Registry se deben cifrar con una clave administrada por el cliente (CMK)
- Debe evitar los contenedores con elevación de privilegios.
- Deben evitarse los contenedores que comparten espacios de nombres de host confidenciales.
- Los contenedores solo deben escuchar en los puertos permitidos.
- El sistema de archivos raíz inmutable (de solo lectura) debe aplicarse para los contenedores.
- Las claves de Key Vault deben tener una fecha de expiración
- Los secretos de Key Vault deben tener una fecha de expiración
- Los almacenes de claves deben tener habilitada la protección contra operaciones de purga
- Los almacenes de claves deben tener habilitada la eliminación temporal
- Deben aplicarse funcionalidades de Linux con privilegios mínimos para los contenedores
- Solo se deben habilitar las conexiones seguras a Redis Cache
- La opción de reemplazar o deshabilitar el perfil de AppArmor de los contenedores debe estar restringida.
- Deben evitarse los contenedores con privilegios.
- Debe evitar la ejecución de contenedores como usuario raíz.
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento
- Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric
- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente
- Los servicios solo deben escuchar en los puertos permitidos.
- No se debe permitir el acceso público a la cuenta de almacenamiento
- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager
- Las cuentas de almacenamiento deben restringir el acceso a la red mediante el uso de reglas de red virtual
- El uso de puertos y redes de hosts debe estar restringido.
- El uso de montajes de volúmenes HostPath de pod debe restringirse a una lista conocida, con el fin de restringir el acceso a los nodos de los contenedores en peligro
- El período de validez de los certificados almacenados en Azure Key Vault no debe superar los 12 meses
- Se deben migrar las máquinas virtuales a nuevos recursos de Azure Resource Manager
- El firewall de aplicaciones web (WAF) debe estar habilitado para Application Gateway
- Web Application Firewall (WAF) debe estar habilitado en Azure Front Door Service

