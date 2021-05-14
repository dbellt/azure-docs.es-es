---
title: Guía de solución de problemas de rendimiento de recursos compartidos de archivos de Azure
description: Solución de problemas de rendimiento conocidos con los recursos compartidos de archivos de Azure. Detecte las posibles causas y soluciones alternativas asociadas cuando se producen estos problemas.
author: roygara
ms.service: storage
ms.topic: troubleshooting
ms.date: 11/16/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: b303dbc20cf0caf4bb0d75f28a2983bc0f27064d
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065032"
---
# <a name="troubleshoot-azure-file-shares-performance-issues"></a>Solución de problemas de rendimiento de recursos compartidos de archivos de Azure

En este artículo se enumeran algunos problemas habituales relacionados con los recursos compartidos de archivos de Azure. Se proporcionan las posibles causas y soluciones alternativas cuando se detectan estos problemas.

## <a name="high-latency-low-throughput-and-general-performance-issues"></a>Alta latencia, rendimiento bajo y problemas de rendimiento general

### <a name="cause-1-share-was-throttled"></a>Causa 1: Se ha limitado el recurso compartido de archivos

Las solicitudes se limitan cuando se alcanzan los límites de operaciones de E/S por segundo (IOPS), entradas o salidas de un recurso compartido de archivos. Para conocer los límites de los recursos compartidos de archivos de nivel Estándar y Premium, consulte [Objetivos de escalabilidad de archivos y recursos compartidos de archivos](./storage-files-scale-targets.md#azure-file-share-scale-targets).

Para confirmar si se está limitando el recurso compartido, puede acceder y usar las métricas de Azure en el portal.

1. En Azure Portal, vaya a la cuenta de almacenamiento.

1. En el panel de la izquierda, en **Supervisión**, seleccione **Métricas**.

1. Seleccione **Archivo** como el espacio de nombres de la métrica para el ámbito de la cuenta de almacenamiento.

1. Seleccione **Transacciones** como métrica.

1. Agregue un filtro para **Tipo de respuesta** y, luego, compruebe si se ha limitado alguna solicitud. 

    Si se limita una solicitud en los recursos compartidos de archivos estándar, se registran los siguientes tipos de respuesta:

    - SuccessWithThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareIopsThrottlingError

    Si se limita una solicitud en los recursos compartidos de archivos prémium, se registran los siguientes tipos de respuesta:

    - SuccessWithShareEgressThrottling
    - SuccessWithShareIngressThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareEgressThrottlingError
    - ClientShareIngressThrottlingError
    - ClientShareIopsThrottlingError

    Para más información sobre cada tipo de respuesta, consulte [Dimensiones de métricas](./storage-files-monitoring-reference.md#metrics-dimensions).

    ![Captura de pantalla de las opciones de métricas para recursos compartidos de archivos Premium que muestra un filtro de propiedad "Tipo de respuesta".](media/storage-troubleshooting-premium-fileshares/metrics.png)

    > [!NOTE]
    > Para recibir una alerta, consulte la sección ["Creación de una alerta si un recurso compartido de archivos se limita"](#how-to-create-an-alert-if-a-file-share-is-throttled) más adelante en este artículo.

### <a name="solution"></a>Solución

- Si usa un recurso compartido de archivos Estándar, habilite [Recursos compartidos de archivos grandes](./storage-files-how-to-create-large-file-share.md?tabs=azure-portal) en su cuenta de almacenamiento. Los recursos compartidos de archivos grandes admiten hasta 10 000 IOPS por recurso compartido.
- Si usa un recurso compartido de archivos Premium, aumente el tamaño del recurso compartido de archivos aprovisionado para aumentar el límite de IOPS. Para más información, consulte [Descripción del aprovisionamiento de recursos compartidos de archivos premium](./understanding-billing.md#provisioned-model).

### <a name="cause-2-metadata-or-namespace-heavy-workload"></a>Causa 2: Carga de trabajo pesada del espacio de nombres o los metadatos

Si la mayoría de las solicitudes se centran en metadatos (como `createfile`, `openfile`, `closefile`, `queryinfo` o `querydirectory`), la latencia será peor en comparación con las operaciones de lectura y escritura.

Para determinar si la mayoría de las solicitudes están centradas en los metadatos, empiece por los pasos del 1 al 4, como se ha descrito anteriormente en la causa 1. En el paso 5, en lugar de agregar un filtro para **Tipo de respuesta**, agregue un filtro de propiedad para **Nombre de API**.

![Captura de pantalla de las opciones de métricas para recursos compartidos de archivos Premium que muestra un filtro de propiedad "Nombre de API".](media/storage-troubleshooting-premium-fileshares/MetadataMetrics.png)

### <a name="workaround"></a>Solución alternativa

- Compruebe si la aplicación se puede modificar para reducir el número de operaciones de metadatos.
- Agregue un disco duro virtual (VHD) al recurso compartido de archivos y móntelo a través de SMB desde el cliente para realizar operaciones de archivos en los datos. Este enfoque funciona para escenarios de escritor o lector único o escenarios con varios lectores y sin escritores. Dado que el sistema de archivos es propiedad del cliente y no de Azure Files, esto permite que las operaciones de metadatos sean locales. La configuración ofrece un rendimiento similar al de un almacenamiento con conexión directa local.

### <a name="cause-3-single-threaded-application"></a>Causa 3: Aplicación de un único subproceso

Si la aplicación que usa tiene un único subproceso, esta configuración puede dar lugar a un rendimiento de IOPS significativamente menor que el máximo posible, en función del tamaño del recurso compartido aprovisionado.

### <a name="solution"></a>Solución

- Aumente el paralelismo de la aplicación aumentando el número de subprocesos.
- Cambie a aplicaciones en las que el paralelismo sea posible. Por ejemplo, para las operaciones de copia, podría usar AzCopy o RoboCopy desde clientes de Windows o el comando **parallel** en los clientes de Linux.

## <a name="very-high-latency-for-requests"></a>Latencia muy alta para las solicitudes

### <a name="cause"></a>Causa

La máquina virtual (VM) del cliente podría encontrarse en una región diferente de la del recurso compartido de archivos. Otro motivo por el que la latencia es alta podría ser por la latencia causada por el cliente o la red.

### <a name="solution"></a>Solución

- Ejecute la aplicación desde una máquina virtual que se encuentre en la misma región que el recurso compartido de archivos.
- En la cuenta de almacenamiento, revise las métricas de transacción **SuccessE2ELatency** y **SuccessServerLatency** a través **Azure Monitor** en Azure Portal. Si hay una gran diferencia entre las métricas SuccessE2ELatency y SuccessServerLatency, significa que probablemente la latencia es debido a la red y el cliente. Consulte [Métricas de transacción](storage-files-monitoring-reference.md#transaction-metrics) en la referencia de datos de supervisión de Azure Files.

## <a name="client-unable-to-achieve-maximum-throughput-supported-by-the-network"></a>El cliente no puede lograr el rendimiento máximo admitido por la red

### <a name="cause"></a>Causa
Una posible causa es la ausencia de compatibilidad multicanal de SMB para recursos compartidos de archivos estándar. Actualmente, Azure Files solo admite un único canal, por lo que solo hay una conexión desde la máquina virtual del cliente al servidor. Esta conexión única se fija a un único núcleo en la máquina virtual del cliente, por lo que el rendimiento máximo alcanzable desde una máquina virtual está limitado por un solo núcleo.

### <a name="workaround"></a>Solución alternativa

- En el caso de recursos compartidos de archivos Premium, [habilite SMB multicanal en una cuenta de FileStorage](storage-files-enable-smb-multichannel.md).
- Obtener una máquina virtual con un núcleo más grande puede ayudar a mejorar el rendimiento.
- Ejecutar la aplicación cliente desde varias máquinas virtuales aumentará el rendimiento.
- Siempre que sea posible, use las API de REST.

## <a name="throughput-on-linux-clients-is-significantly-lower-than-that-of-windows-clients"></a>El rendimiento en los clientes de Linux es significativamente menor que el de los clientes de Windows

### <a name="cause"></a>Causa

Se trata de un problema conocido relacionado con la implementación del cliente de SMB en Linux.

### <a name="workaround"></a>Solución alternativa

- Distribuir la carga entre varias máquinas virtuales.
- En la misma máquina virtual, use varios puntos de montaje con la opción `nosharesock` y propague la carga entre estos puntos de montaje.
- En Linux, intente montar una opción `nostrictsync` para evitar forzar un vaciado de SMB en cada llamada a `fsync`. Para Azure Files, esta opción no afecta a la coherencia de los datos, pero puede producir metadatos de archivos obsoletos en la lista de directorios (comando `ls -l`). La consulta directa de metadatos de archivo mediante el comando `stat` devolverá los metadatos de archivo más recientes.

## <a name="high-latencies-for-metadata-heavy-workloads-involving-extensive-openclose-operations"></a>Latencias altas para cargas de trabajo con muchos metadatos que implican operaciones de apertura y cierre extensas

### <a name="cause"></a>Causa

Falta de compatibilidad para las concesiones de directorios.

### <a name="workaround"></a>Solución alternativa

- Si es posible, evite usar aperturas o cierres excesivos en el mismo directorio en un período de tiempo breve.
- Para las máquinas virtuales de Linux, aumente el tiempo de espera de caché de entrada de directorio especificando `actimeo=<sec>` como opción de montaje. De forma predeterminada, el tiempo de espera es de 1 segundo, por lo que un valor mayor, como 3 o 5 segundos, podría ser útil.
- En el caso de las máquinas virtuales de CentOS Linux o Red Hat Enterprise Linux (RHEL), actualice el sistema a CentOS Linux 8.2 o RHEL 8.2. En cuanto a las máquinas virtuales de Linux, actualice el kernel a 5.0 o una versión posterior.

## <a name="low-iops-on-centos-linux-or-rhel"></a>IOPS bajas en CentOS Linux o RHEL

### <a name="cause"></a>Causa

Una profundidad de E/S mayor que 1 no se admite en CentOS Linux o RHEL.

### <a name="workaround"></a>Solución alternativa

- Actualice a CentOS Linux 8 o RHEL 8.
- Cambie a Ubuntu.

## <a name="slow-file-copying-to-and-from-azure-file-shares-in-linux"></a>Copia de archivos lenta en recursos compartidos de archivos de Azure y desde ellos en Linux

Si experimenta una copia de archivos lenta, consulte la sección "Copia de archivos lenta en y desde Azure Files en Linux" de la [guía de solución de problemas de Linux](storage-troubleshoot-linux-file-connection-problems.md#slow-file-copying-to-and-from-azure-files-in-linux).

## <a name="jittery-or-sawtooth-pattern-for-iops"></a>Patrón fluctuante o de diente de sierra para IOPS

### <a name="cause"></a>Causa

La aplicación cliente supera constantemente la IOPS de base de referencia. Actualmente, no hay suavizado por parte del servicio de la carga de solicitudes. Si el cliente supera la IOPS de base de referencia, el servicio lo limitará. Dicha limitación puede provocar que el cliente experimente un patrón de IOPS fluctuante o de diente de sierra. En este caso, la IOPS promedio alcanzada por el cliente podría ser inferior a la IOPS de base de referencia.

### <a name="workaround"></a>Solución alternativa
- Reduzca la carga de solicitudes desde la aplicación cliente, de modo que el recurso compartido no se vea limitado.
- Aumente la cuota del recurso compartido para que este no se vea limitado.

## <a name="excessive-directoryopendirectoryclose-calls"></a>Exceso de llamadas DirectoryOpen/DirectoryClose

### <a name="cause"></a>Causa

Si el número de llamadas **DirectoryOpen/DirectoryClose** se encuentra entre las llamadas de API principales y no espera que el cliente realice esa cantidad de llamadas, el problema podría estar producido por el software de antivirus instalado en la máquina virtual del cliente de Azure.

### <a name="workaround"></a>Solución alternativa

- La [Actualización de la plataforma para Windows de abril](https://support.microsoft.com/help/4052623/update-for-windows-defender-antimalware-platform) incluye una corrección para este problema.

## <a name="file-creation-is-slower-than-expected"></a>La creación de archivos es más lenta de lo esperado

### <a name="cause"></a>Causa

Las cargas de trabajo que se basan en la creación de un gran número de archivos no verán una diferencia importante en el rendimiento entre los recursos compartidos de archivos Premium y los recursos compartidos de archivos Estándar.

### <a name="workaround"></a>Solución alternativa

- Ninguno.

## <a name="slow-performance-from-windows-81-or-server-2012-r2"></a>Rendimiento lento desde Windows 8.1 o Server 2012 R2

### <a name="cause"></a>Causa

Latencia mayor a la esperada al acceder a los recursos compartidos de Azure para cargas de trabajo intensivas de E/S.

### <a name="workaround"></a>Solución alternativa

- Instale la [revisión](https://support.microsoft.com/help/3114025/slow-performance-when-you-access-azure-files-storage-from-windows-8-1) disponible.

## <a name="smb-multichannel-option-not-visible-under-file-share-settings"></a>Opción SMB multicanal no visible en la configuración de recurso compartido de archivos. 

### <a name="cause"></a>Causa

La suscripción no está registrada para la característica, o no se admite el tipo de región y cuenta.

### <a name="solution"></a>Solución

Asegúrese de que su suscripción está registrada para la característica SMB multicanal. Consulte [Introducción](storage-files-enable-smb-multichannel.md#getting-started) y asegúrese de que el tipo de cuenta es FileStorage (cuenta de archivo Premium) en la página información general de la cuenta. 

## <a name="smb-multichannel-is-not-being-triggered"></a>No se desencadena SMB multicanal.

### <a name="cause"></a>Causa

Cambios recientes en la configuración de SMB multicanal sin volver a montar.

### <a name="solution"></a>Solución
 
-   Después de cualquier cambio en la configuración de SMB multicanal de la cuenta o cliente SMB de Windows, tiene que desmontar el recurso compartido, esperar 60 segundos y volver a montar el recurso compartido para desencadenar el multicanal.
-   Para el sistema operativo del cliente de Windows, genere una carga de E/S con una profundidad de cola alta, como PC = 8, por ejemplo, copiando un archivo para desencadenar SMB multicanal.  En el caso del sistema operativo de servidor, SMB multicanal se desencadena con PC = 1, lo que significa en cuanto se inicie alguna ES en el recurso compartido.

## <a name="high-latency-on-web-sites-hosted-on-file-shares"></a>Alta latencia en sitios web hospedados en recursos compartidos de archivos 

### <a name="cause"></a>Causa  

La notificación de cambio de un número alto de archivos en recursos compartidos de archivos puede producir latencias altas significativas. Esto suele ocurrir con sitios web hospedados en recursos compartidos de archivos con una estructura de directorios anidada profunda. Un escenario típico es la aplicación web hospedada en IIS donde se configura la notificación de cambio de archivos para cada directorio en la configuración predeterminada. Cada cambio ([ReadDirectoryChangesW](/windows/win32/api/winbase/nf-winbase-readdirectorychangesw)) en el recurso compartido en el que está registrado el cliente SMB envía una notificación de cambio desde el servicio de archivos al cliente, que usa recursos del sistema, y el problema empeora con el número de cambios. Esto puede dar lugar a una limitación de recursos compartidos y, por tanto, a una mayor latencia del lado cliente. 

Para confirmarlo, puede usar las métricas de Azure en el portal: 

1. En Azure Portal, vaya a la cuenta de almacenamiento. 
1. En el menú de la izquierda, en Supervisión, seleccione Métricas. 
1. Seleccione Archivo como el espacio de nombres de la métrica para el ámbito de la cuenta de almacenamiento. 
1. Seleccione Transacciones como métrica. 
1. Agregue un filtro para ResponseType y compruebe si alguna solicitud tiene un código de respuesta de SuccessWithThrottling (para SMB) o ClientThrottlingError (para REST).

### <a name="solution"></a>Solución 

- Si no se usa la notificación de cambio de archivo, deshabilite la notificación de cambio de archivo (preferido).
    - [Deshabilite la notificación de cambio de archivo](https://support.microsoft.com/help/911272/fix-asp-net-2-0-connected-applications-on-a-web-site-may-appear-to-sto) actualizando FCNMode. 
    - Actualice el intervalo de sondeo del proceso de trabajo de IIS (W3WP) a 0 estableciendo `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\W3SVC\Parameters\ConfigPollMilliSeconds ` en el registro y reinicie el proceso W3WP. Para obtener más información acerca de esta configuración, vea las [claves de registro comunes que utilizan muchas partes de IIS](/troubleshoot/iis/use-registry-keys#registry-keys-that-apply-to-iis-worker-process-w3wp).
- Aumente la frecuencia del intervalo de sondeo de notificación de cambio de archivos para reducir el volumen.
    - Actualice el intervalo de sondeo del proceso de trabajo W3WP a un valor mayor (por ejemplo, 10 o 30 minutos) en función de sus requisitos. Establezca `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\W3SVC\Parameters\ConfigPollMilliSeconds ` [en el del registro](/troubleshoot/iis/use-registry-keys#registry-keys-that-apply-to-iis-worker-process-w3wp) y reinicie el proceso W3WP.
- Si el directorio físico asignado del sitio web tiene una estructura de directorios anidada, puede intentar limitar el ámbito de notificación de cambio de archivos para reducir el volumen de notificaciones. De forma predeterminada, IIS usa la configuración de los archivos Web.config del directorio físico al que se asigna el directorio virtual, así como en los directorios secundarios de ese directorio físico. Si no desea usar los archivos Web.config de los directorios secundarios, especifique false para el atributo allowSubDirConfig en el directorio virtual. Se pueden encontrar más detalles [aquí](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories). 
    - Establezca el valor "allowSubDirConfig" del directorio virtual de IIS en el archivo Web.config en *false* para excluir los directorios secundarios físicos asignados del ámbito.  

## <a name="how-to-create-an-alert-if-a-file-share-is-throttled"></a>Creación de una alerta si un recurso compartido de archivos se limita

1. Vaya a la **cuenta de almacenamiento** en **Azure Portal**.
2. En la sección **Supervisión**, haga clic en **Alertas** y, después, haga clic en **+ Nueva regla de alertas**.
3. Haga clic en **Editar recurso**, seleccione el **tipo de recurso de archivo** para la cuenta de almacenamiento y, a continuación, haga clic en **Listo**. Por ejemplo, si el nombre de la cuenta de almacenamiento es `contoso`, seleccione el recurso `contoso/file`.
4. Haga clic en **Agregar condición** para agregar una condición.
5. Verá una lista de señales admitidas para la cuenta de almacenamiento, seleccione la métrica **Transacciones**.
6. En la hoja **Configurar lógica de señal**, haga clic en la lista desplegable **Nombre de la dimensión** y seleccione **Tipo de respuesta**.
7. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los tipos de respuesta adecuados para el recurso compartido de archivos.

    Para recursos compartidos de archivos estándar, seleccione los siguientes tipos de respuesta:

    - SuccessWithThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareIopsThrottlingError

    Para recursos compartidos de archivos prémium, seleccione los siguientes tipos de respuesta:

    - SuccessWithShareEgressThrottling
    - SuccessWithShareIngressThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareEgressThrottlingError
    - ClientShareIngressThrottlingError
    - ClientShareIopsThrottlingError

   > [!NOTE]
   > Si los tipos de respuesta no aparecen en la lista desplegable **Valores de dimensión**, significa que el recurso no se ha limitado. Para agregar los valores de dimensión, junto a la lista desplegable **Valores de dimensión**, seleccione **Agregar valor personalizado**, escriba el tipo de respuesta (por ejemplo, **SuccessWithThrottling**), seleccione **Aceptar** y repita estos pasos para agregar todos los tipos de respuesta correspondientes para el recurso compartido de archivos.

8. Para **recursos compartidos de archivos prémium**, haga clic en el menú desplegable **Nombre de la dimensión** y seleccione **Recurso compartido de archivos**. En el caso de **recursos compartidos de archivos estándar**, vaya al **paso n.º 10**.

   > [!NOTE]
   > Si el recurso compartido de archivos es estándar, en la dimensión **Recurso compartido de archivos** no se mostrarán los recursos compartidos de archivo porque las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar. Las alertas de limitación de los recursos compartidos de archivos estándar se desencadenarán si algún recurso compartido de archivos de la cuenta de almacenamiento está limitado y la alerta no identificará qué recurso compartido de archivos se ha limitado. Dado que las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar, se recomienda tener un recurso compartido de archivos por cada cuenta de almacenamiento.

9. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los recursos compartidos de archivos en los que desea generar alertas.
10. Defina los **parámetros de alerta** (umbral, operador, granularidad de agregación y frecuencia de evaluación) y haga clic en **Listo**.

    > [!TIP]
    > Si usa un umbral estático, el gráfico de métricas puede ayudar a determinar un valor de umbral razonable si el recurso compartido de archivos se está limitando actualmente. Si usa un umbral dinámico, el gráfico de métricas mostrará los umbrales calculados según los datos recientes.

11. Haga clic en **Add action groups** (Agregar grupos de acciones) para agregar un **grupo de acciones** (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
12. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
13. Haga clic en **Crear regla de alerta** para crear la alerta.

Para obtener más información sobre cómo configurar alertas en Azure Monitor, consulte [Introducción sobre las alertas en Microsoft Azure]( https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview).

## <a name="how-to-create-alerts-if-a-premium-file-share-is-trending-toward-being-throttled"></a>Cómo crear alertas si un recurso compartido de archivos Premium tiene tendencia al límite

1. En Azure Portal, vaya a la cuenta de almacenamiento.
2. En la sección **Supervisión**, seleccione **Alertas** y, después, seleccione **Nueva regla de alertas**.
3. Seleccione **Editar recurso**, seleccione el **tipo de recurso de archivo** para la cuenta de almacenamiento y, a continuación, seleccione **Listo**. Por ejemplo, si el nombre de la cuenta de almacenamiento es *contoso*, seleccione contoso/recurso de archivo.
4. Seleccione **Seleccionar condición** para agregar una condición.
5. En la lista de señales admitidas para la cuenta de almacenamiento, seleccione la métrica **Salida**.

   > [!NOTE]
   > Tendrá que crear tres alertas independientes para recibir alertas cuando los valores de la entrada, la salida o la transacción superen los umbrales establecidos. Esto se debe a que una alerta solo se desencadena cuando se cumplen todas las condiciones. Por ejemplo, si pone todas las condiciones en una alerta, solo se le avisaría si la entrada, la salida y las transacciones superaran sus cantidades establecidas de umbral.

6. Desplácese hacia abajo. En la lista desplegable **Nombre de la dimensión**, seleccione **Recurso compartido de archivos**.
7. En la lista desplegable **Valores de dimensión**, seleccione los recursos compartidos de archivos de los que quiere generar alertas.
8. Defina los parámetros de alerta mediante la selección de valores en las listas desplegables **Operador**, **Valor de umbral**, **Granularidad de agregación** y **Frecuencia de evaluación** y luego seleccione **Listo**.

   Las métricas de salida, entrada y transacciones se expresan por minuto, aunque se aprovisiona la salida, la entrada y la E/S por segundo. Por lo tanto, por ejemplo, si la salida aprovisionada es 90&nbsp; MiB/s y quiere que el umbral sea el 80&nbsp;por ciento de la salida aprovisionada, seleccione los siguientes parámetros de alerta: 
   - Para **Valor del umbral**: *75497472* 
   - Para **Operador**: *mayor o igual que*
   - Para **Tipo de agregación**: *promedio*
   
   En función del ruido que desee que tenga la alerta, puede seleccionar también valores para **Granularidad de agregación** y **Frecuencia de evaluación**. Por ejemplo, si quiere que la alerta examine el promedio de entrada durante el período de tiempo de 1 hora y desea que la regla de alerta se ejecute cada hora, seleccione lo siguiente:
   - Para **Granularidad de agregación**: *1 hora*
   - Para **Frecuencia de evaluación**: *1 hora*

9. Seleccione **Add action groups** (Agregar grupos de acciones) y agregue un grupo de acciones (por ejemplo, correo electrónico o SMS) a la alerta; para ello, seleccione un grupo de acciones existente o cree uno.
10. Escriba los detalles de la alerta, como el **Nombre de la regla de alertas**, la **Descripción** y la **Gravedad**.
11. Seleccione **Crear regla de alerta** para crear la alerta.

    > [!NOTE]
    > - Para recibir una notificación de que el recurso compartido de archivos Premium está a punto de limitarse *debido a la entrada aprovisionada*, siga las instrucciones anteriores, pero con el siguiente cambio:
    >    - En el paso 5, seleccione la métrica **Entrada** en lugar de **Salida**.
    >
    > - Para recibir una notificación de que el recurso compartido de archivos Premium está a punto de limitarse *debido a la IOPS aprovisionada*, siga las instrucciones anteriores, pero con el siguiente cambio:
    >    - En el paso 5, seleccione la métrica **Transacciones** en lugar de **Salida**.
    >    - En el paso 10, la única opción para **Tipo de agregación** es *Total*. Por lo tanto, el valor de umbral depende de la granularidad de agregación seleccionada. Por ejemplo, si desea que el umbral sea el 80&nbsp;por ciento de la IOPS de base de referencia aprovisionada y selecciona *1 hora* para la **Granularidad de agregación**, el **Valor de umbral** sería su IOPS de base de referencia (en bytes) &times;&nbsp;0,8 &times;&nbsp;3600. 

Para obtener más información sobre cómo configurar alertas en Azure Monitor, consulte [Introducción sobre las alertas en Microsoft Azure]( https://docs.microsoft.com/azure/azure-monitor/platform/alerts-overview).

## <a name="see-also"></a>Consulte también
- [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)  
- [Solución de problemas de Azure Files en Linux](storage-troubleshoot-linux-file-connection-problems.md)  
- [Preguntas más frecuentes de Azure Files](storage-files-faq.md)
