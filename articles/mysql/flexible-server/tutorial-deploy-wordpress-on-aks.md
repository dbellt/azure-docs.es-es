---
title: 'Tutorial: Implementación de WordPress en un clúster de AKS con Servidor flexible de MySQL mediante la CLI de Azure'
description: Aprenda a compilar e implementar rápidamente WordPress en AKS con Servidor flexible de Azure Database for MySQL.
ms.service: mysql
author: mksuni
ms.author: sumuth
ms.topic: tutorial
ms.date: 11/25/2020
ms.custom: vc, devx-track-azurecli
ms.openlocfilehash: 0c6211f4cd647addd6f1d18a153695d16a9d9952
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770190"
---
# <a name="tutorial-deploy-wordpress-app-on-aks-with-azure-database-for-mysql---flexible-server"></a>Tutorial: Implementación de una aplicación WordPress en AKS con Servidor flexible de Azure Database for MySQL

En este inicio rápido se implementa una aplicación WordPress en el clúster de Azure Kubernetes Service (AKS) con Servidor flexible de Azure Database for MySQL (versión preliminar) mediante la CLI de Azure. 
**[AKS](../../aks/intro-kubernetes.md)** es un servicio de Kubernetes administrado que permite implementar y administrar clústeres rápidamente. **[Servidor flexible (versión preliminar) de Azure Database for MySQL](overview.md)** es un servicio de base de datos totalmente administrado diseñado para proporcionar un control más pormenorizado y una mayor flexibilidad sobre las funciones de administración de bases de datos y las opciones de configuración. Actualmente, Servidor flexible se encuentra en versión preliminar.

> [!NOTE]
> - Actualmente, Azure Database for MySQL con la opción Servidor flexible está en versión preliminar pública.
> - En este inicio rápido se presupone un conocimiento básico de los conceptos de Kubernetes, WordPress y MySQL.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión más reciente de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

> [!NOTE]
> Si los comandos de este inicio rápido se ejecutan localmente (en lugar de en Azure Cloud Shell), asegúrese de ejecutarlos como administrador.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un grupo lógico en el que se implementan y administran recursos de Azure. Vamos a crear un grupo de recursos, *wordpress-project*, mediante el comando [az group create][az-group-create] en la ubicación *eastus*.

```azurecli-interactive
az group create --name wordpress-project --location eastus
```

> [!NOTE]
> La ubicación del grupo de recursos es donde se almacenan los metadatos del grupo de recursos. Es también el lugar en el que se ejecutan los recursos en Azure si no se especifica otra región al crear los recursos.

En la siguiente salida de ejemplo se muestra que los recursos se crearon correctamente:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/wordpress-project",
  "location": "eastus",
  "managedBy": null,
  "name": "wordpress-project",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-aks-cluster"></a>Creación de un clúster de AKS

