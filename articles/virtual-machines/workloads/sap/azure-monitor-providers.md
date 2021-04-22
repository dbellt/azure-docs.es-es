---
title: Proveedores de Azure Monitor para soluciones de SAP | Microsoft Docs
description: En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre Azure Monitor para soluciones de SAP.
author: rdeltcheva
ms.service: virtual-machines-sap
ms.topic: article
ms.date: 06/30/2020
ms.author: radeltch
ms.openlocfilehash: 93e97f1f04aea2a31b62b2014a88a5aaa998ed2d
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107376093"
---
# <a name="azure-monitor-for-sap-solutions-providers-preview"></a>Proveedores de Azure Monitor para soluciones de SAP (versión preliminar)

## <a name="overview"></a>Información general  

En el contexto de Azure Monitor para soluciones de SAP, un *tipo de proveedor* hace referencia a un *proveedor* específico. Por ejemplo, *SAP HANA*, que se configura para un componente específico del entorno de SAP, como la base de datos de SAP HANA. Un proveedor contiene la información de conexión del componente correspondiente y ayuda a recopilar datos de telemetría de dicho componente. Un recurso de Azure Monitor para soluciones de SAP (conocido también como recurso de supervisión de SAP) se puede configurar con varios proveedores del mismo tipo de proveedor o con varios proveedores de varios tipos de proveedor.
   
Los clientes pueden optar por configurar distintos tipos de proveedor para habilitar la recopilación de datos del componente correspondiente en su entorno de SAP. Por ejemplo, los clientes pueden configurar un proveedor para el tipo de proveedor de SAP HANA, otro proveedor para el tipo de proveedor de clúster de alta disponibilidad, etc.  

Los clientes también pueden optar por configurar varios proveedores de un tipo de proveedor específico para reutilizar el mismo recurso de monitor de SAP y el grupo administrado asociado. Obtenga más información sobre el grupo de recursos administrados. En la versión preliminar pública, se admiten los siguientes tipos de proveedor:   
- SAP HANA
- Clúster de alta disponibilidad
- Microsoft SQL Server
- SAP NetWeaver

![Proveedores de Azure Monitor para soluciones de SAP](./media/azure-monitor-sap/azure-monitor-providers.png)

Se recomienda a los clientes configurar al menos un proveedor de los tipos de proveedor disponibles en el momento de implementar el recurso de supervisión de SAP. Mediante la configuración de un proveedor, los clientes inician la recopilación de datos del componente correspondiente para el que está configurado el proveedor.   

Si los clientes no configuran ningún proveedor en el momento de implementar el recurso de supervisión de SAP, aunque este se implemente correctamente, no se recopilarán datos de telemetría. Los clientes tienen la opción de agregar proveedores después de la implementación a través de un recurso de supervisión de SAP en Azure Portal. Los clientes pueden agregar o eliminar proveedores del recurso de supervisión de SAP en cualquier momento.

> [!Tip]
> Si quiere que Microsoft implemente un proveedor específico, deje comentarios a través del vínculo que aparece al final de este documento o póngase en contacto con su equipo de cuentas.  

## <a name="provider-type-sap-hana"></a>Tipo de proveedor de SAP HANA

Los clientes pueden configurar uno o varios proveedores del tipo *SAP HANA* para habilitar la recopilación de datos de la base de datos de SAP HANA. El proveedor de SAP HANA se conecta a la base de datos de SAP HANA a través del puerto de SQL, extrae datos de telemetría de la base de datos y los envía al área de trabajo de Log Analytics en la suscripción del cliente. El proveedor de SAP HANA recopila datos cada minuto de la base de datos de SAP HANA.  

En la versión preliminar pública, los clientes deberían ver los datos siguientes con el proveedor de SAP HANA: uso de la infraestructura subyacente, estado del host de SAP HANA, replicación del sistema de SAP HANA y datos de telemetría de la copia de seguridad de SAP HANA. Para configurar el proveedor de SAP HANA, se requiere la dirección IP del host, el número de puerto de SQL de HANA y el nombre de usuario y la contraseña de SYSTEMDB. Se recomienda a los clientes configurar el proveedor de SAP HANA con SYSTEMDB, pero se pueden configurar más proveedores con otros inquilinos de base de datos.

