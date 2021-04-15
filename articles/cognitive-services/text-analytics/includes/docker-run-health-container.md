---
title: Ejecución del ejemplo de contenedor del comando de ejecución de Docker
titleSuffix: Azure Cognitive Services
description: Comando de ejecución de Docker para el contenedor de estado
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/12/2020
ms.author: aahi
ms.openlocfilehash: 3a89ddbc17eab683c658430a64447328ee024a3c
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106089717"
---
## <a name="install-the-container"></a>Instalación del contenedor

Hay varias maneras de instalar y ejecutar el contenedor de Text Analytics for Health. 

- Utilice [Azure Portal](../how-tos/text-analytics-how-to-install-containers.md?tabs=healthcare) para crear un recurso de Text Analytics y Docker para obtener su contenedor.
- Use los siguientes scripts de PowerShell y la CLI de Azure para automatizar la implementación de recursos y la configuración del contenedor.

### <a name="run-the-container-locally"></a>Ejecute el contenedor localmente

Para ejecutar el contenedor en su propio entorno después de descargar la imagen de contenedor, busque el identificador de la imagen:
 
```bash
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

Ejecute el comando `docker run` siguiente. Reemplace los marcadores de posición por sus propios valores:

| Marcador de posición | Value | Formato o ejemplo |
|-------------|-------|---|
| **{CLAVE_API}** | Clave del recurso de Text Analytics. Puede encontrarla en la página **Clave y punto de conexión** del recurso en Azure Portal. |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{URI_PUNTODECONEXIÓN}** | Punto de conexión para acceder a Text Analytics API. Puede encontrarla en la página **Clave y punto de conexión** del recurso en Azure Portal. | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |
| **{IMAGE_ID}** | Identificador de la imagen del contenedor. | `1.1.011300001-amd64-preview` |
| **{INPUT_DIR}** | Directorio de entrada del contenedor. | Windows: `C:\healthcareMount` <br> Linux/MacOS: `/home/username/input` |

```bash
docker run --rm -it -p 5000:5000 --cpus 6 --memory 12g \
--mount type=bind,src={INPUT_DIR},target=/output {IMAGE_ID} \
Eula=accept \
rai_terms=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Disk:Format=json
```

Este comando:

- Supone que el directorio de entrada existe en el equipo host.
- Ejecuta un contenedor de Text Analytics for Health desde la imagen del contenedor.
- Asigna un núcleo de 6 CPU y 12 gigabytes (GB) de memoria.
- Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
- Acepta el contrato de licencia para el usuario final (CLUF) y los términos de AI (RAI) responsables.
- Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

### <a name="demo-ui-to-visualize-output"></a>Interfaz de usuario de demostración para visualizar la salida

> [!NOTE]
> La demostración solo está disponible con el contenedor de Text Analytics for Health.

El contenedor proporciona varias API de puntos de conexión de predicción de consultas basadas en REST.  También hemos proporcionado una herramienta de visualización en el contenedor a la que se puede acceder mediante la anexión de `/demo` al punto de conexión del contenedor. Por ejemplo:

```
http://<serverURL>:5000/demo
```

Use la siguiente solicitud de dirección de URL como ejemplo para enviar una consulta al contenedor que ha implementado reemplazando la variable `serverURL` por el valor adecuado.

```bash
curl -X POST 'http://<serverURL>:5000/text/analytics/v3.1-preview.4/entities/health' --header 'Content-Type: application/json' --header 'accept: application/json' --data-binary @example.json

```

### <a name="install-the-container-using-azure-web-app-for-containers"></a>Instalación del contenedor con el Web App for Containers de Azure

La [Web App for Containers](https://azure.microsoft.com/services/app-service/containers/) de Azure es un recurso de Azure dedicado a la ejecución de contenedores en la nube. Ofrece funcionalidades preparadas como el escalado automático, la compatibilidad con contenedores de Docker y Docker Compose, la compatibilidad con HTTPS y mucho más.

> [!NOTE]
> Con la aplicación Web de Azure, obtendrá automáticamente un dominio en forma de `<appservice_name>.azurewebsites.net`.

Ejecute este script de PowerShell con la CLI de Azure para crear una Web App for Containers mediante su suscripción y la imagen del contenedor a través de HTTPS. Espere a que se complete el script (aproximadamente de 25 a 30 minutos) antes de enviar la primera solicitud.

```azurecli
$subscription_name = ""                    # THe name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           #    and AppSerivce to be attached to.
$resources_location = ""                   # This is the location you wish the AppServicePlan to be deployed to.
                                           #    You can use the "az account list-locations -o table" command to
                                           #    get the list of available locations and location code names.
$appservice_plan_name = ""                 # This is the AppServicePlan name you wish to have.
$appservice_name = ""                      # This is the AppService resource name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_IMAGE_NAME = "mcr.microsoft.com/azure-cognitive-services/textanalytics/healthcare:latest"

