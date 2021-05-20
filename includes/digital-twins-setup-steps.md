---
author: baanders
description: archivo de inclusión para información general de los pasos de la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 10/14/2020
ms.author: baanders
ms.openlocfilehash: 19d7c55a669a3975cb4bc1dce707e2d1165be0b3
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759984"
---
La configuración completa de una instancia nueva de Azure Digital Twins consta de dos partes:
1. **Creación de la instancia**
2. **Configuración de permisos de acceso de usuarios**: los usuarios de Azure deben tener el rol **Propietario de datos de Azure Digital Twins** en la instancia de Azure Digital Twins para poder administrarla y administrar sus datos. En este paso, como propietario o administrador de la suscripción de Azure, se asignará este rol a la persona que vaya a administrar la instancia de Azure Digital Twins. Puede ser usted mismo o alguna otra persona de la organización.
 
>[!IMPORTANT]
>Para completar este artículo completo y configurar completamente una instancia utilizable, necesita permisos para administrar los recursos y el acceso de usuarios en la suscripción de Azure. El primer paso lo puede completar cualquier persona que pueda crear recursos en la suscripción, pero el segundo paso requiere permisos de administración de acceso de usuarios (o la cooperación de alguien con estos permisos). Puede obtener más información sobre este asunto en la sección [Requisitos previos: permisos necesarios](#prerequisites-permission-requirements) para el paso de permisos de acceso de usuarios.
