---
title: 'Conceptos: aspectos básicos de Kubernetes para Azure Kubernetes Services (AKS)'
description: Obtenga información sobre el clúster básico y los componentes de carga de trabajo de Kubernetes y cómo se relacionan con las características de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/05/2020
ms.openlocfilehash: 5e505ed44d221b20178ea5ffb1d9125fb2bddd4c
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105942"
---
# <a name="kubernetes-core-concepts-for-azure-kubernetes-service-aks"></a>Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)

El desarrollo de aplicaciones continúa avanzando hacia un enfoque basado en contenedores, lo que aumenta nuestra necesidad de orquestar y administrar recursos. Como plataforma líder, Kubernetes proporciona una programación de confianza de cargas de trabajo de aplicaciones tolerantes a errores. Azure Kubernetes Service (AKS) es una oferta administrada de Kubernetes que simplifica aún más la implementación y administración de aplicaciones basadas en contenedores.

En este artículo se presentan:
* Componentes básicos de la infraestructura de Kubernetes:
    * *Plano de control*
    * *Nodos*
    * *Grupos de nodos*
* Recursos de carga de trabajo: 
    * *Pods*
    * *deployments*
    * *Conjuntos* 
* Cómo agrupar recursos en *espacios de nombres*.

## <a name="what-is-kubernetes"></a>¿Qué es Kubernetes?

Kubernetes es una plataforma de rápida evolución que administra aplicaciones basadas en contenedores y sus componentes de red y almacenamiento asociados. Se centra en las cargas de trabajo de la aplicación, no en los componentes subyacentes de la infraestructura. Kubernetes proporciona un enfoque declarativo en las implementaciones, respaldado por un sólido conjunto de API para las operaciones de administración.

Usted puede compilar y ejecutar aplicaciones modernas, portátiles y basadas en microservicios, usando Kubernetes para orquestar y administrar la disponibilidad de sus componentes. Kubernetes admite tanto aplicaciones con estado como sin estado, a medida que los equipos progresan a través de la adopción de aplicaciones basadas en microservicios.

Como plataforma abierta, Kubernetes le permite compilar aplicaciones con el lenguaje de programación, el sistema operativo, las bibliotecas o el bus de mensajería que prefiera. La integración continua y las herramientas de entrega continua (CI/CD) existentes pueden integrarse con Kubernetes para programar e implementar versiones.

AKS proporciona un servicio administrado de Kubernetes que reduce la complejidad de la implementación y de tareas básicas de administración, como la coordinación de actualizaciones. La plataforma Azure administra el plano de control de AKS, y usted solo paga los nodos de AKS que ejecutan sus aplicaciones. AKS se ha diseñado sobre el motor de código abierto de Azure Kubernetes Service ([aks-engine][aks-engine]).

## <a name="kubernetes-cluster-architecture"></a>Arquitectura del clúster de Kubernetes

Un clúster de Kubernetes se divide en dos componentes:

- *Plano de control*: proporciona los servicios básicos de Kubernetes y la orquestación de las cargas de trabajo de las aplicaciones.
- *Nodos*: ejecutan las cargas de trabajo de las aplicaciones.

![Plano de control de Kubernetes y componentes de nodo](media/concepts-clusters-workloads/control-plane-and-nodes.png)

## <a name="control-plane"></a>Plano de control

Cuando crea un clúster AKS, se crea y configura automáticamente un plano de control. Este plano de control se proporciona sin coste alguno como recurso administrado de Azure que se extrae del usuario. Usted solo paga los nodos asociados al clúster de AKS. El plano de control y sus recursos solo residen en la región en la que creó el clúster.

El plano de control incluye los siguientes componentes principales de Kubernetes:

| Componente | Descripción |  
| ----------------- | ------------- |  
| *kube-apiserver*                                                                                 | El servidor de API es el modo en el que se exponen las API de Kubernetes subyacentes. Este componente proporciona la interacción de las herramientas de administración, como `kubectl` o el panel de Kubernetes.                                                        |  
| *etcd* | Para mantener el estado del clúster de Kubernetes y la configuración, el componente *etcd* de alta disponibilidad es un almacén de pares clave-valor en Kubernetes.                                      |  
| *kube-scheduler*                                                                            | Al crear o escalar aplicaciones, Scheduler determina qué nodos pueden ejecutar la carga de trabajo y los inicia.                                                                                    |  
| *kube-controller-manager*                                                                            | El administrador de controladores supervisa un número de controladores más pequeños que realizan acciones, como replicar pods y controlar operaciones de nodos.                                                                  |  

