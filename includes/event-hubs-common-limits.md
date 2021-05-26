---
title: include file description: include file services: event-hubs author: spelluru ms.service: event-hubs ms.topic: include ms.date: 05/10/2021 ms.author: spelluru ms.custom: "include file","fasttrack-edit","iot","event-hubs"

---

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

