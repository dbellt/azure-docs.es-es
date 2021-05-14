---
title: Procedimiento de conmutación por error de HANA en el sitio de recuperación ante desastres para SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Cómo realizar la conmutación por error en un sitio de recuperación ante desastres para SAP HANA en Azure (instancias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: Ajayan1008
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/22/2019
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1113e4152d444dbb1d2d1fd3e7727ebff3fe1b86
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736635"
---
# <a name="disaster-recovery-failover-procedure"></a>Procedimiento de conmutación por error de recuperación ante desastres


>[!IMPORTANT]
>Este artículo no sustituye a la documentación de administración de SAP HANA ni a las notas de SAP. Se espera que el lector tenga un conocimiento sólido de la administración y operaciones de SAP HANA, sobre todo en lo relativo a la copia de seguridad, restauración, alta disponibilidad y recuperación ante desastres (DR). En este artículo se muestran capturas de pantalla de SAP HANA Studio. El contenido, la estructura y la naturaleza de las pantallas de las herramientas de administración de SAP y de las herramientas en sí pueden cambiar de una versión a otra de SAP HANA.

Hay dos casos que se deben tener en cuenta al conmutar por error al sitio de DR:

- Necesita que la base de datos de SAP HANA vuelva al estado más reciente de los datos. En este caso, hay un script de autoservicio con el que se puede realizar la conmutación por error sin necesidad de ponerse en contacto con Microsoft. Para la conmutación por recuperación, sí que debe colaborar con Microsoft.
- Quiere realizar la restauración en una instantánea de almacenamiento que no sea la última instantánea replicada. En ese caso, tendrá que colaborar con Microsoft. 

>[!NOTE]
>Los pasos a continuación se deben seguir en la unidad de HANA (instancia grande), que representa la unidad de recuperación ante desastres. 
 
Para restaurar a las instantáneas de almacenamiento replicado más recientes, siga los pasos descritos en "Perform full DR failover - azure_hana_dr_failover" (Conmutación por error completa de recuperación ante desastres: azure_hana_dr_failover) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.3/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.3.pdf) (Herramientas de instantáneas de Microsoft para SAP HANA en Azure). 

Si quiere conmutar por error varias instancias de SAP HANA, ejecute el comando azure_hana_dr_failover varias veces. Cuando se le solicite, escriba el SID de SAP HANA que desea conmutar por error y restaurar. 


También puede probar la conmutación por error de recuperación ante desastres sin que afecte a la relación de replicación real. Para llevar a cabo una conmutación por error de prueba, siga los pasos descritos en "Perform a test DR failover - azure_hana_test_dr_failover" (Realizar una prueba de conmutación por error de recuperación ante desastres: azure_hana_test_dr_failover) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.3/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.3.pdf) (herramientas de instantáneas de Microsoft para SAP HANA en Azure). 

>[!IMPORTANT]
>*No* ejecute transacciones de producción en la instancia que ha creado en el sitio de recuperación ante desastres a través del proceso de **prueba de una conmutación por error**. El comando azure_hana_test_dr_failover crea un conjunto de volúmenes que no tienen relación con el sitio primario. Como consecuencia, *no* se puede realizar una sincronización al sitio principal. 

Si quiere probar varias instancias de SAP HANA, ejecute el script varias veces. Cuando se le solicite, escriba en el SID de SAP HANA de la instancia cuya conmutación por error quiere probar. 

>[!NOTE]
>Este procedimiento es válido si tiene que conmutar por error al sitio de recuperación ante desastres para recuperar algunos datos que se eliminaron hace horas y necesita que los volúmenes de recuperación ante desastres se configuren a una instantánea anterior. 

1. Apague la instancia ajena a producción de HANA en la unidad de recuperación ante desastres de HANA (instancias grandes) que está ejecutando. Una instancia de producción de HANA inactiva está preinstalada.
1. Asegúrese de que no hay ningún proceso de SAP HANA en ejecución. Use el siguiente comando para esta comprobación:

      `/usr/sap/hostctrl/exe/sapcontrol –nr <HANA instance number> - function GetProcessList`.

      El resultado debería mostrar el proceso **hdbdaemon** en estado detenido y ningún otro proceso de HANA en ejecución o iniciado.
