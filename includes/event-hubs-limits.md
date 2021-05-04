---
title: include file description: include file services: event-hubs author: spelluru ms.service: event-hubs ms.topic: include ms.date: 03/31/2021 ms.author: spelluru ms.custom: "include file","fasttrack-edit","iot","event-hubs"

---

En las tablas siguientes se enumeran las cuotas y los límites específicos de [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/). Para más información sobre los precios de Event Hubs, consulte los [precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="common-limits-for-all-tiers"></a>Límites comunes para todos los niveles
Los límites siguientes son comunes en todos los niveles. 

| Límite |  Notas | Value |
| --- |  --- | --- |
| Número de espacios de nombres de Event Hubs por suscripción | Los espacios de nombres de Service Bus están incluidos en este límite. |100 |
| Número de centros de eventos por espacio de nombres | Las solicitudes posteriores para la creación de un nuevo centro de eventos se rechazan. |10 |
| Tamaño del nombre de un centro de eventos |- | 256 caracteres |
| Tamaño del nombre de un grupo de consumidores | El protocolo de Kafka no requiere la creación de un grupo de consumidores. | <p>Kafka: 256 caracteres</p><p>AMQP: 50 caracteres |
| Número de destinatarios no de época por grupo de consumidores |- |5 |
| Número de reglas de autorización por espacio de nombres | Las solicitudes posteriores de creación de reglas de autorización se rechazan.|12 |
| Número de llamadas al método GetRuntimeInformation |  - | 50 por segundo | 
| Número de redes virtuales (VNet) | - | 128 | 
| Número de reglas de configuración de IP | - | 128 | 

### <a name="basic-vs-standard-tiers"></a>Niveles Básico y Estándar
En la tabla siguiente se muestran los límites que pueden ser diferentes para los niveles básico y estándar. 

| Límite | Notas | Básico | Estándar |
|---|---|--|---|
| Tamaño máximo de la publicación de Event Hubs| &nbsp; | 256 KB | 1 MB |
| Número de grupos de consumidores por centro de eventos | &nbsp; |1 |20 |
| Número de conexiones de AMQP por espacio de nombres | Las solicitudes posteriores de conexiones adicionales se rechazan y el código que llama recibe una excepción. |100 |5\.000|
| Período de retención máximo de datos de eventos | &nbsp; |1 día |1-7 días |
| Unidades de rendimiento máximo |Si se supera este límite, los datos se limitan y se genera un [excepción por servidor ocupado](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception). Para solicitar un número mayor de unidades de procesamiento para el nivel Estándar rellene una [solicitud de soporte técnico](../articles/azure-portal/supportability/how-to-create-azure-support-request.md). Las [unidades de rendimiento adicionales](../articles/event-hubs/event-hubs-auto-inflate.md) se encuentran disponibles en bloques de 20 y están sujetas a un compromiso de compra. |20 | 20 | 
| Número de particiones por centro de eventos | |32 | 32 | 

> [!NOTE]
>
> Los eventos se pueden publicar de forma individual o por lotes. 
> El límite de publicación (según la SKU) se aplica independientemente de si se trata de un evento único o un lote. Se rechazará la publicación de eventos que superen el umbral máximo.

### <a name="dedicated-tier-vs-standard-tier"></a>Nivel dedicado frente a nivel estándar
La oferta de Event Hubs dedicado se factura aplicando una tarifa mensual fija con un uso mínimo de 4 horas. El nivel Dedicado ofrece todas las características del plan Estándar, pero con una capacidad de escalado de nivel empresarial para aquellos clientes que tienen cargas de trabajo muy exigentes. 

Consulte este [documento](../articles/event-hubs/event-hubs-dedicated-cluster-create-portal.md), en el que se explica cómo crear un clúster de Event Hubs dedicado mediante Azure Portal.

| Característica | Estándar | Dedicado |
| --- |:---|:---|
| Ancho de banda | 20 TU (hasta 40 TU) | 20 CU |
| Espacios de nombres |  100 por suscripción | 50 por unidad de capacidad (100 por suscripción) |
| Event Hubs |  10 por espacio de nombres | 1000 por espacio de nombres |
| Eventos de entrada | Pago por millones de eventos | Se incluye |
| Tamaño de los mensajes | 1 millón de bytes | 1 millón de bytes |
| Particiones | 32 por centro de eventos | 1024 por centro de eventos<br/>2000 por CU |
| Grupos de consumidores | 20 por centro de eventos | Sin límite por CU, 1000 por centro de eventos |
| Conexiones asincrónicas | 1000 incluidas, 5000 como máximo | 100 000 incluidos y como máximo |
| Retención de mensajes | 7 días, 84 GB incluidas por TU | 90 días, 10 TB incluidas por TU |
| Capturar | Pago por hora | Se incluye |


### <a name="schema-registry-limitations"></a>Limitaciones del registro de esquema

#### <a name="limits-that-are-the-same-for-standard-and-dedicated-tiers"></a>Límites que son iguales para los niveles estándar y dedicado 
| Característica | Límite | 
|---|---|
| Longitud máxima de un nombre de grupo de esquemas | 50 |  
| Longitud máxima de un nombre de esquemas | 100 |    
| Tamaño, en bytes, por esquema | 1 MB |   
| Número de propiedades por grupo de esquemas | 1024 |
| Tamaño, en bytes, por clave de propiedad del grupo | 256 | 
| Tamaño, en bytes, por valor de propiedad del grupo | 1024 | 


#### <a name="limits-that-are-different-for-standard-and-dedicated-tiers"></a>Límites que son distintos para los niveles estándar y dedicado 

| Límite | Estándar | Dedicado | 
|---|---|--|
| Tamaño del registro de esquema (espacio de nombres), en megabytes | 25 |  1024 |
| Números de grupos de esquemas en un registro de esquemas o un espacio de nombres | 1: sin incluir el grupo predeterminado | 1000 |
| Número de versiones del esquema en todos los grupos de esquemas | 25 | 10000 |
