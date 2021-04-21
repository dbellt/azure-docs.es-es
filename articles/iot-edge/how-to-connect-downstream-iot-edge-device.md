---
title: 'Conexión de dispositivos IoT Edge de nivel inferior: Azure IoT Edge | Microsoft Docs'
description: Se describe cómo configurar un dispositivo IoT Edge para conectarse a dispositivos de puerta de enlace Azure IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/01/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: 500833d1bb4fc492942c08239bd488c2d2c16d30
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484326"
---
# <a name="connect-a-downstream-iot-edge-device-to-an-azure-iot-edge-gateway"></a>Conexión de un dispositivo IoT Edge de nivel inferior a una puerta de enlace Azure IoT Edge

[!INCLUDE [iot-edge-version-202011](../../includes/iot-edge-version-202011.md)]

En este artículo se proporcionan instrucciones para establecer una conexión de confianza entre una puerta de enlace IoT Edge y un dispositivo IoT Edge de nivel inferior.

En un escenario de puerta de enlace, un dispositivo IoT Edge puede ser tanto una puerta de enlace como un dispositivo de nivel inferior. Se pueden superponer varias puertas de enlace IoT Edge para crear una jerarquía de dispositivos. Los dispositivos de nivel inferior (o secundarios) pueden autenticar y enviar o recibir mensajes a través de su dispositivo de puerta de enlace (o primario).

Hay dos configuraciones diferentes para los dispositivos IoT Edge en una jerarquía de puertas de enlace, y en este artículo se abordan ambas. La primera es el dispositivo IoT Edge de la **capa superior**. Cuando varios dispositivos IoT Edge se conectan entre sí, cualquier dispositivo que no tenga un dispositivo primario, pero se conecte directamente a IoT Hub, se considera que está en la capa superior. Este dispositivo es responsable de administrar las solicitudes de todos los dispositivos que se encuentran debajo de él. La otra configuración se aplica a cualquier dispositivo IoT Edge en una **capa inferior** de la jerarquía. Estos dispositivos pueden ser una puerta de enlace para otros dispositivos IoT y IoT Edge de nivel inferior, pero también necesitan enrutar las comunicaciones a través de sus propios dispositivos primarios.

Algunas arquitecturas de red requieren que solo el dispositivo IoT Edge superior de una jerarquía pueda conectarse a la nube. En esta configuración, todos los dispositivos IoT Edge en capas inferiores de una jerarquía solo se pueden comunicar con su dispositivo de puerta de enlace (o primario) y con dispositivos de nivel inferior (o secundarios).

Todos los pasos de este artículo se basan en [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md), donde se configura un dispositivo IoT Edge para que sea una puerta de enlace para dispositivos IoT de nivel inferior. Los mismos pasos básicos se aplican a todos los escenarios de puertas de enlace:

* **Autenticación**: Cree identidades de IoT Hub para todos los dispositivos de la jerarquía de puertas de enlace.
* **Autorización**: Configure la relación de elementos primarios y secundarios en IoT Hub para autorizar a los dispositivos secundarios a que se conecten a su dispositivo primario como se conectarían a IoT Hub.
* **Detección de puerta de enlace**: Asegúrese de que el dispositivo secundario pueda encontrar su dispositivo primario en la red local.
* **Conexión segura**: Establezca una conexión segura con certificados de confianza que formen parte de la misma cadena.

## <a name="prerequisites"></a>Requisitos previos

* Un centro de IoT gratis o estándar.
* Al menos dos **dispositivos IoT Edge**, uno para que sea el dispositivo de nivel superior y uno o varios dispositivos de capa inferior. Si no tiene dispositivos IoT Edge disponibles, puede [ejecutar Azure IoT Edge en máquinas virtuales Ubuntu](how-to-install-iot-edge-ubuntuvm.md).
* Si usa la CLI de Azure para crear y administrar dispositivos, tenga instalada la CLI de Azure v2.3.1 con la extensión de Azure IoT v0.10.6 o posterior.

En este artículo se proporcionan pasos detallados y opciones para ayudarle a crear la jerarquía de puertas de enlace correcta para su escenario. Para ver un tutorial guiado, consulte [Creación de una jerarquía de dispositivos IoT Edge mediante puertas de enlace](tutorial-nested-iot-edge.md).

## <a name="create-a-gateway-hierarchy"></a>Creación de una jerarquía de puertas de enlace

