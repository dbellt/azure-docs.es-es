---
title: Configuración de la autenticación para los modelos implementados como servicios web
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo configurar la autenticación para los modelos de Machine Learning implementados en los servicios web en Azure Machine Learning.
services: machine-learning
author: cjgronlund
ms.author: cgronlun
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 11/06/2020
ms.topic: how-to
ms.openlocfilehash: c315949adbad946fef57213f38f1cb9d7d45f668
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107898088"
---
# <a name="configure-authentication-for-models-deployed-as-web-services"></a>Configuración de la autenticación para los modelos implementados como servicios web

Azure Machine Learning le permite implementar los modelos de Machine Learning entrenados como servicios web. En este artículo, aprenderá a configurar la autenticación para estas implementaciones.

Las implementaciones de modelos creadas por Azure Machine Learning se pueden configurar para usar uno de dos métodos de autenticación:

* **Basado en claves**: se usa una clave estática para autenticarse en el servicio web.
* **Basado en tokens**: Se debe obtener un token temporal del área de trabajo de Azure Machine Learning (mediante Azure Active Directory) y usar para autenticarse en el servicio web. Este token expira tras un período de tiempo y debe actualizarse para seguir trabajando con el servicio web.

    > [!NOTE]
    > La autenticación basada en el token solo está disponible al implementarla en Azure Kubernetes Service.

## <a name="key-based-authentication"></a>Autenticación basada en claves

Los servicios web implementados en Azure Kubernetes Service (AKS) tienen la autenticación basada en claves *habilitada* de manera predeterminada.

Los servicios implementados por Azure Container Instances (ACI) tienen la autenticación basada en claves *deshabilitada* de manera predeterminada, pero puede habilitarla estableciendo `auth_enabled=True` al crear el servicio web de ACI. El código siguiente es un ejemplo de creación de una configuración de implementación de ACI con la autenticación basada en claves habilitada.

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(cpu_cores = 1,
                                                memory_gb = 1,
                                                auth_enabled=True)
```

Después, puede usar la configuración de ACI personalizada en la implementación mediante la clase `Model`.

```python
from azureml.core.model import Model, InferenceConfig


inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
aci_service = Model.deploy(workspace=ws,
                       name="aci_service_sample",
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aci_config)
aci_service.wait_for_deployment(True)
```

Para capturar las claves de autenticación, use `aci_service.get_keys()`. Para volver a generar una clave, utilice la función `regen_key()` y pase **Primary** o **Secondary**.

```python
aci_service.regen_key("Primary")
# or
aci_service.regen_key("Secondary")
```

## <a name="token-based-authentication"></a>Autenticación basada en tokens

Cuando se habilita la autenticación por tokens para un servicio web, los usuarios deben presentar un token JSON Web Token de Azure Machine Learning al servicio web para tener acceso a él. El token expira después de un intervalo de tiempo especificado y debe actualizarse para seguir realizando llamadas.

* La autenticación por tokens está **deshabilitada de manera predeterminada** cuando se implementa en Azure Kubernetes Service.
* La autenticación por tokens **no se admite** cuando se implementa en Azure Container Instances.
* La autenticación por tokens **no se puede usar al mismo tiempo que la autenticación basada en claves**.

Para controlar la autenticación por tokens, use el parámetro `token_auth_enabled` cuando cree o actualice una implementación:

```python
from azureml.core.webservice import AksWebservice
from azureml.core.model import Model, InferenceConfig

# Create the config
aks_config = AksWebservice.deploy_configuration()

#  Enable token auth and disable (key) auth on the webservice
aks_config = AksWebservice.deploy_configuration(token_auth_enabled=True, auth_enabled=False)

aks_service_name ='aks-service-1'

# deploy the model
aks_service = Model.deploy(workspace=ws,
                           name=aks_service_name,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target)

aks_service.wait_for_deployment(show_output = True)
```

Si la autenticación por tokens está habilitada, puede usar el método `get_token` para recuperar un token JSON Web Token (JWT) y la hora de expiración del token:

> [!TIP]
> Si usa una entidad de servicio para obtener el token y quiere que tenga el acceso mínimo necesario para recuperar un token, asígnela al rol de **lector** para el área de trabajo.

```python
token, refresh_by = aks_service.get_token()
print(token)
```

> [!IMPORTANT]
> Tendrá que solicitar un nuevo token después de la hora `refresh_by` del token. Si necesita actualizar tokens fuera del SDK de Python, una opción es usar la API REST con la autenticación de entidad de servicio para hacer periódicamente la llamada `service.get_token()`, como se explicó anteriormente.
>
> Se recomienda crear el área de trabajo de Azure Machine Learning en la misma región que el clúster de Azure Kubernetes Service.
>
> Para autenticarse con un token, el servicio web hará una llamada a la región en la que se crea el área de trabajo de Azure Machine Learning. Si la región del área de trabajo no está disponible, no se podrá capturar un token para el servicio web, incluso si el clúster está en una región distinta del área de trabajo. El resultado es que la Autenticación de Azure AD no está disponible hasta que la región del área de trabajo vuelva a estar disponible.
>
> Además, cuanto mayor sea la distancia entre la región del clúster y la región del área de trabajo, más tiempo tardará la captura de un token.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la autenticación en un modelo implementado, vea [Creación de un cliente para un modelo implementado como servicio web](how-to-consume-web-service.md).