1. Determine en qué nombre de instantánea o identificador de copia de seguridad de SAP HANA desea que se restaure el tenga el sitio de recuperación ante desastres. En casos reales de recuperación ante desastres, esta instantánea suele ser la instantánea más reciente. Si necesita recuperar datos perdidos, elija una instantánea anterior.
1. Póngase en contacto con el soporte técnico de Azure a través de una solicitud de soporte técnico de alta prioridad. Solicite la restauración de dicha instantánea (con su nombre y fecha) o el identificador de copia de seguridad de HANA en el sitio de recuperación ante desastres. El valor predeterminado es que las operaciones solo restauran el volumen /hana/data. Si también desea tener los volúmenes /hana/logbackups, debe indicarlo específicamente. *No restaure el volumen /hana/shared.* En su lugar, elija archivos específicos, como global.ini del directorio **.snapshot** y sus subdirectorios después de volver a montar el volumen /hana/shared para PRD. 

   En lo que respecta a las operaciones, se realizan los pasos siguientes:

   a. Se detiene la replicación de instantáneas del volumen de producción en los volúmenes de recuperación ante desastres. Esto podría haber ocurrido ya si una interrupción en el sitio de producción es el motivo que necesita para realizar una recuperación ante desastres.
   
   b. El nombre de la instantánea de almacenamiento o la instantánea con el identificador de copia de seguridad que eligió se restaura en los volúmenes de recuperación ante desastres.
   
   c. Después de la restauración, los volúmenes de recuperación ante desastres están disponibles para montarlos en las unidades de HANA (instancias grandes) en la región de recuperación ante desastres.
      
1. Monte los volúmenes de recuperación ante desastres en la unidad de HANA (instancias grandes) en el sitio de recuperación ante desastres. 
1. Inicie la instancia de producción de SAP HANA inactiva.
1. Si eligió copiar registros de la copia de seguridad del registro de transacciones para reducir el tiempo de RPO, combine las copias de seguridad del registro de transacciones en el directorio /hana/logbackups de recuperación ante desastres que se acaba de montar. No sobrescriba copias de seguridad existentes. Copie las copias de seguridad más recientes que no se han replicado con la última replicación de una instantánea de almacenamiento.
1. También puede restaurar archivos individuales de las instantáneas que no se han replicado en el volumen /hana/shared/PRD en la región de Azure de recuperación ante desastres.

Los pasos siguientes muestran cómo recuperar la instancia de producción de SAP HANA según la instantánea de almacenamiento restaurada y las copias de seguridad del registro de transacciones que están disponibles.

1. Cambie la ubicación de copia de seguridad a **/hana/logbackups** con SAP HANA Studio.

   ![Cambiar la ubicación de copia de seguridad para la recuperación de DR](./media/hana-overview-high-availability-disaster-recovery/change_backup_location_dr1.png)

1. SAP HANA examina las ubicaciones del archivo de copia de seguridad y sugiere la copia de seguridad del registro de transacciones más reciente para restaurar. El examen puede tardar unos minutos hasta que aparezca una pantalla similar a la siguiente:

   ![Lista de copias de seguridad del registro de transacciones para DR](./media/hana-overview-high-availability-disaster-recovery/backup_list_dr2.PNG)

1. Ajuste algunas de las configuraciones predeterminadas:

      - Desactive la casilla **Use Delta Backups** (Usar copias de seguridad diferenciales).
      - Seleccione **Initialize Log Area** (Inicializar el área de registro).

   ![Establecer la inicialización del área de registro](./media/hana-overview-high-availability-disaster-recovery/initialize_log_dr3.PNG)

1. Seleccione **Finalizar**.

   ![Finalizar la restauración de DR](./media/hana-overview-high-availability-disaster-recovery/finish_dr4.PNG)

