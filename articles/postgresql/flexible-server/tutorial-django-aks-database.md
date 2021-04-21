---
title: 'Tutorial: Implementación de Django en un clúster de AKS con Servidor flexible de PostgreSQL mediante la CLI de Azure'
description: Aprenda a compilar e implementar rápidamente Django en AKS con Servidor flexible de Azure Database for PostgreSQL.
ms.service: postgresql
author: mksuni
ms.author: sumuth
ms.topic: tutorial
ms.date: 12/10/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 9315e6fd7dd9880d20108e3f0ed28cd32904f1a3
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791542"
---
# <a name="tutorial-deploy-django-app-on-aks-with-azure-database-for-postgresql---flexible-server"></a>Tutorial: Implementación de una aplicación Django en AKS con Servidor flexible de Azure Database for PostgreSQL

En este inicio rápido se implementa una aplicación Django en el clúster de Azure Kubernetes Service (AKS) con Servidor flexible de Azure Database for PostgreSQL (versión preliminar) mediante la CLI de Azure.

**[AKS](../../aks/intro-kubernetes.md)** es un servicio de Kubernetes administrado que permite implementar y administrar clústeres rápidamente. **[Servidor flexible de Azure Database for PostgreSQL (versión preliminar)](overview.md)** es un servicio de base de datos totalmente administrado diseñado para proporcionar un control más granular y una mayor flexibilidad sobre las funciones de administración de bases de datos y las opciones de configuración.

> [!NOTE]
> - Actualmente Servidor flexible de Azure Database for PostgreSQL se encuentra en versión preliminar.
> - En este inicio rápido se presupone un conocimiento básico de los conceptos de Kubernetes, Django y PostgreSQL.

