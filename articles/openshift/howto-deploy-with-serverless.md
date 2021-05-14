---
title: Implementación de una aplicación en Red Hat OpenShift en Azure con OpenShift Serverless
description: Obtenga información sobre cómo implementar una aplicación en Red Hat OpenShift en Azure con OpenShift sin servidor.
author: sabbour
ms.author: asabbour
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 4/5/2021
keywords: aro, openshift, red hat, serverless
ms.openlocfilehash: 6db87e752745e3df919c93b2ffc8144e5886b319
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107532810"
---
# <a name="deploy-an-application-to-azure-red-hat-openshift-using-openshift-serverless"></a>Implementación de una aplicación en Red Hat OpenShift en Azure con OpenShift Serverless

En este artículo, va a implementar una aplicación en un clúster de Red Hat OpenShift en Azure con [OpenShift Serverless](https://www.openshift.com/learn/topics/serverless). OpenShift Serverless ayuda a los desarrolladores a implementar y ejecutar aplicaciones que se escalarán verticalmente o se reducirán a cero a petición, lo que elimina el consumo de recursos cuando no están en uso.

El código de la aplicación puede empaquetarse en un contenedor junto con los tiempos de ejecución adecuados, y la funcionalidad sin servidor iniciará los contenedores de la aplicación cuando se desencadenen por un evento. Las aplicaciones se pueden desencadenar por varios orígenes de eventos, como eventos de las propias aplicaciones, servicios en la nube de múltiples proveedores, sistemas de software como servicio (SaaS) y otros servicios.

La administración de todos los aspectos de la implementación de cualquier contenedor sin servidor está integrada directamente en la interfaz de OpenShift. Los desarrolladores pueden identificar visualmente qué eventos impulsan el inicio de sus aplicaciones en contenedores, con varias maneras de modificar los parámetros de los eventos. Las aplicaciones de OpenShift Serverless pueden integrarse con otros servicios de OpenShift, como OpenShift Pipelines, Service Mesh y Monitoring, lo que ofrece una experiencia completa de desarrollo e implementación de aplicaciones sin servidor.

## <a name="before-you-begin"></a>Antes de comenzar

[!INCLUDE [aro-howto-beforeyoubegin](includes/aro-howto-before-you-begin.md)]

### <a name="install-the-knative-command-line-tool-kn"></a>Instalación de la herramienta de línea de comandos de Knative (kn)

También puede descargar la versión más reciente de la CLI adecuada para su máquina desde <https://github.com/knative/client/releases/>.

Si va a ejecutar los comandos en Azure Cloud Shell, descargue la CLI más reciente de Knative para Linux.

```azurecli-interactive
cd ~
wget https://github.com/knative/client/releases/download/v0.22.0/kn-linux-amd64

mkdir knative
chmod +x kn-linux-amd64
mv kn-linux-amd64 knative/kn
echo 'export PATH=$PATH:~/knative' >> ~/.bashrc && source ~/.bashrc
```

### <a name="launch-the-web-console"></a>Inicio de la consola web

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

## <a name="install-the-openshift-serverless-operator"></a>Instalación del operador de OpenShift Serverless

Cuando haya iniciado sesión en la consola web de OpenShift, asegúrese de que está en la perspectiva *Administrator* (Administrador). Abra *Operator Hub* (Centro de operadores), busque el operador **OpenShift Serverless** y selecciónelo.

![Búsqueda del operador OpenShift Serverless](media/serverless/serverless-operatorhub.png)

A continuación, abra la página de instalación del operador haciendo clic en **Install** (Instalar).

![Haga clic en Install (Instalar) para instalar el operador.](media/serverless/serverless-clickinstall.png)

Elija el *canal de actualización* adecuado para la versión del clúster de Red Hat OpenShift en Azure e instale el operador en el espacio de nombres `openshift-serverless`. Desplácese hacia abajo y haga clic en **Install** (Instalar).

![Página de instalación del operador](media/serverless/serverless-installpage.png)

En unos minutos, la página de estado debe reflejar que el operador está instalado y está listo para su uso. Haga clic en el botón **View Operator** (Ver operador) para continuar.

![El operador está instalado y está listo para su uso.](media/serverless/serverless-installed.png)

## <a name="install-knative-serving"></a>Instalación de Knative Serving

La funcionalidad de ejecutar cualquier contenedor de forma sin servidor en OpenShift Serverless es posible mediante el uso de Knative ascendente. Knative amplía Kubernetes para proporcionar un conjunto de componentes para implementar, ejecutar y administrar aplicaciones modernas mediante la metodología sin servidor.

### <a name="create-an-instance-of-the-knative-serving"></a>Creación de una instancia de Knative Serving

Cambie al espacio de nombres `knative-serving` haciendo clic en la lista desplegable de proyectos en la parte superior izquierda y, a continuación, en *Provided APIs* (API proporcionadas), haga clic en **Create Instance** (Crear instancia) en la tarjeta *Knative Serving*.

![Haga clic para crear una instancia del servicio Knative.](media/serverless/serverless-createknativeserving.png)

Mantenga los valores predeterminados y desplácese hacia abajo en la página *Create Knative Serving* (Crear Knative Serving) para hacer clic en el botón **Create** (Crear).

![Mantenimiento de los valores predeterminados y clic en Create (Crear)](media/serverless/serverless-createknativeserving2.png)

Espere hasta que la columna *Status* (Estado) muestre **Ready** (Listo); en ese momento, OpenShift Serverless debe estar instalado y está listo para crear un proyecto de OpenShift Serverless.

![Listo para Knative Serving](media/serverless/serverless-createknativeserving3.png)

## <a name="create-a-serverless-project"></a>Creación de un proyecto sin servidor

Para crear un nuevo proyecto denominado `demoserverless`, ejecute el comando:

```azurecli-interactive
oc new-project demoserverless
```

Debería aparecer una salida similar a la siguiente:

```output
Now using project "demoserverless" on server "https://api.wzy5hg7x.eastus.aroapp.io:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app django-psql-example

to build a new example application in Python. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
```

Cambie a la perspectiva *Developer* (Desarrollador) en lugar de la perspectiva *Administrator* (Administrador) en el menú de la izquierda y seleccione `demoserverless` en la lista de proyectos. Debería estar entonces en la página *Topología* del proyecto.

![Topología del proyecto de Red Hat OpenShift en Azure](media/serverless/serverless-topology.png)

## <a name="deploying-using-the-web-console"></a>Implementación mediante la consola web

En las opciones presentadas para implementar una aplicación, seleccione *From Git* (Desde Git). Esto le llevará a la página *Import from Git* (Importar desde Git). Utilice `https://github.com/sclorg/django-ex.git` como **dirección URL del repositorio de Git**. La aplicación web de ejemplo se implementa con el lenguaje de programación Python.

![Proyecto de Red Hat OpenShift en Azure desde Git](media/serverless/serverless-from-git.png)

> [!NOTE]
> OpenShift detecta que se trata de un proyecto de Python y selecciona la imagen del generador adecuada.

Desplácese hacia abajo hasta *Resources* (Recursos) y asegúrese de que **Knative Service** (Servicio Knative) está seleccionado como el tipo de recurso que se va a generar. Esta acción creará un servicio Knative, un tipo de implementación que permite el escalado de OpenShift Serverless a cero cuando está inactivo.

![Proyecto de Red Hat OpenShift en Azure: Knative](media/serverless/serverless-knative.png)


Cuando esté listo, en la parte inferior de la página, haga clic en **Create** (Crear). Esto creará recursos para administrar la compilación e implementación de la aplicación. A continuación, se le redirigirá a la información general de la topología del proyecto.

La información general de la topología proporciona una representación visual de la aplicación que ha implementado. Con esta vista, puede ver la estructura general de la aplicación.

Espere a que la compilación finalice. Esto puede tardar unos minutos. Una vez completada la compilación, aparece una marca de verificación verde en la esquina inferior izquierda del servicio.

![Topología del proyecto de Red Hat OpenShift en Azure: compilación correcta](media/serverless/serverless-ready.png)

## <a name="see-your-application-scale"></a>Visualización de la escala de la aplicación

En la lista *Display Options* (Opciones de visualización) de la parte superior de la vista Topology (Topología), haga clic en *Pod Count* (Número de pods). Espere a que el número de pods se reduzca verticalmente a cero pods. La reducción vertical puede tardar algunos minutos.

![Topología del proyecto de Red Hat OpenShift en Azure: reducción a cero](media/serverless/serverless-scaledtozero.png)

Haga clic en el icono *Open URL* (Abrir dirección URL) en la esquina superior derecha del panel del servicio Knative. La aplicación se abre en una nueva pestaña. Cierre la nueva pestaña del explorador y vuelva a la vista Topology (Topología). En la vista Topology (Topología), puede ver que la aplicación se ha escalado verticalmente a un pod para dar cabida a su solicitud. Al cabo de unos minutos, la aplicación se vuelve a reducir verticalmente a cero pods.

![Topología del proyecto de Red Hat OpenShift en Azure: escalado a uno](media/serverless/serverless-scaledtoone.png)

## <a name="forcing-a-new-revision-and-setting-traffic-distribution"></a>Forzado de una nueva revisión y establecimiento de la distribución del tráfico

Con cada actualización de la configuración de un servicio, se crea una nueva revisión para dicho servicio. De forma predeterminada, la ruta de servicio apunta todo el tráfico a la revisión lista más reciente. Para cambiar este comportamiento, defina qué revisión obtiene una parte del tráfico. Los servicios Knative permiten la asignación de tráfico, lo que significa que las revisiones de un servicio se pueden asignar a una parte asignada del tráfico. La asignación de tráfico también proporciona una opción para crear direcciones URL únicas para revisiones concretas.

En *Topology* (Topología), haga clic en la revisión dentro del servicio para ver sus detalles. Las notificaciones debajo el anillo del pod y en la parte superior del panel de detalles deben ser `(REV)`. En el panel lateral, en la pestaña *Resources* (Recursos), desplácese hacia abajo y haga clic en la configuración asociada al servicio.

![Topología del proyecto de Red Hat OpenShift en Azure: revisión](media/serverless/serverless-revdetails.png)

Para forzar una actualización de configuración, cambie a la pestaña *YAML* y desplácese hacia abajo para editar el valor de `timeoutSeconds` para que sea `301`. Haga clic en el botón **Save** (Guardar) para guardar la configuración actualizada. En un escenario real, esta actualización de configuración también se puede desencadenar mediante la actualización de la etiqueta de imagen de contenedor.

![Forzado de una nueva revisión mediante la actualización de la configuración](media/serverless/serverless-confupdate.png)

Vuelva a la vista *Topology* (Topología) y verá que se ha implementado una nueva revisión. Haga clic en el servicio que termina con la notificación `(KSVC)` y haga clic en el botón **Set Traffic Distribution** (Establecer distribución de tráfico); ahora debería poder dividir el tráfico entre las distintas revisiones del servicio.

![Establecer distribución de tráfico](media/serverless/serverless-trafficdist.png)

La vista *Topology* (Topología) ahora mostrará cómo se distribuye el tráfico entre las dos revisiones.

![Revisión de la distribución del tráfico](media/serverless/serverless-trafficdist2.png)

## <a name="using-the-knative-command-line-tool-kn"></a>Uso de la herramienta de línea de comandos de Knative (kn)

En los pasos anteriores, ha usado la consola web de OpenShift para crear e implementar una aplicación en OpenShift Serverless. Dado que OpenShift Serverless ejecuta Knative por debajo, también puede utilizar la herramienta de línea de comandos Knative (kn) para crear servicios Knative.

> [!NOTE]
> Si aún no ha instalado la CLI de `kn`, asegúrese de seguir los pasos descritos en la sección requisitos previos de este artículo. Asegúrese también de que ha iniciado sesión con la herramienta de línea de comandos de OpenShift `oc`.

Vamos a usar una imagen de contenedor que ya se ha creado en `quay.io/rhdevelopers/knative-tutorial-greeter`.

### <a name="deploy-a-service"></a>Implementación de un servicio

Para implementar el servicio, ejecute el siguiente comando:

```azurecli-interactive
kn service create greeter \
--image quay.io/rhdevelopers/knative-tutorial-greeter:quarkus \
--namespace demoserverless \
--revision-name greeter-v1
```

Verá una salida similar a la siguiente.

```output
Creating service 'greeter' in namespace 'demoserverless':

  0.044s The Route is still working to reflect the latest desired specification.
  0.083s ...
  0.114s Configuration "greeter" is waiting for a Revision to become ready.
 10.420s ...
 10.489s Ingress has not yet been reconciled.
 10.582s Waiting for load balancer to be ready
 10.763s Ready to serve.

Service 'greeter' created to latest revision 'greeter-v1' is available at URL:
http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
```

Para recuperar una lista de rutas en el proyecto, ejecute:

```azurecli-interactive
kn route list
```

Volverá a obtener una lista de rutas en el espacio de nombres. Abra la dirección URL en un explorador para ver el servicio implementado.

```output
NAME      URL                                                            READY
greeter   http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io   True
```

### <a name="deploy-a-new-version-of-the-service"></a>Implementación de una nueva versión del servicio

Implemente una nueva versión de la aplicación mediante la ejecución del comando siguiente, pasando la etiqueta de imagen `:latest` y una variable de entorno `MESSAGE_PREFIX`:

```azurecli-interactive
kn service update greeter \
 --image quay.io/rhdevelopers/knative-tutorial-greeter:latest \
 --namespace demoserverless \
 --env MESSAGE_PREFIX=GreeterV2 \
 --revision-name greeter-v2
```

Recibirá una confirmación de que se ha implementado una nueva revisión, `greeter-v2`.

```output
Updating Service 'greeter' in namespace 'demoserverless':

  5.029s Traffic is not yet migrated to the latest revision.
  5.086s Ingress has not yet been reconciled.
  5.190s Waiting for load balancer to be ready
  5.332s Ready to serve.

Service 'greeter' updated to latest revision 'greeter-v2' is available at URL:
http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
```

Para ver una lista de todas las revisiones y su distribución de tráfico, ejecute el siguiente comando:

```azurecli-interactive
kn revision list
```

Verá una lista similar a la siguiente. Tenga en cuenta que la nueva revisión recibe el 100 % del tráfico.

```output
NAME            SERVICE   TRAFFIC   TAGS   GENERATION   AGE     CONDITIONS   READY   REASON
greeter-v2      greeter   100%             2            90s     3 OK / 4     True
greeter-v1      greeter                    1            5m32s   3 OK / 4     True
```

### <a name="bluegreen-and-canary-deployments"></a>Implementaciones azul-verde y controladas

De forma predeterminada, cuando se implementa una nueva revisión, recibe el 100 % del tráfico. Supongamos que quiere implementar una estrategia de implementación azul-verde donde pueda revertir rápidamente a la versión anterior de la aplicación. Knative lo pone fácil.

Actualice el servicio para crear tres etiquetas de tráfico al enviar el 100 % del tráfico a

- **current** (actual): apunta a la versión implementada actual
- **prev** (anterior): apunta a la versión anterior
- **latest** (más reciente): siempre apunta a la versión más reciente

```azurecli-interactive
kn service update greeter \
   --tag greeter-v2=current \
   --tag greeter-v1=prev \
   --tag @latest=latest
```

Recibirá una confirmación similar a la siguiente.

```output
Updating Service 'greeter' in namespace 'demoserverless':

  0.037s Ingress has not yet been reconciled.
  0.121s Waiting for load balancer to be ready
  0.287s Ready to serve.

Service 'greeter' with latest revision 'greeter-v2' (unchanged) is available at URL:
http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
```

Enumere las rutas con el comando siguiente:

```azurecli-interactive
kn route describe greeter
```

Verá una salida que muestra las direcciones URL de cada una de las etiquetas junto con la distribución del tráfico.

```output
Name:       greeter
Namespace:  demoserverless
Age:        10m
URL:        http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
Service:    greeter

Traffic Targets:
  100%  @latest (greeter-v2) #latest
        URL:  http://latest-greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
    0%  greeter-v1 #prev
        URL:  http://prev-greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
    0%  greeter-v2 #current
        URL:  http://current-greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io

[..]
```

Supongamos que desea revertir rápidamente a la versión anterior; puede actualizar la distribución del tráfico para enviar el 100 % del tráfico a la etiqueta anterior:

```azurecli-interactive
kn service update greeter --traffic current=0 --traffic prev=100
```

Vuelva a comprobarlo enumerando las rutas con el comando siguiente:

```azurecli-interactive
kn route describe greeter
```

Verá una salida que muestra que el 100 % de la distribución del tráfico va a la versión anterior.

```output
Name:       greeter
Namespace:  demoserverless
Age:        19m
URL:        http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
Service:    greeter

Traffic Targets:
    0%  @latest (greeter-v2) #latest
        URL:  http://latest-greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
  100%  greeter-v1 #prev
        URL:  http://prev-greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io
    0%  greeter-v2 #current
        URL:  http://current-greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io

[..]
```

Recorra la distribución del tráfico mientras se actualiza la ruta principal (en `http://greeter-demoserverless.apps.wzy5hg7x.eastus.aroapp.io` este caso) en el explorador.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con la aplicación, puede ejecutar el siguiente comando para eliminar el proyecto:

```azurecli-interactive
oc delete project demoserverless
```

También puede eliminar el clúster; para ello, siga los pasos que aparecen en [Tutorial: Eliminación de un clúster de la versión 4 de Red Hat OpenShift en Azure](./tutorial-delete-cluster.md).

## <a name="next-steps"></a>Pasos siguientes

En esta guía aprendió información sobre lo siguiente:
> [!div class="checklist"]
>
> * Instalación del operador OpenShift Serverless y de Knative Serving
> * Implementación de un proyecto sin servidor mediante la consola web
> * Implementación de un proyecto sin servidor mediante la CLI de Knative (kn)
> * Configuración de implementaciones azul-verde e implementaciones de valores controlados mediante la CLI de Knative (kn)

Obtenga más información sobre cómo compilar e implementar aplicaciones sin servidor y controladas por eventos en Red Hat OpenShift en Azure mediante [OpenShift Serverless](https://www.openshift.com/learn/topics/serverless) y siga los documentos [Introducción a OpenShift Serverless](https://docs.openshift.com/container-platform/4.6/serverless/serverless-getting-started.html) y [creación y administración de aplicaciones sin servidor](https://docs.openshift.com/container-platform/4.6/serverless/serving-creating-managing-apps.html).
