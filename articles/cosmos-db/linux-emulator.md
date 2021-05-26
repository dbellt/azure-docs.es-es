---
title: Ejecución del emulador de Azure Cosmos DB en Docker para Linux
description: Aprenda a ejecutar y usar el emulador de Linux de Azure Cosmos DB en Linux y macOS. Con el emulador, puede desarrollar y probar una aplicación localmente de forma gratuita, sin necesidad de una suscripción a Azure.
ms.service: cosmos-db
ms.topic: how-to
author: StefArroyo
ms.author: esarroyo
ms.date: 05/25/2021
ms.openlocfilehash: fe14c28d817d9c0a2e832d331af9130c935affb8
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386439"
---
# <a name="run-the-emulator-on-docker-for-linux-preview"></a>Ejecución del emulador en Docker para Linux (versión preliminar)

El emulador de Azure Cosmos DB proporciona un entorno local que emula el servicio Azure Cosmos DB con fines de desarrollo. Actualmente, el emulador de Linux solo admite SQL API. Mediante el Emulador de Azure Cosmos DB, puede desarrollar y probar su aplicación localmente, sin crear una suscripción de Azure o realizar algún gasto. Cuando esté satisfecho con el funcionamiento de la aplicación en el emulador de Linux de Azure Cosmos DB, puede cambiar a una cuenta de Azure Cosmos DB en la nube. En este artículo se describe cómo instalar y usar el emulador en entornos macOS y Linux.

> [!NOTE]
> El emulador de Linux de Cosmos DB está actualmente en modo de versión preliminar y solo admite SQL API. Los usuarios pueden experimentar ligeras degradaciones del rendimiento en cuanto al número de solicitudes por segundo procesadas por el emulador en comparación con la versión de Windows. El número predeterminado de particiones físicas que afecta directamente al número de contenedores que se pueden aprovisionar es 10.
> 
> No se recomienda usar el emulador (versión preliminar) en producción. Para cargas de trabajo más pesadas, use nuestro [emulador de Windows](local-emulator.md).

## <a name="how-does-the-emulator-work"></a>¿Cómo funciona el emulador?

El emulador de Linux de Azure Cosmos DB proporciona una emulación de gran fidelidad del servicio Azure Cosmos DB. Cuenta con una funcionalidad equivalente a la de Azure Cosmos DB, que incluye la creación y la consulta de datos, el aprovisionamiento y el escalado de contenedores, así como la ejecución de procedimientos almacenados y desencadenadores. Puede desarrollar y probar aplicaciones mediante el emulador de Linux de Azure Cosmos DB e implementarlas en Azure a escala global actualizando el punto de conexión de Azure Cosmos DB.

Cuando se usa el emulador, no son aplicables las funcionalidades que se basan en la infraestructura de Azure, como la replicación global, la latencia de milisegundos de un solo dígito para lectura/escritura y los niveles de coherencia ajustables.

## <a name="differences-between-the-linux-emulator-and-the-cloud-service"></a>Diferencias entre el emulador de Linux y el servicio en la nube
Dado que el emulador de Azure Cosmos DB proporciona un entorno emulado que se ejecuta en una estación de trabajo de desarrollador local, hay algunas diferencias de funcionalidad entre el emulador y una cuenta de Azure Cosmos en la nube:

- Actualmente, el panel **Explorador de datos** del emulador solo es totalmente compatible con los clientes de SQL API.

- Con el emulador de Linux, solo puede crear una cuenta de Azure Cosmos en modo de [rendimiento aprovisionado](set-throughput.md). Actualmente no se admite el modo [sin servidor](serverless.md).

