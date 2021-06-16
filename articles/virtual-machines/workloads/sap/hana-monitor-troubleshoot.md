---
title: Solución de problemas y supervisión desde el lado de HANA en SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Realice la solución problemas y supervisión desde el lado de HANA en SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: bc9fe19d68d19f4b7533d24298b7d5353dfc1215
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110580221"
---
# <a name="monitoring-and-troubleshooting-from-hana-side"></a>Supervisión y solución de problemas en el lado HANA

Con el fin de analizar de forma eficaz los problemas relacionados con SAP HANA en Azure (Instancias grandes), es conveniente averiguar la causa raíz de un problema. SAP ha publicado una gran cantidad de documentación para ayudarle.

Las preguntas más frecuentes relacionadas con el rendimiento de SAP HANA pueden encontrarse en las siguientes notas de SAP:

- [SAP Note #2222200 – FAQ: SAP HANA Network](https://launchpad.support.sap.com/#/notes/2222200) (Nota de SAP 2222200: preguntas más frecuentes sobre la red de SAP HANA)
- [SAP Note #2100040 – FAQ: SAP HANA CPU](https://launchpad.support.sap.com/#/notes/0002100040) (Nota de SAP 2100040: preguntas más frecuentes sobre la CPU de SAP HANA)
- [SAP Note #199997 – FAQ: SAP HANA Memory](https://launchpad.support.sap.com/#/notes/2177064) (Nota de SAP 199997: preguntas más frecuentes sobre la memoria de SAP HANA)
- [SAP Note #200000 – FAQ: SAP HANA Performance Optimization](https://launchpad.support.sap.com/#/notes/2000000) (Nota de SAP 200000: preguntas más frecuentes sobre la optimización del rendimiento de SAP HANA)
- [SAP Note #199930 – FAQ: SAP HANA I/O Analysis](https://launchpad.support.sap.com/#/notes/1999930) (Nota de SAP 199930: preguntas más frecuentes sobre el análisis de E/S de SAP HANA)
- [SAP Note #2177064 – FAQ: SAP HANA Service Restart and Crashes](https://launchpad.support.sap.com/#/notes/2177064) (Nota de SAP 2177064: preguntas más frecuentes sobre el reinicio y los bloqueos del servicio de SAP HANA)

## <a name="sap-hana-alerts"></a>Alertas de SAP HANA

Como primer paso, compruebe los registros de alerta de SAP HANA actuales. En SAP HANA Studio, vaya a **Consola de administración: Alertas: Mostrar: Todas las alertas**. En esta pestaña se muestran todas las alertas de SAP HANA para determinados valores (memoria física libre, uso de CPU, etc.) que se encuentran fuera de los umbrales mínimo y máximo establecidos. De forma predeterminada, las comprobaciones se actualizan automáticamente cada 15 minutos.

![En SAP HANA Studio, vaya a Consola de administración: Alertas: Mostrar: Todas las alertas.](./media/troubleshooting-monitoring/image1-show-alerts.png)

## <a name="cpu"></a>CPU

En el caso de una alerta activada debido a una configuración incorrecta del umbral, una solución es restablecerlo al valor predeterminado o a un valor más razonable.

![Restablecer el umbral al valor predeterminado o a un valor más razonable](./media/troubleshooting-monitoring/image2-cpu-utilization.png)

Las siguientes alertas pueden indicar problemas de recursos de CPU:

- Uso de CPU del host (alerta 5)
- Operación del punto de retorno más reciente (alerta 28)
- Duración del punto de retorno (alerta 54)

Puede observar el uso de CPU elevado en la base de datos de SAP HANA de las siguientes formas:

- La alerta 5 (Uso de CPU del host) se desencadena para el uso de CPU actual o antiguo
- El uso de CPU mostrado en la pantalla de información general

![Uso de CPU mostrado en la pantalla de información general](./media/troubleshooting-monitoring/image3-cpu-usage.png)

El gráfico Carga podría mostrar un uso de CPU elevado o un uso de elevado en el pasado:

![El gráfico Carga podría mostrar un uso de CPU elevado o un uso de elevado en el pasado](./media/troubleshooting-monitoring/image4-load-graph.png)

Una alerta desencadenada debido al uso elevado de CPU puede tener varias causas; entre ellas, la ejecución de ciertas transacciones, la carga de datos, trabajos que no responden, instrucciones SQL de ejecución prolongada y un rendimiento de consulta incorrecto (por ejemplo, con BW en cubos HANA).

Consulte el sitio sobre la [solución de problemas de SAP HANA: causas relacionadas con la CPU y soluciones](https://help.sap.com/saphelp_hanaplatform/helpdata/en/4f/bc915462db406aa2fe92b708b95189/content.htm?frameset=/en/db/6ca50424714af8b370960c04ce667b/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=46&amp;show_children=false) para ver los pasos detallados para la solución de problemas.

## <a name="operating-system"></a>Sistema operativo

Una de las comprobaciones más importantes para SAP HANA en Linux es asegurarse de que las páginas gigantes transparentes están deshabilitadas; consulte la [SAP Note #2131662 – Transparent Huge Pages (THP) on SAP HANA Servers](https://launchpad.support.sap.com/#/notes/2131662) (Nota de SAP #2131662: páginas gigantes transparentes (THP) en servidores de SAP HANA).

- Puede comprobar si las páginas gigantes transparentes están habilitadas con el siguiente comando de Linux: **cat /sys/kernel/mm/transparent\_hugepage/enabled**
- Si _always_ está encerrado entre corchetes como a continuación, significa que las páginas gigantes transparentes están habilitadas: [always] madvise never; si _never_ está encerrado entre corchetes como a continuación, significa que las páginas gigantes transparentes están deshabilitadas: always madvise [never]

El siguiente comando de Linux no debe devolver nada: **rpm -qa | grep ulimit.** Si indica que _ulimit_ está instalado, desinstálelo inmediatamente.

## <a name="memory"></a>Memoria

Puede observar que la cantidad de memoria asignada por la base de datos de SAP HANA es mayor de lo esperado. Las siguientes alertas indican problemas con un uso de memoria elevado:

- Uso de memoria física del host (alerta 1)
- Uso de memoria del servidor de nombres (alerta 12)
- Uso total de la memoria de las tablas de almacenamiento de columnas (alerta 40)
- Uso de memoria de los servicios (alerta 43)
- Uso de memoria del almacenamiento principal de las tablas de almacenamiento de columnas (alerta 45)
- Archivos de volcado de tiempo de ejecución (alerta 46)

Consulte el sitio sobre la [solución de problemas de SAP HANA: problemas de memoria](https://help.sap.com/saphelp_hanaplatform/helpdata/en/db/6ca50424714af8b370960c04ce667b/content.htm?frameset=/en/59/5eaa513dde43758b51378ab3315ebb/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=26&amp;show_children=false) para ver los pasos detallados para la solución de problemas.

## <a name="network"></a>Red

Consulte [SAP Note #2081065 – Troubleshooting SAP HANA Network](https://launchpad.support.sap.com/#/notes/2081065) (Nota de SAP #2081065: solución de problemas de la red de SAP HANA) y realice los pasos para solucionar los problemas de red indicados en ella.

1. Analice el tiempo de ida y vuelta entre el cliente y el servidor.
  A. Ejecute el script SQL [_HANA\_Network\_Clients_](https://launchpad.support.sap.com/#/notes/1969700) _._
  
2. Analice la comunicación entre nodos.
  A. Ejecute el script SQL [_HANA\_Network\_Services_](https://launchpad.support.sap.com/#/notes/1969700) _._

3. Ejecute el comando de Linux **ifconfig** (la salida muestra si se producen las pérdidas de paquetes).
4. Ejecute el comando de Linux **tcpdump**.

Además, utilice la herramienta de código abierto [IPERF](https://iperf.fr/) (u otra similar) para medir el rendimiento de red real de la aplicación.

Consulte el sitio sobre la [solución de problemas de SAP HANA: problemas de conectividad y de rendimiento de red](https://help.sap.com/saphelp_hanaplatform/helpdata/en/a3/ccdff1aedc4720acb24ed8826938b6/content.htm?frameset=/en/dc/6ff98fa36541e997e4c719a632cbd8/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=142&amp;show_children=false) para ver los pasos detallados para la solución de problemas.

## <a name="storage"></a>Storage

Desde la perspectiva del usuario final, una aplicación (o el sistema en su conjunto) se ejecuta lentamente, está inactiva o incluso parece que deja de responder si hay problemas de rendimiento de E/S. En la pestaña **Volúmenes** de SAP HANA Studio, puede ver los volúmenes conectados y los utilizados por cada servicio.

![En la pestaña Volúmenes de SAP HANA Studio, puede ver los volúmenes conectados y los utilizados por cada servicio](./media/troubleshooting-monitoring/image5-volumes-tab-a.png)

Para los volúmenes conectados, en la parte inferior de la pantalla puede ver los detalles de los volúmenes, como los archivos y las estadísticas de E/S.

![Para los volúmenes conectados, en la parte inferior de la pantalla puede ver los detalles de los volúmenes, como los archivos y las estadísticas de E/S](./media/troubleshooting-monitoring/image6-volumes-tab-b.png)

Consulte el sitio sobre la [solución de problemas de SAP HANA: causas principales y soluciones relacionadas con E/S](https://help.sap.com/saphelp_hanaplatform/helpdata/en/dc/6ff98fa36541e997e4c719a632cbd8/content.htm?frameset=/en/47/4cb08a715c42fe9f7cc5efdc599959/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=55&amp;show_children=false) y el sitio sobre la [solución de problemas de SAP HANA: causas principales relacionadas con el disco y soluciones](https://help.sap.com/saphelp_hanaplatform/helpdata/en/47/4cb08a715c42fe9f7cc5efdc599959/content.htm?frameset=/en/44/3e1db4f73d42da859008df4f69e37a/frameset.htm&amp;current_toc=/en/85/d132c3f05e40a2b20c25aa5fd6331b/plain.htm&amp;node_id=53&amp;show_children=false) para ver los pasos detallados para la solución de problemas.

## <a name="diagnostic-tools"></a>Herramientas de diagnóstico

Realice una comprobación del estado de SAP HANA mediante HANA\_Configuration\_Minichecks. Esta herramienta devuelve problemas técnicos potencialmente críticos que ya deberían haber aparecido como alertas en SAP HANA Studio.

Consulte [SAP Note #1969700 – SQL statement collection for SAP HANA](https://launchpad.support.sap.com/#/notes/1969700) (Nota de SAP #1969700: recopilación de instrucciones SQL para SAP HANA) y descargue el archivo SQL Statements.zip adjunto a esa nota. Almacene este archivo .zip en el disco duro local.

En SAP HANA Studio, en la pestaña **System Information** (Información del sistema), haga clic en la columna **Name** (Nombre) y seleccione **Import SQL Statements** (Instrucciones SQL de importación).

![En SAP HANA Studio, en la pestaña System Information (Información del sistema), haga clic en la columna Name (Nombre) y seleccione Import SQL Statements (Instrucciones SQL de importación)](./media/troubleshooting-monitoring/image7-import-statements-a.png)

Seleccione el archivo SQL Statements.zip que está almacenado localmente y se importará una carpeta con las instrucciones SQL correspondientes. En este punto, se pueden ejecutar las numerosas comprobaciones de diagnóstico con estas instrucciones SQL.

Por ejemplo, para probar los requisitos de ancho de banda de la replicación del sistema de SAP HANA, haga clic en la instrucción **Ancho de banda** en **Replicación: Ancho de banda** y seleccione **Abrir** en la consola de SQL.

La instrucción SQL completa se abre, lo que permite cambiar y, después, ejecutar los parámetros de entrada (sección de modificación).

![La instrucción SQL completa se abre, lo que permite cambiar y, después, ejecutar los parámetros de entrada (sección de modificación)](./media/troubleshooting-monitoring/image8-import-statements-b.png)

Otro ejemplo es hacer clic con el botón derecho en las instrucciones en **Replicación: Información general**. Seleccione **Execute** (Ejecutar) en el menú contextual:

![Otro ejemplo es hacer clic con el botón derecho en las instrucciones en Replicación: Información general. Seleccione Execute (Ejecutar) en el menú contextual](./media/troubleshooting-monitoring/image9-import-statements-c.png)

Esto genera información que ayuda a solucionar el problema:

![Esto generará información que ayudará a solucionar el problema](./media/troubleshooting-monitoring/image10-import-statements-d.png)

Haga lo mismo para HANA\_Configuration\_Minichecks y compruebe si hay alguna marca _X_ en la columna _C_ (Crítico).

Ejemplo de salidas:

**HANA\_Configuration\_MiniChecks\_Rev102.01+1** para comprobaciones general de SAP HANA.

![HANA\_Configuration\_MiniChecks\_Rev102.01+1 para comprobaciones general de SAP HANA](./media/troubleshooting-monitoring/image11-configuration-minichecks.png)

**HANA\_Services\_Overview** para información general sobre los servicios de SAP HANA que se están ejecutando.

![HANA\_Services\_Overview para información general sobre los servicios de SAP HANA que se están ejecutando](./media/troubleshooting-monitoring/image12-services-overview.png)

**HANA\_Services\_Statistics** para información sobre los servicio de SAP HANA (CPU, memoria, etc.).

![HANA\_Services\_Statistics para información sobre los servicio de SAP HANA](./media/troubleshooting-monitoring/image13-services-statistics.png)

**HANA\_Configuration\_Overview\_Rev110+** para información general sobre la instancia de SAP HANA.

![HANA\_Configuration\_Overview\_Rev110+ para información general sobre la instancia de SAP HANA](./media/troubleshooting-monitoring/image14-configuration-overview.png)

**HANA\_Configuration\_Parameters\_Rev70+** para comprobar los parámetros de SAP HANA.

![HANA\_Configuration\_Parameters\_Rev70+ para comprobar los parámetros de SAP HANA](./media/troubleshooting-monitoring/image15-configuration-parameters.png)

**Pasos siguientes**

- Consulte [Configuración de alta disponibilidad en SUSE mediante STONITH](ha-setup-with-stonith.md).