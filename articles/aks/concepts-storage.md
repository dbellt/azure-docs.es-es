---
title: 'Conceptos: almacenamiento en Azure Kubernetes Service (AKS)'
description: Obtenga información sobre el almacenamiento en Azure Kubernetes Service (AKS), incluidos los volúmenes, los volúmenes persistentes, las clases de almacenamiento y las notificaciones
services: container-service
ms.topic: conceptual
ms.date: 03/11/2021
ms.openlocfilehash: a1f68b06c31597a1d2d044686274e86a79e6e9a3
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105789"
---
# <a name="storage-options-for-applications-in-azure-kubernetes-service-aks"></a>Opciones de almacenamiento de aplicaciones en Azure Kubernetes Service (AKS)

Las aplicaciones que se ejecutan en Azure Kubernetes Service (AKS) pueden necesitar almacenar y recuperar datos. Aunque algunas cargas de trabajo de aplicación pueden usar almacenamiento local y rápido en nodos vacíos innecesarios, otras requieren almacenamiento que se conserva en volúmenes de datos más normales dentro de la plataforma Azure. 

Es posible que varios pods necesiten:
* compartir los mismos volúmenes de datos o 
* volver a adjuntar volúmenes de datos si el pod se programa de nuevo en otro nodo. 

Por último, es posible que deba insertar datos confidenciales o información de configuración de la aplicación en los pods.

En este artículo se presentan los conceptos básicos que proporcionan almacenamiento para sus aplicaciones en AKS:

