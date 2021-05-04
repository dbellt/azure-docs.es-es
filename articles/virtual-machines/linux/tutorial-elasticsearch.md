---
title: Implementar ElasticSearch en una máquina virtual de desarrollo de Azure
description: Instalación de Elastic Stack (ELK) en una máquina virtual de desarrollo con Linux en Azure
services: virtual-machines
author: rloutlaw
manager: justhe
ms.service: virtual-machines
ms.collection: linux
ms.workload: infrastructure-services
ms.devlang: azurecli
ms.topic: how-to
ms.date: 10/11/2017
ms.author: routlaw
ms.openlocfilehash: 3ce28efa30b04a2a762c39f47dac739633eac209
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107818483"
---
# <a name="install-the-elastic-stack-elk-on-an-azure-vm"></a>Instalación de Elastic Stack (ELK) en una máquina virtual de Azure

En este artículo se le guiará acerca de cómo implementar [Elasticsearch](https://www.elastic.co/products/elasticsearch), [Logstash](https://www.elastic.co/products/logstash) y [Kibana](https://www.elastic.co/products/kibana) en una máquina virtual con Ubuntu en Azure. Para ver Elastic Stack en acción, puede conectarse opcionalmente a Kibana y trabajar con algunos de los datos de registro de ejemplo. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una máquina virtual con Ubuntu en un grupo de recursos de Azure
> * Instalar Elasticsearch, Logstash y Kibana en la máquina virtual
> * Enviar datos de ejemplo a Elasticsearch con Logstash 
> * Abrir los puertos y trabajar con datos en la consola de Kibana


 Esta implementación es adecuada para el desarrollo básico con Elastic Stack. Para más información acerca de Elastic Stack, incluidas recomendaciones para un entorno de producción, consulte la [documentación de Elastic](https://www.elastic.co/guide/index.html) y [Azure Architecture Center](/azure/architecture/elasticsearch/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.4 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree la máquina virtual con el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente, se crea una máquina virtual denominada *myVM* y las claves SSH si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`.  

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Cuando se ha creado la máquina virtual, la CLI de Azure muestra información similar al ejemplo siguiente. Anote el valor de `publicIpAddress`. Esta dirección se utiliza para tener acceso a la máquina virtual.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="ssh-into-your-vm"></a>Conexión SSH con la máquina virtual

Si no conoce la dirección IP pública de la máquina virtual, ejecute el comando [az network public-ip list](/cli/azure/network/public-ip):

```azurecli-interactive
az network public-ip list --resource-group myResourceGroup --query [].ipAddress
```

Ejecute el comando siguiente para crear una sesión SSH con la máquina virtual. Sustituya la dirección IP pública correcta de la máquina virtual. En este ejemplo, la dirección IP es *40.68.254.142*.

```bash
ssh azureuser@40.68.254.142
```

## <a name="install-the-elastic-stack"></a>Instalación de Elastic Stack

Importe la clave de firma de Elasticsearch y actualice la lista de orígenes de APT para incluir el repositorio del paquete de Elastic:

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

Instale Java Virtual en la máquina virtual y configure la variable JAVA_HOME necesaria para que se ejecuten los componentes de Elastic Stack.

```bash
sudo apt update && sudo apt install openjdk-8-jre-headless
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Ejecute los comandos siguientes para actualizar los orígenes de paquetes de Ubuntu e instalar Elasticsearch, Kibana y Logstash.

```bash
sudo apt update && sudo apt install elasticsearch kibana logstash   
```

> [!NOTE]
> Las instrucciones de instalación detalladas, incluidos los diseños de directorio y la configuración inicial, se incluyen en la [documentación de Elastic](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html)

## <a name="start-elasticsearch"></a>Iniciar Elasticsearch 

Inicie Elasticsearch en la máquina virtual con el siguiente comando:

```bash
sudo systemctl start elasticsearch.service
```

Este comando no produce ningún resultado, por lo tanto, compruebe que Elasticsearch está ejecutándose en la máquina virtual con este comando `curl`:

```bash
sudo curl -XGET 'localhost:9200/'
```

Si Elasticsearch se está ejecutando, verá un resultado similar al siguiente:

```json
{
  "name" : "w6Z4NwR",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "SDzCajBoSK2EkXmHvJVaDQ",
  "version" : {
    "number" : "5.6.3",
    "build_hash" : "1a2f265",
    "build_date" : "2017-10-06T20:33:39.012Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

## <a name="start-logstash-and-add-data-to-elasticsearch"></a>Iniciar Logstash y agregar datos a Elasticsearch

Inicie Logstash con el siguiente comando:

```bash 
sudo systemctl start logstash.service
```

Pruebe Logstash en el modo interactivo para asegurarse de que funciona correctamente:

```bash
sudo /usr/share/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

Se trata de una [canalización](https://www.elastic.co/guide/en/logstash/5.6/pipeline.html) básica de Logstash que repite la entrada estándar en la salida estándar. 

```output
The stdin plugin is now waiting for input:
hello azure
2017-10-11T20:01:08.904Z myVM hello azure
```

Configure Logstash para reenviar los mensajes del kernel de esta máquina virtual a Elasticsearch. Cree un archivo nuevo en un directorio vacío denominado `vm-syslog-logstash.conf` y pegue la configuración de Logstash siguiente:

```Logstash
input {
    stdin {
        type => "stdin-type"
    }

    file {
        type => "syslog"
        path => [ "/var/log/*.log", "/var/log/*/*.log", "/var/log/messages", "/var/log/syslog" ]
        start_position => "beginning"
    }
}

output {

    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts  => "localhost:9200"
    }
}
```

Pruebe esta configuración y envíe los datos de Syslog a Elasticsearch:

```bash
sudo /usr/share/logstash/bin/logstash -f vm-syslog-logstash.conf
```

Verá las entradas de Syslog de su terminal repetidas cuando se envíen a Elasticsearch. Use `CTRL+C` para salir de Logstash una vez envíe algunos datos.

## <a name="start-kibana-and-visualize-the-data-in-elasticsearch"></a>Inicie Kibana y visualice los datos en Elasticsearch

Edite `/etc/kibana/kibana.yml` y cambie la dirección IP que escucha Kibana para poder acceder a la misma desde el explorador web.

```bash
server.host: "0.0.0.0"
```

Inicie Kibana con el siguiente comando:

```bash
sudo systemctl start kibana.service
```

Abra el puerto 5601 desde la CLI de Azure para permitir el acceso remoto a la consola de Kibana:

```azurecli-interactive
az vm open-port --port 5601 --resource-group myResourceGroup --name myVM
```

Abra la consola de Kibana y seleccione **Crear** para generar un índice predeterminado en función de los datos de Syslog que envió previamente a Elasticsearch. 

![Captura de pantalla que muestra la consola de Kibana y resalta el botón Create (Crear).](media/elasticsearch-install/kibana-index.png)

Seleccione **Discover**  (Detectar) en la consola de Kibana para buscar, examinar y filtrar a través de los eventos de Syslog.

![Examinar los eventos de Syslog en Kibana](media/elasticsearch-install/kibana-search-filter.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, implementó Elastic Stack en una máquina virtual de desarrollo en Azure. Ha aprendido a:

> [!div class="checklist"]
> * Crear una máquina virtual con Ubuntu en un grupo de recursos de Azure
> * Instalar Elasticsearch, Logstash y Kibana en la máquina virtual
> * Enviar datos de ejemplo a Elasticsearch desde Logstash 
> * Abrir los puertos y trabajar con datos en la consola de Kibana
