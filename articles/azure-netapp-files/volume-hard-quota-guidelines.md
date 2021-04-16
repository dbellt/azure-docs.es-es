---
title: Implicaciones de cambiar a la cuota máxima de volumen para el servicio de Azure NetApp Files | Microsoft Docs
description: Describe la transición al uso de cuotas máximas de volumen, cómo planear el cambio y cómo supervisar y administrar las capacidades.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: b-juche
ms.openlocfilehash: 52e855cb4ab42ed78a055ecdc31cffa886dc9bf2
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105731996"
---
# <a name="what-changing-to-volume-hard-quota-means-for-your-azure-netapp-files-service"></a>Implicaciones de cambiar a la cuota máxima de volumen para el servicio de Azure NetApp Files

Desde el principio del servicio, Azure NetApp Files ha usado un aprovisionamiento de grupo de capacidad y un mecanismo de crecimiento automático. Los volúmenes de Azure NetApp Files se aprovisionan de forma fina sobre un grupo de capacidad subyacente aprovisionado por el cliente de un nivel y tamaño seleccionados. Los tamaños de volumen (cuotas) se usan para proporcionar rendimiento y capacidad, y las cuotas se pueden ajustar sobre la marcha en cualquier momento. Este comportamiento implica que, actualmente, la cuota de volumen es un factor de rendimiento que se usa para controlar el ancho de banda al volumen. Actualmente, los grupos de capacidades subyacentes aumentan automáticamente cuando se llena la capacidad.   

> [!IMPORTANT] 
> El comportamiento de Azure NetApp Files para el aprovisionamiento del grupo de capacidad y volumen se cambiará a un mecanismo *manual* y *controlable*. **A partir del 30 de abril de 2021 (actualizado), los tamaños de volumen (cuota) determinarán el rendimiento del ancho de banda, así como la capacidad aprovisionada, y los grupos de capacidad subyacentes ya no crecerán automáticamente.** 

## <a name="reasons-for-the-change-to-volume-hard-quota"></a>Motivos para el cambio a cuota máxima de volumen

Muchos clientes han indicado tres desafíos importantes con el comportamiento *inicial*:
* Los clientes de VM veían la capacidad de aprovisionamiento fino (100 TiB) de cualquier volumen al usar las herramientas de supervisión del espacio o la capacidad del sistema operativo, lo que ofrecía datos inexactos sobre la capacidad del cliente o la aplicación.
* Los propietarios de la aplicación no tenían ningún control sobre el espacio aprovisionado del grupo de capacidad (y el costo asociado), debido al comportamiento de crecimiento automático del grupo de capacidad. Esta situación resulta complicada en entornos en los que los "procesos descontrolados" puede llenarse y aumentar rápidamente el costo y la capacidad aprovisionados.
* Los clientes quieren ver y mantener una correlación directa entre el tamaño del volumen (cuota) y el rendimiento. Con el comportamiento actual de sobresuscripción (implícita) de un volumen (en términos de capacidad) y el crecimiento automático de grupos, los clientes no tienen una correlación directa hasta que la cuota de volumen se ha establecido o restablecido activamente. 

Muchos clientes han solicitado control directo sobre la capacidad aprovisionada. Quieren controlar y equilibrar la capacidad de almacenamiento y la utilización. También quieren controlar el costo y obtener visibilidad tanto para las aplicaciones como para el cliente de la capacidad y rendimiento disponibles, utilizados y aprovisionados para los volúmenes de aplicaciones. 

## <a name="what-is-the-volume-hard-quota-change"></a>Qué es el cambio a la cuota máxima de volumen   

Con el cambio a la cuota máxima de volumen, los volúmenes de Azure NetApp Files dejarán de aprovisionarse de manera fina a 100 TiB (el máximo). Los volúmenes se aprovisionarán según el tamaño configurado real (cuota). Además, los grupos de capacidades subyacentes dejarán de aumentar automáticamente al alcanzar el consumo de capacidad total. Este cambio reflejará un comportamiento como el de Azure Managed Disks, que también se aprovisiona tal cual, sin aumento automático de la capacidad.

