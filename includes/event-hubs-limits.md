---
title: include file description: include file services: event-hubs author: spelluru ms.service: event-hubs ms.topic: include ms.date: 03/31/2021 ms.author: spelluru ms.custom: "include file","fasttrack-edit","iot","event-hubs"

---

En las tablas siguientes se enumeran las cuotas y los límites específicos de [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/). Para más información sobre los precios de Event Hubs, consulte los [precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="common-limits-for-all-tiers"></a>Límites comunes para todos los niveles
Los límites siguientes son comunes en todos los niveles. 

| Límite |  Notas | Value |
| --- |  --- | --- |
 Tamaño del nombre de un centro de eventos |- | 256 caracteres |
| Tamaño del nombre de un grupo de consumidores | El protocolo de Kafka no requiere la creación de un grupo de consumidores. | <p>Kafka: 256 caracteres</p><p>AMQP: 50 caracteres |
| Número de destinatarios no de época por grupo de consumidores |- |5 |
| Número de reglas de autorización por espacio de nombres | Las solicitudes posteriores de creación de reglas de autorización se rechazan.|12 |
| Número de llamadas al método GetRuntimeInformation |  - | 50 por segundo | 
| Número de redes virtuales (VNet) | - | 128 | 
| Número de reglas de configuración de IP | - | 128 | 
| Longitud máxima de un nombre de grupo de esquemas | | 50 |  
| Longitud máxima de un nombre de esquemas | | 100 |    
| Tamaño, en bytes, por esquema | | 1 MB |   
| Número de propiedades por grupo de esquemas | | 1024 |
| Tamaño, en bytes, por clave de propiedad del grupo de esquemas | | 256 | 
| Tamaño, en bytes, por valor de propiedad del grupo de esquemas | | 1024 | 

### <a name="basic-vs-standard-vs-dedicated-tiers"></a>Niveles básico, estándar y dedicado
En la tabla siguiente se muestran los límites que pueden ser diferentes para los niveles básico, estándar y dedicado. En la tabla, CU es [unidad de capacidad](../articles/event-hubs/event-hubs-dedicated-overview.md) y TU es [unidad de rendimiento](../articles/event-hubs/event-hubs-faq.yml#what-are-event-hubs-throughput-units-). 

| Límite | Básico | Estándar | Dedicado |
| ----- | ----- | -------- | -------- | 
| Tamaño máximo de la publicación de Event Hubs | 256 KB | 1 MB | 1 MB |
| Número de grupos de consumidores por centro de eventos | 1 | 20 | Sin límite por CU, 1000 por centro de eventos |
| Número de conexiones de AMQP por espacio de nombres | 100 | 5\.000 | 100 000 incluidos y como máximo |
| Período de retención máximo de datos de eventos | 1 día | 1-7 días | 90 días, 10 TB incluidas por TU |
| Número máximo de TU o CU |20 TU | 20 TU | 20 CU |
| Número de particiones por centro de eventos | 32 | 32 | 1024 por centro de eventos
2000 por CU |
| Número de espacios de nombres por suscripción | 100 | 100 | 100 (50 por CU) |
| Número de centros de eventos por espacio de nombres | 10 | 10 | 1000 |
| Eventos de entrada | | Pago por millones de eventos | Se incluye|
| Capturar | N/D | Pago por hora | Se incluye |
| Tamaño del registro de esquema (espacio de nombres), en megabytes | N/D | 25 |  1024 |
| Números de grupos de esquemas en un registro de esquemas o un espacio de nombres | N/D | 1: sin incluir el grupo predeterminado | 1000 |
| Número de versiones del esquema en todos los grupos de esquemas | N/D | 25 | 10000 |

> [!NOTE]
> Los eventos se pueden publicar de forma individual o por lotes. El límite de publicación (según la SKU) se aplica independientemente de si se trata de un evento único o un lote. Se rechazará la publicación de eventos que superen el umbral máximo.

