---
title: Arquitectura de agentes de Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 03/03/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: En este artículo se proporciona información general de los agentes de Kubernetes habilitado para Azure Arc.
keywords: Kubernetes, Arc, Azure, containers
ms.openlocfilehash: f59a897e4868d7b16d0a50c28ce2142320992f71
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106442548"
---
# <a name="azure-arc-enabled-kubernetes-agent-architecture"></a>Arquitectura de agentes de Kubernetes habilitado para Azure Arc

Por sí solo, [Kubernetes](https://kubernetes.io/) puede implementar de forma coherente cargas de trabajo en contenedores en entornos híbridos y de varias nubes. Sin embargo, Kubernetes habilitado para Azure Arc funciona como plano de control centralizado que administra directivas, gobernanza y seguridad en estos entornos heterogéneos. En este artículo se proporciona:

* Información general sobre la arquitectura de conexión de clústeres a Azure Arc.
* El patrón de conectividad que siguen los agentes.
* Una descripción de los datos que se intercambian entre el entorno de los clústeres y Azure.

## <a name="deploy-agents-to-your-cluster"></a>Implementación de agentes en un clúster

La mayoría de los centros de datos locales aplican estrictas reglas de red que evitan la comunicación entrante en el firewall del límite de la red. Kubernetes habilitado para Azure Arc usa estas restricciones al no requerir ningún puerto de entrada en el firewall y solo habilitar ciertos puntos de conexión de salida para la comunicación saliente. Los agentes de Kubernetes habilitado para Azure Arc inician esta comunicación saliente. 

![Introducción a la arquitectura](./media/architectural-overview.png)

### <a name="connect-a-cluster-to-azure-arc"></a>Conexión de un clúster a Azure Arc

1. Cree un clúster de Kubernetes en la infraestructura que prefiera (VMware vSphere, Amazon Web Services, Google Cloud Platform, etc.). 

    > [!NOTE]
    > Dado que actualmente Kubernetes habilitado para Azure Arc solo admite la conexión de clústeres de Kubernetes existentes a Azure Arc, los clientes deben crear y administrar el ciclo de vida del clúster de Kubernetes por sí mismos.  

1. Inicie el registro de Azure Arc para el clúster mediante la CLI de Azure.
    * La CLI de Azure usa Helm para implementar el gráfico Helm del agente en el clúster.
    * Los nodos de clúster inician una comunicación saliente con [Microsoft Container Registry](https://github.com/microsoft/containerregistry) y extraen las imágenes necesarias para crear los siguientes agentes en el espacio de nombres `azure-arc`:

        | Agente | Descripción |
        | ----- | ----------- |
        | `deployment.apps/clusteridentityoperator` | Actualmente, Kubernetes habilitado para Azure Arc solo admite [identidades asignadas por el sistema](../../active-directory/managed-identities-azure-resources/overview.md). `clusteridentityoperator` inicia la primera comunicación saliente. La primera comunicación captura el certificado de Managed Service Identity (MSI) que usan otros agentes para comunicarse con Azure. |
        | `deployment.apps/config-agent` | Comprueba en el clúster conectado los recursos de configuración de control de código fuente aplicados en el clúster. Actualiza el estado de cumplimiento. |
        | `deployment.apps/controller-manager` | Operador de operadores que organiza las interacciones entre los componentes de Azure Arc. |    
        | `deployment.apps/metrics-agent` | Recopila las métricas de otros agentes de Arc para comprobar el rendimiento óptimo. |
        | `deployment.apps/cluster-metadata-operator` | Recopila metadatos del clúster, incluida la versión del clúster, el recuento de nodos y la versión del agente de Azure Arc. |
        | `deployment.apps/resource-sync-agent` | Sincroniza estos metadatos mencionados anteriormente con Azure. |
        | `deployment.apps/flux-logs-agent` | Recopila registros de los operadores de Flux implementados como parte de la configuración de control de código fuente. |
        | `deployment.apps/extension-manager` | Instala y administra el ciclo de vida de los gráficos de Helm de extensiones. |  
        | `deployment.apps/clusterconnect-agent` | Agente proxy inverso que habilita la característica de conexión de clúster para proporcionar acceso al `apiserver` del clúster. Se trata de un componente opcional implementado solo si la característica `cluster-connect` está habilitada en el clúster   |
        | `deployment.apps/guard` | Servidor de webhook de autenticación y autorización usado para la característica RBAC de AAD. Se trata de un componente opcional implementado solo si la característica `azure-rbac` está habilitada en el clúster   |

1. Una vez que todos los pods de los agentes de Kubernetes habilitado para Azure Arc estén en estado `Running`, compruebe que el clúster esté conectado a Azure Arc. Debería ver lo siguiente:
    * Un recurso de Kubernetes habilitado para Azure Arc en [Azure Resource Manager](../../azure-resource-manager/management/overview.md). Azure hace un seguimiento de este recurso como una proyección del clúster de Kubernetes administrado por el cliente, no del propio clúster de Kubernetes real.
    * Los metadatos del clúster (como la versión de Kubernetes, la versión del agente y el número de nodos) aparecen en el recurso de Kubernetes habilitado para Azure Arc como metadatos.

## <a name="data-exchange-between-cluster-environment-and-azure"></a>Intercambio de datos entre el entorno del clúster y Azure

| Tipo de datos | Escenario | Modo de comunicación |
| --------- | -------- | ------------------ |
| Versión del clúster de Kubernetes | Metadatos del clúster | Inserciones del agente en Azure |
| Número de nodos del clúster | Metadatos del clúster | Inserciones del agente en Azure |
| Versión del agente | Metadatos del clúster | Inserciones del agente en Azure |
| Tipo de distribución de Kubernetes | Metadatos del clúster | Inserciones de la CLI de Azure en Azure |
| Tipo de infraestructura (AWS/GCP/vSphere/...) | Metadatos del clúster | Inserciones de la CLI de Azure en Azure |
| Número de nodos de vCPU en el clúster | Facturación | Inserciones de la CLI de Azure en Azure |
| Latido de agente | Estado de los recursos | Inserciones del agente en Azure |
| Consumo de recursos (memoria/CPU) por agentes | Diagnósticos y compatibilidad | Inserciones del agente en Azure |
| Registros de todos los contenedores de agentes | Diagnósticos y compatibilidad | Inserciones del agente en Azure |
| Actualización de disponibilidad del agente | Actualización del agente | Extracción del agente de Azure |
| Estado deseado de configuración: URL del repositorio de GIT, parámetros del operador de Flux, clave privada, contenido de hosts conocidos, nombre de usuario de HTTPS, token o contraseña | Configuración | Extracción del agente de Azure |
| Estado de instalación del operador de Flux | Configuración | Inserciones del agente en Azure |
| Asignaciones de Azure Policy que necesitan el cumplimiento de Gatekeeper en el clúster | Azure Policy | Extracción del agente de Azure |
| Auditoría y estado de cumplimiento de las directivas del clúster | Azure Policy | Inserciones del agente en Azure |
| Métricas y registros de las cargas de trabajo del cliente | Azure Monitor | Inserciones del agente en el recurso del área de trabajo de Log Analytics de la suscripción y del inquilino del cliente |

## <a name="connectivity-status"></a>Estado de conectividad

| Status | Descripción |
| ------ | ----------- |
| Connecting | El recurso Kubernetes habilitado para Azure Arc se creó en Azure Resource Manager, pero el servicio aún no ha recibido el latido del agente. |
| Conectado | El servicio Kubernetes habilitado para Azure Arc ha recibido un latido del agente en los últimos 15 minutos. |
| Sin conexión | El recurso Kubernetes habilitado para Azure Arc se conectó anteriormente, pero el servicio no ha recibido ningún latido de agente durante 15 minutos. |
| Expirada | El certificado deMSI tiene una ventana de expiración de 90 días después de su emisión. Una vez que este certificado expira, el recurso se considera `Expired` y todas las características, como la configuración, la supervisión y la directiva, dejan de funcionar en este clúster. En el [artículo de P+F](./faq.md#how-to-address-expired-azure-arc-enabled-kubernetes-resources) encontrará más información sobre cómo abordar la expiración de los recursos de Kubernetes habilitado para Azure Arc. |

## <a name="understand-connectivity-modes"></a>Información sobre los modos de conectividad

| Modo de conectividad | Descripción |
| ----------------- | ----------- |
| Completamente conectado | Los agentes pueden comunicarse de forma coherente con Azure con poco retraso en la propagación de configuraciones de GitOps, la aplicación de directivas de Azure Policy y Gatekeeper, y la recopilación de métricas de cargas de trabajo y registros en Azure Monitor. |
| Con conexión parcial | El certificado de MSI que ha extraído `clusteridentityoperator` tiene una validez de 90 días antes de que expire. Una vez que expira, el recurso Kubernetes habilitado para Azure Arc deja de funcionar. Para reactivar todas las características de Azure Arc en el clúster, elimine y vuelva a crear tanto el recurso de Kubernetes habilitado para Azure Arc como los agentes. Durante los 90 días, conecte el clúster al menos una vez cada 30 días. |
| Escenario desconectado | Actualmente, Kubernetes habilitado para Azure Arc no admite clústeres de Kubernetes en entornos desconectados que no pueden acceder a Azure. Si esta funcionalidad le interesa, envíe o vote dicha idea en el [foro de UserVoice de Azure Arc](https://feedback.azure.com/forums/925690-azure-arc).

## <a name="next-steps"></a>Pasos siguientes

* Siga el inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* Obtenga más información sobre la creación de conexiones entre el clúster y un repositorio de GIT como un [recurso de configuración con Kubernetes habilitado para Azure Arc](./conceptual-configurations.md).