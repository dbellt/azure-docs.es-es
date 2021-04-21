---
title: 'Procedimientos recomendados para operadores: Administración de imágenes de contenedor en Azure Kubernetes Services (AKS)'
description: Obtenga información sobre los procedimientos recomendados del operador de clústeres para saber cómo administrar y proteger las imágenes de contenedor en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/11/2021
ms.openlocfilehash: 998d8602b6aa0e71a04f75aff1c29551ba09c8a3
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105126"
---
# <a name="best-practices-for-container-image-management-and-security-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para la administración y la protección de las imágenes de contenedor en Azure Kubernetes Service (AKS)

La seguridad de los contenedores y de las imágenes de contenedor es una prioridad importante a la hora de desarrollar y ejecutar aplicaciones en Azure Kubernetes Service (AKS). Los contenedores que incluyen imágenes base obsoletas o entornos de ejecución de aplicaciones sin revisar suponen un riesgo para la seguridad y un posible vector de ataque. 

Minimice los riesgos mediante la integración y ejecución de herramientas de análisis y corrección en los contenedores en tiempo de compilación y ejecución. Cuanto antes se detecte la vulnerabilidad o la imagen base obsoleta, más seguro será el clúster. 

En este artículo, *"contenedores"* significa lo siguiente:
* Las imágenes de contenedor almacenadas en un registro de contenedor.
* Los contenedores en ejecución.

En este artículo se indica cómo proteger los contenedores de AKS. Aprenderá a:

> [!div class="checklist"]
> * Buscar y corregir vulnerabilidades de la imagen.
> * Desencadenar automáticamente y volver a implementar imágenes de contenedor cuando se actualiza una imagen base.

También puede leer los procedimientos recomendados para la [seguridad del clúster][best-practices-cluster-security] y la [seguridad de pod][best-practices-pod-security].

También puede usar la [seguridad de contenedor de Security Center][security-center-containers] para facilitar el análisis de los contenedores en busca de vulnerabilidades. La [integración de Azure Container Registry][security-center-acr] con Security Center ayuda a proteger las imágenes y el Registro de vulnerabilidades.

## <a name="secure-the-images-and-run-time"></a>Proteger las imágenes y el tiempo de ejecución

> **Guía de procedimientos recomendados** 
>
> Examine las imágenes de contenedor para detectar vulnerabilidades. Implemente solo imágenes validadas. Actualice periódicamente las imágenes base y el entorno de ejecución de la aplicación. Vuelva a implementar las cargas de trabajo en el clúster de AKS.

Al adoptar cargas de trabajo basadas en contenedores, querrá comprobar la seguridad de las imágenes y el entorno de ejecución usados para compilar sus propias aplicaciones. ¿Cómo evitar introducir vulnerabilidades de seguridad en las implementaciones? 
* Incluya un proceso para examinar imágenes de contenedor mediante herramientas como [Twistlock][twistlock] o [Aqua][aqua] en su flujo de trabajo de implementación.
* Permitir solo la implementación de imágenes comprobadas.

![Analizar y corregir imágenes de contenedor, validar e implementar](media/operator-best-practices-container-security/scan-container-images-simplified.png)

Por ejemplo, puede usar una integración continua y una canalización de implementación continua (CI/CD) para automatizar el análisis, la comprobación y la implementación de imágenes. Azure Container Registry incluye estas funcionalidades de análisis de vulnerabilidades.

## <a name="automatically-build-new-images-on-base-image-update"></a>Compilar automáticamente nuevas imágenes en la actualización de imagen base

> **Guía de procedimientos recomendados** 
>
> Si usa imágenes base para las imágenes de la aplicación, use la automatización para crear nuevas imágenes cuando actualice la imagen base. Puesto que las imágenes base actualizadas suelen incluir correcciones de seguridad, actualice todas las imágenes de contenedor de aplicación de nivel inferior.

Cada vez que se actualice una imagen base, también se deben actualizar las imágenes de contenedor de nivel inferior. Integre este proceso de compilación en las canalizaciones de validación e implementación, como [Azure Pipelines][azure-pipelines] o Jenkins. Estas canalizaciones aseguran que sus aplicaciones siguen ejecutándose en las imágenes base actualizadas. Una vez que se validan las imágenes de contenedor de aplicación, se pueden actualizar las implementaciones de AKS para ejecutar las imágenes más recientes y seguras.

Azure Container Registry Tasks también puede actualizar automáticamente las imágenes de contenedor cuando se actualiza la imagen base. Gracias a esta característica, puede crear unas cuantas imágenes base y mantenerlas actualizadas con correcciones de errores y seguridad.

Para más información sobre las actualizaciones de imagen base, consulte el artículo de [Automatización de compilaciones de imágenes al actualizarse una imagen base con Azure Container Registry Tasks][acr-base-image-update].

## <a name="next-steps"></a>Pasos siguientes

En este artículo se indica cómo proteger los contenedores. Para implementar algunas de estas áreas, consulte los artículos siguientes:

* [Automatización de compilaciones de imágenes al actualizarse una imagen base con Azure Container Registry Tasks][acr-base-image-update]

<!-- EXTERNAL LINKS -->
[azure-pipelines]: /azure/devops/pipelines/
[twistlock]: https://www.twistlock.com/
[aqua]: https://www.aquasec.com/

<!-- INTERNAL LINKS -->
[best-practices-cluster-security]: operator-best-practices-cluster-security.md
[best-practices-pod-security]: developer-best-practices-pod-security.md
[acr-base-image-update]: ../container-registry/container-registry-tutorial-base-image-update.md
[security-center-containers]: ../security-center/container-security.md
[security-center-acr]: ../security-center/defender-for-container-registries-introduction.md