---
title: Creación de un canal de Conda personalizado para la administración de paquetes
description: Obtenga información sobre cómo crear un canal de Conda personalizado para la administración de paquetes.
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 02/26/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: 26b6adefd2d334c9fe570bfa7e63bb06b55b9d20
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588775"
---
# <a name="create-a-custom-conda-channel-for-package-management"></a>Creación de un canal de Conda personalizado para la administración de paquetes 
Al instalar paquetes de Python, el administrador de paquetes de Conda usa canales para buscar paquetes. Es posible que tenga que crear un canal de Conda personalizado por varios motivos. Por ejemplo:

- El área de trabajo está protegida contra la filtración de datos y las conexiones salientes están bloqueadas.  
- Tiene paquetes que no quiere que se carguen en repositorios públicos.
- Quiere configurar un repositorio alternativo para los usuarios dentro del área de trabajo.

En este artículo, encontrará una guía paso a paso que le ayudará a crear su canal de Conda personalizado dentro de su cuenta de Azure Data Lake Storage.

## <a name="set-up-your-local-machine"></a>Configuración de la máquina local

1. Instale Conda en la máquina local. Puede consultar el [entorno de ejecución de Spark en Azure Synapse](./apache-spark-version-support.md) para identificar la versión de Conda que se usa en el mismo entorno en tiempo de ejecución.
   
2. Para crear un canal personalizado, instale conda-build.
```
conda install conda-build
```
3. Organice todos los paquetes para la plataforma que quiere atender. En este ejemplo, se instalará el archivo Anaconda en la máquina local.

```
sudo wget https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh 
sudo chmod +x Anaconda3-4.4.0-Linux-x86_64.sh  
sudo bash Anaconda3-4.4.0-Linux-x86_64.sh -b -p /usr/lib/anaconda3 
export PATH="/usr/lib/anaconda3/bin:$PATH" 
sudo chmod 777 -R /usr/lib/anaconda3a.  
```
## <a name="mount-the-storage-account-onto-your-machine"></a>Montaje de la cuenta de almacenamiento en la máquina
A continuación, montaremos la cuenta Azure Data Lake Storage Gen2 en la máquina local. Este proceso también se puede realizar con una cuenta de WASB. Sin embargo, veremos un ejemplo con la cuenta de ADLSg2. 
 
Para obtener más información sobre cómo montar la cuenta de almacenamiento en la máquina local, puede visitar [esta página](https://github.com/Azure/azure-storage-fuse#blobfuse ). 

1. Puede instalar blobfuse desde el repositorio de software de Linux para productos de Microsoft.

```
wget https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb 
sudo dpkg -i packages-microsoft-prod.deb 
sudo apt-get update 
sudo apt-get install blobfuse fuse 
export AZURE_STORAGE_ACCOUNT=<<myaccountname>
export AZURE_STORAGE_ACCESS_KEY=<<myaccountkey>>
export AZURE_STORAGE_BLOB_ENDPOINT=*.dfs.core.windows.net 
```

2. Cree el punto de montaje (```mkdir /path/to/mount```) y monte un contenedor de blobs con blobfuse. En este ejemplo, usaremos el valor **privatechannel** para la variable **mycontainer**.
   
```
blobfuse /path/to/mount --container-name=mycontainer --tmp-path=/mnt/blobfusetmp --use-adls=true --log-level=LOG_DEBUG 
sudo mkdir -p /mnt/blobfusetmp
sudo chown <myuser> /mnt/blobfusetmp
```
## <a name="create-the-channel"></a>Creación del canal
En el siguiente conjunto de pasos, crearemos un canal de Conda personalizado. 

1. En la máquina local, cree un directorio para organizar todos los paquetes para el canal personalizado.
   
```
mkdir /home/trusted-service-user/privatechannel 
cd ~/privatechannel/ 
mkdir channel1/linux64 
```

2. Organice todos los paquetes ```tar.bz2``` de https://repo.anaconda.com/pkgs/main/linux-64/ en el subdirectorio. Asegúrese de incluir también todos los paquetes tar.bz2 dependientes. 

```
cd channel1 
mkdir noarch 
echo '{}' > noarch/repodata.json 
bzip2 -k noarch/repodata.json 

// Create channel 

conda index channel1/noarch 
conda index channel1/linux-64 
conda index channel1 
```

Para obtener más información, también puede [visitar la guía de usuario de Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/create-custom-channels.html) para crear canales personalizados. 

## <a name="storage-account-permissions"></a>Permisos de la cuenta de almacenamiento
Ahora, tendrá que validar los permisos en la cuenta de almacenamiento. Para establecer estos permisos, vaya a la ruta de acceso donde se creará el canal personalizado. A continuación, cree un token de SAS para ```privatechannel``` que tenga permisos de lectura, lista y ejecución. 

El nombre del canal ahora será la dirección URL de SAS del blob que se genera a partir de este proceso.  

## <a name="create-a-sample-conda-environment-configuration-file"></a>Creación de un archivo de configuración de ejemplo para el entorno de Conda
Por último, compruebe el proceso de instalación mediante la creación de un archivo ```environment.yml``` de Conda de muestra. Si tiene un área de trabajo habilitada para DEP, debe especificar el canal ``nodefaults`` en el archivo del entorno.

Este es un ejemplo de archivo de configuración de Conda:
```
name: sample 
channels: 
  - https://<<storage account name>>.blob.core.windows.net/privatechannel/channel?<<SAS Token>
  - nodefaults 
dependencies: 
  - openssl 
  - ncurses 
```
Una vez que haya creado el archivo de Conda de ejemplo, puede crear un entorno de Conda virtual. 

```
conda env create –file sample.yml  
source activate env 
conda list 
```
Ahora que ha comprobado el canal personalizado, puede usar el proceso de [administración de grupos de Python](./apache-spark-manage-python-packages.md) para actualizar las bibliotecas en el grupo de Apache Spark.

## <a name="next-steps"></a>Pasos siguientes
- Visualización de las bibliotecas predeterminadas: [Compatibilidad de las versiones de Azure Spark](apache-spark-version-support.md)
- Administración de paquetes de Python: [Administración de paquetes de Python](./apache-spark-manage-python-packages.md)

