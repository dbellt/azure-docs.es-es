---
title: Procedimientos recomendados para el almacenamiento y las copias de seguridad
titleSuffix: Azure Kubernetes Service
description: Obtenga más información acerca de los procedimientos recomendados del operador de clústeres para el almacenamiento, el cifrado de datos y las copias de seguridad en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/10/2021
ms.openlocfilehash: 9b3ee6fd7eea958a573743b21bf8940458e2a965
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107104922"
---
# <a name="best-practices-for-storage-and-backups-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)

A medida que se crean y gestionan clústeres en Azure Kubernetes Service (AKS), las aplicaciones a menudo necesitan almacenamiento. Asegúrese de que comprende las necesidades de rendimiento de los pods y los métodos de acceso para poder seleccionar el mejor almacenamiento para la aplicación. El tamaño del nodo de AKS puede afectar a las opciones de almacenamiento. Debe planear la manera de realizar copias de seguridad y de probar el proceso de restauración del almacenamiento conectado.

Este artículo de procedimientos recomendados se centra en las consideraciones de almacenamiento de los operadores de clústeres. En este artículo, aprenderá lo siguiente:

> [!div class="checklist"]
> * Los tipos de almacenamiento disponibles.
> * El dimensionamiento correcto del tamaño de los nodos de AKS para el rendimiento del almacenamiento.
> * Las diferencias entre el aprovisionamiento de volúmenes dinámico y estático.
> * Las formas de realizar copias de seguridad y proteger los volúmenes de datos.

## <a name="choose-the-appropriate-storage-type"></a>Elección del tipo de almacenamiento adecuado

> **Guía de procedimientos recomendados**
> 
> Conozca las necesidades de la aplicación para seleccionar el almacenamiento correcto. Utilice almacenamiento de alto rendimiento respaldado por SSD para cargas de trabajo de producción. Planee el almacenamiento basado en red cuando se necesiten varias conexiones simultáneas.

Las aplicaciones a menudo requieren diferentes tipos y velocidades de almacenamiento. Determine el tipo de almacenamiento más adecuado mediante las siguientes preguntas. 
* ¿Necesitan las aplicaciones almacenamiento que se conecte a pods individuales?
* ¿Necesitan las aplicaciones almacenamiento compartido entre varios pods? 
* El almacenamiento, ¿es para el acceso de solo lectura a los datos?
* ¿Se usará el almacenamiento para escribir grandes cantidades de datos estructurados? 

En la tabla siguiente se describen los tipos de almacenamiento disponibles y sus capacidades:

| Caso de uso | Complemento de volumen | Lectura/escritura una vez | Solo lectura varias veces | Lectura/escritura varias veces | Compatibilidad con contenedores de Windows Server |
|----------|---------------|-----------------|----------------|-----------------|--------------------|
| Configuración compartida       | Azure Files   | Sí | Sí | Sí | Sí |
| Datos de la aplicación estructurados        | Azure Disks   | Sí | No  | No  | Sí |
| Datos no estructurados, operaciones del sistema de archivos | [BlobFuse][blobfuse] | Sí | Sí | Sí | No |

AKS proporciona dos tipos principales de almacenamiento seguro para los volúmenes respaldados por Azure Disks o Azure Files. Ambos usan el cifrado Storage Service Encryption (SSE) predeterminado de Azure que cifra los datos en reposo. No se pueden cifrar los discos con Azure Disk Encryption en el nivel de nodo de AKS.

Tanto Azure Files como Azure Disks están disponibles en los niveles de rendimiento Estándar y Premium:

- Discos *prémium*
    - Respaldados por discos de estado sólido (SSD) de alto rendimiento. 
    - Se recomiendan para las cargas de trabajo de producción.
- Discos *estándar*
    - Respaldados por discos de rotación normales (HDD).
    - Buenos para datos de archivo o a los que se accede con poca frecuencia.

Comprender las necesidades de rendimiento de la aplicación y los patrones de acceso para elegir el nivel de almacenamiento adecuado. Para más información sobre los tamaños y niveles de rendimiento de Managed Disks, consulte [Introducción a los discos administrados de Azure][managed-disks]

