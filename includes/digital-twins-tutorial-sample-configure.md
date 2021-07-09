---
author: baanders
description: Archivo de inclusión para los tutoriales de Azure Digital Twins (configuración del proyecto de ejemplo)
ms.service: digital-twins
ms.topic: include
ms.date: 5/25/2020
ms.author: baanders
ms.openlocfilehash: 60d65e530c9d98ac86f8d958072d2f26a68836fe
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111350978"
---
## <a name="configure-the-sample-project"></a>Configuración del proyecto de ejemplo

A continuación, configure una aplicación cliente de ejemplo que interactuará con su instancia de Azure Digital Twins.

Vaya en la máquina hasta el archivo que descargó anteriormente en el artículo con [ejemplos de un extremo a otro de Azure Digital Twins](/samples/azure-samples/digital-twins-samples/digital-twins-samples) (y descomprímalo si aún no lo ha hecho).

Una vez que esté dentro de la carpeta, vaya a _AdtSampleApp_. Abra _**AdtE2ESample.sln**_ en Visual Studio 2019. 

En Visual Studio, seleccione el archivo _SampleClientApp > **appsettings.json**_ para abrirlo en la ventana de edición. Este servirá como archivo JSON predefinido con las variables de configuración necesarias para ejecutar el proyecto.

En el cuerpo del archivo, cambie el valor de `instanceUrl` por la *dirección URL del nombre de host* de la instancia de Azure Digital Twins (mediante la adición de **_https://_** delante del *nombre de host*, tal y como se muestra a continuación).

```json
{
  "instanceUrl": "https://<your-Azure-Digital-Twins-instance-host-name>"
}
```

Guarde y cierre el archivo. 

Después, configure el archivo *appsettings.json* para que se copie al directorio de salida cuando compile *SampleClientApp*. Para ello, seleccione el archivo *appsettings.json* con el botón derecho y elija **Propiedades.** . En el inspector de **Propiedades**, busque la propiedad *Copiar en el directorio de resultados*. Cambie el valor a **Copiar si es posterior** si tiene otro valor.

:::image type="content" source="../articles/digital-twins/media/includes/copy-config.png" alt-text="Captura de pantalla del Explorador de soluciones en Visual Studio con appsettings.json y la propiedad &quot;Copiar en el directorio de salida&quot; resaltada en Propiedades." border="false" lightbox="../articles/digital-twins/media/includes/copy-config.png":::

Mantenga el _**AdtE2ESample**_ proyecto abierto en Visual Studio para seguir utilizándolo en el tutorial.

[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](digital-twins-local-credentials-outer.md)]
