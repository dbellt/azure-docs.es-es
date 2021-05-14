---
title: Compatibilidad de la atestación con el DaemonSet auxiliar de cita de Intel SGX en Azure (versión preliminar)
description: DaemonSet para generar citas fuera del proceso de la aplicación de Intel SGX. En este artículo se explica cómo se proporciona la utilidad de atestación fuera del proceso para cargas de trabajo confidenciales que se ejecutan dentro de un contenedor.
ms.service: container-service
ms.subservice: confidential-computing
author: agowdamsft
ms.topic: overview
ms.date: 2/12/2021
ms.author: amgowda
ms.openlocfilehash: 849fd7afa3f9365f31ee8e03d9f9cc2174d64304
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484411"
---
# <a name="platform-software-management-with-intel-sgx-quote-helper-daemonset-preview"></a>Administración del software de la plataforma con el controlador DaemonSet auxiliar de cita de Intel SGX (versión preliminar)

Las [aplicaciones de enclave](confidential-computing-enclaves.md) que realizan la atestación remota requieren que se genere una cita. Esta cita proporciona una prueba criptográfica de la identidad y del estado de la aplicación, así como el entorno en el que se ejecuta el enclave. La generación de la cita requiere componentes de software de confianza que formen parte de los componentes de software de plataforma (PSW) de Intel.

## <a name="overview"></a>Información general
 
Intel admite dos modos de atestación para ejecutar la generación de citas:

- *En proceso*: hospeda los componentes de software de confianza que hay en el proceso de la aplicación de enclave.

- *Fuera del proceso*: hospeda los componentes de software de confianza que están fuera del proceso de la aplicación de enclave.
 
Las aplicaciones de la extensión Intel Software Guard (Intel SGX) creadas mediante el SDK de Open Enclave usan el modo de atestación "En proceso" de forma predeterminada. Las aplicaciones basadas en Intel SGX permiten el modo de atestación "fuera del proceso". Si desea usar este modo, necesita hospedaje adicional y debe exponer los componentes necesarios, como Architectural Enclave Service Manager (AESM), externos a la aplicación.

El uso de esta característica mejora el tiempo de actividad de las aplicaciones de enclave durante las actualizaciones de la plataforma Intel o de los controladores de DCAP. Por este motivo, se recomienda su uso.

Para habilitar esta característica en un clúster de Azure Kubernetes Services (AKS), agregue el comando `--enable-sgxquotehelper` a la CLI de Azure al habilitar el complemento de computación confidencial. 

```azurecli-interactive
# Create a new AKS cluster with system node pool with Confidential Computing addon enabled and SGX Quote Helper
az aks create -g myResourceGroup --name myAKSCluster --generate-ssh-keys --enable-addon confcom --enable-sgxquotehelper
```

Para más información, consulte [Inicio rápido: Implementación de un clúster de AKS con nodos de computación confidencial mediante la CLI de Azure](confidential-nodes-aks-get-started.md).

## <a name="benefits-of-the-out-of-process-mode"></a>Ventajas del modo "fuera del proceso"

En la lista siguiente se describen algunas de las principales ventajas de este modo de atestación:

-   No se requieren actualizaciones de los componentes de generación de citas de PSW para cada aplicación contenedorizada. Los propietarios de los contenedores no necesitan administrar las actualizaciones de su contenedor. En su lugar, los propietarios de los contenedores confían en la interfaz del proveedor, que invoca el servicio centralizado fuera del contenedor. El proveedor actualiza y administra el contenedor.

-   No hace falta que se preocupe por los errores de atestación debidos a componentes de PSW desactualizados. El proveedor administra las actualizaciones de estos componentes.

-   El modo "fuera del proceso" proporciona un mejor uso de la memoria EPC que el modo "En proceso". En el modo "En proceso", cada aplicación de enclave necesita crear una instancia de la copia de QE y PCE para la atestación remota. Con el modo "fuera del proceso", no es preciso que el contenedor hospede esos enclaves, por lo que no consume memoria de enclave de la cuota del contenedor.

-   Cuando el controlador de Intel SGX se transmite a un kernel de Linux, se requerirá que el enclave tenga privilegios más elevados. Este privilegio permite que el enclave invoque a PCE, lo que interrumpirá la aplicación de enclave que se ejecuta en el modo "En proceso". De forma predeterminada, los enclaves no obtienen este permiso. La concesión de este privilegio a una aplicación de enclave requiere cambios en el proceso de instalación de la misma. Por el contrario, en el modo "fuera del proceso", el proveedor del servicio que controla las solicitudes fuera de proceso garantiza que el servicio se instala con este privilegio.

-   No es preciso comprobar la compatibilidad con versiones anteriores de PSW y DCAP. Antes de la actualización, el proveedor valida que las actualizaciones de los componentes de generación de citas de PSW sean compatibles con versiones anteriores. Esto le ayuda a solucionar los problemas de compatibilidad antes de implementar las actualizaciones para cargas de trabajo confidenciales.

