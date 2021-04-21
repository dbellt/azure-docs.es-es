---
title: Implementación de una aplicación Java con Open Liberty/WebSphere Liberty en un clúster de Red Hat OpenShift en Azure 4
description: Implemente una aplicación Java con Open Liberty/WebSphere Liberty en un clúster de Red Hat OpenShift en Azure 4.
author: jiangma
ms.author: jiangma
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 10/30/2020
keywords: java, jakartaee, javaee, microprofile, open-liberty, websphere-liberty, aro, openshift, red hat
ms.openlocfilehash: 2a308c7de754f395a3ef8a1bd97ed2441d27d21d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783582"
---
# <a name="deploy-a-java-application-with-open-libertywebsphere-liberty-on-an-azure-red-hat-openshift-4-cluster"></a>Implementación de una aplicación Java con Open Liberty/WebSphere Liberty en un clúster de Red Hat OpenShift en Azure 4

En esta guía se muestra cómo ejecutar una aplicación Java, Java EE, [Jakarta EE](https://jakarta.ee/) o [MicroProfile](https://microprofile.io/) en el runtime de Open Liberty/WebSphere Liberty y, luego, implementar la aplicación en contenedores en un clúster de Red Hat OpenShift en Azure (ARO) 4 con el operador de Open Liberty. Este artículo lo guiará en la preparación de una aplicación Liberty, en la creación de la imagen de Docker de la aplicación y en la ejecución de la aplicación en contenedores en un clúster de ARO 4.  Para más detalles sobre Open Liberty, consulte [la página del proyecto Open Liberty](https://openliberty.io/). Para más información sobre IBM WebSphere Liberty, consulte [la página del producto WebSphere Liberty](https://www.ibm.com/cloud/websphere-liberty).

[!INCLUDE [aro-support](includes/aro-support.md)]

## <a name="prerequisites"></a>Requisitos previos

Complete estos requisitos previos para recorrer correctamente esta guía.

> [!NOTE]
> Red Hat OpenShift en Azure requiere 40 núcleos como mínimo para crear y ejecutar un clúster de OpenShift. La cuota predeterminada de recursos de Azure para una suscripción nueva de Azure no cumple este requisito. Para solicitar un aumento del límite de recursos, consulte [Cuota estándar: Aumento de los límites por serie de máquinas virtuales](../azure-portal/supportability/per-vm-quota-requests.md). Tenga en cuenta que la suscripción de evaluación gratuita no es válida para un aumento de cuota, por lo que debe [actualizar a una suscripción de pago por uso](../cost-management-billing/manage/upgrade-azure-subscription.md) antes de solicitar ese aumento.

1. Prepare una máquina local con un sistema operativo similar a Unix instalado (por ejemplo, Ubuntu o macOS).
1. Instale una implementación de Java SE (por ejemplo, [AdoptOpenJDK OpenJDK 8 LTS/OpenJ9](https://adoptopenjdk.net/?variant=openjdk8&jvmVariant=openj9)).
1. Instale [Maven](https://maven.apache.org/download.cgi) 3.5.0 o una versión superior.
1. Instale [Docker](https://docs.docker.com/get-docker/) para el sistema operativo.
1. Instale la [CLI de Azure](/cli/azure/install-azure-cli) 2.0.75 o una versión posterior.
1. Compruebe e instale [`envsubst`](https://command-not-found.com/envsubst) si no está preinstalado en el sistema operativo.
1. Clone el código para este ejemplo en el sistema local. El ejemplo se encuentra en [GitHub](https://github.com/Azure-Samples/open-liberty-on-aro).
1. Siga las instrucciones que aparecen en [Creación de un clúster de Red Hat OpenShift en Azure 4](./tutorial-create-cluster.md).

   Si bien el paso "Obtención de un secreto de extracción de Red Hat" está etiquetado como opcional, **es necesario para este artículo**.  El secreto de extracción permite que el clúster de Red Hat OpenShift en Azure busque el operador de Open Liberty.

   Si planea ejecutar en el clúster aplicaciones que utilizan mucha memoria, especifique el tamaño de máquina virtual correspondiente para los nodos de trabajo mediante el parámetro `--worker-vm-size`. Por ejemplo, `Standard_E4s_v3` es el tamaño de máquina virtual mínimo para instalar el operador de Elasticsearch en un clúster. Para más información, consulte:

   * [CLI de Azure para crear un clúster](/cli/azure/aro#az_aro_create)
   * [Tamaños de máquina virtual admitidos para la memoria optimizada](./support-policies-v4.md#memory-optimized)
   * [Requisitos previos para instalar el operador de Elasticsearch](https://docs.openshift.com/container-platform/4.3/logging/cluster-logging-deploying.html#cluster-logging-deploy-eo-cli_cluster-logging-deploying)

1. Para conectarse al clúster, siga los pasos que aparecen en [Conexión a un clúster de Red Hat OpenShift en Azure 4](./tutorial-connect-cluster.md).
   * Asegúrese de seguir los pasos que aparecen en "Instalación de la CLI de OpenShift", porque usaremos el comando `oc` más adelante en este artículo.
   * Anota la dirección URL de la consola del clúster, que tiene este aspecto: `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.
   * Anote las credenciales de `kubeadmin`.

1. Compruebe que puede iniciar sesión en la CLI de OpenShift con el token para el usuario `kubeadmin`.

### <a name="enable-the-built-in-container-registry-for-openshift"></a>Habilitación del registro de contenedor integrado para OpenShift

En los pasos de este tutorial se crean una imagen de Docker que se debe insertar en un registro de contenedor accesible para OpenShift. La opción más sencilla es usar el registro integrado proporcionado por OpenShift. Para habilitar el registro de contenedor integrado, siga los pasos que aparecen en [Configuración del registro de contenedor integrado para Red Hat OpenShift en Azure 4](built-in-container-registry.md). En este artículo se usan tres elementos de estos pasos.

* El nombre de usuario y la contraseña del usuario de Azure AD para iniciar sesión en la consola web de OpenShift.
* La salida de `oc whoami` después de seguir los pasos para iniciar sesión en la CLI de OpenShift.  Para el análisis, este valor se llama **aad-user**.
* La dirección URL del registro de contenedor.

Anote estos elementos a medida que completa los pasos para habilitar el registro de contenedor integrado.

### <a name="create-an-openshift-namespace-for-the-java-app"></a>Creación de un espacio de nombres de OpenShift para la aplicación Java

1. Inicie sesión en la consola web de OpenShift desde el explorador con las credenciales de `kubeadmin`.
2. Vaya a **Administración** > **Espacios de nombres** > **Crear espacio de nombres**.
3. Escriba `open-liberty-demo` en **Nombre** y seleccione **Crear**, como se muestra a continuación.

   ![crear espacio de nombres](./media/howto-deploy-java-liberty-app/create-namespace.png)

### <a name="create-an-administrator-for-the-demo-project"></a>Creación de un administrador para el proyecto de demostración

Además de la administración de imágenes, **aad-user** también tendrá permisos administrativos para administrar los recursos del proyecto de demostración del clúster de ARO-4.  Inicie sesión en la CLI de OpenShift y siga estos pasos para conceder a **aad-user** los privilegios necesarios.

1. Inicie sesión en la consola web de OpenShift desde el explorador con las credenciales de `kubeadmin`.
1. En la parte superior derecha de la consola web, expanda el menú contextual del usuario que inició sesión y, luego, seleccione **Copy Login Command** (Copiar comando de inicio de sesión).
1. Si es necesario, inicie sesión en una pestaña nueva con el mismo usuario.
1. Seleccione **Mostrar token**.
1. Copie el valor que aparece debajo de **Iniciar sesión con este token** en el portapapeles y ejecútelo en un shell, como se muestra aquí.
1. Ejecute los comandos siguientes para conceder el rol `admin` a **aad-user** en el espacio de nombres `open-liberty-demo`.

   ```bash
   # Switch to project "open-liberty-demo"
   oc project open-liberty-demo
   Now using project "open-liberty-demo" on server "https://api.x8xl3f4y.eastus.aroapp.io:6443".
   # Note: replace "<aad-user>" with the one noted by executing the steps in
   # Configure built-in container registry for Azure Red Hat OpenShift 4
   oc adm policy add-role-to-user admin <aad-user>
   clusterrole.rbac.authorization.k8s.io/admin added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

### <a name="install-the-open-liberty-openshift-operator"></a>Instalación del operador OpenShift de Open Liberty

Después de crear el clúster y conectarse a él, instale el operador de Open Liberty.  La página de inicio principal del operador de Open Liberty está en [GitHub](https://github.com/OpenLiberty/open-liberty-operator).

1. Inicie sesión en la consola web de OpenShift desde el explorador con las credenciales de `kubeadmin`.
2. Vaya a **Operadores** > **OperatorHub** y busque el **Operador de Open Liberty**.
3. Seleccione **Operador de Open Liberty** en los resultados de la búsqueda.
4. Seleccione **Instalar**.
5. En la ventana emergente **Crear suscripción del operador**, active **Todos los espacios de nombres en el clúster (valor predeterminado)** en el **Modo de instalación**, **beta** en **Canal de actualización** y **Automático** en **Estrategia de aprobación**:

   ![crear suscripción del operador para el operador de Open Liberty](./media/howto-deploy-java-liberty-app/install-operator.png)
6. Seleccione **Suscribirse** y espere unos minutos hasta que aparezca el operador de Open Liberty.
7. Observe el operador de Open Liberty con el estado "Correcto".  Si no es así, debe diagnosticar y resolver el problema antes de seguir.
   :::image type="content" source="media/howto-deploy-java-liberty-app/open-liberty-operator-installed.png" alt-text="Operadores instalados que muestran que se instaló Open Liberty.":::

## <a name="prepare-the-liberty-application"></a>Preparación de la aplicación Liberty

En esta guía se usará como ejemplo una aplicación Java EE 8. Open Liberty es un servidor compatible con el [perfil completo de Java EE 8](https://javaee.github.io/javaee-spec/javadocs/), por lo que puede ejecutar fácilmente la aplicación.  Open Liberty también es [compatible con el perfil completo de Jakarta EE 8](https://jakarta.ee/specifications/platform/8/apidocs/).

### <a name="run-the-application-on-open-liberty"></a>Ejecución de la aplicación en Open Liberty

Para ejecutar la aplicación en Open Liberty, debe crear un archivo de configuración de servidor Open Liberty para que el [complemento de Maven para Liberty](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin) pueda empaquetar la aplicación para su implementación. El complemento de Maven para Liberty no es necesario para implementar la aplicación en OpenShift.  Sin embargo, en este ejemplo lo usaremos con el modo de desarrollador (dev) de Open Liberty.  El modo de desarrollador le permite ejecutar la aplicación fácilmente de manera local. Complete estos pasos en el equipo local.

1. Copie `2-simple/src/main/liberty/config/server.xml` en `1-start/src/main/liberty/config` y sobrescriba el archivo existente de longitud cero. Este archivo `server.xml` configura el servidor Open Liberty con características de Java EE.
1. Copie `2-simple/pom.xml` en `1-start/pom.xml`.  Este paso agrega el complemento `liberty-maven-plugin` al archivo POM.
1. Cambie el directorio a `1-start` del clon local.
1. Ejecute `mvn clean package` en una consola para generar un paquete WAR `javaee-cafe.war` en el directorio `./target`.
1. Ejecute `mvn liberty:dev` para iniciar Open Liberty en modo de desarrollador.
1. Espere hasta que se inicie el servidor. La salida de la consola debe finalizar con el mensaje siguiente:

   ```Text
   [INFO] CWWKM2015I: Match number: 1 is [6/10/20 10:26:09:517 CST] 00000022 com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 6.447 seconds..
   [INFO] Press the Enter key to run tests on demand. To stop the server and quit dev mode, use Ctrl-C or type 'q' and press the Enter key.
   [INFO] Source compilation was successful.
   ```

1. Abra `http://localhost:9080/` en el explorador para ir a la página principal de la aplicación. La aplicación tendrá un aspecto similar a la imagen siguiente:

   ![Interfaz de usuario web de JavaEE Cafe](./media/howto-deploy-java-liberty-app/javaee-cafe-web-ui.png)
1. Presione **Control-C** para detener la aplicación y el servidor Open Liberty.

El directorio `2-simple` del clon local muestra el proyecto de Maven con los cambios anteriores ya aplicados.

## <a name="prepare-the-application-image"></a>Preparación de la imagen de la aplicación

Para implementar y ejecutar la aplicación Liberty en un clúster de ARO 4, incluya en un contenedor la aplicación como una imagen de Docker mediante las [imágenes de contenedor de Open Liberty](https://github.com/OpenLiberty/ci.docker) o las [imágenes de contenedor de WebSphere Liberty](https://github.com/WASdev/ci.docker).

### <a name="build-application-image"></a>Creación de la imagen de la aplicación

Complete estos pasos para crear la imagen de la aplicación:

1. Cambie el directorio a `2-simple` del clon local.
2. Ejecute `mvn clean package` para empaquetar la aplicación.
3. Ejecute uno de estos comandos para crear la imagen de la aplicación.
   * Creación con una imagen base de Open Liberty:

     ```bash
     # Build and tag application image. This will cause Docker to pull the necessary Open Liberty base images.
     docker build -t javaee-cafe-simple:1.0.0 --pull .
     ```

   * Creación con una imagen base de WebSphere Liberty:

     ```bash
     # Build and tag application image. This will cause Docker to pull the necessary WebSphere Liberty base images.
     docker build -t javaee-cafe-simple:1.0.0 --pull --file=Dockerfile-wlp .
     ```

### <a name="run-the-application-locally-with-docker"></a>Ejecución local de la aplicación con Docker

Antes de implementar la aplicación en contenedor en un clúster remoto, ejecútela con la instancia de Docker local para comprobar si funciona:

1. Ejecute `docker run -it --rm -p 9080:9080 javaee-cafe-simple:1.0.0` en la consola.
2. Espere a que el servidor Liberty se inicie y la aplicación se implemente correctamente.
3. Abra `http://localhost:9080/` en el explorador para ir a la página principal de la aplicación.
4. Presione **Control-C** para detener la aplicación y el servidor Liberty.

### <a name="push-the-image-to-the-container-image-registry"></a>Inserción de la imagen en el registro de imágenes de contenedor

Cuando esté satisfecho con el estado de la aplicación, siga las instrucciones que se indican a continuación para insertarla en el registro de imágenes de contenedor integrado.

#### <a name="log-in-to-the-openshift-cli-as-the-azure-ad-user"></a>Inicio de sesión en la CLI de OpenShift como usuario de Azure AD

1. Inicie sesión en la consola web de OpenShift desde el explorador con las credenciales de un usuario de Azure AD.

   1. Use InPrivate, Incógnito u otra característica de ventana del explorador equivalente para iniciar sesión en la consola.
   1. Seleccione **openid**.

   > [!NOTE]
   > Anote el nombre de usuario y la contraseña que usa para iniciar sesión aquí. Este nombre de usuario y contraseña funcionarán como administrador para otras acciones en este y en otros artículos.
1. Siga estos pasos para iniciar sesión con la CLI de OpenShift.  Para el análisis, este proceso se conoce como `oc login`.
   1. En la parte superior derecha de la consola web, expanda el menú contextual del usuario que inició sesión y, luego, seleccione **Copy Login Command** (Copiar comando de inicio de sesión).
   1. Si es necesario, inicie sesión en una pestaña nueva con el mismo usuario.
   1. Seleccione **Mostrar token**.
   1. Copie el valor que aparece debajo de **Iniciar sesión con este token** en el portapapeles y ejecútelo en un shell, como se muestra aquí.

       ```bash
       oc login --token=XOdASlzeT7BHT0JZW6Fd4dl5EwHpeBlN27TAdWHseob --server=https://api.aqlm62xm.rnfghf.aroapp.io:6443
       Logged into "https://api.aqlm62xm.rnfghf.aroapp.io:6443" as "kube:admin" using the token provided.

       You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'

       Using project "default".
       ```

#### <a name="push-the-container-image-to-the-container-registry-for-openshift"></a>Inserción de la imagen de contenedor en el registro de contenedor de OpenShift

Ejecute estos comandos para insertar la imagen en el registro de contenedor de OpenShift.

```bash
# Note: replace "<Container_Registry_URL>" with the fully qualified name of the registry
Container_Registry_URL=<Container_Registry_URL>

# Create a new tag with registry info that refers to source image
docker tag javaee-cafe-simple:1.0.0 ${Container_Registry_URL}/open-liberty-demo/javaee-cafe-simple:1.0.0

# Sign in to the built-in container image registry
docker login -u $(oc whoami) -p $(oc whoami -t) ${Container_Registry_URL}
```

La salida correcta será similar a la siguiente.

```bash
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```

Use el comando siguiente para insertar la imagen en el registro de imágenes de contenedor integrado.

```bash

docker push ${Container_Registry_URL}/open-liberty-demo/javaee-cafe-simple:1.0.0
```

## <a name="deploy-application-on-the-aro-4-cluster"></a>Implementación de una aplicación en el clúster de ARO 4

Ahora que puede implementar la aplicación Liberty de ejemplo en el clúster de Red Hat OpenShift en Azure 4 que creó antes en la sección de requisitos previos.

### <a name="deploy-the-application-from-the-web-console"></a>Implementación de la aplicación desde la consola web

Como usamos el operador de Open Liberty para administrar aplicaciones Liberty, es necesario crear una instancia de su *definición de recurso personalizado* de tipo "OpenLibertyApplication". Luego, el operador se ocupará de todos los aspectos que implica la administración de los recursos de OpenShift que se necesitan para la implementación.

1. Inicie sesión en la consola web de OpenShift desde el explorador con las credenciales del usuario de Azure AD.
1. Expanda **Inicio**, seleccione **Proyectos** > **open-liberty-demo**.
1. Vaya a **Operadores** > **Operadores instalados**.
1. En el medio de la página, seleccione **Operador de Open Liberty**.
1. En el medio de la página, seleccione **Aplicación Open Liberty**.  La navegación de los elementos de la interfaz de usuario refleja la jerarquía de contención real de las tecnologías en uso.
   <!-- Diagram source https://github.com/Azure-Samples/open-liberty-on-aro/blob/master/diagrams/aro-java-containment.vsdx -->
   ![Contención de Java ARO](./media/howto-deploy-java-liberty-app/aro-java-containment.png)
1. Seleccione **Crear OpenLibertyApplication**.
1. Reemplace el archivo YAML generado por el suyo, que se encuentra en `<path-to-repo>/2-simple/openlibertyapplication.yaml`.
1. Seleccione **Crear**. Será redirigido a la lista de OpenLibertyApplications.
1. Seleccione **javaee-cafe-simple**.
1. En el medio de la página, seleccione **Recursos**.
1. En la tabla, seleccione el vínculo para **javaee-cafe-simple** con la **Variante** de **Ruta**.
1. En la página que se abre, seleccione el vínculo debajo de **Ubicación**.

Verá que la página principal de la aplicación se abre en el explorador.

### <a name="delete-the-application-from-the-web-console"></a>Eliminación de la aplicación desde la consola web

Cuando termine de usar la aplicación, siga estos pasos para borrarla de OpenShift.

1. En el panel de navegación de la izquierda, expanda la entrada de **Operadores**.
1. Seleccione **Operadores instalados**.
1. Seleccione **Operador de Open Liberty**.
1. En el medio de la página, seleccione **Aplicación Open Liberty**.
1. Seleccione los puntos suspensivos verticales (tres puntos verticales) y, luego, seleccione **Eliminar la aplicación OpenLiberty**.

### <a name="deploy-the-application-from-cli"></a>Implementación de la aplicación desde la CLI

En lugar de usar la GUI de la consola web, puede implementar la aplicación desde la CLI. Si todavía no lo ha hecho, descargue e instale la herramienta de línea de comandos `oc` según lo indicado en la documentación de Red Hat [Introducción a la CLI](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html).

1. Inicie sesión en la consola web de OpenShift desde el explorador con las credenciales del usuario de Azure AD.
2. Inicie sesión en la CLI de OpenShift con el token de usuario de Azure AD.
3. Cambie el directorio a `2-simple` del clon local y ejecute los comandos siguientes para implementar la aplicación Liberty en el clúster de ARO 4.  La salida del comando también se muestra insertada.

   ```bash
   # Switch to namespace "open-liberty-demo" where resources of demo app will belong to
   oc project open-liberty-demo

   Now using (or already on) project "open-liberty-demo" on server "https://api.aqlm62xm.rnfghf.aroapp.io:6443".

   # Create OpenLibertyApplication "javaee-cafe-simple"
   oc create -f openlibertyapplication.yaml

   openlibertyapplication.openliberty.io/javaee-cafe-simple created

   # Check if OpenLibertyApplication instance is created
   oc get openlibertyapplication javaee-cafe-simple

   NAME                 IMAGE                      EXPOSED   RECONCILED   AGE
   javaee-cafe-simple   javaee-cafe-simple:1.0.0   true      True         36s

   # Check if deployment created by Operator is ready
   oc get deployment javaee-cafe-simple

   NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
   javaee-cafe-simple   1/1     1            0           102s
   ```

4. Antes de seguir, compruebe que ve `1/1` debajo de la columna `READY`.  Si no es así, investigue y resuelva el problema antes de continuar.
5. Descubra el host de la ruta a la aplicación con el comando `oc get route`, tal como se muestra aquí.

   ```bash
   # Get host of the route
   HOST=$(oc get route javaee-cafe-simple --template='{{ .spec.host }}')
   echo "Route Host: $HOST"

   Route Host: javaee-cafe-simple-open-liberty-demo.apps.aqlm62xm.rnfghf.aroapp.io
   ```

   Una vez que la aplicación Liberty está en ejecución, abra la salida de **Route Host** en el explorador para visitar la página principal de la aplicación.

### <a name="delete-the-application-from-cli"></a>Eliminación de la aplicación desde la CLI

Para eliminar la aplicación desde la CLI, ejecute este comando.

```bash
oc delete -f openlibertyapplication.yaml
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el clúster de ARO, siga los pasos que aparecen en [Tutorial: Eliminación de un clúster de Red Hat OpenShift en Azure 4](./tutorial-delete-cluster.md)

## <a name="next-steps"></a>Pasos siguientes

En esta guía aprendió información sobre lo siguiente:
> [!div class="checklist"]
>
> * Preparación de la aplicación Liberty
> * Creación de la imagen de la aplicación
> * Ejecución de la aplicación en contenedor en un clúster de ARO 4 mediante la GUI y la CLI

Puede obtener más información en las referencias que se usan en esta guía:

* [Open Liberty](https://openliberty.io/)
* [Red Hat OpenShift en Azure](https://azure.microsoft.com/services/openshift/)
* [Operador de Open Liberty](https://github.com/OpenLiberty/open-liberty-operator)
* [Configuración del servidor Open Liberty](https://openliberty.io/docs/ref/config/)
* [Complemento de Maven para Liberty](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin)
* [Imágenes de contenedor de Open Liberty](https://github.com/OpenLiberty/ci.docker)
* [Imágenes de contenedor de WebSphere Liberty](https://github.com/WASdev/ci.docker)