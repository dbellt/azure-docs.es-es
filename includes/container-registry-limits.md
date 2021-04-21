---
title: archivo de inclusión
description: archivo de inclusión
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 06/18/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: b4701260a7d8da030f9f3019060aaa83e7a3a483
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "104803615"
---
| Resource | Básico | Estándar | Premium |
|---|---|---|---|
| Se incluye almacenamiento<sup>1</sup> (GiB) | 10 | 100 | 500 |
| Límite de almacenamiento (TiB) | 20| 20 | 20 |
| Tamaño máximo de la capa de imagen (GiB) | 200 | 200 | 200 |
| Operaciones de lectura por minuto<sup>2, 3</sup> | 1,000 | 3,000 | 10 000 |
| Operaciones de escritura por minuto<sup>2, 4</sup> | 100 | 500 | 2\.000 |
| Ancho de banda de descarga <sup>2</sup> (Mbps) | 30 | 60 | 100 |
| Ancho de banda de carga <sup>2</sup> (Mbps) | 10 | 20 | 50 |
| webhooks | 2 | 10 | 500 |
| Replicación geográfica | N/D | N/D | [Compatible][geo-replication] |
| Zonas de disponibilidad | N/D | N/D | [Versión preliminar][zones] |
| Confianza de contenido | N/D | N/D | [Compatible][content-trust] |
| Vínculo privado con puntos de conexión privados | N/D | N/D | [Compatible][plink] |
| &bull; Puntos de conexión privados | N/D | N/D | 10 |
| Reglas de red IP pública | N/D | N/D | 100 |
| Acceso a red virtual del punto de conexión de servicio | N/D | N/D | [Versión preliminar][vnet] |
| Claves administradas por el cliente | N/D | N/D | [Compatible][cmk] |
| Permisos de ámbito de repositorio | N/D | N/D | [Versión preliminar][token]|
| &bull; Tokens | N/D | N/D | 20.000 |
| &bull; Asignaciones de ámbito | N/D | N/D | 20.000 |
| &bull; Repositorios por asignación de ámbito | N/D | N/D | 500 |


<sup>1</sup> Almacenamiento incluido en la tarifa diaria de cada nivel. Se puede usar más almacenamiento, hasta el límite que imponga el almacenamiento del registro, con una tarifa diaria por GiB adicional. Para más información, consulte [Precios de Azure Container Registry][pricing]. Si necesita más almacenamiento que el que proporciona el límite de almacenamiento del registro, póngase en contacto con el soporte técnico de Azure.

<sup>2</sup>*ReadOps*, *WriteOps* y *ancho de banda* son estimaciones mínimas. Azure Container Registry se esfuerza por mejorar el rendimiento adaptado a su uso.

<sup>3</sup>[docker pull](https://docs.docker.com/registry/spec/api/#pulling-an-image) se traduce en varias operaciones de lectura en función del número de capas de la imagen, además de la recuperación del manifiesto.

<sup>4</sup>[docker push](https://docs.docker.com/registry/spec/api/#pushing-an-image) se traduce en varias operaciones de escritura, en función del número de capas que se deben insertar. Un elemento `docker push` incluye *operaciones de lectura* para recuperar un manifiesto para una imagen existente.

<!-- LINKS - External -->
[pricing]: https://azure.microsoft.com/pricing/details/container-registry/

<!-- LINKS - Internal -->
[geo-replication]: ../articles/container-registry/container-registry-geo-replication.md
[content-trust]: ../articles/container-registry/container-registry-content-trust.md
[vnet]: ../articles/container-registry/container-registry-vnet.md
[plink]: ../articles/container-registry/container-registry-private-link.md
[cmk]: ../articles/container-registry/container-registry-customer-managed-keys.md
[token]: ../articles/container-registry/container-registry-repository-scoped-permissions.md
[zones]: ../articles/container-registry/zone-redundancy.md
