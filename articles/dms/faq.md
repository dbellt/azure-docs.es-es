---
title: 'Preguntas frecuentes: Azure Database Migration Service'
description: Preguntas más frecuentes sobre cómo usar Azure Database Migration Service para realizar migraciones de base de datos.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: 29240f371d39ef2d3234e3207cc91768478bf3ff
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107748510"
---
# <a name="faq-about-using-azure-database-migration-service"></a>Preguntas más frecuentes sobre el uso de Azure Database Migration Service

En este artículo aparecen las preguntas más comunes sobre el uso de Azure Database Migration Service junto con las respuestas relacionadas.

## <a name="overview"></a>Información general

**P. ¿Qué es Azure Database Migration Service?**
Azure Database Migration Service es un servicio totalmente administrado diseñado para permitir migraciones completas desde varios orígenes de base de datos hasta las plataformas de datos de Azure con un tiempo de inactividad mínimo. El servicio está actualmente disponible con carácter general, con trabajos de desarrollo continuados centrados en:

* Confiabilidad y rendimiento
* Adición iterativa de pares de origen-destino
* Inversión continuada en migraciones libres de problemas

**P. ¿Qué pares origen/destino admite actualmente Azure Database Migration Service?**
Actualmente, el servicio admite una variedad de pares origen/destino o escenarios de migración. Para obtener una lista completa del estado de cada escenario de migración disponible, consulte el artículo [Estado de los escenarios de migración que admite Azure Database Migration Service](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/dms/resource-scenario-status.md).

**P. ¿Qué versiones de SQL Server admite Azure Database Migration Service como origen?**
Al realizar la migración desde SQL Server, los orígenes admitidos de Azure Database Migration Service son las versiones desde SQL Server 2005 hasta SQL Server 2019.

**P: Al usar Azure Database Migration Service, ¿cuál es la diferencia entre una migración sin conexión y en línea?**
Puede usar Azure Database Migration Service para realizar migraciones sin conexión o en línea. Con una migración *sin conexión*, el tiempo de inactividad de la aplicación se inicia cuando comienza la migración. Con una migración *en línea*, el tiempo de inactividad se limita al momento de la migración al final del proceso. Se recomienda que pruebe una migración sin conexión para determinar si el tiempo de inactividad es aceptable; si no es así, realice una migración en línea.