### <a name="create-and-use-storage-classes-to-define-application-needs"></a>Crear y utilizar clases de almacenamiento para definir las necesidades de la aplicación

Defina el tipo de almacenamiento que desea mediante las *clases de almacenamiento* de Kubernetes. A continuación, se hacer referencia a la clase de almacenamiento en la especificación de almacenamiento o en el pod. Estas definiciones de clase de almacenamiento funcionan conjuntamente para crear el almacenamiento adecuado y conectarlo a los pods. 

Para más información, consulte [Clases de almacenamiento en AKS][aks-concepts-storage-classes].

## <a name="size-the-nodes-for-storage-needs"></a>Cambiar el tamaño de los nodos según las necesidades de almacenamiento

> **Guía de procedimientos recomendados**
> 
> Cada tamaño de nodo tiene un límite máximo de discos. Los diferentes tamaños de nodos también proporcionan distintas cantidades de almacenamiento local y de ancho de banda de red. Realice el planeamiento adecuado a la demanda de la aplicación para implementar el tamaño correcto de nodo.

Los nodos de AKS se ejecutan tipos y tamaños de máquina virtual de Azure distintos. Cada tamaño de máquina virtual proporciona:
* Una cantidad diferente de recursos principales, como CPU y memoria. 
* Un número máximo de discos que se pueden conectar. 

El rendimiento del almacenamiento también varía en función de los tamaños de VM para el IOPS (operaciones de entrada/salida por segundo) máximo de discos conectados y locales.

Si las aplicaciones requieren Azure Disks como solución de almacenamiento, planee y elija un tamaño de máquina virtual para el nodo adecuado. Las funcionalidades de almacenamiento y las cantidades de CPU y memoria desempeñan un papel principal a la hora de decidir el tamaño de máquina virtual. 

Por ejemplo, aunque *Standard_B2ms* y *Standard_DS2_v2* incluyen una cantidad similar de recursos de CPU y memoria, su rendimiento de almacenamiento difiere:

| Tamaño y tipo de nodo | vCPU | Memoria (GiB) | Discos de datos máx. | IOPS de disco máximo sin almacenamiento en caché | Rendimiento máximo sin almacenamiento en caché (MBps) |
|--------------------|------|--------------|----------------|------------------------|--------------------------------|
| Standard_B2ms      | 2    | 8            | 4              | 1\.920                  | 22.5                           |
| Standard_DS2_v2    | 2    | 7            | 8              | 6\.400                  | 96                             |

En este ejemplo, *Standard_DS2_v2* ofrece el doble de discos conectados y de tres a cuatro veces la cantidad de IOPS y rendimiento del disco. Si solo comparó los recursos de proceso principales y los costos, quizá haya elegido *Standard_B2ms* como tamaño de máquina virtual, que presenta limitaciones y menor rendimiento del almacenamiento. 

Trabaje con su equipo de desarrollo de aplicaciones para comprender sus necesidades de rendimiento y de capacidad de almacenamiento. Elija el tamaño de VM adecuado para los nodos de AKS para cumplir o superar las necesidades de rendimiento. Establezca regularmente las aplicaciones para ajustar el tamaño de VM según sea necesario.

Para más información sobre los tamaños de máquina virtual disponibles, consulte [Tamaños de las máquinas virtuales Linux en Azure][vm-sizes].

## <a name="dynamically-provision-volumes"></a>Aprovisionar volúmenes dinámicamente

> **Guía de procedimientos recomendados** 
>
> Para reducir la sobrecarga de administración y habilitar el escalado, evite crear y asignar volúmenes persistentes estáticamente. Use el aprovisionamiento dinámico. En las clases de almacenamiento, defina la directiva de reclamación adecuada para reducir los costos de almacenamiento innecesarios una vez que se eliminan los pods.

Para conectar el almacenamiento a los pods, use volúmenes persistentes. Los volúmenes persistentes se pueden crear manual o dinámicamente. La creación manual de volúmenes persistentes agrega sobrecarga de administración y limita su capacidad de escalado. En su lugar, aprovisione el volumen persistente de manera dinámica para simplificar la administración del almacenamiento y permitir que las aplicaciones crezcan y se escalen según sea necesario.

