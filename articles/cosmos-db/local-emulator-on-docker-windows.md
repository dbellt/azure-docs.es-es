---
title: Ejecución del emulador en Docker para Windows
itleSuffix: Running the Azure Cosmos DB emulator on Docker for Windows
description: Aprenda a ejecutar y usar el emulador de Azure Cosmos DB en Docker para Windows. Con el emulador, puede desarrollar y probar una aplicación localmente de forma gratuita, sin necesidad de crear una suscripción a Azure.
ms.service: cosmos-db
ms.topic: how-to
author: StefArroyo
ms.author: esarroyo
ms.date: 04/20/2021
ms.openlocfilehash: 0e38f39edeb68577470868e0bd68a37cb7fe0516
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386432"
---
# <a name="use-the-emulator-on-docker-for-windows"></a><a id="run-on-windows-docker"></a>Uso del emulador en Docker para Windows

Puede ejecutar el emulador de Azure Cosmos DB en el contenedor de Docker de Windows. Vea en [Docker Hub](https://hub.docker.com/r/microsoft/azure-cosmosdb-emulator/) el comando docker pull y en [GitHub](https://github.com/Azure/azure-cosmos-db-emulator-docker) el archivo `Dockerfile` y más información. Actualmente, el emulador no funciona en Docker para Oracle Linux. Complete las instrucciones siguientes para ejecutar el emulador en Docker para Windows:

1. Después de instalar [Docker para Windows](https://www.docker.com/docker-windows), cambie a contenedores de Windows haciendo clic con el botón derecho en el icono de Docker en la barra de herramientas y seleccionando **Switch to Windows containers** (Cambiar a contenedores de Windows).

1. A continuación, extraiga la imagen del emulador de Docker Hub ejecutando el siguiente comando desde el shell que prefiera.

   ```bash
   docker pull mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```

1. Para iniciar la imagen, ejecute los siguientes comandos en función de si utiliza la línea de comandos o el entorno de PowerShell:

   # <a name="command-line"></a>[Línea de comandos](#tab/cli)

   ```bash

   md %LOCALAPPDATA%\CosmosDBEmulator\bind-mount

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%LOCALAPPDATA%\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator
   ```
   Es posible que las imágenes de Docker basadas en Windows no sean generalmente compatibles con todos los sistemas operativos host de Windows. Por ejemplo, la imagen predeterminada del emulador de Azure Cosmos DB solo es compatible con Windows 10 y Windows Server 2016. Si necesita una imagen compatible con Windows Server 2019, ejecute el siguiente comando:

   ```bash
   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=%hostDirectory%,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/winsrv2019/azure-cosmos-emulator:latest
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell

   md $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount 2>null

   docker run --name azure-cosmosdb-emulator --memory 2GB --mount "type=bind,source=$env:LOCALAPPDATA\CosmosDBEmulator\bind-mount,destination=C:\CosmosDB.Emulator\bind-mount" --interactive --tty -p 8081:8081 -p 8900:8900 -p 8901:8901 -p 8902:8902 -p 10250:10250 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 -p 10255:10255 -p 10256:10256 -p 10350:10350 mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator

   ```

   La respuesta será similar a la siguiente:

   ```bash
   Starting emulator
   Emulator Endpoint: https://172.20.229.193:8081/
   Primary Key: C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==
   Exporting SSL Certificate
   You can import the SSL certificate from an administrator command prompt on the host by running:
   cd /d %LOCALAPPDATA%\CosmosDBEmulatorCert
   powershell .\importcert.ps1
   --------------------------------------------------------------------------------------------------
   Starting interactive shell
   ```
   ---

   > [!NOTE]
   > Si al ejecutar el comando `docker run` ve un error de conflicto de puerto (es decir, si el puerto especificado ya está en uso), pase un puerto personalizado modificando los números de puerto. Por ejemplo, puede cambiar el parámetro "-p 8081:8081" a "-p 443:8081".

1. Ahora, use el punto de conexión del emulador y la clave principal de la respuesta e importe el certificado TLS/SSL en el host. Para importar el certificado TLS/SSL, ejecute los siguientes pasos desde un símbolo del sistema de administrador:

   # <a name="command-line"></a>[Línea de comandos](#tab/cli)

   ```bash
   cd  %LOCALAPPDATA%\CosmosDBEmulator\bind-mount
   powershell .\importcert.ps1
   ```

   # <a name="powershell"></a>[PowerShell](#tab/powershell)

   ```powershell
   cd $env:LOCALAPPDATA\CosmosDBEmulator\bind-mount
   .\importcert.ps1
   ```
   ---

1. Si cierra el shell interactivo una vez iniciado el emulador, se cerrará el contenedor del emulador. Para volver a abrir el explorador de datos, vaya a la siguiente dirección URL en el explorador web. El punto de conexión del emulador se proporciona en el mensaje de respuesta mostrado anteriormente.

   `https://<emulator endpoint provided in response>/_explorer/index.html`

Si tiene una aplicación cliente de .NET que se ejecuta en un contenedor de Docker de Linux y ejecuta el emulador de Azure Cosmos DB en una máquina host, siga las instrucciones de la sección siguiente para importar el certificado en el contenedor de Docker de Linux.

## <a name="regenerate-the-emulator-certificates"></a>Regeneración de los certificados del emulador

Al ejecutar el emulador en un contenedor de Docker, los certificados asociados al emulador se vuelven a generar cada vez que se detiene y se reinicia el contenedor correspondiente. Debido a esto, tiene que volver a importar los certificados tras cada inicio del contenedor. Para evitar esta limitación, puede usar un archivo de Docker Compose para enlazar el contenedor de Docker a una dirección IP determinada y una imagen de contenedor.

Por ejemplo, puede usar la siguiente configuración en el archivo de Docker Compose. Asegúrese de darle formato según sus requisitos: 

```yml
version: '2.4' # Do not upgrade to 3.x yet, unless you plan to use swarm/docker stack: https://github.com/docker/compose/issues/4513

networks:
  default:
    external: false
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"

services:

  # First create a directory that will hold the emulator traces and certificate to be imported
  # set hostDirectory=C:\emulator\bind-mount
  # mkdir %hostDirectory%

  cosmosdb:
    container_name: "azurecosmosemulator"
    hostname: "azurecosmosemulator"
    image: 'mcr.microsoft.com/cosmosdb/windows/azure-cosmos-emulator'
    platform: windows
    tty: true
    mem_limit: 3GB
    ports:
        - '8081:8081'
        - '8900:8900'
        - '8901:8901'
        - '8902:8902'
        - '10250:10250'
        - '10251:10251'
        - '10252:10252'
        - '10253:10253'
        - '10254:10254'
        - '10255:10255'
        - '10256:10256'
        - '10350:10350'
    networks:
      default:
        ipv4_address: 172.16.238.246
    volumes:
        - '${hostDirectory}:C:\CosmosDB.Emulator\bind-mount'
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a usar el emulador local para el desarrollo gratuito de aplicaciones en un entorno local. Ahora puede avanzar a los siguientes artículos:

* [Exportación de los certificados del emulador de Azure Cosmos DB para su uso con aplicaciones en Java, Python y Node.js](local-emulator-export-ssl-certificates.md)
* [Uso de parámetros de línea de comandos y comandos de PowerShell para controlar el emulador](emulator-command-line-parameters.md)
* [Depuración de problemas con el emulador](troubleshoot-local-emulator.md)