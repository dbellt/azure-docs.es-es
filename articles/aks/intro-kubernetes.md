---
title: Introducción a Azure Kubernetes Service
description: Aprenda las características y ventajas de Azure Kubernetes Service para implementar y administrar aplicaciones basadas en contenedor en Azure.
services: container-service
ms.topic: overview
ms.date: 02/24/2021
ms.custom: mvc
ms.openlocfilehash: 315230a19967e4006e378b836b0ea70f6b5c19b9
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110094301"
---
# <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Azure Kubernetes Service (AKS) simplifica la implementación de un clúster de Kubernetes administrado en Azure, ya que descarga la sobrecarga operativa en Azure. Al ser un servicio de Kubernetes hospedado, Azure controla tareas críticas como la supervisión del estado y el mantenimiento. Como Azure administra los maestros de Kubernetes, el usuario solo administra y mantiene los nodos de agente. Por consiguiente, AKS es gratuito; solo se paga por los nodos de agente en los clústeres, no por los maestros.  

Puede crear un clúster de AKS mediante:
* [La CLI de Azure](kubernetes-walkthrough.md)
* [Portal de Azure](kubernetes-walkthrough-portal.md)
* [Azure PowerShell](kubernetes-walkthrough-powershell.md)
* El uso de opciones de implementación controladas por plantillas, como las [plantillas de Azure Resource Manager](kubernetes-walkthrough-rm-template.md) y Terraform 

Al implementar un clúster de AKS, el maestro y todos los nodos de Kubernetes se implementan y configuran automáticamente. Las redes avanzadas, la integración de Azure Active Directory (Azure AD), la supervisión y otras características se pueden configurar durante el proceso de implementación. 

Para más información sobre los conceptos básicos de Kubernetes, consulte [Conceptos básicos de Kubernetes para AKS][concepts-clusters-workloads].

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]
> AKS también admite contenedores de Windows Server.

## <a name="access-security-and-monitoring"></a>Acceso, seguridad y supervisión

Para mejorar la seguridad y administración, AKS permite realizar la integración con Azure AD para:
* Utilizar el control de acceso basado en roles de Kubernetes (RBAC de Kubernetes). 
* Supervisar el mantenimiento del clúster y los recursos.

### <a name="identity-and-security-management"></a>Administración de identidades y seguridad

#### <a name="kubernetes-rbac"></a>RBAC de Kubernetes

Para limitar el acceso a los recursos de clúster, AKS admite [RBAC de Kubernetes][kubernetes-rbac]. RBAC de Kubernetes controla el acceso a los recursos y espacios de nombres de Kubernetes, y los permisos de los mismos.  

#### <a name="azure-ad"></a>Azure AD

Puede configurar un clúster de AKS para que se integre con Azure AD. Con la integración con Azure AD, puede configurar el acceso a Kubernetes en función de la identidad y la pertenencia a grupos existentes. Puede proporcionar una experiencia de inicio de sesión integrada a sus usuarios y grupos de Azure AD existentes, así como acceso a recursos de AKS.  

Para más información sobre la identidad, consulte [Opciones de acceso e identidad para AKS][concepts-identity].

Para proteger sus clústeres de AKS, consulte [Integrate Azure Active Directory with AKS][aks-aad] (Integración de Azure Active Directory con AKS).

### <a name="integrated-logging-and-monitoring"></a>Supervisión y registro integrados

Azure Monitor para el estado de los contenedores recopila métricas de procesador y memoria de contenedores, nodos y controladores dentro del clúster de AKS y las aplicaciones implementadas. Puede revisar tanto los registros de contenedores como los [registros maestros de Kubernetes][aks-master-logs], que:
* Se almacenan en un área de trabajo de Azure Log Analytics.
* Están disponible mediante Azure Portal, la CLI de Azure o un punto de conexión de REST.

Para más información, consulte [Supervisión del mantenimiento de contenedores de Azure Kubernetes Service][container-health].

## <a name="clusters-and-nodes"></a>Clústeres y nodos

