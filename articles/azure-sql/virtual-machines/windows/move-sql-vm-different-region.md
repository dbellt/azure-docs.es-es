---
title: Desplazamiento de una máquina virtual a otra región (Azure Site Recovery)
description: Obtenga información sobre cómo puede migrar una máquina virtual con SQL Server desde una región a otra dentro de Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: aa5bf144-37a3-4781-892d-e0e300913d03
ms.service: virtual-machines-sql
ms.subservice: migration
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/30/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 0cd917ced00220d1251b5aa1118f4287623b9445
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080046"
---
# <a name="move-a-sql-server-vm-to-another-region-within-azure-with-azure-site-recovery"></a>Traslado de una VM con SQL Server a otra región dentro de Azure con Azure Site Recovery
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se enseña cómo usar Azure Site Recovery para migrar una máquina virtual (VM) con SQL Server desde una región a otra dentro de Azure. 

Para trasladar una VM con SQL Server a otra región, es necesario hacer lo siguiente:
1. [Preparando](#prepare-to-move): confirme que tanto la VM con SQL Server como la región de destino están correctamente preparadas para la migración. 
1. [Configuración](#configure-azure-site-recovery-vault): para trasladar la VM con SQL Server, es necesario que sea un objeto replicado dentro del almacén de Azure Site Recovery. Debe agregar la VM con SQL Server al almacén de Azure Site Recovery. 
1. [Pruebas](#test-move-process): migrar la VM con SQL Server requiere conmutarla por error desde la región de origen a la región de destino duplicada. Para garantizar que el proceso de traslado se complete correctamente, primero debe probar que la VM con SQL Server puede conmutar por error de manera correcta a la versión de destino. Esto lo ayudará a exponer los problemas y a evitarlos cuando se realice el traslado real. 
1. [Traslado](#move-the-sql-server-vm): una vez que se haya superado la conmutación por error de prueba y que sea seguro migrar la VM con SQL Server, puede realizar el traslado de la máquina virtual a la región de destino. 
1. [Limpieza](#clean-up-source-resources): para evitar cargos de facturación, quite la VM con SQL Server del almacén y cualquier recurso innecesario que quede en el grupo de recursos. 

## <a name="verify-prerequisites"></a>Verificar los requisitos previos 

- Confirme que [se admite](../../../site-recovery/azure-to-azure-support-matrix.md#region-support) el traslado desde la región de origen a la región de destino.  
- Revise la [arquitectura del escenario y sus componentes](../../../site-recovery/azure-to-azure-architecture.md), así como los [requisitos y las limitaciones de compatibilidad](../../../site-recovery/azure-to-azure-support-matrix.md). 
- Compruebe los permisos de la cuenta. Si ha creado su cuenta de Azure gratis, ya es el administrador de la suscripción. Si no es el administrador de la suscripción, solicite al administrador que le asigne los permisos que necesita. Para habilitar la replicación para una máquina virtual y copiar los datos mediante Azure Site Recovery, debe tener: 
    - Permisos para crear una máquina virtual. El rol integrado *Colaborador de la máquina virtual* tiene estos permisos, que incluyen: 
        - Permisos para crear una máquina virtual en el grupo de recursos seleccionado. 
        - Permisos para crear una máquina virtual en la red virtual seleccionada. 
        - Permisos para escribir en la cuenta de almacenamiento seleccionada. 
      - Permisos para administrar las operaciones de Azure Site Recovery. El rol *Colaborador de Site Recovery* tiene todos los permisos necesarios para administrar las operaciones de Site Recovery en un almacén de Recovery Services.  

## <a name="prepare-to-move"></a>Preparación para la migración
Prepare la VM con SQL Server de origen y la región de destino para la migración. 

### <a name="prepare-the-source-sql-server-vm"></a>Preparación de la VM con SQL Server de origen

- Asegúrese de que todos los certificaos raíz más recientes están en la VM con SQL Server que quiere mover. Si los certificados raíz más recientes no están presentes ahí, las restricciones de seguridad no permitirán la copia de datos en la región de destino. 
- Para las máquinas virtuales de Windows, instale las actualizaciones de Windows más recientes en la máquina virtual, de modo que todos los certificados raíz de confianza estén en ella. En un entorno desconectado, siga los procesos estándar de actualización de certificados y de Windows Update de su organización. 
- En las máquinas virtuales Linux, para obtener los últimos certificados raíz de confianza y la lista de revocación de certificados en la máquina virtual, siga las instrucciones proporcionadas por su distribuidor de Linux. 
- Asegúrese de que no utiliza un proxy de autenticación para controlar la conectividad de red de las máquinas virtuales que quiere trasladar. 
- Si la máquina virtual que está intentando trasladar no tiene acceso a Internet, o utiliza un proxy de firewall para controlar el acceso de salida, compruebe los requisitos. 
- Identifique el diseño de red de origen y todos los recursos que está usando actualmente. Esto incluye, pero no se limita a los equilibradores de carga, grupos de seguridad de red (NSG), e IP públicas. 

### <a name="prepare-the-target-region"></a>Preparación de la región de destino

- Compruebe que su suscripción de Azure permite crear máquinas virtuales en la región de destino que se usa para la recuperación ante desastres. Para habilitar la cuota necesaria, póngase en contacto con el soporte técnico. 
- Asegúrese de que su suscripción tiene suficientes recursos para admitir máquinas virtuales con tamaños que se correspondan con las máquinas virtuales de origen. Si está usando Site Recovery para copiar datos en el destino, Site Recovery elije el mismo tamaño para la máquina virtual de destino o el más cercano posible. 
- Asegúrese de que crea un recurso de destino para cada componente identificado en el diseño de la red de origen. Este paso es importante para asegurarse de que las máquinas virtuales tengan en la región de destino toda la funcionalidad y características que tenían en la región de origen. 
    - Azure Site Recovery automáticamente detecta y crea una red virtual al habilitar la replicación para la máquina virtual de origen. También puede crear previamente una red y asignarla a la máquina virtual en el flujo de usuario para habilitar la replicación. Debe crear manualmente cualquier otro recurso en la región de destino.
- Para crear los recursos de red más utilizados que considere apropiados, en función de la configuración de la máquina virtual de origen, consulte la siguiente documentación: 
    - [Grupos de seguridad de red](../../../virtual-network/tutorial-filter-network-traffic.md) 
    - [Equilibrador de carga](../../../load-balancer/quickstart-load-balancer-standard-internal-portal.md)
    - [Dirección IP pública](../../../virtual-network/virtual-network-public-ip-address.md)
    - Para cualquier componente de red adicional, consulte la [documentación de red](../../../virtual-network/virtual-networks-overview.md).
- Si desea probar la configuración antes de realizar la migración final a la región de destino, cree manualmente una red sin producción en la región de destino. Se recomienda realizar este paso porque garantiza interferencias mínimas con la red de producción. 

## <a name="configure-azure-site-recovery-vault"></a>Configuración del almacén de Azure Site Recovery

En los pasos siguientes aprenderá a utilizar Azure Site Recovery para copiar datos en la región de destino. Cree el almacén de Recovery Services en cualquier región distinta de la región de origen. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
1. Elija **Crear un recurso** en la esquina superior izquierda del panel de navegación. 
1. Seleccione **IT & Management tools** (Herramientas de TI y administración) y, luego, **Backup and Site Recovery**. 
1. En la pestaña **Basics** (Aspectos básicos), en **Detalles del proyecto**, cree un grupo de recursos nuevo en la región de destino o seleccione uno existente en la región de destino. 
1. En **Detalles de la instancia**, especifique un nombre para el almacén y, luego, seleccione la **región** de destino en el menú desplegable. 
1. Haga clic en **Revisar y crear** para crear el almacén de Recovery Services. 
1. Seleccione **Todos los servicios** en la esquina superior izquierda del panel de navegación y escriba `recovery services` en el cuadro de búsqueda. 
1. (Opcionalmente) Seleccione la estrella junto a la opción **Almacenes de Recovery Services** para agregarla a la barra de navegación rápida. 
1. Seleccione **Almacenes de Recovery Services** y, luego, el almacén de Recovery Services que creó. 
1. En el panel **Información general**, seleccione **Replicar**. 

   ![Configuración de la replicación](./media/move-sql-vm-different-region/configure-replication.png)

1. Seleccione **Origen** y, luego, seleccione **Azure** como el origen. Seleccione los valores adecuados para los otros campos desplegables, como la ubicación de las máquinas virtuales de origen. Solo los grupos de recursos ubicados en la región **Ubicación de origen** estarán visibles en el campo **Grupo de recursos de origen**. 
1. Seleccione **Máquinas virtuales** y elija las máquinas virtuales que quiere migrar. Seleccione **Aceptar** para guardar su selección de VM. 
1. Seleccione **Configuración** y elija la **ubicación de destino** en el menú desplegable. Debe ser el grupo de recursos que preparó anteriormente. 
1. Una vez que haya personalizado la replicación, seleccione **Crear recursos de destino** para crear los recursos en la ubicación nueva. 
1. Cuando se complete la creación de los recursos, seleccione **Habilitar replicación** para empezar a replicar la VM con SQL Server desde la región de origen a la de destino.
1. Para comprobar el estado de la replicación, vaya al almacén de recuperación, seleccione **Elementos replicados** y vea el **estado** de la VM con SQL Server. Un estado de **Protegido** indica que la replicación se completó. 

   ![Comprobación del estado de la replicación](./media/move-sql-vm-different-region/check-replication-status.png)

## <a name="test-move-process"></a>Prueba del proceso de traslado
En los pasos siguientes se muestra cómo usar Azure Site Recovery para probar el proceso de traslado. 

1. Vaya a su **almacén de Recovery Services** en [Azure Portal](https://portal.azure.com) y seleccione **Elementos replicados**. 
1. Seleccione la VM con SQL Server que quiere trasladar, compruebe que el **estado de la replicación** sea **Correcto** y, luego, seleccione **Conmutación por error de prueba**. 

   ![Conmutación por error de prueba para la VM](./media/move-sql-vm-different-region/test-failover-of-replicated-vm.png)

1. En la página **Conmutación por error de prueba**, seleccione el punto de recuperación **Más reciente coherente con la aplicación** que se usará para la conmutación por error, porque es el único tipo de instantánea que puede garantizar la coherencia de los datos de SQL Server. 
1. Seleccione la red virtual en la **red virtual de Azure** y, luego, seleccione **Aceptar** para realizar la conmutación por error de prueba. 
   
   >[!IMPORTANT]
   > Se recomienda usar una red de VM de Azure independiente para la prueba de la conmutación por error. No use la red de producción que se configuró cuando habilitó la replicación y a la que finalmente desea mover las máquinas virtuales. 

1. Para supervisar el progreso, vaya al almacén, seleccione **Trabajos de Site Recovery** en **Supervisión** y, luego, seleccione el trabajo **Conmutación por error de prueba** que está en curso.

   ![Supervisión del progreso de la prueba de la conmutación por error](./media/move-sql-vm-different-region/monitor-failover-test-job.png)

1. Una vez que se complete la prueba, vaya a **Máquinas virtuales** en el portal y revise la máquina virtual recién creada. Asegúrese de que la VM con SQL Server está en funcionamiento, tiene el tamaño adecuado y está conectada a la red apropiada. 
1. Elimine la máquina virtual que se creó como parte de la prueba, porque la opción **Conmutación por error** aparecerá atenuada hasta que se limpien los recursos de la prueba de conmutación por error. Vuelva al almacén, seleccione **Elementos replicados**, la VM con SQL Server y, luego, seleccione **Limpiar conmutación por error de prueba**. Registre y guarde cualquier observación asociada con la prueba en la sección **Notas** y active la casilla junto a **La prueba se ha completado. Elimine las máquinas virtuales de la conmutación por error de prueba**. Seleccione **Aceptar** para borrar los recursos después de la prueba. 

   ![Eliminación de los elementos después de la prueba de la conmutación por error](./media/move-sql-vm-different-region/cleanup-test-items.png)

## <a name="move-the-sql-server-vm"></a>Traslado de la VM con SQL Server 
En los pasos siguientes se muestra cómo mover la VM con SQL Server desde la región de origen a la región de destino. 

1. Vaya al almacén de **Recovery Services**, seleccione **Elementos replicados**, seleccione la máquina virtual y, luego, **Conmutación por error**. 

   ![Iniciar conmutación por error](./media/move-sql-vm-different-region/initiate-failover.png)

1. Seleccione el punto de recuperación **más reciente coherente con la aplicación** en **Punto de recuperación**. 
1. Active la casilla junto a **Shut down the machine before beginning failover** (Apagar la máquina antes de empezar la conmutación por error). Después, Site Recovery intentará apagar la máquina virtual de origen antes de desencadenar la conmutación por error. La conmutación por error seguirá incluso si se produce un error al apagar la máquina virtual. 
1. Seleccione **Aceptar** para iniciar la conmutación por error.
1. Puede supervisar el proceso de conmutación por error desde la misma página **Trabajos de Site Recovery** que vio al supervisar la prueba de la conmutación por error en la sección anterior. 
1. Una vez que se complete el trabajo, compruebe que la VM con SQL Server aparece en la región de destino según lo esperado. 
1. Vuelva al almacén, seleccione **Elementos replicados**, seleccione la VM con SQL Server y, luego, **Confirmar** para finalizar el proceso de traslado a la región de destino. Espere hasta que el trabajo de confirmación finalice. 
1. Registre la VM con SQL Server con la extensión del Agente de IaaS de SQL permitir la capacidad de administración de **máquinas virtuales con SQL** en Azure Portal y las características asociadas con la extensión. Para más información, consulte [Registro de una VM SQL Server con la extensión del Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md). 

  > [!WARNING]
  > La coherencia de los datos de SQL Server solo la garantizan las instantáneas coherentes con la aplicación. La instantánea **procesada más recientemente** no se puede usar para la conmutación por error de SQL Server porque una instantánea de recuperación tras bloqueo no puede garantizar la coherencia de los datos de SQL Server. 

## <a name="clean-up-source-resources"></a>Limpieza de los recursos de origen
Para evitar los cargos de facturación, quite la VM con SQL Server del almacén y elimine los recursos asociados innecesarios. 

1. Vuelva al almacén de **Site Recovery**, seleccione **Elementos replicados** y la VM con SQL Server. 
1. Seleccione **Deshabilitar replicación**. Seleccione un motivo para deshabilitar la protección y, luego, seleccione **Aceptar** para deshabilitar la replicación. 

   >[!IMPORTANT]
   > Es importante realizar este paso para evitar cargos por replicación de Azure Site Recovery. 

1. Si no tiene planeado reutilizar ninguno de los recursos de la región de origen, elimine todos los recursos de red pertinentes y las cuentas de almacenamiento correspondientes. 


## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes para SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.yml)
* [Orientación de precios de SQL Server para máquinas virtuales Windows](pricing-guidance.md)
* [SQL Server on a Windows VM release notes](doc-changes-updates-release-notes.md) (Notas de la versión de SQL Server en máquinas virtuales Windows)