---
title: Alternativas para el portal para desarrolladores autohospedado
titleSuffix: Azure API Management
description: Obtenga información sobre los enfoques alternativos que puede usar al autohospedar un portal para desarrolladores en Azure API Management.
author: dlepow
ms.author: apimpm
ms.date: 03/25/2021
ms.service: api-management
ms.topic: how-to
ms.openlocfilehash: 207f5cbfb0e8f6d9d52920f7e4c72d7e663489fc
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459356"
---
# <a name="alternative-approaches-to-self-host-developer-portal"></a>Enfoques alternativos al autohospedaje del portal para desarrolladores

Hay varios enfoques alternativos que puede explorar al [autohospedar un portal para desarrolladores](developer-portal-self-host.md):

* Usar compilaciones de producción del diseñador y del publicador.

* Usar una aplicación de Azure Functions para publicar el portal.

* Presentar los archivos del portal con una instancia de Content Delivery Network (CDN) para reducir los tiempos de carga de páginas.

En este artículo se proporciona información sobre cada uno de estos enfoques. 

Si aún no lo ha hecho, configure un [entorno local](developer-portal-self-host.md#step-1-set-up-local-environment) para la versión más reciente del portal para desarrolladores.

## <a name="build-for-production"></a>Compilación para producción

Si quiere hospedar el entorno de desarrollo del portal en línea con fines de colaboración, use compilaciones de producción del diseñador y del publicador. Las compilaciones de producción agrupan los archivos, excluyen los mapas de origen, etc.

Para crear una agrupación en el directorio `./dist/designer`, ejecute el comando:

```sh
npm run build-designer
```

El resultado es una aplicación de página única, por lo que todavía puede implementarla en un host web estático, como el sitio web estático de Azure Blob Storage.

De manera similar, coloque un publicador compilado y optimizado en la carpeta `./dist/publisher`:

```sh
npm run build-publisher
```

## <a name="use-function-app-to-publish-the-portal"></a>Uso de una aplicación de funciones para publicar el portal

Ejecute el paso de publicación en la nube como alternativa a ejecutarlo localmente.

Para implementar la publicación con una aplicación de Azure Functions, necesita los siguientes requisitos previos:

- [Cree una instancia de Azure Functions](../azure-functions/functions-get-started.md). La función tiene que ser una función en lenguaje JavaScript.
- Instale Azure Functions Core Tools:
    ```console
    npm install –g azure-function-core-tools
    ```

### <a name="step-1-configure-output-storage"></a>Paso 1: Configuración del almacenamiento de salida

Cargue el contenido directamente en el sitio web de hospedaje (contenedor "$web" del almacenamiento de salida), en lugar de una carpeta local. Configure este cambio en el archivo `./src/config.publish.json`:

```json
{
   ...
   "outputBlobStorageContainer": "$web",
   "outputBlobStorageConnectionString": "DefaultEndpointsProtocol=...",
   ...
}
```

### <a name="step-2-build-and-deploy-the-function-app"></a>Paso 2: Compilación e implementación de la aplicación de funciones

Hay una función de desencadenador HTTP de ejemplo en la carpeta `./examples`. Para compilarla y colocarla en `./dist/function`, ejecute el siguiente comando:

```sh
npm run build-function
```

A continuación, inicie sesión en la CLI de Azure e impleméntela:

```azurecli
az login
cd ./dist/function
func azure functionapp publish <function app name>
```

Una vez que se haya implementado, puede invocarla con una llamada HTTP:

```sh
curl -X POST https://<function app name>.azurewebsites.net/api/publish
```

## <a name="hosting-and-cdn"></a>Hospedaje y CDN

En [Autohospedaje de un portal para desarrolladores](developer-portal-self-host.md), se recomienda usar una cuenta de almacenamiento de Azure para hospedar el sitio web. Sin embargo, puede publicar los archivos a través de cualquier solución, incluidos los servicios de proveedores de hospedaje.

También puede presentar los archivos con una instancia de Content Delivery Network (CDN) para reducir los tiempos de carga de páginas. Se recomienda usar [Azure CDN](https://azure.microsoft.com/services/cdn/).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)
