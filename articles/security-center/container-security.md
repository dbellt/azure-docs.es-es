---
title: Seguridad de los contenedores con Azure Security Center y Azure Defender
description: Obtenga información sobre las características de seguridad de contenedores de Azure Security Center.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 04/06/2021
ms.author: memildin
ms.openlocfilehash: 9fddb27ee6a1139fa8b07c6c19dd4fdf1a20096e
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029151"
---
# <a name="container-security-in-security-center"></a>Seguridad de los contenedores en Security Center

Azure Security Center es la solución nativa de Azure para proteger los contenedores.

Security Center puede proteger los siguientes tipos de recursos de contenedor:

| Tipo de recurso | Protecciones ofrecidas por Security Center |
|:--------------------:|-----------|
| ![Servicio Kubernetes](./media/security-center-virtual-machine-recommendations/icon-kubernetes-service-rec.png)<br>**Clústeres de Kubernetes** | Evaluación continua de los clústeres para proporcionar visibilidad sobre las configuraciones incorrectas y las directrices que le ayudarán a mitigar las amenazas identificadas. Más información sobre la [protección del entorno mediante recomendaciones de seguridad](#environment-hardening).<br><br>Protección contra amenazas para clústeres y nodos Linux. [Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md) proporciona alertas de actividades sospechosas. Este plan de Azure Defender protegerá los clústeres de Kubernetes tanto si se hospedan en Azure Kubernetes Service (AKS), localmente o en otros proveedores de nube. Clústeres. <br>Más información sobre la [protección en tiempo de ejecución de los clústeres y nodos de Kubernetes](#run-time-protection-for-kubernetes-nodes-and-clusters).|
| ![Host de contenedor](./media/security-center-virtual-machine-recommendations/icon-container-host-rec.png)<br>**Hosts de contenedor**<br>(Máquinas virtuales que ejecutan Docker) | Evaluación continua de los entornos de Docker para proporcionar visibilidad de los errores de configuración e instrucciones que le ayudarán a mitigar las amenazas identificadas con la funcionalidad opcional [Azure Defender para servidores](defender-for-servers-introduction.md).<br>Más información sobre la [protección del entorno mediante recomendaciones de seguridad](#environment-hardening).|
| ![Registro de contenedor](./media/security-center-virtual-machine-recommendations/icon-container-registry-rec.png)<br>**Registros de Azure Container Registry (ACR)** | Herramientas de evaluación y administración de vulnerabilidades para las imágenes de los registros de ACR basados en Azure Resource Manager con la funcionalidad opcional [Azure Defender para registros de contenedor](defender-for-container-registries-introduction.md).<br>Más información sobre el [examen de las imágenes de contenedor en busca de vulnerabilidades](#vulnerability-management---scanning-container-images). |
|||

En este artículo se describe cómo puede usar Security Center, junto con los planes de Azure Defender opcionales para los registros de contenedor, los servidores y Kubernetes, a fin de mejorar, supervisar y mantener la seguridad de los contenedores y sus aplicaciones.

Aprenderá a usar Security Center para que le ayude con estos aspectos básicos de la seguridad de los contenedores:

- [Administración de vulnerabilidades: análisis de imágenes de contenedor](#vulnerability-management---scanning-container-images)
- [Protección del entorno](#environment-hardening)
- [Protección en tiempo de ejecución de los clústeres y nodos de Kubernetes](#run-time-protection-for-kubernetes-nodes-and-clusters)

En la captura de pantalla siguiente se muestran la página del inventario de recursos y los distintos tipos de recursos de contenedor protegidos por Security Center.

:::image type="content" source="./media/container-security/inventory-container-resources.png" alt-text="Recursos relacionados con contenedores en la página del inventario de recursos de Security Center" lightbox="./media/container-security/inventory-container-resources.png":::

## <a name="vulnerability-management---scanning-container-images"></a>Administración de vulnerabilidades: análisis de imágenes de contenedor

Para supervisar las imágenes en los registros de contenedor de Azure basados en Azure Resource Manager, habilite [Azure Defender para registros de contenedor](defender-for-container-registries-introduction.md). Security Center examina las imágenes extraídas durante los últimos 30 días, insertadas en el registro o importadas. El detector integrado lo proporciona el proveedor de análisis de vulnerabilidades líder del sector, Qualys.

Si Qualys o Security Center detectan incidencias, recibirá una notificación en el [panel de Azure Defender](azure-defender-dashboard.md). Por cada vulnerabilidad, Security Center proporciona recomendaciones útiles, junto con una clasificación de gravedad e información sobre cómo corregir el problema. Para más información sobre las recomendaciones de Security Center para los contenedores, consulte la [lista de recomendaciones de referencia](recommendations-reference.md#recs-compute).

Security Center filtra y clasifica los resultados del análisis. Cuando una imagen es correcta, Security Center la marca como tal. Security Center solo genera recomendaciones de seguridad para las imágenes que tienen incidencias sin resolver. Al enviar notificaciones solo cuando hay problemas, Security Center reduce las alertas informativas no deseadas.

## <a name="environment-hardening"></a>Protección del entorno

### <a name="continuous-monitoring-of-your-docker-configuration"></a>Supervisión continua de la configuración de Docker

Azure Security Center identifica contenedores no administrados y que están hospedados en VM de IaaS Linux u otras máquinas de Linux que ejecutan contenedores de Docker. Security Center evalúa continuamente las configuraciones de estos contenedores. A continuación, las compara con el [Banco de prueba para Docker del Centro de seguridad de Internet (CIS)](https://www.cisecurity.org/benchmark/docker/).

Security Center incluye todo el conjunto de reglas del banco de prueba de Docker de CIS y le avisa si los contenedores no cumplen ninguno de los controles. Cuando encuentra configuraciones inactivas, Security Center genera recomendaciones de seguridad. Use la **página de recomendaciones** de Azure Security Center para ver las recomendaciones y corregir los problemas. Las comprobaciones del banco de pruebas de CIS no se ejecutan en instancias administradas por AKS o en VM administradas por Databricks.

Para obtener información sobre las recomendaciones pertinentes de Security Center que pueden aparecer para esta característica, consulte la [sección de proceso](recommendations-reference.md#recs-compute) de la tabla de referencia de recomendaciones.

Cuando explora los problemas de seguridad de una máquina virtual, Security Center proporciona información adicional sobre los contenedores de la máquina. Esta información incluye la versión de Docker y el número de imágenes que se ejecutan en el host. 

Para supervisar contenedores no administrados hospedados en máquinas virtuales Linux de IaaS, habilite la funcionalidad opcional [Azure Defender para servidores](defender-for-servers-introduction.md).


### <a name="continuous-monitoring-of-your-kubernetes-clusters"></a>Supervisión continua de los clústeres de Kubernetes
Security Center funciona junto con Azure Kubernetes Service (AKS), el servicio de orquestación de contenedores administrados de Microsoft que le permite desarrollar, implementar y administrar las aplicaciones en contenedores.

AKS proporciona controles de seguridad y visibilidad sobre la postura de seguridad de los clústeres. Security Center usa estas características para supervisar constantemente la configuración de los clústeres de AKS y generar recomendaciones de seguridad que se alineen con los estándares del sector.

Este es un diagrama general de la interacción entre Azure Security Center, Azure Kubernetes Service y Azure Policy:

:::image type="content" source="./media/defender-for-kubernetes-intro/kubernetes-service-security-center-integration-detailed.png" alt-text="Arquitectura de alto nivel de la interacción entre Azure Security Center, Azure Container Service y Azure Policy" lightbox="./media/defender-for-kubernetes-intro/kubernetes-service-security-center-integration-detailed.png":::

Puede ver que los elementos recibidos y analizados por Security Center incluyen:

- registros de auditoría del servidor de API,
- eventos de seguridad sin procesar del agente de Log Analytics,

    > [!NOTE]
    > Actualmente no se admite la instalación del agente de Log Analytics en los clústeres de Azure Kubernetes Service que se ejecutan en conjuntos de escalado de máquinas virtuales.

- información de configuración del clúster de AKS y
- configuración de la carga de trabajo en Azure Policy (mediante el complemento de **Azure Policy para Kubernetes**).

Para obtener información sobre las recomendaciones pertinentes de Security Center que pueden aparecer para esta característica, consulte la [sección de proceso](recommendations-reference.md#recs-compute) de la tabla de referencia de recomendaciones.


###  <a name="workload-protection-best-practices-using-kubernetes-admission-control"></a>Procedimientos recomendados de protección de cargas de trabajo con el control de admisión de Kubernetes

Para obtener un conjunto de recomendaciones para proteger las cargas de trabajo de los contenedores Kubernetes, instale el **complemento de Azure Policy para Kubernetes**. También puede implementar automáticamente este complemento tal y como se explica en [Configuración del aprovisionamiento automático de agentes y extensiones desde Azure Security Center](security-center-enable-data-collection.md#auto-provision-mma). Cuando el aprovisionamiento automático del complemento esté establecido en "activado", la extensión se habilitará de forma predeterminada en todos los clústeres existentes y futuros (que cumplan los requisitos de instalación del complemento).

Como se explica en [esta página de Azure Policy para Kubernetes](../governance/policy/concepts/policy-for-kubernetes.md), el complemento extiende el webhook de controlador de admisión de código abierto [Gatekeeper v3](https://github.com/open-policy-agent/gatekeeper)  de  [Open Policy Agent](https://www.openpolicyagent.org/). Los controladores de admisión de Kubernetes son complementos que exigen el uso de los clústeres. El complemento se registra como un webhook en el control de admisión de Kubernetes y permite aplicar las implementaciones a escala y las medidas de seguridad en los clústeres de una manera centralizada y coherente. 

Con el complemento en el clúster de AKS, todas las solicitudes al servidor de la API de Kubernetes se supervisarán según el conjunto predefinido de procedimientos recomendados antes de que se guarden en el clúster. Después, puede realizar la configurar para **aplicar** los procedimientos recomendados y exigirlos para futuras cargas de trabajo. 

Por ejemplo, puede exigir que no se creen los contenedores con privilegios y que se bloqueen las solicitudes futuras para este fin.

Obtenga más información en [Protección de las cargas de trabajo de Kubernetes](kubernetes-workload-protections.md).


## <a name="run-time-protection-for-kubernetes-nodes-and-clusters"></a>Protección en tiempo de ejecución de los clústeres y nodos de Kubernetes

[!INCLUDE [AKS in ASC threat protection](../../includes/security-center-azure-kubernetes-threat-protection.md)]



## <a name="next-steps"></a>Pasos siguientes

En esta información general, ha aprendido los elementos básicos de la seguridad de los contenedores en Azure Security Center. Para obtener material relacionado, vea lo siguiente:

- [Introducción a Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md)
- [Introducción a Azure Defender para registros de contenedor](defender-for-container-registries-introduction.md)