![Notificaciones de volúmenes persistentes en un clúster de Azure Kubernetes Service (AKS)](media/concepts-storage/persistent-volume-claims.png)

Una notificación de volumen persistente (PVC) le permite crear almacenamiento dinámicamente según sea necesario. Los discos subyacentes de Azure se crean a medida que los solicitan los pods. En la definición del pod se solicita la creación de un volumen y su conexión a una ruta de acceso de montaje designada.

Para los conceptos sobre cómo crear y usar volúmenes dinámicamente, consulte [Notificaciones de volúmenes persistentes][aks-concepts-storage-pvcs].

Para ver los volúmenes en acción, consulte cómo crear y usar dinámicamente un volumen persistente con [Azure Disks][dynamic-disks] o [Azure Files][dynamic-files].

Como parte de las definiciones de clase de almacenamiento, configure la directiva *reclaimPolicy* apropiada. Esta directiva reclaimPolicy controla el comportamiento del recurso de almacenamiento subyacente de Azure cuando se elimina el pod. El recurso de almacenamiento subyacente se puede eliminar o conservar para su uso con un pod en el futuro. Establezca la directiva reclaimPolicy en *Retener* o *Eliminar*. 

Conozca las necesidades de su aplicación e implemente comprobaciones periódicas del almacenamiento que se conserva para minimizar la cantidad de espacio de almacenamiento que no se usa y se factura.

Para más información sobre las opciones de clase de almacenamiento, consulte las [directivas de reclamación de almacenamiento][reclaim-policy].

## <a name="secure-and-back-up-your-data"></a>Proteja y realice una copia de seguridad de los datos

> **Guía de procedimientos recomendados** 
> 
> Realice una copia de seguridad de los datos con una herramienta apropiada para su tipo de almacenamiento, como Velero o Azure Backup. Compruebe la integridad y seguridad de las copias de seguridad.

Cuando las aplicaciones almacenan y consumen datos que persisten en los discos o en los archivos, es necesario realizar copias de seguridad o instantáneas periódicamente de esos datos. La tecnología Azure Disks puede usar la tecnología integrada de instantánea. Las aplicaciones pueden necesitar vaciar las escrituras en el disco para la operación de instantánea. [Velero][velero] puede realizar copias de seguridad de volúmenes persistentes junto con recursos y configuraciones de clústeres adicionales. Si no puede [eliminar el estado de sus aplicaciones][remove-state], realice una copia de seguridad de los datos de los volúmenes persistentes y pruebe regularmente las operaciones de restauración para verificar la integridad de los datos y los procesos necesarios.

Comprenda las limitaciones de los diferentes enfoques de las copias de seguridad de datos y si necesita dejar sus datos en reposo antes de la instantánea. Las copias de seguridad de datos no necesariamente le permiten restaurar el entorno de la aplicación de la implementación del clúster. Para más información acerca de estos escenarios, consulte [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres en AKS][best-practices-multi-region].

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en el procedimiento recomendado de almacenamiento en AKS. Para más información sobre los conceptos básicos de almacenamiento en Kubernetes, consulte [Opciones de almacenamiento de aplicaciones en Azure Kubernetes Service (AKS)][aks-concepts-storage].

<!-- LINKS - External -->
[velero]: https://github.com/heptio/velero
[blobfuse]: https://github.com/Azure/azure-storage-fuse

<!-- LINKS - Internal -->
[aks-concepts-storage]: concepts-storage.md
[vm-sizes]: ../virtual-machines/sizes.md
[dynamic-disks]: azure-disks-dynamic-pv.md
[dynamic-files]: azure-files-dynamic-pv.md
[reclaim-policy]: concepts-storage.md#storage-classes
[aks-concepts-storage-pvcs]: concepts-storage.md#persistent-volume-claims
[aks-concepts-storage-classes]: concepts-storage.md#storage-classes
[managed-disks]: ../virtual-machines/managed-disks-overview.md
[best-practices-multi-region]: operator-best-practices-multi-region.md
[remove-state]: operator-best-practices-multi-region.md#remove-service-state-from-inside-containers