- [Volúmenes](#volumes)
- [Volúmenes persistentes](#persistent-volumes)
- [Clases de almacenamiento](#storage-classes)
- [Notificaciones de volúmenes persistentes](#persistent-volume-claims)

![Opciones de almacenamiento de aplicaciones en un clúster de Azure Kubernetes Service (AKS)](media/concepts-storage/aks-storage-options.png)

## <a name="volumes"></a>Volúmenes

Kubernetes suele tratar los pods individuales como recursos efímeros y descartables. Las aplicaciones disponen de diferentes enfoques para usar y conservar datos. Un *volumen* representa una manera de almacenar, recuperar y conservar datos entre pods y durante el ciclo de vida de la aplicación.

Los volúmenes tradicionales se crean como recursos de Kubernetes respaldados por Azure Storage. Puede crear manualmente volúmenes de datos para que se asignen directamente a los pods, o bien hacer que Kubernetes los cree automáticamente. Los volúmenes de datos pueden utilizar Azure Disks o Azure Files.

### <a name="azure-disks"></a>Azure Disks

Use *Azure Disks* para crear un recurso *DataDisk* de Kubernetes. Los discos pueden usar:
* Azure Premium Storage, respaldado por SSD de alto rendimiento o 
* Azure Standard Storage, respaldado por HDD normales. 

> [!TIP]
>Para la mayoría de las cargas de trabajo de producción y desarrollo, utilice el almacenamiento Premium. 

Los discos de Azure Disks se montan como *ReadWriteOnce*, por lo que solo están disponibles para un único pod. Para los volúmenes de almacenamiento a los que pueden acceder varios pods simultáneamente, use Azure Files.

### <a name="azure-files"></a>Azure Files
Use *Azure Files* para montar un recurso compartido de SMB 3.0 respaldado por una cuenta de Azure Storage en los pods. Con Azure Files, puede compartir datos entre varios nodos y pods, además de utilizar:
* Azure Premium Storage, respaldado por SSD de alto rendimiento o 
* Azure Standard Storage, respaldado por HDD normales.

### <a name="volume-types"></a>Tipos de volúmenes
Los volúmenes de Kubernetes representan algo más que un disco tradicional para almacenar y recuperar información. Los volúmenes de Kubernetes pueden utilizarse también como una forma de insertar datos en un pod para su uso en contenedores. 

Algunos tipos de volumen comunes de Kubernetes son, entre otros:

#### <a name="emptydir"></a>emptyDir

Se suele utilizar como espacio temporal para un pod. Todos los contenedores dentro de un pod pueden acceder a los datos del volumen. Los datos escritos en este tipo de volumen solo se conservan durante la vida útil del pod. Una vez eliminado el pod, también se elimina el volumen. Este volumen suele usar el almacenamiento en disco del nodo local subyacente, aunque también puede existir solo en la memoria del nodo.

#### <a name="secret"></a>secret

Los volúmenes *secret* se usan para insertar en pods datos confidenciales, como contraseñas. 
1. Cree un secreto mediante la API de Kubernetes. 
1. Defina el pod o la implementación y solicite un secreto específico. 
    * Los secretos solo se proporcionan a nodos con un pod programado que los requiera.
    * El secreto se almacena en *tmpfs*, no se escribe en el disco. 
1. Cuando se elimina el último pod de un nodo que requiere un secreto, dicho secreto se elimina del tmpfs del nodo. 
   * Los secretos se almacenan en un espacio de nombres determinado y solo son accesibles para los pods del mismo espacio de nombres.

#### <a name="configmap"></a>configMap

Los volúmenes *configMap* pueden usarse para insertar en pods propiedades de pares clave-valor, como la información de configuración de la aplicación. Defina esta información como un recurso de Kubernetes que se actualice fácilmente y se aplique a nuevas instancias de pods a medida que se implementen. 

Al igual que al usar un secreto, debe:
1. crear un volumen ConfigMap mediante la API de Kubernetes y 
1. solicitar el volumen ConfigMap al definir un pod o una implementación. 
   * Los volúmenes ConfigMap se almacenan en un espacio de nombres determinado y solo son accesibles para los pods del mismo espacio de nombres.

## <a name="persistent-volumes"></a>Volúmenes persistentes

Los volúmenes definidos y creados como parte del ciclo de vida del pod solo existen hasta que se elimina este. Los pods suelen esperar que su almacenamiento se conserve si un pod se vuelve a programar en un host diferente durante un evento de mantenimiento, especialmente en StatefulSets. Un *volumen persistente* es un recurso de almacenamiento creado y administrado por la API de Kubernetes, que puede existir más allá de la duración de un pod individual.

Puede usar Azure Disks o Azure Files para proporcionar el volumen PersistentVolume. Como se ha indicado en la sección [Volúmenes](#volumes), la elección de Azure Disks o Azure Files suele venir determinada por la necesidad de acceso simultáneo a los datos o al nivel de rendimiento.

![Volúmenes persistentes en un clúster de Azure Kubernetes Service (AKS)](media/concepts-storage/persistent-volumes.png)

Un volumen PersistentVolume puede crearlo *de forma estática* un administrador de clústeres o *de forma dinámica* el servidor de API de Kubernetes. Si un pod está programado y solicita almacenamiento que no está disponible actualmente, Kubernetes puede crear el almacenamiento subyacente de Azure Disks o Azure Files y conectarlo al pod. El aprovisionamiento dinámico usa *StorageClass* para identificar el tipo de almacenamiento que Azure Storage debe crear.

## <a name="storage-classes"></a>Clases de almacenamiento

Para definir niveles de almacenamiento diferentes, como Premium y Estándar, puede crear una clase *StorageClass*. 

La clase StorageClass también define la directiva *reclaimPolicy*. Si se elimina el pod y el volumen persistente deja de ser necesario, la directiva reclaimPolicy controla el comportamiento del recurso de almacenamiento subyacente de Azure. Este recurso puede eliminarse o conservarse para usarse con un pod futuro.

En AKS, se crean cuatro `StorageClasses` iniciales para el clúster mediante los complementos de almacenamiento en árbol:

| Permiso | Motivo |
|---|---|
| `default` | Usa almacenamiento de SSD estándar de Azure para crear un disco administrado. La directiva de reclamación garantiza que el almacenamiento de Azure Disks subyacente se elimina cuando se elimina el volumen persistente que lo utiliza. |
| `managed-premium` | Usa Azure Premium Storage para crear un disco administrado. De nuevo, la directiva de reclamación garantiza que el almacenamiento de Azure Disks subyacente se elimina cuando se elimina el volumen persistente que lo utiliza. |
| `azurefile` | Usa Azure Standard Storage para crear un recurso compartido de archivos de Azure. La directiva de reclamación garantiza que el recurso compartido de archivos de Azure subyacente se elimina cuando se elimina el volumen persistente que lo usa. |
| `azurefile-premium` | Usa Azure Premium Storage para crear un recurso compartido de archivos de Azure. La directiva de reclamación garantiza que el recurso compartido de archivos de Azure subyacente se elimina cuando se elimina el volumen persistente que lo usa.|

En el caso de los clústeres que usan los nuevos complementos externos de Container Storage Interface (CSI, versión preliminar), se crean las siguientes clases `StorageClasses` extras:

| Permiso | Motivo |
|---|---|
| `managed-csi` | Usa almacenamiento con redundancia local de SSD Estándar de Azure para crear un disco administrado. La directiva de reclamación garantiza que el almacenamiento de Azure Disks subyacente se elimina cuando se elimina el volumen persistente que lo utiliza. La clase de almacenamiento también configura los volúmenes persistentes para que se puedan expandir, solo es necesario editar la notificación de volumen persistente con el nuevo tamaño. |
| `managed-csi-premium` | Usa almacenamiento con redundancia local Premium de Azure para crear un disco administrado. De nuevo, la directiva de reclamación garantiza que el almacenamiento de Azure Disks subyacente se elimina cuando se elimina el volumen persistente que lo utiliza. De forma similar, esta clase de almacenamiento permite expandir los volúmenes persistentes. |
| `azurefile-csi` | Usa Azure Standard Storage para crear un recurso compartido de archivos de Azure. La directiva de reclamación garantiza que el recurso compartido de archivos de Azure subyacente se elimina cuando se elimina el volumen persistente que lo usa. |
| `azurefile-csi-premium` | Usa Azure Premium Storage para crear un recurso compartido de archivos de Azure. La directiva de reclamación garantiza que el recurso compartido de archivos de Azure subyacente se elimina cuando se elimina el volumen persistente que lo usa.|

A menos que especifique una clase StorageClass para un volumen persistente, se usará la clase StorageClass predeterminada. Cuando solicite volúmenes persistentes, asegúrese de que usen el almacenamiento adecuado que necesita. 

Puede crear una clase StorageClass para satisfacer necesidades adicionales mediante `kubectl`. En el siguiente ejemplo se utilizan discos administrados Premium y se especifica que el disco subyacente de Azure Disks debe *conservarse* cuando se elimine el pod:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-retain
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

> [!NOTE]
> AKS concilia las clases de almacenamiento predeterminadas y sobrescribirá los cambios que realice en ellas.

## <a name="persistent-volume-claims"></a>Notificaciones de volúmenes persistentes

Una notificación PersistentVolumeClaim solicita almacenamiento en disco o archivo de una clase StorageClass, un modo de acceso y un tamaño determinados. El servidor de API de Kubernetes puede aprovisionar dinámicamente el recurso de almacenamiento subyacente de Azure si no existe ningún recurso para satisfacer la notificación de acuerdo con la clase StorageClass definida. 

La definición del pod incluye el montaje del volumen una vez que se este se ha conectado al pod.

![Notificaciones de volúmenes persistentes en un clúster de Azure Kubernetes Service (AKS)](media/concepts-storage/persistent-volume-claims.png)

Una vez que se ha asignado un recurso de almacenamiento disponible al pod que lo solicita, se *enlaza* un volumen PersistentVolume a una notificación PersistentVolumeClaim. Los volúmenes persistentes se asignan a las notificaciones de uno a uno.

El manifiesto YAML de ejemplo siguiente muestra una notificación de volumen persistente que usa la clase StorageClass *managed-premium* y solicita un disco de *5 Gi* de tamaño:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

Al crear una definición de pod, también se especifica lo siguiente:
* La notificación de volumen persistente para solicitar el almacenamiento deseado 
* El objeto *volumeMount* para que sus aplicaciones lean y escriban datos 

El siguiente manifiesto YAML de ejemplo muestra cómo se puede usar la notificación de volumen persistente anterior para montar un volumen en */mnt/azure*:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: myfrontend
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      volumeMounts:
      - mountPath: "/mnt/azure"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

Para montar un volumen en un contenedor de Windows, especifique la letra de unidad y la ruta de acceso. Por ejemplo:

```yaml
...      
       volumeMounts:
        - mountPath: "d:"
          name: volume
        - mountPath: "c:\k"
          name: k-dir
...
```

## <a name="next-steps"></a>Pasos siguientes

Para consultar los procedimientos recomendados asociados, consulte [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Para ver cómo crear volúmenes dinámicos y estáticos que usen Azure Disks o Azure Files, consulte los artículos de procedimientos siguientes:

- [Creación de un volumen estático con Azure Disks][aks-static-disks]
- [Creación de un volumen estático con Azure Files][aks-static-files]
- [Creación de un volumen dinámico con Azure Disks][aks-dynamic-disks]
- [Creación de un volumen dinámico con Azure Files][aks-dynamic-files]

Para obtener más información sobre los conceptos básicos de Kubernetes y AKS, consulte los artículos siguientes:

- [Clústeres y cargas de trabajo de Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Identidad de Kubernetes/AKS][aks-concepts-identity]
- [Seguridad de Kubernetes/AKS][aks-concepts-security]
- [Redes virtuales de Kubernetes/AKS][aks-concepts-network]
- [Escala de Kubernetes/AKS][aks-concepts-scale]

<!-- EXTERNAL LINKS -->

<!-- INTERNAL LINKS -->
[aks-static-disks]: azure-disk-volume.md
[aks-static-files]: azure-files-volume.md
[aks-dynamic-disks]: azure-disks-dynamic-pv.md
[aks-dynamic-files]: azure-files-dynamic-pv.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-network]: concepts-network.md
[operator-best-practices-storage]: operator-best-practices-storage.md
