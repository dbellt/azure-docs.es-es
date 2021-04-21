---
title: 'Inicio rápido: Implementación de un clúster de AKS con nodos de computación confidencial mediante la CLI de Azure'
description: Aprenda a crear un clúster de Azure Kubernetes Service (AKS) con nodos confidenciales e implementar una aplicación Hola mundo mediante la CLI de Azure.
author: agowdamsft
ms.service: container-service
ms.subservice: confidential-computing
ms.topic: quickstart
ms.date: 04/08/2021
ms.author: amgowda
ms.custom: contentperf-fy21q3
ms.openlocfilehash: b012a8a5856b344b366f1ddd89fc5059a6f3c8ae
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107283531"
---
# <a name="quickstart-deploy-an-aks-cluster-with-confidential-computing-nodes-by-using-the-azure-cli"></a>Inicio rápido: Implementación de un clúster de AKS con nodos de computación confidencial mediante la CLI de Azure

En este inicio rápido, va a usar la CLI de Azure para implementar un clúster de Azure Kubernetes Service (AKS) con nodos de computación confidencial (DCsv2). A continuación, ejecutará una aplicación Hola mundo en un enclave. También puede aprovisionar un clúster y agregar nodos de computación confidencial desde Azure Portal, pero este inicio rápido se centra en la CLI de Azure.

AKS es un servicio de Kubernetes administrado que permite a los desarrolladores y operadores del clúster implementar y administrar clústeres rápidamente. Para más información, consulte [Azure Kubernetes Service](../aks/intro-kubernetes.md) y [Nodos de computación confidencial en Azure Kubernetes Service](confidential-nodes-aks-overview.md).

Las características de los nodos de computación confidencial incluyen:

- Nodos de trabajo Linux que admiten contenedores Linux.
- Máquina virtual de generación 2 con nodos de máquina virtual con Ubuntu 18.04.
- CPU compatible con Intel SGX para ayudar a ejecutar los contenedores en un enclave protegido por confidencialidad que aprovecha la memoria caché de páginas cifrada (EPC). Para más información, consulte [Preguntas frecuentes de computación confidencial de Azure](./faq.md).
- Controlador Intel SGX DCAP preinstalado en los nodos de computación confidencial. Para más información, consulte [Preguntas frecuentes de computación confidencial de Azure](./faq.md).

> [!NOTE]
> Las máquinas virtuales de la serie DCsv2 utilizan hardware especializado que está sujeto a precios más altos y disponibilidad de regiones. Para más información, consulte [Soluciones en máquinas virtuales de Azure](virtual-machine-solutions.md).

## <a name="prerequisites"></a>Requisitos previos

Esta guía de inicio rápido requiere:

- Una suscripción de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- La versión 2.0.64 de la CLI de Azure, o cualquier versión posterior, instalada y configurada en la máquina de implementación. 

  Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](../container-registry/container-registry-get-started-azure-cli.md).
- Un mínimo de seis núcleos DCsv2 disponibles en la suscripción. 

  De manera predeterminada, la cuota para computación confidencial por cada suscripción de Azure es de ocho núcleos de máquina virtual. Si planea aprovisionar un clúster que requiera más de ocho núcleos, siga [estas instrucciones](../azure-portal/supportability/per-vm-quota-requests.md) para generar una incidencia de aumento de cuota.

## <a name="create-an-aks-cluster-with-confidential-computing-nodes-and-add-on"></a>Creación de un clúster de AKS con nodos y complementos de computación confidencial

Utilice las instrucciones siguientes para crear un clúster de AKS con el complemento de computación confidencial habilitado, agregar un grupo de nodos al clúster y comprobar lo que ha creado.

### <a name="create-an-aks-cluster-with-a-system-node-pool"></a>Creación de un clúster de AKS con un grupo de nodos del sistema

