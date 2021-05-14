---
title: 'Configuración de contenedores: Face'
titleSuffix: Azure Cognitive Services
description: El entorno en tiempo de ejecución del contenedor de Face se configura mediante los argumentos del comando `docker run`. Hay configuraciones obligatorias y opcionales.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/29/2021
ms.author: aahi
ms.openlocfilehash: dc9aa1636bf4594d1e0b51315b520c6e893ced88
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108288479"
---
# <a name="configure-face-docker-containers-retiring"></a>Configuración de los contenedores de Face en Docker (en retirada)

> [!IMPORTANT]
> La versión preliminar del contenedor de Face ya no acepta aplicaciones, y el contenedor ha quedado en desuso a partir del 29 de abril de 2021. El contenedor de Face se retirará por completo el 26 de julio de 2021.

El entorno en tiempo de ejecución del contenedor de **Face** se configura mediante los argumentos del comando `docker run`. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales. Hay disponibles varios [ejemplos](#example-docker-run-commands) del comando. La configuración específica del contenedor es la configuración de facturación. 

## <a name="configuration-settings"></a>Parámetros de configuración

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Las opciones [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) y [`Eula`](#eula-setting) se usan en conjunto y debe proporcionar valores válidos para las tres; en caso contrario, no se inicia el contenedor. Para obtener más información sobre el uso de estas opciones de configuración para crear instancias de un contenedor, consulte [Facturación](face-how-to-install-containers.md#billing).

## <a name="apikey-configuration-setting"></a>Opción de configuración ApiKey

La opción de configuración `ApiKey` especifica la clave de recurso de Azure usada para realizar un seguimiento de la información de facturación del contenedor. Debe especificar un valor para ApiKey que debe ser una clave válida para el recurso de _Cognitive Services_ especificado para la opción de configuración [`Billing`](#billing-configuration-setting).

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: Administración de recursos de **Cognitivas Services**, en **Claves**.

## <a name="applicationinsights-setting"></a>Opción de configuración ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Opción de configuración Billing

La opción de configuración `Billing` especifica el URI del punto de conexión del recurso de _Cognitive Services_ que se usa para medir la información de facturación del contenedor. Debe especificar un valor para esta opción de configuración, que debe ser un URI de punto de conexión válido para un recurso de _Cognitive Services_ en Azure. El contenedor informa sobre el uso cada 10 a 15 minutos.

Este valor se puede encontrar en el siguiente lugar:

* Azure Portal: Información general de **Cognitivas Services**, con la etiqueta `Endpoint`

No olvide agregar la ruta _Face_ al URI de punto de conexión, tal como se muestra en el ejemplo. 

|Obligatorio| Nombre | Tipo de datos | Descripción |
|--|------|-----------|-------------|
|Sí| `Billing` | String | Identificador URI del punto de conexión de facturación. Para más información sobre cómo obtener el URI de facturación, consulte la [recopilación de los parámetros necesarios](face-how-to-install-containers.md#gathering-required-parameters). Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](../cognitive-services-custom-subdomains.md) |

<!-- specific to face only -->

## <a name="cloudai-configuration-settings"></a>Opciones de configuración CloudAI

Las opciones de configuración de la sección `CloudAI` proporcionan opciones específicas de contenedor únicas para el contenedor. Se admiten las opciones de configuración y objetos siguientes para el contenedor de Face en la sección `CloudAI`.

| Nombre | Tipo de datos | Descripción |
|------|-----------|-------------|
| `Storage` | Object | Escenario de almacenamiento utilizado por el contenedor de Face. Para obtener más información sobre los escenarios de almacenamiento y la configuración asociada para el objeto `Storage`, consulte [Configuración del escenario de almacenamiento](#storage-scenario-settings) |

### <a name="storage-scenario-settings"></a>Configuración del escenario de almacenamiento

El contenedor de Face almacena datos de blob, de caché, de metadatos y de la cola, en función de lo que se va a almacenar. Por ejemplo, los índices de aprendizaje y los resultados de un objeto **LargePersonGroup** se almacenan como datos de blob. El contenedor de Face proporciona dos escenarios de almacenamiento diferentes al interactuar con estos tipos de datos y almacenarlos:

* Memoria  
  Los cuatro tipos de datos se almacenan en la memoria. No se distribuyen ni son persistentes. Si el contenedor de Face se detiene o elimina, se destruyen todos los datos en el almacenamiento de dicho contenedor.  
  Se trata del escenario de almacenamiento predeterminado para el contenedor de Face.
* Azure  
  El contenedor de Face utiliza Azure Storage y Azure Cosmos DB para distribuir estos cuatro tipos de datos en el almacenamiento persistente. Los datos de blob y de cola se controlan mediante Azure Storage. Los datos de la memoria caché y los metadatos se controlan mediante Azure Cosmos DB. Si el contenedor de Face se detiene o elimina, todos los datos en el almacenamiento de dicho contenedor permanecen almacenados en Azure Storage y Azure Cosmos DB.  
  Los recursos utilizados por el escenario de Azure Storage tienen los siguientes requisitos adicionales:
  * El recurso de Azure Storage debe usar el tipo de cuenta StorageV2.
  * El recurso de Azure Cosmos DB debe usar la API de Azure Cosmos DB para MongoDB.

Los escenarios de almacenamiento y las opciones de configuración asociadas se administran mediante el objeto `Storage`, en la sección de configuración `CloudAI`. Las opciones de configuración siguientes están disponibles en el objeto `Storage`:

| Nombre | Tipo de datos | Descripción |
|------|-----------|-------------|
| `StorageScenario` | String | Escenario de almacenamiento admitido por el contenedor. Están disponibles los valores siguientes:<br/>`Memory`: valor predeterminado. El contenedor usa un almacenamiento en memoria, no persistente y no distribuido para uso temporal y de nodo único. Si el contenedor se detiene o elimina, se destruye el almacenamiento de dicho contenedor.<br/>`Azure`: el contenedor usa recursos de Azure para el almacenamiento. Si el contenedor se detiene o elimina, se conserva el almacenamiento de dicho contenedor.|
| `ConnectionStringOfAzureStorage` | String | Cadena de conexión para el recurso de Azure Storage usado por el contenedor.<br/>Esta configuración solo se aplica si se especifica `Azure` para la opción de configuración `StorageScenario`. |
| `ConnectionStringOfCosmosMongo` | String | Cadena de conexión de MongoDB para el recurso de Azure Cosmos DB usado por el contenedor.<br/>Esta configuración solo se aplica si se especifica `Azure` para la opción de configuración `StorageScenario`. |

Por ejemplo, el siguiente comando especifica el escenario de almacenamiento de Azure y proporciona las cadenas de conexión de ejemplo para los recursos de Azure Storage y Cosmos DB usados para almacenar datos del contenedor de Face.

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 containerpreview.azurecr.io/microsoft/cognitive-services-face Eula=accept Billing=https://westcentralus.api.cognitive.microsoft.com/face/v1.0 ApiKey=0123456789 CloudAI:Storage:StorageScenario=Azure CloudAI:Storage:ConnectionStringOfCosmosMongo="mongodb://samplecosmosdb:0123456789@samplecosmosdb.documents.azure.com:10255/?ssl=true&replicaSet=globaldb" CloudAI:Storage:ConnectionStringOfAzureStorage="DefaultEndpointsProtocol=https;AccountName=sampleazurestorage;AccountKey=0123456789;EndpointSuffix=core.windows.net"
  ```

El escenario de almacenamiento se administra de forma independiente desde los montajes de entrada y salida. Puede especificar una combinación de estas características para un solo contenedor. Por ejemplo, el comando siguiente define un montaje de enlace de Docker para la carpeta `D:\Output` en el equipo host como montaje de salida y, a continuación, crea una instancia de un contenedora partir de la imagen de contenedor de Face, guardando los archivos de registro en formato JSON para el montaje de salida. El comando también especifica el escenario de almacenamiento de Azure y proporciona las cadenas de conexión de ejemplo para los recursos de Azure Storage y Cosmos DB usados para almacenar datos del contenedor de Face.

  ```Docker
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 --mount type=bind,source=D:\Output,destination=/output containerpreview.azurecr.io/microsoft/cognitive-services-face Eula=accept Billing=https://westcentralus.api.cognitive.microsoft.com/face/v1.0 ApiKey=0123456789 Logging:Disk:Format=json CloudAI:Storage:StorageScenario=Azure CloudAI:Storage:ConnectionStringOfCosmosMongo="mongodb://samplecosmosdb:0123456789@samplecosmosdb.documents.azure.com:10255/?ssl=true&replicaSet=globaldb" CloudAI:Storage:ConnectionStringOfAzureStorage="DefaultEndpointsProtocol=https;AccountName=sampleazurestorage;AccountKey=0123456789;EndpointSuffix=core.windows.net"
  ```

## <a name="eula-setting"></a>Opción de configuración Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Opción de configuración Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Configuración de las credenciales del proxy HTTP

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Opción de configuración Logging
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Configuración de montaje

Utilice montajes de enlace para leer y escribir datos hacia y desde el contenedor. Puede especificar un montaje de entrada o un montaje de salida mediante la opción `--mount` del comando [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Los contenedores de Face no usan los montajes de entrada o salida para almacenar datos de entrenamiento o del servicio. 

La sintaxis exacta de la ubicación de montaje del host varía según el sistema operativo del host. Además, la ubicación de montaje del [equipo host](face-how-to-install-containers.md#the-host-computer) puede no ser accesible debido a un conflicto entre los permisos que usa la cuenta de servicio de Docker y los permisos de la ubicación de montaje del host. 

|Opcional| Nombre | Tipo de datos | Descripción |
|-------|------|-----------|-------------|
|No permitida| `Input` | String | Los contenedores de Face no usan esto.|
|Opcional| `Output` | String | Destino del montaje de salida. El valor predeterminado es `/output`. Esta es la ubicación de los registros. Esto incluye los registros de contenedor. <br><br>Ejemplo:<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Comandos de ejemplo de docker run 

Los ejemplos siguientes usan las opciones de configuración para ilustrar cómo escribir y usar comandos `docker run`.  Una vez que se está ejecutando, el contenedor continúa ejecutándose hasta que lo [detenga](face-how-to-install-containers.md#stop-the-container).

* **Carácter de continuación de línea**: Los comandos de Docker de las secciones siguientes usan la barra diagonal inversa (`\`) como un carácter de continuación de línea. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host. 
* **Orden de los argumentos**: No cambie el orden de los argumentos a menos que esté muy familiarizado con los contenedores de Docker.

Reemplace {_argument_name_} por sus propios valores:

| Marcador de posición | Value | Formato o ejemplo |
|-------------|-------|---|
| **{CLAVE_API}** | La clave del punto de conexión del recurso `Face` en la página Claves de `Face` de Azure. | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
| **{URI_PUNTODECONEXIÓN}** | El valor del punto de conexión de facturación está disponible en la página Información general de Azure `Face`.| Consulte el apartado de [recopilación de los parámetros necesarios](face-how-to-install-containers.md#gathering-required-parameters) para ejemplos explícitos. |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](face-how-to-install-containers.md#billing).
> El valor de ApiKey es la **clave** de la página de claves de recursos de Azure `Cognitive Services`. 

## <a name="face-container-docker-examples"></a>Ejemplos de Docker del contenedor de Face

En los siguientes ejemplos de Docker encontrará el contenedor de Face. 

### <a name="basic-example"></a>Ejemplo básico 

  ```
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
  containerpreview.azurecr.io/microsoft/cognitive-services-face \
  Eula=accept \
  Billing={ENDPOINT_URI} \
  ApiKey={API_KEY} 
  ```

### <a name="logging-example"></a>Ejemplo de registro 

  ```
  docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 containerpreview.azurecr.io/microsoft/cognitive-services-face \
  Eula=accept \
  Billing={ENDPOINT_URI} ApiKey={API_KEY} \
  Logging:Console:LogLevel:Default=Information
  ```

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Instalación y ejecución de contenedores](face-how-to-install-containers.md)
