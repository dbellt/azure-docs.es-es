---
title: Integración de Git con Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo Azure Machine Learning se integra con un repositorio de Git local para realizar el seguimiento de la información del repositorio, la rama y la confirmación actual como parte de una ejecución de entrenamiento.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.date: 04/08/2021
ms.openlocfilehash: 2dc50702113f591075b790878347c4ca47beec4e
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107027812"
---
# <a name="git-integration-for-azure-machine-learning"></a>Integración de Git con Azure Machine Learning

[Git](https://git-scm.com/) es un conocido sistema de control de versiones que le permite compartir sus proyectos y colaborar con otros usuarios en ellos. 

Azure Machine Learning es totalmente compatible con los repositorios de Git para supervisar el trabajo; puede clonar repositorios directamente al sistema de archivos compartido de su área de trabajo, usar Git en su estación de trabajo local o usarlo desde una canalización de CI/CD.

Al enviar un trabajo a Azure Machine Learning, si los archivos de origen se almacenan en un repositorio de Git local, se realiza un seguimiento de la información sobre el repositorio como parte del proceso de entrenamiento.

Dado que Azure Machine Learning realiza un seguimiento de la información desde un repositorio de Git local, no está vinculado a ningún repositorio central específico. El repositorio se puede clonar desde GitHub, GitLab, Bitbucket, Azure DevOps o cualquier otro servicio compatible con Git.

> [!TIP]
> Use Visual Studio Code para interactuar con Git a través de una interfaz gráfica de usuario. Para conectarse a una instancia de proceso remota de Azure Machine Learning mediante Visual Studio Code, consulte [Conexión a una instancia de proceso de Azure Machine Learning en Visual Studio Code (versión preliminar)](how-to-set-up-vs-code-remote.md).
>
> Para obtener más información sobre las características de control de versiones de Visual Studio Code, consulte [Uso del control de versiones de Visual Studio Code](https://code.visualstudio.com/docs/editor/versioncontrol) y [Trabajo con GitHub en Visual Studio Code](https://code.visualstudio.com/docs/editor/github) (ambos en inglés).

## <a name="clone-git-repositories-into-your-workspace-file-system"></a>Clonación de repositorios de Git en el sistema de archivos del área de trabajo
Azure Machine Learning ofrece un sistema de archivos compartido para todos los usuarios del área de trabajo.
Para clonar un repositorio de Git en este recurso compartido de archivos, se recomienda crear una instancia de proceso y [abrir un terminal](how-to-access-terminal.md).
Una vez abierto el terminal, tendrá acceso a un cliente completo de Git y podrá clonar y trabajar con Git a través de la experiencia de CLI de Git.

Se recomienda clonar el repositorio en el directorio de los usuarios, de modo que otros no generen conflictos directamente en la rama en funcionamiento.

Puede clonar cualquier repositorio de Git en el que pueda autenticarse (GitHub, Azure Repos, BitBucket, etc.).

Para más información sobre la clonación, consulte la guía sobre [cómo usar la CLI de Git](https://guides.github.com/introduction/git-handbook/).

## <a name="authenticate-your-git-account-with-ssh"></a>Autenticación de la cuenta de Git con SSH
### <a name="generate-a-new-ssh-key"></a>Generación de una nueva clave SSH
1) [Abra la ventana de terminal](./how-to-access-terminal.md) en la pestaña Notebook de Azure Machine Learning.

2) Pegue el texto siguiente y sustituya la dirección de correo electrónico.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Se crea una clave SSH y se usa el correo electrónico proporcionado como etiqueta.

```
> Generating public/private rsa key pair.
```

3) Cuando se le pida que especifique un archivo en el que guardar la clave, presione Entrar. Con esta acción se acepta la ubicación predeterminada del archivo.

4) Compruebe que la ubicación predeterminada es "/home/azureuser/.ssh" y presione Entrar. De lo contrario, especifique la ubicación "/home/azureuser/.ssh".

