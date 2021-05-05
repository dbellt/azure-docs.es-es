---
title: Preparación de los recursos técnicos de contenedor de Azure
description: Recursos técnicos e instrucciones para ayudarle a configurar una oferta de contenedor en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 03/30/2021
ms.openlocfilehash: 0bbb1cb1cac4f531b090fc49bd7848b2bb4a2b8f
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107892546"
---
# <a name="prepare-azure-container-technical-assets"></a>Preparación de los recursos técnicos de contenedor de Azure

En este artículo se proporcionan recursos técnicos y recomendaciones para ayudarle a crear una oferta de contenedor en Azure Marketplace.

## <a name="before-you-begin"></a>Antes de empezar

Para ver guías de inicio rápido, tutoriales y ejemplos, consulte [Documentación de Azure Container Instances](../container-instances/index.yml).

## <a name="fundamental-technical-knowledge"></a>Conocimientos técnicos básicos

El diseño, la compilación y las pruebas de estos recursos lleva tiempo y requiere conocimientos técnicos de la plataforma de Azure y las tecnologías que se usan para crear la oferta.

Además del dominio de la solución, el equipo de ingeniería debe tener conocimientos sobre las siguientes tecnologías de Microsoft:

- Conocimientos básicos de los [Servicios de Azure](https://azure.microsoft.com/services/).
- Cómo [diseñar y estructurar las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/).
- Conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage) y [redes de Azure](https://azure.microsoft.com/services/?filter=networking).
- Conocimientos prácticos de [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
- Conocimientos prácticos de [JSON](https://www.json.org/).

## <a name="suggested-tools"></a>Herramientas sugeridas

Elija uno o ambos de los entornos de scripting siguientes para ayudar a administrar la imagen de contenedor:

- [Azure PowerShell](/powershell/azure/)
- [CLI de Azure](/cli/azure/)

Se recomienda agregar estas herramientas al entorno de desarrollo:

- [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows)
- [Visual Studio Code](https://code.visualstudio.com/)
  - Extensión: [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
  - Extensión: [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
  - Extensión: [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json).

Revise las herramientas disponibles en la página [Herramientas de desarrollo de Azure](https://azure.microsoft.com/). Si usa Visual Studio, revise las herramientas disponibles en [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="create-the-container-image"></a>Creación de la imagen de contenedor

No se puede implementar una imagen de un registro local en Azure Container Instances.

- Si ya tiene un contenedor de trabajo en el registro local, cree un registro de Azure y cargue la imagen de contenedor en Azure Container Registry. Para más información, consulte el [Tutorial: Compilación e implementación de imágenes de contenedor en la nube con Azure Container Registry Tasks](../container-registry/container-registry-tutorial-quick-task.md).

- Si aún no tiene una imagen de contenedor y necesita incluir en un contenedor la aplicación o crear una aplicación basada en contenedor, clone el código fuente de la aplicación desde GitHub, cree una imagen de contenedor desde el origen de la aplicación y pruebe la imagen en un entorno local de Docker. Para más información, consulte el [Tutorial: Creación de una imagen de contenedor para su implementación en Azure Container Instances](../container-instances/container-instances-tutorial-prepare-app.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de la oferta de contenedor](azure-container-offer-setup.md)