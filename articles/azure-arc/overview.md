---
title: Introducción a Azure Arc
description: Obtenga información sobre qué es Azure Arc y cómo ayuda a los clientes a habilitar la administración y el gobierno de sus recursos híbridos con otros servicios y características de Azure.
ms.date: 05/25/2021
ms.topic: overview
ms.openlocfilehash: b9e3cac865a92023d0079a5face9fbc8a3734c6e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111967082"
---
# <a name="azure-arc-overview"></a>Introducción a Azure Arc

En la actualidad, las empresas luchan por controlar y gobernar entornos cada vez más complejos. Estos entornos se extienden a través de los centros de datos, varias nubes y el borde. Cada entorno y nube poseen su propio conjunto de herramientas de administración separadas que tiene que aprender y operar.

En paralelo, los nuevos modelos operativos DevOps y ITOps son difíciles de implementar, ya que las herramientas existentes no proporcionan compatibilidad con los nuevos patrones nativos en la nube.

Azure Arc simplifica el gobierno y la administración al ofrecer una plataforma de administración local y multinube coherente. Azure Arc le permite:
* Administrar todo el entorno, con un panel único, mediante la proyección de los recursos existentes que no son de Azure, los recursos locales u otros recursos en la nube en Azure Resource Manager. 
* Administrar las máquinas virtuales, los clústeres de Kubernetes y las bases de datos como si se ejecutaran en Azure. 
* Usar los servicios y funcionalidades de administración de Azure que conozca, independientemente de dónde se encuentren. 
* Seguir usando ITOps tradicionales, al tiempo que presenta prácticas de DevOps para admitir en el entorno nuevos patrones nativos en la nube.
* Configurar ubicaciones personalizadas como una capa de abstracción sobre el clúster de Kubernetes habilitado para Azure Arc, la conexión de clústeres y las extensiones de clúster.  

:::image type="content" source="./media/overview/azure-arc-control-plane.png" alt-text="Diagrama del plano de control de la administración de Azure Arc" border="false":::

En la actualidad, Azure Arc le permite administrar los siguientes tipos de recursos hospedados fuera de Azure:

* Servidores: máquinas físicas y virtuales que ejecutan Windows o Linux.
* Clústeres de Kubernetes: admiten varias distribuciones de Kubernetes.
* Servicios de datos de Azure: servicios de hiperescala de Azure SQL Database y PostgreSQL.

## <a name="what-does-azure-arc-deliver"></a>¿Qué ofrece Azure Arc?

Entre las principales características de Azure Arc se incluyen:

* Implementación de un inventario, administración, gobierno y seguridad coherentes para los servidores en todo el entorno.

* Configuración de [extensiones de máquina virtual de Azure](./servers/manage-vm-extensions.md) para usar los servicios de administración de Azure y supervisar, proteger y actualizar los servidores.

* Administración y control de los clústeres de Kubernetes a escala.

* Use GitOps para implementar la configuración en uno o varios clústeres de repositorios Git.

*  Cumplimiento y configuración automáticos de los clústeres de Kubernetes mediante Azure Policy.

* Ejecución de los [servicios de datos de Azure](../azure-arc/kubernetes/custom-locations.md) en cualquier entorno de Kubernetes como si se ejecutaran en Azure (en concreto en Azure SQL Managed Instance e Hiperescala para Azure Database for PostgreSQL, con ventajas como las actualizaciones, la seguridad y la supervisión). Uso del escalado elástico y aplicación de actualizaciones, sin tiempo de inactividad de la aplicación, aunque no haya una conexión continua con Azure.

* Creación de [ubicaciones personalizadas](./kubernetes/custom-locations.md) sobre los clústeres de [Kubernetes habilitados para Azure Arc](./kubernetes/overview.md), usándolos como ubicaciones de destino para implementar instancias de servicios de Azure. Implementación de las extensiones de clúster de servicio de Azure para [Data Services habilitado para Azure Arc](./data/deploy-data-controller-direct-mode.md), [App Services en Azure Arc](../app-service/overview-arc-integration.md) (incluidas aplicaciones web, aplicaciones de funciones y aplicaciones lógicas) y [Event Grid en Kubernetes](../event-grid/kubernetes/overview.md).

* Una experiencia unificada para ver los recursos habilitados para Azure Arc, tanto si usa Azure Portal, la CLI de Azure, Azure PowerShell o la API REST de Azure.

## <a name="how-much-does-azure-arc-cost"></a>¿Cuánto cuesta Azure Arc?

A continuación se muestran los detalles de precios de las características disponibles hoy en día con Azure Arc.

### <a name="arc-enabled-servers"></a>Servidores habilitados para Arc

La siguiente funcionalidad del plano de control de Azure Arc se ofrece sin costo adicional:

* Organización de recursos mediante etiquetas y grupos de administración de Azure.

* Búsqueda e indexación mediante Azure Resource Graph.

* Acceso y seguridad mediante Azure RBAC y suscripciones.

* Entornos y automatización mediante plantillas y extensiones.

* Administración de la actualización.

Cualquier servicio de Azure que se use en los servidores habilitados para Arc, como Azure Security Center o Azure Monitor, se cobrará según los precios de ese servicio. Para más información, consulte la [página de precios de Azure](https://azure.microsoft.com/pricing/).

### <a name="azure-arc-enabled-kubernetes"></a>Kubernetes habilitado para Azure Arc

Cualquier servicio de Azure que se use en Kubernetes habilitado para Arc, como Azure Security Center o Azure Monitor, se cobrará según los precios de ese servicio. Para más información sobre los precios de las configuraciones de Kubernetes habilitado para Azure Arc, consulte la [página de precios de Azure](https://azure.microsoft.com/pricing/).

### <a name="azure-arc-enabled-data-services"></a>Servicios de datos habilitados para Azure Arc

En la fase de versión preliminar actual, los servicios de datos habilitados para Azure Arc se ofrecen sin costo adicional.

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de los servidores habilitados para Arc, consulte la siguiente [información general](./servers/overview.md)

* Para más información acerca de Kubernetes habilitado para Arc, consulte la siguiente [información general](./kubernetes/overview.md)

* Para más información acerca de los servicios de datos habilitados para Arc, consulte la siguiente [información general](https://azure.microsoft.com/services/azure-arc/hybrid-data-services/)

* Utilice los servicios habilitados para Arc desde la [prueba de concepto de JumpStart](https://azurearcjumpstart.io/azure_arc_jumpstart/)