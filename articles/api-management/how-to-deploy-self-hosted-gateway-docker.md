---
title: Implementación de una puerta de enlace autohospedada en Docker | Microsoft Docs
description: Aprenda a implementar un componente de puerta de enlace autohospedada de Azure API Management en Docker
services: api-management
documentationcenter: ''
author: vladvino
manager: gwallace
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 04/19/2021
ms.author: apimpm
ms.openlocfilehash: 531421726bc1e081d85eca9d535267520d3fea5f
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107725618"
---
# <a name="deploy-an-azure-api-management-self-hosted-gateway-to-docker"></a>Implementación de una puerta de enlace autohospedada de Azure API Management en Docker

En este artículo se detallan los pasos para implementar un componente de puerta de enlace autohospedada de Azure API Management en un entorno de Docker.

> [!NOTE]
> El hospedaje de una puerta de enlace autohospedada en Docker es más adecuado para los casos de uso de evaluación y desarrollo. Kubernetes está recomendado para un uso de producción. Consulte [este](how-to-deploy-self-hosted-gateway-kubernetes.md) documento para aprender a implementar una puerta de enlace autohospedada en Kubernetes.

## <a name="prerequisites"></a>Requisitos previos

- Completar la guía de inicio rápido siguiente: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)
- Cree un entorno de Docker. [Docker for Desktop](https://www.docker.com/products/docker-desktop) es una buena opción para fines de desarrollo y evaluación. Consulte la [documentación de Docker](https://docs.docker.com) para obtener información sobre todas las ediciones de Docker, sus características y la documentación completa de Docker.
- [Aprovisione un recurso de puerta de enlace en la instancia de API Management](api-management-howto-provision-self-hosted-gateway.md)

> [!NOTE]
> La puerta de enlace autohospedada se empaqueta como un contenedor de Docker basado en Linux x86-64.

## <a name="deploy-the-self-hosted-gateway-to-docker"></a>Implementación de la puerta de enlace autohospedada en Docker

1. Seleccione **Puertas de enlace** en **Deployment and infrastructure** (Implementación e infraestructura).
2. Seleccione el recurso de puerta de enlace que desea implementar.
3. Seleccione **Implementación**.
4. Tenga en cuenta que en el cuadro de texto **Token** se generará automáticamente un token de acceso con los valores predeterminados de **Expiración** y **Clave secreta**. Si es necesario, elija los valores deseados en uno o ambos controles para generar un nuevo token.
5. Asegúrese de que **Docker** esté seleccionado en **Scripts de implementación**.
6. Seleccione el vínculo del archivo **env.conf** junto al **Entorno** para descargar el archivo.
7. Seleccione el icono **Copiar** situado en el extremo derecho del cuadro de texto **Ejecutar** para copiar el comando de Docker en el portapapeles.
8. Pegue el comando en la ventana de terminal (o comando). Ajuste las asignaciones de puerto y el nombre del contenedor según sea necesario. Tenga en cuenta que el comando da por supuesto que el archivo de entorno descargado está presente en el directorio actual.
   ```
       docker run -d -p 80:8080 -p 443:8081 --name <gateway-name> --env-file env.conf mcr.microsoft.com/azure-api-management/gateway:<tag>
   ```
9. Ejecute el comando. El comando indica al entorno de Docker que ejecute el contenedor mediante la [imagen de contenedor](https://aka.ms/apim/sputnik/dhub) descargada del Registro de contenedor de Microsoft y que asigne los puertos HTTP (8080) y HTTPS (8081) del contenedor a los puertos 80 y 443 del host.
10. Ejecute el siguiente comando para comprobar si se está ejecutando el contenedor de puerta de enlace:
    ```console
    docker ps
    CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                         NAMES
    895ef0ecf13b        mcr.microsoft.com/azure-api-management/gateway:latest   "/bin/sh -c 'dotnet …"   5 seconds ago       Up 3 seconds        0.0.0.0:80->8080/tcp, 0.0.0.0:443->8081/tcp   my-gateway
    ```
10. Vuelva a Azure Portal, haga clic en **Información general** y confirme que el contenedor de puerta de enlace autohospedada que acaba de implementar está notificando un estado correcto.

    ![estado de la puerta de enlace](media/how-to-deploy-self-hosted-gateway-docker/status.png)

> [!TIP]
> Use el comando <code>console docker container logs <gateway-name></code> para ver una instantánea del registro de la puerta de enlace autohospedada.
>
> Use el comando <code>docker container logs --help</code> para ver todas las opciones de visualización del registro.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la puerta de enlace autohospedada, consulte [Introducción a la puerta de enlace autohospedada de Azure API Management](self-hosted-gateway-overview.md).
* [Configure el nombre de dominio personalizado para la puerta de enlace autohospedada](api-management-howto-configure-custom-domain-gateway.md).