![Proveedores de Azure Monitor para soluciones de SAP: SAP HANA](./media/azure-monitor-sap/azure-monitor-providers-hana.png)

## <a name="provider-type-high-availability-cluster"></a>Tipo de proveedor de clúster de alta disponibilidad
Los clientes pueden configurar uno o varios proveedores del tipo *clúster de alta disponibilidad* para habilitar la recopilación de datos de un clúster de Pacemaker en el entorno de SAP. El proveedor de clúster de alta disponibilidad se conecta a Pacemaker mediante el punto de conexión [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter), extrae datos de telemetría de la base de datos y los envía al área de trabajo de Log Analytics en la suscripción del cliente. El proveedor de clúster de alta disponibilidad recopila datos cada 60 segundos desde Pacemaker.  

En la versión preliminar pública, los clientes deberían ver los datos siguientes con el proveedor de clúster de alta disponibilidad:   
 - Estado del clúster representado como la acumulación del estado de los nodos y de los recursos 
 - [Otros](https://github.com/ClusterLabs/ha_cluster_exporter/blob/master/doc/metrics.md) 

![Proveedores de Azure Monitor para soluciones de SAP: clúster de alta disponibilidad](./media/azure-monitor-sap/azure-monitor-providers-pacemaker-cluster.png)

Para configurar al proveedor de clúster de alta disponibilidad, se deben llevar a cabo dos pasos principales:

1. Instale [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter) en *cada nodo* del clúster de Pacemaker.

   Tiene dos opciones para instalar ha_cluster_exporter:
   
   - Use scripts de Azure Automation para implementar un clúster de alta disponibilidad. Los scripts instalan [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter) en cada nodo del clúster.  
   - Realice una [instalación manual](https://github.com/ClusterLabs/ha_cluster_exporter#manual-clone--build). 

2. Configure un proveedor de clúster de alta disponibilidad para *cada* nodo del clúster de Pacemaker.

   Para configurar al proveedor de clústeres de alta disponibilidad, se requiere la siguiente información:
   
   - **Nombre**. Nombre del proveedor. Debe ser único para esta instancia Azure Monitor para soluciones de SAP.
   - **Punto de conexión de Prometheus**. http\://\<servername or ip address\>:9664/metrics.
   - **SID**. En el caso de los sistemas SAP, use el SID de SAP. En el caso de otros sistemas (por ejemplo, clústeres NFS), use un nombre de tres caracteres para el clúster. El SID debe ser distinto de otros clústeres que se vayan a supervisar.   
   - **Nombre del clúster**. Nombre de clúster que se usó al crear el clúster. El nombre del clúster se puede encontrar en la propiedad `cluster-name` del clúster.
   - **Hostname**. Nombre de host Linux de la máquina virtual.  


## <a name="provider-type-os-linux"></a>Sistema operativo del tipo de proveedor (Linux)
Los clientes pueden configurar uno o varios proveedores del tipo SO (Linux) para habilitar la recopilación de datos de BareMetal o VM Node. El proveedor del sistema operativo (Linux) se conecta a instancias de BareMetal o de VM Node mediante el punto de conexión  [Node_Exporter](https://github.com/prometheus/node_exporter) , extrae datos de telemetría de los nodos y los envía al área de trabajo de Log Analytics de la suscripción del cliente. El proveedor del sistema operativo (Linux) recopila datos cada 60 segundos para la mayoría de las métricas de Nodes. 

En la versión preliminar pública, los clientes deberían ver los datos siguientes con el proveedor del sistema operativo (Linux): 
   - Uso de CPU, uso de CPU por proceso 
   - Uso de disco, lectura y escritura de E/S 
   - Distribución de la memoria, uso de la memoria, uso de la memoria de intercambio 
   - Uso de red, detalles del tráfico de entrada y salida de la red. 

Para configurar un proveedor de sistema operativo (Linux), se deben llevar a cabo dos pasos principales:
1. Instale [Node_Exporter](https://github.com/prometheus/node_exporter) en todas las instancias de BareMetal o VM Node.
   Tiene dos opciones para instalar [Node_exporter](https://github.com/prometheus/node_exporter): 
      - Para la instalación de Automation con Ansible, use [Node_Exporter](https://github.com/prometheus/node_exporter) en las instancias de BareMetal o VM Node para instalar el proveedor del sistema operativo (Linux).  
      - Realice una  [instalación manual](https://prometheus.io/docs/guides/node-exporter/).

2. Configure un proveedor de sistema operativo (Linux) para cada instancia de BareMetal o VM Node en su entorno. 
   Para configurar al proveedor de sistema operativo (Linux), se requiere la siguiente información: 
      - Name (Nombre). Nombre del proveedor. Debe ser único para esta instancia Azure Monitor para soluciones de SAP. 
      - Punto de conexión de Node Exporter. Normalmente, http://<servername or ip address>:9100/metrics 

> [!NOTE]
> 9100 es un puerto expuesto para el punto de conexión de Node_Exporter.

> [!Warning]
> Asegúrese de que Node Exporter sigue ejecutándose después del reinicio del nodo. 


## <a name="provider-type-microsoft-sql-server"></a>Tipo de proveedor de Microsoft SQL Server

Los clientes pueden configurar uno o varios proveedores del tipo *Microsoft SQL Server* para habilitar la recopilación de datos de [SQL Server en Virtual Machines](https://azure.microsoft.com/services/virtual-machines/sql-server/). El proveedor de SQL Server se conecta a Microsoft SQL Server a través del puerto de SQL, extrae datos de telemetría de la base de datos y los envía al área de trabajo de Log Analytics en la suscripción del cliente. Es necesario configurar SQL Server para la autenticación de SQL y crear un inicio de sesión de SQL Server, con la base de datos de SAP como base de datos predeterminada para el proveedor. El proveedor de SQL Server recopila datos de SQL Server en un intervalo que va de cada 60 segundos a cada hora.  

En la versión preliminar pública, los clientes deberían ver los datos siguientes con el proveedor de SQL Server: uso de la infraestructura subyacente, principales instrucciones SQL, tabla de mayor tamaño, problemas detectados en los registros de errores de SQL Server, procesos de bloqueo, etc.  

Para configurar el proveedor de Microsoft SQL Server, se necesitan el identificador del sistema SAP, la dirección IP del host, el número del puerto de SQL Server y el nombre de inicio de sesión y la contraseña de SQL Server.

![Proveedores de Azure Monitor para soluciones de SAP: SQL](./media/azure-monitor-sap/azure-monitor-providers-sql.png)

## <a name="provider-type-sap-netweaver"></a>Tipo de proveedor: SAP NetWeaver

Los clientes pueden configurar uno o varios proveedores del tipo SAP NetWeaver para habilitar la recopilación de datos de la capa SAP NetWeaver. El proveedor de AMS NetWeaver aprovecha la interfaz del [servicio web SAPControl](https://www.sap.com/documents/2016/09/0a40e60d-8b7c-0010-82c7-eda71af511fa.html) existente para recuperar la información de telemetría adecuada.

A continuación, se muestran los métodos web SOAP estándar de fábrica que AMS invoca para la versión actual.
|método web|    ABAP|   Java|   Métricas|
|--|--|--|--|
|GetSystemInstanceList| X|  X|  Disponibilidad de instancias, servidor de mensajes, puerta de enlace, ICM, disponibilidad de ABAP|
|GetProcessList|    X|  X|  Si la lista de instancias es ROJA, podemos ver qué proceso hace que el servidor aparezca en ROJO|
|GetQueueStatistic| X|  X|  Estadísticas de cola (DIA/BATCH/UPD)|
|ABAPGetWPTable|    X|   -| Uso de procesos de trabajo|
|EnqGetStatistic|   X   |X  |Bloqueos|

En la versión preliminar pública, los clientes deberían ver los datos siguientes con el proveedor de SAP NetWeaver: 
- Disponibilidad del sistema y de la instancia
- Uso de procesos de trabajo
- Uso de la cola
- Estadísticas de bloqueos puestos en cola.

![imagen](https://user-images.githubusercontent.com/75772258/114581825-a9f2eb00-9c9d-11eb-8e6f-79cee7c5093f.png)

## <a name="next-steps"></a>Pasos siguientes

- Cree su primer recurso de Azure Monitor para soluciones de SAP.
- ¿Tiene alguna pregunta sobre Azure Monitor para soluciones de SAP? Consulte la sección de [preguntas más frecuentes](./azure-monitor-faq.md).
