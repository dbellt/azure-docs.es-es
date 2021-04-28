---
title: Controles de seguridad para el servicio Azure Spring Cloud
description: Use los controles de seguridad integrados en el servicio Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 04/23/2020
ms.custom: devx-track-java
ms.openlocfilehash: faa278b60f3b120fc6f1aad14cd6f8b6dbfb1e21
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129196"
---
# <a name="security-controls-for-azure-spring-cloud-service"></a>Controles de seguridad para el servicio Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java ✔️ C#

Los controles de seguridad están integrados en el servicio Azure Spring Cloud.

Un control de seguridad es una cualidad o característica de un servicio de Azure que ayuda a dicho servicio a prevenir y detectar vulnerabilidades de seguridad, así como a responder a ellas.  En cada control, utilizamos *Sí* o *No* para indicar si actualmente está disponible para el servicio.  Usamos *N/D* para un control que no se pueda aplicar al servicio. 

**Controles de seguridad para la protección de datos**

| Control de seguridad | Sí/No | Notas | Documentación |
|:-------------|:-------|:-------------------------------|:----------------------|
| Cifrado del lado servidor en reposo: Claves administradas por Microsoft | Sí | El origen y los artefactos cargados por el usuario, los valores del servidor de configuración, la configuración de la aplicación y los datos en el almacenamiento persistente se almacenan en Azure Storage, que cifra automáticamente el contenido en reposo.<br><br>La memoria caché del servidor de configuración, los archivos binarios en tiempo de ejecución compilados a partir del código fuente cargado y los registros de aplicación durante la vigencia de la aplicación se guardan en un disco administrado de Azure que cifra automáticamente el contenido en reposo.<br><br>Las imágenes de contenedor compiladas a partir del código fuente cargado por el usuario se guardan en Azure Container Registry, que cifra automáticamente el contenido de la imagen en reposo. | [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)<br><br>[Cifrado del lado servidor de Azure Managed Disks](../virtual-machines/disk-encryption.md)<br><br>[Almacenamiento de imágenes en Azure Container Registry](../container-registry/container-registry-storage.md) |
| Cifrado en tránsito | Sí | Los puntos de conexión públicos de la aplicación de usuario usan HTTPS para el tráfico de entrada de forma predeterminada. |  |
| Llamadas a API cifradas | Sí | Las llamadas de administración para configurar el servicio Azure Spring Cloud se producen mediante llamadas de Azure Resource Manager sobre HTTPS. | [Azure Resource Manager](../azure-resource-manager/index.yml) |

**Controles de seguridad de acceso a la red**

| Control de seguridad | Sí/No | Notas | Documentación |
|:-------------|:-------|:-------------------------------|:----------------------|
| Etiqueta de servicio | Sí | Use la etiqueta de servicio **AzureSpringCloud** para definir controles de acceso de red de salida en [grupos de seguridad de red](../virtual-network/network-security-groups-overview.md#security-rules) o [Azure Firewall](../firewall/service-tags.md) para permitir el tráfico a las aplicaciones de Azure Spring Cloud. | [Etiquetas de servicio](../virtual-network/service-tags-overview.md) |

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Implementación de la primera aplicación de Azure Spring Cloud](./quickstart.md)