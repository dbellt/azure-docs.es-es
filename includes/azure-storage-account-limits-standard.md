---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/30/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: ef9efe389894af7c792e980922ca422e9d05929b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026185"
---
En la tabla siguiente se describen los límites predeterminados para las cuentas de blob en bloques, Blob Storage y de uso general v1 y v2 de Azure. El límite de *entrada* hace referencia a todos los datos que se envían a una cuenta de almacenamiento. El límite de *salida* hace referencia a todos los datos que se reciben de una cuenta de almacenamiento.

> [!NOTE]
> Puede solicitar unos límites de capacidad y de entrada mayores. Para solicitar un aumento, póngase en contacto con [Soporte técnico de Azure](https://azure.microsoft.com/support/faq/).

| Resource | Límite |
| --- | --- |
| Número de cuentas de almacenamiento por región y suscripción, incluidas las cuentas de almacenamiento Estándar y Premium.| 250 |
| Capacidad máxima de la cuenta de almacenamiento | 5 PiB <sup>1</sup>|
| Número máximo de contenedores de blobs, blobs, recursos compartidos de archivos, tablas, colas, entidades o mensajes por cuenta de almacenamiento | Sin límite |
| Tasa de solicitud total<sup>1</sup> por cuenta de almacenamiento | 20 000 solicitudes por segundo |
| Entrada máxima<sup>1</sup> por cuenta de almacenamiento (regiones de Estados Unidos y Europa) | 10 Gbps |
| Entrada máxima<sup>1</sup> por cuenta de almacenamiento (regiones distintas de Estados Unidos y Europa) | 5 Gbps si RA-GRS o GRS está habilitado, 10 Gbps en el caso de LRS o ZRS<sup>2</sup> |
| Salida máxima para las cuentas de uso general v2 y de Blob Storage (todas las regiones) | 50 Gbps |
| Salida máxima para cuentas de almacenamiento de uso general v1 (regiones de EE. UU.) | 20 Gbps si RA-GRS o GRS están habilitado, 30 Gbps en el caso de LRS o ZRS<sup>2</sup> |
| Salida máxima para cuentas de almacenamiento de uso general v1 (regiones no de EE. UU.) | 10 Gbps si RA-GRS o GRS está habilitado, 15 Gbps en el caso de LRS o ZRS<sup>2</sup> |
| Número máximo de reglas de red virtual por cuenta de almacenamiento | 200 |
| Número máximo de reglas de dirección IP por cuenta de almacenamiento | 200 |

<sup>1</sup>Las cuentas estándar de Azure Storage admiten límites de capacidad y límites de entrada por solicitud superiores. Para solicitar un aumento en los límites de cuenta, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/faq/).

<sup>2</sup> Si la cuenta de almacenamiento tiene habilitado el acceso de lectura con almacenamiento con redundancia geográfica (RA-GRS) o el almacenamiento con redundancia de zona geográfica (RA-GZRS), los destinos de salida de la ubicación secundaria son idénticos a los de la ubicación principal. Para más información, consulte [Replicación de Azure Storage](../articles/storage/common/storage-redundancy.md).

> [!NOTE]
> Microsoft recomienda usar una cuenta de almacenamiento de uso general v2 en la mayoría de los escenarios. Puede actualizar fácilmente una cuenta de Azure Blob Storage o de uso general v1 a una cuenta de uso general v2 sin tiempo de inactividad y sin la necesidad de copiar datos. Para más información, consulte [Actualización a una cuenta de almacenamiento de uso general v2](../articles/storage/common/storage-account-upgrade.md).

Todas las cuentas de almacenamiento se ejecutan en una topología de red plana, independientemente del momento en que se hayan creado. Para obtener más información acerca de la arquitectura de red plana de Azure Storage y de la escalabilidad, vea [Microsoft Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](/archive/blogs/hanuk/windows-azures-flat-network-storage-to-enable-higher-scalability-targets) (Microsoft Azure Storage: Un servicio de almacenamiento en nube altamente disponible de gran coherencia).