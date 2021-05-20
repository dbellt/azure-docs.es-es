---
title: Notificación e instrucciones para la retirada de Azure Blockchain Service
description: Migración de Azure Blockchain Service a una oferta de cadena de bloques administrada o administrada por el cliente
ms.date: 05/10/2021
ms.topic: how-to
ms.openlocfilehash: 6fb86e426e446ba5515a285b04587093ee6fe4e5
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109752740"
---
# <a name="migrate-azure-blockchain-service"></a>Migración de Azure Blockchain Service

Puede migrar los datos del libro de contabilidad de Azure Blockchain Service a una oferta alternativa.

> [!IMPORTANT]
> El **10 de septiembre de 2021**, Azure Blockchain será retirado. Migre los datos del libro de contabilidad de Azure Blockchain Service a una oferta alternativa en función del estado de desarrollo en producción o evaluación.

## <a name="evaluate-alternatives"></a>Evaluación de alternativas

El primer paso al planear una migración es evaluar ofertas alternativas. Evalúe las siguientes alternativas en función del estado de desarrollo, si está en producción o evaluación.

### <a name="production-or-pilot-phase"></a>Fase de producción o piloto

Si ya ha implementado y desarrollado una solución de cadena de bloques que se encuentra en la fase de producción o piloto, considere las siguientes alternativas.

#### <a name="quorum-blockchain-service"></a>Quorum Blockchain Service

Quorum Blockchain Service es una oferta administrada de ConsenSys en Azure que admite Quorum como tecnología de libro de contabilidad.

