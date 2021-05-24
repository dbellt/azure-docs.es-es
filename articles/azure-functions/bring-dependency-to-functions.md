---
title: Traslado de dependencias y bibliotecas de terceros a Azure Functions
description: Aprenda a trasladar archivos o bibliotecas de terceros.
ms.date: 4/6/2020
ms.topic: article
zone_pivot_groups: bring-third-party-dependency-programming-functions
ms.openlocfilehash: 3cd1139e8c733650acc879bf1d0d5d9efb2f6fb4
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109716440"
---
# <a name="bring-dependencies-or-third-party-library-to-azure-functions"></a>Traslado de dependencias o bibliotecas de terceros a Azure Functions

En este artículo, aprenderá a trasladar dependencias de terceros a las aplicaciones de funciones. Algunos ejemplos de dependencias de terceros son los archivos JSON, los archivos binarios y los modelos de aprendizaje automático. 

En este artículo aprenderá a:
> [!div class="checklist"]
> * Trasladar dependencias mediante un proyecto de código de Functions 
::: zone pivot="programming-language-python"
> [!div class="checklist"]
> * Trasladar dependencias mediante el montaje de un recurso compartido de archivos de Azure
::: zone-end

## <a name="bring-in-dependencies-from-the-project-directory"></a>Trasladar dependencias desde el directorio del proyecto
::: zone pivot="programming-language-python"
Una de las formas más sencillas de trasladar dependencias es colocar los archivos o artefactos junto con el código de la aplicación de funciones en la estructura de directorios del proyecto de Functions. Este es uno de los ejemplos de directorio de un proyecto de funciones de Python:
```
<project_root>/
 | - my_first_function/
 | | - __init__.py
 | | - function.json
 | | - example.py
 | - dependencies/
 | | - dependency1
 | - .funcignore
 | - host.json
 | - local.settings.json
```
Al colocar las dependencias en una carpeta dentro del directorio del proyecto de la aplicación de funciones, la carpeta de dependencias se implementará junto con el código. Como resultado, el código de función puede acceder a las dependencias en la nube mediante la API del sistema de archivos. 

### <a name="accessing-the-dependencies-in-your-code"></a>Acceso a las dependencias del código

En este ejemplo se accede y se ejecuta la dependencia ```ffmpeg``` que se encuentra en el directorio ```<project_root>/ffmpeg_lib```. 


```python
import logging

import azure.functions as func
import subprocess

FFMPEG_RELATIVE_PATH = "../ffmpeg_lib/ffmpeg"

def main(req: func.HttpRequest,
         context: func.Context) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    command = req.params.get('command')
    # If no command specified, set the command to help
    if not command:
        command = "-h"

    # context.function_directory returns the current directory in which functions is executed 
    ffmpeg_path = "/".join([str(context.function_directory), FFMPEG_RELATIVE_PATH])
    
    try:
        byte_output  = subprocess.check_output([ffmpeg_path, command])
        return func.HttpResponse(byte_output.decode('UTF-8').rstrip(),status_code=200)
    except Exception as e:
        return func.HttpResponse("Unexpected exception happened when executing ffmpeg. Error message:" + str(e),status_code=200)
```
>[!NOTE]
> Puede que tenga que usar `chmod` para proporcionar derechos `Execute` al binario ffmpeg en un entorno Linux.
::: zone-end

::: zone pivot="programming-language-java"
Una de las formas más sencillas de trasladar dependencias es colocar los archivos o artefactos junto con el código de la aplicación de funciones en la estructura de directorios del proyecto de Functions. Este es uno de los ejemplos de directorio de un proyecto de funciones de Java:
```
<project_root>/
 | - src/
 | | - main/java/com/function
 | | | - Function.java
 | | - test/java/com/function
 | - artifacts/
 | | - dependency1
 | - host.json
 | - local.settings.json
 | - pom.xml
```
En el caso concreto de Java, debe incluir específicamente los artefactos en la carpeta de creación o destino al copiar recursos. Este es un ejemplo de cómo se hace en Maven:

