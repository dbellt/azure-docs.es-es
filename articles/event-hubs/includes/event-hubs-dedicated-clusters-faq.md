---
title: Archivo de inclusión
description: archivo de inclusión
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 10/23/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 0335481566ae3f28ac0f1e6bddce7050a65e7dc2
ms.sourcegitcommit: 5163ebd8257281e7e724c072f169d4165441c326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/21/2021
ms.locfileid: "112412972"
---
### <a name="what-can-i-achieve-with-a-cluster"></a>¿Qué puedo conseguir con un clúster?

En un clúster de Event Hubs, la cantidad de datos que puede ingerir y transmitir depende de varios factores, como los productores, los consumidores, la velocidad a la que se ingiere y procesa, etc. 

En la tabla siguiente se muestran los resultados del banco de pruebas que se han obtenido durante las pruebas:

| Forma de la carga | Receptores | Ancho de banda de entrada| Mensajes de entrada | Ancho de banda de salida | Mensajes de salida | TU totales | TU por CU |
| ------------- | --------- | ---------------- | ------------------ | ----------------- | ------------------- | --------- | ---------- |
| Lotes de 100 x 1 KB | 2 | 400 MB/s | 400 000 mensajes/s | 800 MB/s | 800 000 mensajes/s | 400 TU | 100 TU | 
| Lotes de 10 x 10 KB | 2 | 666 MB/s | 66 600 mensajes/s | 1,33 GB/s | 133 000 mensajes/s | 666 TU | 166 TU |
| Lotes de 6 x 32 KB | 1 | 1,05 GB/s | 34 000 mensajes/s | 1,05 GB/s | 34 000 mensajes/s | 1000 TU | 250 TU |

En las pruebas, se usaron los siguientes criterios:

- Se usó un clúster de Event Hubs dedicado con cuatro unidades de capacidad (CU). 
- El centro de eventos usado para la ingesta tenía 200 particiones. 
- Dos aplicaciones receptoras recibieron los datos ingeridos de todas las particiones.

### <a name="can-i-scale-updown-my-cluster"></a>¿Se puede escalar o reducir verticalmente el clúster?

Una vez creados los clústeres, se factura un mínimo de 4 horas de uso. En la versión preliminar de la experiencia de autoservicio, puede enviar una [solicitud de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) al equipo de Event Hubs en **Técnico** > **Cuota** > **Request to Scale Up or Scale Down Dedicated Cluster (Solicitud para escalar o reducir verticalmente el clúster dedicado)** para escalar o reducir verticalmente el clúster. La solicitud para reducir verticalmente el clúster puede tardar hasta 7 días en completarse. 

### <a name="how-does-geo-dr-work-with-my-cluster"></a>¿Cómo funciona la recuperación ante desastres con localización geográfica en el clúster?

Puede emparejar geográficamente un espacio de nombres de un clúster del nivel Dedicado con otro espacio de nombres de un clúster del mismo nivel. No es recomendable emparejar un espacio de nombres del nivel Dedicado con otro de la oferta Estándar, ya que el límite de rendimiento será incompatible y se producirán errores. 

### <a name="can-i-migrate-my-standard-namespaces-to-belong-to-a-dedicated-tier-cluster"></a>¿Puedo migrar espacios de nombres del nivel Estándar para que pertenezcan a un clúster del nivel Dedicado?
Actualmente, no se admite ningún proceso automático para migrar los datos de los centros de eventos de un espacio de nombres Estándar a otro Dedicado. 