Por ejemplo, imagine un volumen de Azure NetApp Files configurado con un tamaño de 1 TiB (cuota) en un grupo de capacidad de nivel de servicio Ultra de 4 TiB. Una aplicación está escribiendo datos continuamente en el volumen.

El comportamiento *inicial* era el siguiente:  
* Ancho de banda previsto: 128 MiB/s
* Capacidad total utilizable (y visible para el cliente): 100 TiB   
    No podrá escribir en el volumen más datos que superen este tamaño.
* Grupo de capacidad: Aumenta automáticamente en incrementos de 1 TiB cuando está lleno.
* Cambio a cuota del volumen: Solo cambia el rendimiento (ancho de banda) del volumen. No cambia la capacidad utilizable o visible para el cliente.

Comportamiento *modificado*:  
* Ancho de banda previsto: 128 MiB/s
* Capacidad total utilizable (y visible para el cliente): 1 TiB. No podrá escribir en el volumen más datos que superen este tamaño.
* Grupo de capacidad: Sigue teniendo un tamaño de 4 TiB y no aumenta automáticamente. 
* Cambio a cuota del volumen: Cambia el rendimiento (ancho de banda) y la capacidad utilizable o visible para el cliente del volumen.

Debe supervisar de forma proactiva el uso de los volúmenes y grupos de capacidad de Azure NetApp Files. Debe cambiar deliberadamente el uso del volumen y del grupo para que el consumo sea casi total. Azure NetApp Files seguirá permitiendo ñas [operaciones de cambio de tamaño del grupo de capacidad y de volumen sobre la marcha](azure-netapp-files-resize-capacity-pools-or-volumes.md).

## <a name="how-to-operationalize-the-volume-hard-quota-change"></a>Cómo poner en funcionamiento el cambio de cuota máxima de volumen

En esta sección se proporcionan instrucciones sobre cómo poner en funcionamiento el cambio a la cuota máxima de volumen para una transición sin problemas. También proporciona información para administrar los volúmenes y grupos de capacidad aprovisionados actualmente, la supervisión continua y las opciones de alertas y administración de capacidad.

### <a name="currently-provisioned-volumes-and-capacity-pools"></a>Volúmenes y grupos de capacidad aprovisionados actualmente

Debido al cambio de cuota máxima de volumen, debe cambiar el modelo operativo. Los volúmenes y grupos de capacidad aprovisionados requerirán una administración continua de la capacidad.  Dado que el comportamiento modificado se realizará de manera instantánea, el equipo de Azure NetApp Files recomienda una serie de medidas correctivas únicas para los volúmenes y grupos de capacidad existentes previamente aprovisionados, tal como se describe en esta sección.

#### <a name="one-time-corrective-or-preventative-measures-recommendations"></a>Recomendaciones de medidas correctivas o preventivas únicas  

El cambio de cuota máxima del volumen provocará cambios en la capacidad aprovisionada y disponible para los volúmenes y grupos aprovisionados previamente. Como resultado, pueden generarse algunos desafíos de asignación de capacidad. Para evitar situaciones de falta de espacio a corto plazo para los clientes, el equipo de Azure NetApp Files recomienda las siguientes medidas correctivas o preventivas únicas: 

