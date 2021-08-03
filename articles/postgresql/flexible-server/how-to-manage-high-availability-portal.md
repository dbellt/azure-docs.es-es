---
title: Administración de la alta disponibilidad con redundancia de zona en un servidor flexible de Azure Database for PostgreSQL mediante Azure Portal
description: En este artículo se describe cómo habilitar o deshabilitar la alta disponibilidad con redundancia de zona en un servidor flexible de Azure Database for PostgreSQL mediante Azure Portal.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 06/07/2021
ms.openlocfilehash: 5a7b8c2f76abc9dd41894280e7cf610d0cae85ea
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111554896"
---
# <a name="manage-zone-redundant-high-availability-in-flexible-server"></a>Administración de la alta disponibilidad con redundancia de zona en un servidor flexible

> [!IMPORTANT]
> La opción de implementación Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

En este artículo se describe cómo habilitar o deshabilitar la configuración de la alta disponibilidad con redundancia de zona en un servidor flexible.

La característica de alta disponibilidad proporciona una réplica principal y una réplica en espera separadas físicamente en distintas zonas. Para más información, consulte la [documentación sobre los conceptos de alta disponibilidad](./concepts-high-availability.md). Puede optar por habilitar la alta disponibilidad en el momento de la creación del servidor flexible o después. En esta página se proporcionan instrucciones sobre cómo habilitar o deshabilitar la alta disponibilidad. Esta operación no cambia las demás opciones, como la configuración de red virtual, la configuración de firewall y la retención de copias de seguridad. Del mismo modo, la habilitación y deshabilitación de la alta disponibilidad es una operación en línea y no afecta a la conectividad ni a las operaciones de las aplicaciones.

## <a name="pre-requisites"></a>Requisitos previos

La alta disponibilidad con redundancia de zona solo está disponible en regiones donde se admiten varias zonas. 

## <a name="enable-high-availability-during-server-creation"></a>Habilitación de la alta disponibilidad durante la creación del servidor

En esta sección se proporcionan detalles específicos de los campos relacionados con la alta disponibilidad. Puede seguir estos pasos para implementar la alta disponibilidad al crear el servidor flexible.