AKS proporciona un plano de control de inquilino único con un servidor de API dedicado, Scheduler, etc. Usted define el número de nodos y su tamaño, mientras que la plataforma Azure configura la comunicación segura entre el plano de control y los nodos. La interacción con el plano de control se produce a través de las API de Kubernetes, como `kubectl` o el panel de Kubernetes.

Aunque no es necesario configurar componentes (como el almacén *etcd* de alta disponibilidad) con este plano de control administrado, usted no puede obtener acceso al plano de control directamente. Las actualizaciones del plano de control y de los nodos de Kubernetes se orquestan a través de CLI de Azure o de Azure Portal. Para solucionar los posibles problemas, puede revisar los registros del plano de control mediante registros de Azure Monitor.

Para configurar un plano de control o acceder a este directamente, implemente su propio clúster de Kubernetes mediante [aks-engine][aks-engine].

Para los procedimientos recomendados asociados, consulte [Procedimientos recomendados para administrar la seguridad y las actualizaciones de los clústeres en Azure Kubernetes Service (AKS)][operator-best-practices-cluster-security].

## <a name="nodes-and-node-pools"></a>Nodos y grupos de nodos

Para ejecutar las aplicaciones y los servicios de soporte técnico, necesitará un *nodo* de Kubernetes. Un clúster de AKS tiene al menos un nodo, una máquina virtual de Azure que ejecuta los componentes de nodo de Kubernetes y el entorno de ejecución del contenedor.

