---
title: Automatización de implementaciones del portal para desarrolladores
titleSuffix: Azure API Management
description: Obtenga información sobre cómo migrar automáticamente el contenido del portal para desarrolladores autohospedado entre dos servicios API Management.
author: erikadoyle
ms.author: apimpm
ms.date: 04/15/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: e189a9339f6ca3bc81148b86206ddd052392074f
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741429"
---
# <a name="automate-developer-portal-deployments"></a>Automatización de implementaciones del portal para desarrolladores

El portal para desarrolladores de API Management admite el acceso mediante programación al contenido. Le permite importar o exportar datos desde un servicio API Management a través de la [API REST de administración de contenido](/rest/api/apimanagement/). El acceso a la API REST funciona para los portales tanto administrados como autohospedados.

## <a name="automated-migration-script"></a>Script de migración automatizada

Puede usar la API para automatizar la migración de contenido entre dos servicios API Management; por ejemplo, un servicio en el entorno de prueba y un servicio en el entorno de producción. El script `scripts.v3/migrate.js` del [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal/blob/master/scripts.v3/migrate.js) del portal para desarrolladores de API Management simplifica este proceso de automatización.

> [!WARNING]
> El script quita el contenido del portal para desarrolladores en el servicio de API Management de destino. Si esto le preocupa, asegúrese de hacer una copia de seguridad.

> [!NOTE]
> Si usa un portal autohospedado con una cuenta de almacenamiento personalizada definida explícitamente para hospedar archivos multimedia (es decir, define el valor `blobStorageUrl` en el archivo de configuración `config.design.json`), tiene que usar el [script](https://github.com/Azure/api-management-developer-portal/blob/master/scripts.v2/migrate.js) `scripts/migrate.js` original. El script original no funciona para los portales administrados ni autohospedados con la cuenta de almacenamiento multimedia administrada por API Management. En ese caso, use el script de la carpeta `/scripts.v3` en su lugar.

Este script lleva a cabo los pasos siguientes:

1. Captura el contenido del portal y los medios del servicio API Management de origen.
1. Quita el contenido del portal y los medios del servicio API Management de destino.
1. Carga el contenido del portal y los medios en el servicio API Management de destino.
1. De manera opcional y solo para los portales administrados, publica el portal automáticamente.

Una vez que el script se ha ejecutado correctamente, el servicio API Management de destino debería contener el mismo contenido del portal que el servicio de origen, y podrá verlo como administrador.

* Si usa un portal administrado, puede establecer el script para que publique automáticamente el portal de destino para que la versión migrada esté disponible automáticamente para los visitantes. 
* Si usa un portal autohospedado, tiene que publicar el portal de destino manualmente. Siga las instrucciones de publicación y hospedaje del tutorial para [configurar un portal para desarrolladores autohospedado](developer-portal-self-host.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)
- [Autohospedaje del portal para desarrolladores](developer-portal-self-host.md)