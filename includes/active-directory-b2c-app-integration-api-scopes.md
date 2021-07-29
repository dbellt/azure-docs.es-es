---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 06/11/2021
ms.author: mimart
ms.openlocfilehash: cd6bddb6988555b336cb920c481fe675e9f7a276
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112073179"
---
1. Seleccione la aplicación *my-api1* que creó para abrir la página **Información general**.
1. En **Administrar**, seleccione **Exponer una API**.
1. Junto a **URI de id. de aplicación**, seleccione el vínculo **Establecer**. Reemplace el valor predeterminado (un GUID) por un nombre único, como `tasks-api`, y, a continuación, seleccione **Guardar**. Cuando la aplicación web solicite un token de acceso para la API web, deberá agregar este URI como prefijo para cada ámbito que se defina para la API.
1. En **Ámbitos definidos con esta API**, seleccione **Agregar un ámbito**.
1. Escriba los valores siguientes para crear un ámbito que defina el acceso de lectura a la API y, a continuación, seleccione **Agregar ámbito**:
    1. **Nombre de ámbito**: `tasks.read`
    1. **Nombre para mostrar del consentimiento del administrador**: `Read access to tasks API`
    1. **Descripción del consentimiento del administrador**: `Allows read access to the tasks API`
1. Seleccione **Agregar un ámbito**, escriba los valores siguientes para agregar un ámbito que defina el acceso de escritura a la API y, a continuación, seleccione **Agregar ámbito**:
    1. **Nombre de ámbito**: `tasks.write`
    1. **Nombre para mostrar del consentimiento del administrador**: `Write access to tasks API`
    1. **Descripción del consentimiento del administrador**: `Allows write access to the tasks API`