az login
az account set -s $subscription_name
az appservice plan create -n $appservice_plan_name -g $resource_group_name --is-linux -l $resources_location --sku P3V2
az webapp create -g $resource_group_name -p $appservice_plan_name -n $appservice_name -i $DOCKER_IMAGE_NAME 
az webapp config appsettings set -g $resource_group_name -n $appservice_name --settings Eula=accept rai_terms=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: https://<appservice_name>.azurewebsites.net
```

### <a name="install-the-container-using-azure-container-instance"></a>Instalación del contenedor con la instancia de contenedor de Azure

También puede usar una instancia de contenedor de Azure (ACI) para facilitar la implementación. ACI es un recurso que le permite ejecutar contenedores de Docker a petición en un entorno de Azure administrado y sin servidor. 

Consulte [Cómo usar Azure Container Instances](../../containers/azure-container-instance-recipe.md) para conocer los pasos necesarios para implementar un recurso ACI con Azure Portal. También puede usar el siguiente script de PowerShell con CLI de Azure, que creará un ACI en su suscripción con la imagen del contenedor.  Espere a que se complete el script (aproximadamente de 25 a 30 minutos) antes de enviar la primera solicitud.  Debido al límite en el número máximo de CPU por recurso de ACI, no seleccione esta opción si espera enviar más de 5 documentos grandes (aproximadamente 5000 caracteres cada uno) por solicitud.
Para obtener información de disponibilidad, consulte el artículo [Compatibilidad de regiones de ACI](../../../container-instances/container-instances-region-availability.md). 

> [!NOTE] 
> Azure Container Instances no incluye compatibilidad con HTTPS para los dominios integrados. Si necesita HTTPS, tendrá que configurarlo manualmente, incluida la creación de un certificado y el registro de un dominio. Puede encontrar instrucciones para hacerlo con NGINX a continuación.

```azurecli
$subscription_name = ""                    # The name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           # and AppService to be attached to.
$resources_location = ""                   # This is the location you wish the web app to be deployed to.
                                           # You can use the "az account list-locations -o table" command to
                                           # Get the list of available locations and location code names.
$azure_container_instance_name = ""        # This is the AzureContainerInstance name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DNS_LABEL = ""                            # This is the DNS label name you wish your ACI will have
$DOCKER_IMAGE_NAME = "mcr.microsoft.com/azure-cognitive-services/textanalytics/healthcare:latest"

az login
az account set -s $subscription_name
az container create --resource-group $resource_group_name --name $azure_container_instance_name --image $DOCKER_IMAGE_NAME --cpu 4 --memory 12 --port 5000 --dns-name-label $DNS_LABEL --environment-variables Eula=accept rai_terms=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: http://<unique_dns_label>.<resource_group_region>.azurecontainer.io:5000
```

### <a name="secure-aci-connectivity"></a>Conectividad ACI segura

De forma predeterminada, no se proporciona ninguna seguridad al usar ACI con el API del contenedor. Esto es porque la mayoría de las veces el contenedor se ejecuta como parte de un pod que está protegido desde fuera por un puente de red. Sin embargo, puede modificar un contenedor con un componente orientado hacia delante, manteniendo como privado el punto de conexión del contenedor. En los ejemplos siguientes se usa [NGINX](https://www.nginx.com) como puerta de enlace de entrada para admitir HTTPS/SSL y la autenticación del certificado de cliente.

> [!NOTE]
> NGINX es un servidor HTTP y proxy de alto rendimiento de código abierto. Un contenedor NGINX se puede usar para terminar una conexión TLS para un único contenedor. También se pueden producir soluciones de terminación TLS más complejas basadas en la entrada NGINX.

#### <a name="set-up-nginx-as-an-ingress-gateway"></a>Configuración de NGINX como puerta de enlace de entrada

NGINX usa [archivos de configuración](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/) para habilitar características en runtime. Para habilitar la terminación TLS para otro servicio, debe especificar un certificado SSL para finalizar la conexión TLS y `proxy_pass` para especificar una dirección para el servicio. Se proporciona una muestra a continuación.


> [!NOTE]
> `ssl_certificate` espera que se especifique una ruta de acceso en el sistema de archivos local del contenedor de NGINX. La dirección especificada para `proxy_pass` debe estar disponible en la red del contenedor de NGINX.

El contenedor NGINX cargará todos los archivos del `_.conf_` que se montan bajo `/etc/nginx/conf.d/` en la ruta de acceso de configuración de HTTP.

```nginx
server {
  listen              80;
  return 301 https://$host$request_uri;
}
server {
  listen              443 ssl;
  # replace with .crt and .key paths
  ssl_certificate     /cert/Local.crt;
  ssl_certificate_key /cert/Local.key;

  location / {
    proxy_pass http://cognitive-service:5000;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP  $remote_addr;
  }
}
```

#### <a name="example-docker-compose-file"></a>Archivo de muestra de Docker Compose

En el ejemplo siguiente se muestra cómo se puede crear un archivo de [Docker Compose](https://docs.docker.com/compose/reference/overview) a fin de implementar los contenedores de NGINX y Text Analytics para el estado:

```yaml
version: "3.7"
services:
  cognitive-service:
    image: {IMAGE_ID}
    ports:
      - 5000:5000
    environment:
      - eula=accept
      - billing={ENDPOINT_URI}
      - apikey={API_KEY}
      - Logging:Disk:Format=json
    volumes:
        # replace with path to logs folder
      - <path-to-logs-folder>:/output
  nginx:
    image: nginx
    ports:
      - 443:443
    volumes:
        # replace with paths for certs and conf folders
      - <path-to-certs-folder>:/cert
      - <path-to-conf-folder>:/etc/nginx/conf.d/
```

Para iniciar este archivo de Docker Compose, ejecute el siguiente comando desde una consola en el nivel raíz del archivo:

```bash
docker-compose up
```

Para obtener más información, consulte la documentación de NGINX sobre la [Terminación SSL de NGINX](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/).