> [!TIP]
> Asegúrese de que la clave SSH esté guardada en "/home/azureuser/.ssh". Este archivo se guarda en la instancia de proceso y solo es accesible para el propietario de esta.

```
> Enter a file in which to save the key (/home/azureuser/.ssh/id_rsa): [Press enter]
```

5) Cuando se le pida, escriba una frase de contraseña segura. Para mayor seguridad, se recomienda agregar una frase de contraseña a la clave SSH.

```
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

### <a name="add-the-public-key-to-git-account"></a>Adición de la clave pública a la cuenta de Git
1) En la ventana de terminal, copie el contenido del archivo de clave pública. Si ha cambiado el nombre de la clave, reemplace id_rsa.pub por el nombre del archivo de clave pública.

```bash
cat ~/.ssh/id_rsa.pub
```
> [!TIP]
> **Copiar y pegar en el terminal**
> * Windows: `Ctrl-Insert` para copiar y `Ctrl-Shift-v` o `Shift-Insert` para pegar.
> * Mac OS: `Cmd-c` para copiar y `Cmd-v` para pegar.
> * Es posible que Firefox o IE no admitan los permisos del Portapapeles correctamente.

2) Seleccione y copie la salida de la clave en el portapapeles.

+ [GitHub](https://docs.github.com/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

+ [GitLab](https://docs.gitlab.com/ee/ssh/#adding-an-ssh-key-to-your-gitlab-account)

+ [Azure DevOps](/azure/devops/repos/git/use-ssh-keys-to-authenticate#step-2--add-the-public-key-to-azure-devops-servicestfs) Comience en el **paso 2**.

+ [BitBucket](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/#SetupanSSHkey-ssh2). Comience en el **paso 4**.

### <a name="clone-the-git-repository-with-ssh"></a>Clonación del repositorio de Git con SSH

1) Copie la dirección URL de clonación de Git de SSH del repositorio de Git.

2) Pegue la dirección URL en el comando `git clone` siguiente para usar la dirección URL del repositorio de Git de SSH. Se parecerá a esta:

```bash
git clone git@example.com:GitUser/azureml-example.git
Cloning into 'azureml-example'...
```

Verá una respuesta como la siguiente:

```bash
The authenticity of host 'example.com (192.30.255.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.255.112' (RSA) to the list of known hosts.
```

Puede que SSH muestre la huella digital de SSH del servidor y le pida que la verifique. Deberá comprobar que la huella digital mostrada coincida con una de las huellas digitales de la página de claves públicas de SSH.

SSH muestra esta huella digital cuando se conecta a un host desconocido para protegerle de [ataques de tipo "Man in the Middle"](/previous-versions/windows/it-pro/windows-2000-server/cc959354(v=technet.10)). Una vez que acepte la huella digital del host, SSH no volverá a pedírsela a menos que la cambie.

3) Cuando se le pregunte si quiere continuar con la conexión, escriba `yes`. Git clonará el repositorio y configurará el origen remoto para conectarse con SSH en futuros comandos de Git.

## <a name="track-code-that-comes-from-git-repositories"></a>Seguimiento del código proveniente de repositorios de Git

Cuando se envía una ejecución de entrenamiento desde el SDK de Python o la CLI de Machine Learning, los archivos necesarios para entrenar el modelo se cargan en el área de trabajo. Si el comando `git` está disponible en el entorno de desarrollo, el proceso de carga lo usa para comprobar si los archivos se almacenan en un repositorio Git. Si es así, la información del repositorio de Git también se carga como parte de la ejecución de entrenamiento. Esta información se almacena en las siguientes propiedades para la ejecución de entrenamiento:

| Propiedad | Comando de Git que se usa para obtener el valor | Descripción |
| ----- | ----- | ----- |
| `azureml.git.repository_uri` | `git ls-remote --get-url` | El URI desde el que se clonó el repositorio. |
| `mlflow.source.git.repoURL` | `git ls-remote --get-url` | El URI desde el que se clonó el repositorio. |
| `azureml.git.branch` | `git symbolic-ref --short HEAD` | Rama activa cuando se envió la ejecución. |
| `mlflow.source.git.branch` | `git symbolic-ref --short HEAD` | Rama activa cuando se envió la ejecución. |
| `azureml.git.commit` | `git rev-parse HEAD` | Hash de confirmación del código enviado para la ejecución. |
| `mlflow.source.git.commit` | `git rev-parse HEAD` | Hash de confirmación del código enviado para la ejecución. |
| `azureml.git.dirty` | `git status --porcelain .` | `True`, si la rama/confirmación se ha modificado; de lo contrario, `false`. |

Esta información se envía para las ejecuciones que usan un estimador, una canalización de aprendizaje automático o una ejecución de scripts.

Si los archivos de entrenamiento no se encuentran en un repositorio de Git en el entorno de desarrollo, o el comando `git` no está disponible, no se realiza el seguimiento de ninguna información relacionada con Git.

> [!TIP]
> Para comprobar si el comando GIT está disponible en el entorno de desarrollo, abra una sesión del shell, un símbolo del sistema, PowerShell u otra interfaz de la línea de comandos y escriba el siguiente comando:
>
> ```
> git --version
> ```
>
> Si está instalado y, en la ruta de acceso, recibirá una respuesta similar a `git version 2.4.1`. Para obtener más información sobre la instalación de GIT en el entorno de desarrollo, consulte el [sitio web de GIT](https://git-scm.com/).

## <a name="view-the-logged-information"></a>Visualización de la información registrada

La información de Git se almacena en las propiedades para la ejecución de entrenamiento. Puede ver esta información mediante Azure Portal, el SDK de Python y la CLI. 

### <a name="azure-portal"></a>Portal de Azure

1. En el [portal de Studio](https://ml.azure.com), seleccione su área de trabajo.
1. Seleccione __Experimentos__ y, a continuación, seleccione uno de los experimentos.
1. Seleccione una de las ejecuciones de la columna __NÚMERO DE EJECUCIÓN__.
1. Seleccione __Resultados y registros__ y, a continuación, expanda las entradas __logs__ y __azureml__. Seleccione el vínculo que comienza por __###\_azure__.

La información registrada contiene texto similar al siguiente JSON:

```json
"properties": {
    "_azureml.ComputeTargetType": "batchai",
    "ContentSnapshotId": "5ca66406-cbac-4d7d-bc95-f5a51dd3e57e",
    "azureml.git.repository_uri": "git@github.com:azure/machinelearningnotebooks",
    "mlflow.source.git.repoURL": "git@github.com:azure/machinelearningnotebooks",
    "azureml.git.branch": "master",
    "mlflow.source.git.branch": "master",
    "azureml.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "mlflow.source.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "azureml.git.dirty": "True",
    "AzureML.DerivedImageName": "azureml/azureml_9d3568242c6bfef9631879915768deaf",
    "ProcessInfoFile": "azureml-logs/process_info.json",
    "ProcessStatusFile": "azureml-logs/process_status.json"
}
```

### <a name="python-sdk"></a>SDK de Python

Después de enviar una ejecución de entrenamiento, se devuelve un objeto [Run](/python/api/azureml-core/azureml.core.run%28class%29). El atributo `properties` de este objeto contiene la información de Git registrada. Por ejemplo, el código siguiente recupera el hash de confirmación:

```python
run.properties['azureml.git.commit']
```

### <a name="cli"></a>CLI

El comando de la CLI `az ml run` se puede usar para recuperar las propiedades de una ejecución. Por ejemplo, el comando siguiente devuelve las propiedades de la última ejecución en el experimento denominado `train-on-amlcompute`:

```azurecli-interactive
az ml run list -e train-on-amlcompute --last 1 -w myworkspace -g myresourcegroup --query '[].properties'
```

Para más información, consulte la documentación de referencia de [az ml run](/cli/azure/ext/azure-cli-ml/ml/run).

## <a name="next-steps"></a>Pasos siguientes

* [Uso de los destinos de proceso para el entrenamiento de modelos](how-to-set-up-training-targets.md)