---
title: Directiva de soporte técnico del clúster de la versión 4 de Red Hat OpenShift en Azure
description: Información acerca de la directiva de soporte técnico de la versión 4 de Red Hat OpenShift
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 03/05/2021
ms.openlocfilehash: d6f3368619395934dd358536e39f227bc66dfbfd
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110497405"
---
# <a name="azure-red-hat-openshift-support-policy"></a>Directiva de soporte técnico de Red Hat OpenShift en Azure

Ciertas configuraciones de los clústeres de la versión 4 de Red Hat OpenShift en Azure pueden afectar a la compatibilidad del clúster. La versión 4 de Red Hat OpenShift en Azure permite a los administradores de clúster realizar cambios en los componentes internos del clúster, pero no se admiten todos los cambios. La directiva de soporte técnico siguiente comparte qué modificaciones infringen la directiva y anulan el soporte técnico de Microsoft y Red Hat.

> [!NOTE]
> Las características con la marca de vista previa de tecnología en la plataforma de contenedores de OpenShift no se admiten en Red Hat OpenShift en Azure.

## <a name="cluster-configuration-requirements"></a>Requisitos de configuración de clústeres

* Todos los operadores de clúster de OpenShift deben permanecer en estado administrado. La lista de operadores de clústeres se puede devolver mediante la ejecución de `oc get clusteroperators`.
* El clúster debe tener al menos tres nodos de trabajo y tres nodos de administrador. No tenga intolerancias que impidan la programación de los componentes de OpenShift. No escale los trabajos del clúster a cero ni intente realizar un cierre correcto del clúster.
* No quite ni modifique los servicios Prometheus y Alertmanager del clúster.
* No quite las reglas de Alertmanager del servicio.
* No quite ni modifique los grupos de seguridad de red.
* No quite ni modifique el registro de servicios de Red Hat OpenShift en Azure (pods mdsd).
* No quite ni modifique el secreto de extracción del clúster "arosvc.azurecr.io".
* Todas las máquinas virtuales del clúster deben tener acceso directo de salida a Internet, al menos a los puntos de conexión de Azure Resource Manager (ARM) y del registro de servicios (Geneva).  No se admite ningún tipo de proxy HTTPS.
* No modifique la configuración de DNS de la red virtual del clúster. Se debe usar la resolución de Azure DNS predeterminada.
* No invalide ninguno de los objetos MachineConfig del clúster (por ejemplo, la configuración de kubelet) de ningún modo.
* No establezca ninguna opción unsupportedConfigOverrides. Establecer estas opciones impide que la versión secundaria se actualice.
* El servicio Red Hat OpenShift en Azure accede al clúster a través del servicio Private Link.  No quite ni modifique el acceso al servicio.
* No se admiten nodos de ejecución que no sean de Red Hat Enterprise Linux CoreOS. Por ejemplo, no puede usar un nodo de proceso de Red Hat Enterprise Linux.
* No coloque directivas dentro de su suscripción o grupo de administración que impidan que los SRE realicen un mantenimiento normal en el clúster de ARO, como requerir etiquetas en el grupo de recursos de clúster administrado por ARO RP.

## <a name="supported-virtual-machine-sizes"></a>Tamaños de máquinas virtuales que se admiten

La versión 4 de Red Hat OpenShift en Azure admite instancias de nodo de trabajo en los siguientes tamaños de máquina virtual:

### <a name="general-purpose"></a>Uso general

|Serie|Size|vCPU|Memoria: GiB|
|-|-|-|-|
|Dasv4|Standard_D4as_v4|4|16|
|Dasv4|Standard_D8as_v4|8|32|
|Dasv4|Standard_D16as_v4|16|64|
|Dasv4|Standard_D32as_v4|32|128|
|Dsv3|Standard_D4s_v3|4|16|
|Dsv3|Standard_D8s_v3|8|32|
|Dsv3|Standard_D16s_v3|16|64|
|Dsv3|Standard_D32s_v3|32|128|

### <a name="memory-optimized"></a>Memoria optimizada

|Serie|Size|vCPU|Memoria: GiB|
|-|-|-|-|
|Esv3|Standard_E4s_v3|4|32|
|Esv3|Standard_E8s_v3|8|64|
|Esv3|Standard_E16s_v3|16|128|
|Esv3|Standard_E32s_v3|32|256|

### <a name="compute-optimized"></a>Proceso optimizado

|Serie|Size|vCPU|Memoria: GiB|
|-|-|-|-|
|Fsv2|Standard_F4s_v2|4|8|
|Fsv2|Standard_F8s_v2|8|16|
|Fsv2|Standard_F16s_v2|16|32|
|Fsv2|Standard_F32s_v2|32|64|

### <a name="master-nodes"></a>Nodos maestros

|Serie|Size|vCPU|Memoria: GiB|
|-|-|-|-|
|Dsv3|Standard_D8s_v3|8|32|
|Dsv3|Standard_D16s_v3|16|64|
|Dsv3|Standard_D32s_v3|32|128|
