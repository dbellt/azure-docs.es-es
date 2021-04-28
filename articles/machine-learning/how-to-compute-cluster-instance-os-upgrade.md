---
title: Actualizar el sistema operativo del host para el clúster de proceso y la instancia
titleSuffix: Azure Machine Learning
description: Actualice el sistema operativo del host para el clúster de proceso y la instancia de proceso de Ubuntu 16,04 LTS a 18,04 LTS.
services: machine-learning
author: nishankgu
ms.author: nigup
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 03/03/2021
ms.topic: how-to
ms.openlocfilehash: 863b10f0ff41c1e28a59357f07be983715adbb55
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108131429"
---
# <a name="upgrade-compute-instance-and-compute-cluster-host-os"></a>Actualización de la instancia de proceso y del sistema operativo del host de clúster de proceso

El __clúster de proceso__ y la __instancia de proceso__ de Azure Machine Learning son una infraestructura de proceso administrada. Como servicio administrado, Microsoft administra el sistema operativo del host y los paquetes y las versiones de software que están instalados.

El sistema operativo de host para el clúster de proceso y la instancia de proceso ha sido Ubuntu 16,04 LTS. El **30 de abril de 2021**, Ubuntu será compatible con 16,04. A partir del __15 de marzo de 2021__, Microsoft actualizará automáticamente el sistema operativo del host a Ubuntu 18,04 lts. La actualización a 18,04 garantizará actualizaciones de seguridad continuas y soporte técnico de la comunidad de Ubuntu. Esta actualización se implementará en todas las regiones de Azure y estará disponible en todas las regiones el __09 de abril de 2021__. Para más información sobre la compatibilidad con Ubuntu de 16,04, consulte el blog de la [versión de Ubuntu](https://wiki.ubuntu.com/Releases).

> [!TIP]
> * El sistema operativo del host no es la versión del sistema operativo que se puede especificar para un [entorno](how-to-use-environments.md) al entrenar o implementar un modelo. Los entornos se ejecutan dentro de Docker. Docker se ejecuta en el sistema operativo del host.
> * Si actualmente usa entornos basados en Ubuntu 16,04 para el entrenamiento o la implementación, Microsoft recomienda que cambie al uso de imágenes basadas en Ubuntu 18,04. Para obtener más información, consulte [uso de entornos](how-to-use-environments.md) y el [repositorio de contenedores de Azure Machine Learning](https://github.com/Azure/AzureML-Containers/tree/master/base).
> * Cuando se usa una instancia de proceso de Azure Machine Learning basada en Ubuntu 18,04, la versión de Python predeterminada es _Python 3,8_.
## <a name="creating-new-resources"></a>Crear nuevos recursos

El clúster de proceso o las instancias de proceso creadas después del __09 de abril de 2021__ usan Ubuntu 18,04 LTS como el sistema operativo del host de forma predeterminada. No es posible seleccionar sistema operativo de host diferente.

## <a name="upgrade-existing-resources"></a>Actualización de recursos existentes

Si tiene clústeres de cálculo o instancias de proceso creados antes del __15 de marzo de 2021__, debe tomar medidas para actualizar el sistema operativo del host a Ubuntu 18,04. En función de la región a la que acceda Azure Machine Learning, se recomienda realizar estas acciones después del __09 de abril de 2021__ para asegurarse de que los cambios se han implementado en todas las regiones:

* __Clúster de proceso de Azure Machine Learning__:

    * Si el clúster está configurado con __nodos min = 0__, se actualizará automáticamente cuando se completen todos los trabajos y se reduzca a cero nodos.
    * Si los __nodos min > 0__, cambie temporalmente el número mínimo de nodos a cero y permita que el clúster se reduzca a cero nodos.

    Para obtener más información sobre cómo cambiar los nodos mínimos, vea el comando CLI de Azure [az ml computetarget update amlcompute](/cli/azure/ml/computetarget/update#az_ml_computetarget_update_amlcompute) o la referencia del SDK de [AmlCompute.update()](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute#update-min-nodes-none--max-nodes-none--idle-seconds-before-scaledown-none-).

* __Instancia de proceso de Azure Machine Learning__: cree una instancia de proceso nueva (que usará Ubuntu 18,04) y elimine la instancia anterior.

    * Cualquier cuaderno almacenado en el área de trabajo de archivos compartidos, almacenes de datos, de conjuntos de datos será accesible desde la nueva instancia de proceso.
    * Si ha creado entornos Conda personalizados, puede exportar esos entornos desde la instancia existente e importarlos en la nueva instancia. Para obtener información sobre la exportación e importación de Conda, consulte la [documentación de CONDA](https://docs.conda.io/) en docs.conda.io.

    Para obtener más información, consulte los artículos [Qué es la instancia de proceso](concept-compute-instance.md) y [crear y administrar una instancia de proceso de Azure Machine Learning](how-to-create-manage-compute-instance.md)

## <a name="check-host-os-version"></a>Comprobar la versión del sistema operativo del host

Para obtener información sobre cómo comprobar la versión del sistema operativo del host, consulte la página wiki de la comunidad de Ubuntu en [comprobación de la versión de Ubuntu](https://help.ubuntu.com/community/CheckingYourUbuntuVersion).

> [!TIP]
> Para usar el comando `lsb_release -a` desde la wiki, puede [usar una sesión de terminal en una instancia de proceso](how-to-access-terminal.md).
## <a name="next-steps"></a>Pasos siguientes

Si tiene comentarios o preguntas, póngase en contacto con nosotros en [ubuntu18azureml@service.microsoft.com](mailto:ubuntu18azureml@service.microsoft.com).