* **Tamaños de volúmenes aprovisionados**:   
    Cambie el tamaño de todos los volúmenes aprovisionados para que tengan un búfer adecuado en función de la tasa de cambios y alertas o del tiempo de respuesta de cambio de tamaño (por ejemplo, un 20 % en función de las consideraciones típicas de la carga de trabajo), con un máximo de 100 TiB (que es el [límite de tamaño del volumen](azure-netapp-files-resource-limits.md#resource-limits)). Este nuevo tamaño de volumen, incluida la capacidad de búfer, debe basarse en los siguientes factores:
    * La capacidad de volumen **aprovisionada**, en caso de que la capacidad usada sea menor que la cuota de volumen aprovisionada.
    * La capacidad de volumen **utilizada**, en el caso de que la capacidad usada sea mayor que la cuota de volumen aprovisionada.  
    No hay ningún cargo adicional por el aumento de la capacidad del nivel de volumen si no es necesario aumentar el grupo de capacidad subyacente. Como consecuencia de este cambio, puede notar un *aumento* del límite de ancho de banda para el volumen (en caso de que se use el [tipo de grupo de capacidad QoS automático](azure-netapp-files-understand-storage-hierarchy.md#qos_types) ).

* **Tamaños de grupo de capacidad aprovisionada**:   
    después de los ajustes de tamaño del volumen, si la suma de los tamaños de los volúmenes es mayor que el tamaño del grupo de capacidad que los hospeda, el grupo de capacidad tendrá que aumentar hasta un tamaño igual o mayor que la suma de los volúmenes, con un máximo de 500 TiB (que es el [límite de tamaño del grupo de capacidad](azure-netapp-files-resource-limits.md#resource-limits)). La capacidad adicional del grupo de capacidad estará sujeta a cargos de ACR de la forma habitual.

Debe colaborar con sus especialistas en Azure NetApp Files para validar su entorno si necesita ayuda con la configuración de la supervisión o las alertas, tal y como se describe en las secciones siguientes.

### <a name="ongoing-capacity-management"></a>Administración de la capacidad continua  

Después de llevar a cabo las medidas correctivas únicas, debe preparar procesos continuos para supervisar y administrar la capacidad. En las secciones siguientes se proporcionan sugerencias y alternativas sobre la supervisión y administración de la capacidad.

### <a name="monitor-capacity-utilization"></a>Supervisión del uso de capacidad

Puede supervisar el uso de capacidad en varios niveles. 

#### <a name="vm-level-monitoring"></a>Supervisión en el nivel de VM 

El nivel más general de supervisión (más cercano a la aplicación) es desde la máquina virtual de la aplicación. Notará un cambio de comportamiento en información de capacidad desde el sistema operativo cliente de la VM.

En los dos escenarios siguientes, imagine un volumen de Azure NetApp Files configurado con un tamaño de 1 TiB (cuota) en un grupo de capacidad de nivel de servicio Ultra de 4 TiB. 

##### <a name="windows"></a>Windows

Los clientes de Windows pueden comprobar la capacidad usada y disponible de un volumen mediante las propiedades de unidad asignada a la red. Puede usar la opción **Explorador** -> **Unidad** -> **Propiedades**.  

En los siguientes ejemplos se muestra la información sobre capacidad del volumen en Windows *antes* del comportamiento modificado:

![Capturas de pantallas que muestran la capacidad de almacenamiento de un volumen de ejemplo antes del cambio de comportamiento.](../media/azure-netapp-files/hard-quota-windows-capacity-before.png)

También puede usar el comando `dir` en el símbolo del sistema como se muestra a continuación:

![Captura de pantalla que muestra el uso de un comando para mostrar la capacidad de almacenamiento de un volumen antes del cambio de comportamiento.](../media/azure-netapp-files/hard-quota-command-capacity-before.png)

En los siguientes ejemplos se muestra la información sobre capacidad del volumen en Windows *después* del comportamiento modificado:

![Capturas de pantallas que muestran la capacidad de almacenamiento de un volumen de ejemplo después del cambio de comportamiento.](../media/azure-netapp-files/hard-quota-windows-capacity-after.png)

En el siguiente ejemplo se muestra la salida del comando `dir`:  

![Captura de pantalla que muestra el uso de un comando para mostrar la capacidad de almacenamiento de un volumen después del cambio de comportamiento.](../media/azure-netapp-files/hard-quota-command-capacity-after.png)

##### <a name="linux"></a>Linux 

Los clientes de Linux pueden comprobar la capacidad usada y disponible de un volumen mediante el [comando `df`](https://linux.die.net/man/1/df). La opción `-h` mostrará el tamaño, el espacio usado y el espacio disponible en formato legible, con tamaños de unidad M, G y T.

En el siguiente ejemplo se muestra la información sobre capacidad del volumen en Linux *antes* del comportamiento modificado:  

![Captura de pantalla que muestra el uso de Linux para mostrar la capacidad de almacenamiento de un volumen antes del cambio de comportamiento.](../media/azure-netapp-files/hard-quota-linux-capacity-before.png)

En el siguiente ejemplo se muestra la información sobre capacidad del volumen en Linux *después* del comportamiento modificado:  

![Captura de pantalla que muestra el uso de Linux para mostrar la capacidad de almacenamiento de un volumen después del cambio de comportamiento.](../media/azure-netapp-files/hard-quota-linux-capacity-after.png)


### <a name="configure-alerts-using-anfcapacitymanager"></a>Configuración de alertas mediante ANFCapacityManager

Puede usar la herramienta ANFCapacityManager de Logic Apps con soporte de la comunidad para supervisar la capacidad de Azure NetApp Files y recibir alertas adaptadas. La herramienta ANFCapacityManager está disponible en la página [de ANFCapacityManager en GitHub](https://github.com/ANFTechTeam/ANFCapacityManager).

ANFCapacityManager es una aplicación lógica de Azure que administra reglas de alertas basadas en la capacidad. Aumenta automáticamente el tamaño de los volúmenes para evitar que los volúmenes de Azure NetApp Files se queden sin espacio. Es fácil de implementar y proporciona las siguientes funcionalidades de Alert Management:

* Cuando se crea un grupo de capacidad o volumen de Azure NetApp Files, ANFCapacityManager crea una regla de alertas de métricas basada en el umbral de porcentaje consumido que se especifique.
* Cuando se cambia el tamaño de un grupo de capacidad o volumen de Azure NetApp Files, ANFCapacityManager modifica la regla de alertas de métricas basada en el umbral de porcentaje de capacidad consumido que se especifique. Si la regla de alertas no existe, se creará.
* Cuando se elimina un grupo de capacidad o volumen de Azure NetApp Files, se eliminará la regla de alertas de métricas correspondiente.

Puede configurar las siguientes opciones de alertas importantes:  

* **Umbral de porcentaje completo de grupo de capacidad**: este valor determina el umbral utilizado que desencadena una alerta para los grupos de capacidad. Un valor de 90 haría que se desencadene una alerta cuando el grupo de capacidad alcance el 90 % de consumo.
* **Umbral de porcentaje completo de volumen**: esta opción determina el umbral utilizado que desencadena una alerta para los volúmenes. Un valor de 80 haría que se desencadene una alerta cuando el volumen alcance el 80 % de consumo.
* **Grupo de acciones existente para las notificaciones de capacidad**: este valor es el grupo de acciones que se desencadenará para las alertas basadas en la capacidad. El usuario debe crear este valor previamente. El grupo de acciones puede enviar correos electrónicos, SMS u otros formatos.

La siguiente ilustración muestra la configuración de alertas:  

![Ilustración en la que se muestra la configuración de alertas mediante ANFCapacityManager.](../media/azure-netapp-files/hard-quota-anfcapacitymanager-configuration.png)

Después de instalar ANFCapacityManager, puede esperarse el siguiente comportamiento: Cuando se crea, modifica o elimina un grupo de capacidad o volumen de Azure NetApp Files, la aplicación lógica crea, modifica o elimina automáticamente una regla de alerta de métricas basada en la capacidad con el nombre `ANF_Pool_poolname` o `ANF_Volume_poolname_volname`. 

### <a name="manage-capacity"></a>Administrar la capacidad

Además de la supervisión y las alertas, también debe incorporar prácticas de administración de la capacidad de la aplicación para administrar el consumo de capacidad (aumentado) de Azure NetApp Files. Cuando se llena un volumen o grupo de capacidad de Azure NetApp Files, [se puede proporcionar capacidad adicional sin interrumpir la aplicación](azure-netapp-files-resize-capacity-pools-or-volumes.md). En esta sección se describen varias formas manuales y automatizadas para aumentar el espacio aprovisionado del grupo de capacidad y del volumen según sea necesario.
 
#### <a name="manual"></a>Manual 

Puede usar el portal o la CLI para aumentar manualmente el tamaño del volumen o del grupo de capacidad. 

##### <a name="portal"></a>Portal 

Puede [cambiar el tamaño de un volumen](azure-netapp-files-resize-capacity-pools-or-volumes.md#resize-a-volume) según sea necesario. El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo.

1. En la hoja de administración de cuenta de NetApp, haga clic en **Volúmenes**.  
2. Haga clic con el botón derecho en el nombre del volumen cuyo tamaño quiere cambiar o haga clic en el icono `…` al final de la fila del volumen para mostrar el menú contextual. 
3. Use las opciones del menú contextual para cambiar el tamaño del volumen o eliminarlo.   

   ![Captura de pantalla que muestra las opciones del menú contextual de un volumen.](../media/azure-netapp-files/hard-quota-volume-options.png) 

   ![Captura de pantalla que muestra la ventada Actualizar cuota de volumen.](../media/azure-netapp-files/hard-quota-update-volume-quota.png) 

En algunos casos, el grupo de capacidad de hospedaje no tiene suficiente capacidad para cambiar el tamaño de los volúmenes. No obstante, puede [cambiar el tamaño del grupo de capacidad](azure-netapp-files-resize-capacity-pools-or-volumes.md#resize-the-capacity-pool) en incrementos o decrementos de 1 TiB. El tamaño del grupo de capacidad no puede ser inferior a 4 TiB. *Al cambiar el tamaño del grupo de capacidad, cambia la capacidad adquirida de Azure NetApp Files.*

1. En la hoja de administración de cuenta de NetApp, haga clic en el grupo de capacidad cuyo tamaño desee cambiar.
2. Haga clic con el botón derecho en el nombre del grupo de capacidad o haga clic en el icono `…` al final de la fila del grupo de capacidad para mostrar el menú contextual.
3. Use las opciones del menú contextual para cambiar el tamaño del grupo de capacidad o eliminarlo.    

   ![Captura de pantalla que muestra las opciones del menú contextual de un grupo de capacidad.](../media/azure-netapp-files/hard-quota-pool-options.png) 

   ![Captura de pantalla que muestra la ventana Cambiar el tamaño del grupo.](../media/azure-netapp-files/hard-quota-update-resize-pool.png) 


##### <a name="cli-or-powershell"></a>CLI o PowerShell

Puede usar [las herramientas de la CLI de Azure NetApp Files](azure-netapp-files-sdk-cli.md#cli-tools), incluidas la CLI de Azure y Azure PowerShell, para cambiar manualmente el tamaño del volumen o del grupo de capacidad.  Puede usar los dos comandos siguientes para administrar los recursos de grupo y volumen de Azure NetApp Files:  

* [`az netappfiles pool`](/cli/azure/netappfiles/pool)
* [`az netappfiles volume`](/cli/azure/netappfiles/volume)

Para administrar los recursos de Azure NetApp Files mediante la CLI de Azure, puede abrir Azure Portal y seleccionar el vínculo Azure **Cloud Shell** en la parte superior de la barra de menús: 

[ ![Captura de pantalla que muestra cómo acceder al vínculo de Cloud Shell.](../media/azure-netapp-files/hard-quota-update-cloud-shell-link.png) ](../media/azure-netapp-files/hard-quota-update-cloud-shell-link.png#lightbox)

Esta acción abrirá Azure Cloud Shell:

[ ![Captura de pantalla que muestra la ventana de Cloud Shell.](../media/azure-netapp-files/hard-quota-update-cloud-shell-window.png) ](../media/azure-netapp-files/hard-quota-update-cloud-shell-window.png#lightbox)

En los ejemplos siguientes se usan los comandos para [mostrar](/cli/azure/netappfiles/volume#az-netappfiles-volume-show) y [actualizar](/cli/azure/netappfiles/volume#az-netappfiles-volume-update) el tamaño de un volumen:
 
[ ![Captura de pantalla que muestra el uso de PowerShell para mostrar el tamaño del volumen.](../media/azure-netapp-files/hard-quota-update-powershell-volume-show.png) ](../media/azure-netapp-files/hard-quota-update-powershell-volume-show.png#lightbox)

[ ![Captura de pantalla que muestra el uso de PowerShell para actualizar el tamaño del volumen.](../media/azure-netapp-files/hard-quota-update-powershell-volume-update.png) ](../media/azure-netapp-files/hard-quota-update-powershell-volume-update.png#lightbox)

En los ejemplos siguientes se usan los comandos para [mostrar](/cli/azure/netappfiles/pool#az-netappfiles-pool-show) y [actualizar](/cli/azure/netappfiles/pool#az-netappfiles-pool-update) el tamaño de un grupo de capacidad:

[ ![Captura de pantalla que muestra el uso de PowerShell para mostrar el tamaño del grupo de capacidad.](../media/azure-netapp-files/hard-quota-update-powershell-pool-show.png) ](../media/azure-netapp-files/hard-quota-update-powershell-pool-show.png#lightbox) 

[ ![Captura de pantalla que muestra el uso de PowerShell para actualizar el tamaño del grupo de capacidad.](../media/azure-netapp-files/hard-quota-update-powershell-pool-update.png) ](../media/azure-netapp-files/hard-quota-update-powershell-pool-update.png#lightbox)

#### <a name="automated"></a>Automatizado  

Puede crear un proceso automatizado para administrar el comportamiento modificado.

##### <a name="rest-api"></a>API DE REST   

La API REST para el servicio Azure NetApp Files define las operaciones HTTP en los recursos como la cuenta de NetApp, el grupo de capacidades, los volúmenes y las instantáneas. La especificación de la API REST de Azure NetApp Files se publica a través de la [página de GitHub de Azure NetApp Files Resource Manager](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager)]. Puede encontrar [código de ejemplo para usarlo con las API REST](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager/Microsoft.NetApp/stable/2020-06-01/examples) en GitHub.

Consulte [Desarrollo para Azure NetApp Files con la API REST](azure-netapp-files-develop-with-rest-api.md). 

##### <a name="rest-api-using-powershell"></a>API REST mediante PowerShell  

La API REST para el servicio Azure NetApp Files define las operaciones HTTP en los recursos como la cuenta de NetApp, el grupo de capacidades, los volúmenes y las instantáneas. La [especificación de la API REST de Azure NetApp Files](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/netapp/resource-manager) se publica a través de GitHub.

Consulte [Desarrollo para Azure NetApp Files con la API REST mediante PowerShell](develop-rest-api-powershell.md).

##### <a name="capacity-management-using-anfcapacitymanager"></a>Administración de la capacidad mediante ANFCapacityManager

ANFCapacityManager es una aplicación lógica de Azure que administra reglas de alertas basadas en la capacidad. Aumenta automáticamente el tamaño de los volúmenes para evitar que los volúmenes de Azure NetApp Files se queden sin espacio. Además de enviar alertas, puede habilitar el aumento automático del tamaño de los volúmenes y grupos de capacidad para evitar que los volúmenes de Azure NetApp Files se queden sin espacio: 

* Opcionalmente, cuando un volumen de Azure NetApp Files alcanza el umbral de porcentaje consumido especificado, la cuota (tamaño) de volumen aumentará en función del porcentaje especificado entre 10 y 100.  
* Si el aumento del tamaño del volumen supera la capacidad del grupo de capacidad contenedor, también se aumentará el tamaño del grupo de capacidad para alojar el tamaño del nuevo volumen.

Puede configurar el siguiente valor de administración de capacidad importante:  

* **Porcentaje de aumento del crecimiento automático**: porcentaje del tamaño del volumen existente que un volumen aumentará si alcanza el **umbral de porcentaje completo** especificado. Un valor de 0 (cero) deshabilitará la característica de crecimiento automático. Se recomienda un valor entre 10 y 100.

    ![Captura de pantalla que muestra la ventana Establecer porcentaje de crecimiento automático del volumen.](../media/azure-netapp-files/hard-quota-volume-anfcapacitymanager-auto-grow-percent.png) 

## <a name="faq"></a>Preguntas más frecuentes 

En esta sección se responden algunas preguntas sobre el cambio de cuota máxima de volumen. 

### <a name="does-snapshot-space-count-towards-the-usable-or-provisioned-capacity-of-a-volume"></a>¿Se cuenta el espacio de instantáneas en la capacidad utilizable o aprovisionada de un volumen?

Sí, la capacidad de instantáneas consumida cuenta para el espacio aprovisionado del volumen. En caso de que el volumen se llene, plantéese dos opciones de corrección:

* Cambie el tamaño del volumen como se describe en este artículo.
* Quite las instantáneas más antiguas para liberar espacio en el volumen de hospedaje.

### <a name="does-this-change-mean-the-volume-auto-grow-behavior-will-disappear-from-azure-netapp-files"></a>¿Este cambio implica que el comportamiento del crecimiento automático del volumen desaparecerá de Azure NetApp Files?

Un error frecuente es creer que los *volúmenes* de Azure NetApp Files aumentarán su tamaño automáticamente al llenarse. Los volúmenes se aprovisionaban de forma fina con un tamaño de 100 TiB, independientemente de la cuota real establecida, mientras que el *grupo de capacidad* subyacente aumentaba de tamaño automáticamente en incrementos de 1 TiB. Este cambio corregirá el tamaño *volumen* (visible y utilizable) a la cuota establecida y los *grupos de capacidad* ya no aumentarán de tamaño automáticamente. Este cambio da como resultado un espacio de cliente e información de capacidad más precisos y comúnmente solicitados. Evita el consumo de capacidad "descontrolada".

### <a name="does-this-change-have-any-effect-on-volumes-replicated-with-cross-region-replication-preview"></a>¿Este cambio tiene algún efecto en los volúmenes replicados mediante la replicación entre regiones (versión preliminar)? 

La cuota máxima de volumen no se aplica en los volúmenes de destino de replicación.

### <a name="does-this-change-have-any-effect-on-metrics-currently-available-in-azure-monitor"></a>¿Este cambio tiene algún efecto en las métricas disponibles actualmente en Azure Monitor?

Las métricas del portal y las estadísticas de Azure Monitor reflejarán con precisión el nuevo modelo de asignación y uso.

### <a name="does-this-change-have-any-effect-on-the-resource-limits-for-azure-netapp-files"></a>¿Este cambio tiene algún efecto en los límites de recursos para Azure NetApp Files?

No hay ningún cambio en los límites de recursos para Azure NetApp Files adicionales a los cambios de cuota descritos en este artículo.

### <a name="is-there-an-example-anfcapacitymanager-workflow"></a>¿Hay algún flujo de trabajo de ANFCapacityManager de ejemplo?  

Sí. Consulte la [página del ejemplo de flujo de trabajo de crecimiento automático de volúmenes en GitHub](https://github.com/ANFTechTeam/ANFCapacityManager/blob/master/ResizeWorkflow.md).

### <a name="is-anfcapacitymanager-microsoft-supported"></a>¿ANFCapacityManager es compatible con Microsoft?  

[La aplicación lógica ANFCapacityManager se proporciona tal cual y no es compatible con NetApp ni Microsoft](https://github.com/ANFTechTeam/ANFCapacityManager#disclaimer). Se recomienda que la modifique para que se ajuste a su entorno o requisitos específicos. Debe probar la funcionalidad antes de implementarla en cualquier entorno de producción o crítico para la empresa.

### <a name="how-can-i-report-a-bug-or-submit-a-feature-request-for-anfcapacitymanger"></a>¿Cómo puedo informar de un error o enviar la solicitud de una característica para ANFCapacityManger?
Para enviar errores y solicitudes de características, haga clic en **New Issue** (problema nuevo) en la página [ANFCapacityManager de GitHub](https://github.com/ANFTechTeam/ANFCapacityManager/issues).

## <a name="next-steps"></a>Pasos siguientes
* [Cambiar el tamaño de un grupo de capacidad o de un volumen](azure-netapp-files-resize-capacity-pools-or-volumes.md) 
* [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md)