Los nodos de AKS se ejecutan en máquinas virtuales de Azure (VM). Con los nodos de AKS, puede conectar el almacenamiento a nodos y pods, actualizar los componentes de clúster y usar GPU. AKS admite clústeres de Kubernetes que ejecutan varios grupos de nodos, con el fin de que se puedan usar sistemas operativos mixtos y contenedores de Windows Server.  

Para más información sobre las funcionalidades de los clústeres, nodos y grupos de nodos de Kubernetes, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][concepts-clusters-workloads].

### <a name="cluster-node-and-pod-scaling"></a>Escalado de pods y nodos de clúster

A medida que cambia la demanda de recursos, el número de pods o nodos de clúster que ejecutan sus servicios se escala vertical u horizontalmente. Puede ajustar el escalador automático del pod horizontal o el escalador automático del clúster en función de las demandas u y ejecutar solo los recursos necesarios.

Para más información, consulte [Escalado de un clúster de Azure Kubernetes Service (AKS)][aks-scale].

### <a name="cluster-node-upgrades"></a>Actualizaciones de nodos de clúster

AKS ofrece varias versiones de Kubernetes. A medida que hay nuevas versiones disponibles en AKS, puede actualizar el clúster mediante Azure Portal o la CLI de Azure. Durante el proceso de actualización, los nodos se acordonan y vacían minuciosamente para minimizar las interrupciones en las aplicaciones en ejecución.  

Para obtener más información sobre las versiones del ciclo de vida, vea [Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)][aks-supported versions]. Para obtener información sobre los pasos de actualización, vea [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade].

### <a name="gpu-enabled-nodes"></a>Nodos habilitados para GPU

AKS admite la creación de grupos de nodos habilitados para GPU. Azure proporciona actualmente máquinas virtuales habilitadas para una o varias GPU. Las máquinas virtuales habilitadas para GPU están diseñadas para cargas de trabajo de proceso intensivo, uso intensivo de gráficos y visualización.

Para más información, consulte [Uso de GPU en AKS][aks-gpu].

### <a name="confidential-computing-nodes-public-preview"></a>Nodos de computación confidencial (versión preliminar pública)

AKS admite la creación de grupos de nodos de computación confidencial basados en Intel SGX (máquinas virtuales DCSv2). Los nodos de computación confidencial permiten que los contenedores se ejecuten en un entorno de ejecución de confianza basado en hardware (enclaves). El aislamiento entre contenedores en combinación con la integridad del código mediante la atestación puede contribuir a la estrategia de seguridad de contenedores de defensa en profundidad. Los nodos de computación confidencial admiten tanto contenedores confidenciales (aplicaciones de Docker existentes) como contenedores compatibles con enclave.

Para más información, consulte [Nodos de computación confidencial en Azure Kubernetes Service][conf-com-node].

### <a name="storage-volume-support"></a>Compatibilidad con volúmenes de almacenamiento

Para admitir las cargas de trabajo de la aplicación, puede montar volúmenes de almacenamiento estáticos o dinámicos para los datos persistentes. En función del número de pods conectados que se espere que vayan a compartir los volúmenes de almacenamiento, se puede usar un almacenamiento respaldado por:
* Azure Disks para el acceso de un solo pod, o bien 
* Azure Files para el acceso de varios pod al mismo tiempo.

Para más información, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

Empiece con volúmenes persistentes dinámicos mediante [Azure Disks][azure-disk] o [Azure Files][azure-files].

## <a name="virtual-networks-and-ingress"></a>Redes virtuales y entrada

Un clúster de AKS se puede implementar en una red virtual existente. En esta configuración, cada pod del clúster tiene asignada una dirección IP en la red virtual y puede comunicarse directamente con:
* Otros pods del clúster. 
* Otros nodos de la red virtual. 

Un pod se puede conectar también con otros servicios de una red virtual emparejada y con redes locales mediante ExpressRoute o conexiones VPN de sitio a sitio (S2S).  

Para más información, consulte [Conceptos de red para aplicaciones en AKS][aks-networking].

### <a name="ingress-with-http-application-routing"></a>Entrada con el enrutamiento de aplicación HTTP