Debería aparecer una ventana de progreso, como la que se muestra aquí. Recuerde que el ejemplo trata de una restauración de recuperación ante desastres de una configuración de SAP HANA de escalado horizontal de tres nodos.

![Progreso de la restauración](./media/hana-overview-high-availability-disaster-recovery/restore_progress_dr5.PNG)

Si la restauración deja de responder en la pantalla **Finish** (Finalizar) y no se muestra la pantalla de progreso, confirme que todas las instancias de SAP HANA de los nodos de trabajo se están ejecutando. Si es necesario, inicie manualmente las instancias de SAP HANA.


## <a name="failback-from-a-dr-to-a-production-site"></a>Conmutación por recuperación desde un sitio de recuperación ante desastres a uno de producción
Puede conmutar por recuperación desde DR a un sitio de producción. Echemos un vistazo a un escenario en el que la conmutación por error en el sitio de recuperación ante desastres se debió a problemas en la región de Azure de producción, no a que el usuario necesitara recuperar datos perdidos. 

La carga de trabajo de producción de SAP lleva un tiempo ejecutándose en el sitio de recuperación ante desastres. Una vez que se resuelven los problemas en el sitio de producción, es posible que el usuario quiera realizar la conmutación por recuperación con el sitio de producción. Como no se pueden perder datos, el paso de regreso al sitio de producción implica varios pasos y una cooperación estrecha con SAP HANA en el equipo de operaciones de Azure. Es tarea del usuario desencadenar el equipo de operaciones para iniciar la sincronización de vuelta al sitio de producción una vez que se hayan resuelto los problemas.

Siga estos pasos:

1. SAP HANA en el equipo de operaciones de Azure obtiene el desencadenador para sincronizar los volúmenes de almacenamiento de producción desde los volúmenes de almacenamiento de recuperación ante desastres, que ahora representan el estado de la producción. En este estado, la unidad de instancia grande de HANA en el sitio de producción está apagada.
1. SAP HANA en el equipo de operaciones de Azure supervisa la replicación y garantiza que se actualice antes de informar al usuario.
1. El usuario apaga las aplicaciones que usan la instancia de HANA de producción en el sitio de recuperación ante desastres. Después, realiza una copia de seguridad del registro de transacciones de HANA. Luego, detiene la instancia de HANA que se ejecuta en las unidades de HANA (instancias grandes) en el sitio de recuperación ante desastres.
1. Una vez que se apaga la instancia de HANA que se ejecuta en la unidad de HANA (instancias grandes) del sitio de recuperación ante desastres, el equipo de operaciones vuelve a sincronizar manualmente los volúmenes de disco.
1. SAP HANA en el equipo de operaciones de Azure inicia de nuevo la unidad de instancia grande de HANA en el sitio de producción. Se la entregan a usted. Usted se asegura de que la instancia de SAP HANA esté apagada en el momento en que se inicia la unidad de instancia grande de HANA.
1. Sigue los mismos pasos de la restauración de la base de datos que siguió cuando se realizó la conmutación por error al sitio de recuperación ante desastres anteriormente.

## <a name="monitor-disaster-recovery-replication"></a>Supervisión de la replicación de recuperación ante desastres

Para supervisar el estado del progreso de la replicación de almacenamiento, ejecute el script `azure_hana_replication_status`. Para que funcione como cabría esperar, este comando se debe ejecutar desde una unidad que ejecute en la ubicación de recuperación ante desastres. El comando funciona independientemente de si la replicación está activa. El comando se puede ejecutar para cada unidad de instancia grande de HANA del inquilino en la ubicación de recuperación ante desastres. No se puede usar para obtener detalles sobre el volumen de arranque. 

Para más información sobre el comando y su salida, consulte "Get DR replication status - azure_hana_replication_status" (Obtener el estado de replicación de DR: azure_hana_replication_status) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/snapshot_tools_v4.3/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20v4.3.pdf) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).


## <a name="next-steps"></a>Pasos siguientes
- Consulte [Supervisión y solución de problemas en el lado HANA](hana-monitor-troubleshoot.md).
