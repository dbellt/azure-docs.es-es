---
title: Ingesta de datos de OPC UA con Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Pasos para obtener los datos de Azure OPC UA en Azure Digital Twins
author: danhellem
ms.author: dahellem
ms.date: 5/20/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: db195f845a9e2f19108b0e40d569d76939dd6b6b
ms.sourcegitcommit: eb20dcc97827ef255cb4ab2131a39b8cebe21258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111373141"
---
# <a name="ingesting-opc-ua-data-with-azure-digital-twins"></a>Ingesta de datos de OPC UA con Azure Digital Twins

La plataforma [OPC Unified Architecture (OPC UA)](https://opcfoundation.org/about/opc-technologies/opc-ua/) es una arquitectura que no depende de plataformas, orientada a servicios y pensada para el sector de la fabricación. Se usa para obtener datos de telemetría de los dispositivos. 

Para que los datos del servidor de OPC UA se transfieran a Azure Digital Twins se necesitan varios componentes instalados en distintos dispositivos, así como algunos valores y código personalizables que se deben configurar. 

En este artículo se muestra cómo conectar todos estos componentes para transferir los nodos de OPC UA en Azure Digital Twins. Puede basarse en esta guía para crear sus propias soluciones.

> [!NOTE]
> En este artículo no se aborda la conversión de nodos de OPC UA en DTDL. Solo se trata la obtención de datos de telemetría del servidor de OPC UA en Azure Digital Twins. Si está interesado en generar modelos DTDL a partir de datos de OPC UA, visite los repositorios [UANodeSetWebViewer](https://github.com/barnstee/UANodesetWebViewer) y [OPCUA2DTDL](https://github.com/khilscher/OPCUA2DTDL).

## <a name="prerequisites"></a>Prerrequisitos

Antes de seguir con este artículo, complete los siguientes requisitos previos:
* **Descargar el repositorio de ejemplo**: en este artículo se usa un archivo de [modelos DTDL](concepts-models.md) y un cuerpo de función de Azure desde [OPC UA al repositorio de GitHub de Azure Digital Twins](https://github.com/Azure-Samples/opcua-to-azure-digital-twins). Empiece por descargar el repositorio de ejemplo en la máquina. Puede seleccionar el botón **Código** del repositorio para clonar el repositorio o descargarlo como un archivo .zip en la máquina.

    :::image type="content" source="media/how-to-ingest-opcua-data/download-repo.png" alt-text="Captura de pantalla del repositorio digital-twins-samples en GitHub, donde están resaltados los pasos para clonar o descargar el código." lightbox="media/how-to-ingest-opcua-data/download-repo.png":::
    
    Si descarga el repositorio como un archivo .zip, asegúrese de descomprimirlo y extraer los archivos.
* **Descargar Visual Studio**: en este artículo se utiliza Visual Studio para publicar una función de Azure. Puede descargar la versión más reciente de Visual Studio desde [Descargas de Visual Studio](https://visualstudio.microsoft.com/downloads/).

## <a name="architecture"></a>Architecture

Estos son los componentes que se incluirán en esta solución.

 :::image type="content" source="media/how-to-ingest-opcua-data/opcua-to-adt-diagram-1.png" alt-text="Esquema de la arquitectura de OPC UA en Azure Digital Twins" lightbox="media/how-to-ingest-opcua-data/opcua-to-adt-diagram-1.png":::    

| Componente | Descripción |
| --- | --- |
| Servidor OPC UA | Servidor OPC UA de [ProSys](https://www.prosysopc.com/products/opc-ua-simulation-server/) o [Kepware](https://www.kepware.com/en-us/products/#KEPServerEX) para simular los datos de OPC UA. |
| [Azure IoT Edge](../iot-edge/about-iot-edge.md) | IoT Edge es un servicio de IoT Hub que se instala en un dispositivo de puerta de enlace de Linux local. Es necesario que el módulo OPC Publisher se ejecute y envíe datos a IoT Hub. |
| [OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher) | Se trata de un módulo de IoT Edge que ha creado el equipo de Azure Industrial IoT. Este módulo se conecta al servidor OPC UA y envía los datos del nodo a Azure IoT Hub. |
| [Azure IoT Hub](../iot-hub/about-iot-hub.md) | OPC Publisher envía los datos de telemetría de OPC UA a Azure IoT Hub. Después, puede procesar los datos mediante una función de Azure y en Azure Digital Twins. |
| Azure Digital Twins | La plataforma que le permite crear una representación digital de cosas, lugares, procesos empresariales y personas reales. |
| [Función de Azure](../azure-functions/functions-overview.md) | Se usa una función personalizada de Azure para procesar los datos de telemetría que se transfieren desde Azure IoT Hub a los gemelos y propiedades adecuados de Azure Digital Twins. |

## <a name="set-up-edge-components"></a>Configuración de los componentes perimetrales

El primer paso es configurar los dispositivos y el software en el perímetro. Estos son los componentes perimetrales que configurará, en este orden:
1. [Servidor de simulación de OPC UA](#set-up-opc-ua-server)
1. [Dispositivo IoT Hub e IoT Edge](#set-up-iot-edge-device)
1. [Dispositivo de puerta de enlace](#set-up-gateway-device)

En esta sección se describe una breve configuración de cada uno. 

Para obtener información más detallada sobre cómo instalar cada uno de estos componentes, consulte los siguientes recursos:
* [Guía paso a paso para la instalación de OPC Publisher en Azure IoT Edge](https://www.linkedin.com/pulse/step-by-step-guide-installing-opc-publisher-azure-iot-kevin-hilscher) 
* [Instalación de IoT Edge en Linux](../iot-edge/how-to-install-iot-edge.md) 
* [OPC Publisher en GitHub](https://github.com/Azure/iot-edge-opc-publisher)
* [Configuración de OPC Publisher](../iot-accelerators/howto-opc-publisher-configure.md)

### <a name="set-up-opc-ua-server"></a>Configuración del servidor OPC UA

Para este artículo, no necesita acceso a dispositivos físicos que ejecuten un servidor OPC UA. En su lugar, puede instalar la versión gratuita de [Prosys OPC UA Simulation Server](https://www.prosysopc.com/products/opc-ua-simulation-server/) en una máquina virtual Windows para generar datos de OPC UA. En esta sección se indica cómo hacerlo.

Si ya tiene un dispositivo OPC UA físico u otro servidor de simulación de OPC UA que le gustaría usar, puede pasar a la sección siguiente, [Configuración del dispositivo IoT Edge](#set-up-iot-edge-device).

#### <a name="create-windows-10-virtual-machine"></a>Creación de una máquina virtual Windows 10

El software de Prosys requiere un recurso virtual simple. Mediante [Azure Portal](https://portal.azure.com), [cree una máquina virtual Windows 10](../virtual-machines/windows/quick-create-portal.md) con las especificaciones siguientes:
* **Opciones de disponibilidad**: no se requiere redundancia de la infraestructura.
* **Imagen**: Windows 10 Pro, versión 2004 - Gen2
* **Tamaño**: Standard_B1s - 1 vcpu, 1 GiB de memoria

:::image type="content" source="media/how-to-ingest-opcua-data/create-windows-virtual-machine-1.png" alt-text="Captura de pantalla de Azure Portal, en la que se muestra la pestaña Aspectos básicos de la configuración de la máquina virtual Windows." lightbox="media/how-to-ingest-opcua-data/create-windows-virtual-machine-1.png":::

La máquina virtual debe ser accesible a través de Internet. Para simplificar este tutorial, puede abrir todos los puertos y asignar a la máquina virtual una dirección IP pública. Esto se hace en la pestaña **Redes** de la configuración de la máquina virtual.

:::image type="content" source="media/how-to-ingest-opcua-data/create-windows-virtual-machine-2.png" alt-text="Captura de pantalla de Azure Portal, en la que se muestra la pestaña Redes de la configuración de la máquina virtual Windows.":::

> [!WARNING]
> No se recomienda abrir todos los puertos a Internet de las soluciones de producción, ya que esto puede suponer un riesgo de seguridad. Es posible que quiera considerar otras estrategias de seguridad para su entorno.

Recopile el valor de **dirección IP pública** que se usará en el paso siguiente.
 
Termine la configuración de la máquina virtual.

#### <a name="install-opc-ua-simulation-software"></a>Instalación del software de simulación de OPC UA

En la nueva máquina virtual Windows, instale [Prosys OPC UA Simulation Server](https://www.prosysopc.com/products/opc-ua-simulation-server/).

Una vez completada la descarga e instalación, inicie el servidor. El servidor de OPC UA puede tardar unos minutos en iniciarse. Una vez que esté listo, el estado del servidor debe aparecer como **En ejecución**.

A continuación, copie el valor de **dirección de conexión (UA TCP).** Péguelo en un lugar seguro para usarlo más adelante. En el valor de dirección pegado, reemplace la parte del nombre de la máquina por la **dirección IP pública** de la máquina virtual anterior, de la siguiente forma: 

`opc.tcp://<ip-address>:53530/OPCUA/SimulationServer`

Utilizará este valor actualizado más adelante en este artículo.

Por último, para ver los nodos de simulación que se proporcionan de forma predeterminada con el servidor, seleccione la pestaña **Objetos** y expanda las carpetas Objects::FolderType y Simulation::FolderType. Verá los nodos de simulación, cada uno con su propio valor `NodeId` único. 

Capture los valores `NodeId` de los nodos simulados que desea publicar. Necesitará estos identificadores más adelante en el artículo para simular datos de estos nodos.

> [!TIP]
> Para comprobar que el servidor OPC UA es accesible siga los pasos que se indican en "Comprobación de que el servicio OPC UA está en funcionamiento y es accesible" en la [guía paso a paso para la instalación de OPC Publisher en Azure IoT Edge](https://www.linkedin.com/pulse/step-by-step-guide-installing-opc-publisher-azure-iot-kevin-hilscher).

#### <a name="verify-completion"></a>Comprobación de la finalización

En esta sección, configurará el servidor OPC UA para simular datos. Compruebe que ha completado la siguiente lista de comprobación:

> [!div class="checklist"]
> * El servidor de simulación de Prosys está configurado y en funcionamiento
> * Ha copiado la dirección de conexión de UA TCP (`opc.tcp://<ip-address>:53530/OPCUA/SimulationServer`)
> * Ha capturado la lista de valores de `NodeId` para los nodos simulados que desea publicar. 

### <a name="set-up-iot-edge-device"></a>Configuración de un dispositivo IoT Edge

En esta sección, configurará una instancia de IoT Hub y un dispositivo IoT Edge. 

En primer lugar, [cree una instancia de Azure IoT Hub](../iot-hub/iot-hub-create-through-portal.md). Para este artículo, puede crear una instancia en el nivel **F1 - Gratis**.

:::image type="content" source="media/how-to-ingest-opcua-data/iot-hub.png" alt-text="Captura de pantalla de Azure Portal que muestra las propiedades de una instancia de IoT Hub":::.

Una vez que haya creado la instancia de Azure IoT Hub, seleccione **IoT Edge** en el menú de navegación izquierdo de la instancia y seleccione **Agregar un dispositivo IoT Edge**.

:::image type="content" source="media/how-to-ingest-opcua-data/iot-edge-1.png" alt-text="Captura de pantalla de la incorporación de un dispositivo IoT Edge en Azure Portal":::.

Siga las indicaciones para crear un dispositivo. 

Una vez creado el dispositivo, copie el valor de **Cadena de conexión principal** o **Cadena de conexión secundaria**. Lo necesitará más adelante cuando configure el dispositivo IoT Edge.

:::image type="content" source="media/how-to-ingest-opcua-data/iot-edge-2.png" alt-text="Captura de pantalla de Azure Portal que muestra las cadenas de conexión del dispositivo IoT Edge.":::

#### <a name="verify-completion"></a>Comprobación de la finalización

En esta sección, configurará IoT Edge y IoT Hub como preparación para crear un dispositivo de puerta de enlace. Compruebe que ha completado la siguiente lista de comprobación:
> [!div class="checklist"]
> * Se ha creado la instancia de IoT Hub.
> * Se ha aprovisionado el dispositivo IoT Edge.

### <a name="set-up-gateway-device"></a>Configuración del dispositivo de puerta de enlace

Para transferir los datos del servidor OPC UA a IoT Hub, necesita un dispositivo que ejecute IoT Edge con el módulo OPC Publisher. Posteriormente, OPC Publisher escuchará las actualizaciones de nodos de OPC UA y publicará los datos de telemetría en IoT Hub en formato JSON.

#### <a name="create-ubuntu-server-virtual-machine"></a>Creación de una máquina virtual de Ubuntu Server

Mediante [Azure Portal](https://portal.azure.com), cree una máquina virtual de Ubuntu Server con las especificaciones siguientes:
* **Opciones de disponibilidad**: no se requiere redundancia de la infraestructura.
* **Imagen**: Ubuntu Server 18.04 LTS: Gen1.
* **Tamaño**: Standard_B1ms - 1 vcpu, 2 GiB de memoria
    - El tamaño predeterminado (Standard_b1s - vcpu, 1 GiB de memoria) es demasiado lento para RDP. La actualización a una memoria de 2 GiB proporcionará una mejor experiencia de RDP.

:::image type="content" source="media/how-to-ingest-opcua-data/ubuntu-virtual-machine.png" alt-text="Captura de pantalla de Azure Portal que muestra la configuración de la máquina virtual Ubuntu":::.

> [!NOTE]
> Si decide usar RDP en la máquina virtual Ubuntu, puede seguir las instrucciones de [Instalación y configuración de xrdp para usar Escritorio remoto con Ubuntu](../virtual-machines/linux/use-remote-desktop.md).

#### <a name="install-iot-edge-container"></a>Instalación del contenedor de IoT Edge

Siga las instrucciones de [Instalación de IoT Edge en Linux](../virtual-machines/linux/use-remote-desktop.md).

Una vez completada la instalación, ejecute el siguiente comando para comprobar el estado de la instalación:

```bash
admin@gateway:~$ sudo iotedge check
```

Este comando ejecutará varias pruebas para asegurarse de que la instalación esté lista para comenzar.

#### <a name="install-opc-publisher-module"></a>Instalación del módulo OPC Publisher

A continuación, instale el módulo OPC Publisher en el dispositivo de puerta de enlace. 

Empiece por obtener el módulo de [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft_iot.iotedge-opc-publisher).

:::image type="content" source="media/how-to-ingest-opcua-data/opc-publisher-1.png" alt-text="Captura de pantalla de OPC Publisher en Azure Marketplace":::.

A continuación, siga los pasos de instalación documentados en el [repositorio de GitHub de OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher) para instalar el módulo en la máquina virtual Ubuntu.

En el paso para [especificar las opciones de creación del contenedor](https://github.com/Azure/iot-edge-opc-publisher#specifying-container-create-options-in-the-azure-portal), asegúrese de agregar el siguiente json:

```JSON
{
    "Hostname": "opcpublisher",
    "Cmd": [
        "--pf=/appdata/publishednodes.json",
        "--aa"
    ],
    "HostConfig": {
        "Binds": [
            "/iiotedge:/appdata"
        ]
    }
}
```

:::image type="content" source="media/how-to-ingest-opcua-data/opc-publisher-2.png" alt-text="Captura de pantalla de las opciones de creación del contenedor de OPC Publisher.":::

>[!NOTE]
>Las opciones de creación anteriores deberían funcionar en la mayoría de los casos sin necesidad de realizar ningún cambio, pero si va a usar su propio dispositivo de puerta de enlace diferente del que se ha empleado en el artículo hasta ahora, puede que tenga que ajustar la configuración a su situación.

Siga el resto de las indicaciones para crear el módulo. 

Después de unos 15 segundos, puede ejecutar el comando `iotedge list` en el dispositivo de puerta de enlace, el cual enumera todos los módulos en funcionamiento en el dispositivo IoT Edge. Aquí aparecerá el módulo OPC Publisher en funcionamiento.

:::image type="content" source="media/how-to-ingest-opcua-data/iotedge-list.png" alt-text="Captura de pantalla de los resultados de la lista de IoT Edge.":::

Por último, vaya al directorio `/iiotedge` y cree un archivo *unapublishednodes.json*. Los identificadores del archivo deben coincidir con los valores de `NodeId` que [recopiló anteriormente del servidor OPC](#install-opc-ua-simulation-software). El archivo debe tener un aspecto similar al siguiente:

```JSON
[
    {
        "EndpointUrl": "opc.tcp://20.185.195.172:53530/OPCUA/SimulationServer",
        "UseSecurity": false,
        "OpcNodes": [
            {
                "Id": "ns=3;i=1001"
            },
            {
                "Id": "ns=3;i=1002"
            },
            {
                "Id": "ns=3;i=1003"
            },
            {
                "Id": "ns=3;i=1004"
            },
            {
                "Id": "ns=3;i=1005"
            },
            {
                "Id": "ns=3;i=1006"
            }
        ]
    }
]
```

Guarde los cambios en el archivo *publishednodes.json*.

Ejecute el siguiente comando:

```bash
sudo iotedge logs OPCPublisher -f
```

El comando dará como resultado la salida de los registros de OPC Publisher. Si todo está configurado y funcionando correctamente, verá algo parecido a lo siguiente:

:::image type="content" source="media/how-to-ingest-opcua-data/iotedge-logs.png" alt-text="Captura de pantalla de los registros de IoT Edge en el terminal. Hay una columna de campos de información de diagnóstico a la izquierda y una columna de valores a la derecha.":::

Los datos se deben ahora transferir desde un servidor OPC UA a IoT Hub.

Para supervisar los mensajes que se transfieren a Azure IoT Hub, puede usar el siguiente comando:

```azurecli-interactive
az iot hub monitor-events -n <iot-hub-instance> -t 0
```

> [!TIP]
> Pruebe a usar [Azure IoT Explorer](../iot-pnp/howto-use-iot-explorer.md) para supervisar los mensajes de IoT Hub.

#### <a name="verify-completion"></a>Comprobación de la finalización

En esta sección, configurará un dispositivo de puerta de enlace que ejecuta IoT Edge y que recibirá datos de un servidor OPC UA. Compruebe que ha completado la siguiente lista de comprobación:
> [!div class="checklist"]
> * Se ha creado la máquina virtual de Ubuntu Server.
> * Se ha instalado IoT Edge y se encuentra en la máquina virtual Ubuntu.
> * Se ha instalado el módulo OPC Publisher.
> * Se ha creado y configurado el archivo *publishednodes.json*.
> * El módulo OPC Publisher está en ejecución y los datos de telemetría se transfieren a IoT Hub.

En el paso siguiente, enviará estos datos de telemetría a Azure Digital Twins.

## <a name="set-up-azure-digital-twins"></a>Configuración de Azure Digital Twins

Ahora que los datos se están transfiriendo del servidor OPC UA a Azure IoT Hub, el siguiente paso es configurar Azure Digital Twins. 

En este ejemplo, usará un único modelo y una única instancia de Digital Twins para que coincidan con las propiedades del servidor de simulación. 

>[!TIP]
>Si está interesado en ver un escenario más complejo con más modelos y gemelos, vea el ejemplo de la fábrica de chocolate en el [repositorio de GitHub de OPC UA en Azure Digital Twins](https://github.com/Azure-Samples/opcua-to-azure-digital-twins).

### <a name="create-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En primer lugar, implemente una nueva instancia de Azure Digital Twins, con las instrucciones que se indican en [Procedimiento de configuración de una instancia y de autenticación](how-to-set-up-instance-portal.md).

### <a name="upload-model-and-create-twin"></a>Carga del modelo y creación del gemelo

A continuación, agregue un modelo y un gemelo a la instancia. El archivo de modelo que cargará en la instancia forma parte del proyecto de ejemplo que descargó en la sección [Requisitos previos](#prerequisites) y que se encuentra en *Ejemplo de simulación/simulation-dtdl-model.json*.

Puede usar [Azure Digital Twins Explorer](concepts-azure-digital-twins-explorer.md) para cargar el modelo de simulación y crear un nuevo gemelo denominado **simulation-1**.

:::image type="content" source="media/how-to-ingest-opcua-data/azure-digital-twins-explorer.png" alt-text="Captura de pantalla de Azure Digital Twins Explorer en la que aparece el modelo de simulación y el gemelo simulation-1.":::

### <a name="verify-completion"></a>Comprobación de la finalización

En esta sección, configurará una instancia de Azure Digital Twins que contiene un modelo y un gemelo. Compruebe que ha completado la siguiente lista de comprobación:
> [!div class="checklist"]
> * Se ha implementado la instancia de Azure Digital Twins.
> * Se ha cargado el modelo de simulación en la instancia de Azure Digital Twins.
> * Se ha creado el gemelo simulation-1 a partir del modelo de simulación.

## <a name="set-up-azure-function"></a>Configuración de una función de Azure

Ahora que los nodos de OPC UA envían datos a IoT Hub y que una instancia de Azure Digital Twins está lista para recibirlos, deberá asignar y guardar los datos en los gemelos y propiedades correctos de Azure Digital Twins. En esta sección, configurará esto mediante una función de Azure y un archivo *opcua-mapping.json*.

El flujo de datos de esta sección implica estos pasos:

1. Una función de Azure usa una suscripción a eventos para recibir mensajes procedentes de IoT Hub.
1. La función de Azure procesa cada evento de telemetría que llega. Extrae el valor de `NodeId` del evento y lo busca en los elementos del *archivo opcua-mapping.json*. El archivo asigna cada valor de `NodeId` a un determinado `twinId` y a una propiedad de Azure Digital Twins en la que se debe guardar el valor del nodo.
1. La función de Azure genera el documento de revisión adecuado para actualizar el gemelo digital correspondiente y ejecuta el comando de actualización de la propiedad del gemelo.

### <a name="create-opcua-mappingjson-file"></a>Creación del archivo opcua-mapping.json

En primer lugar, cree el archivo *opcua-mapping.json*. Comience con un archivo JSON en blanco y rellene las entradas que asignan los valores de `NodeId` a los valores de `twinId` y a las propiedades de Azure Digital Twins, según el ejemplo y el esquema siguientes. Deberá crear una entrada de asignación para cada valor de `NodeId`.

```JSON
[
    {
        "NodeId": "1001",
        "TwinId": "simulation",
        "Property": "Counter",
        "ModelId": "dtmi:com:microsoft:iot:opcua:simulation;1"
    },
    ...
]
```

Este es el esquema de las entradas:

| Propiedad | Descripción | Requerido |
| --- | --- | --- |
| NodeId | Valor del nodo de OPC UA. Por ejemplo: ns=3;i={value} | ✔ |
| TwinId | Valor TwinId ($dtId) del gemelo para el que desea guardar el valor de telemetría | ✔ |
| Propiedad | Nombre de la propiedad en el gemelo para guardar el valor de telemetría | ✔ |
| ModelId  | El valor modelId para crear el gemelo si TwinId no existe |  |

> [!TIP]
> Para obtener un ejemplo completo de un archivo *opcua-mapping.json*, consulte el [repositorio de GitHub de OPC UA en Azure Digital Twins](https://github.com/Azure-Samples/opcua-to-azure-digital-twins).

Cuando haya terminado de agregar asignaciones, guarde el archivo.

Ahora que tiene el archivo de asignación, deberá almacenarlo en una ubicación a la que pueda acceder la función de Azure. Esta ubicación puede ser [Azure Blob Storage](../storage/blobs/storage-blobs-overview.md).

Siga las instrucciones para [crear un contenedor de almacenamiento](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) e importe el archivo *opcua-mapping.json* en el contenedor. También puede realizar eventos de administración de almacenamiento mediante [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/). 

A continuación, cree una [firma de acceso compartido para el contenedor](../storage/common/storage-sas-overview.md) y guarde esa dirección URL. Más adelante, proporcionará la dirección URL a la función de Azure para que pueda acceder al archivo almacenado.

:::image type="content" source="media/how-to-ingest-opcua-data/azure-storage-explorer.png" alt-text="Captura de pantalla del Explorador de Azure Storage que muestra el cuadro de diálogo para crear un token de SAS.":::

### <a name="publish-azure-function"></a>Publicación de una función de Azure

En esta sección, publicará una función de Azure que descargó en [Requisitos previos](#prerequisites) que procesará los datos de OPC UA y actualizará Azure Digital Twins.

#### <a name="step-1-open-the-function-in-visual-studio"></a>Paso 1: Abrir la función en Visual Studio

Vaya al proyecto [OPC UA en Azure Digital Twins](https://github.com/Azure-Samples/opcua-to-azure-digital-twins) descargado en la máquina local y en la carpeta *Azure Functions/OPCUAFunctions*. Abra la solución **OPCUAFunctions.sln** en Visual Studio.

#### <a name="step-2-publish-the-function"></a>Paso 2: Publicar la función

Publique el proyecto de función en una aplicación de funciones en Azure.

Para instrucciones sobre cómo hacerlo, consulte la sección [Publicación de la aplicación de funciones en Azure](how-to-create-azure-function.md#publish-the-function-app-to-azure) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

#### <a name="step-3-configure-the-function-app"></a>Paso 3: Configurar la aplicación de funciones

**Asigne un rol de acceso** para la función y **configure las opciones de la aplicación** para que pueda acceder a la instancia de Azure Digital Twins. Para instrucciones sobre cómo hacerlo, consulte la sección [Configuración del acceso de seguridad para la aplicación de funciones](how-to-create-azure-function.md#set-up-security-access-for-the-function-app) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

#### <a name="step-4-add-application-settings"></a>Paso 4: Agregar la configuración de la aplicación

También deberá agregar algunos valores de configuración de la aplicación para configurar completamente el entorno. Para ir a la función de Azure recién creada en [Azure Portal](https://portal.azure.com), búsquela por su nombre en la barra de búsqueda del portal.

Seleccione Configuración en el menú de navegación izquierdo de la función Use el botón **+ Nueva configuración de la aplicación** para empezar a crear una nueva configuración.

:::image type="content" source="media/how-to-ingest-opcua-data/azure-function-settings-1.png" alt-text="Captura de pantalla de la adición de la configuración de la aplicación a una función de Azure en Azure Portal.":::

Hay tres configuraciones de aplicación que debe crear:

| Configuración | Descripción | Requerido |
| --- | --- | --- |
| ADT_SERVICE_URL | Dirección URL para la instancia de Azure Digital Twins. Ejemplo: `https://example.api.eus.digitaltwins.azure.net` | ✔ |
| JSON_MAPPINGFILE_URL | Dirección URL de la firma de acceso compartido para el archivo opcua-mapping.json | ✔ |
| LOG_LEVEL | Nivel de detalle del registro. El valor predeterminado es 100. El valor actual es 300 | |

:::image type="content" source="media/how-to-ingest-opcua-data/azure-function-settings-2.png" alt-text="Captura de pantalla de la configuración de la aplicación para una función de Azure en Azure Portal. Se ha agregado la configuración anterior.":::

> [!TIP]
> Establezca la configuración de la aplicación `LOG_LEVEL` en la función en 300 para obtener una experiencia de registro más detallada. 

### <a name="create-event-subscription"></a>Creación de la suscripción de eventos

Por último, cree una suscripción a eventos para conectar la aplicación de funciones y la función ProcessOPCPublisherEventsToADT a IoT Hub. La suscripción a eventos es necesaria para que los datos puedan transferirse desde el dispositivo de puerta de enlace a IoT Hub mediante la función, la cual actualiza, a continuación, Azure Digital Twins.

Para obtener instrucciones, siga los mismos pasos que se usaron en [Conexión del centro de IoT a la función de Azure](tutorial-end-to-end.md#connect-the-iot-hub-to-the-azure-function) del *Tutorial: Conexión de una solución de un extremo a otro* de Azure Digital Twins.

La suscripción a eventos tendrá un tipo de punto de conexión de **función de Azure** y un punto de conexión de **ProcessOPCPublisherEventsToADT**.

:::image type="content" source="media/how-to-ingest-opcua-data/event-subscription.png" alt-text="Captura de pantalla de Azure Portal que muestra la creación de una nueva suscripción a eventos.":::

Después de este paso, se deben instalar y ejecutar todos los componentes necesarios. Los datos se deben transferir desde el servidor de OPC UA Simulation Server, a través de Azure IoT Hub a la instancia de Azure Digital Twins. 

En la sección siguiente se proporcionan algunos comandos de la CLI de Azure que puede ejecutar para supervisar los eventos y comprobar que todo funciona correctamente.

### <a name="verify-and-monitor"></a>Comprobación y supervisión

Los comandos de esta sección se pueden ejecutar en [Azure Cloud Shell](https://shell.azure.com) o en una [ventana de la CLI de Azure local](/cli/azure/install-azure-cli).

Ejecute este comando para supervisar eventos de IoT Hub:
```azurecli-interactive
az iot hub monitor-events -n <IoT-hub-name> -t 0
```

Ejecute este comando para supervisar el procesamiento de eventos de la función de Azure:
```azurecli-interactive
az webapp log tail –name <function-name> --resource-group <resource-group-name>
```

Por último, puede usar Azure Digital Twins Explorer para supervisar manualmente las actualizaciones de propiedades de los gemelos. 

:::image type="content" source="media/how-to-ingest-opcua-data/adt-explorer-2.png" alt-text="Captura de pantalla del uso de Azure Digital Twins Explorer para supervisar las actualizaciones de propiedades de los gemelos":::

### <a name="verify-completion"></a>Comprobación de la finalización

En esta sección, configurará una función de Azure para conectar los datos de OPC UA a Azure Digital Twins. Compruebe que ha completado la siguiente lista de comprobación:
> [!div class="checklist"]
> * Se ha creado e importado un archivo *opcua-mapping.json* en un contenedor de Blob Storage. 
> * Se ha publicado la función de ejemplo ProcessOPCPublisherEventsToADT en una aplicación de funciones de Azure.
> * Se han agregado tres nuevas opciones de configuración de la aplicación a Azure Functions.
> * Ha creado una suscripción a eventos para enviar eventos de IoT Hub a la aplicación de funciones.
> * Se han empleado comandos de la CLI de Azure para comprobar el flujo de datos final.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, configurará un flujo de datos completo para transferir datos simulados de un servidor OPC UA a Azure Digital Twins, donde se actualizará una propiedad en un gemelo digital.

A continuación, use los siguientes recursos para obtener más información sobre las herramientas y los procesos de soporte técnico que se usaron en este artículo:

* [Guía paso a paso para la instalación de OPC Publisher en Azure IoT Edge](https://www.linkedin.com/pulse/step-by-step-guide-installing-opc-publisher-azure-iot-kevin-hilscher) 
* [Instalación de IoT Edge en Linux](../iot-edge/how-to-install-iot-edge.md) 
* [OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher)
* [Configuración de OPC Publisher](../iot-accelerators/howto-opc-publisher-configure.md)
* [UANodeSetWebViewer](https://github.com/barnstee/UANodesetWebViewer) 
* [OPCUA2DTDL](https://github.com/khilscher/OPCUA2DTDL)
