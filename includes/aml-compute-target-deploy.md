---
title: archivo de inclusión
description: archivo de inclusión
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 12/11/2020
ms.openlocfilehash: c03550b8f0b6c146caa5fb33f06eba6671fa313b
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107950284"
---
El destino de proceso que use para hospedar el modelo afectará al costo y la disponibilidad del punto de conexión implementado. Use esta tabla para elegir un destino de proceso adecuado.

| Destino de proceso | Se usa para | Compatibilidad con GPU | Compatibilidad con FPGA | Descripción |
| ----- | ----- | ----- | ----- | ----- |
| [Servicio&nbsp;web&nbsp;local](../articles/machine-learning/how-to-deploy-local-container-notebook-vm.md) | Pruebas y depuración | &nbsp; | &nbsp; | Se usa para pruebas limitadas y solución de problemas. La aceleración de hardware depende del uso de bibliotecas en el sistema local.
| [Azure Kubernetes Service (AKS)](../articles/machine-learning/how-to-deploy-azure-kubernetes-service.md) | Inferencia en tiempo real |  [Sí](../articles/machine-learning/how-to-deploy-with-triton.md) (implementación del servicio web) | [Sí](../articles/machine-learning/how-to-deploy-fpga-web-service.md)   |Se usa para las implementaciones de producción a gran escala. Proporciona un tiempo de respuesta rápido y el escalado automático del servicio implementado. No se admite el escalado automático de clúster a través del SDK de Azure Machine Learning. Para cambiar los nodos del clúster de AKS, use la interfaz de usuario para el clúster de AKS en Azure Portal. <br/><br/> Se admite en el diseñador. |
| [Azure Container Instances](../articles/machine-learning/how-to-deploy-azure-container-instance.md) | Pruebas o desarrollo | &nbsp;  | &nbsp; | Se usa para cargas de trabajo basadas en CPU a pequeña escala que requieren menos de 48 GB de RAM. <br/><br/> Se admite en el diseñador. |
| [Clústeres de proceso de Azure Machine Learning](../articles/machine-learning/tutorial-pipeline-batch-scoring-classification.md) | Inferencia de lote&nbsp; | [Sí](../articles/machine-learning/tutorial-pipeline-batch-scoring-classification.md) (canalización de aprendizaje automático) | &nbsp;  | Ejecute la puntuación por lotes en un proceso sin servidor. Admite máquinas virtuales de prioridad normal y baja. No se admite la inferencia en tiempo real.|

> [!NOTE]
> Aunque los destinos de proceso locales y los clústeres de proceso de Azure Machine Learning, admiten GPU para el entrenamiento y la experimentación, el uso de la GPU para la inferencia _cuando se implementa como un servicio web_ solo se admite en AKS.
>
> El uso de una GPU para la inferencia _cuando se realiza la puntuación con una canalización de aprendizaje automático_ solo se admite en el proceso de Azure Machine Learning.
> 
> Al elegir una SKU de clúster, primero realice el escalado vertical y, luego, el horizontal. Comience con una máquina que tenga el 150 % de la RAM que necesita el modelo, genere un perfil del resultado y busque una máquina que tenga el rendimiento que necesita. Cuando haya aprendido a hacer esto, aumente el número de máquinas para que se adapten a su necesidad de inferencia simultánea.

> [!NOTE]
> * Las instancias de contenedor solo son adecuadas para modelos pequeños con un tamaño inferior a 1 GB.
> * Use clústeres de AKS de nodo único para el desarrollo y las pruebas de modelos más grandes.