## <a name="confidential-workloads"></a>Cargas de trabajo confidenciales

El solicitante de la cita y la generación de la cita se ejecutan por separado, pero en la misma máquina física. La generación de citas está centralizada y atiende solicitudes de citas de todas las entidades. Es preciso que la interfaz se defina y pueda detectarse correctamente en cualquier entidad para que solicite citas.

![Diagrama que muestra las relaciones entre el solicitante de la cita, la generación de citas y la interfaz.](./media/confidential-nodes-out-of-proc-attestation/aesmmanager.png)

Este modelo abstracto se aplica a un escenario de carga de trabajo confidencial aprovechando el servicio AESM que ya está disponible. AESM se encuentra en un contenedor y se implementa como un DaemonSet en el clúster de Kubernetes. Kubernetes garantiza que, en cada nodo de agente, se implementará una única instancia de un contenedor de servicios AESM, encapsulada en un pod. El nuevo DaemonSet de citas de Intel SGX tendrá una dependencia del DaemonSet sgx-device-plugin, ya que el contenedor del servicio AESM solicitará memoria EPC a sgx-device-plugin para iniciar los enclaves QE y PCE.

Todos los contenedores deben participar para usar la generación de citas fuera del proceso. Para ello, es preciso establecer la variable de entorno `SGX_AESM_ADDR=1` durante la creación. El contenedor también debe incluir el paquete libsgx-quote-ex, que es el responsable de dirigir la solicitud al socket de dominio Unix predeterminado.

Las aplicaciones pueden seguir usando la atestación "en proceso" como antes, pero los modos "en proceso" y "fuera del proceso" no se pueden usar simultáneamente en una aplicación. La infraestructura del modo "fuera del proceso" está disponible de forma predeterminada y consume recursos.

## <a name="sample-implementation"></a>Implementación de ejemplo

El siguiente archivo de Docker es un ejemplo de una aplicación basada en Open Enclave. Establezca la variable de entorno `SGX_AESM_ADDR=1` del archivo de Docker, o bien establézcala en el archivo de implementación. En el ejemplo siguiente se proporcionan detalles para el archivo de Docker y la implementación. 

  > [!Note] 
  > Para que el modo "fuera del proceso" funcione correctamente, es necesario que el paquete libsgx-quote-ex de Intel se empaquete en el contenedor de la aplicación.
    
```yaml
# Refer to Intel_SGX_Installation_Guide_Linux for details
FROM ubuntu:18.04 as sgx_base
RUN apt-get update && apt-get install -y \
    wget \
    gnupg

# Add the repository to sources, and add the key to the list of
# trusted keys used by the apt to authenticate packages
RUN echo "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu bionic main" | tee /etc/apt/sources.list.d/intel-sgx.list \
    && wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | apt-key add -
# Add Microsoft repo for az-dcap-client
RUN echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main" | tee /etc/apt/sources.list.d/msprod.list \
    && wget -qO - https://packages.microsoft.com/keys/microsoft.asc | apt-key add -

FROM sgx_base as sgx_sample
RUN apt-get update && apt-get install -y \
    clang-7 \
    libssl-dev \
    gdb \
    libprotobuf10 \
    libsgx-dcap-ql \
    libsgx-quote-ex \
    az-dcap-client \
    open-enclave
WORKDIR /opt/openenclave/share/openenclave/samples/remote_attestation
RUN . /opt/openenclave/share/openenclave/openenclaverc \
    && make build
# This sets the flag for out-of-process attestation mode. Alternatively you can set this flag on the deployment files.
ENV SGX_AESM_ADDR=1 

CMD make run
```
Como alternativa, puede establecer el modo de atestación "fuera del proceso" en el archivo .yaml de implementación. A continuación, se indica cómo puede hacerlo.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sgx-test
spec:
  template:
    spec:
      containers:
      - name: sgxtest
        image: <registry>/<repository>:<version>
        env:
        - name: SGX_AESM_ADDR
          value: 1
        resources:
          limits:
            kubernetes.azure.com/sgx_epc_mem_in_MiB: 10
        volumeMounts:
        - name: var-run-aesmd
          mountPath: /var/run/aesmd
      restartPolicy: "Never"
      volumes:
      - name: var-run-aesmd
        hostPath:
          path: /var/run/aesmd
```

## <a name="next-steps"></a>Pasos siguientes

[Inicio rápido: Implementación de un clúster de AKS con nodos de computación confidencial mediante la CLI de Azure](./confidential-nodes-aks-get-started.md)

[Inicio rápido con ejemplos de contenedores confidenciales](https://github.com/Azure-Samples/confidential-container-samples)

[SKU de DCsv2](../virtual-machines/dcv2-series.md)
