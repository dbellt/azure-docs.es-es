---
title: Solución de problemas de Live Video Analytics on IoT Edge - Azure
description: En este artículo se describen los pasos de solución de problemas para Live Video Analytics on IoT Edge.
author: IngridAtMicrosoft
ms.topic: how-to
ms.author: inhenkel
ms.date: 12/04/2020
ms.openlocfilehash: f806a700fa9f1788e2435a43d8fdcb19fa8134ce
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372660"
---
# <a name="troubleshoot-live-video-analytics-on-iot-edge"></a>Solución de problemas de Live Video Analytics on IoT Edge

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

En este artículo se describen los pasos de solución de problemas para Live Video Analytics on Azure IoT Edge.

## <a name="troubleshoot-deployment-issues"></a>Solución de problemas de implementación

### <a name="diagnostics"></a>Diagnóstico

Como parte de la implementación de Live Video Analytics, va a configurar recursos de Azure como IoT Hub y dispositivos IoT Edge. Como primer paso para diagnosticar problemas, siga estas instrucciones para asegurarse de que el dispositivo Edge esté bien configurado:

1. [Ejecute el comando `check`](../../iot-edge/troubleshoot.md#run-the-check-command).
1. [Compruebe la versión de IoT Edge](../../iot-edge/troubleshoot.md#check-your-iot-edge-version).
1. [Compruebe el estado del administrador de seguridad de IoT Edge y sus registros](../../iot-edge/troubleshoot.md#check-the-status-of-the-iot-edge-security-manager-and-its-logs).
1. [Vea los mensajes que se envían a través del centro de IoT Edge](../../iot-edge/troubleshoot.md#view-the-messages-going-through-the-iot-edge-hub).
1. [Reinicie los contenedores](../../iot-edge/troubleshoot.md#restart-containers).
1. [Compruebe las reglas de configuración de los puertos y el firewall](../../iot-edge/troubleshoot.md#check-your-firewall-and-port-configuration-rules).

### <a name="pre-deployment-issues"></a>Problemas anteriores a la implementación

Si la infraestructura perimetral está bien, puede comprobar si existen problemas con el archivo de manifiesto de implementación. Para implementar el módulo Live Video Analytics on IoT Edge en el dispositivo IoT Edge junto con cualquier otro módulo de IoT, puede usar un manifiesto de implementación que contenga el centro de IoT Edge, el agente de IoT Edge y otros módulos con sus propiedades. Puede usar el siguiente comando para implementar el archivo de manifiesto:

```
az iot edge set-modules --hub-name <iot-hub-name> --device-id lva-sample-device --content <path-to-deployment_manifest.json>
```
Si el código JSON no tiene el formato correcto, podría recibir el siguiente error:   
&nbsp;&nbsp;&nbsp; **No se pudo analizar el código JSON del archivo: "<deployment manifest.json>" para el argumento "content" con la excepción: "Datos adicionales: línea 101, columna 1 (char 5325)"**

Si se produce este error, se recomienda comprobar en el archivo JSON si faltan corchetes o existe algún otro problema con la estructura del archivo. Puede usar un cliente como [Notepad++ con el complemento del visor de JSON](https://riptutorial.com/notepadplusplus/example/18201/json-viewer) o una herramienta en línea como el [formateador y validador de JSON](https://jsonformatter.curiousconcept.com/) para validar la estructura de archivos.

### <a name="during-deployment-diagnose-with-media-graph-direct-methods"></a>Durante la implementación: Diagnóstico con los métodos directos de grafo multimedia 

Una vez implementado correctamente el módulo Live Video Analytics on IoT Edge en el dispositivo IoT Edge, puede crear y ejecutar el grafo multimedia invocando [métodos directos](direct-methods.md).  
>[!NOTE]
>  Las llamadas de método directo solo deben realizarse al módulo **`lvaEdge`** .

Puede usar Azure Portal para ejecutar el diagnóstico del grafo multimedia mediante métodos directos:

1. En Azure Portal, vaya al centro de IoT conectado al dispositivo IoT Edge.

1. Busque **Administración automática de dispositivos** y, después, seleccione **IoT Edge**.  

1. En la lista de dispositivos Edge, seleccione el dispositivo que desea diagnosticar.  
         
    ![Captura de pantalla de Azure Portal que muestra una lista de los dispositivos Edge.](./media/troubleshoot-how-to/lva-sample-device.png)


1. Compruebe si el código de respuesta es *200-OK*. Existen otros códigos de respuesta para el [entorno de ejecución de IoT Edge](../../iot-edge/iot-edge-runtime.md), entre los que se incluyen:
    * 400 - La configuración de implementación tiene un formato incorrecto o no es válida.
    * 417 - El dispositivo no tiene un conjunto de configuración de implementación.
    * 412 - La versión del esquema de la configuración de implementación no es válida.
    * 406 - El dispositivo de IoT Edge está sin conexión o no envía informes de estado.
    * 500 - Error en el entorno en tiempo de ejecución de Azure IoT Edge.

    > [!TIP]
    > Si experimenta problemas al ejecutar módulos de Azure IoT Edge en el entorno, use **[Pasos de diagnóstico estándar de Azure IoT Edge](../../iot-edge/troubleshoot.md?preserve-view=true&view=iotedge-2018-06)** como guía para la solución de problemas y el diagnóstico.
### <a name="post-deployment-direct-method-error-code"></a>Después de la implementación: Código de error de método directo
1. Si recibe un `501 code` de estado, compruebe si el nombre del método directo es adecuado. Si el nombre del método y la carga útil de la solicitud son adecuados, debería obtener resultados junto con el código 200 de ejecución correcta. 
1. Si la carga de la solicitud es incorrecta, obtendrá un estado `400 code` y una carga de respuesta que indica el código de error, así como un mensaje que debería resultar útil para diagnosticar el problema con la llamada de método directo.
    * La comprobación de las propiedades notificadas y deseadas puede ayudarle a entender si las propiedades del módulo se han sincronizado con la implementación. Si no es así, puede reiniciar el dispositivo IoT Edge. 
    * Use la guía de [métodos directos](direct-methods.md) para invocar algunos métodos, especialmente los sencillos, como GraphTopologyList. En la guía también se especifican las cargas de solicitud y respuesta esperadas, así como los códigos de error. Si los métodos directos sencillos funcionan correctamente, puede estar seguro de que el módulo Live Video Analytics on IoT Edge está operativo.
        
       ![Captura de pantalla del panel "Método directo" del módulo de IoT Edge.](./media/troubleshoot-how-to/direct-method.png) 

1. Si las columnas **Especificado en la implementación** y **Notificado por el dispositivo** indican *Sí*, puede invocar métodos directos en el módulo Live Video Analytics on IoT Edge. Seleccione el módulo para ir a una página en la que puede comprobar las propiedades deseadas y notificadas, así como invocar métodos directos. Tenga en cuenta lo siguiente: 

### <a name="post-deployment-diagnose-logs-for-issues-during-the-run"></a>Después de la implementación: Registros de diagnóstico de problemas durante la ejecución 

Los registros de contenedor del módulo de IoT Edge deben contener información de diagnóstico que ayude a depurar los problemas durante el tiempo de ejecución del módulo. Puede [comprobar si hay problemas en los registros del contenedor](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) y realizar un diagnóstico automático de estos. 

Si ha ejecutado todas las comprobaciones anteriores y todavía hay problemas, recopile los registros del dispositivo IoT Edge [con el comando `support bundle`](../../iot-edge/troubleshoot.md#gather-debug-information-with-support-bundle-command) para su posterior análisis por parte del equipo de Azure. Puede [ponerse en contacto](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) con nosotros para obtener soporte técnico y enviar los registros recopilados.

## <a name="common-error-resolutions"></a>Resoluciones de errores comunes

Live Video Analytics se implementa como un módulo de IoT Edge en el dispositivo IoT Edge y funciona en colaboración con el agente de IoT Edge y los módulos del centro de conectividad. Algunos de los errores comunes que se encontrará con la implementación de Live Video Analytics se deben a los problemas con la infraestructura de IoT subyacente. Entre los errores se incluyen:

* [El agente de IoT Edge se detiene después de, aproximadamente, un minuto](../../iot-edge/troubleshoot-common-errors.md#iot-edge-agent-stops-after-about-a-minute).
* [El agente de IoT Edge no puede acceder a la imagen de un módulo (403)](../../iot-edge/troubleshoot-common-errors.md#iot-edge-agent-cant-access-a-modules-image-403).
* [El módulo Agente de IoT Edge notifica "archivo de configuración vacío" y no se inicia ningún módulo en el dispositivo](../../iot-edge/troubleshoot-common-errors.md#edge-agent-module-reports-empty-config-file-and-no-modules-start-on-the-device).
* [No se puede iniciar el centro de IoT Edge](../../iot-edge/troubleshoot-common-errors.md#iot-edge-hub-fails-to-start).
* [Error de nombre de host no válido del demonio de seguridad de IoT Edge](../../iot-edge/troubleshoot-common-errors.md#iot-edge-security-daemon-fails-with-an-invalid-hostname).
* [Live Video Analytics o cualquier otro módulo de IoT Edge personalizado no puede enviar un mensaje al centro de IoT Edge con el error 404](../../iot-edge/troubleshoot-common-errors.md#iot-edge-module-fails-to-send-a-message-to-edgehub-with-404-error).
* [El módulo de IoT Edge se implementa correctamente y, a continuación, desaparece del dispositivo](../../iot-edge/troubleshoot-common-errors.md#iot-edge-module-deploys-successfully-then-disappears-from-device).

    > [!TIP]
    > Si experimenta problemas al ejecutar módulos de Azure IoT Edge en el entorno, use **[Pasos de diagnóstico estándar de Azure IoT Edge](../../iot-edge/troubleshoot.md?preserve-view=true&view=iotedge-2018-06)** como guía para la solución de problemas y el diagnóstico.

También podría encontrar problemas al ejecutar el **[script de configuración de los recursos de Live Video Analytics](https://github.com/Azure/live-video-analytics/tree/master/edge/setup)** . Algunos problemas comunes son:

* El uso de una suscripción en la que no tenga privilegios de propietario. Esto hará que en el script se produzca el error **ForbiddenError** o **AuthorizationFailed**.
    * Para solucionar este problema, asegúrese de tener privilegios de **PROPIETARIO** sobre la suscripción que planea usar. Si no puede hacerlo por sí mismo, póngase en contacto con el administrador de suscripciones para conceder los privilegios correctos.
* **Se produjo un error en la implementación de la plantilla debido a la infracción de la directiva.**
    * Para solucionar este problema, trabaje con el administrador de TI para que las llamadas para crear máquinas virtuales pasen por alto el bloqueo de la autenticación de SSH. Esto no será necesario, dado que se usa una red Bastion que requiere un nombre de usuario y una contraseña para comunicarse con los recursos de Azure. Estas credenciales se almacenarán en el archivo **~/clouddrive/lva-sample/vm-edge-device-credentials.txt** de Cloud Shell, una vez que la máquina virtual se haya creado, implementado y asociado correctamente al centro de IoT.
* El script de instalación no puede crear una entidad de servicio o recursos de Azure.
    * Para solucionar este problema, compruebe que la suscripción y el inquilino de Azure no hayan alcanzado sus límites de servicio máximos. Más información sobre [Restricciones y límites del servicio Azure AD](../../active-directory/enterprise-users/directory-service-limits-restrictions.md) y [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).

> [!TIP]
> Si hay algún problema adicional con el que pueda necesitar ayuda, consulte **[Recopilación de registros para enviar una incidencia de soporte técnico](#collect-logs-for-submitting-a-support-ticket)** . También puede ponerse en contacto con nosotros enviándonos un correo electrónico a **[amshelp@microsoft.com](mailto:amshelp@microsoft.com)** .
### <a name="live-video-analytics-working-with-external-modules"></a>Live Video Analytics trabajando con módulos externos

Live Video Analytics, mediante el procesador de extensiones del grafo multimedia, puede ampliar el grafo multimedia para enviar y recibir datos de otros módulos de IoT Edge mediante HTTP o protocolos gRPC. Como [ejemplo específico](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/httpExtension), este grafo multimedia puede enviar fotogramas de vídeo como imágenes a un módulo de inferencia externo, como Yolo v3, y recibir resultados de análisis basados en JSON mediante el protocolo HTTP. En esta topología, el destino final de los eventos es, principalmente, el centro de IoT. En situaciones en las que no vea los eventos de inferencia en el centro de conectividad, compruebe lo siguiente:

* Compruebe si el centro de conectividad en el que publica el grafo multimedia es el mismo que está examinando. Cuando crea varias implementaciones, puede que haya muchos centros de conectividad y podría estar comprobando el centro incorrecto para los eventos.
* En Azure Portal, compruebe si el módulo externo está implementado y en ejecución. En la imagen de ejemplo que se muestra aquí, rtspsim, yolov3, tinyyolov3 y logAnalyticsAgent son módulos de IoT Edge que se ejecutan fuera del módulo lvaEdge.

    [ ![Captura de pantalla que muestra el estado de ejecución de los módulos en Azure IoT Hub.](./media/troubleshoot-how-to/iot-hub-azure.png) ](./media/troubleshoot-how-to/iot-hub-azure.png#lightbox)

* Compruebe que está enviando eventos al punto de conexión de URL correcto. El contenedor de inteligencia artificial externo expone una dirección URL y un puerto a través del cual recibe y devuelve los datos de las solicitudes POST. Esta dirección URL se especifica como una propiedad `endpoint: url` para el procesador de extensión HTTP. Como se aprecia en la [dirección URL de la topología](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/httpExtension/2.0/topology.json), el punto de conexión se establece en el parámetro URL de inferencia. Asegúrese de que el valor predeterminado del parámetro o el valor pasado son correctos. Puede realizar una prueba para ver si funciona mediante una dirección URL del cliente (cURL).  

    Por ejemplo, este es un contenedor de Yolo v3 que se ejecuta en una máquina local con esta dirección IP: 172.17.0.3.  
    
    ```
    curl -X POST http://172.17.0.3/score -H "Content-Type: image/jpeg" --data-binary @<fullpath to jpg>
    ```

    Resultado que se devuelve:

    ```
    {"inferences": [{"type": "entity", "entity": {"tag": {"value": "car", "confidence": 0.8668569922447205}, "box": {"l": 0.3853073438008626, "t": 0.6063712999658677, "w": 0.04174524943033854, "h": 0.02989496027381675}}}]}
    ```
    > [!TIP]
    > Utilice el **[comando docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/)** para buscar la dirección IP de la máquina.
    
* Si está ejecutando una o varias instancias de un grafo que usa el procesador de extensiones de grafos multimedia, debe usar el campo `samplingOptions` para administrar los fotogramas por segundo (FPS) de la fuente de vídeo. 

   * En determinadas situaciones en las que la CPU o la memoria de la máquina perimetral tienen un uso muy elevado, puede perder ciertos eventos de inferencia. Para solucionar este problema, establezca un valor bajo para la propiedad `maximumSamplesPerSecond` del campo `samplingOptions`. Puede establecerlo en 0,5 ("maximumSamplesPerSecond": "0,5") en cada instancia del grafo y volver a ejecutar la instancia para obtener los eventos de inferencia del centro de conectividad.
    
### <a name="multiple-direct-methods-in-parallel--timeout-failure"></a>Varios métodos directos en paralelo: error de tiempo de espera 

Live Video Analytics on IoT Edge proporciona un modelo de programación basado en métodos directos que permite configurar varias topologías y varias instancias de grafos. Como parte de la configuración de la topología y el grafo, puede invocar varias llamadas de métodos directos en el módulo de IoT Edge. Si invoca estas llamadas de métodos múltiples en paralelo, especialmente las que inician y detienen los grafos, puede experimentar un error de tiempo de espera como el siguiente: 

Assembly Initialization method Microsoft.Media.LiveVideoAnalytics.Test.Feature.Edge.AssemblyInitializer.InitializeAssemblyAsync threw exception. Microsoft.Azure.Devices.Common.Exceptions.IotHubException: Microsoft.Azure.Devices.Common.Exceptions.IotHubException:<br/> `{"Message":"{\"errorCode\":504101,\"trackingId\":\"55b1d7845498428593c2738d94442607-G:32-TimeStamp:05/15/2020 20:43:10-G:10-TimeStamp:05/15/2020 20:43:10\",\"message\":\"Timed out waiting for the response from device.\",\"info\":{},\"timestampUtc\":\"2020-05-15T20:43:10.3899553Z\"}","ExceptionMessage":""}. Aborting test execution. `

Se recomienda *no* llamar a métodos directos en paralelo. Llámelos de manera secuencial (es decir, realice una llamada de método directo solo después de que finalice la anterior).

### <a name="collect-logs-for-submitting-a-support-ticket"></a>Recopilación de registros para enviar una incidencia de soporte técnico

Si los pasos de solución de problemas autoguiados no resuelven el problema, debe ir a Azure Portal y [abrir una incidencia de soporte técnico](../../azure-portal/supportability/how-to-create-azure-support-request.md).

> [!WARNING]
> Los registros pueden contener información de identificación personal (PII) como su dirección IP. Se eliminarán todas las copias locales de los registros en cuanto terminemos de examinarlos y cerremos la incidencia de soporte técnico.  

Para recopilar los registros significativos que se deben agregar al vale, siga las instrucciones que se indican a continuación en orden y cargue los archivos de registro en el panel de **detalles** de la solicitud de soporte técnico.  
1. [Configure el módulo de Live Video Analytics para que recopile registros detallados](#configure-live-video-analytics-module-to-collect-verbose-logs).
1. [Active los registros de depuración](#live-video-analytics-debug-logs).
1. Reproduzca el problema
1. Conéctese a la máquina virtual desde la página **IoT Hub** del portal.
    1. Comprima todos los archivos de la carpeta *debugLogs*.

       > [!NOTE]
       > Estos archivos de registro no están diseñados para el autodiagnóstico, sino para que el equipo de ingeniería de Azure analice sus problemas.

       * En el siguiente comando, asegúrese de reemplazar **$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE** por la ubicación de los registros de depuración en el dispositivo Edge que configuró anteriormente el **paso 2**.  

           ```
           sudo apt install zip unzip  
           zip -r debugLogs.zip $DEBUG_LOG_LOCATION_ON_EDGE_DEVICE 
           ```

    1. Adjunte el archivo *debugLogs.zip* a la incidencia de soporte técnico.
1. Ejecute el [comando support bundle](#use-the-support-bundle-command), recopile los registros y adjúntelos a la incidencia de soporte técnico.

### <a name="configure-live-video-analytics-module-to-collect-verbose-logs"></a>Configuración del módulo de Live Video Analytics para que recopile registros detallados
Configure el módulo de Live Video Analytics para que recopile registros detallados estableciendo `logLevel` y `logCategories` de la siguiente manera:
```
"logLevel": "Verbose",
"logCategories": "Application,Events,MediaPipeline",
```

Esto se puede hacer:
* En **Azure Portal**, actualizando las propiedades de Identidad de módulo gemela en el módulo de Live Video Analytics. [ ![Propiedades de Identidad de módulo gemela.](media/troubleshoot-how-to/module-twin.png)](media/troubleshoot-how-to/module-twin.png#lightbox)    
* O bien, en el archivo de **manifiesto de implementación**, puede agregar estas entradas en el nodo de propiedades del módulo de Live Video Analytics.

### <a name="use-the-support-bundle-command"></a>Uso del comando support-bundle

Cuando tenga que recopilar registros de un dispositivo IoT Edge, la manera más sencilla es usar el comando `support-bundle`. Este comando recopila:

- Registros del módulo
- Administrador de seguridad de IoT Edge y registros del motor de contenedor
- Salidas JSON de comprobación de IoT Edge
- Información de depuración útil

1. Ejecute el comando `support-bundle` con la marca *--since* para especificar cuánto tiempo desea que abarquen los registros. Por ejemplo, 2 h obtendrá los registros de las últimas dos horas. Puede cambiar el valor de esta marca para incluir los registros de un período diferente.

    ```
    sudo iotedge support-bundle --since 2h
    ```

   Este comando crea un archivo denominado *support_bundle.zip* en el directorio donde se ejecutó el comando. 
   
1. Adjunte el archivo *support_bundle.zip* a la incidencia de soporte técnico.

### <a name="live-video-analytics-debug-logs"></a>Registros de depuración de Live Video Analytics

Para configurar el módulo de Live Video Analytics on IoT Edge para que genere registros de depuración, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al centro de IoT.
1. En el panel izquierdo, seleccione **IoT Edge**.
1. Seleccione el identificador del dispositivo de destino en la lista de dispositivos.
1. En la parte superior del panel, seleccione **Establecer módulos**.

   ![Captura de pantalla del botón "Establecer módulos" en Azure Portal.](media/troubleshoot-how-to/set-modules.png)

1. En la sección **módulos de IoT Edge**, busque y seleccione **lvaEdge**.
1. Seleccione **Opciones de creación del contenedor**.
1. En la sección **Enlaces**, agregue el comando siguiente:

    `/var/local/mediaservices/logs:/var/lib/azuremediaservices/logs`

    > [!NOTE] 
    > Este comando enlaza las carpetas de registros entre el dispositivo Edge y el contenedor. Si desea recopilar los registros en una ubicación diferente, use el comando siguiente, reemplazando **$LOG_LOCATION_ON_EDGE_DEVICE** por la ubicación que desea usar: `/var/$LOG_LOCATION_ON_EDGE_DEVICE:/var/lib/azuremediaservices/logs`

1. Seleccione **Actualizar**.
1. Seleccione **Revisar + crear**. Se publica un mensaje de validación correcto en un banner verde.
1. Seleccione **Crear**.
1. Actualice **Identidad de módulo gemela** para que apunte al parámetro DebugLogsDirectory a fin de seleccionar el directorio en el que se recopilarán los registros:

    a. En la tabla **Módulos**, seleccione **IvaEdge**.  
    b. En la parte superior del panel, seleccione **Identidad de módulo gemela**. Se abre un panel editable.  
    c. Agregue el siguiente par clave-valor en la **clave que desee**:  
    `"DebugLogsDirectory": "/var/lib/azuremediaservices/logs"`

    > [!NOTE] 
    > Este comando enlaza las carpetas de registros entre el dispositivo Edge y el contenedor. Si quiere recopilar los registros en una ubicación diferente en el dispositivo:
    > 1. Cree un enlace para la ubicación del registro de depuración en la sección **Enlaces**, y reemplace los elementos **$DEBUG _LOG_LOCATION_ON_EDGE_DEVICE** y **$DEBUG_LOG_LOCATION** por la ubicación que desee: `/var/$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE:/var/$DEBUG_LOG_LOCATION`
    > 2. Use el comando siguiente y reemplace **$DEBUG _LOG_LOCATION** por la ubicación usada en el paso anterior:  
    > `"DebugLogsDirectory": "/var/$DEBUG_LOG_LOCATION"`  
    
    d. Seleccione **Guardar**.


1. La recopilación de registros se puede detener estableciendo ese valor en **Identidad de módulo gemela** en *NULL*. Vuelva a la página **Identidad de módulo gemela** y actualice los siguientes parámetros como:

    `"DebugLogsDirectory": ""`

### <a name="best-practices-around-logging"></a>Procedimientos recomendados sobre el registro

La [supervisión y el registro](monitoring-logging.md) le ayudarán a entender la taxonomía y a generar registros que le ayudarán a depurar problemas de Live Video Analytics. 

Dado que la implementación del servidor de gRPC es diferente según el idioma, no hay ninguna manera estándar de agregar el registro dentro del servidor.  

Por ejemplo, si crea un servidor de gRPC con .NET Core, el servicio gRPC agregará registros en la categoría **Grpc**. Para habilitar registros detallados de gRPC, configure los prefijos de gRPC en el nivel de depuración del archivo appsettings.json agregando los siguientes elementos a la subsección LogLevel del registro: 

```
{ 
  "Logging": { 
    "LogLevel": { 
      "Default": "Debug", 
      "System": "Information", 
      "Microsoft": "Information", 
      "Grpc": "Debug" 
       } 
  } 
} 
``` 

También puede configurarlos en el archivo Startup.cs con ConfigureLogging: 

```
public static IHostBuilder CreateHostBuilder(string[] args) => 
    Host.CreateDefaultBuilder(args) 
        .ConfigureLogging(logging => 
        { 

           logging.AddFilter("Grpc", LogLevel.Debug); 
        }) 
        .ConfigureWebHostDefaults(webBuilder => 
        { 
            webBuilder.UseStartup<Startup>(); 
        }); 

``` 

[Registro y diagnóstico en gRPC en .NET](/aspnet/core/grpc/diagnostics?preserve-view=true&view=aspnetcore-3.1) proporciona instrucciones para recopilar algunos registros de diagnóstico de un servidor de gRPC. 

### <a name="a-failed-grpc-connection"></a>Una conexión de gRPC con errores 

Si un grafo está activo y haciendo streaming desde una cámara, Live Video Analytics mantendrá la conexión. 

### <a name="monitoring-and-balancing-the-load-of-cpu-and-gpu-resources-when-these-resources-become-bottlenecks"></a>Supervisión y equilibrio de la carga de los recursos de CPU y GPU cuando estos recursos se convierten en cuellos de botella

Live Video Analytics no supervisa ni proporciona ninguna supervisión de recursos de hardware. Los desarrolladores tendrán que usar las soluciones de supervisión de los fabricantes de hardware. Sin embargo, si usa contenedores de Kubernetes, puede supervisar el dispositivo mediante el [panel de Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). 

Los documentos de gRPC en .NET Core también comparten información valiosa sobre [procedimientos recomendados de rendimiento](/aspnet/core/grpc/performance?preserve-view=true&view=aspnetcore-3.1) y [equilibrio de carga](/aspnet/core/grpc/performance?preserve-view=true&view=aspnetcore-3.1#load-balancing).  

### <a name="troubleshooting-an-inference-server-when-it-does-not-receive-any-frames-and-you-are-receiving-an-unknown-protocol-error"></a>Solución de problemas de un servidor de inferencia cuando no recibe ningún fotograma y aparece un error de protocolo "desconocido" 

Hay varias cosas que puede hacer para obtener más información sobre el problema.  

* Incluya la categoría de registro "**ediaPipeline**" en las propiedades deseadas del módulo de Live Video Analytics y asegúrese de que el nivel de registro esté establecido en `Information`.  
* Para probar la conectividad de red, puede ejecutar el siguiente comando desde el dispositivo perimetral. 

   ```
   sudo docker exec lvaEdge /bin/bash -c “apt update; apt install -y telnet; telnet <inference-host> <inference-port>” 
   ```

   Si el comando genera una cadena corta de texto desordenado, significa que Telnet pudo abrir correctamente una conexión con el servidor de inferencia y un canal gRPC binario. Si no aparece, Telnet notificará un error de red. 
* En el servidor de inferencia puede habilitar registros adicionales en la biblioteca de gRPC. Esto puede proporcionar información adicional sobre el propio canal de gRPC. Hacer esto puede variar según el lenguaje. Aquí se proporcionan instrucciones para [C#](/aspnet/core/grpc/diagnostics?preserve-view=true&view=aspnetcore-3.1). 

### <a name="picking-more-images-from-buffer-of-grpc-without-sending-back-result-for-first-buffer"></a>Selección de más imágenes desde el búfer de gRPC sin devolver el resultado del primer búfer

Como parte del contrato de transferencia de datos de gRPC, se deben confirmar todos los mensajes que Live Video Analytics envía al servidor de inferencia de gRPC. Si no se confirma la recepción de un fotograma de imagen, se interrumpe el contrato de datos y se pueden producir situaciones no deseadas.  

Para usar el servidor de gRPC con Live Video Analytics, se puede usar la memoria compartida para obtener el mejor rendimiento. Esto requiere usar las funcionalidades de memoria compartida de Linux expuestas por el entorno o lenguaje de programación. 

1. Abra el identificador de memoria compartida de Linux.
1. Tras la recepción de un fotograma, acceda al desplazamiento de dirección dentro de la memoria compartida.
1. Confirme que el procesamiento del fotograma se realizó correctamente para que Live Video Analytics pueda reclamar su memoria.

   > [!NOTE]
   > Si se retrasa al confirmar la recepción del fotograma en Live Video Analytics durante mucho tiempo, puede dar lugar a que la memoria compartida se llene y provoque caídas de datos.
1. Almacene cada fotograma en una estructura de datos de su elección (lista, matriz, etc.) en el servidor de inferencia.
1. Después, puede ejecutar la lógica de procesamiento cuando tenga el número deseado de fotogramas de imagen.
1. Devuelva el resultado de la inferencia a Live Video Analytics cuando esté listo.

## <a name="next-steps"></a>Pasos siguientes

[Tutorial: Grabación de vídeo basada en eventos en la nube y reproducción desde la nube](event-based-video-recording-tutorial.md)