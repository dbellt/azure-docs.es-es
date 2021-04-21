---
title: 'Conceptos: seguridad en Azure Kubernetes Service (AKS)'
description: Obtenga información acerca de la seguridad en Azure Kubernetes Service (AKS), incluidos los secretos de Kubernetes, las directivas de red, y la comunicación con el maestro y los nodos.
services: container-service
author: mlearned
ms.topic: conceptual
ms.date: 03/11/2021
ms.author: mlearned
ms.openlocfilehash: 3fafbe3f4b1c53f929682f4ca160fb19a5e91918
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105313"
---
# <a name="security-concepts-for-applications-and-clusters-in-azure-kubernetes-service-aks"></a>Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)

La seguridad del clúster protege los datos de los clientes a medida que ejecuta cargas de trabajo de aplicaciones en Azure Kubernetes Service (AKS). 

Kubernetes incluye componentes de seguridad, tales como *directivas de red* y *secretos*. Mientras tanto, Azure incluye componentes como grupos de seguridad de red y actualizaciones de clústeres orquestadas. AKS combina estos componentes de seguridad para:
* Hacer que el clúster de AKS ejecute las últimas actualizaciones de seguridad del sistema operativo y versiones de Kubernetes.
* Proporcionar tráfico de pod seguro y acceso a credenciales confidenciales.

En este artículo se presentan los conceptos básicos para proteger sus aplicaciones en AKS:

