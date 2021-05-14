---
title: Implementación de una aplicación desde el origen a Red Hat OpenShift en Azure
description: Obtenga información sobre cómo implementar una aplicación para Red Hat OpenShift en Azure desde el código fuente mediante la estrategia de compilación de origen a imagen (S2I).
author: sabbour
ms.author: asabbour
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 4/5/2021
keywords: aro, openshift, red hat, s2i, source to image
ms.openlocfilehash: 270e5d915e2a77cdb4377a616abc97a836a09710
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107558810"
---
# <a name="deploy-an-application-from-source-to-azure-red-hat-openshift"></a>Implementación de una aplicación desde el origen a Red Hat OpenShift en Azure

En este artículo, va a implementar una aplicación en un clúster de Red Hat OpenShift en Azure desde el código fuente mediante una compilación de origen a imagen (S2I). [Origen a imagen (S2I)](https://docs.openshift.com/container-platform/4.6/builds/understanding-image-builds.html#builds-strategy-s2i-build_understanding-image-builds) es un proceso para compilar imágenes de contenedor reproducibles a partir del código fuente. S2I produce imágenes listas para su ejecución mediante la inserción de código fuente en una imagen de contenedor y dejando que el contenedor prepare ese código fuente para su ejecución. Puede hacer que OpenShift compile una aplicación desde el origen para implementarla, por lo que no tiene que construir un contenedor con cada cambio. A continuación, OpenShift puede compilar e implementar nuevas versiones automáticamente cuando se le notifique de los cambios en el código fuente.

## <a name="before-you-begin"></a>Antes de comenzar

[!INCLUDE [aro-howto-beforeyoubegin](includes/aro-howto-before-you-begin.md)]

## <a name="create-a-project"></a>Crear un proyecto

Para crear un nuevo proyecto denominado `demoproject`, ejecute el comando:

```azurecli-interactive
oc new-project demoproject
```

Debería aparecer una salida similar a la siguiente:

```output
Now using project "demoproject" on server "https://api.wzy5hg7x.eastus.aroapp.io:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app django-psql-example

to build a new example application in Python. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```

## <a name="launch-the-web-console"></a>Inicio de la consola web

Para averiguar la dirección URL de la consola web del clúster, ejecute:

```azurecli-interactive
 az aro show \
    --name <cluster name> \
    --resource-group <resource group> \
    --query "consoleProfile.url" -o tsv
```

Debería obtener una dirección URL similar a esta:

```output
https://console-openshift-console.apps.wzy5hg7x.eastus.aroapp.io/
```

Inicie la dirección URL de la consola en un explorador e inicie sesión con las credenciales de `kubeadmin`.

:::image type="content" source="media/login.png" alt-text="Pantalla de inicio de sesión de Red Hat OpenShift en Azure":::

Cambie a la perspectiva *Developer* (Desarrollador) en lugar de la perspectiva *Administrator* (Administrador) en el menú de la izquierda y seleccione `demoproject` en la lista de proyectos. Debería estar entonces en la página *Topología* del proyecto.

:::image type="content" source="media/s2i/project-topology.png" alt-text="Topología de proyecto de Red Hat OpenShift en Azure":::

Como el proyecto está vacío, no se deben encontrar cargas de trabajo y se le presentarán varias opciones para implementar una aplicación.

## <a name="deploying-using-the-web-console"></a>Implementación mediante la consola web

En las opciones presentadas para implementar una aplicación, seleccione *From Git* (Desde Git). Esto le llevará a la página *Import from Git* (Importar desde Git). Utilice `https://github.com/sclorg/django-ex.git` como **dirección URL del repositorio de Git**. La aplicación web de ejemplo se implementa con el lenguaje de programación Python.

:::image type="content" source="media/s2i/from-git.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git":::

> [!NOTE]
> OpenShift detecta que se trata de un proyecto de Python y selecciona la imagen del compilador adecuada.

Desplácese hacia abajo, hacia *Advanced Options* (Opciones avanzadas) y asegúrese de que la opción **Create a route to the application** (Crear una ruta a la aplicación) está activada. Esta acción creará una ruta de OpenShift, una manera de exponer un servicio al darle un nombre de host accesible externamente.

:::image type="content" source="media/s2i/from-git-route.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: configuración de rutas":::

Cuando esté listo, en la parte inferior de la página, haga clic en **Create** (Crear). Esto creará recursos para administrar la compilación e implementación de la aplicación. A continuación, se le redirigirá a la información general de la topología del proyecto.

:::image type="content" source="media/s2i/demo-app-topology.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: topología":::

La información general de la topología proporciona una representación visual de la aplicación que ha implementado. Con esta vista, puede ver la estructura general de la aplicación.

Se puede hacer clic en el icono de Git para ir al repositorio de Git desde el que se creó el código fuente de la aplicación. El icono que se muestra en la parte inferior izquierda muestra el estado de compilación de la aplicación. Al hacer clic en este icono, se le llevará a la sección de detalles de compilación. Si la aplicación ha expuesto rutas, se puede hacer clic en el icono de la parte superior derecha para abrir la dirección URL de la ruta de aplicación que se creó.

Mientras la aplicación se escala o reduce verticalmente, inicia lanzamientos y vuelve a crear pods, la representación de la aplicación en la vista de topología se animará para ofrecer una vista en tiempo real de lo que está sucediendo.

Al hacer clic en el icono de la aplicación, se mostrarán más detalles, como se muestra a continuación.

:::image type="content" source="media/s2i/demo-app-details.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: detalles":::

## <a name="viewing-the-builder-logs"></a>Visualización de registros del compilador

Una vez iniciada la compilación, haga clic en el vínculo *View Logs* (Ver registros) que se muestra en el panel *Resources* (Recursos).

:::image type="content" source="media/s2i/demo-app-build-logs.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: registros de compilación":::

Esto le permitirá supervisar el progreso de la compilación mientras se ejecuta. La imagen del generador, Python en este caso, insertará el código fuente de la aplicación en la imagen final antes de insertarlo en el registro de imágenes interno de OpenShift. La compilación se habrá completado correctamente cuando vea un mensaje final de inserción correcta.

## <a name="accessing-the-application"></a>Acceso a la aplicación

Una vez completada la compilación de la imagen de aplicación, se implementará.

Haga clic en *Topology* (Topology) en la barra de menús de la izquierda para volver a la vista de topología del proyecto. Cuando creó la aplicación mediante la consola web, se creó automáticamente una *ruta* para la aplicación y se expondrá fuera del clúster. La dirección URL que se puede usar para acceder a la aplicación desde un explorador web estaba visible en la pestaña *Resources* (Recursos) de la aplicación que ha visto anteriormente.

Desde la vista de topología, puede acceder a la dirección URL de la aplicación implementada haciendo clic en el icono situado en la parte superior derecha del anillo. Una vez completada la implementación, haga clic en el icono y verá la aplicación que ha implementado.

:::image type="content" source="media/s2i/demo-app-browse.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: examen de la aplicación":::

## <a name="deploying-using-the-command-line"></a>Implementación mediante la línea de comandos

Ya ha aprendido a implementar una aplicación mediante la consola web, ahora vamos a implementar la misma aplicación web, pero esta vez con la herramienta de línea de comandos `oc`.

Ejecute el siguiente comando para eliminar el proyecto y comenzar de nuevo:

```azurecli-interactive
oc delete project demoproject
```

Debería recibir un mensaje de confirmación que indica que `demoproject` se eliminó.

```output
project.project.openshift.io "demoproject" deleted
```

Vuelva a crear `demoproject` mediante la ejecución de:

```azurecli-interactive
oc new-project demoproject
```

Dentro del proyecto, cree una nueva aplicación desde el origen en GitHub y especifique el generador de S2I para la versión más reciente de Python proporcionada.

```azurecli-interactive
oc new-app python:latest~https://github.com/sclorg/django-ex.git
```

Debe mostrar una salida similar a la siguiente:

```output
--> Found image 8ec6f0d (4 weeks old) in image stream "openshift/python" under tag "latest" for "python:latest"

    Python 3.8
    ----------
   [...]

    Tags: builder, python, python38, python-38, rh-python38

    * A source build using source code from https://github.com/sclorg/django-ex.git will be created
      * The resulting image will be pushed to image stream tag "django-ex:latest"
      * Use 'oc start-build' to trigger a new build
    * This image will be deployed in deployment config "django-ex"
    * Port 8080/tcp will be load balanced by service "django-ex"
      * Other containers can access this service through the hostname "django-ex"

--> Creating resources ...
    imagestream.image.openshift.io "django-ex" created
    buildconfig.build.openshift.io "django-ex" created
    deploymentconfig.apps.openshift.io "django-ex" created
    service "django-ex" created
--> Success
    Build scheduled, use 'oc logs -f bc/django-ex' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/django-ex'
    Run 'oc status' to view your app.
```

OpenShift usará el nombre del repositorio de Git como nombre de la aplicación. Revise el estado de la compilación y la implementación mediante la ejecución de lo siguiente:

```azurecli-interactive
oc status
```

Una vez completada la compilación y la implementación, debería ver una salida similar a la siguiente.

```output
In project demoproject on server https://api.wzy5hg7x.eastus.aroapp.io:6443

svc/django-ex - 172.30.200.50:8080
  dc/django-ex deploys istag/django-ex:latest <-
    bc/django-ex source builds https://github.com/sclorg/django-ex.git on openshift/python:latest
    deployment #1 deployed about a minute ago - 1 pod


2 infos identified, use 'oc status --suggest' to see details.
```

Para exponer la aplicación fuera del clúster de OpenShift, deberá crear una ruta mediante la ejecución de lo siguiente:

```azurecli-interactive
oc expose service/django-ex
```

Debe obtener una confirmación.

```output
route.route.openshift.io/django-ex exposed
```

Para recuperar la dirección URL, ejecute:

```azurecli-interactive
oc get route django-ex
```

Debe recuperar el nombre de host asignado a la ruta que puede usar para ir a la aplicación implementada.

```output
NAME        HOST/PORT                                              PATH   SERVICES    PORT       TERMINATION   WILDCARD
django-ex   django-ex-demoproject.apps.wzy5hg7x.eastus.aroapp.io          django-ex   8080-tcp                 None
```

## <a name="triggering-a-new-binary-build"></a>Desencadenamiento de una nueva compilación binaria

A medida que trabaja en la aplicación, es probable que quiera realizar cambios y verlos implementados. Puede configurar fácilmente un webhook que desencadenará una nueva compilación e implementación con cada confirmación de código. Sin embargo, esto puede no ser deseable, ya que a veces le gustaría ver los cambios sin tener que insertar todos los cambios de código en el repositorio.

En los casos en los que se está iterando rápidamente en los cambios, puede usar lo que se denomina una compilación binaria. Para demostrar este escenario, clone localmente el repositorio de Git para la aplicación mediante la ejecución de lo siguiente:

```azurecli-interactive
git clone https://github.com/sclorg/django-ex.git
```

Esto creará un subdirectorio `django-ex` que contiene el código fuente de la aplicación:

```output
Cloning into 'django-ex'...
remote: Enumerating objects: 980, done.
remote: Total 980 (delta 0), reused 0 (delta 0), pack-reused 980
Receiving objects: 100% (980/980), 276.23 KiB | 4.85 MiB/s, done.
Resolving deltas: 100% (434/434), done.
```

Cambie al nuevo subdirectorio:

```azurecli-interactive
cd django-ex
```

Abra el editor Azure Cloud Shell integrado:

```azurecli-interactive
code welcome/templates/welcome/index.html
```

Desplácese hacia abajo y cambie la línea que dice `Welcome to your Django application on OpenShift` para que diga `Welcome to Azure Red Hat OpenShift`. Guarde el archivo y cierre el editor mediante el menú `...` de la parte superior derecha.

:::image type="content" source="media/s2i/cloudshell-editor.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: edición de la aplicación en el editor de Azure Cloud Shell":::

Ejecute el comando para iniciar una nueva compilación:

```azurecli-interactive
oc start-build django-ex --from-dir=. --wait
```

Al pasar la marca `--from-dir=.`, la línea de comandos de OpenShift cargará el código fuente desde el directorio especificado y, a continuación, iniciará el proceso de compilación e implementación. Debería obtener una salida similar a la siguiente y, después de unos minutos, se debería completar la compilación.

```output
Uploading directory "." as binary input for the build ...
.
Uploading finished
build.build.openshift.io/django-ex-2 started
```

Si actualiza el explorador con la aplicación, debería ver el título actualizado.

:::image type="content" source="media/s2i/demo-app-browse-updated.png" alt-text="Proyecto de Red Hat OpenShift en Azure desde Git: examen de la aplicación actualizada":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con la aplicación, puede ejecutar el siguiente comando para eliminar el proyecto:

```azurecli-interactive
oc delete project demoproject
```

También puede eliminar el clúster; para ello, siga los pasos que aparecen en [Tutorial: Eliminación de un clúster de la versión 4 de Red Hat OpenShift en Azure](./tutorial-delete-cluster.md).

## <a name="next-steps"></a>Pasos siguientes

En esta guía aprendió información sobre lo siguiente:
> [!div class="checklist"]
>
> * Crear un proyecto
> * Implementación de una aplicación desde el código fuente mediante la consola web
> * Implementación de una aplicación desde el código fuente mediante la línea de comandos de OpenShift
> * Desencadenamiento de una compilación binaria mediante la línea de comandos de OpenShift

Obtenga más información sobre cómo compilar e implementar aplicaciones mediante las estrategias de origen a imagen y [otras estrategias de compilación](https://docs.openshift.com/container-platform/4.6/builds/understanding-image-builds.html).