- **Oferta administrada**: Quorum Blockchain Service no tiene ninguna sobrecarga de administración adicional en comparación con Azure Blockchain Service.
- **Tecnología de libro de contabilidad**: se basa en ConsenSys Quorum, que es una versión mejorada de la tecnología de libro de contabilidad GoQuorum usada en Azure Blockchain Service. No es necesario un nuevo aprendizaje. Para más información, consulte las [preguntas más frecuentes acerca de Consensys Quorum](https://consensys.net/quorum/faq).
- **Continuidad**: puede migrar los datos existentes a Quorum Blockchain Service de ConsenSys. Para más información, consulte [Exportación de datos desde Azure Blockchain Service](#export-data-from-azure-blockchain-service).

Para más información, consulte [Quorum Blockchain Service](https://consensys.net/QBS).

#### <a name="azure-vm-based-deployment"></a>Implementación basada en máquinas virtuales de Azure

Hay varias plantillas de administración de recursos de cadena de bloques que puede usar para implementar la cadena de bloques en máquinas virtuales de IaaS.

- **Tecnología de libro de contabilidad**: puede seguir usando la tecnología de libro de contabilidad de Quorum, incluido el nuevo Cuórum de ConsenSys.
- **Administrado por el cliente**: una vez implementada, el cliente administra la infraestructura y la pila de la cadena de bloques.

### <a name="new-deployment-or-evaluation-phase"></a>Nueva implementación o fase de evaluación

Si está empezando a desarrollar una nueva solución o está en fase de evaluación, tenga en cuenta las siguientes alternativas en función de los requisitos del escenario.

- [Plantilla de Quorum de Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/consensys.quorum-dev-quickstart)
- [Plantilla de Besu de Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/consensys.hyperledger-besu-quickstart)

### <a name="how-to-migrate-to-an-alternative"></a>Migración a una alternativa

Para migrar una carga de trabajo de producción, [exporte primero los datos desde Azure Blockchain Service](#export-data-from-azure-blockchain-service). Una vez que tenga una copia de los datos, puede realizar la transición de estos datos a su alternativa preferida.

El destino de migración recomendado es ConsenSys Quorum Blockchain Service. Para incorporarse a este servicio, regístrese en la página de [Quorum Blockchain Service](https://consensys.net/QBS).

Para administrar por sí mismo la solución de cadena de bloques mediante máquinas virtuales en Azure, consulte [Guía de Quorum basado en máquinas virtuales de Azure](#azure-vm-based-quorum-guidance) para configurar los nodos de transacción y de validación.
## <a name="export-data-from-azure-blockchain-service"></a>Exportación de datos desde Azure Blockchain Service

En función del estado de desarrollo actual, puede optar por usar los datos del libro de contabilidad existentes en Azure Blockchain Service o iniciar una nueva red y usar la solución que prefiera. Se recomienda crear un nuevo consorcio basado en la solución de su elección en todos los escenarios en los que no necesite o tenga previsto usar los datos del libro de contabilidad existentes en Azure Blockchain Service.

### <a name="open-support-case"></a>Apertura de un caso de soporte técnico

Si tiene un plan de soporte técnico de pago, abra una incidencia de soporte técnico de Microsoft para pausar el consorcio y exportar los datos de la cadena de bloques.

1. Use Azure Portal para abrir una incidencia de soporte técnico. En *Descripción del problema*, escriba los detalles siguientes:

    ![Formulario de descripción del problema de la incidencia de soporte técnico en Azure Portal](./media/migration-guide/problem-description.png)

    | Campo | Response |
    |-------|--------- |
    | Tipo de problema | Técnicos |
    | Servicio | Azure Blockchain Service: versión preliminar |
    | Resumen | Solicitud de datos para la migración |
    | Tipo de problema | Otros |

1. En *Detalles adicionales*, incluya los detalles siguientes:

    ![Formulario de detalles adicionales de la incidencia de soporte técnico en Azure Portal](./media/migration-guide/additional-details.png)

    - Identificador de suscripción o identificador de recurso de Azure Resource Manager
    - Inquilino
    - Nombre del consorcio
    - Region (Región)
    - Nombre del miembro
    - Fecha y hora preferidas para iniciar la migración

Si el consorcio tiene varios miembros, se requiere que cada miembro abra una incidencia de soporte técnico independiente para los datos de los miembros correspondientes.

### <a name="pause-consortium"></a>Pausa del consorcio

Debe coordinarse con los miembros del consorcio para exportar los datos, ya que el consorcio se pausará para la exportación de datos y se producirá un error en las transacciones durante este tiempo.

El equipo de Azure Blockchain Service pone en pausa el consorcio, exporta una instantánea de los datos y hace que los datos estén disponibles mediante una dirección URL de SAS de corta duración para su descarga en un formato cifrado. El consorcio se reanuda después de tomar la instantánea.

> [!IMPORTANT]
> Debe detener todas las aplicaciones que inicien nuevas transacciones de cadena de bloques en la red. Las aplicaciones activas pueden provocar la pérdida de datos o que las redes originales y migradas no se sincronicen.

### <a name="download-data"></a>Descarga de datos

Descargue los datos mediante el vínculo a la dirección URL de SAS de corta duración proporcionada por el equipo de soporte técnico de Microsoft.

> [!IMPORTANT]
> Debe descargar los datos en un plazo de siete días.

Descifre los datos mediante la clave de acceso de API. Puede [obtener la clave desde Azure Portal](configure-transaction-nodes.md#access-keys) o [mediante la API REST](/rest/api/blockchain/2019-06-01-preview/blockchainmembers/listapikeys).

> [!CAUTION]
> Solo se usa la clave de acceso 1 de la API del nodo de transacción predeterminado para cifrar todos los datos de los nodos de ese miembro.
>
> No restablezca la clave de acceso de API durante la migración.

Puede usar los datos con ConsenSys Quorum Blockchain Service o su implementación basada en máquinas virtuales de IaaS.

Para la migración de ConsenSys Quorum Blockchain Service, póngase en contacto con ConsenSys en [qbsmigration@consensys.net](mailto:qbsmigration@consensys.net).

Para usar los datos con la implementación basada en máquinas virtuales de IaaS, siga los pasos descritos en la sección [Guía de Quorum basado en máquinas virtuales de Azure](#azure-vm-based-quorum-guidance) de este artículo.

### <a name="delete-resources"></a>Eliminar recursos

Una vez que haya completado la copia de datos, se recomienda eliminar los recursos del miembro de Azure Blockchain. Se le seguirá facturando mientras existan estos recursos.

## <a name="azure-vm-based-quorum-guidance"></a>Guía de Quorum basado en máquinas virtuales de Azure

Siga estos pasos para crear nodos de transacción y nodos de validación.

### <a name="transaction-node"></a>Nodo de transacción

Un nodo de transacción tiene dos componentes. Tessera se usa para las transacciones privadas y Geth se usa para la aplicación Quorum. Los nodos de validación solo requieren el componente Geth.

#### <a name="tessera"></a>Tessera

1. Instale Java 11. Por ejemplo, `apt install default-jre`.
1. Actualice las rutas de acceso en el archivo `tessera-config.json`. Cambie todas las referencias de `/working-dir/**` a `/opt/blockchain/data/working-dir/**`.
1. Actualice la dirección IP de los otros nodos de transacción según la nueva dirección IP. HTTPS no funcionará porque no está habilitado en la configuración de Tessera. Para obtener información sobre cómo configurar TLS, consulte el artículo [Configuración de TLS de Tessera](https://docs.tessera.consensys.net/en/stable/HowTo/Configure/TLS/).
1. Actualice las reglas del NSG para permitir las conexiones entrantes al puerto 9000.
1. Ejecute Tessera con el comando siguiente:

    ```bash
    java -Xms512M -Xmx1731M -Dlogback.configurationFile=/tessera/logback-tessera.xml -jar tessera.jar -configfile /opt/blockchain/data/working-dir/tessera-config.json > tessera.log 2>&1 &
    ```

#### <a name="geth"></a>Geth

1. Actualice las direcciones IP de las direcciones enode en `/opt/blockchain/data/working-dir/dd/static-nodes.json`. Se permite una dirección IP pública.
1. Realice los mismos cambios de dirección IP en la clave StaticNodes de `/geth/config.toml`.
1. Actualice las reglas del NSG para permitir las conexiones entrantes al puerto 30303.
1. Ejecute Geth con los comandos siguientes:

    ```bash
    export NETWORK_ID='' # Get network ID from metadata. The network ID is the same for consortium.

    PRIVATE_CONFIG=tm.ipc geth --config /geth/config.toml --datadir /opt/blockchain/data/working-dir/dd --networkid $NETWORK_ID --istanbul.blockperiod 5 --nodiscover --nousb --allow-insecure-unlock --verbosity 3 --txpool.globalslots 80000 --txpool.globalqueue 80000 --txpool.accountqueue 50000 --txpool.accountslots 50000 --targetgaslimit 700000000 --miner.gaslimit 800000000 --syncmode full --rpc --rpcaddr 0.0.0.0 --rpcport 3100 --rpccorsdomain '*' --rpcapi admin,db,eth,debug,net,shh,txpool,personal,web3,quorum,istanbul --ws --wsaddr 0.0.0.0 --wsport 3000 --wsorigins '*' --wsapi admin,db,eth,debug,net,shh,txpool,personal,web3,quorum,istanbul
    ```

### <a name="validator-node"></a>Nodo de validación

Los pasos del nodo de validación son similares a los del nodo de transacción, salvo que el comando startup de Geth tendrá la marca adicional `-mine`. Tessera no se inicia en un nodo de validación. Para ejecutar Geth sin una instancia de Tessera emparejada, pase el elemento `PRIVATE_CONFIG=ignore` en el comando de Geth. Ejecute Geth con los comandos siguientes:

```bash
export NETWORK_ID=`j q '.APP_SETTINGS | fromjson | ."network-id"' env.json`

PRIVATE_CONFIG=ignore geth --config /geth/config.toml --datadir /opt/blockchain/data/working-dir/dd --networkid $NETWORK_ID --istanbul.blockperiod 5 --nodiscover --nousb --allow-insecure-unlock --verbosity 3 --txpool.globalslots 80000 --txpool.globalqueue 80000 --txpool.accountqueue 50000 --txpool.accountslots 50000 --targetgaslimit 700000000 --miner.gaslimit 800000000 --syncmode full --rpc --rpcaddr 0.0.0.0 --rpcport 3100 --rpccorsdomain '*' --rpcapi admin,db,eth,debug,net,shh,txpool,personal,web3,quorum,istanbul --ws --wsaddr 0.0.0.0 --wsport 3000 --wsorigins '*' --wsapi admin,db,eth,debug,net,shh,txpool,personal,web3,quorum,istanbul –mine
```

## <a name="upgrading-quorum"></a>Actualización de Quorum

Azure Blockchain Service puede estar ejecutando una de las siguientes versiones enumeradas de Quorum. Puede optar por usar la misma versión de Quorum o seguir los pasos siguientes para usar la versión más reciente de ConsenSys Quorum.

### <a name="upgrade-quorum-version-260-or-270-to-consensys-2110"></a>Actualización de la versión 2.6.0 o 2.7.0 de Quorum a la versión 21.1.0 de ConsenSys

La actualización desde la versión 2.6 o 2.7 de Quorum es sencilla. Descargue y actualice mediante los vínculos siguientes.
1. Descargue [ConsenSys Quorum y los archivos binarios relacionados v21.1.0](https://github.com/ConsenSys/quorum/releases/tag/v21.1.0).
1. Descargue la versión más reciente de Tessera: [tessera-app-21.1.0-app.jar](https://github.com/ConsenSys/tessera/releases/tag/tessera-21.1.0).

### <a name="upgrade-quorum-version-250-to-consensys-2110"></a>Actualización de la versión 2.5.0 de Quorum a la versión 21.1.0 de ConsenSys

1. Descargue [ConsenSys Quorum y los archivos binarios relacionados v21.1.0](https://github.com/ConsenSys/quorum/releases/tag/v21.1.0).
1. Descargue la versión más reciente de Tessera: [tessera-app-21.1.0-app.jar](https://github.com/ConsenSys/tessera/releases/tag/tessera-21.1.0).
En el caso de las versiones 2.5.0, hay algunos cambios menores en el archivo genesis. Realice los siguientes cambios en el archivo genesis.

1. El valor de `byzantiumBlock` estaba establecido en 1 y no puede ser menor que el valor de `constantinopleBlock`, que es 0. Establezca el valor de `byzantiumBlock` en 0.
1. Establezca `petersburgBlock` e `istanbulBlock` en un bloque futuro. Este valor debe ser el mismo en todos los nodos.
1. Este paso es opcional. `ceil2Nby3Block` estaba colocado incorrectamente en la versión 2.5.0 de Quorum de Azure Blockchain Service. Debe estar dentro de la configuración de istanbul y se debe establecer el valor del bloque futuro. Este valor debe ser el mismo en todos los nodos.
1. Ejecute geth para reinicializar el bloque genesis mediante el siguiente comando:

    ```bash
    geth --datadir "Data Directory Path" init "genesis file path"
    ```

1.  Ejecute Geth.

## <a name="exported-data-reference"></a>Referencia de los datos exportados

En esta sección se describen los metadatos y la estructura de carpetas para ayudar a importar los datos en la implementación de máquinas virtuales de IaaS.

### <a name="metadata-info"></a>Información de metadatos

| Nombre               | Muestra                | Descripción           |
|--------------------|-----------------------|-----------------------|
| consortium_name    | \<ConsortiumName\>    | Nombre del consorcio (único en Azure Blockchain Service). |
| Consortium_Member_Count || Número de miembros del consorcio. |
| member_name        | \<memberName\>        | Nombre del miembro de la cadena de bloques (único en Azure Blockchain Service). |
| node_name          | transaction-node      | Nombre del nodo (cada miembro tiene varios nodos). |
| network_id         | 543                   | Identificador de red de Geth.      |
| is_miner           | False                 | Is_Miner == true (nodo de validación), Is_Miner == false (nodo de transacción) |
| quorum_version     | 2.7.0                 | Versión de Quorum     |
| tessera_version    | 0.10.5                | Versión de Tessera      |
| java_version       | java-11-openjdk-amd64 | Versión de Java que usa Tessera |
| CurrentBlockNumber |                       | Número de bloque actual de la red de la cadena de bloques |

## <a name="migrated-data-folder-structure"></a>Estructura de carpetas de los datos migrados

En el nivel superior, hay carpetas que corresponden a cada uno de los nodos de los miembros.

- **SKU Estándar**: dos nodos de validación (validator-node-0 y validator-node-1)
- **SKU Básica**: un nodo de validación (validator-node-0)
- **Nodo de transacción**: nodo de transacción predeterminado, llamado transaction-node.

Otras carpetas de nodos de transacción tienen el nombre del nodo de transacción.

### <a name="node-level-folder-structure"></a>Estructura de carpetas de nivel de nodo

Cada carpeta de nivel de nodo contiene un archivo ZIP que se cifra mediante la clave de cifrado. Para más información sobre cómo obtener la clave de cifrado, consulte la sección [Descarga de datos](#download-data) de este artículo.

| Directorio o archivo | Descripción |
|----------------|--------------|
| /config/config.toml | Parámetros de Geth. Los parámetros de la línea de comandos tienen prioridad. |
| /config/genesis.json | Archivo Genesis |
| /config/logback-tessera.xml | Configuración de Logback para Tessera |
| /config/static-nodes.json | Nodos estáticos. Los nodos de arranque se quitan y la detección automática está deshabilitada. |
| /config/tessera-config.json | Configuración de Tessera |
| /data/c/ | Base de datos de Tessera |
| /data/dd/ | Directorio de datos de Geth |
| /env/env | Metadatos |
| /keys/ | Claves de Tessera |
| /scripts/ | Scripts de inicio (proporcionados solo como referencia) |

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="what-does-service-retirement-mean-for-existing-customers"></a>¿Qué significa la retirada del servicio para los clientes existentes?

Las implementaciones de Azure Blockchain Service existentes no pueden continuar más allá del 10 de septiembre de 2021. Comience a evaluar las alternativas sugeridas en este artículo antes de la retirada en función de sus requisitos.

### <a name="what-happens-to-existing-deployments-after-the-announcement-of-retirement"></a>¿Qué ocurre con las implementaciones existentes después del anuncio de retirada?

Las implementaciones existentes se admiten hasta el 10 de septiembre de 2021. Evalúe las alternativas sugeridas, migre los datos a la oferta alternativa, prepare sus requisitos en la oferta alternativa y comience la migración desde la implementación en Azure Blockchain Service.

### <a name="how-long-will-the-existing-deployments-be-supported-on-azure-blockchain-service"></a>¿Cuánto tiempo se admiten las implementaciones existentes en Azure Blockchain Service?

Las implementaciones existentes se admiten hasta el 10 de septiembre de 2021.

### <a name="will-i-be-allowed-to-create-new-azure-blockchain-members-while-in-retirement-phase"></a>¿Se me permitirá crear nuevos miembros de Azure Blockchain durante la fase de retirada?

Después del 10 de mayo de 2021, no se admiten nuevas implementaciones ni la creación de miembros.