```xml
...
<execution>
    <id>copy-resources</id>
    <phase>package</phase>
    <goals>
        <goal>copy-resources</goal>
    </goals>
    <configuration>
        <overwrite>true</overwrite>
        <outputDirectory>${stagingDirectory}</outputDirectory>
        <resources>
            <resource>
                <directory>${project.basedir}</directory>
                <includes>
                    <include>host.json</include>
                    <include>local.settings.json</include>
                    <include>artifacts/**</include>
                </includes>
            </resource>
        </resources>
    </configuration>
</execution>
...
```
Al colocar las dependencias en una carpeta dentro del directorio del proyecto de la aplicación de funciones, la carpeta de dependencias se implementará junto con el código. Como resultado, el código de función puede acceder a las dependencias en la nube mediante la API del sistema de archivos. 

### <a name="accessing-the-dependencies-in-your-code"></a>Acceso a las dependencias del código

En este ejemplo se accede y se ejecuta la dependencia ```ffmpeg``` que se encuentra en el directorio ```<project_root>/ffmpeg_lib```. 


```java
public class Function {
    final static String BASE_PATH = "BASE_PATH";
    final static String FFMPEG_PATH = "/artifacts/ffmpeg/ffmpeg.exe";
    final static String HELP_FLAG = "-h";
    final static String COMMAND_QUERY = "command";

    @FunctionName("HttpExample")
    public HttpResponseMessage run(
            @HttpTrigger(
                name = "req",
                methods = {HttpMethod.GET, HttpMethod.POST},
                authLevel = AuthorizationLevel.ANONYMOUS)
                HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) throws IOException{
        context.getLogger().info("Java HTTP trigger processed a request.");
            
        // Parse query parameter
        String flags = request.getQueryParameters().get(COMMAND_QUERY);
        
        if (flags == null || flags.isBlank()) {
            flags = HELP_FLAG;
        }

        Runtime rt = Runtime.getRuntime();
        String[] commands = { System.getenv(BASE_PATH) + FFMPEG_PATH, flags};
        Process proc = rt.exec(commands);
        
        BufferedReader stdInput = new BufferedReader(new 
        InputStreamReader(proc.getInputStream()));
   
        String out = stdInput.lines().collect(Collectors.joining("\n"));
        if(out.isEmpty()) {
            BufferedReader stdError = new BufferedReader(new 
                InputStreamReader(proc.getErrorStream()));
            out = stdError.lines().collect(Collectors.joining("\n"));
        }
        return request.createResponseBuilder(HttpStatus.OK).body(out).build();

    }
```
>[!NOTE]
> Para que este fragmento de código funcione en Azure, debe especificar una configuración de la aplicación personalizada de "BASE_PATH" con el valor "/home/site/wwwroot".
::: zone-end


::: zone pivot="programming-language-python"
## <a name="bring-dependencies-by-mounting-a-file-share"></a>Traslado de dependencias mediante el montaje de un recurso compartido de archivos

Cuando ejecuta la aplicación de funciones en Linux, hay otra manera de trasladar las dependencias de terceros. Functions le permite montar un recurso compartido de archivos hospedado en Azure Files. Tenga en cuenta este enfoque si desea desacoplar dependencias o artefactos desde el código de la aplicación.

En primer lugar, tiene que crear una cuenta de Azure Storage. En la cuenta, también debe crear un recurso compartido de archivos en Azure Files. Para crear estos recursos, siga esta [guía](../storage/files/storage-how-to-use-files-portal.md).

