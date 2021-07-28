---
author: baanders
description: archivo de inclusión para comprobar la asignación de roles en la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/22/2020
ms.author: baanders
ms.openlocfilehash: 044342122c2bd1d2b59a7fc3abb4ed6ca7bbc05f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110473845"
---
Una manera de comprobar que configuró correctamente la asignación de roles es consultar las asignaciones de roles para la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com). Vaya a la instancia de Azure Digital Twins en Azure Portal (puede buscarla en la página de [instancias de Azure Digital Twins](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) o buscar su nombre en la barra de búsqueda del portal).

Luego, consulte todos sus roles asignados en *Control de acceso (IAM) > Asignaciones de roles*. El usuario debe aparecer en la lista con el rol *Propietario de datos de Azure Digital Twins*. 

:::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Captura de pantalla de las asignaciones de roles para una instancia de Azure Digital Twins en Azure Portal.":::