---
title: Conservación del estado en Windows - Azure Event Grid IoT Edge | Microsoft Docs
description: Conservación del estado en Windows
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 23428688d7f65bf311ecd23d47a23ab9b2e89617
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367750"
---
# <a name="persist-state-in-windows"></a>Conservación del estado en Windows

Los temas y las suscripciones creados en el módulo de Event Grid se almacenan de forma predeterminada en el sistema de archivos de contenedor. Sin persistencia, si se vuelve a implementar el módulo, se perderán todos los metadatos creados. Para conservar los datos en las implementaciones y reinicios, debe conservar los datos fuera del sistema de archivos de contenedor. 

De forma predeterminada, solo se conservan los metadatos y los eventos todavía se almacenan en memoria para mejorar el rendimiento. Siga la sección Conservación de eventos para habilitar también la conservación de eventos.

En este artículo se proporcionan los pasos necesarios para implementar el módulo de Event Grid con persistencia en las implementaciones de Windows.

> [!NOTE]
>El módulo de Event Grid se ejecuta como un usuario con pocos privilegios **ContainerUser** en Windows.

## <a name="persistence-via-volume-mount"></a>Persistencia a través del montaje del volumen

[Los volúmenes de Docker](https://docs.docker.com/storage/volumes/) se usan para conservar los datos en las implementaciones. Para montar un volumen, debe crearlo con los comandos de Docker, conceder permisos para que el contenedor pueda leer y escribir en él y, a continuación, implementar el módulo.

1. Cree un volumen mediante la ejecución del siguiente comando:

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine volume create <your-volume-name-here>
    ```

    Por ejemplo,

   ```sh
   docker -H npipe:////./pipe/iotedge_moby_engine volume create myeventgridvol
   ```
1. Obtenga el directorio host al que se asigna el volumen mediante la ejecución del comando siguiente.

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine volume inspect <your-volume-name-here>
    ```

    Por ejemplo,

   ```sh
   docker -H npipe:////./pipe/iotedge_moby_engine volume inspect myeventgridvol
   ```

   Salida de ejemplo:

   ```json
   [
          {
            "CreatedAt": "2019-07-30T21:20:59Z",
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "C:\\ProgramData\\iotedge-moby\u000bolumes\\myeventgridvol\\_data",
            "Name": "myeventgridvol",
            "Options": {},
            "Scope": "local"
          }
   ]
   ```
1. Agregue el grupo **Usuarios** al valor al que apunta el **punto de montaje** como se indica a continuación:
    1. Inicie el explorador de archivos.
    1. Navegue hasta la carpeta a la que apunta el **punto de montaje**.
    1. Haga clic con el botón derecho y seleccione **Propiedades**.
    1. Seleccione **Seguridad**.
    1. En *Nombre de grupo o usuario, seleccione **Editar**.
    1. Seleccione **Agregar**, escriba `Users`, seleccione **Comprobar nombres** y, a continuación, seleccione **Aceptar**.
    1. En los *permisos para usuarios*, seleccione **Modificar** y luego **Aceptar**.
1. Use los **enlaces** para montar este volumen y volver a implementar el módulo de Event Grid desde Azure Portal.

   Por ejemplo,

    ```json
        {
              "Env": [
                "inbound__serverAuth__tlsPolicy=strict",
                "inbound__serverAuth__serverCert__source=IoTEdge",
                "inbound__clientAuth__sasKeys__enabled=false",
                "inbound__clientAuth__clientCert__enabled=true",
                "inbound__clientAuth__clientCert__source=IoTEdge",
                "inbound__clientAuth__clientCert__allowUnknownCA=true",
                "outbound__clientAuth__clientCert__enabled=true",
                "outbound__clientAuth__clientCert__source=IoTEdge",
                "outbound__webhook__httpsOnly=true",
                "outbound__webhook__skipServerCertValidation=false",
                "outbound__webhook__allowUnknownCA=true"
              ],
              "HostConfig": {
                "Binds": [
                  "<your-volume-name-here>:C:\\app\\metadataDb"
                ],
                "PortBindings": {
                  "4438/tcp": [
                     {
                        "HostPort": "4438"
                     }
                  ]
                }
              }
        }
    ```

   >[!IMPORTANT]
   >No cambie la segunda parte del valor de enlace. Apunta a una ubicación concreta dentro del módulo. Para el módulo de Event Grid en Windows, debe ser **C:\\app\\metadataDb**.


    Por ejemplo,

    ```json
    {
        "Env": [
            "inbound__serverAuth__tlsPolicy=strict",
            "inbound__serverAuth__serverCert__source=IoTEdge",
            "inbound__clientAuth__sasKeys__enabled=false",
            "inbound__clientAuth__clientCert__enabled=true",
            "inbound__clientAuth__clientCert__source=IoTEdge",
            "inbound__clientAuth__clientCert__allowUnknownCA=true",
            "outbound__clientAuth__clientCert__enabled=true",
            "outbound__clientAuth__clientCert__source=IoTEdge",
            "outbound__webhook__httpsOnly=true",
            "outbound__webhook__skipServerCertValidation=false",
            "outbound__webhook__allowUnknownCA=true"
         ],
         "HostConfig": {
            "Binds": [
                "myeventgridvol:C:\\app\\metadataDb",
                "C:\\myhostdir2:C:\\app\\eventsDb"
             ],
             "PortBindings": {
                    "4438/tcp": [
                         {
                            "HostPort": "4438"
                          }
                    ]
              }
         }
    }
    ```

## <a name="persistence-via-host-directory-mount"></a>Persistencia mediante el montaje de directorios de host

En lugar de montar un volumen, puede crear un directorio en el sistema host y montar ese directorio.

1. En el sistema de archivos host, cree un directorio ejecutando el comando siguiente.

   ```sh
   mkdir <your-directory-name-here>
   ```

   Por ejemplo,

   ```sh
   mkdir C:\myhostdir
   ```
1. Use los **enlaces** para montar el directorio y volver a implementar el módulo de Event Grid desde Azure Portal.

    ```json
    {
         "HostConfig": {
            "Binds": [
                "<your-directory-name-here>:C:\\app\\metadataDb"
             ]
         }
    }
    ```

    >[!IMPORTANT]
    >No cambie la segunda parte del valor de enlace. Apunta a una ubicación concreta dentro del módulo. Para el módulo de Event Grid en Windows, debe ser **C:\\app\\metadataDb**.

    Por ejemplo,

    ```json
    {
        "Env": [
            "inbound__serverAuth__tlsPolicy=strict",
            "inbound__serverAuth__serverCert__source=IoTEdge",
            "inbound__clientAuth__sasKeys__enabled=false",
            "inbound__clientAuth__clientCert__enabled=true",
            "inbound__clientAuth__clientCert__source=IoTEdge",
            "inbound__clientAuth__clientCert__allowUnknownCA=true",
            "outbound__clientAuth__clientCert__enabled=true",
            "outbound__clientAuth__clientCert__source=IoTEdge",
            "outbound__webhook__httpsOnly=true",
            "outbound__webhook__skipServerCertValidation=false",
            "outbound__webhook__allowUnknownCA=true"
         ],
         "HostConfig": {
            "Binds": [
                "C:\\myhostdir:C:\\app\\metadataDb",
                "C:\\myhostdir2:C:\\app\\eventsDb"
             ],
             "PortBindings": {
                    "4438/tcp": [
                         {
                            "HostPort": "4438"
                          }
                    ]
              }
         }
    }
    ```
## <a name="persist-events"></a>Conservación de eventos

Para habilitar la conservación de eventos, primero debe habilitar la conservación de eventos mediante el montaje de volúmenes o de directorios de host, siguiendo las secciones anteriores.

Aspectos importantes que se deben tener en cuenta sobre la conservación de eventos:

* La conservación de eventos se habilita en cada suscripción de evento y puede activarse una vez que se ha montado un volumen o directorio.
* La conservación de eventos se configura en una suscripción de eventos en el momento de su creación y no se puede modificar una vez creada la suscripción de eventos. Para cambiar la conservación de eventos, debe eliminar y volver a crear la suscripción de eventos.
* La conservación de eventos casi siempre es más lenta que en las operaciones de memoria; sin embargo, la diferencia de velocidad depende en gran medida de las características de la unidad. El equilibrio entre la velocidad y la confiabilidad es inherente a todos los sistemas de mensajería, aunque solo es obvio a gran escala.

Para habilitar la conservación de eventos en una suscripción de eventos, establezca `persistencePolicy` en `true`:

 ```json
        {
          "properties": {
            "persistencePolicy": {
              "isPersisted": "true"
            },
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-webhook-url>"
              }
            }
          }
        }
 ```