---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 11/03/2016
ms.author: cephalin
ms.openlocfilehash: 0019e50615f3e66778709ad8cb28f92967c66e2e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96018464"
---
## <a name="deployment-customization"></a>Personalización de la implementación

En el proceso de implementación se da por supuesto que el archivo .zip que se inserta contiene una aplicación lista para ejecutarse. No se ejecuta ninguna personalización de forma predeterminada. Para habilitar los mismos procesos de compilación que se obtienen con la integración continua, agregue los siguientes elementos a la configuración de la aplicación:

`SCM_DO_BUILD_DURING_DEPLOYMENT=true`

Cuando se realiza la implementación de inserción del archivo .zip, el valor que se establece de forma predeterminada es **false**. El valor predeterminado es **true** para las implementaciones de integración continua. Cuando el valor se establece en **true**, la configuración relacionada con la implementación se usa durante la implementación. Puede configurar estas opciones a modo de configuración de la aplicación o en un archivo de configuración de implementación que se encuentra en la raíz del archivo .zip. Para obtener más información, consulte [Repository and deployment-related settings](https://github.com/projectkudu/kudu/wiki/Configurable-settings#repository-and-deployment-related-settings) (Repositorio y configuración relacionada con la implementación) en la referencia de la implementación.