- El emulador de Linux no es un servicio escalable y no admite un gran número de contenedores. Cuando se usa el emulador de Azure Cosmos DB, de manera predeterminada se pueden crear hasta 10 contenedores de tamaño fijo a 400 RU/s (solo compatibles con los SDK de Azure Cosmos DB) o 5 contenedores ilimitados. Para obtener más información sobre cómo cambiar este valor, consulte el artículo sobre la [configuración del valor de PartitionCount](emulator-command-line-parameters.md#set-partitioncount).

- Aunque los [niveles de coherencia](consistency-levels.md), como el servicio en la nube, se pueden ajustar mediante argumentos de línea de comandos solo para escenarios de prueba (la configuración predeterminada es Sesión), es posible que un usuario no espere el mismo comportamiento que en el servicio en la nube. Por ejemplo, la coherencia fuerte y de obsolescencia limitada no tiene ningún efecto en el emulador, aparte de señalar al SDK de Cosmos DB la coherencia predeterminada de la cuenta.

- El emulador de Linux no ofrece [replicación en varias regiones](distribute-data-globally.md).

- Es posible que la copia del emulador de Linux de Azure Cosmos DB no tenga siempre los cambios más recientes del servicio Azure Cosmos DB, por lo que debe consultar la herramienta de [planeamiento de capacidad de Azure Cosmos DB](estimate-ru-with-capacity-planner.md) para calcular con precisión las necesidades de rendimiento (RU) de la aplicación. <add link>

- El emulador de Linux admite un tamaño máximo de la propiedad ID de 254 caracteres.

## <a name="run-the-linux-emulator-on-macos"></a><a id="run-on-macos"></a>Ejecución del emulador de Linux en macOS

> [!NOTE]
> El emulador solo admite MacBooks con procesadores Intel.

Para empezar, visite el Docker Hub e instale [Docker Desktop para macOS](https://hub.docker.com/editions/community/docker-ce-desktop-mac/). Siga estos pasos para ejecutar el emulador en macOS:

[!INCLUDE[linux-emulator-instructions](includes/linux-emulator-instructions.md)]

## <a name="install-the-certificate"></a><a id="install-certificate"></a>Instalar el certificado

1. Una vez que se ejecute el emulador, con un terminal diferente, cargue la dirección IP de la máquina local en una variable.

    ```bash
    ipaddr="`ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1`"
    ```

1. A continuación, descargue el certificado del emulador.

    ```bash
    curl -k https://$ipaddr:8081/_explorer/emulator.pem > emulatorcert.crt
    ```
    Como alternativa, el punto de conexión anterior, que descarga el certificado autofirmado del emulador, también se puede usar para la señalización cuando el punto de conexión del emulador está listo para recibir solicitudes de otra aplicación.

1. Copie el archivo CRT en la carpeta que contiene los certificados personalizados de la distribución de Linux. Normalmente, en las distribuciones de Debian, se encuentra en `/usr/local/share/ca-certificates/`.

   ```bash
   cp YourCTR.crt /usr/local/share/ca-certificates/
   ```

1. Actualice los certificados TLS/SSL, que actualizarán la carpeta `/etc/ssl/certs/`.

   ```bash
   update-ca-certificates
   ```

En el caso de las aplicaciones basadas en Java, el certificado debe importarse en el [almacén de confianza de Java.](local-emulator-export-ssl-certificates.md)

## <a name="consume-the-endpoint-via-ui"></a><a id="consume-endpoint-ui"></a>Consumo del punto de conexión a través de la interfaz de usuario

El emulador usa un certificado autofirmado para proteger la conectividad a su punto de conexión y debe ser de confianza manual. Siga estos pasos para consumir el punto de conexión a través de la interfaz de usuario mediante el explorador web deseado:

1. Asegúrese de descargar el certificado autofirmado del emulador.

   ```bash
   curl -k https://$ipaddr:8081/_explorer/emulator.pem > emulatorcert.crt
   ```

1. Abra la aplicación de **Acceso a Llaveros** en el Mac para importar el certificado del emulador.

1. Seleccione **Archivo** e **Importar elementos e**  importe el archivo **emulatorcert.crt**.

1. Después de cargar el archivo *emulatorcert.crt* en Llaveros, haga doble clic en el nombre de **localhost** y cambie la configuración de confianza a **Confiar siempre**.

1. Ahora puede ir a `https://localhost:8081/_explorer/index.html` o `https://{your_local_ip}:8081/_explorer/index.html` y recuperar la cadena de conexión del emulador.

## <a name="run-the-linux-emulator-on-linux-os"></a><a id="run-on-linux"></a>Ejecución del emulador de Linux en el sistema operativo Linux

Para empezar, use el paquete `apt` e instale la versión más reciente de Docker.

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Si usa Subsistema de Windows para Linux (WSL), ejecute el siguiente comando para obtener `ifconfig`:

```bash
sudo apt-get install net-tools
```

Siga estos pasos para ejecutar el emulador en Linux:

[!INCLUDE[linux-emulator-instructions](includes/linux-emulator-instructions.md)]

4. Una vez que se ejecute el emulador, con un terminal diferente, cargue la dirección IP de la máquina local en una variable.

    ```bash
    ipaddr="`ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}' | head -n 1`"
    ```

5. A continuación, descargue el certificado del emulador. Como alternativa, el punto de conexión siguiente, que descarga el certificado autofirmado del emulador, también se puede usar para la señalización cuando el punto de conexión del emulador está listo para recibir solicitudes de otra aplicación.

    ```bash
    curl -k https://$ipaddr:8081/_explorer/emulator.pem > ~/emulatorcert.crt
    ```

6. Copie el archivo CRT en la carpeta que contiene los certificados personalizados de la distribución de Linux. Normalmente, en las distribuciones de Debian, se encuentra en `/usr/local/share/ca-certificates/`.

   ```bash
   cp ~/emulatorcert.crt /usr/local/share/ca-certificates/
   ```

7. Actualice los certificados TLS/SSL, que actualizarán la carpeta `/etc/ssl/certs/`.

   ```bash
   update-ca-certificates
   ```

    En el caso de las aplicaciones basadas en Java, el certificado debe importarse en el [almacén de confianza de Java](local-emulator-export-ssl-certificates.md).

    ```bash
    keytool -keystore ~/cacerts -importcert -alias  emulator_cert -file ~/emulatorcert.crt
    java -ea -Djavax.net.ssl.trustStore=~/cacerts -Djavax.net.ssl.trustStorePassword="changeit" $APPLICATION_ARGUMENTS
    ```

## <a name="configuration-options"></a><a id="config-options"></a>Opciones de configuración

|Nombre  |Valor predeterminado  |Descripción  |
|---------|---------|---------|
|  Puertos: `-p`   |         |   Actualmente, el punto de conexión del emulador solo necesita los puertos 8081 y 10251-10255.     |
| `AZURE_COSMOS_EMULATOR_PARTITION_COUNT`    |    10     |    Controla el número total de particiones físicas, que a cambio controla el número de contenedores que se pueden crear y pueden existir en un momento dado. Se recomienda empezar por valores pequeños para mejorar el tiempo de actividad de inicio del emulador, es decir, 3.     |
|  Memoria: `-m`   |         | En memoria, se requieren 3 GB o más.     |
| Núcleos: `--cpus`  |         |   Asegúrese de aprovisionar suficiente memoria y núcleos de CPU. Aunque el emulador podría ejecutarse con tan solo 0,5 núcleos (muy lentos), se recomiendan al menos 2 núcleos.      |
|`AZURE_COSMOS_EMULATOR_ENABLE_DATA_PERSISTENCE`  | false  | Esta configuración usada por el emulador ayudará a conservar los datos entre reinicios de contenedor.  |

## <a name="troubleshoot-issues"></a><a id="troubleshoot-issues"></a>Solución de problemas

En esta sección se proporcionan sugerencias para solucionar errores al usar el emulador de Linux.

### <a name="connectivity-issues"></a>Problemas de conectividad

#### <a name="my-app-cant-connect-to-emulator-endpoint-the-ssl-connection-could-not-be-established-or-i-cant-start-the-data-explorer"></a>Mi aplicación no se puede conectar al punto de conexión del emulador ("No se pudo establecer la conexión SSL") o no puedo iniciar el Explorador de datos

- Asegúrese de que el emulador se está ejecutando con el siguiente comando:

    ```bash
    docker ps --all
    ```

- Compruebe que el contenedor específico del emulador está en estado de ejecución.

- Compruebe que ninguna otra aplicación usa puertos del emulador: 8081 y 10250-10255.

- Compruebe que el puerto 8081 del contenedor está asignado correctamente y es accesible desde un entorno fuera del contenedor.  

   ```bash
   netstat -lt
   ```

- Intente acceder al punto de conexión y al puerto del emulador mediante la dirección IP del contenedor de Docker en lugar de "localhost".

- Asegúrese de que el certificado autofirmado del emulador se ha agregado correctamente a [Llaveros](#consume-endpoint-ui).

- Asegúrese de que el certificado autofirmado del emulador se ha importado correctamente en la ubicación esperada:
  - .NET: consulte la [sección Certificados](#run-on-linux).
  - Java: consulte la [sección Almacén de certificados de Java](#run-on-linux).

#### <a name="the-docker-container-failed-to-start"></a>No se pudo iniciar el contenedor de Docker

El emulador produce un error con el mensaje siguiente:

```bash
/palrun: ERROR: Invalid mapping of address 0x40037d9000 in reserved address space below 0x400000000000. Possible causes:
1. The process (itself, or via a wrapper) starts up its own running environment sets the stack size limit to unlimited via syscall setrlimit(2);
2. The process (itself, or via a wrapper) adjusts its own execution domain and flag the system its legacy personality via syscall personality(2);
3. Sysadmin deliberately sets the system to run on legacy VA layout mode by adjusting a sysctl knob vm.legacy_va_layout.
```

Este error es probable que se deba a que el tipo de procesador de host de Docker actual no es compatible con nuestra imagen de Docker. Es decir, el equipo es un MacBook con un conjunto de chips M1.

#### <a name="my-app-received-too-many-connectivity-related-timeouts"></a>Mi aplicación ha recibido demasiados tiempos de espera relacionados con la conectividad

- El contenedor de Docker no se aprovisiona con suficientes recursos [(núcleos o memoria)](#config-options). Se recomienda aumentar el número de núcleos y, como alternativa, reducir el número de particiones físicas aprovisionadas durante el inicio.

- Asegúrese de que el número de conexiones TCP no supere la configuración actual del sistema operativo.

- Intente reducir el tamaño de los documentos de la aplicación.
    
#### <a name="my-app-could-not-provision-databasescontainers"></a>Mi aplicación no pudo aprovisionar bases de datos o contenedores

El número de particiones físicas aprovisionadas en el emulador es demasiado bajo. Elimine las bases de datos o colecciones no utilizadas o inicie el emulador con un [mayor número de particiones físicas](#config-options).

### <a name="reliability-and-crashes"></a>Confiabilidad y bloqueos

- No se puede iniciar el emulador:

  - Asegúrese de que está [ejecutando la imagen más reciente del emulador de Cosmos DB para Linux.](#refresh-linux-container) De lo contrario, consulte la sección anterior sobre problemas relacionados con la conectividad.

  - Si la carpeta de datos del emulador de Cosmos DB está "montada en el volumen", asegúrese de que el volumen tiene suficiente espacio y es de lectura /escritura.

  - Confirme que funciona la creación de un contenedor con la configuración recomendada. En caso afirmativo, lo más probable es que la causa del error sea la configuración adicional pasada a través del comando de Docker correspondiente al iniciar el contenedor.

  - Si el emulador no se puede iniciar con el siguiente error:
  
    ```bash
    "Failed loading Emulator secrets certificate. Error: 0x8009000f or similar, a new policy might have been added to your host that prevents an application such as Azure Cosmos DB emulator from creating and adding self signed certificate files into your certificate store."
    ```

    Esto puede ocurrir incluso cuando se ejecuta en el contexto de administrador, ya que la directiva específica que normalmente agrega el departamento de TI tiene prioridad sobre el administrador local. En su lugar, el uso de una imagen de Docker para el emulador puede ayudar en este caso, siempre y cuando tenga el permiso para agregar el certificado SSL del emulador autofirmado en el contexto del equipo host (esto es necesario para la aplicación cliente del SDK de Java y .NET Cosmos).

- El emulador se bloquea:

  - Confirme que funciona la creación de un contenedor con la [configuración recomendada](#run-on-linux). En caso afirmativo, lo más probable es que la causa del error sea la configuración adicional pasada a través del comando de Docker correspondiente al iniciar el contenedor.

  - Inicie el contenedor de Docker del emulador en un modo conectado (consulte `docker start -it`).

  - Recopile el volcado o los datos relacionados con el bloqueo y siga los [pasos descritos](#report-an-emulator-issue) para notificar el problema.  

### <a name="data-explorer-errors"></a>Errores del Explorador de datos

- No puedo ver mis datos:

  - Consulte la sección anterior sobre los problemas relacionados con la conectividad.

  - Asegúrese de que el certificado del emulador autofirmado se importa correctamente y se confía manualmente para que el explorador acceda a la página del explorador de datos.

  - Pruebe a crear una base de datos o un contenedor e insertar un elemento mediante el Explorador de datos. Si se realiza correctamente, lo más probable es que la causa del problema resida en la aplicación. Si no es así, [póngase en contacto con el equipo de Cosmos DB](#report-an-emulator-issue).

### <a name="performance-issues"></a>Problemas de rendimiento

El número de solicitudes por segundo es bajo y la latencia de las solicitudes es alta:

- El contenedor de Docker no se aprovisiona con suficientes recursos [(núcleos o memoria)](#config-options). Se recomienda aumentar el número de núcleos y, como alternativa, reducir el número de particiones físicas aprovisionadas durante el inicio.

## <a name="refresh-linux-container"></a><a id="refresh-linux-container"></a>Actualización de contenedor de Linux

Siga estos pasos para actualizar el contenedor de Linux:

1. Ejecute el comando siguiente para ver todos los contenedores de Docker.

   ```bash
   docker ps --all
   ```

1. Quite el contenedor con el identificador recuperado del comando anterior.

   ```bash
   docker rm ID_OF_CONTAINER_FROM_ABOVE
   ```

1. A continuación, se muestran todas las imágenes de Docker.

   ```bash
   docker images
   ```

1. Quite la imagen con el identificador recuperado del paso anterior.

   ```bash
   docker rmi ID_OF_IMAGE_FROM_ABOVE
   ```

1. Extraiga la imagen más reciente del emulador de Linux de Cosmos DB.

   ```bash
   docker pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator
   ```

1. Para iniciar un contenedor detenido, ejecute lo siguiente:

   ```bash
   docker start -ai ID_OF_CONTAINER
   ```

## <a name="report-an-emulator-issue"></a>Notificación de un problema del emulador

Al notificar un problema con el emulador de Linux, proporcione tanta información como sea posible sobre el problema. Estos detalles incluyen:

- Descripción del error o problema detectado
- Entorno (sistema operativo, configuración de host)
- Tipo de equipo y procesador
- Comando que se usa para crear e iniciar el emulador (archivo YML si se usa Docker Compose)
- Descripción de la carga de trabajo
- Ejemplo de la base de datos o colección y el elemento usados
- Inclusión de la salida de la consola al iniciar el contenedor de Docker para el emulador en modo adjunto
- Envíe todo lo anterior al [equipo de Azure Cosmos DB](mailto:cdbportalfeedback@microsoft.com).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a usar el emulador de Linux de Azure Cosmos DB para el desarrollo gratuito de aplicaciones en un entorno local. Ahora puede avanzar a los siguientes artículos:

- [Exportación de los certificados del emulador de Azure Cosmos DB para su uso con aplicaciones en Java, Python y Node.js](local-emulator-export-ssl-certificates.md)
- [Depuración de problemas con el emulador](troubleshoot-local-emulator.md)