## <a name="pre-requisites"></a>Requisitos previos
[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

- Use [Azure Cloud Shell](../../cloud-shell/quickstart.md) con el entorno de Bash.

   [![Insertar inicio](https://shell.azure.com/images/launchcloudshell.png "Inicio de Azure Cloud Shell")](https://shell.azure.com)  
- Si lo prefiere, [instale](/cli/azure/install-azure-cli) la CLI de Azure para ejecutar los comandos de referencia de la CLI.
  - Si usa una instalación local, inicie sesión con la CLI de Azure mediante el comando [az login](/cli/azure/reference-index#az_login).  Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.  Para ver otras opciones de inicio de sesión, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).
  - Cuando se le solicite, instale las extensiones de la CLI de Azure la primera vez que la use.  Para más información, consulte [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview).
  - Ejecute [az version](/cli/azure/reference-index?#az_version) para buscar cuál es la versión y las bibliotecas dependientes que están instaladas. Para realizar la actualización a la versión más reciente, ejecute [az upgrade](/cli/azure/reference-index?#az_upgrade). En este artículo se necesita la versión más reciente de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

> [!NOTE]
> Si los comandos de este inicio rápido se ejecutan localmente (en lugar de en Azure Cloud Shell), asegúrese de ejecutarlos como administrador.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un grupo lógico en el que se implementan y administran recursos de Azure. Vamos a crear un grupo de recursos, *django-project*, mediante el comando [az group create][az-group-create] en la ubicación *eastus*.

```azurecli-interactive
az group create --name django-project --location eastus
```

> [!NOTE]
> La ubicación del grupo de recursos es donde se almacenan los metadatos del grupo de recursos. Es también el lugar en el que se ejecutan los recursos en Azure si no se especifica otra región al crear los recursos.

En la siguiente salida de ejemplo se muestra que los recursos se crearon correctamente:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/django-project",
  "location": "eastus",
  "managedBy": null,
  
  "name": "django-project",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-aks-cluster"></a>Creación de un clúster de AKS

Use el comando [az aks create](/cli/azure/aks#az_aks_create) para crear un clúster de AKS. En el siguiente ejemplo se crea un clúster denominado *myAKSCluster* con un nodo. Este proceso tardará varios minutos en completarse.

```azurecli-interactive
az aks create --resource-group django-project --name djangoappcluster --node-count 1 --generate-ssh-keys
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
az aks get-credentials --resource-group django-project --name djangoappcluster
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

## <a name="create-an-azure-database-for-postgresql---flexible-server"></a>Creación de una instancia de Servidor flexible de Azure Database for PostgreSQL
Cree un servidor flexible con el comando [az postgreSQL flexible-server create](/cli/azure/postgres/flexible-server#az_postgres_flexible_server_create). El siguiente comando crea un servidor con los valores predeterminados de servicio y los valores del contexto local de la CLI de Azure:

```azurecli-interactive
az postgres flexible-server create --public-access <YOUR-IP-ADDRESS>
```

El servidor creado tiene los siguientes atributos:
- Una nueva base de datos vacía, ```postgres``` se crea cuando se aprovisiona el servidor por primera vez. En este inicio rápido, vamos a utilizar esta base de datos.
- Nombre del servidor generado automáticamente, nombre de usuario de administrador, contraseña de administrador, nombre del grupo de recursos (si aún no se ha especificado en el contexto local) y en la misma ubicación que el grupo de recursos.
- Valores predeterminados de servicio para las configuraciones de servidor restantes: nivel de proceso (De uso general), tamaño de proceso/SKU (Standard_D2s_v3 que usa 2 núcleos virtuales), período de retención de copia de seguridad (7 días) y versión de PostgreSQL (12)
- El uso de un argumento de acceso público le permite crear un servidor con acceso público protegido por reglas de firewall. Al proporcionar su dirección IP para agregar la regla de firewall, permite el acceso desde la máquina cliente.
- Como el comando utiliza el contexto local, creará el servidor en el grupo de recursos ```django-project``` y en la región ```eastus```.


## <a name="build-your-django-docker-image"></a>Compilación de la imagen de Docker de Django

Cree una [aplicación Django](https://docs.djangoproject.com/en/3.1/intro/) o use el proyecto Django existente. Asegúrese de que el código se encuentra en esta estructura de carpetas.

```
└───my-djangoapp
    └───views.py
    └───models.py
    └───forms.py
    ├───templates
          . . . . . . .
    ├───static
         . . . . . . .
└───my-django-project
    └───settings.py
    └───urls.py
    └───wsgi.py
        . . . . . . .
    └─── Dockerfile
    └─── requirements.txt
    └─── manage.py
    
```
Actualice ```ALLOWED_HOSTS``` en ```settings.py``` para asegurarse de que la aplicación Django usa la IP externa que se asigna a la aplicación kubernetes.

```python
ALLOWED_HOSTS = ['*']
```

Actualice la sección ```DATABASES={ }``` del archivo ```settings.py```. El siguiente fragmento de código lee el host de base de datos, el nombre de usuario y la contraseña del archivo de manifiesto de Kubernetes.

```python
DATABASES={
   'default':{
      'ENGINE':'django.db.backends.postgresql_psycopg2',
      'NAME':os.getenv('DATABASE_NAME'),
      'USER':os.getenv('DATABASE_USER'),
      'PASSWORD':os.getenv('DATABASE_PASSWORD'),
      'HOST':os.getenv('DATABASE_HOST'),
      'PORT':'5432',
      'OPTIONS': {'sslmode': 'require'}
   }
}
```

### <a name="generate-a-requirementstxt-file"></a>Generación de un archivo requirements.txt
Cree un archivo ```requirements.txt``` para enumerar las dependencias de la aplicación Django. Este es un archivo ```requirements.txt``` de ejemplo. Puede usar [``` pip freeze > requirements.txt```](https://pip.pypa.io/en/stable/reference/pip_freeze/) para generar un archivo requirements.txt para la aplicación existente.

``` text
Django==2.2.17
postgres==3.0.0
psycopg2-binary==2.8.6
psycopg2-pool==1.1
pytz==2020.4
```

### <a name="create-a-dockerfile"></a>Creación de un archivo Dockerfile
Cree un archivo llamado ```Dockerfile``` y copie el siguiente fragmento de código. Use este archivo Dockerfile al configurar Python 3.8 e instalar todos los requisitos enumerados en el archivo requirements.txt.

```docker
# Use the official Python image from the Docker Hub
FROM python:3.8.2

# Make a new directory to put our code in.
RUN mkdir /code

# Change the working directory.
WORKDIR /code

# Copy to code folder
COPY . /code/

# Install the requirements.
RUN pip install -r requirements.txt

# Run the application:
CMD python manage.py runserver 0.0.0.0:8000
```

### <a name="build-your-image"></a>Compilación de la imagen
Asegúrese de que se encuentra en el directorio ```my-django-app``` en un terminal con el comando ```cd```. Ejecute el siguiente comando para compilar la imagen del tablón de anuncios:

``` bash

docker build --tag myblog:latest .

```

Inserte la imagen en [Docker hub](https://docs.docker.com/get-started/part3/#create-a-docker-hub-repository-and-push-your-image) o [Azure Container Registry](../../container-registry/container-registry-get-started-azure-cli.md).

> [!IMPORTANT]
>Si utiliza Azure Container Registry (ACR), ejecute el comando ```az aks update``` para asociar la cuenta de ACR con el clúster de AKS.
>
>```azurecli-interactive
>az aks update -n myAKSCluster -g django-project --attach-acr <your-acr-name>
> ```
>

## <a name="create-kubernetes-manifest-file"></a>Creación del archivo de manifiesto de Kubernetes

Un archivo de manifiesto de Kubernetes define un estado deseado del clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. Vamos a crear un archivo de manifiesto denominado ```djangoapp.yaml``` y a copiarlo en la siguiente definición de YAML.

>[!IMPORTANT]
> - Reemplace ```[DOCKER-HUB-USER/ACR ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]``` por el nombre y la etiqueta de la imagen real de Docker de Django, por ejemplo ```docker-hub-user/myblog:latest```.
> - Actualice la sección ```env``` siguiente con ```SERVERNAME```, ```YOUR-DATABASE-USERNAME```, ```YOUR-DATABASE-PASSWORD``` del servidor flexible de Postgres.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: django-app
  template:
    metadata:
      labels:
        app: django-app
    spec:
      containers:
      - name: django-app
        image: [DOCKER-HUB-USER-OR-ACR-ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "SERVERNAME.postgres.database.azure.com"
        - name: DATABASE_USERNAME
          value: "YOUR-DATABASE-USERNAME"
        - name: DATABASE_PASSWORD
          value: "YOUR-DATABASE-PASSWORD"
        - name: DATABASE_NAME
          value: "postgres"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - django-app
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: v1
kind: Service
metadata:
  name: python-svc
spec:
  type: LoadBalancer
  ports:
    - port: 8000
  selector:
    app: django-app
```

## <a name="deploy-django-to-aks-cluster"></a>Implementación de Django en un clúster de AKS
Implemente la aplicación mediante el comando [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f djangoapp.yaml
```

En la salida de ejemplo siguiente se muestran las implementaciones y los servicios creados correctamente:

```output
deployment "django-app" created
service "python-svc" created
```

Un ```django-app``` de implementación permite describir los detalles de la implementación, como por ejemplo las imágenes que se van a usar para la aplicación, el número de pods y su configuración. Se crea un servicio ```python-svc``` para exponer la aplicación a través de una IP externa.

## <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet. Este proceso puede tardar unos minutos en completarse.

Para supervisar el progreso, utilice el comando [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) con el argumento `--watch`.

```azurecli-interactive
kubectl get service django-app --watch
```

En un primer momento, el parámetro *EXTERNAL-IP* del servicio de *django-app* se muestra como *pendiente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
django-app   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Cuando la dirección *EXTERNAL-IP* cambie de *pendiente* a una dirección IP pública real, use `CTRL-C` para detener el proceso de inspección de `kubectl`. En la salida del ejemplo siguiente se muestra una dirección IP pública válida asignada al servicio:

```output
django-app  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Ahora abra un explorador web en la dirección IP externa del servicio para ver la aplicación Django.  

>[!NOTE]
> - Actualmente, el sitio de Django no utiliza HTTPS. Se recomienda [HABILITAR TLS con sus propios certificados](../../aks/ingress-own-tls.md).
> - Puede habilitar el [enrutamiento HTTP](../../aks/http-application-routing.md) para el clúster. Cuando se habilita el enrutamiento HTTP, configura un controlador de entrada en el clúster de AKS. A medida que se implementan aplicaciones, la solución también crea nombres DNS accesibles públicamente para los puntos de conexión de aplicación.

## <a name="run-database-migrations"></a>Ejecución de migraciones de base de datos

En el caso de las aplicaciones Django, debería ejecutar la migración de base de datos o recopilar archivos estáticos. Para ejecutar estos comandos de la shell de Django se usa ```$ kubectl exec <pod-name> -- [COMMAND]```.  Antes de ejecutar el comando debe buscar el nombre del pod mediante ```kubectl get pods```. 

```bash
$ kubectl get pods
```

Verá una salida similar a esta:
```output
NAME                             READY   STATUS          RESTARTS   AGE
django-app-5d9cd6cd8-l6x4b     1/1     Running              0       2m
```

Una vez que haya encontrado el nombre del pod, puede ejecutar las migraciones de base de datos de Django con el comando ```$ kubectl exec <pod-name> -- [COMMAND]```. Tenga en cuenta que ```/code/``` es el directorio de trabajo del proyecto que se ha definido en ```Dockerfile```.

```bash
$ kubectl exec django-app-5d9cd6cd8-l6x4b -- python /code/manage.py migrate
```

La salida sería similar a esta: 
```output 
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  . . . . . . 
```

Si surgen problemas, ejecute ```kubectl logs <pod-name>``` para ver qué excepción genera la aplicación. Si la aplicación funciona correctamente, verá una salida como la siguiente al ejecutar ```kubectl logs```.

```output
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
December 08, 2020 - 23:24:14
Django version 2.2.17, using settings 'django_postgres_app.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

## <a name="clean-up-the-resources"></a>Limpiar los recursos

Para evitar los cargos de Azure, se recomienda limpiar los recursos que no sean necesarios.  Cuando el clúster ya no se necesite, puede usar el comando [az group delete](/cli/azure/group#az_group_delete) para eliminar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurecli-interactive
az group delete --name django-project --yes --no-wait
```

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS](../../aks/kubernetes-service-principal.md#additional-considerations). Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [acceder al panel web de Kubernetes](../../aks/kubernetes-dashboard.md) del clúster de AKS.
- Aprenda a [habilitar la implementación continua](../../aks/deployment-center-launcher.md)
- Aprenda a [escalar el clúster](../../aks/tutorial-kubernetes-scale.md).
- Aprenda a administrar el [servidor flexible de Postgres](./quickstart-create-server-cli.md).
- Aprenda a [configurar parámetros de servidor](./howto-configure-server-parameters-using-cli.md) en el servidor de bases de datos.