> [!NOTE]
> Si ya tiene un clúster de AKS que cumple los criterios de requisitos previos enumerados anteriormente, [vaya a la siguiente sección](#add-a-user-node-pool-with-confidential-computing-capabilities-to-the-aks-cluster) para agregar un nuevo grupo de nodos de computación confidencial.

Primero, cree un grupo de recursos para el clúster con el comando [az group create][az-group-create]. En el ejemplo siguiente se crea un grupo de recursos llamado *myResourceGroup* en la región *westus2*:

```azurecli-interactive
az group create --name myResourceGroup --location westus2
```

Ahora cree un clúster de AKS con el complemento de computación confidencial habilitado mediante el comando[az aks create][az-aks-create]:

```azurecli-interactive
az aks create -g myResourceGroup --name myAKSCluster --generate-ssh-keys --enable-addon confcom
```

### <a name="add-a-user-node-pool-with-confidential-computing-capabilities-to-the-aks-cluster"></a>Adición de un grupo de nodos de usuario con funcionalidades de computación confidencial al clúster de AKS 

Ejecute el siguiente comando para agregar un grupo de nodos de usuario de tamaño `Standard_DC2s_v2` con tres nodos al clúster de AKS. Puede elegir otra SKU de la [lista de SKU de la serie DCsv2 y regiones admitidas](../virtual-machines/dcv2-series.md).

```azurecli-interactive
az aks nodepool add --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup --node-vm-size Standard_DC2s_v2
```

Después de ejecutar el comando, debe estar visible un nuevo grupo de nodos con DCsv2 con los DaemonSets del complemento de computación confidencial ([complemento de dispositivo SGX](confidential-nodes-aks-overview.md#confidential-computing-add-on-for-aks)).

### <a name="verify-the-node-pool-and-add-on"></a>Comprobación del complemento y del grupo de nodos

Obtenga las credenciales del clúster de AKS mediante el comando [az aks get-credentials][az-aks-get-credentials]:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Use el comando `kubectl get pods` para comprobar que los nodos se han creado correctamente y que los DaemonSets relacionados con SGX están en ejecución en los grupos de nodos DCsv2:

```console
$ kubectl get pods --all-namespaces

kube-system     sgx-device-plugin-xxxx     1/1     Running
```

Si la salida coincide con el código anterior, el clúster de AKS ya está listo para ejecutar aplicaciones confidenciales.

Puede ir a la sección [Implementación de Hola mundo desde una aplicación de enclave aislado](#hello-world) de este inicio rápido para probar una aplicación en un enclave. O bien, use las instrucciones siguientes para agregar más grupos de nodos en AKS. (AKS admite la combinación de grupos de nodos SGX y grupos de nodos que no son SGX).

## <a name="add-a-confidential-computing-node-pool-to-an-existing-aks-cluster"></a>Adición de un grupo de nodos de computación confidencial a un clúster de AKS existente<a id="existing-cluster"></a>

En esta sección se supone que ya tiene un clúster de AKS en ejecución que cumple los criterios de requisitos previos enumerados anteriormente en este inicio rápido.

### <a name="enable-the-confidential-computing-aks-add-on-on-the-existing-cluster"></a>Habilitación del complemento de AKS de computación confidencial en el clúster existente

Ejecute el siguiente comando para habilitar el complemento de computación confidencial:

```azurecli-interactive
az aks enable-addons --addons confcom --name MyManagedCluster --resource-group MyResourceGroup 
```

### <a name="add-a-dcsv2-user-node-pool-to-the-cluster"></a>Agregue un grupo de nodos de usuario DCsv2 al clúster

> [!NOTE]
> Para usar la funcionalidad de computación confidencial, el clúster de AKS existente debe tener al menos un grupo de nodos basado en una SKU de máquina virtual DCsv2. Para más información sobre las SKU de las máquinas virtuales DCs-v2 para computación confidencial, consulte las [SKU disponibles y regiones admitidas](virtual-machine-solutions.md).

Ejecute el siguiente comando para crear un grupo de nodos:

```azurecli-interactive
az aks nodepool add --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup --node-count 1 --node-vm-size Standard_DC4s_v2
```

Compruebe que se ha creado el nuevo grupo de nodos con el nombre *confcompool1*:

```azurecli-interactive
az aks nodepool list --cluster-name myAKSCluster --resource-group myResourceGroup
```

### <a name="verify-that-daemonsets-are-running-on-confidential-node-pools"></a>Comprobación de que los DaemonSets están en ejecución en los grupos de nodos confidenciales

Inicie sesión en el clúster de AKS existente para realizar la siguiente comprobación:

```console
kubectl get nodes
```

La salida debe mostrar el grupo *confcompool1* recién agregado al clúster de AKS. También puede ver otros DaemonSets.

```console
$ kubectl get pods --all-namespaces

kube-system     sgx-device-plugin-xxxx     1/1     Running
```

Si la salida coincide con el código anterior, el clúster de AKS ya está listo para ejecutar aplicaciones confidenciales. 

## <a name="deploy-hello-world-from-an-isolated-enclave-application"></a>Implementación de Hola mundo desde una aplicación de enclave aislado <a id="hello-world"></a>
Ahora está listo para implementar una aplicación de prueba. 

Cree un archivo llamado *hello-world-enclave.yaml* y pegue el siguiente manifiesto YAML. Puede encontrar este código de aplicación de ejemplo en [Proyecto Open Enclave](https://github.com/openenclave/openenclave/tree/master/samples/helloworld). En esta implementación se da por supuesto que ha implementado el complemento *confcom*.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sgx-test
  labels:
    app: sgx-test
spec:
  template:
    metadata:
      labels:
        app: sgx-test
    spec:
      containers:
      - name: sgxtest
        image: oeciteam/sgx-test:1.0
        resources:
          limits:
            sgx.intel.com/epc: 5Mi # This limit will automatically place the job into a confidential computing node and mount the required driver volumes. Alternatively, you can target deployment to node pools with node selector.
      restartPolicy: Never
  backoffLimit: 0
  ```

Ahora, use el comando `kubectl apply` para crear un trabajo de ejemplo que se abrirá en un enclave seguro, como se muestra en la siguiente salida de ejemplo:

```console
$ kubectl apply -f hello-world-enclave.yaml

job "sgx-test" created
```

Puede confirmar que la carga de trabajo ha creado correctamente un entorno de ejecución de confianza (enclave) mediante la ejecución de los siguientes comandos:

```console
$ kubectl get jobs -l app=sgx-test

NAME       COMPLETIONS   DURATION   AGE
sgx-test   1/1           1s         23s
```

```console
$ kubectl get pods -l app=sgx-test

NAME             READY   STATUS      RESTARTS   AGE
sgx-test-rchvg   0/1     Completed   0          25s
```

```console
$ kubectl logs -l app=sgx-test

Hello world from the enclave
Enclave called into host to print: Hello World!
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar el grupo de nodos de computación confidencial que ha creado en este inicio rápido, use el siguiente comando: 

```azurecli-interactive
az aks nodepool delete --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup
```

Utilice el comando siguiente para eliminar el clúster de AKS: 

```azurecli-interactive
az aks delete --resource-group myResourceGroup --name myAKSCluster
```

## <a name="next-steps"></a>Pasos siguientes

* Ejecute Python, Node u otras aplicaciones mediante contenedores confidenciales con los [ejemplos de contenedores confidenciales de GitHub](https://github.com/Azure-Samples/confidential-container-samples).

* Para ejecutar aplicaciones compatibles con enclaves, consulte los [ejemplos de contenedores de Azure compatibles con enclaves en GitHub](https://github.com/Azure-Samples/confidential-computing/blob/main/containersamples/).

<!-- LINKS -->
[az-group-create]: /cli/azure/group#az_group_create
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