- [Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)](#security-concepts-for-applications-and-clusters-in-azure-kubernetes-service-aks)
  - [Seguridad de componentes maestros](#master-security)
  - [Seguridad de nodos](#node-security)
    - [Aislamiento de proceso](#compute-isolation)
  - [Actualización de un clúster de Service Fabric](#cluster-upgrades)
    - [Acordonamiento y purga](#cordon-and-drain)
  - [Seguridad de las redes](#network-security)
    - [Grupos de seguridad de red de Azure](#azure-network-security-groups)
  - [Secretos de Kubernetes](#kubernetes-secrets)
  - [Pasos siguientes](#next-steps)

## <a name="master-security"></a>Seguridad de componentes maestros

En AKS, los componentes maestros de Kubernetes forman parte del servicio administrado, proporcionado y mantenido por Microsoft. Cada clúster de AKS tiene su propio maestro de Kubernetes dedicado con un solo inquilino para proporcionar el servidor de API, Scheduler, etc.

De forma predeterminada, el servidor de API de Kubernetes utiliza una dirección IP pública y un nombre de dominio completo (FQDN). Puede limitar el acceso al punto de conexión del servidor de API mediante los [intervalos IP autorizados][authorized-ip-ranges]. También puede crear un [clúster privado][private-clusters] por completo para limitar el acceso del servidor de la API a la red virtual.

Puede regular el acceso al servidor de API mediante control de acceso basado en rol de Kubernetes (RBAC de Kubernetes) y RBAC de Azure. Para más información, consulte el artículo de [integración de Azure AD con AKS][aks-aad].

## <a name="node-security"></a>Seguridad de nodos

Los nodos de AKS son máquinas virtuales (VM) de Azure que el usuario administra y mantiene. 
* Los nodos de Linux ejecutan una distribución Ubuntu optimizada con `containerd` o el entorno de ejecución del contenedor de Moby. 
* Los nodos de Windows Server ejecutan una versión de Windows Server 2019 optimizada con `containerd` o el entorno de ejecución del contenedor de Moby. 

Cuando se crea o se escala verticalmente un clúster de AKS, los nodos se implementan automáticamente con las actualizaciones de seguridad del sistema operativo y las configuraciones más recientes.

> [!NOTE]
> Clústeres de AKS que usan:
> * Grupos de nodos con la versión 1.19 de Kubernetes y posterior usan `containerd` como entorno de ejecución del contenedor. 
> * Grupos de nodos con versiones anteriores a la 1.19 de Kubernetes usan [Moby](https://mobyproject.org/) (Docker ascendente) como entorno de ejecución del contenedor.

### <a name="node-security-patches"></a>Parches de seguridad de nodo

#### <a name="linux-nodes"></a>Nodos de Linux
La plataforma Azure aplica automáticamente las revisiones de seguridad del sistema operativo a los nodos de Linux del clúster durante la noche. Si una actualización de seguridad del sistema operativo de Linux requiere un reinicio del host, no se realizará automáticamente. Puede:
* Reiniciar manualmente los nodos de Linux.
* Usar [Kured][kured], un demonio de reinicio de código abierto para Kubernetes. Kured se ejecuta como un elemento [DaemonSet][aks-daemonsets] y supervisa en todos los nodos un archivo que indique que hace falta un reinicio. 

Los reinicios se administran en el clúster con el mismo [proceso de acordonar y purgar](#cordon-and-drain) que una actualización de clúster.

#### <a name="windows-server-nodes"></a>Nodos de Windows Server

Para los nodos de Windows Server, Windows Update no ejecuta ni aplica las actualizaciones más recientes de manera automática. Programe las actualizaciones del grupo de nodos de Windows Server en el clúster de AKS según el ciclo de Windows Update normal y su propio proceso de validación. Este proceso de actualización crea nodos que ejecutan la imagen y las revisiones más recientes de Windows Server y elimina los nodos anteriores. Para obtener más información sobre este proceso, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

### <a name="node-deployment"></a>Implementación de nodo
Los nodos se implementan en una subred de una red privada virtual, sin ninguna dirección IP pública asignada. Con fines de solución de problemas y administración, SSH está habilitado de manera predeterminada y solo es accesible mediante la dirección IP interna.

### <a name="node-storage"></a>Almacenamiento del nodo
Para proporcionar almacenamiento, los nodos usan Azure Managed Disks. Para la mayoría de los tamaños de nodo de máquina virtual, las instancias de Azure Managed Disks son discos Premium respaldados por SSD de alto rendimiento. Los datos almacenados en discos administrados se cifran automáticamente en reposo dentro de la plataforma Azure. Para mejorar la redundancia, las instancias de Azure Managed Disks se replican de forma segura en el centro de datos de Azure.

### <a name="hostile-multi-tenant-workloads"></a>Cargas de trabajo multiinquilino hostiles

Actualmente, los entornos de Kubernetes no están completamente seguros ante el uso de varios inquilinos hostiles. Las características de seguridad adicionales, como las *directivas de seguridad de pods* o Kubernetes RBAC para nodos, bloquean eficazmente las vulnerabilidades de seguridad. Para una verdadera seguridad al ejecutar cargas de trabajo multiinquilino hostiles, solo debe confiar en un hipervisor. El dominio de seguridad de Kubernetes se convierte en todo el clúster, no en un nodo específico. 

En el caso de estos tipos de cargas de trabajo multiinquilino hostiles, debe usar clústeres que estén físicamente aislados. Para obtener más información sobre las formas de aislar las cargas de trabajo, consulte [Procedimientos recomendados para el aislamiento de clústeres en AKS][cluster-isolation].

### <a name="compute-isolation"></a>Aislamiento de proceso

Debido a los requisitos normativos o de cumplimiento, ciertas cargas de trabajo pueden requerir un alto grado de aislamiento de otras cargas de trabajo del cliente. Para estas cargas de trabajo, Azure proporciona [VM aisladas](../virtual-machines/isolation.md) para usarse como nodos de agente en un clúster de AKS. Estas VM están aisladas en un tipo de hardware específico y están dedicadas a un solo cliente. 

Seleccione [uno de los tamaños de VM aisladas](../virtual-machines/isolation.md) como **tamaño de nodo** al crear un clúster de AKS o agregar un grupo de nodos.

## <a name="cluster-upgrades"></a>Actualizaciones de clústeres

Azure proporciona herramientas de orquestación de actualizaciones para actualizar un clúster de AKS y sus componentes, mantener la seguridad y el cumplimiento y acceder a las características más recientes. La orquestación de esta actualización incluye los componentes tanto de maestro como de agente de Kubernetes. 

Para iniciar el proceso de actualización, especifique una de las [versiones de Kubernetes disponibles de la lista](supported-kubernetes-versions.md). A continuación, Azure acordona y purga de manera segura cada uno de los nodos de AKS y los actualiza.

### <a name="cordon-and-drain"></a>Acordonar y purgar

Durante el proceso de actualización, los nodos de AKS se acordonan de manera individual desde el clúster para evitar que se programen nuevos pods en ellos. A continuación, los nodos se purgan y actualizan de la siguiente manera:

1.  Se implementa un nuevo nodo en el grupo de nodos. 
    * Este nodo ejecuta la imagen de sistema operativo y las revisiones más recientes.
1. Se identifica uno de los nodos existentes para la actualización. 
1. Los pods del nodo identificado finalizan correctamente y se programan en los otros nodos del grupo.
1. El nodo vacío se elimina del clúster de AKS.
1. Los pasos del 1 al 4 se repiten hasta que todos los nodos se reemplazan correctamente como parte del proceso de actualización.

Para más información, consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade-cluster].

## <a name="network-security"></a>Seguridad de las redes

Por motivos de conectividad y seguridad con las redes locales, puede implementar el clúster de AKS en subredes de red virtual de Azure existentes. Estas redes virtuales se conectan de vuelta a su red local con una VPN de sitio a sitio o ExpressRoute de Azure. Defina controladores de entrada de Kubernetes con direcciones IP privadas internas para limitar el acceso de los servicios a la conexión de red interna.

### <a name="azure-network-security-groups"></a>Grupos de seguridad de red de Azure

Para filtrar el flujo del tráfico de red virtual, Azure usa reglas de grupo de seguridad de red. Estas reglas definen los intervalos, los puertos y los protocolos de IP de origen y destino que tienen permitido o denegado el acceso a los recursos. Se crean reglas predeterminadas para permitir el tráfico TLS al servidor de API de Kubernetes. Los servicios se crean con equilibradores de carga, asignaciones de puertos o rutas de entrada. AKS modifica automáticamente el grupo de seguridad de red para el flujo de tráfico.

Si proporciona su propia subred para el clúster de AKS, **no** modifique el grupo de seguridad de red del nivel de subred que administra AKS. En su lugar, cree más grupos de seguridad de red de nivel de subred para modificar el flujo de tráfico. Asegúrese de que no interfieran con el tráfico necesario para administrar el clúster, como el acceso al equilibrador de carga, la comunicación con el plano de control y [la salida][aks-limit-egress-traffic].

### <a name="kubernetes-network-policy"></a>Directiva de red de Kubernetes

Para limitar el tráfico de red entre los pods del clúster, AKS ofrece compatibilidad con las [directivas de red de Kubernetes][network-policy]. Con las directivas de red, puede permitir o denegar las rutas de acceso de red específicas en el clúster en función de los espacios de nombres y los selectores de etiquetas.

## <a name="kubernetes-secrets"></a>Secretos de Kubernetes

Con un *secreto* de Kubernetes, puede insertar datos confidenciales en pods, como credenciales de acceso o claves. 
1. Cree un secreto mediante la API de Kubernetes. 
1. Defina el pod o la implementación y solicite un secreto específico. 
    * Los secretos solo se proporcionan a los nodos que tienen un pod programado que los requiere.
    * El secreto se almacena en *tmpfs*, no se escribe en el disco. 
1. Cuando se elimina el último pod de un nodo que requiere un secreto, dicho secreto se elimina del tmpfs del nodo. 
   * Los secretos se almacenan en un espacio de nombres determinado y solo son accesibles para los pods del mismo espacio de nombres.

El uso de secretos reduce la información confidencial definida en el pod o el manifiesto YAML del servicio. En su lugar, solicite el secreto almacenado en el servidor de API de Kubernetes como parte de su manifiesto YAML. Este enfoque proporciona acceso al secreto solo al pod específico. 

> [!NOTE]
> Los archivos de manifiesto de secreto sin formato contienen los datos secretos en formato base64 (consulte la [documentación oficial][secret-risks] para más detalles). Trate estos archivos como información confidencial y no los confirme nunca en el control de código fuente.

Los secretos de Kubernetes se almacenan en etcd, un almacén distribuido de pares clave-valor. El almacén etcd está totalmente administrado por AKS y [los datos se cifran en reposo dentro de la plataforma de Azure][encryption-atrest]. 

## <a name="next-steps"></a>Pasos siguientes

Para empezar a proteger los clústeres de AKS, consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade-cluster].

Para los procedimientos recomendados asociados, consulte [Procedimientos recomendados para la seguridad de clústeres y las actualizaciones en AKS][operator-best-practices-cluster-security] y [Procedimientos recomendados para la seguridad de pod en AKS][developer-best-practices-pod-security].

Para obtener más información sobre los conceptos básicos de Kubernetes y AKS, consulte:

- [Clústeres y cargas de trabajo de Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Identidad de Kubernetes/AKS][aks-concepts-identity]
- [Redes virtuales de Kubernetes/AKS][aks-concepts-network]
- [Almacenamiento de Kubernetes/AKS][aks-concepts-storage]
- [Escala de Kubernetes/AKS][aks-concepts-scale]

<!-- LINKS - External -->
[kured]: https://github.com/weaveworks/kured
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/
[secret-risks]: https://kubernetes.io/docs/concepts/configuration/secret/#risks
[encryption-atrest]: ../security/fundamentals/encryption-atrest.md

<!-- LINKS - Internal -->
[aks-daemonsets]: concepts-clusters-workloads.md#daemonsets
[aks-upgrade-cluster]: upgrade-cluster.md
[aks-aad]: ./managed-aad.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[aks-limit-egress-traffic]: limit-egress-traffic.md
[cluster-isolation]: operator-best-practices-cluster-isolation.md
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[developer-best-practices-pod-security]:developer-best-practices-pod-security.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[authorized-ip-ranges]: api-server-authorized-ip-ranges.md
[private-clusters]: private-clusters.md
[network-policy]: use-network-policies.md