| Componente | Descripción |  
| ----------------- | ------------- |  
| `kubelet`                                                                                 | Agente de Kubernetes que procesa las solicitudes de orquestación desde el plano de control y la programación de la ejecución de los contenedores solicitados.                                                        |  
| *kube-proxy* | Controla las redes virtuales en cada nodo. El proxy enruta el tráfico de red y administra las direcciones IP para los servicios y los pods.                                      |  
| *entorno de ejecución de contenedor*                                                                            | Permite que las aplicaciones contenedorizadas ejecuten recursos adicionales, como la red virtual y el almacenamiento, e interactúen con estos. Los clústeres de AKS que usan grupos de nodos con la versión 1.19 de Kubernetes u otras posteriores usan `containerd` como entorno de ejecución del contenedor. Los clústeres de AKS que usan grupos de nodos con versiones de Kubernetes anteriores a la 1.19 utilizan [Moby](https://mobyproject.org/) (docker ascendente) como entorno de ejecución de contenedor.                                                                                    |  


![Máquina virtual de Azure y recursos auxiliares para un nodo de Kubernetes](media/concepts-clusters-workloads/aks-node-resource-interactions.png)

El tamaño de la máquina virtual de Azure para los nodos define las CPU, la memoria, el tamaño y el tipo del almacenamiento disponible (por ejemplo, SSD de alto rendimiento o HDD normal). Planee el tamaño de los nodos en función de si las aplicaciones pueden requerir grandes cantidades de CPU y memoria o almacenamiento de alto rendimiento. Escale horizontalmente el número de nodos del clúster de AKS para satisfacer la demanda.

En AKS, la imagen de máquina virtual para los nodos del clúster se basa en Ubuntu Linux o Windows Server 2019. Al crear un clúster de AKS o escalar horizontalmente el número de nodos, la plataforma Azure crea y configura automáticamente el número solicitado de máquinas virtuales. Los nodos de agente se facturan como máquinas virtuales estándar, por lo que cualquier descuento en el tamaño de la máquina virtual (incluidas [reservas de Azure][reservation-discounts]) se aplica automáticamente.

Implemente su propio clúster de Kubernetes con [aks-engine][aks-engine] si usa un sistema operativo host diferente o un entorno de ejecución de contenedor, o bien si incluye distintos paquetes personalizados. El componente `aks-engine` ascendente incluye y ofrece opciones de configuración antes de que se admitan en los clústeres de AKS. Por lo tanto, si quiere usar un entorno de ejecución de contenedor que no sea `containerd` ni [Moby](https://mobyproject.org/), puede ejecutar `aks-engine` para configurar e implementar un clúster de Kubernetes que satisfaga sus necesidades actuales.

### <a name="resource-reservations"></a>Reservas de recursos

AKS usa recursos de nodo para ayudar al nodo a funcionar como parte del clúster. Este uso puede crear discrepancias entre los recursos totales del nodo y los recursos que se pueden asignar en AKS. Tenga esto presente al establecer solicitudes y límites para los pods implementados por el usuario.

Para buscar la ejecución de los recursos de un nodo que se pueden asignar:
```kubectl
kubectl describe node [NODE_NAME]
```

Para mantener la funcionalidad y el rendimiento de los nodos, AKS reserva recursos en cada nodo. A medida que aumentan los recursos de un nodo, la reserva de recursos también crece debido a la mayor necesidad de administrar pods implementados por el usuario.

>[!NOTE]
> El uso de complementos AKS como Container Insights (OMS) consumirá recursos de nodo adicionales.

Hay reservados dos tipos de recursos:

- **CPU**  
    La CPU reservada depende del tipo de nodo y de la configuración del clúster, lo que puede provocar que haya menos CPU para asignar, debido a la ejecución de otras características.

   | Núcleos de CPU en el host | 1    | 2    | 4    | 8    | 16 | 32|64|
   |---|---|---|---|---|---|---|---|
   |Reservado para Kube (milinúcleos)|60|100|140|180|260|420|740|

- **Memoria**  
    La memoria que utiliza AKS incluye la suma de dos valores.

   1. **Demonio `kubelet`**    
       El demonio `kubelet` se instala en todos los nodos de agente de Kubernetes para administrar la creación y terminación de contenedores. 
   
        De forma predeterminada en AKS, el demonio `kubelet` tiene la regla de expulsión *memory.available<750Mi*, que garantiza que un nodo siempre tenga al menos 750 Mi asignables en todo momento. Cuando un host esté por debajo de ese umbral de memoria disponible, se activará `kubelet` para finalizar uno de los pods en ejecución y liberar memoria en el equipo host.

   2. **Tasa regresiva de reservas de memoria** para que el demonio “kubelet” funcione correctamente (*kube-reserved*).
      - 25 % de los primeros 4 GB de memoria
      - 20 % de los siguientes 4 GB de memoria (hasta 8 GB)
      - 10 % de los siguientes 8 GB de memoria (hasta 16 GB)
      - 6 % de los siguientes 112 GB de memoria (hasta 128 GB)
      - 2 % de cualquier memoria que esté por encima de 128 GB

Las reglas de asignación de memoria y CPU:
* mantienen los nodos de agente en buen estado, incluidos algunos pods del sistema host críticos para el mantenimiento del clúster y 
* hacen que el nodo notifique un volumen menor de memoria y CPU asignables que el que notificaría si no formase parte de un clúster de Kubernetes. 

Las reservas de recursos anteriores no se pueden cambiar.

Por ejemplo, si un nodo ofrece 7 GB, informará del 34 % de la memoria no asignable, incluido el umbral de expulsión estricto de 750 Mi.

`0.75 + (0.25*4) + (0.20*3) = 0.75GB + 1GB + 0.6GB = 2.35GB / 7GB = 33.57% reserved`

Además de las reservas para Kubernetes mismo, el sistema operativo del nodo subyacente también reserva una cantidad de recursos de CPU y memoria para mantener las funciones del sistema operativo.

Para consultar los procedimientos recomendados asociados, consulte[Procedimientos recomendados para características básicas del programador en Azure Kubernetes Service (AKS)][operator-best-practices-scheduler].

### <a name="node-pools"></a>Grupos de nodos

Los nodos de la misma configuración se agrupan en *grupos de nodos*. Un clúster de Kubernetes contiene al menos un grupo de nodos. El número de nodos y el tamaño iniciales se definen al crear un clúster de AKS, que crea un *grupo de nodos predeterminado*. Este grupo de nodos predeterminado de AKS contiene las máquinas virtuales subyacentes que ejecutan los nodos del agente.

> [!NOTE]
> Para asegurarse de que el clúster funcione de forma fiable, debe ejecutar al menos dos (2) nodos del grupo predeterminado.

Los clústeres de AKS se escalan o actualizan con relación al grupo de nodos predeterminado. También se puede optar por escalar o actualizar un grupo de nodos específico. Para las operaciones de actualización, los contenedores en ejecución se programan en otros nodos del grupo de nodos hasta que todos los nodos se actualizan correctamente.

Para obtener más información sobre cómo usar varios grupos de nodos en AKS, consulte [Creación y administración de varios grupos de nodos para un clúster en AKS][use-multiple-node-pools].

### <a name="node-selectors"></a>Selectores de nodos

En un clúster de AKS con varios grupos de nodos, es posible que tenga que indicar a Scheduler de Kubernetes qué grupo de nodos se debe utilizar para un recurso determinado. Por ejemplo, los controladores de entrada no deben ejecutarse en los nodos de Windows Server. 

Los selectores de nodo le permiten definir varios parámetros, por ejemplo, el sistema operativo del nodo, para controlar dónde se debe programar un pod.

El siguiente ejemplo básico programa una instancia de NGINX en un nodo Linux mediante el selector de nodos *"beta.kubernetes.io/os": linux*:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: myfrontend
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.12-alpine
  nodeSelector:
    "beta.kubernetes.io/os": linux
```

Para más información sobre cómo controlar pods en grupos de nodos, consulte [Procedimientos recomendados para las características avanzadas del programador en AKS][operator-best-practices-advanced-scheduler].

## <a name="pods"></a>Pods

Kubernetes utiliza *pods* para ejecutar una instancia de la aplicación. Un pod representa una instancia individual de la aplicación. 

Los pods suelen tener una asignación de uno a uno con un contenedor. En escenarios avanzados, un pod puede contener varios contenedores. Los pods de varios contenedores se programan conjuntamente en el mismo nodo y permiten que los contenedores compartan recursos relacionados.

Al crear un pod, puede definir *solicitudes de recursos* para solicitar una determinada cantidad de recursos de memoria o CPU. Scheduler de Kubernetes intenta satisfacer la solicitud programando los pods para que se ejecuten en un nodo con recursos disponibles. También puede especificar límites de recursos máximos, con el fin de impedir que un pod consuma demasiados recursos de proceso del nodo subyacente. El procedimiento recomendado consiste en incluir límites de recursos para todos los pods, con el fin de ayudar a Scheduler de Kubernetes a identificar qué recursos son necesarios y cuáles se permiten.

Para obtener más información, consulte [Pods de Kubernetes][kubernetes-pods] y [Ciclo de vida de pods de Kubernetes][kubernetes-pod-lifecycle].

Un pod es un recurso lógico, pero las cargas de trabajo de aplicaciones se ejecutan en los contenedores. Los pods suelen ser recursos efímeros y descartables. Los pods programados por separado carecen de algunas de las características de alta disponibilidad y redundancia de Kubernetes. En su lugar, los pods se implementan y administran mediante *controladores* de Kubernetes, como el controlador de implementación.

## <a name="deployments-and-yaml-manifests"></a>Implementaciones y manifiestos YAML

Una *implementación* representa pods idénticos administrados por el controlador de implementación de Kubernetes. Una implementación define el número de *réplicas* de pod que deben crearse. Scheduler de Kubernetes garantiza que se programen pods adicionales en nodos correctos si los pods o los nodos tienen problemas.

Puede actualizar implementaciones para cambiar la configuración de los pods, la imagen del contenedor que se ha utilizado o el almacenamiento conectado. El controlador de implementación:
* purga y finaliza un número determinado de réplicas;
* crea réplicas a partir de la nueva definición de implementación;
* continúa el proceso hasta que se actualizan todas las réplicas de la implementación.

La mayoría de las aplicaciones sin estado de AKS debe usar el modelo de implementación en lugar de programar pods individuales. Kubernetes puede supervisar el mantenimiento de las implementaciones para garantizar que se ejecute el número de réplicas necesario dentro del clúster. Cuando se programan por separado, los pods no se reinician en caso de producirse un problema, ni tampoco se vuelven a programar en nodos correctos si el que sufre un problema es su nodo actual.

No es aconsejable trastocar decisiones de administración con un proceso de actualización si la aplicación requiere un número mínimo de instancias disponibles. Los *presupuestos de interrupción de pods* definen el número de réplicas de una implementación que se pueden quitar durante una actualización o la actualización de un nodo. Por ejemplo, si tiene *cinco (5)*  réplicas en la implementación, puede definir una interrupción del pod de *4 (cuatro)* para permitir que solo se elimine o se vuelva a programar una réplica a la vez. Como en el caso de los límites de recursos de pods, el procedimiento recomendado consiste en definir presupuestos de interrupción de pods en aplicaciones que requieran que siempre esté presente un número mínimo de réplicas.

Normalmente, las implementaciones se crean o administran con `kubectl create` o `kubectl apply`. Cree una implementación definiendo un archivo de manifiesto en formato YAML. 

En el ejemplo siguiente, se crea una implementación básica del servidor web NGINX. La implementación especifica que se crearán *tres (3)* réplicas y requiere que el puerto *80* esté abierto en el contenedor. También se definen las solicitudes de recursos y los límites de CPU y memoria.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: mcr.microsoft.com/oss/nginx/nginx:1.15.2-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 250m
            memory: 64Mi
          limits:
            cpu: 500m
            memory: 256Mi
```

Es posible crear aplicaciones más complejas incluyendo servicios (como equilibradores de carga) en el manifiesto YAML.

Para más información, consulte el artículo sobre las [implementaciones de Kubernetes][kubernetes-deployments].

### <a name="package-management-with-helm"></a>Administración de paquetes con Helm

[Helm][helm] se usa normalmente para administrar aplicaciones en Kubernetes. Puede implementar recursos compilando y usando gráficos públicos de *Helm* ya existentes que contengan una versión empaquetada de código de aplicación y manifiestos YAML de Kubernetes. Estos gráficos de Helm pueden almacenarse localmente o en un repositorio remoto, como un [repositorio de gráficos de Helm de Azure Container Registry][acr-helm].

Para usar Helm, instale el cliente de Helm en el equipo o use el cliente de Helm en [Azure Cloud Shell][azure-cloud-shell]. Busque o cree gráficos de Helm y, después, instálelos en el clúster de Kubernetes. Para más información, consulte [Instalación de aplicaciones existentes con Helm en AKS][aks-helm].

## <a name="statefulsets-and-daemonsets"></a>StatefulSets y DaemonSets

El controlador de implementación usa Scheduler de Kubernetes para ejecutar réplicas en cualquier nodo disponible que cuente con recursos. Aunque este enfoque puede bastar en el caso de aplicaciones sin estado, el controlador de implementación no es la opción ideal en el caso de aplicaciones que requieran lo siguiente:
* Una convención de nomenclatura o un almacenamiento persistentes 
* Una réplica en cada nodo seleccionado dentro de un clúster

Pero hay dos recursos de Kubernetes con los que puede administrar estos tipos de aplicaciones:

- *StatefulSets*: mantienen el estado de las aplicaciones más allá del ciclo de vida de un determinado pod, como el almacenamiento.
- *DaemonSets*: garantizan la ejecución de una instancia en cada nodo en un fase inicial del proceso de arranque de Kubernetes.

### <a name="statefulsets"></a>StatefulSets

El desarrollo de aplicaciones modernas suele tener como objetivo aplicaciones sin estado. En el caso de las aplicaciones con estado, como las que incluyen componentes de base de datos, puede usar *StatefulSets*. Al igual que las implementaciones, un StatefulSet crea y administra al menos un pod idéntico. Las réplicas de un StatefulSet siguen un enfoque estable y secuencial de implementación, escalado, actualización y finalización. Con un StatefulSet, la convención de nomenclatura, los nombres de red y el almacenamiento persisten como réplicas que se vuelven a programar.

Defina la aplicación en formato YAML usando `kind: StatefulSet`. Después, el controlador del StatefulSet controla la implementación y administración de las réplicas necesarias. Los datos se escriben en el almacenamiento persistente, proporcionado por Azure Managed Disks o Azure Files. Con un StatefulSet, el almacenamiento persistente subyacente permanece aunque se elimine el StatefulSet.

Para más información, consulte el [objeto StatefulSets de Kubernetes][kubernetes-statefulsets].

Las réplicas de StatefulSet se programan y se ejecutan en cualquier nodo disponible en un clúster de AKS. Para asegurarse de que al menos un pod del conjunto se ejecute en un nodo, use un DaemonSet en su lugar.

### <a name="daemonsets"></a>DaemonSets

Para recopilar o supervisar registros específicos, es posible que tenga que ejecutar un pod en todos los nodos o en una selección de estos. Aunque un *DaemonSet* puede usarse para implementar uno o más pods idénticos, su controlador garantiza que cada nodo especificado ejecute una instancia del pod.

El controlador de DaemonSet puede programar los pods en los nodos al principio del proceso de arranque del clúster, antes de que se haya iniciado el programador de Kubernetes predeterminado. Esta capacidad garantiza que los pods de DaemonSet se inicien antes de que se programen los pods tradicionales en una implementación o StatefulSet.

Como StatefulSets, DaemonSet se define como parte de una definición de YAML mediante `kind: DaemonSet`.

Para más información, consulte el [objeto DaemonSets de Kubernetes][kubernetes-daemonset].

> [!NOTE]
> Si usa el [complemento de nodos virtuales](virtual-nodes-cli.md#enable-virtual-nodes-addon), DaemonSets no creará los pods en el nodo virtual.

## <a name="namespaces"></a>Espacios de nombres

Los recursos de Kubernetes, como pods e implementaciones, se agrupan de forma lógica en un *espacio de nombres*, con el fin de dividir un clúster de AKS y restringir el acceso para crear, visualizar o administrar recursos. Por ejemplo, puede crear espacios de nombres para separar grupos de negocios. Los usuarios solo pueden interactuar con los recursos dentro de sus espacios de nombres asignados.

![Espacios de nombres de Kubernetes para dividir de forma lógica los recursos y las aplicaciones](media/concepts-clusters-workloads/namespaces.png)

Cuando se crea un clúster de AKS, están disponibles los siguientes espacios de nombres:

| Espacio de nombres | Descripción |  
| ----------------- | ------------- |  
| *default*                                                                                 | Espacio donde se crean los pods y las implementaciones de forma predeterminada si no se proporciona ninguno. En entornos más pequeños, puede implementar aplicaciones directamente en el espacio de nombres predeterminado sin crear separaciones lógicas adicionales. Al interactuar con la API de Kubernetes, como con `kubectl get pods`, el espacio de nombres predeterminado se utiliza cuando no se especifica ninguno.                                                        |  
| *kube-system* | Espacio donde existen los recursos básicos, por ejemplo, características de red como el DNS y el proxy, o bien el panel de Kubernetes. Normalmente, el usuario no implementa sus propias aplicaciones en este espacio de nombres.                                      |  
| *kube-public*                                                                            | Aunque no suele utilizarse, este espacio puede usarse para que los recursos sean visibles en todo el clúster y para que puedan verlos todos los usuarios.                                                                                    |  


Para más información, consulte los [espacios de nombres de Kubernetes][kubernetes-namespaces].

## <a name="next-steps"></a>Pasos siguientes

En este artículo se tratan algunos de los componentes básicos de Kubernetes y cómo se aplican a los clústeres de AKS. Para obtener más información sobre los conceptos básicos de Kubernetes y AKS, consulte los artículos siguientes:

- [Acceso a Kubernetes/AKS e identidad][aks-concepts-identity]
- [Seguridad de Kubernetes/AKS][aks-concepts-security]
- [Redes virtuales de Kubernetes/AKS][aks-concepts-network]
- [Almacenamiento de Kubernetes/AKS][aks-concepts-storage]
- [Escala de Kubernetes/AKS][aks-concepts-scale]

<!-- EXTERNAL LINKS -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubernetes-pods]: https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
[kubernetes-pod-lifecycle]: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
[kubernetes-deployments]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
[kubernetes-statefulsets]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
[kubernetes-daemonset]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
[kubernetes-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[helm]: https://helm.sh/
[azure-cloud-shell]: https://shell.azure.com

<!-- INTERNAL LINKS -->
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[acr-helm]: ../container-registry/container-registry-helm-repos.md
[aks-helm]: kubernetes-helm.md
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[operator-best-practices-scheduler]: operator-best-practices-scheduler.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[reservation-discounts]:../cost-management-billing/reservations/save-compute-costs-reservations.md