Use el comando [az aks create](/cli/azure/aks#az_aks_create) para crear un clúster de AKS. En el siguiente ejemplo se crea un clúster denominado *myAKSCluster* con un nodo. Este proceso tardará varios minutos en completarse.

```azurecli-interactive
az aks create --resource-group wordpress-project --name myAKSCluster --node-count 1 --generate-ssh-keys
```

Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON sobre el clúster.

> [!NOTE]
> Al crear un clúster de AKS, se crea automáticamente un segundo grupo de recursos para almacenar los recursos de AKS. Consulte [¿Por qué se crean dos grupos de recursos con AKS?](../../aks/faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, usará [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. Para instalar `kubectl` localmente, use el comando [az aks install-cli](/cli/azure/aks#az_aks_install_cli):

```azurecli-interactive
az aks install-cli
```

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials](/cli/azure/aks#az_aks_get_credentials). Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurecli-interactive
az aks get-credentials --resource-group wordpress-project --name myAKSCluster
```

> [!NOTE]
> El comando anterior usa la ubicación predeterminada para el [archivo de configuración de Kubernetes](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), `~/.kube/config`. Puede especificar otra ubicación para el archivo de configuración de Kubernetes con la opción *--file*.

Para comprobar la conexión al clúster, use el comando [kubectl get]( https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) para devolver una lista de los nodos del clúster.

```azurecli-interactive
kubectl get nodes
```

La salida del ejemplo siguiente muestra el nodo único creado en los pasos anteriores. Asegúrese de que el estado del nodo es *Listo*:

```output
NAME                       STATUS   ROLES   AGE     VERSION
aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.12.8
```

## <a name="create-an-azure-database-for-mysql---flexible-server"></a>Creación de un Servidor flexible de Azure Database for MySQL
Cree un servidor flexible con el comando [az mysql flexible-server create](/cli/azure/mysql/flexible-server). El siguiente comando crea un servidor con los valores predeterminados de servicio y los valores del contexto local de la CLI de Azure:

```azurecli-interactive
az mysql flexible-server create --public-access <YOUR-IP-ADDRESS>
```

El servidor creado tiene los siguientes atributos:
- Una nueva base de datos vacía, ```flexibleserverdb``` se crea cuando se aprovisiona el servidor por primera vez. En este inicio rápido, vamos a utilizar esta base de datos.
- Nombre del servidor generado automáticamente, nombre de usuario de administrador, contraseña de administrador, nombre del grupo de recursos (si aún no se ha especificado en el contexto local) y en la misma ubicación que el grupo de recursos.
- Valores predeterminados de servicio para las configuraciones de servidor restantes: nivel de proceso (Flexible), tamaño de proceso/SKU (B1MS), período de retención de copia de seguridad (7 días) y versión de MySQL (5.7)
- El uso de un argumento de acceso público le permite crear un servidor con acceso público protegido por reglas de firewall. Al proporcionar su dirección IP para agregar la regla de firewall, permite el acceso desde la máquina cliente.
- Como el comando utiliza el contexto local, creará el servidor en el grupo de recursos ```wordpress-project``` y en la región ```eastus```.


### <a name="build-your-wordpress-docker-image"></a>Compilación de la imagen de Docker de WordPress

Descargue la versión de [WordPress más reciente](https://wordpress.org/download/). Cree un nuevo directorio ```my-wordpress-app``` para el proyecto y utilice esta estructura de carpetas sencilla.

```
└───my-wordpress-app
    └───public
        ├───wp-admin
        │   ├───css
        . . . . . . .
        ├───wp-content
        │   ├───plugins
        . . . . . . .
        └───wp-includes
        . . . . . . .
        ├───wp-config-sample.php
        ├───index.php
        . . . . . . .
    └─── Dockerfile

```


Cambie el nombre de ```wp-config-sample.php``` a ```wp-config.php``` y reemplace las líneas de 21 a 32 por este fragmento de código. El siguiente fragmento de código lee el host de base de datos, el nombre de usuario y la contraseña del archivo de manifiesto de Kubernetes.

```php
//Using environment variables for DB connection information

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */

$connectstr_dbhost = getenv('DATABASE_HOST');
$connectstr_dbusername = getenv('DATABASE_USERNAME');
$connectstr_dbpassword = getenv('DATABASE_PASSWORD');

/** MySQL database name */
define('DB_NAME', 'flexibleserverdb');

/** MySQL database username */
define('DB_USER', $connectstr_dbusername);

/** MySQL database password */
define('DB_PASSWORD',$connectstr_dbpassword);

/** MySQL hostname */
define('DB_HOST', $connectstr_dbhost);

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');


/** SSL*/
define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
```

### <a name="create-a-dockerfile"></a>Creación de un archivo Dockerfile
Cree un nuevo Dockerfile y copie este fragmento de código. Este Dockerfile configura el servidor web Apache con PHP y habilita la extensión de mysqli.

```docker
FROM php:7.2-apache
COPY public/ /var/www/html/
RUN docker-php-ext-install mysqli
RUN docker-php-ext-enable mysqli
```

### <a name="build-your-docker-image"></a>Compilación de la imagen de Docker
Asegúrese de que se encuentra en el directorio ```my-wordpress-app``` en un terminal con el comando ```cd```. Ejecute el siguiente comando para compilar la imagen:

``` bash

docker build --tag myblog:latest .

```

Inserte la imagen en [Docker hub](https://docs.docker.com/get-started/part3/#create-a-docker-hub-repository-and-push-your-image) o [Azure Container Registry](../../container-registry/container-registry-get-started-azure-cli.md).

> [!IMPORTANT]
>Si utiliza Azure Container Registry (ACR), ejecute el comando ```az aks update``` para asociar la cuenta de ACR con el clúster de AKS.
>
>```azurecli-interactive
>az aks update -n myAKSCluster -g wordpress-project --attach-acr <your-acr-name>
> ```
>

## <a name="create-kubernetes-manifest-file"></a>Creación del archivo de manifiesto de Kubernetes

Un archivo de manifiesto de Kubernetes define un estado deseado del clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. Vamos a crear un archivo de manifiesto denominado `mywordpress.yaml` y a copiarlo en la siguiente definición de YAML.

>[!IMPORTANT]
> - Reemplace ```[DOCKER-HUB-USER/ACR ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]``` por el nombre y la etiqueta de la imagen real de Docker de WordPress, por ejemplo ```docker-hub-user/myblog:latest```.
> - Actualice la sección ```env``` siguiente con ```SERVERNAME```, ```YOUR-DATABASE-USERNAME```, ```YOUR-DATABASE-PASSWORD``` del servidor flexible de MySQL.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress-blog
  template:
    metadata:
      labels:
        app: wordpress-blog
    spec:
      containers:
      - name: wordpress-blog
        image: [DOCKER-HUB-USER-OR-ACR-ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "SERVERNAME.mysql.database.azure.com"
        - name: DATABASE_USERNAME
          value: "YOUR-DATABASE-USERNAME"
        - name: DATABASE_PASSWORD
          value: "YOUR-DATABASE-PASSWORD"
        - name: DATABASE_NAME
          value: "flexibleserverdb"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - wordpress-blog
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: v1
kind: Service
metadata:
  name: php-svc
spec:
  type: LoadBalancer
  ports:
    - port: 80
  selector:
    app: wordpress-blog
```

## <a name="deploy-wordpress-to-aks-cluster"></a>Implementación de WordPress en el clúster de AKS
Implemente la aplicación mediante el comando [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f mywordpress.yaml
```

En la salida de ejemplo siguiente se muestran las implementaciones y los servicios creados correctamente:

```output
deployment "wordpress-blog" created
service "php-svc" created
```

## <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet. Este proceso puede tardar unos minutos en completarse.

Para supervisar el progreso, utilice el comando [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) con el argumento `--watch`.

```azurecli-interactive
kubectl get service wordpress-blog --watch
```

En un primer momento, el parámetro *EXTERNAL-IP* del servicio *wordpress-blog* se muestra como *pendiente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress-blog   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Cuando la dirección *EXTERNAL-IP* cambie de *pendiente* a una dirección IP pública real, use `CTRL-C` para detener el proceso de inspección de `kubectl`. En la salida del ejemplo siguiente se muestra una dirección IP pública válida asignada al servicio:

```output
wordpress-blog  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

### <a name="browse-wordpress"></a>Examen de WordPress

Abra un explorador web en la dirección IP externa del servicio para ver la página de instalación de WordPress.

   :::image type="content" source="./media/tutorial-deploy-wordpress-on-aks/wordpress-aks-installed-success.png" alt-text="Instalación de WordPress correcta en AKS y el servidor flexible de MySQL":::

>[!NOTE]
> - Actualmente, el sitio de WordPress no utiliza HTTPS. Se recomienda [HABILITAR TLS con sus propios certificados](../../aks/ingress-own-tls.md).
> - Puede habilitar el [enrutamiento HTTP](../../aks/http-application-routing.md) para el clúster.

## <a name="clean-up-the-resources"></a>Limpiar los recursos

Para evitar los cargos de Azure, se recomienda limpiar los recursos que no sean necesarios.  Cuando el clúster ya no se necesite, puede usar el comando [az group delete](/cli/azure/group#az_group_delete) para eliminar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurecli-interactive
az group delete --name wordpress-project --yes --no-wait
```

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS](../../aks/kubernetes-service-principal.md#additional-considerations). Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [acceder al panel web de Kubernetes](../../aks/kubernetes-dashboard.md) del clúster de AKS.
- Aprenda a [escalar el clúster](../../aks/tutorial-kubernetes-scale.md).
- Aprenda a administrar el [servidor flexible de MySQL](./quickstart-create-server-cli.md).
- Aprenda a [configurar parámetros de servidor](./how-to-configure-server-parameters-cli.md) en el servidor de bases de datos.