> [!NOTE]
> El uso de Azure Database Migration Service para realizar una migración en línea requiere la creación de una instancia basada en el plan de tarifa Premium. Para más información, consulte la página de [precios](https://azure.microsoft.com/pricing/details/database-migration/) de Azure Database Migration Service.

**P. ¿Cómo se compara Azure Database Migration Service con otras herramientas de migración de bases de datos de Microsoft, como Database Migration Assistant (DMA) o SQL Server Migration Assistant (SSMA)?**
Azure Database Migration Service es el método preferido para la migración de bases de datos a Microsoft Azure a escala. Para obtener más detalles sobre cómo se compara Azure Database Migration Service con otras herramientas de migración de bases de datos de Microsoft y para consultar recomendaciones sobre cómo usar el servicio en distintos escenarios, vea esta entrada de blog sobre [cómo diferenciar los servicios y las herramientas de migración de bases de datos de Microsoft](https://techcommunity.microsoft.com/t5/microsoft-data-migration/differentiating-microsoft-s-database-migration-tools-and/ba-p/368529).

**P. ¿Cómo se compara Azure Database Migration Service con la oferta de Azure Migrate?**
Azure Migrate ayuda con la migración de máquinas virtuales locales a IaaS de Azure. El servicio evalúa la idoneidad de la migración y el ajuste de tamaño basado en el rendimiento, y proporciona estimaciones del costo que supone la ejecución de máquinas virtuales locales en Azure. Azure Migrate es útil para las migraciones mediante lift-and-shift de cargas de trabajo basadas en VM locales a máquinas virtuales de IaaS de Azure. En cambio, a diferencia de Azure Database Migration Service, Azure Migrate no es una oferta de servicios de migración de bases de datos especializada para las plataformas de bases de datos relacionales de PaaS de Azure, como Azure SQL Database o Instancia administrada de Azure SQL.

**P. ¿Database Migration Service almacena los datos de los clientes?**
No. Database Migration Service no almacena los datos de los clientes.

## <a name="setup"></a>Configurar

**P. ¿Cuáles son los requisitos previos para usar Azure Database Migration Service?**
Hay varios requisitos previos necesarios para garantizar que Azure Database Migration Service se ejecute sin problemas al realizar migraciones de bases de datos. Algunos de los requisitos previos se aplican en todos los escenarios (pares origen-destino) compatibles con el servicio, mientras que otros son exclusivos para un escenario específico.

Los requisitos de Azure Database Migration Service que son comunes en todos los escenarios de migración compatibles incluyen la necesidad de:

* Cree una instancia de Azure Virtual Network para Azure Database Migration Service mediante el modelo de implementación de Azure Resource Manager, que proporciona conectividad de sitio a sitio a los servidores de origen local mediante [ExpressRoute](../expressroute/expressroute-introduction.md) o [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloqueen el puerto 443 para ServiceTags de ServiceBus, Storage y Azure Monitor. Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Cuando se usa un dispositivo de firewall frente a las bases de datos de origen, puede que sea necesario agregar reglas de firewall para permitir que Azure Database Migration Service acceda a las bases de datos de origen para realizar la migración.

Para obtener una lista de todos los requisitos previos que se necesitan para completar los escenarios de migración específicos mediante Azure Database Migration Service, consulte los tutoriales relacionados en la [documentación](./dms-overview.md) de Azure Database Migration Service que se encuentra en docs.microsoft.com.

**P. ¿Cómo busco la dirección IP de Azure Database Migration Service para poder crear una lista de las reglas de firewall permitidas que se usan para acceder a mi base de datos de origen para la migración?**
Puede que tenga que agregar reglas de firewall para permitir que Azure Database Migration Service acceda a la base de datos de origen para la migración. La dirección IP del servicio es dinámica, pero si se usa ExpressRoute, esta dirección la asigna la red corporativa de manera privada. La manera más sencilla de identificar la dirección IP adecuada es buscar en el mismo grupo de recursos que el recurso de Azure Database Migration Service aprovisionado para buscar la interfaz de red asociada. Habitualmente, el nombre del recurso de interfaz de red comienza con el prefijo NIC y continúa con una secuencia única de caracteres y números, por ejemplo, NIC-jj6tnztnmarpsskr82rbndyp. Al seleccionar este recurso de interfaz de red, puede ver la dirección IP que se debe incluir en la lista de permitidos en la página de información general de los recursos de Azure Portal.

Puede que también tenga que incluir en la lista de permitidos el origen de puerto en que SQL Server escucha. De manera predeterminada, se trata del puerto 1433, pero la instancia de SQL Server de origen también puede estar configurada para escuchar en otros puertos. En este caso, debe incluir también estos puertos en la lista de permitidos. Puede determinar el puerto en que SQL Server escuchar mediante una consulta de vista de administración dinámica:

```sql
    SELECT DISTINCT
        local_tcp_port
    FROM sys.dm_exec_connections
    WHERE local_tcp_port IS NOT NULL
```

También puede determinar el puerto en que SQL Server escucha mediante una consulta del registro de errores de SQL Server:

```sql
    USE master
    GO
    xp_readerrorlog 0, 1, N'Server is listening on'
    GO
```

**P. ¿Cómo se configura una instancia de Microsoft Azure Virtual Network?**
Si bien existen varios tutoriales de Microsoft que pueden guiarlo en el proceso de configurar una red virtual, la documentación oficial aparece en el artículo [Azure Virtual Network](../virtual-network/virtual-networks-overview.md).

## <a name="usage"></a>Uso

**P. ¿Cuál es el resumen de los pasos que se necesitan para usar Azure Database Migration Service para realizar una migración de bases de datos?**
Durante una migración de base de datos sencilla típica, debe:

1. Crear bases de datos de destino.
2. Evaluar las bases de datos de origen.
    * En el caso de las migraciones homogéneas, evalúe las bases de datos existentes mediante [DMA](https://www.microsoft.com/download/details.aspx?id=53595).
    * En el caso de las migraciones heterogéneas (desde orígenes de competición), evalúe las bases de datos existentes con [SSMA](/sql/ssma/sql-server-migration-assistant). También se usa SSMA para convertir los objetos de base de datos y migrar el esquema a la plataforma de destino.
3. Crear una instancia de Azure Database Migration Service.
4. Crear un proyecto de migración al especificar las bases de datos de origen, las bases de datos de destino y las tablas que se migrarán.
5. Iniciar toda la carga.
6. Seleccionar la validación posterior.
7. Realizar un cambio manual del entorno de producción a la nueva base de datos basada en la nube.

## <a name="troubleshooting-and-optimization"></a>Solución de problemas y optimización

**P. Estoy configurando un proyecto de migración en DMS y tengo dificultades para conectarme a la base de datos de origen. ¿qué debo hacer?**
Si tiene problemas para conectarse al sistema de la base de datos de origen mientras trabaja en la migración, cree una máquina virtual en la misma subred de la red virtual con la que ha configurado la instancia de DMS. En la máquina virtual, debería poder ejecutar una prueba de conexión (por ejemplo, usar un archivo UDL para probar una conexión con SQL Server o descargar Robo 3T para probar las conexiones de MongoDB). Si la prueba de conexión se realiza correctamente, no debería tener ningún problema al conectarse a la base de datos de origen. Si la prueba de conexión no se realiza correctamente, póngase en contacto con el administrador de la red.

**P. ¿Por qué mi instancia de Azure Database Migration Service no está disponible o está detenida?**
Si el usuario detiene explícitamente la instancia de Azure Database Migration Service (DMS) o si el servicio está inactivo durante un período de 24 horas, este estará en pausa o detenido automáticamente. En cada caso, el servicio no estará disponible y se encontrará en estado detenido.  Para reanudar las migraciones activas, reinicie el servicio.

**P. ¿Existen recomendaciones sobre cómo optimizar el rendimiento de Azure Database Migration Service?**
Hay algunas opciones que puede realizar para acelerar la migración de las bases de datos mediante el servicio:

* Use el plan de tarifa de uso general de varias CPU cuando cree la instancia de servicio para permitir que el servicio aproveche las diversas CPU virtuales para la paralelización y realizar una transferencia de datos más rápida.
* De manera temporal, escale verticalmente la instancia de destino de su base de datos de Azure SQL a la SKU de nivel Premium durante la operación de migración de datos para minimizar la limitación de Azure SQL Database que puede afectar las actividades de transferencia de datos al usar las SKU de nivel inferior.

## <a name="next-steps"></a>Pasos siguientes

Para información general sobre Azure Database Migration Service y la disponibilidad regional, consulte el artículo [¿Qué es la versión preliminar de Azure Database Migration Service?](dms-overview.md)