Después de crear la cuenta de almacenamiento y el recurso compartido de archivos, use el comando [az webapp config storage-account add](/cli/azure/webapp/config/storage-account#az_webapp_config_storage_account_add) para adjuntar el recurso compartido de archivos a la aplicación de funciones, como se muestra en el ejemplo siguiente.

```console
az webapp config storage-account add \
  --name < Function-App-Name > \
  --resource-group < Resource-Group > \
  --subscription < Subscription-Id > \
  --custom-id < Unique-Custom-Id > \
  --storage-type AzureFiles \
  --account-name < Storage-Account-Name > \
  --share-name < File-Share-Name >  \
  --access-key < Storage-Account-AccessKey > \
  --mount-path </path/to/mount>
```



|       Marca    |  Value     |
| ------------- | ---------------------------------- | 
| custom-id      |  Cualquier cadena única |
| storage-type      |  Actualmente solo se admite Azure Files |
| share-name      |  Recurso compartido ya existente |
| mount-path     |   Ruta de acceso en la que se podrá acceder al recurso compartido dentro del contenedor. El valor debe tener el formato `/dir-name` y no puede empezar por `/home`. |

[Aquí](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-update) puede encontrar más comandos para modificar o eliminar la configuración del recurso compartido de archivos.


### <a name="uploading-the-dependencies-to-azure-files"></a>Carga de las dependencias en Azure Files

Una opción para cargar la dependencia en Azure Files es mediante Azure Portal. Consulte esta [guía](../storage/files/storage-how-to-use-files-portal.md#upload-a-file) para obtener instrucciones para cargar dependencias mediante el portal. Otras opciones para cargar las dependencias en Azure Files son mediante la [CLI de Azure](../storage/files/storage-how-to-use-files-cli.md#upload-a-file) y [PowerShell](../storage/files/storage-how-to-use-files-powershell.md#upload-a-file).


### <a name="accessing-the-dependencies-in-your-code"></a>Acceso a las dependencias del código

Después de cargar las dependencias en el recurso compartido de archivos, puede acceder a estas desde el código. El recurso compartido montado está disponible en la *ruta de acceso de montaje* especificada como, por ejemplo, ```/path/to/mount```. Puede acceder al directorio de destino mediante las API del sistema de archivos.

En el ejemplo siguiente se muestra un código de desencadenador HTTP que accede a la biblioteca `ffmpeg`, que está almacenada en un recurso compartido de archivos montado.

```python
import logging

import azure.functions as func
import subprocess 

FILE_SHARE_MOUNT_PATH = os.environ['FILE_SHARE_MOUNT_PATH']
FFMPEG = "ffmpeg"

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    command = req.params.get('command')
    # If no command specified, set the command to help
    if not command:
        command = "-h"

    try:
        byte_output  = subprocess.check_output(["/".join(FILE_SHARE_MOUNT_PATH, FFMPEG), command])
        return func.HttpResponse(byte_output.decode('UTF-8').rstrip(),status_code=200)
    except Exception as e:
        return func.HttpResponse("Unexpected exception happened when executing ffmpeg. Error message:" + str(e),status_code=200)
```

Al implementar este código en una aplicación de funciones de Azure, debe [crear una configuración de la aplicación](functions-how-to-use-azure-function-app-settings.md#settings) con un nombre de clave `FILE_SHARE_MOUNT_PATH` y un valor de la ruta de acceso del recurso compartido de archivos montado que, para este ejemplo es `/azure-files-share`. Para realizar la depuración local, debe rellenar `FILE_SHARE_MOUNT_PATH` con la ruta de acceso del archivo donde se almacenan las dependencias en la máquina local. Este es un ejemplo para establecer `FILE_SHARE_MOUNT_PATH` mediante `local.settings.json`:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "FILE_SHARE_MOUNT_PATH" : "PATH_TO_LOCAL_FFMPEG_DIR"
  }
}

```
::: zone-end



## <a name="next-steps"></a>Pasos siguientes

+ [Guía de Azure Functions para desarrolladores de Python](functions-reference-python.md)
+ [Guía de Azure Functions para desarrolladores de Java](functions-reference-java.md)
+ [Referencia para desarrolladores de Azure Functions](functions-reference.md)
