---
title: Secretos de autenticación en ejecuciones de entrenamiento
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo pasar secretos a ejecuciones de entrenamiento de manera segura mediante la instancia de Azure Key Vault para el área de trabajo.
services: machine-learning
author: rastala
ms.author: roastala
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 03/09/2020
ms.topic: how-to
ms.openlocfilehash: b934a5b5d7781465d01b0e16927bf213f9fa23df
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107897043"
---
# <a name="use-authentication-credential-secrets-in-azure-machine-learning-training-runs"></a>Uso de secretos de credenciales de autenticación en ejecuciones de entrenamiento de Azure Machine Learning


En este artículo, aprenderá a usar secretos en las ejecuciones de entrenamiento de forma segura. Los datos de autenticación (por ejemplo, el nombre de usuario y la contraseña) son secretos. Por ejemplo, si se conecta a una base de datos externa para consultar datos de entrenamiento, deberá pasar el nombre de usuario y la contraseña al contexto de ejecución remoto. La codificación de estos valores en los scripts de entrenamiento en texto no cifrado no es segura, ya que expondría el secreto. 

En su lugar, el área de trabajo de Azure Machine Learning tiene un recurso asociado llamado [Azure Key Vault](../key-vault/general/overview.md). Use esta instancia de Key Vault para pasar secretos a ejecuciones remotas de forma segura mediante un conjunto de API en el SDK de Python de Azure Machine Learning.

El flujo estándar para el uso de secretos es:
 1. En el equipo local, inicie sesión en Azure y conéctese al área de trabajo.
 2. En el equipo local, establezca un secreto en el almacén de claves del área de trabajo.
 3. Envíe una ejecución remota.
 4. Dentro de la ejecución remota, obtenga el secreto de Key Vault y úselo.

## <a name="set-secrets"></a>Establecimiento de secretos

En Azure Machine Learning, la clase [Keyvault](/python/api/azureml-core/azureml.core.keyvault.keyvault) contiene métodos para establecer secretos. En la sesión de Python local, obtenga primero una referencia al área de trabajo de Key Vault y, a continuación, use el método [`set_secret()`](/python/api/azureml-core/azureml.core.keyvault.keyvault#set-secret-name--value-) para establecer un secreto por nombre y valor. El método __set_secret__ actualiza el valor del secreto si el nombre ya existe.

```python
from azureml.core import Workspace
from azureml.core import Keyvault
import os


ws = Workspace.from_config()
my_secret = os.environ.get("MY_SECRET")
keyvault = ws.get_default_keyvault()
keyvault.set_secret(name="mysecret", value = my_secret)
```

No ponga el valor del secreto en código Python porque no es seguro almacenarlo en un archivo como texto no cifrado. En su lugar, obtenga el valor del secreto de una variable de entorno, por ejemplo, el secreto de compilación de Azure DevOps, o de la entrada interactiva del usuario.

Puede enumerar los nombres de secreto mediante el método [`list_secrets()`](/python/api/azureml-core/azureml.core.keyvault.keyvault#list-secrets--). También hay una versión de lote, [set_secrets ()](/python/api/azureml-core/azureml.core.keyvault.keyvault#set-secrets-secrets-batch-), que le permite establecer varios secretos a la vez.

## <a name="get-secrets"></a>Obtención de secretos

En el código local, puede usar el método [`get_secret()`](/python/api/azureml-core/azureml.core.keyvault.keyvault#get-secret-name-) para obtener el valor del secreto por nombre.

En el caso de las ejecuciones enviadas con [`Experiment.submit`](/python/api/azureml-core/azureml.core.experiment.experiment#submit-config--tags-none----kwargs-), utilice el método [`get_secret()`](/python/api/azureml-core/azureml.core.run.run#get-secret-name-) con la clase [`Run`](/python/api/azureml-core/azureml.core.run%28class%29). Como la ejecución enviada conoce su área de trabajo, este método establece un acceso directo a la creación de instancias del área de trabajo y devuelve el valor del secreto directamente.

```python
# Code in submitted run
from azureml.core import Experiment, Run

run = Run.get_context()
secret_value = run.get_secret(name="mysecret")
```

Tenga cuidado de no exponer el valor del secreto escribiéndolo o imprimiéndolo.

También hay una versión de lote, [get_secrets ()](/python/api/azureml-core/azureml.core.run.run#get-secrets-secrets-), que permite acceder a varios secretos a la vez.

## <a name="next-steps"></a>Pasos siguientes

 * [Visualización de cuaderno de ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/manage-azureml-service/authentication-in-azureml/authentication-in-azureml.ipynb)
 * [Información sobre seguridad de empresa con Azure Machine Learning](concept-enterprise-security.md)