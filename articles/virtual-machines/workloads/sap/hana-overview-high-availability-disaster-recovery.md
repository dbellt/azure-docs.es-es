---
title: Alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: En este documento se explica cómo establecer una estrategia de alta disponibilidad y recuperación ante desastres de SAP HANA en Azure (instancias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: Ajayan1008
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: df80e1a6c4d7521aaca96ca3abc32616a9f6c2bb
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736617"
---
# <a name="sap-hana-large-instances-high-availability-and-disaster-recovery-on-azure"></a>Alta disponibilidad y recuperación ante desastres de SAP HANA para instancias grandes en Azure 

>[!IMPORTANT]
>Esta documentación no sustituye a la documentación de administración de SAP HANA ni a las notas de SAP. Se espera que el lector tenga unos sólidos conocimientos de la administración y operaciones de SAP HANA, sobre todo en los temas relativos a la copia de seguridad, restauración, alta disponibilidad y recuperación ante desastres.

Es importante que practique los pasos y procesos que se realizan en su entorno y con sus versiones de HANA. Algunos procesos descritos en esta documentación se han simplificado para mejorar la comprensión general y no están diseñados para utilizarse como pasos detallados en posibles manuales de uso. Si desea crear manuales de operaciones de sus configuraciones, debe probar y ejecutar los procesos, así como documentar aquellos relacionados con sus configuraciones concretas. 


Tanto alta disponibilidad como la recuperación ante desastres (DR) son aspectos cruciales de la ejecución de servidores críticos de SAP HANA en Azure (instancias grandes). Es importante trabajar con SAP, un integrador de sistemas o Microsoft para diseñar e implementar correctamente las estrategias correctas de alta disponibilidad y recuperación ante desastres. También es importante tener en cuenta el objetivo de punto de recuperación (RPO) y de tiempo de recuperación, que son específicos del entorno.

Microsoft admite algunas funcionalidades de alta disponibilidad de SAP HANA con instancias grandes de HANA. Estas son algunas de ellas:

- **Replicación de almacenamiento**: la capacidad que tiene el propio sistema de almacenamiento para replicar todos los datos a otra marca de instancias grandes de HANA en otra región de Azure. SAP HANA funciona de forma independiente con respecto a este método. Esta funcionalidad es el mecanismo de recuperación ante desastres predeterminado que se ofrece para HANA (instancias grandes).
- **Replicación del sistema de HANA**: la [replicación de todos los datos de SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html) a un sistema SAP HANA independiente. El objetivo de tiempo de recuperación se minimiza a través de la replicación de datos a intervalos regulares. SAP HANA admite los modos asincrónico, sincrónico en memoria y sincrónico. El modo sincrónico solo se usa en sistemas de SAP HANA que estén en el mismo centro de datos, o a menos de 100 km de este. Con el diseño actual de las marcas de HANA (instancias grandes), la replicación del sistema de HANA solo puede usarse para lograr alta disponibilidad en una sola región. La replicación del sistema de HANA requiere un componente de enrutamiento o de proxy inverso de terceros para las configuraciones de recuperación ante desastres en otra región de Azure. 
- **Conmutación por error automática del host**: una solución local de recuperación ante errores para SAP HANA que es una alternativa a la replicación del sistema de HANA. Si el nodo maestro deja de estar disponible, se configura uno o varios nodos en espera de SAP HANA en el modo de escalado horizontal y SAP HANA automáticamente conmuta por error a un nodo en espera.

SAP HANA en Azure (instancias grandes) se ofrece en dos regiones de Azure de cuatro áreas geopolíticas distintas (EE. UU., Australia, Europa y Japón). Las dos regiones de un área geopolítica que hospedan HANA (instancias grandes) están conectadas a circuitos independientes de una red dedicada. Se utilizan para la replicación de instantáneas de almacenamiento, con el fin de proporcionar distintos métodos de recuperación ante desastres. La replicación no se establece de forma predeterminada, pero se configura para los clientes que solicitan de la funcionalidad de recuperación ante desastres. La replicación de almacenamiento depende del uso de instantáneas de almacenamiento para instancias grandes de HANA. No es posible elegir una región de Azure como región de recuperación ante desastre que se encuentra en otra área geopolítica. 

En la tabla siguiente se muestran los métodos y las combinaciones de alta disponibilidad y recuperación ante desastres que se admiten actualmente:

| Escenario compatible con instancias grandes de HANA | Opción de alta disponibilidad | Opción de recuperación ante desastres | Comentarios |
| --- | --- | --- | --- |
| Nodo único | No disponible. | Configuración de recuperación ante desastres dedicada.<br /> Configuración de recuperación ante desastres multipropósito. | |
| Conmutación por error automática de host: Escalado horizontal (con o sin estado de espera)<br /> incluido 1+1 | Posible con el modo en espera tomando el rol activo.<br /> HANA controla el cambio de rol. | Configuración de recuperación ante desastres dedicada.<br /> Configuración de recuperación ante desastres multipropósito.<br /> Sincronización de recuperación ante desastres mediante replicación de almacenamiento. | Los conjuntos de volúmenes de HANA se adjuntan a todos los nodos.<br /> El sitio de recuperación ante desastres debe tener el mismo número de nodos. |
| Replicación del sistema de HANA | Posible con configuración principal o secundaria.<br /> La configuración secundaria pasa al rol principal en caso de conmutación por error.<br /> Replicación del sistema de HANA y conmutación por error de control del SO. | Configuración de recuperación ante desastres dedicada.<br /> Configuración de recuperación ante desastres multipropósito.<br /> Sincronización de recuperación ante desastres mediante replicación de almacenamiento.<br /> La recuperación ante desastres mediante replicación del sistema de HANA todavía no es posible sin componentes de terceros. | El conjunto independiente de volúmenes de discos se adjunta a cada nodo.<br /> Solo los volúmenes de disco de la réplica secundaria en el sitio de producción se replican a la ubicación de recuperación ante desastres.<br /> Se requiere un conjunto de volúmenes en el sitio de recuperación ante desastres. | 

Una configuración de recuperación ante desastres dedicada es aquella en la que no se usa la unidad de instancia grande de HANA en el sitio de recuperación ante desastres para ejecutar ninguna otra carga de trabajo o ningún sistema que no sea de producción. La unidad es pasiva y solo se implementa si se ejecuta una conmutación por error ante desastres. Sin embargo, esta no es la configuración preferida para muchos clientes.

Consulte los [escenarios admitidos por HLI](hana-supported-scenario.md) para conocer el diseño del almacenamiento y los detalles de Ethernet correspondientes a su arquitectura.

> [!NOTE]
> [Implementaciones MCOD de SAP HANA](https://launchpad.support.sap.com/#/notes/1681092) (varias instancias de HANA en una unidad) como escenarios superpuestos que funcionan con los métodos de alta disponibilidad y recuperación ante desastres indicados en la tabla. Una excepción es el uso de la replicación del sistema de HANA con un clúster de conmutación por error automática basado en Pacemaker. Este caso solo admite una instancia de HANA por unidad. En el caso de las implementaciones [MDC de SAP HANA](https://launchpad.support.sap.com/#/notes/2096000), solo funcionan los métodos de recuperación ante desastres y alta disponibilidad sin almacenamiento si se implementa más de un inquilino. Con un inquilino implementado, todos los métodos mencionados son válidos.  

Una configuración de recuperación ante desastres multipropósito es aquella en la que la unidad de instancia grande de HANA en el sitio de recuperación ante desastres ejecuta una carga de trabajo no de producción. En caso de desastre, se apaga el sistema que no sea de producción, se montan los conjuntos de volúmenes replicados en el almacenamiento (adicionales) y se inicia la instancia de HANA de producción. La mayoría de los clientes que usan la funcionalidad de recuperación ante desastres de HANA (instancias grandes) utilizan esta configuración. 


Para más información sobre la alta disponibilidad de SAP HANA, consulte los artículos siguientes sobre SAP: 

- [SAP HANA High-Availability Whitepaper](https://go.sap.com/documents/2016/05/f8e5eeba-737c-0010-82c7-eda71af511fa.html) (Documento técnico sobre la alta disponibilidad de SAP HANA)
- [SAP HANA Administration Guide](https://help.sap.com/hana/SAP_HANA_Administration_Guide_en.pdf) (Guía de administración de SAP HANA)
- [Vídeo de SAP HANA acerca de la replicación del sistema de SAP HANA](https://scn.sap.com/community/hana-in-memory/blog/2015/05/19/sap-hana-system-replication)
- [Nota de soporte técnico de SAP 1999880: preguntas más frecuentes sobre la replicación del sistema de SAP HANA](https://apps.support.sap.com/sap/support/knowledge/preview/en/1999880)
- [Nota de soporte técnico de SAP 2165547: copia de seguridad y restauración de SAP HANA en el entorno de replicación del sistema de SAP HANA](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3231363535343726)
- [Nota de soporte técnico de SAP 1984882: uso de la replicación del sistema de SAP HANA para el cambio de hardware con el tiempo de inactividad mínimo o cero](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3139383438383226)

## <a name="network-considerations-for-disaster-recovery-with-hana-large-instances"></a>Consideraciones de red para recuperación ante desastres con instancias grandes de HANA

Para sacar provecho de la funcionalidad de recuperación ante desastres de HANA (instancias grandes), es preciso diseñar la conectividad de red de las dos regiones de Azure. Se necesita la conexión de un circuito de Azure ExpressRoute desde un entorno local de la región principal de Azure y la conexión de otro circuito desde un entorno local a la región de recuperación antes desastres. Esta medida cubre una situación en la que hay un problema en la región de Azure, incluida una ubicación de Microsoft Enterprise Edge Router (MSEE).

Como segunda medida, puede conectar todas las redes virtuales de Azure que se conectan a SAP HANA en Azure (instancias grandes) de una región a un circuito ExpressRoute que conecta HANA (instancias grandes) de la otra región. Con este *tipo de conexión*, los servicios que se ejecutan en una red virtual de Azure de la región 1 se pueden conectar a unidades de HANA (instancias grandes) de la región 2 y viceversa. Así se soluciona el caso en el que se queda sin conexión solo una de las ubicaciones de MSEE que conecta a su ubicación local con Azure.

En el gráfico siguiente se ilustra una configuración resistente para la recuperación ante desastres:

![Configuración óptima para la recuperación ante desastres](./media/hana-overview-high-availability-disaster-recovery/image1-optimal-configuration.png)



## <a name="other-requirements-with-hana-large-instances-storage-replication-for-disaster-recovery"></a>Otros requisitos de la replicación de almacenamiento de HANA (instancias grandes) para la recuperación ante desastres

Además de los requisitos anteriores de una configuración de recuperación ante desastres con HANA (instancias grandes) debe:

- Solicite SKU de SAP HANA en Azure (instancias grandes) que tengan el mismo tamaño que las SKU de producción e impleméntelas en la región de recuperación ante desastres. En las implementaciones de cliente actuales, estas instancias se usan para ejecutar instancias de HANA que no son de producción. Estas configuraciones se denominan *configuraciones multipropósito de recuperación ante desastres*.   
- Solicite almacenamiento adicional en el sitio de recuperación ante desastres para todas las SKU de SAP HANA en Azure (instancias grandes) que desea recuperar en el sitio de recuperación ante desastres. La compra de almacenamiento adicional le permite asignar los volúmenes de almacenamiento. Puede asignar los volúmenes que sean el destino de la replicación de almacenamiento de la región de Azure de producción a la región de Azure de recuperación ante desastres.
- En el caso de que haya configurado HSR en el principal y configurado la replicación basada en almacenamiento en el sitio de recuperación ante desastres, debe adquirir almacenamiento adicional en el sitio de recuperación ante desastres para que los datos de los nodos primarios y secundarios se repliquen en el sitio de recuperación ante desastres.

  **Pasos siguientes**
- Consulte [Copia de seguridad y restauración](hana-backup-restore.md).













