---
title: Flujos de trabajo de acciones de GitHub para Azure Static Web Apps
description: Obtenga información sobre cómo usar repositorios de GitHub para configurar la implementación continua en Azure Static Web Apps.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 04/09/2021
ms.author: cshoe
ms.openlocfilehash: f47ad3d63ca99ffbe095498d2db7646e8a4a3ae7
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110072485"
---
# <a name="github-actions-workflows-for-azure-static-web-apps"></a>Flujos de trabajo de acciones de GitHub para Azure Static Web Apps

Cuando se crea un nuevo recurso de Azure Static Web Apps, Azure genera un flujo de trabajo de Acciones de GitHub para controlar la implementación continua de la aplicación. Un archivo YAML controla el flujo de trabajo. En este artículo se detalla la estructura y las opciones del archivo de flujo de trabajo.

Los [desencadenadores](#triggers), que ejecutan [trabajos](#jobs) definidos mediante [pasos](#steps) individuales inician las implementaciones.

> [!NOTE]
> Azure Static Web Apps también admite Azure DevOps. Consulte [Publicación con Azure DevOps](publish-devops.md) para obtener información sobre cómo configurar una canalización.

## <a name="file-location"></a>Ubicación del archivo

Cuando vincula el repositorio de GitHub a Azure Static Web Apps, se agrega un archivo de flujo de trabajo al repositorio.

Siga los pasos a continuación para ver el archivo de flujo de trabajo generado.

1. Abra el repositorio de la aplicación en GitHub.
1. En la pestaña _Código_, haga clic en la carpeta `.github/workflows`.
1. Haga clic en el archivo con un nombre similar a `azure-static-web-apps-<RANDOM_NAME>.yml`.

El archivo YAML del repositorio será similar al ejemplo siguiente:

```yml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
          repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
          action: 'upload'
          ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
          app_location: '/' # App source code path
          api_location: 'api' # Api source code path - optional
          output_location: 'dist' # Built app content directory - optional
          ###### End of Repository/Build Configurations ######

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
      - name: Close Pull Request
        id: closepullrequest
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
          action: 'close'
```

## <a name="triggers"></a>Desencadenadores

Las acciones de GitHub de tipo [desencadenador](https://help.github.com/actions/reference/events-that-trigger-workflows) notifican a un flujo de trabajo de acciones de GitHub para que ejecute un trabajo basado en desencadenadores de eventos. Los desencadenadores se muestran al usar la propiedad `on` en el archivo de flujo de trabajo.

```yml
on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main
```

Mediante la configuración asociada a la propiedad `on`, puede definir qué ramas desencadenan un trabajo y establecer desencadenadores para que se activen con diferentes estados de solicitud de incorporación de cambios ("pull request").

En este ejemplo, se inicia un flujo de trabajo a medida que cambia la rama _main_. Entre los cambios que inician el flujo de trabajo se incluyen la inserción ("push") de confirmaciones y la apertura de solicitudes de incorporación de cambios ("pull request") en la rama elegida.

## <a name="jobs"></a>Trabajos

Cada desencadenador de eventos requiere un controlador de eventos. Los [trabajos](https://help.github.com/actions/reference/workflow-syntax-for-github-actions#jobs) definen lo que ocurre cuando se desencadena un evento.

En el archivo de flujo de trabajo de Static Web Apps, hay dos trabajos disponibles.

| Nombre                     | Descripción                                                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------------------------- |
| `build_and_deploy_job`   | Se ejecuta al insertar ("push") confirmaciones o abrir una solicitud de incorporación de cambios ("pull request") en la rama mostrada en la propiedad `on`.          |
| `close_pull_request_job` | SOLO se ejecuta cuando se cierra una solicitud de incorporación de cambios, lo que quita el entorno de ensayo creado a partir de solicitudes de incorporación de cambios. |

## <a name="steps"></a>Pasos

Los pasos son tareas secuenciales de un trabajo. Un paso lleva a cabo acciones como la instalación de dependencias, la ejecución de pruebas y la implementación de la aplicación a la fase de producción.

Un archivo de flujo de trabajo define los siguientes pasos.

| Trabajo                      | Pasos                                                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `build_and_deploy_job`   | <ol><li>Extrae el repositorio del entorno de la acción.<li>Compila e implementa el repositorio en Azure Static Web Apps.</ol> |
| `close_pull_request_job` | <ol><li>Notifica a Azure Static Web Apps que se ha cerrado una solicitud de incorporación de cambios ("pull request").</ol>                                                        |

## <a name="build-and-deploy"></a>Compilación e implementación

El paso denominado `Build and Deploy` se compila e implementa en la instancia de Azure Static Web Apps. En la sección `with`, puede personalizar los siguientes valores para su implementación.

```yml
with:
  azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
  repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
  action: 'upload'
  ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
  app_location: '/' # App source code path
  api_location: 'api' # Api source code path - optional
  output_location: 'dist' # Built app content directory - optional
  ###### End of Repository/Build Configurations ######
```

[!INCLUDE [static-web-apps-folder-structure](../../includes/static-web-apps-folder-structure.md)]

Azure Static Web Apps configura automáticamente los valores de `repo_token`, `action` y `azure_static_web_apps_api_token`, y estos no deben cambiarse manualmente.

## <a name="custom-build-commands"></a>Comandos de compilación personalizados

Puede tener un control más preciso sobre los comandos que se ejecutan durante una implementación. Los siguientes comandos se pueden definir en la sección `with` de un trabajo.

La implementación siempre llama a `npm install` antes de cualquier comando personalizado.

| Get-Help             | Descripción                                                                                                                                                                                                                                                                                                                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `app_build_command` | Define un comando personalizado para que se ejecute durante la implementación de la aplicación de contenido estático.<br><br>Por ejemplo, para configurar una compilación de producción para una aplicación angular, cree un script NPM denominado `build-prod` para ejecutar `ng build --prod` y escriba `npm run build-prod` como comando personalizado. Si se deja en blanco, el flujo de trabajo intenta ejecutar los comandos `npm run build` o `npm run build:azure`. |
| `api_build_command` | Define un comando personalizado para que se ejecute durante la implementación de la aplicación de la API de Azure Functions.                                                                                                                                                                                                                                                                                                  |

## <a name="skip-app-build"></a>Omisión de la compilación de la aplicación

Si necesita un control total sobre cómo se compila la aplicación de front-end, puede agregar pasos de compilación personalizados en el flujo de trabajo. A continuación, puede configurar la acción Static Web Apps para omitir el proceso de compilación automática e implementar simplemente la aplicación que se creó en un paso anterior.

Para omitir la creación de la aplicación, establezca `skip_app_build` en `true` y `app_location` en la ubicación de la carpeta que se implementará.

```yml
with:
  azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
  repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
  action: 'upload'
  ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
  app_location: 'dist' # Application build output generated by a previous step
  api_location: 'api' # Api source code path - optional
  output_location: '' # Leave this empty
  skip_app_build: true
  ###### End of Repository/Build Configurations ######
```

| Propiedad         | Descripción                                                 |
| ---------------- | ----------------------------------------------------------- |
| `skip_app_build` | Establezca el valor en `true` para omitir la creación de la aplicación de front-end. |

> [!NOTE]
> Solo puede omitir la compilación de la aplicación de front-end. Si la aplicación tiene una API, la seguirá compilando la acción Static Web Apps de GitHub.

## <a name="environment-variables"></a>Variables de entorno

Puede establecer variables de entorno para la compilación a través de la sección `env` de la configuración de un trabajo.

```yaml
jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: 'upload'
          ###### Repository/Build Configurations
          app_location: '/'
          api_location: 'api'
          output_location: 'public'
          ###### End of Repository/Build Configurations ######
        env: # Add environment variables here
          HUGO_VERSION: 0.58.0
```

## <a name="monorepo-support"></a>Compatibilidad con monorrepositorios

Un monorrepositorio es un repositorio que contiene código para más de una aplicación. De manera predeterminada, un archivo de flujo de trabajo de Static Web Apps hace un seguimiento de todos los archivos de un repositorio, pero se puede ajustar para que tenga como destino una sola aplicación. Por lo tanto, para los monorrepositorios, cada aplicación estática tiene su propio archivo de configuración que vive en paralelo en la carpeta _.github/workflows_ del repositorio.

```files
├── .github
│   └── workflows
│       ├── azure-static-web-apps-purple-pond.yml
│       └── azure-static-web-apps-yellow-shoe.yml
│
├── app1  👉 controlled by: azure-static-web-apps-purple-pond.yml
├── app2  👉 controlled by: azure-static-web-apps-yellow-shoe.yml
│
├── api1  👉 controlled by: azure-static-web-apps-purple-pond.yml
├── api2  👉 controlled by: azure-static-web-apps-yellow-shoe.yml
│
└── README.md
```

Para que un archivo de flujo de trabajo tenga como destino una única aplicación, especifique las rutas de acceso en las secciones `push` y `pull_request`.

En el ejemplo siguiente se muestra cómo agregar un nodo `paths` a las secciones `push` y `pull_request` de un archivo denominado _azure-static-web-apps-purple-pond.yml_.

```yml
on:
  push:
    branches:
      - main
    paths:
      - app1/**
      - api1/**
      - .github/workflows/azure-static-web-apps-purple-pond.yml
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main
    paths:
      - app1/**
      - api1/**
      - .github/workflows/azure-static-web-apps-purple-pond.yml
```

En esta instancia, solo los cambios realizados en los siguientes archivos desencadenan una nueva compilación:

- Cualquier archivo dentro de la carpeta _app1_
- Cualquier archivo dentro de la carpeta _api1_
- Cambios en el archivo de flujo de trabajo _azure-static-web-apps-purple-pond.yml_ de la aplicación

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Revisión de las solicitudes de incorporación de cambios en entornos de pre-producción](review-publish-pull-requests.md)