Cree una jerarquía de puertas de enlace IoT Edge mediante la definición de relaciones de elementos primarios y secundarios para los dispositivos IoT Edge en el escenario. Para establecer un dispositivo primario, puede crear una nueva identidad del dispositivo o puede administrar el elemento primario y los elementos secundarios de una identidad del dispositivo existente.

El paso de configuración de las relaciones de elementos primarios y secundarios autoriza a los dispositivos secundarios a conectarse a su dispositivo primario como se conectarían a IoT Hub.

Solo los dispositivos IoT Edge pueden ser dispositivos primarios, pero tanto los dispositivos IoT Edge como los dispositivos IoT pueden ser secundarios. Un elemento primario puede tener muchos elementos secundarios, pero un elemento secundario solo puede tener un elemento primario. Una jerarquía de puertas de enlace se crea encadenando conjuntos de elementos primarios y secundarios para que el elemento secundario de un dispositivo sea el primario de otro.

<!-- TODO: graphic of gateway hierarchy -->

# <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, puede administrar la relación de elementos primarios y secundarios al crear nuevas identidades de dispositivos o editando los dispositivos existentes.

Al crear un dispositivo IoT Edge, tiene la opción de elegir dispositivos primarios y secundarios de la lista de dispositivos IoT Edge existentes en ese centro.

