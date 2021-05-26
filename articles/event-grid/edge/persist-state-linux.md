---
title: Conservación del estado en Linux - Azure Event Grid IoT Edge | Microsoft Docs
description: Persistencia de los metadatos en Linux
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: 28f9e8847fe6a3aad1c68e09dacfbd1b8fc81e77
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367733"
---
# <a name="persist-state-in-linux"></a>Conservación del estado en Linux

Los temas y las suscripciones creados en el módulo de Event Grid se almacenan de forma predeterminada en el sistema de archivos de contenedor. Sin persistencia, si se vuelve a implementar el módulo, se perderán todos los metadatos creados. Para conservar los datos en las implementaciones y reinicios, debe conservar los datos fuera del sistema de archivos de contenedor.

De forma predeterminada, solo se conservan los metadatos y los eventos todavía se almacenan en memoria para mejorar el rendimiento. Siga la sección Conservación de eventos para habilitar también la conservación de eventos.

En este artículo se proporcionan los pasos para implementar el módulo de Event Grid con persistencia en las implementaciones de Linux.

> [!NOTE]
>El módulo de Event Grid se ejecuta como un usuario con pocos privilegios con UID `2000` y nombre `eventgriduser`.

## <a name="persistence-via-volume-mount"></a>Persistencia a través del montaje del volumen

 [Los volúmenes de Docker](https://docs.docker.com/storage/volumes/) se usan para conservar los datos en las implementaciones. Puede permitir que Docker cree automáticamente un volumen con nombre como parte de la implementación del módulo de Event Grid. Esta opción es la más sencilla. Puede especificar el nombre del volumen que se va a crear en la sección **Binds** de la siguiente manera:

```json
  {
     "HostConfig": {
          "Binds": [
                 "<your-volume-name-here>:/app/metadataDb"
           ]
      }
   }
```

>[!IMPORTANT]
>No cambie la segunda parte del valor de enlace. Apunta a una ubicación concreta dentro del módulo. Para el módulo de Event Grid en Linux, debe ser **/app/metadataDb**.

Por ejemplo, la siguiente configuración producirá la creación del volumen **egmetadataDbVol** donde se conservarán los metadatos.

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
      "egmetadataDbVol:/app/metadataDb",
      "egdataDbVol:/app/eventsDb"
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

En lugar de montar un volumen, puede crear un directorio en el sistema host y montar ese directorio.

## <a name="persistence-via-host-directory-mount"></a>Persistencia mediante el montaje de directorios de host

En lugar de un volumen de Docker, también tiene la opción de montar una carpeta de host.

1. En primer lugar, cree un usuario con el nombre **eventgriduser** y el identificador **2000** en el equipo host mediante la ejecución del siguiente comando:

    ```sh
    sudo useradd -u 2000 eventgriduser
    ```
1. En el sistema de archivos host, cree un directorio ejecutando el comando siguiente.

   ```sh
   md <your-directory-name-here>
   ```

    Por ejemplo, al ejecutar el siguiente comando, se creará un directorio llamado **myhostdir**.

    ```sh
    md /myhostdir
    ```
1. A continuación, cree el propietario **eventgriduser** de esta carpeta ejecutando el siguiente comando.

   ```sh
   sudo chown eventgriduser:eventgriduser -hR <your-directory-name-here>
   ```

    Por ejemplo,

    ```sh
    sudo chown eventgriduser:eventgriduser -hR /myhostdir
    ```
1. Use **Binds** para montar el directorio y volver a implementar el módulo de Event Grid desde Azure Portal.

    ```json
    {
         "HostConfig": {
            "Binds": [
                "<your-directory-name-here>:/app/metadataDb",
                "<your-directory-name-here>:/app/eventsDb",
             ]
         }
    }
    ```

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
                  "/myhostdir:/app/metadataDb",
                  "/myhostdir2:/app/eventsDb"
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
    >No cambie la segunda parte del valor de enlace. Apunta a una ubicación concreta dentro del módulo. Para el módulo de Event Grid en Linux, debe ser **/app/metadataDb** y **/app/eventsDb**.


## <a name="persist-events"></a>Conservación de eventos

Para habilitar la conservación de eventos, primero debe habilitar la conservación de metadatos mediante el montaje de volúmenes o el montaje de directorios de host, siguiendo las secciones anteriores.

Aspectos importantes que se deben tener en cuenta sobre la conservación de eventos:

* La conservación de eventos se habilita en cada suscripción de evento y puede activarse una vez que se ha montado un volumen o directorio.
* La conservación de eventos se configura en una suscripción de eventos en el momento de su creación y no se puede modificar una vez creada la suscripción de eventos. Para cambiar la conservación de eventos, debe eliminar y volver a crear la suscripción de eventos.
* La conservación de eventos casi siempre es más lenta que en las operaciones de memoria; sin embargo, la diferencia de velocidad depende en gran medida de las características de la unidad. El equilibrio entre la velocidad y la confiabilidad es inherente a todos los sistemas de mensajería, aunque generalmente solo es obvio a gran escala.

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