El complemento de enrutamiento de aplicaciones de HTTP le ayuda a acceder fácilmente a las aplicaciones implementadas en el clúster de AKS. Cuando está habilitada, la solución de enrutamiento de aplicaciones HTTP configura un controlador de entrada en el clúster de AKS.  

A medida que se implementan las aplicaciones, los nombres DNS con acceso público se configuran automáticamente. El enrutamiento de aplicación HTTP configura una zona DNS y la integra con el clúster de AKS. A continuación, puede implementar recursos de entrada de Kubernetes de la forma habitual.  

Para empezar con el tráfico de entrada, consulte [Enrutamiento de aplicación HTTP][aks-http-routing].

## <a name="development-tooling-integration"></a>Integración de herramientas de desarrollo

Kubernetes tiene un ecosistema completo de herramientas de desarrollo y administración que funcionan sin problemas con AKS. Estas herramientas incluyen Helm y la extensión Kubernetes para Visual Studio Code. 

Azure proporciona varias herramientas, como DevOps Starter, que ayudan a simplificar Kubernetes.  

### <a name="devops-starter"></a>DevOps Starter

DevOps Starter ofrece una solución sencilla para llevar a Azure el código y los repositorios de Git existentes. DevOps Starter hace lo siguiente de forma automática:
* Crea recursos de Azure (como AKS), 
* Configura una canalización de versión en Azure DevOps Services que incluye una canalización de compilación para integración continua, 
* Configura una canalización de versión para implementación continua y 
* Genera un recurso de Azure Application Insights para la supervisión. 

Para más información, consulte [DevOps Starter][azure-devops].

## <a name="docker-image-support-and-private-container-registry"></a>Compatibilidad con imágenes de Docker y registro de contenedor privado

AKS admite el formato de imagen de Docker. Para el almacenamiento privado de las imágenes de Docker, puede realizar la integración de AKS con Azure Container Registry (ACR).

Para crear un almacén de imágenes privado, consulte [Azure Container Registry][acr-docs].

## <a name="kubernetes-certification"></a>Certificación de Kubernetes

AKS cuenta con el certificado CNCF de cumplimiento con Kubernetes.

## <a name="regulatory-compliance"></a>Cumplimiento de normativas

AKS cumple con SOC, ISO, PCI DSS e HIPAA. Para más información, consulte [Introducción al cumplimiento de Microsoft Azure][compliance-doc].

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la implementación y administración de AKS con el inicio rápido de la CLI de Azure.

> [!div class="nextstepaction"]
> [Implementación de un clúster de AKS Cluster mediante la CLI de Azure][aks-cli]

<!-- LINKS - external -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubectl-overview]: https://kubernetes.io/docs/user-guide/kubectl-overview/
[compliance-doc]: https://azure.microsoft.com/en-us/overview/trusted-cloud/compliance/

<!-- LINKS - internal -->
[acr-docs]: ../container-registry/container-registry-intro.md
[aks-aad]: ./azure-ad-integration-cli.md
[aks-cli]: ./kubernetes-walkthrough.md
[aks-gpu]: ./gpu-cluster.md
[aks-http-routing]: ./http-application-routing.md
[aks-networking]: ./concepts-network.md
[aks-portal]: ./kubernetes-walkthrough-portal.md
[aks-scale]: ./tutorial-kubernetes-scale.md
[aks-upgrade]: ./upgrade-cluster.md
[azure-dev-spaces]: ../dev-spaces/index.yml
[azure-devops]: ../devops-project/overview.md
[azure-disk]: ./azure-disks-dynamic-pv.md
[azure-files]: ./azure-files-dynamic-pv.md
[container-health]: ../azure-monitor/containers/container-insights-overview.md
[aks-master-logs]: ./view-control-plane-logs.md
[aks-supported versions]: supported-kubernetes-versions.md
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[kubernetes-rbac]: concepts-identity.md#kubernetes-rbac
[concepts-identity]: concepts-identity.md
[concepts-storage]: concepts-storage.md
[conf-com-node]: ../confidential-computing/confidential-nodes-aks-overview.md