1. En [Azure Portal](https://portal.azure.com), navegue hasta su centro de IoT.
1. Seleccione **IoT Edge** en el menú de navegación.
1. Seleccione **Agregar un dispositivo IoT Edge**.
1. Además de establecer el identificador del dispositivo y la configuración de autenticación, puede **establecer un dispositivo primario** o **elegir dispositivos secundarios**.
1. Elija el o los dispositivos que quiera como primario o secundario.

También puede crear o administrar relaciones de elementos primarios y secundarios para los dispositivos existentes.

1. En [Azure Portal](https://portal.azure.com), navegue hasta su centro de IoT.
1. Seleccione **IoT Edge** en el menú de navegación.
1. Seleccione el dispositivo que quiera administrar de la lista de **dispositivos IoT Edge**.
1. Seleccione **Establecer un dispositivo primario** o **Administrar dispositivos secundarios**.
1. Agregue o quite dispositivos primarios o secundarios.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

La extensión [azure-iot](/cli/azure/iot) de la CLI de Azure proporciona comandos para administrar los recursos de IoT. Puede administrar la relación de elementos primarios y secundarios de dispositivos IoT y IoT Edge al crear nuevas identidades de dispositivos o editando los dispositivos existentes.

El conjunto de comandos [az iot hub device-identity](/cli/azure/iot/hub/device-identity) permite administrar las relaciones de elementos primarios y secundarios para un dispositivo determinado.

El comando `create` incluye parámetros para agregar dispositivos secundarios y establecer un dispositivo primario en el momento de la creación del dispositivo.

Los comandos adicionales de identidades de dispositivos, incluidos `add-children`, `list-children` y `remove-children`, o `get-parent` y `set-parent`, permiten administrar las relaciones de elementos primarios y secundarios para los dispositivos existentes.

---

## <a name="prepare-certificates"></a>Preparar certificados

Se debe instalar una cadena de certificados coherente en los dispositivos de la misma jerarquía de puertas de enlace para establecer una comunicación segura entre ellos. Todos los dispositivos de la jerarquía, ya sean dispositivos IoT Edge o dispositivos de hoja de IoT, necesitan una copia del mismo certificado de CA raíz. Cada dispositivo IoT Edge de la jerarquía luego usa ese certificado de CA raíz como raíz para su certificado de CA de dispositivo.

Con esta configuración, cada dispositivo IoT Edge de nivel inferior o dispositivo de hoja de IoT puede comprobar la identidad de su elemento primario comprobando que el edgeHub al que se conecta tenga un certificado de servidor firmado por el certificado de CA raíz compartido.

<!-- TODO: certificate graphic -->

Cree los siguientes certificados:

* Un **certificado de CA raíz**, que es el certificado compartido de nivel superior para todos los dispositivos de una jerarquía de puertas de enlace determinada. Este certificado está instalado en todos los dispositivos.
* Todo **certificado intermedio** que quiera incluir en la cadena de certificados raíz.
* Un **certificado de CA de dispositivo** y su **clave privada**, generada por los certificados raíz e intermedios. Necesita un único certificado de CA de dispositivo para cada dispositivo IoT Edge en la jerarquía de puertas de enlace.

Puede usar una entidad de certificado autofirmado o comprar uno a una entidad de certificación comercial de confianza, como Baltimore, Verisign, DigiCert o GlobalSign.

Si no tiene sus propios certificados para usar, puede [crear certificados de demostración para probar las características de dispositivo IoT Edge](how-to-create-test-certificates.md). Siga los pasos de este artículo para crear un conjunto de certificados raíz e intermedios y, a continuación, crear certificados de CA de dispositivos IoT Edge para cada uno de los dispositivos.

## <a name="configure-iot-edge-on-devices"></a>Configuración de IoT Edge en los dispositivos

Los pasos para configurar IoT Edge como puerta de enlace son muy similares a los pasos para configurar IoT Edge como dispositivo de nivel inferior.

Para habilitar la detección de puertas de enlace, cada dispositivo de puerta de enlace IoT Edge debe configurarse con un **nombre de host** que sus dispositivos secundarios usarán para encontrarlo en la red local. Cada dispositivo IoT Edge de nivel inferior debe configurarse con un **parent_hostname** para conectarse. Si un único dispositivo IoT Edge es tanto un dispositivo primario como uno secundario, necesita ambos parámetros.

Para habilitar las conexiones seguras, cada dispositivo IoT Edge en un escenario de puerta de enlace debe configurarse con un único certificado de CA de dispositivo y una copia del certificado de CA raíz compartido por todos los dispositivos de la jerarquía de puertas de enlace.

Ya tiene que haber instalado IoT Edge en el dispositivo. Si no es así, siga los pasos que se describen en [Registro de un dispositivo IoT Edge en IoT Hub](how-to-register-device.md) y, luego, los de [Instalación del entorno de ejecución de Azure IoT Edge](how-to-install-iot-edge.md).

En los pasos de esta sección se hace referencia al **certificado de CA raíz** y al **certificado de CA de dispositivo y clave privada** que se han descrito anteriormente en este artículo. Si ha creado esos certificados en otro dispositivo, haga que estén disponibles en este dispositivo. Puede transferir los archivos físicamente, como con una unidad USB, con un servicio, como [Azure Key Vault](../key-vault/general/overview.md), o con una función, como [Secure file copy](https://www.ssh.com/ssh/scp/).

Siga los pasos a continuación para configurar IoT Edge en el dispositivo.

Asegúrese de que el usuario **iotedge** tenga permisos de lectura para el directorio que contiene los certificados y las claves.

1. Instale el **certificado de CA raíz** en este dispositivo IoT Edge.

   ```bash
   sudo cp <path>/<root ca certificate>.pem /usr/local/share/ca-certificates/<root ca certificate>.pem.crt
   ```

1. Actualice el almacén de certificados.

   ```bash
   sudo update-ca-certificates
   ```

   La salida de este comando debe indicar que se ha agregado un certificado en /etc/ssl/certs.

1. Abra el archivo de configuración de IoT Edge.

   ```bash
   sudo nano /etc/aziot/config.toml
   ```

   >[!TIP]
   >Si el archivo de configuración todavía no existe en el dispositivo, use `/etc/aziot/config.toml.edge.template` como plantilla para crear uno.

1. Busque la sección **Hostname** en el archivo de configuración. Quite la marca de comentario de la línea con el parámetro `hostname` y actualice el valor para que sea el nombre de dominio completo (FQDN) o la dirección IP del dispositivo IoT Edge.

   El valor de este parámetro es lo que los dispositivos de nivel inferior usarán para conectarse a esta puerta de enlace. El nombre de host toma el nombre de la máquina de manera predeterminada, pero el FQDN o la dirección IP son necesarios para conectar los dispositivos de nivel inferior.

   Use un nombre de host de menos de 64 caracteres, que es el límite de caracteres de un nombre común del certificado de servidor.

   Sea coherente con el patrón del nombre de host en una jerarquía de puertas de enlace. Use FQDN o direcciones IP, pero no ambos.

1. *Si se trata de un dispositivo secundario*, busque la sección **Parent hostname**. Quite la marca de comentario y actualice el parámetro `parent_hostname` para que sea el FQDN o la dirección IP del dispositivo primario, que coincida con lo que se indicó como nombre de host en el archivo de configuración del dispositivo primario.

1. Busque la sección **Trust bundle cert**. Quite la marca de comentario y actualice el parámetro `trust_bundle_cert` con el URI del archivo para el certificado de CA raíz del dispositivo.

1. Compruebe que el dispositivo IoT Edge usará la versión correcta del agente de IoT Edge cuando se inicie.

   Busque la sección **Default Edge Agent** y compruebe que el valor de la imagen es IoT Edge versión 1.2. Si no es así, actualícela:

   ```toml
   [agent.config]
   image: "mcr.microsoft.com/azureiotedge-agent:1.2"
   ```

1. Busque la sección **Edge CA certificate** del archivo de configuración. Quite las marcas de comentario de las líneas de esta sección y proporcione las rutas de acceso del URI de archivo para los archivos de certificado y clave en el dispositivo IoT Edge.

   ```toml
   [edge_ca]
   cert = "file:///<path>/<device CA cert>"
   pk = "file:///<path>/<device CA key>"
   ```

1. Guarde (`Ctrl+O`) y cierre (`Ctrl+X`) el archivo de configuración.

1. Si anteriormente ha usado cualquier otro certificado para IoT Edge, elimine los archivos de los dos directorios siguientes para asegurarse de que se apliquen los nuevos certificados:

   * `/var/lib/aziot/certd/certs`
   * `/var/lib/aziot/keyd/keys`

1. Aplique los cambios.

   ```bash
   sudo iotedge config apply
   ```

1. Compruebe los errores en la configuración.

   ```bash
   sudo iotedge check --verbose
   ```

   >[!TIP]
   >La herramienta de comprobación de IoT Edge usa un contenedor para realizar algunas de las comprobaciones de diagnóstico. Si quiere usar esta herramienta en dispositivos IoT Edge de nivel inferior, asegúrese de que pueden acceder a `mcr.microsoft.com/azureiotedge-diagnostics:latest`, o mantenga la imagen de contenedor en el registro de contenedor privado.

## <a name="network-isolate-downstream-devices"></a>Aislamiento de red de dispositivos de nivel inferior

Hasta ahora, los pasos de este artículo configuran dispositivos IoT Edge como puerta de enlace o como dispositivo de nivel inferior, y crean una conexión de confianza entre ellos. El dispositivo de puerta de enlace controla las interacciones entre el dispositivo de nivel inferior y IoT Hub, incluida la autenticación y el enrutamiento de mensajes. Los módulos implementados en dispositivos IoT Edge de nivel inferior pueden crear sus propias conexiones a los servicios en la nube.

Algunas arquitecturas de red, como las que siguen el estándar ISA-95, buscan minimizar el número de conexiones a Internet. En estos casos, puede configurar los dispositivos IoT Edge de nivel inferior sin conectividad a Internet. Más allá del enrutamiento de las comunicaciones de IoT Hub a través del dispositivo de puerta de enlace, los dispositivos IoT Edge de nivel inferior pueden depender del dispositivo de puerta de enlace para todas las conexiones en la nube

Esta configuración de red requiere que solo el dispositivo IoT Edge de la capa superior de una jerarquía de puertas de enlace tenga conexiones directas a la nube. Los dispositivos IoT Edge de las capas inferiores solo se pueden comunicar con su dispositivo primario o con los dispositivos secundarios. Los módulos especiales de los dispositivos de puerta de enlace permiten este escenario, entre los que se incluyen:

* El **módulo de proxy de API** es necesario en cualquier puerta de enlace IoT Edge que tenga otro dispositivo IoT Edge debajo. Esto significa que debe estar en *cada capa* de una jerarquía de puertas de enlace, excepto la capa última. Este módulo usa un proxy inverso [nginx](https://nginx.org) para enrutar los datos HTTP a través de las capas de red mediante un único puerto. Es altamente configurable a través de su módulo gemelo y las variables de entorno, por lo que se puede ajustar para adaptarse a los requisitos de su escenario de puerta de enlace.

* El **módulo de registro de Docker** puede implementarse en la puerta de enlace IoT Edge en la *capa superior* de una jerarquía de puertas de enlace. Este módulo es responsable de la recuperación y el almacenamiento en caché de las imágenes de contenedor en nombre de todos los dispositivos IoT Edge de las capas inferiores. La alternativa a la implementación de este módulo en la capa superior es usar un registro local, o cargar manualmente las imágenes de contenedor en los dispositivos y establecer la directiva de extracción del módulo en **nunca**.

* **Azure Blob Storage en IoT Edge** puede implementarse en la puerta de enlace IoT Edge en la *capa superior* de una jerarquía de puertas de enlace. Este módulo es responsable de la carga de blobs en nombre de todos los dispositivos IoT Edge de las capas inferiores. La capacidad de cargar blobs también hace posible una funcionalidad útil de solución de problemas para dispositivos IoT Edge en capas inferiores, como la carga de registros de módulos y la carga de paquetes de soporte técnico.

### <a name="network-configuration"></a>Configuración de red

Para cada dispositivo de puerta de enlace de la capa superior, los operadores de red deben:

* Proporcionar una dirección IP estática o nombre de dominio completo (FQDN).
* Autorizar las comunicaciones salientes desde esta dirección IP a su nombre de host de Azure IoT Hub a través de los puertos 443 (HTTPS) y 5671 (AMQP).
* Autorizar las comunicaciones salientes desde esta dirección IP a su nombre de host de Azure Container Registry a través del puerto 443 (HTTPS).

  El módulo de proxy de API solo puede controlar conexiones a un registro de contenedor a la vez. Se recomienda almacenar todas las imágenes de contenedor, incluidas las imágenes públicas proporcionadas por el Registro de contenedor de Microsoft (mcr.microsoft.com), en el registro de contenedor privado.

Para cada dispositivo de puerta de enlace de una capa inferior, los operadores de red deben:

* Proporcionar una dirección IP estática.
* Autorizar las comunicaciones salientes desde esta dirección IP a la dirección IP de la puerta de enlace primaria a través de los puertos 443 (HTTPS) y 5671 (AMQP).

### <a name="deploy-modules-to-top-layer-devices"></a>Implementación de módulos en el dispositivos de capa superior

El dispositivo IoT Edge en la capa superior de una jerarquía de puertas de enlace tiene un conjunto de módulos necesarios que se deben implementar en él, además de los módulos de carga de trabajo que se puedan ejecutar en el dispositivo.

El módulo de proxy de API se diseñó para personalizarse y administrar los escenarios de puerta de enlace más comunes. En este artículo se proporciona un ejemplo de cómo establecer los módulos en una configuración básica. Consulte [Configuración del módulo de proxy de API para el escenario de jerarquías de puertas de enlace](how-to-configure-api-proxy-module.md) para obtener información más detallada y ejemplos.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), navegue hasta su centro de IoT.
1. Seleccione **IoT Edge** en el menú de navegación.
1. Seleccione el dispositivo de la capa superior que va a configurar en la lista de **dispositivos IoT Edge**.
1. Seleccione **Set modules** (Establecer módulos).
1. En la sección **Módulos de IoT Edge**, seleccione **Agregar** y, a continuación, elija **Módulo de Marketplace**.
1. Busque y seleccione el módulo **Proxy de API de IoT Edge**.
1. Seleccione el nombre del módulo de proxy de API en la lista de módulos implementados y actualice la siguiente configuración del módulo:
   1. En la pestaña **Variables de entorno**, actualice el valor de **NGINX_DEFAULT_PORT** a `443`.
   1. En la pestaña **Opciones de creación del contenedor**, actualice los enlaces de puerto para hacer referencia al puerto 443.

      ```json
      {
        "HostConfig": {
          "PortBindings": {
            "443/tcp": [
              {
                "HostPort": "443"
              }
            ]
          }
        }
      }
      ```

   Estos cambios configuran el módulo de proxy de API para que escuche en el puerto 443. Para evitar colisiones de enlace de puertos, debe configurar el módulo edgeHub para que no escuche en el puerto 443. En su lugar, el módulo de proxy de API enrutará todo el tráfico de edgeHub en l puerto 443.

1. Seleccione **Configuración del entorno de ejecución** y busque las opciones de creación del módulo edgeHub. Elimine el enlace de puerto para el puerto 443, y deje los enlaces a los puertos 5671 y 8883.

   ```json
   {
     "HostConfig": {
       "PortBindings": {
         "5671/tcp": [
           {
             "HostPort": "5671"
           }
         ],
         "8883/tcp": [
           {
             "HostPort": "8883"
           }
         ]
       }
     }
   }
   ```

1. Seleccione **Guardar** para guardar los cambios en la configuración del entorno de ejecución.
1. Seleccione **Agregar** de nuevo y, a continuación, elija **Módulo IoT Edge**.
1. Proporcione los siguientes valores para agregar el módulo del registro de Docker a su implementación:
   1. **Nombre del módulo IoT Edge**: `registry`
   1. En la pestaña **Configuración del módulo**, **URI de la imagen**: `registry:latest`
   1. En la pestaña **Variables de entorno**, agregue las siguientes variables de entorno:

      * **Nombre**: `REGISTRY_PROXY_REMOTEURL` **Valor**: Dirección URL del registro de contenedor al que quiere que se asigne este módulo del registro. Por ejemplo, `https://myregistry.azurecr`.

        El módulo del registro solo se puede asignar a un registro de contenedor, por lo que se recomienda tener todas las imágenes de contenedor en un solo registro de contenedor privado.

      * **Nombre**: `REGISTRY_PROXY_USERNAME` **Valor**: Nombre de usuario para autenticarse en el registro de contenedor.

      * **Nombre**: `REGISTRY_PROXY_PASSWORD` **Valor**: Contraseña para autenticarse en el registro de contenedor.

   1. En la pestaña **Opciones de creación del contenedor**, pegue:

      ```json
      {
          "HostConfig": {
              "PortBindings": {
                  "5000/tcp": [
                      {
                          "HostPort": "5000"
                      }
                  ]
              }
          }
      }
      ```

1. Seleccione **Agregar** para agregar el módulo a la implementación.
1. Seleccione **Siguiente: Rutas** para ir al paso siguiente.
1. Para habilitar los mensajes del dispositivo a la nube desde dispositivos de nivel inferior para que se comuniquen con IoT Hub, incluya una ruta que pase todos los mensajes a IoT Hub. Por ejemplo:
    1. **Nombre**: `Route`
    1. **Valor**: `FROM /messages/* INTO $upstream`
1. Seleccione **Revisar y crear** para ir al paso final.
1. Seleccione **Crear** para implementar el dispositivo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. En [Azure Cloud Shell](https://shell.azure.com/), cree un archivo JSON de implementación. Por ejemplo:

   ```json
   {
       "modulesContent": {
           "$edgeAgent": {
               "properties.desired": {
                   "modules": {
                       "dockerContainerRegistry": {
                           "settings": {
                               "image": "registry:latest",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5000/tcp\":[{\"HostPort\":\"5000\"}]}}}"
                           },
                           "type": "docker",
                           "version": "1.0",
                           "env": {
                               "REGISTRY_PROXY_REMOTEURL": {
                                   "value": "The URL for the container registry you want this registry module to map to. For example, https://myregistry.azurecr"
                               },
                               "REGISTRY_PROXY_USERNAME": {
                                   "value": "Username to authenticate to the container registry."
                               },
                               "REGISTRY_PROXY_PASSWORD": {
                                   "value": "Password to authenticate to the container registry."
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       },
                       "IoTEdgeAPIProxy": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-api-proxy:1.0",
                               "createOptions": "{\"HostConfig\": {\"PortBindings\": {\"443/tcp\": [{\"HostPort\":\"443\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "NGINX_DEFAULT_PORT": {
                                   "value": "443"
                               },
                               "DOCKER_REQUEST_ROUTE_ADDRESS": {
                                   "value": "registry:5000"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always",
                           "version": "1.0"
                       }
                   },
                   "runtime": {
                       "settings": {
                           "minDockerVersion": "v1.25"
                       },
                       "type": "docker"
                   },
                   "schemaVersion": "1.1",
                   "systemModules": {
                       "edgeAgent": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-agent:1.2",
                               "createOptions": ""
                           },
                           "type": "docker"
                       },
                       "edgeHub": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-hub:1.2",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                           },
                           "type": "docker",
                           "env": {},
                           "status": "running",
                           "restartPolicy": "always"
                       }
                   }
               }
           },
           "$edgeHub": {
               "properties.desired": {
                   "routes": {
                       "route": "FROM /messages/* INTO $upstream"
                   },
                   "schemaVersion": "1.1",
                   "storeAndForwardConfiguration": {
                       "timeToLiveSecs": 7200
                   }
               }
           }
       }
   }
   ```

   Este archivo de implementación configura el módulo de proxy de API para que escuche en el puerto 443. Para evitar colisiones de enlace de puertos, el archivo configura el módulo edgeHub para que no escuche en el puerto 443. En su lugar, el módulo de proxy de API enrutará todo el tráfico de edgeHub en l puerto 443.

1. Escriba el siguiente comando para crear una implementación en un dispositivo IoT Edge:

   ```bash
   az iot edge set-modules --device-id <device_id> --hub-name <iot_hub_name> --content ./<deployment_file_name>.json
   ```

---

### <a name="deploy-modules-to-lower-layer-devices"></a>Implementación de módulos en dispositivos de capas inferiores

Los dispositivos IoT Edge en las capas inferiores de una jerarquía de puertas de enlace tienen un módulo necesario que se debe implementar en ellos, además de los módulos de carga de trabajo que se puedan ejecutar en los dispositivos.

#### <a name="route-container-image-pulls"></a>Enrutamiento de extracciones de imágenes de contenedor

Antes de analizar el módulo de proxy necesario para los dispositivos IoT Edge en las jerarquías de puertas de enlace, es importante comprender cómo los dispositivos IoT Edge de las capas inferiores obtienen sus imágenes de módulo.

Si los dispositivos de capas inferiores no se pueden conectar a la nube, pero quiere que extraigan imágenes de módulo como de costumbre, el dispositivo de la capa superior de la jerarquía de puertas de enlace debe configurarse para controlar estas solicitudes. El dispositivo de la capa superior tiene que ejecutar un módulo de **registro** de Docker que esté asignado al registro de contenedor. A continuación, configure el módulo de proxy de API para enrutar a él las solicitudes de contenedor. Estos detalles se describen en las secciones anteriores de este artículo. En esta configuración, los dispositivos de capas inferiores no deben apuntar a los registros de contenedor en la nube, sino al registro que se ejecuta en la capa superior.

Por ejemplo, en lugar de llamar a `mcr.microsoft.com/azureiotedge-api-proxy:1.0`, los dispositivos de las capas inferiores deben llamar a `$upstream:443/azureiotedge-api-proxy:1.0`.

El parámetro **$upstream** apunta al elemento primario de un dispositivo de capa inferior, por lo que la solicitud se enrutará a través de todas las capas hasta que llegue a la capa superior, que tiene un entorno de proxy que enruta las solicitudes al módulo del registro. El puerto `:443` en este ejemplo debe reemplazarse por el puerto en el que esté escuchando el módulo de proxy de API en el dispositivo primario.

El módulo de proxy de API solo puede enrutar a un módulo del registro, y cada módulo del registro solo puede asignarse a un registro de contenedor. Por lo tanto, las imágenes que los dispositivos de las capas inferiores tengan que extraer deben almacenarse en un registro de contenedor único.

Si no quiere que los dispositivos de las capas inferiores realicen solicitudes de extracción de módulos a través de una jerarquía de puertas de enlace, otra opción es administrar una solución de registro local. O bien, inserte las imágenes del módulo en los dispositivos antes de crear las implementaciones y, a continuación, establezca **imagePullPolicy** en **never**.

#### <a name="bootstrap-the-iot-edge-agent"></a>Arranque del agente de IoT Edge

El agente de IoT Edge es el primer componente del entorno de ejecución en iniciarse en cualquier dispositivo IoT Edge. Debe asegurarse de que los dispositivos IoT Edge de nivel inferior pueden acceder a la imagen del módulo edgeAgent cuando se inician y, a continuación, pueden acceder a las implementaciones e iniciar el resto de las imágenes del módulo.

Cuando vaya al archivo de configuración en un dispositivo IoT Edge para proporcionar la información de autenticación, los certificados y el nombre de host primario, actualice también la imagen del contenedor edgeAgent.

Si el dispositivo de puerta de enlace de nivel superior está configurado para controlar las solicitudes de imagen de contenedor, reemplace `mcr.microsoft.com` por el nombre de host primario y el puerto de escucha del proxy de API. En el manifiesto de implementación, puede usar `$upstream` como acceso directo, pero esto requiere que el módulo edgeHub controle el enrutamiento y que ese módulo no se haya iniciado en este momento. Por ejemplo:

```toml
[agent]
name = "edgeAgent"
type = "docker"

[agent.config]
image: "{Parent FQDN or IP}:443/azureiotedge-agent:1.2"
```

Si usa un registro de contenedor local o proporciona las imágenes de contenedor manualmente en el dispositivo, actualice el archivo de configuración en consecuencia.

#### <a name="configure-runtime-and-deploy-proxy-module"></a>Configuración del entorno de ejecución e implementación del módulo de proxy

El **módulo de proxy de API** es necesario para enrutar todas las comunicaciones entre la nube y los dispositivos IoT Edge de nivel inferior. Un dispositivo IoT Edge en la capa última de la jerarquía, sin dispositivos IoT Edge de nivel inferior, no necesita este módulo.

El módulo de proxy de API se diseñó para personalizarse y administrar los escenarios de puerta de enlace más comunes. En este artículo se analizan brevemente los pasos para establecer los módulos en una configuración básica. Consulte [Configuración del módulo de proxy de API para el escenario de jerarquías de puertas de enlace](how-to-configure-api-proxy-module.md) para obtener información más detallada y ejemplos.

1. En [Azure Portal](https://portal.azure.com), navegue hasta su centro de IoT.
1. Seleccione **IoT Edge** en el menú de navegación.
1. Seleccione el dispositivo de la capa inferior que va a configurar en la lista de **dispositivos IoT Edge**.
1. Seleccione **Set modules** (Establecer módulos).
1. En la sección **Módulos de IoT Edge**, seleccione **Agregar** y, a continuación, elija **Módulo de Marketplace**.
1. Busque y seleccione el módulo **Proxy de API de IoT Edge**.
1. Seleccione el nombre del módulo de proxy de API en la lista de módulos implementados y actualice la siguiente configuración del módulo:
   1. En la pestaña **Configuración del módulo**, actualice el valor de **URI de la imagen**. Reemplace `mcr.microsoft.com` por `$upstream:443`.
   1. En la pestaña **Variables de entorno**, actualice el valor de **NGINX_DEFAULT_PORT** a `443`.
   1. En la pestaña **Opciones de creación del contenedor**, actualice los enlaces de puerto para hacer referencia al puerto 443.

      ```json
      {
        "HostConfig": {
          "PortBindings": {
            "443/tcp": [
              {
                "HostPort": "443"
              }
            ]
          }
        }
      }
      ```

   Estos cambios configuran el módulo de proxy de API para que escuche en el puerto 443. Para evitar colisiones de enlace de puertos, debe configurar el módulo edgeHub para que no escuche en el puerto 443. En su lugar, el módulo de proxy de API enrutará todo el tráfico de edgeHub en l puerto 443.

1. Seleccione **Configuración del entorno de ejecución**.
1. Actualice la configuración del módulo edgeHub:

   1. En el campo **Imagen**, reemplace `mcr.microsoft.com` por `$upstream:443`.
   1. En el campo **Opciones de creación**, elimine el enlace de puerto para el puerto 443, y deje los enlaces a los puertos 5671 y 8883.

   ```json
   {
     "HostConfig": {
       "PortBindings": {
         "5671/tcp": [
           {
             "HostPort": "5671"
           }
         ],
         "8883/tcp": [
           {
             "HostPort": "8883"
           }
         ]
       }
     }
   }
   ```

1. Actualice la configuración del módulo edgeAgent:
   1. En el campo **Imagen**, reemplace `mcr.microsoft.com` por `$upstream:443`.

1. Seleccione **Guardar** para guardar los cambios en la configuración del entorno de ejecución.
1. Seleccione **Siguiente: Rutas** para ir al paso siguiente.
1. Para habilitar los mensajes del dispositivo a la nube desde dispositivos de nivel inferior para que se comuniquen con IoT Hub, incluya una ruta que pase todos los mensajes a `$upstream`. El parámetro ascendente apunta al dispositivo primario en el caso de los dispositivos de capas inferiores. Por ejemplo:
    1. **Nombre**: `Route`
    1. **Valor**: `FROM /messages/* INTO $upstream`
1. Seleccione **Revisar y crear** para ir al paso final.
1. Seleccione **Crear** para implementar el dispositivo.

## <a name="next-steps"></a>Pasos siguientes

[Uso de un dispositivo IoT Edge como puerta de enlace](iot-edge-as-gateway.md)

[Configuración del módulo de proxy de API para el escenario de jerarquías de puertas de enlace](how-to-configure-api-proxy-module.md)