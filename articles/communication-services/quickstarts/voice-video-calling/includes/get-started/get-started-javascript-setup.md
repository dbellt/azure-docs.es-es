---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: cc1912deb224324eb3092322a3ca08043c1afbf3
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111560372"
---
## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

```console
mkdir calling-quickstart && cd calling-quickstart
```

Ejecute `npm init -y` para crear un archivo **package.json** con la configuración predeterminada.

```console
npm init -y
```

### <a name="install-the-package"></a>Instalar el paquete

Use el comando `npm install` para instalar el SDK de llamadas de Azure Communication Services para JavaScript.

```console
npm install @azure/communication-common --save
npm install @azure/communication-calling@1.1.0-beta.1 --save
```

Para este inicio rápido se recomiendan las siguientes versiones de webpack:

```console
"webpack": "^4.42.0",
"webpack-cli": "^3.3.11",
"webpack-dev-server": "^3.10.3"
```

La opción `--save` muestra la biblioteca como dependencia en el archivo **package.json**.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Esta guía de inicio rápido usa webpack para agrupar los recursos de la aplicación. Ejecute el siguiente comando para instalar los paquetes de npm webpack, webpack-cli y webpack-dev-server, y mostrarlos como dependencias de desarrollo en el archivo **package.json**:

```console
npm install webpack@4.42.0 webpack-cli@3.3.11 webpack-dev-server@3.10.3 --save-dev
```

Cree un archivo **index.html** en el directorio raíz del proyecto. Este archivo lo usaremos para configurar un diseño básico que permitirá al usuario realizar una llamada.
