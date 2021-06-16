---
title: Principios de recuperación ante desastres y preparación de SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Principios de recuperación ante desastres y preparación de SAP HANA en Azure (instancias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: Ajayan1008
manager: gwallace
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/10/2021
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1a7127477be68d931efa2b0de69961f314b88879
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110580187"
---
# <a name="disaster-recovery-principles"></a>Principios de la recuperación ante desastres

HANA (instancias grandes) ofrece una funcionalidad de recuperación ante desastres entre las marcas de HANA (instancias grandes) de diferentes regiones de Azure. Por ejemplo, si implementa unidades HANA (instancias grandes) en la región Oeste de EE. UU. de Azure, puede usar unidades de HANA (instancias grandes) de la región Este de EE. UU. como unidades de recuperación ante desastres. Como se mencionó anteriormente, la recuperación ante desastres no se configura automáticamente porque es necesario que se pague por otra unidad de HANA (instancias grandes) en la región de la recuperación ante desastres. El programa de instalación de la recuperación ante desastres funciona tanto en configuraciones de escalado vertical como de escalado horizontal. 

En los escenarios implementados hasta ahora, los clientes usan la unidad de la región de recuperación ante desastres para ejecutar sistemas que no son de producción que usan una instancia instalada de HANA. La unidad de instancia grande de HANA debe tener la misma SKU que la que se usa con fines de producción. La siguiente imagen muestra cómo es la configuración de discos entre la unidad del servidor de la región de producción de Azure y la región de recuperación ante desastres:

![Configuración de la recuperación ante desastres desde el punto de vista de los discos](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_setup.PNG)

Como se muestra en este gráfico de información general, necesita solicitar un segundo conjunto de volúmenes de disco. Los volúmenes de disco de destino tienen el mismo tamaño que los volúmenes de producción para la instancia de producción en las unidades de recuperación ante desastres. Estos volúmenes de disco están asociados a la unidad del servidor de instancia grande de HANA en el sitio de recuperación ante desastres. Los siguientes volúmenes se replican desde la región de producción al sitio de recuperación ante desastres:

- /hana/data
- /hana/logbackups 
- /hana/shared (incluye /usr/sap)

El volumen /hana/log no se replica porque no se necesita el registro de transacciones de SAP HANA al restaurar desde esos volúmenes. 

La base de la funcionalidad de recuperación ante desastres que se ofrece es la replicación de almacenamiento que brinda la infraestructura de HANA (instancias grandes). La funcionalidad que se usa en el almacenamiento no es un flujo constante de los cambios que se replican de manera asincrónica a medida que se realizan los cambios en el volumen de almacenamiento. En su lugar, se trata de un mecanismo que se basa en instantáneas de estos volúmenes que se crean de manera habitual. La diferencia entre una instantánea ya replicada y una instantánea nueva que aún no se ha replicado se transfiere al sitio de recuperación ante desastres, a los volúmenes de disco de destino.  Estas instantáneas se almacenan en los volúmenes y, si hay una conmutación por error de recuperación ante desastres, se debe restaurar en esos volúmenes.  

La primera transmisión de los datos completos del volumen debe ser antes de que la cantidad de datos sea menor que las diferencias entre las instantáneas. Como resultado, los volúmenes que se encuentran en el sitio de recuperación ante desastres contienen cada una de las instantáneas de volumen realizadas en el sitio de producción. Eventualmente, puede usar ese sistema de recuperación ante desastres para ir a un estado anterior con el fin de recuperar los datos perdidos sin tener que revertir el sistema de producción.

Si hay una implementación MCOD con varias instancias de SAP HANA independientes en una unidad de HANA (instancias grandes), se espera que todas las instancias de SAP HANA obtengan almacenamiento replicado en el lado de recuperación ante desastres.

En casos donde se use la replicación del sistema de HANA como característica de alta disponibilidad en el sitio de producción, y se use la replicación basada en almacenamiento para el sitio de recuperación ante desastres, se replican los volúmenes de ambos nodos desde el sitio principal a la instancia de la recuperación ante desastres. Debe adquirir almacenamiento adicional (el mismo tamaño que el nodo principal) en el sitio de recuperación ante desastres para dar cabida a la replicación de principal y secundario en el sitio de recuperación ante desastres. 



>[!NOTE]
>La funcionalidad de replicación del almacenamiento de HANA (instancias grandes) crea reflejos y réplicas de las instantáneas de almacenamiento. Si no realiza instantáneas de almacenamiento como se indicó en la sección Copia de seguridad y restauración de este artículo, no puede haber ninguna replicación en el sitio de recuperación ante desastres. La ejecución de la instantánea de almacenamiento es un requisito previo para la replicación del almacenamiento en el sitio de recuperación ante desastres.



## <a name="preparation-of-the-disaster-recovery-scenario"></a>Preparación del escenario de recuperación ante desastres
En este escenario, tiene un sistema de producción que se ejecuta en HANA (instancias grandes) en la región de Azure de producción. Para los siguientes pasos, supongamos que el SID del sistema HANA es "PRD" y que tiene un sistema ajeno a la producción que se ejecuta en HANA (instancias grandes) en la región de Azure de recuperación ante desastres. Por último, supongamos que su SID es "TST". La siguiente imagen muestra esta configuración:

![Inicio de la configuración de recuperación ante desastres](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start1.PNG)

Si la instancia del servidor no se ha solicitado aún con el conjunto de volúmenes de almacenamiento adicional, SAP HANA en Azure Service Management asocia el conjunto adicional de volúmenes como un destino para la réplica de producción a la unidad de HANA (instancias grandes) en la que se ejecuta la instancia de TST HANA. Para ello, debe proporcionar el SID de la instancia de HANA de producción. Después de que SAP HANA en Azure Service Management confirme la asociación de esos volúmenes, debe montar dichos volúmenes en la unidad de instancia grande de HANA.

![Paso siguiente en la configuración de recuperación ante desastres](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start2.PNG)

El paso siguiente es instalar la segunda instancia de SAP HANA en la unidad de HANA (instancias grandes) en la región de Azure de recuperación ante desastres, donde se ejecuta la instancia de TST HANA. La instancia de SAP HANA recién instalada debe tener el mismo SID. Los usuarios creados necesitan tener el mismo UID e identificador de grupo que la instancia de producción. Lea [Copia de seguridad y restauración](hana-backup-restore.md) para obtener más información. Si la instalación se realizó correctamente, necesita:

- Ejecute el paso 2 de la preparación de la instantánea de almacenamiento que se describe en [Copia de seguridad y restauración](hana-backup-restore.md).
- Cree una clave pública para la unidad de recuperación ante desastres de HANA (instancias grandes), en caso de que aún no lo haya hecho. Vea el paso 3 de la preparación de la instantánea de almacenamiento que se describe en [Copia de seguridad y restauración](hana-backup-restore.md).
- Conserve el archivo *HANABackupCustomerDetails.txt* con la nueva instancia de HANA y pruebe si la conectividad en el almacenamiento funciona correctamente.  
- Detenga la instancia de SAP HANA que acaba de instalar en la unidad de HANA (instancias grandes) en la región de Azure de recuperación ante desastres.
- Desmontar los volúmenes PRD y ponerse en contacto con SAP HANA en Azure Service Management. Los volúmenes no pueden permanecer montados en la unidad porque no se puede acceder a ellos mientras funcionen como destino de replicación de almacenamiento.  

![Diagrama que muestra la relación de replicación entre los volúmenes PRD de la región de Azure de producción y los volúmenes PRD de la región de Azure de recuperación ante desastres.](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start3.PNG)

El equipo de operaciones establece la relación de replicación entre los volúmenes PRD de la región de Azure de producción y los volúmenes PRD de la región de Azure de recuperación ante desastres.

>[!IMPORTANT]
>El volumen /hana/log no se replica porque no es necesario para restaurar la base de datos de SAP HANA replicada a un estado coherente en el sitio de recuperación ante desastres.

A continuación, configure o ajuste la programación de copia de seguridad de instantánea de almacenamiento para acceder a su RTO y RPO en caso de desastre. Para minimizar el objetivo de punto de recuperación, establezca los siguientes intervalos de replicación en el servicio de instancias grandes de HANA:
- En el caso de los volúmenes que cubre la instantánea combinada (tipo de instantánea **hana**), establezca que se repliquen cada 15 minutos en los destinos de volúmenes de almacenamiento equivalentes del sitio de recuperación ante desastres.
- En cuanto al volumen de la copia de seguridad del registro de transacciones (tipo de instantánea **logs**), establezca que se replique cada 3 minutos en los destinos de volúmenes de almacenamiento equivalentes del sitio de recuperación ante desastres.

Para minimizar el objetivo de punto de recuperación, configure lo siguiente:
- Realice una instantánea de almacenamiento de tipo **hana** (consulte el "Paso 7: Realizar instantáneas") entre cada treinta minutos y una hora.
- Realice copias de seguridad del registro de transacciones de SAP HANA cada 5 minutos.
- Realice una instantánea de almacenamiento de tipo **logs** entre cada cinco o quince minutos. Con este período, se logra un RPO de entre 15 y 25 minutos.

Con esta configuración, la secuencia de copias de seguridad del registro de transacciones, las instantáneas de almacenamiento y la replicación del volumen de copia de seguridad del registro de transacciones de HANA y /hana/data y /hana/log, /hana/shared (incluye /usr/sap) podría ser similar a los datos de este gráfico:

 ![Relación entre una instantánea de copia de seguridad del registro de transacciones y un reflejo de instantáneas en un eje temporal](./media/hana-overview-high-availability-disaster-recovery/snapmirror.PNG)

Para conseguir un RPO incluso mejor en caso de recuperación ante desastres, puede copiar las copias de seguridad del registro de transacciones de HANA de SAP HANA en Azure (instancias grandes) a la otra región de Azure. Para alcanzar esta reducción adicional del RPO, siga estos pasos:

1. Realizar una copia de seguridad del registro de transacciones de HANA tan frecuentemente como sea posible en /hana/logbackups.
1. Utilice rsync para copiar las copias de seguridad del registro de transacciones en las máquinas virtuales de Azure hospedadas en el recurso compartido NFS. Las VM se encuentran en redes virtuales de Azure en la región de producción de Azure y en las regiones de recuperación ante desastres. Debe conectar ambas redes virtuales de Azure al circuito conectando las instancias grandes de HANA de producción a Azure. Ver los gráficos de la sección [Consideraciones de red para recuperación ante desastres con instancias grandes de HANA](#Network-considerations-for-disaster recovery-with-HANA-Large-Instances). 
1. Conserve las copias de seguridad del registro de transacciones en la región de la máquina virtual asociada al almacenamiento exportado de NFS.
1. En caso de una conmutación por error ante desastres, complemente las copias de seguridad del registro de transacciones que encuentre en el volumen /hana/logbackups con copias de seguridad del registro de transacciones creadas más recientemente en el recurso compartido NFS del sitio de recuperación ante desastres. 
1. Inicie una copia de seguridad del registro de transacciones que se restaure a la copia de seguridad más reciente que se podría guardar en la región de recuperación ante desastres.

Cuando las operaciones de HANA (instancias grandes ) confirman la configuración de la relación de replicación e inicia las copias de seguridad de instantánea de almacenamiento de ejecución, los datos empiezan a replicarse.

![Paso de la configuración de recuperación ante desastres antes de establecer la replicación](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start4.PNG)

A medida que la replicación progresa, las instantáneas de los volúmenes PRD de las regiones de Azure de recuperación ante desastres no se restauran. Solo se almacenan. Si los volúmenes se montan en ese estado, representan el estado en el que desmontó esos volúmenes después de que la instancia de SAP HANA PRD se instalara en la unidad del servidor en la región de Azure de recuperación ante desastres. También representan las copias de seguridad de almacenamiento que no se han restaurado todavía.

Si hay una conmutación por error, también puede optar por restaurar a una instantánea de almacenamiento más antigua en lugar de la más reciente.

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Procedimiento de conmutación por error de recuperación ante desastres](hana-failover-procedure.md).