1.  En [Azure Portal](https://portal.azure.com/), seleccione Servidor flexible y haga clic en Crear.  Para más información sobre cómo rellenar los datos de **Suscripción**, **Grupo de recursos**, **Nombre del servidor**, **Región**, y otros campos, consulte la documentación de procedimientos de la creación del servidor.
   
    :::image type="content" source="./media/how-to-manage-high-availability-portal/subscription-region.png" alt-text="Visualización de la suscripción y la región":::

2.  Elija su **zona de disponibilidad**. Resulta útil si quiere colocar la aplicación en la misma zona de disponibilidad que la base de datos a fin de reducir la latencia. Elija **No Preference** (Sin preferencias) si quiere que el servidor flexible se implemente en cualquier zona de disponibilidad.
    ![Selección de AZ]() :::image type="content" source="./media/how-to-manage-high-availability-portal/zone-selection.png" alt-text="Selección de la zona de disponibilidad":::  

3.  Haga clic en la casilla **Zone redundant high availability** (Alta disponibilidad con redundancia de zona) en la opción de disponibilidad.

    :::image type="content" source="./media/how-to-manage-high-availability-portal/high-availability-checkbox.png" alt-text="Casilla Alta disponibilidad":::

4.  Si quiere cambiar el proceso y el almacenamiento predeterminados, haga clic en **Configure server** (Configurar servidor).
 
    :::image type="content" source="./media/how-to-manage-high-availability-portal/configure-server.png" alt-text="configure server (configurar servidor): compute+storage (proceso y almacenamiento)":::  

5.  Si la opción de alta disponibilidad está activada, el nivel por ráfagas no estará disponible para elegirlo. Puede elegir los niveles de proceso **De uso general** u **Optimizado para memoria**. Después, puede seleccionar **tamaño de proceso** para elegirlo en la lista desplegable.

    :::image type="content" source="./media/how-to-manage-high-availability-portal/select-compute.png" alt-text="Selección del nivel de proceso":::  


6.  Seleccione el **tamaño de almacenamiento** en GiB con la barra deslizante y seleccione el **período de retención de copia de seguridad** entre 7 y 35 días.
   
    :::image type="content" source="./media/how-to-manage-high-availability-portal/storage-backup.png" alt-text="Copia de seguridad de almacenamiento"::: 

7. Haga clic en **Guardar**. 

## <a name="enable-high-availability-post-server-creation"></a>Habilitación de la alta disponibilidad después de la creación del servidor

Siga estos pasos para habilitar la alta disponibilidad para el servidor flexible existente.

1.  En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible de PostgreSQL existente.

2.  En la página del servidor flexible, haga clic en **Alta disponibilidad** en el panel izquierdo para abrir la página de alta disponibilidad.
   
     :::image type="content" source="./media/how-to-manage-high-availability-portal/high-availability-left-panel.png" alt-text="Selección del panel izquierdo"::: 

3.  Haga clic en la casilla **Zone redundant high availability** (Alta disponibilidad con redundancia de zona) para **habilitar** la opción y luego en **Guardar** para guardar el cambio.

     :::image type="content" source="./media/how-to-manage-high-availability-portal/enable-high-availability.png" alt-text="Habilitación de la alta disponibilidad"::: 

4.  Aparece un cuadro de diálogo de confirmación que indica que, al habilitar la alta disponibilidad, el costo aumentará debido a la implementación adicional del servidor y del almacenamiento.

5.  Haga clic en el botón **Enable HA** (Habilitar la alta disponibilidad) para la habilitar la alta disponibilidad.

6.  Se mostrará una notificación que indica que la operación está en curso.

## <a name="disable-high-availability"></a>Deshabilitación de la alta disponibilidad

Siga estos pasos para deshabilitar la alta disponibilidad en el servidor flexible que ya está configurado con redundancia de zona.

1.  En [Azure Portal](https://portal.azure.com/), seleccione su servidor flexible de Azure Database for PostgreSQL.

2.  En la página del servidor flexible, haga clic en **Alta disponibilidad** en el panel frontal para abrir la página de alta disponibilidad.
   
    :::image type="content" source="./media/how-to-manage-high-availability-portal/high-availability-left-panel.png" alt-text="Selección del panel izquierdo"::: 

3.  Haga clic en la casilla **Zone redundant high availability** (Alta disponibilidad con redundancia de zona) para **deshabilitar** la opción. A continuación, haga clic en **Guardar** para guardar el cambio.

     :::image type="content" source="./media/how-to-manage-high-availability-portal/disable-high-availability.png" alt-text="Deshabilitación de la alta disponibilidad"::: 

4.  Aparece un cuadro de diálogo de confirmación en el que puede confirmar la deshabilitación de la alta disponibilidad.

5.  Haga clic en el botón **Disable HA** (Deshabilitar la alta disponibilidad) para deshabilitar esta opción.

6.  Aparece una notificación que muestra que la retirada de la implementación de la alta disponibilidad está en curso.

## <a name="forced-failover"></a>conmutación por error forzada

Siga estos pasos para forzar la conmutación por error desde el servidor flexible principal al servidor flexible en espera. Esto desactivará de inmediato la base de datos principal y desencadenará una conmutación por error al servidor en espera. Esto es útil para casos en que quiere probar el tiempo de conmutación por error de una interrupción no planeada para la carga de trabajo.

1.  En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible existente que ya tiene habilitada la característica de alta disponibilidad.
2.  En la página del servidor flexible, haga clic en Alta disponibilidad en el panel frontal para abrir la página de alta disponibilidad.
3.  Compruebe la zona de disponibilidad principal y la zona de disponibilidad en espera
4.  Haga clic en Conmutación por error forzada para iniciar el procedimiento de conmutación por error manual. Una ventana emergente le informará sobre el tiempo de inactividad potencial hasta que se complete la conmutación por error. Lea el mensaje y haga clic en Aceptar.
5.  Aparecerá una notificación donde se menciona que la conmutación por error está en curso.
6.  Una vez que se complete la conmutación por error al servidor en espera, se abrirá una ventana emergente.
7.  Compruebe la nueva zona de disponibilidad principal y la zona de disponibilidad en espera.
    
    :::image type="content" source="./media/how-to-manage-high-availability-portal/ha-forced-failover.png" alt-text="Conmutación por error forzada a petición"::: 

>[!IMPORTANT] 
> * No realice conmutaciones por error inmediatas consecutivas. Espere al menos entre 15 y 20 minutos entre una conmutación por error y otra, lo que también permitirá que el nuevo servidor en espera esté totalmente establecido.
>
> * El tiempo total de la operación de un extremo a otro, tal como se notifica en el portal, puede ser mayor que el tiempo de inactividad real experimentado por la aplicación. Mida el tiempo de inactividad desde la perspectiva de la aplicación. 

## <a name="planned-failover"></a>Conmutación por error planeada

Siga estos pasos para realizar una conmutación por error planeada desde el servidor flexible principal al servidor flexible en espera. Con esto se preparará primero el servidor en espera y se realizará la conmutación por error. Esto proporciona el menor tiempo de inactividad, ya que se realiza una conmutación por error correcta en el servidor en espera para situaciones en las que, por ejemplo, después de un evento de conmutación por error, quiere devolver el servidor principal a la zona de disponibilidad preferida.
1.  En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible existente que ya tiene habilitada la característica de alta disponibilidad.
2.  En la página del servidor flexible, haga clic en Alta disponibilidad en el panel frontal para abrir la página de alta disponibilidad.
3.  Compruebe la zona de disponibilidad principal y la zona de disponibilidad en espera
4.  Haga clic en Conmutación por error planeada para iniciar el procedimiento de conmutación por error manual. Una ventana emergente le informará del proceso. Lea el mensaje y haga clic en Aceptar.
5.  Aparecerá una notificación donde se menciona que la conmutación por error está en curso.
6.  Una vez que se complete la conmutación por error al servidor en espera, se abrirá una ventana emergente.
7.  Compruebe la nueva zona de disponibilidad principal y la zona de disponibilidad en espera.
        :::image type="content" source="./media/how-to-manage-high-availability-portal/ha-planned-failover.png" alt-text="Conmutación por error planeada a petición"::: 

>[!IMPORTANT] 
>
> * No realice conmutaciones por error inmediatas consecutivas. Espere al menos entre 15 y 20 minutos entre una conmutación por error y otra, lo que también permitirá que el nuevo servidor en espera esté totalmente establecido.
>
> * Se recomienda realizar la conmutación por error planeada durante un período de baja actividad.
>
> * El tiempo total de la operación de un extremo a otro puede ser mayor que el tiempo de inactividad real experimentado por la aplicación. Mida el tiempo de inactividad desde la perspectiva de la aplicación.


## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre la [continuidad empresarial](./concepts-business-continuity.md)
-   Obtenga información sobre la [alta disponibilidad con redundancia de zona](./concepts-high-availability.md)
