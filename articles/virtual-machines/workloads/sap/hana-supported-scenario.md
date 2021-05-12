---
title: Escenarios admitidos para SAP HANA en Azure (instancias grandes)| Microsoft Docs
description: Escenarios admitidos y sus detalles de arquitectura de SAP HANA en Azure (instancias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: Ajayan1008
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/26/2019
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 6e1868514919cdb40a0ac607b446ab944e8c36da
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738849"
---
# <a name="supported-scenarios-for-hana-large-instances"></a>Escenarios admitidos para instancias grandes de HANA
En este artículo se describen los escenarios admitidos y los detalles de arquitectura para SAP HANA (instancias grandes) (HLI).

>[!NOTE]
>Si el escenario que necesita no se menciona en este artículo, póngase en contacto con el equipo de Microsoft Service Management para que evalúe sus requisitos.
Antes de configurar la unidad de HLI, valide el diseño con SAP o con su asociado de implementación de servicio.

## <a name="terms-and-definitions"></a>Términos y definiciones
Vamos a describir los términos y las definiciones que se usan en este artículo:

- **SID**: identificador del sistema para sistemas HANA
- **HLI**: HANA (instancias grandes)
- **DR**: Recuperación ante desastres
- **Recuperación ante desastres normal**: una configuración del sistema con un recurso dedicado usado únicamente con fines de recuperación ante desastres
- **Recuperación ante desastres multipropósito**: un sistema de sitio de recuperación ante desastres que está configurado para usar un entorno que no es de producción junto con una instancia de producción configurada para un evento de DR 
- **SID único**: sistema con una instancia instalada
- **Varios SID**: sistema con varias instancias configuradas; también se conoce como "entorno MCOS"
- **HSR**: Replicación del sistema de SAP HANA

## <a name="overview"></a>Información general
HANA (instancias grandes) admite una diversidad de arquitecturas que lo ayudan a cumplir con los requisitos empresariales. En las secciones siguientes se describen los escenarios de arquitectura y sus detalles de configuración. 

El diseño de arquitectura obtenido es simplemente desde una perspectiva de infraestructura, así que debe consultar con SAP o con sus asociados de implementación todo lo relativo a la implementación de HANA. Si sus escenarios no se mencionan en este artículo, póngase en contacto con el equipo de cuentas de Microsoft para que se revise la arquitectura y se obtenga una solución para usted.

> [!NOTE]
> Estas arquitecturas son totalmente compatibles con el diseño de integración de datos personalizados (TDI) y son compatibles con SAP.

En este artículo se describen los detalles de los dos componentes de cada arquitectura admitida:

- Ethernet
- Storage

### <a name="ethernet"></a>Ethernet

Cada servidor aprovisionado viene preconfigurado con conjuntos de interfaces Ethernet. Las interfaces Ethernet configuradas en cada unidad de HLI se clasifican en cuatro tipos:

- **R.** : se usa para el acceso de cliente.
- **B**: se usa para la comunicación nodo a nodo. Esta interfaz está configurada en todos los servidores (sin importar la topología solicitada), pero solo se usa en los escenarios de escalabilidad horizontal.
- **C**: se usa para la conectividad nodo a almacenamiento.
- **D**: se usa para la conexión de dispositivos nodo a iSCSI para la configuración de STONITH. Esta interfaz solo se configura cuando se solicita una configuración de HSR.  

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Nodo a nodo|
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | STONITH |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Nodo a nodo|
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | STONITH |

La interfaz se elige en función de la topología configurada en la unidad de HLI. Por ejemplo, la interfaz "B" está configurada para la comunicación nodo a nodo, lo que resulta útil si tiene configurada una topología de escalabilidad horizontal. Esta interfaz no se utiliza para configuraciones de escalabilidad vertical de un solo nodo. Para más información sobre el uso de la interfaz, revise los escenarios necesarios (más adelante en este artículo). 

Si es necesario, puede definir tarjetas NIC adicionales por su cuenta. Sin embargo, las configuraciones de NIC existentes *no se pueden* cambiar.

>[!NOTE]
>Puede encontrar interfaces adicionales que sean interfaces físicas o uniones. Debe considerar solo las interfaces mencionadas previamente para su caso de uso. Puede omitir cualquier otra.

La distribución para las unidades con dos direcciones IP asignadas debería ser como lo siguiente:

- La Ethernet "A" debe tener asignada una dirección IP que esté dentro del intervalo de direcciones del grupo de direcciones IP de servidor que envió a Microsoft. Esta dirección IP se debe mantener en el directorio */etc/hosts* del sistema operativo.

- La Ethernet "C" debe tener asignada una dirección IP que se usa para la comunicación con NFS. Esta dirección *no* es necesario mantenerla en el directorio *etc/hosts* para permitir el tráfico de instancia a instancia dentro del inquilino.

En el caso de la implementación escalada de HANA o de la replicación del sistema de HANA, no es adecuada una configuración de hoja con dos direcciones IP asignadas. Si solo tiene dos direcciones IP asignadas y quiere implementar este tipo de configuración, póngase en contacto con SAP HANA en Azure Service Management. Pueden asignarle una tercera dirección IP en una tercera VLAN. Para las unidades de HANA (instancias grandes) con tres direcciones IP asignadas en tres puertos NIC, se aplican estas reglas de uso:

- La Ethernet "A" debe tener asignada una dirección IP que esté fuera del intervalo de direcciones del grupo de direcciones IP de servidor que envió a Microsoft. Esta dirección IP no se debe mantener en el directorio *etc/hosts* del sistema operativo.

- La Ethernet "B" se debe mantener exclusivamente en el directorio *etc/hosts* para la comunicación entre las distintas instancias. Estas son las direcciones IP que se deben mantener en las configuraciones de HANA de escalabilidad horizontal como las direcciones IP que HANA usa para la configuración entre nodos.

- La Ethernet "C" debe tener asignada una dirección IP que se usa para la comunicación con el almacenamiento NFS. Este tipo de dirección no se debe mantener en el directorio *etc/hosts*.

- La Ethernet "D" se debe usar exclusivamente para el acceso a dispositivos STONITH para Pacemaker. Esta interfaz es necesaria cuando configura la replicación del sistema HANA y quiere conseguir la conmutación por error automática del sistema operativo con un dispositivo basado en SBD.


### <a name="storage"></a>Storage
El almacenamiento está preconfigurado en función de la topología solicitada. Los tamaños de volumen y los puntos de montaje varían en función del número de servidores, el número de SKU y la topología configurada. Para más información, revise los escenarios necesarios (más adelante en este artículo). Si necesita más almacenamiento, puede comprarlo en incrementos de 1 TB.

>[!NOTE]
>El punto de montaje /usr/sap/\<SID> es un vínculo simbólico al punto de montaje /hana/shared.


## <a name="supported-scenarios"></a>Escenarios admitidos

En los diagramas de arquitectura de las secciones siguientes se usan estas anotaciones:

[ ![Tabla de los diagramas de arquitectura](media/hana-supported-scenario/Legends.png)](media/hana-supported-scenario/Legends.png#lightbox)

Estos son los escenarios admitidos:

* Nodo único con un SID
* MCOS de nodo único
* Nodo único con recuperación ante desastres (normal)
* Nodo único con recuperación ante desastres (multipropósito)
* HSR con STONITH
* HSR con recuperación ante desastres (normal/multipropósito) 
* Conmutación por error automática de host (1+1) 
* Escalabilidad horizontal con espera
* Escalabilidad horizontal sin espera
* Escalabilidad horizontal con recuperación ante desastres

## <a name="single-node-with-one-sid"></a>Nodo único con un SID

Esta topología admite un nodo en una configuración de escalabilidad horizontal con un SID.

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Nodo único con un SID](media/hana-supported-scenario/Single-node-with-one-SID.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|/hana/shared/SID | Instalación de HANA | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro | 
|/hana/logbackups/SID | Registros de rehacer |

### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.

## <a name="single-node-mcos"></a>MCOS de nodo único

Esta topología admite un nodo en una configuración de escalabilidad horizontal con varios SID.

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![MCOS de nodo único](media/hana-supported-scenario/single-node-mcos.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|/hana/shared/SID1 | Instalación de HANA para SID1 | 
|/hana/data/SID1/mnt00001 | Instalación de archivos de datos para SID1 | 
|/hana/log/SID1/mnt00001 | Instalación de archivos de registro para SID1 | 
|/hana/logbackups/SID1 | Registros de rehacer para SID1 |
|/hana/shared/SID2 | Instalación de HANA para SID2 | 
|/hana/data/SID2/mnt00001 | Instalación de archivos de datos para SID2 | 
|/hana/log/SID2/mnt00001 | Instalación de archivos de registro para SID2 | 
|/hana/logbackups/SID2 | Registros de rehacer para SID2 |

### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).

## <a name="single-node-with-dr-using-storage-replication"></a>Nodo único con recuperación ante desastres mediante replicación de almacenamiento
 
Esta topología admite un nodo en una configuración de escalabilidad horizontal con uno o varios SID con la replicación basada en almacenamiento en el sitio de recuperación ante desastres para un SID principal. En el diagrama, solo se representa un sistema de un SID único en el sitio primario, pero también se admiten los sistemas de MCOS.

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Nodo único con recuperación ante desastres mediante replicación de almacenamiento](media/hana-supported-scenario/Single-node-with-dr.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|/hana/shared/SID | Instalación de HANA para SID | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para SID | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para SID | 
|/hana/logbackups/SID | Registros de rehacer para SID |


### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En MCOS: La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como "obligatorios para la instalación de HANA") para la instalación de instancias de HANA de producción en la unidad de HLI de recuperación ante desastres. 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros y los volúmenes compartidos (marcados como "replicación de almacenamiento") se replican mediante una instantánea desde el sitio de producción. Estos volúmenes se montan solo durante la conmutación por error. Para más información, consulte [Procedimiento de conmutación por error de recuperación ante desastres](./hana-overview-high-availability-disaster-recovery.md).
- El volumen de arranque de *clase SKU tipo I* se replica en el nodo de recuperación ante desastres.


## <a name="single-node-with-dr-multipurpose-using-storage-replication"></a>Nodo único con recuperación ante desastres (multipropósito) mediante replicación de almacenamiento
 
Esta topología admite un nodo en una configuración de escalabilidad horizontal con uno o varios SID con la replicación basada en almacenamiento en el sitio de recuperación ante desastres para un SID principal. En el diagrama, solo se representa un sistema de un SID único en el sitio primario, pero también se admiten los sistemas de varios SID (MCOS). En el sitio de recuperación ante desastres, la unidad HLI se usa para la instancia de QA mientras las operaciones de producción se ejecutan desde el sitio primario. Durante la conmutación por error de recuperación ante desastres (o prueba de conmutación por error), se retira la instancia de QA en el sitio de recuperación ante desastres.

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Nodo único con recuperación ante desastres (multipropósito) mediante replicación de almacenamiento](media/hana-supported-scenario/single-node-with-dr-multipurpose.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el sitio principal**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el sitio de recuperación ante desastres**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/shared/QA-SID | Instalación de HANA para el SID de QA | 
|/hana/data/QA-SID/mnt00001 | Instalación de archivos de datos para el SID de QA | 
|/hana/log/QA-SID/mnt00001 | Instalación de archivos de registro para el SID de QA |
|/hana/logbackups/QA-SID | Registros de rehacer para el SID de QA |

### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En MCOS: La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como "obligatorios para la instalación de HANA") para la instalación de instancias de HANA de producción en la unidad de HLI de recuperación ante desastres. 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros y los volúmenes compartidos (marcados como "replicación de almacenamiento") se replican mediante una instantánea desde el sitio de producción. Estos volúmenes se montan solo durante la conmutación por error. Para más información, consulte [Procedimiento de conmutación por error de recuperación ante desastres](./hana-overview-high-availability-disaster-recovery.md). 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros, los registros y los volúmenes compartidos para QA (marcados como "instalación de instancia de QA") se configuran para la instalación de la instancia de QA.
- El volumen de arranque de *clase SKU tipo I* se replica en el nodo de recuperación ante desastres.

## <a name="hsr-with-stonith-for-high-availability"></a>HSR con STONITH para alta disponibilidad
 
Esta topología admite dos nodos para la configuración de replicación de sistema de HANA. Esta configuración solo se admite para instancias únicas de HANA en un nodo. Esto significa que los escenarios MCOS *no* son compatibles.

> [!NOTE]
> A partir de diciembre de 2019, esta arquitectura se admite solo para el sistema operativo SUSE.


### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![HSR con STONITH para alta disponibilidad](media/hana-supported-scenario/HSR-with-STONITH.png)



### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Usado para STONITH |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Usado para STONITH |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el nodo principal**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el nodo secundario**|
|/hana/shared/SID | Instalación de HANA para el SID secundario | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID secundario | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID secundario | 
|/hana/logbackups/SID | Registros de rehacer para el SID secundario |

### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En MCOS: La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).
- STONITH: se configura un SBD para la configuración de STONITH. Sin embargo, el uso de STONITH es opcional.


## <a name="high-availability-with-hsr-and-dr-with-storage-replication"></a>Alta disponibilidad con HSR y recuperación ante desastres con replicación de almacenamiento
 
Esta topología admite dos nodos para la configuración de replicación de sistema de HANA. Se admite tanto la recuperación ante desastres normal como multipropósito. Estas configuraciones solo se admiten para instancias únicas de HANA en un nodo. Esto significa que los escenarios MCOS *no* son compatibles con estas configuraciones.

En el diagrama, el escenario multipropósito se representa en el sitio de recuperación ante desastres, la unidad HLI se usa para la instancia de QA mientras las operaciones de producción se ejecutan desde el sitio primario. Durante la conmutación por error de recuperación ante desastres (o prueba de conmutación por error), se retira la instancia de QA en el sitio de recuperación ante desastres. 

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Alta disponibilidad con HSR y recuperación ante desastres con replicación de almacenamiento](media/hana-supported-scenario/HSR-with-DR.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Usado para STONITH |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Usado para STONITH |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el nodo principal en el sitio principal**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el nodo secundario en el sitio principal**|
|/hana/shared/SID | Instalación de HANA para el SID secundario | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID secundario | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID secundario | 
|/hana/logbackups/SID | Registros de rehacer para el SID secundario |
|**En el sitio de recuperación ante desastres**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/shared/QA-SID | Instalación de HANA para el SID de QA | 
|/hana/data/QA-SID/mnt00001 | Instalación de archivos de datos para el SID de QA | 
|/hana/log/QA-SID/mnt00001 | Instalación de archivos de registro para el SID de QA |
|/hana/logbackups/QA-SID | Registros de rehacer para el SID de QA |

### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En MCOS: La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).
- STONITH: se configura un SBD para la configuración de STONITH. Sin embargo, el uso de STONITH es opcional.
- En el sitio de recuperación ante desastres: *se necesitan dos conjuntos de volúmenes de almacenamiento* para la replicación de nodo principal y secundaria.
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como "obligatorios para la instalación de HANA") para la instalación de instancias de HANA de producción en la unidad de HLI de recuperación ante desastres. 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros y los volúmenes compartidos (marcados como "replicación de almacenamiento") se replican mediante una instantánea desde el sitio de producción. Estos volúmenes se montan solo durante la conmutación por error. Para más información, consulte [Procedimiento de conmutación por error de recuperación ante desastres](./hana-overview-high-availability-disaster-recovery.md). 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros, los registros y los volúmenes compartidos para QA (marcados como "instalación de instancia de QA") se configuran para la instalación de la instancia de QA.
- El volumen de arranque de *clase SKU tipo I* se replica en el nodo de recuperación ante desastres.


## <a name="host-auto-failover-11"></a>Conmutación por error automática de host (1+1)
 
Esta topología admite dos nodos en una configuración de conmutación por error automática del host. Hay un nodo con un rol maestro o de trabajo y otro como nodo en espera. *SAP admite este escenario solo para S/4 HANA*. Para más información, consulte la [nota de OSS 2408419: compatibilidad con varios nodos de SAP S/4HANA](https://launchpad.support.sap.com/#/notes/2408419).



### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Conmutación por error automática de host (1+1)](media/hana-supported-scenario/scaleup-with-standby.png)

### <a name="ethernet&quot;></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Comunicación nodo a nodo |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Comunicación nodo a nodo |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name=&quot;storage&quot;></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En los nodos principales y de espera**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |



### <a name=&quot;key-considerations&quot;></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En espera: los volúmenes y los puntos de montaje están configurados (marcados como &quot;obligatorios para la instalación de HANA") para la instalación de instancias de HANA en la unidad en espera.
 

## <a name="scale-out-with-standby"></a>Escalabilidad horizontal con espera
 
Esta topología es compatible con varios nodos en una configuración de escalabilidad horizontal. Hay un nodo con el rol maestro, uno o más nodos con el rol de trabajo y uno o varios nodos como nodos en espera. Sin embargo, solo puede haber un nodo maestro en un momento determinado.


### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Escalabilidad horizontal con espera](media/hana-supported-scenario/scaleout-nm-standby.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Comunicación nodo a nodo |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Comunicación nodo a nodo |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En los nodos principal, de trabajo y de espera**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |


## <a name="scale-out-without-standby"></a>Escalabilidad horizontal sin espera
 
Esta topología es compatible con varios nodos en una configuración de escalabilidad horizontal. Hay un nodo con el rol maestro y uno o varios nodos con el rol de trabajo. Sin embargo, solo puede haber un nodo maestro en un momento determinado.


### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Escalabilidad horizontal sin espera](media/hana-supported-scenario/scaleout-nm.png)


### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Comunicación nodo a nodo |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Comunicación nodo a nodo |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En los nodos principal y de trabajo**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |


### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.

## <a name="scale-out-with-dr-using-storage-replication"></a>Escalabilidad horizontal con recuperación ante desastres mediante replicación de almacenamiento
 
Esta topología es compatible con varios nodos en una configuración de escalabilidad horizontal con recuperación ante desastres. Se admite tanto la recuperación ante desastres normal como multipropósito. En el diagrama solo se representa la recuperación ante desastres de un propósito. Puede solicitar esta topología con o sin el nodo de espera.


### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Escalabilidad horizontal con recuperación ante desastres mediante replicación de almacenamiento](media/hana-supported-scenario/scaleout-with-dr.png)


### <a name="ethernet&quot;></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI |
| B | TIPO I | eth2.tenant | eno3.tenant | Comunicación nodo a nodo |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Comunicación nodo a nodo |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name=&quot;storage&quot;></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el nodo principal**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el nodo de recuperación ante desastres**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 


### <a name=&quot;key-considerations&quot;></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
-  En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como &quot;obligatorios para la instalación de HANA") para la instalación de instancias de HANA de producción en la unidad de HLI de recuperación ante desastres. 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros y los volúmenes compartidos (marcados como "replicación de almacenamiento") se replican mediante una instantánea desde el sitio de producción. Estos volúmenes se montan solo durante la conmutación por error. Para más información, consulte [Procedimiento de conmutación por error de recuperación ante desastres](./hana-overview-high-availability-disaster-recovery.md). 
- El volumen de arranque de *clase SKU tipo I* se replica en el nodo de recuperación ante desastres.


## <a name="single-node-with-dr-using-hsr"></a>Nodo único con recuperación ante desastres mediante HSR
 
Esta topología admite un nodo en una configuración de escalabilidad vertical con un SID con la replicación del sistema HANA en el sitio de recuperación ante desastres para un SID principal. En el diagrama, solo se representa un sistema de un SID único en el sitio primario, pero también se admiten los sistemas de varios SID (MCOS).

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Nodo único con recuperación ante desastres mediante HSR](media/hana-supported-scenario/single-node-hsr-dr-111.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI/HSR |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI/HSR |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados en las unidades HLI (principal y recuperación ante desastres):

| Punto de montaje | Caso de uso | 
| --- | --- |
|/hana/shared/SID | Instalación de HANA para SID | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para SID | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para SID | 
|/hana/logbackups/SID | Registros de rehacer para SID |


### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En MCOS: La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).
- El nodo principal se sincroniza con el nodo de recuperación ante desastres mediante la replicación del sistema de HANA. 
- [Global Reach](../../../expressroute/expressroute-global-reach.md) se usa para vincular los circuitos de ExpressRoute con el fin de crear una red privada entre las redes regionales.



## <a name="single-node-hsr-to-dr-cost-optimized"></a>HSR de nodo único a recuperación ante desastres (optimizado para costo) 
 
 Esta topología admite un nodo en una configuración de escalabilidad vertical con un SID con la replicación del sistema HANA en el sitio de recuperación ante desastres para un SID principal. En el diagrama, solo se representa un sistema de un SID único en el sitio primario, pero también se admiten los sistemas de varios SID (MCOS). En el sitio de recuperación ante desastres, una unidad HLI se usa para la instancia de QA mientras las operaciones de producción se ejecutan desde el sitio primario. Durante la conmutación por error de recuperación ante desastres (o prueba de conmutación por error), se retira la instancia de QA en el sitio de recuperación ante desastres.

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![HSR de nodo único a recuperación ante desastres (optimizado para costo)](media/hana-supported-scenario/single-node-hsr-dr-cost-optimized-121.png)

### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI/HSR |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI/HSR |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el sitio principal**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el sitio de recuperación ante desastres**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|/hana/shared/QA-SID | Instalación de HANA para el SID de QA | 
|/hana/data/QA-SID/mnt00001 | Instalación de archivos de datos para el SID de QA | 
|/hana/log/QA-SID/mnt00001 | Instalación de archivos de registro para el SID de QA |
|/hana/logbackups/QA-SID | Registros de rehacer para el SID de QA |

### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En MCOS: La distribución de tamaño de volumen se basa en el tamaño de la base de datos en memoria. Para información sobre qué tamaños de base de datos en memoria se admiten en un entorno de múltiples SID, consulte [Información general y arquitectura](./hana-overview-architecture.md).
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como "Instancia de PROD en el sitio de recuperación ante desastres") para la instalación de instancias de HANA de producción en la unidad HLI de recuperación ante desastres. 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros, los registros y los volúmenes compartidos para QA (marcados como "instalación de instancia de QA") se configuran para la instalación de la instancia de QA.
- El nodo principal se sincroniza con el nodo de recuperación ante desastres mediante la replicación del sistema de HANA. 
- [Global Reach](../../../expressroute/expressroute-global-reach.md) se usa para vincular los circuitos de ExpressRoute con el fin de crear una red privada entre las redes regionales.

## <a name="high-availability-and-disaster-recovery-with-hsr"></a>Alta disponibilidad y recuperación ante desastres con HSR 
 
 Esta topología admite dos nodos para la configuración de replicación del sistema HANA en las regiones locales de alta disponibilidad. En el caso de la recuperación ante desastres, el tercer nodo de la región de recuperación ante desastres se sincroniza desde el sitio principal mediante HSR (modo asincrónico). 

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Alta disponibilidad y recuperación ante desastres con HSR](media/hana-supported-scenario/hana-system-replication-dr-131.png)

### <a name="ethernet&quot;></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI/HSR |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI/HSR |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name=&quot;storage&quot;></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el sitio principal**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el sitio de recuperación ante desastres**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |


### <a name=&quot;key-considerations&quot;></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como &quot;Instancia de PROD de recuperación ante desastres") para la instalación de instancias de HANA de producción en la unidad HLI de recuperación ante desastres. 
- El sitio primario se sincroniza con el nodo de recuperación ante desastres mediante la replicación del sistema de HANA. 
- [Global Reach](../../../expressroute/expressroute-global-reach.md) se usa para vincular los circuitos de ExpressRoute con el fin de crear una red privada entre las redes regionales.

## <a name="high-availability-and-disaster-recovery-with-hsr-cost-optimized"></a>Alta disponibilidad y recuperación ante desastres con HSR (optimizado para costo)
 
 Esta topología admite dos nodos para la configuración de replicación del sistema HANA en las regiones locales de alta disponibilidad. En el caso de la recuperación ante desastres, el tercer nodo de la región de recuperación ante desastres se sincroniza con el sitio primario mediante HSR (modo asincrónico), mientras que otra instancia (por ejemplo, QA) ya está en ejecución desde el nodo de recuperación ante desastres. 

### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

![Alta disponibilidad y recuperación ante desastres con HSR (optimizado para costo)](media/hana-supported-scenario/hana-system-replication-dr-cost-optimized-141.png)

### <a name="ethernet&quot;></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI/HSR |
| B | TIPO I | eth2.tenant | eno3.tenant | Configurado pero no en uso |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI/HSR |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Configurado pero no en uso |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name=&quot;storage&quot;></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el sitio principal**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el sitio de recuperación ante desastres**|
|/hana/shared/SID | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|/hana/shared/QA-SID | Instalación de HANA para el SID de QA | 
|/hana/data/QA-SID/mnt00001 | Instalación de archivos de datos para el SID de QA | 
|/hana/log/QA-SID/mnt00001 | Instalación de archivos de registro para el SID de QA |
|/hana/logbackups/QA-SID | Registros de rehacer para el SID de QA |

### <a name=&quot;key-considerations&quot;></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados (marcados como &quot;Instancia de PROD de recuperación ante desastres") para la instalación de instancias de HANA de producción en la unidad HLI de recuperación ante desastres. 
- En el sitio de recuperación ante desastres: los datos, las copias de seguridad de registros, los registros y los volúmenes compartidos para QA (marcados como "instalación de instancia de QA") se configuran para la instalación de la instancia de QA.
- El sitio primario se sincroniza con el nodo de recuperación ante desastres mediante la replicación del sistema de HANA. 
- [Global Reach](../../../expressroute/expressroute-global-reach.md) se usa para vincular los circuitos de ExpressRoute con el fin de crear una red privada entre las redes regionales.

## <a name="scale-out-with-dr-using-hsr"></a>Escalabilidad horizontal con recuperación ante desastres mediante HSR
 
Esta topología es compatible con varios nodos en una configuración de escalabilidad horizontal con recuperación ante desastres. Puede solicitar esta topología con o sin el nodo de espera. El nodo del sitio primario se sincroniza con el nodo de sitio de recuperación ante desastres mediante la replicación del sistema de HANA (modo asincrónico).


### <a name="architecture-diagram"></a>Diagrama de la arquitectura  

[ ![Escalabilidad horizontal con recuperación ante desastres mediante HSR](media/hana-supported-scenario/scale-out-dr-hsr-151.png)](media/hana-supported-scenario/scale-out-dr-hsr-151.png#lightbox)


### <a name="ethernet"></a>Ethernet
Las siguientes interfaces de red están preconfiguradas:

| Interfaz lógica de NIC | Tipo de SKU | Nombre con sistema operativo SUSE | Nombre con sistema operativo RHEL | Caso de uso|
| --- | --- | --- | --- | --- |
| A | TIPO I | eth0.tenant | eno1.tenant | Cliente a HLI/HSR |
| B | TIPO I | eth2.tenant | eno3.tenant | Comunicación nodo a nodo |
| C | TIPO I | eth1.tenant | eno2.tenant | Nodo a almacenamiento |
| D | TIPO I | eth4.tenant | eno4.tenant | Configurado pero no en uso |
| A | TIPO II | vlan\<tenantNo> | team0.tenant | Cliente a HLI/HSR |
| B | TIPO II | vlan\<tenantNo+2> | team0.tenant+2 | Comunicación nodo a nodo |
| C | TIPO II | vlan\<tenantNo+1> | team0.tenant+1 | Nodo a almacenamiento |
| D | TIPO II | vlan\<tenantNo+3> | team0.tenant+3 | Configurado pero no en uso |

### <a name="storage"></a>Storage
Los puntos de montaje siguientes están preconfigurados:

| Punto de montaje | Caso de uso | 
| --- | --- |
|**En el nodo principal**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |
|**En el nodo de recuperación ante desastres**|
|/hana/shared | Instalación de HANA para el SID de producción | 
|/hana/data/SID/mnt00001 | Instalación de archivos de datos para el SID de producción | 
|/hana/log/SID/mnt00001 | Instalación de archivos de registro para el SID de producción | 
|/hana/logbackups/SID | Registros de rehacer para el SID de producción |


### <a name="key-considerations"></a>Consideraciones clave
- /usr/sap/SID es un vínculo simbólico a /hana/shared/SID.
- En el sitio de recuperación ante desastres: los volúmenes y los puntos de montaje están configurados para la instalación de instancias de HANA de producción en la unidad HLI de recuperación ante desastres. 
- El sitio primario se sincroniza con el nodo de recuperación ante desastres mediante la replicación del sistema de HANA. 
- [Global Reach](../../../expressroute/expressroute-global-reach.md) se usa para vincular los circuitos de ExpressRoute con el fin de crear una red privada entre las redes regionales.


## <a name="next-steps"></a>Pasos siguientes
- [Infraestructura y conectividad](./hana-overview-infrastructure-connectivity.md) para HANA (instancias grandes)
- [Alta disponibilidad y recuperación ante desastres](./hana-overview-high-availability-disaster-recovery.md) para HANA (instancias grandes)
