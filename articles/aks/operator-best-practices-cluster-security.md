---
title: Procedimientos recomendados para la seguridad de los clústeres
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre los procedimientos recomendados del operador de clústeres para saber cómo administrar la seguridad y las actualizaciones de los clústeres en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: 5cb103d843aafbb7f72c03d65b45fe3a84f8d1cd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782988"
---
# <a name="best-practices-for-cluster-security-and-upgrades-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para administrar la seguridad y las actualizaciones de los clústeres en Azure Kubernetes Service (AKS)

A medida que administra los clústeres en Azure Kubernetes Service (AKS), es clave considerar la seguridad de los datos y de las cargas de trabajo. Cuando ejecuta clústeres multiinquilino con aislamiento lógico, debe proteger especialmente el acceso a los recursos y a las cargas de trabajo. Minimice el riesgo de ataque mediante la aplicación de las actualizaciones de seguridad más recientes de Kubernetes y del sistema operativo del nodo.

En este artículo se indica cómo proteger el clúster de AKS. Aprenderá a:

> [!div class="checklist"]
> * Usar Azure Active Directory y el control de acceso basado en rol de Kubernetes (RBAC de Kubernetes) para proteger el acceso al servidor de API.
> * Proteger el acceso del contenedor a los recursos del nodo.
> * Actualizar un clúster de AKS a la última versión de Kubernetes.
> * Mantener actualizados los nodos y aplicar automáticamente los parches de seguridad.

También puede leer las prácticas recomendadas para la [administración de imágenes de contenedor][best-practices-container-image-management] y para [seguridad de pod][best-practices-pod-security].

También puede usar la [integración de Azure Kubernetes Services con Security Center][security-center-aks] para ayudar a detectar amenazas y ver recomendaciones para proteger los clústeres de AKS.

## <a name="secure-access-to-the-api-server-and-cluster-nodes"></a>Proteger el acceso a los nodos del clúster y al servidor de API

> **Guía de procedimientos recomendados** 
>
> Una de las formas más importantes de proteger el clúster es proteger el acceso al servidor de API de Kubernetes. Integre el control de acceso basado en rol de Kubernetes con Azure Active Directory para controlar el acceso al servidor de API. Con estos controles puede proteger AKS del mismo modo que protege el acceso a las suscripciones a Azure.

El servidor de API de Kubernetes proporciona un punto de conexión único para las solicitudes para realizar acciones dentro de un clúster. Para proteger y auditar el acceso al servidor de API, limite el acceso y proporcione los niveles de permiso más bajos posibles. Aunque este enfoque no es exclusivo de Kubernetes, es especialmente importante cuando ha aislado lógicamente el clúster de AKS para el uso multiinquilino.

Azure AD proporciona una solución de administración de identidades para el ámbito empresarial que se puede integrar con los clústeres de AKS. Dado que Kubernetes no proporciona una solución de administración de identidades, es posible que se sienta obligado a restringir de forma individual el acceso al servidor de API. Con los clústeres integrados con Azure AD en AKS, usted utiliza sus cuentas de usuario y de grupo existentes para autenticar a los usuarios en el servidor de API.

![Integración de Azure Active Directory para clústeres de AKS](media/operator-best-practices-cluster-security/aad-integration.png)

Mediante la integración del control de acceso basado en rol de Kubernetes y Azure AD, puede proteger el servidor de API y proporcionar los permisos mínimos necesarios para un conjunto de recursos con ámbito como, por ejemplo, un espacio de nombres único. Puede conceder a diferentes usuarios o grupos de Azure AD diferentes roles de Kubernetes. Con permisos específicos puede restringir el acceso al servidor de API y proporcionar una pista de auditoría clara de las acciones realizadas.

El procedimiento recomendado consiste en utilizar *grupos* para proporcionar acceso a archivos y carpetas en lugar de identidades individuales. Por ejemplo, utilizar una pertenencia a un *grupo* de Azure AD para enlazar usuarios a roles de Kubernetes en lugar de *usuarios* individuales. A medida que cambie la pertenencia al grupo de un usuario, también lo harán sus permisos de acceso en el clúster de AKS. 

Mientras tanto, supongamos que enlaza al usuario individual directamente a un rol y su función de trabajo cambia. Aunque las pertenencias al grupo de Azure AD se actualizarán, sus permisos en el clúster de AKS no lo harán. En este escenario, el usuario al final cuenta con más permisos de los que necesita.

Para más información sobre la integración de Azure AD, RBAC de Kubernetes y Azure RBAC, consulte los [procedimientos recomendados para la autenticación y autorización en AKS][aks-best-practices-identity].

## <a name="secure-container-access-to-resources"></a>Proteger el acceso del contenedor a los recursos

> **Guía de procedimientos recomendados** 
> 
> Limite el acceso a las acciones que los contenedores pueden realizar. Proporcione el menor número de permisos y evite el uso del acceso a raíz y la elevación de privilegios.

Por el mismo motivo que debería conceder a los usuarios o a los grupos el menor número de privilegios necesarios, también debería limitar los contenedores a solo las acciones y procesos necesarios. Para minimizar el riesgo de ataques, evite configurar las aplicaciones y los contenedores que requieren elevación de privilegios o acceso a raíz. 

Por ejemplo, establezca `allowPrivilegeEscalation: false` en el manifiesto de pod. Estos *contextos de seguridad de pod* integrados en Kubernetes le permiten definir permisos adicionales, como el usuario o grupo como el que se ejecutará, o qué funcionalidades de Linux se expondrán. Para más recomendaciones, consulte [Protección del acceso del pod a los recursos][pod-security-contexts].

Para un control aún más detallado de las acciones de los contenedores, también puede usar las características de seguridad incorporadas de Linux como *AppArmor* y *seccomp*. 
1. Defina las características de seguridad de Linux en el nivel de nodo.
1. Implemente las características mediante un manifiesto de pod. 

Las características de seguridad integradas de Linux solo están disponibles en los pods y los nodos de Linux.

> [!NOTE]
> Actualmente, los entornos de Kubernetes no están completamente seguros ante el uso de varios inquilinos hostiles. Las características de seguridad adicionales, como *AppArmor*, *seccomp*, las *directivas de seguridad de pods* o el control de acceso basado en rol de Kubernetes para los nodos, bloquean eficazmente las vulnerabilidades de seguridad. 
>
>Para una verdadera seguridad al ejecutar cargas de trabajo multiinquilino hostiles, solo debe confiar en un hipervisor. El dominio de seguridad de Kubernetes se convierte en todo el clúster, no en un nodo específico. 
>
> En el caso de estos tipos de cargas de trabajo multiinquilino hostiles, debe usar clústeres que estén físicamente aislados.

### <a name="app-armor"></a>AppArmor

Para limitar las acciones de los contenedores, puede usar el módulo de seguridad del kernel de Linux denominado [AppArmor][k8s-apparmor]. AppArmor está disponible como parte del SO del nodo de AKS subyacente del sistema operativo y está habilitado de forma predeterminada. Puede crear perfiles de AppArmor para restringir acciones como leer, escribir o ejecutar, o funciones del sistema como montar sistemas de archivos. Los perfiles de AppArmor predeterminados restringen el acceso a diferentes ubicaciones de `/proc` y `/sys`, y proporcionan un medio para aislar lógicamente los contenedores desde el nodo subyacente. AppArmor funciona para cualquier aplicación que se ejecuta en Linux, no solo para los pods de Kubernetes.

![Perfiles de AppArmor en uso en un clúster AKS para limitar las acciones del contenedor](media/operator-best-practices-container-security/apparmor.png)

Para ver AppArmor en acción, en el ejemplo siguiente se crea un perfil que impide la escritura en archivos. 
1. Acceda mediante [SSH][aks-ssh] a un nodo de AKS.
1. Cree un archivo denominado *deny-write.profile*.
1. Pegue el siguiente contenido:

    ```
    #include <tunables/global>
    profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
      #include <abstractions/base>
  
      file,
      # Deny all file writes.
      deny /** w,
    }
    ```

Los perfiles de AppArmor se agregan con el comando `apparmor_parser`. 
1. Agregue el perfil a AppArmor.
1. Especifique el nombre del perfil creado en el paso anterior:

    ```console
    sudo apparmor_parser deny-write.profile
    ```

    Si el perfil se analiza y se aplica correctamente a AppArmor, no verá ninguna salida y se le devolverá al símbolo del sistema.

1. Desde la máquina local, cree un manifiesto de pod denominado *aks-apparmor.yaml*. Este manifiesto:
    * Define una anotación para `container.apparmor.security.beta.kubernetes`.
    * Hace referencia al perfil *deny-write* creado en los pasos anteriores.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: hello-apparmor
      annotations:
        container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
    spec:
      containers:
      - name: hello
        image: mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11
        command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
    ```

1. Con el pod implementado, compruebe que el pod *hello-apparmor* aparece *bloqueado*:

    ```
    $ kubectl get pods

    NAME             READY   STATUS    RESTARTS   AGE
    aks-ssh          1/1     Running   0          4m2s
    hello-apparmor   0/1     Blocked   0          50s
    ```

Para más información sobre AppArmor, consulte los [perfiles de AppArmor en Kubernetes][k8s-apparmor].

### <a name="secure-computing"></a>Informática segura

Si bien AppArmor funciona con cualquier aplicación de Linux, [seccomp (*sec* ure *comp* uting)][seccomp] funciona en el nivel de proceso. Seccomp también es un módulo de seguridad del kernel de Linux, y es compatible de forma nativa con el tiempo de ejecución de Docker que utilizan los nodos de AKS. Con seccomp, puede limitar las llamadas de proceso de contenedor. Adopte el procedimiento recomendado de conceder al contenedor el permiso mínimo solo para ejecutarlo mediante:
* La definición que filtra qué acciones se permiten o deniegan.
* La anotación dentro de un manifiesto YAML de pod para asociarlo al filtro de seccomp. 

Para ver seccomp en acción, cree un filtro que evite el cambio de permisos en un archivo. 
1. Acceda mediante [SSH][aks-ssh] a un nodo de AKS.
1. Cree un filtro de seccomp denominado */var/lib/kubelet/seccomp/prevent-chmod*.
1. Pegue el siguiente contenido:

    ```json
    {
      "defaultAction": "SCMP_ACT_ALLOW",
      "syscalls": [
        {
          "name": "chmod",
          "action": "SCMP_ACT_ERRNO"
        },
        {
          "name": "fchmodat",
          "action": "SCMP_ACT_ERRNO"
        },
        {
          "name": "chmodat",
          "action": "SCMP_ACT_ERRNO"
        }
      ]
    }
    ```

    En la versión 1.19 y versiones posteriores, debe configurar lo siguiente:

    ```json
    {
      "defaultAction": "SCMP_ACT_ALLOW",
      "syscalls": [
        {
          "names": ["chmod","fchmodat","chmodat"],
          "action": "SCMP_ACT_ERRNO"
        }
      ]
    }
    ```

1. En la máquina local, cree un manifiesto de pod denominado *aks-seccomp.yaml* y pegue el contenido siguiente. Este manifiesto:
    * Define una anotación para `seccomp.security.alpha.kubernetes.io`.
    * Hace referencia al filtro *prevent-chmod* que se creó en el paso anterior.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: chmod-prevented
      annotations:
        seccomp.security.alpha.kubernetes.io/pod: localhost/prevent-chmod
    spec:
      containers:
      - name: chmod
        image: mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11
        command:
          - "chmod"
        args:
         - "777"
         - /etc/hostname
      restartPolicy: Never
    ```

    En la versión 1.19 y versiones posteriores, debe configurar lo siguiente:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: chmod-prevented
    spec:
      securityContext:
        seccompProfile:
          type: Localhost
          localhostProfile: prevent-chmod
      containers:
      - name: chmod
        image: mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11
        command:
          - "chmod"
        args:
         - "777"
         - /etc/hostname
      restartPolicy: Never
    ```

1. Implemente el pod de ejemplo con el comando [kubectl apply][kubectl-apply]:

    ```console
    kubectl apply -f ./aks-seccomp.yaml
    ```

1. Consulte el estado de los pod mediante el comando [kubectl get pods][kubectl-get]. 
    * El pod indica un error. 
    * El filtro seccomp impide que se ejecute el comando `chmod`, tal como se muestra en la salida del ejemplo siguiente:    

    ```
    $ kubectl get pods

    NAME                      READY     STATUS    RESTARTS   AGE
    chmod-prevented           0/1       Error     0          7s
    ```

Para más información sobre los filtros disponibles, consulte [Perfiles de seguridad de Seccomp para Docker][seccomp].

## <a name="regularly-update-to-the-latest-version-of-kubernetes"></a>Actualización regular a la versión más reciente de Kubernetes

> **Guía de procedimientos recomendados** 
> 
> Para mantenerse al día con las nuevas características y correcciones de errores, actualice regularmente la versión de Kubernetes en el clúster de AKS.

Kubernetes presenta nuevas características a un ritmo más rápido que otras plataformas de infraestructura más tradicionales. Las actualizaciones de Kubernetes incluyen:
* Nuevas características
* Correcciones de errores o de seguridad 

Las nuevas características normalmente pasan al estado *alfa* y *beta* antes de convertirse en *estables*. Una vez que son estables, están disponibles con carácter general y se recomiendan para su uso en producción. Este nuevo ciclo de versión de actualización de características debería permitirle actualizar Kubernetes sin encontrar normalmente cambios importantes ni tener que ajustar las implementaciones ni las plantillas.

AKS es compatible con tres versiones secundarias de Kubernetes. Cuando se introduce una nueva versión secundaria de revisión, se retiran la versión secundaria y la versión de revisión compatibles más antiguas. Las actualizaciones secundarias de Kubernetes se realizan periódicamente. Para poder disponer de soporte técnico, asegúrese de que tiene un proceso de gobernanza para comprobar si hay actualizaciones necesarias. Para más información, consulte [Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)][aks-supported-versions].

Para comprobar las versiones que están disponibles para el clúster, use el comando [az aks get-upgrades][az-aks-get-upgrades] tal como se muestra en el ejemplo siguiente:

```azurecli-interactive
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
```

A continuación, puede actualizar el clúster de AKS con el comando [az aks upgrade][az-aks-upgrade]. De forma segura, el proceso de actualización:
* Acordona y purga un nodo cada vez.
* Programa pods en los nodos restantes.
* Implementa un nuevo nodo que ejecuta las versiones más recientes del sistema operativo y Kubernetes.

>[!IMPORTANT]
> Pruebe las nuevas versiones secundarias en un entorno de desarrollo y pruebas, y compruebe que la carga de trabajo funciona correctamente con la nueva versión de Kubernetes. 
>
> Kubernetes puede poner en desuso las API (como en la versión 1.16) en las que se basan las cargas de trabajo. Al incorporar nuevas versiones en producción, considere la posibilidad de usar [varios grupos de nodos en versiones independientes](use-multiple-node-pools.md) y actualizar grupos individuales de uno en uno para implementar progresivamente la actualización en un clúster. Si se ejecutan varios clústeres, actualice un clúster cada vez para supervisar progresivamente el impacto o los cambios.
>
>```azurecli-interactive
>az aks upgrade --resource-group myResourceGroup --name myAKSCluster --kubernetes-version KUBERNETES_VERSION
>```

Para obtener más información sobre las actualizaciones de AKS, consulte [Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)][aks-supported-versions] y [Actualización de un clúster de AKS][aks-upgrade].

## <a name="process-linux-node-updates-and-reboots-using-kured"></a>Proceso de las actualizaciones y reinicios de nodos Linux con kured

> **Guía de procedimientos recomendados** 
> 
> Aunque AKS descarga e instala automáticamente correcciones de seguridad en cada uno de los nodos Linux, no se reinicia automáticamente. 
> 1. Use `kured` para ver si hay reinicios pendientes.
> 1. Acordone y purgue el nodo de forma segura para permitir que el nodo se reinicie.
> 1. Aplique las actualizaciones.
> 1. Mantenga la mayor seguridad posible con respecto al sistema operativo. 

Para los nodos de Windows Server, realice periódicamente una operación de actualización de AKS para acordonar y drenar los pods de forma segura, e implemente los nodos actualizados.

Cada noche, los nodos Linux de AKS obtienen las actualizaciones de seguridad a través de su canal de actualización de distribuciones. Este comportamiento se configura automáticamente cuando se implementan los nodos en un clúster de AKS. Para minimizar las interrupciones y el posible impacto sobre las cargas de trabajo en ejecución, los nodos no se reinician automáticamente si lo requiere una revisión de seguridad o una actualización de kernel.

El proyecto [kured (KUbernetes REboot Daemon)][kured] de código abierto de Weaveworks vigila los reinicios pendientes del nodo. Cuando un nodo Linux aplica actualizaciones que requieren un reinicio, el nodo se acordona y se drena de forma segura para mover y programar los pods en otros nodos del clúster. Una vez que se reinicia el nodo, se vuelve a agregar al clúster y Kubernetes vuelve a reanudar la programación de pods. Para minimizar las interrupciones, solo se permite que `kured` reinicie un único nodo a la vez.

![Proceso de reinicio del nodo de AKS con kured](media/operator-best-practices-cluster-security/node-reboot-process.png)

Si desea un control aún más estrecho sobre los reinicios, `kured` puede integrarse con Prometheus para evitar reinicios si hay otros eventos de mantenimiento o problemas de clúster en curso. Esta integración reduce las complicaciones al reiniciar los nodos mientras está solucionando activamente otros problemas.

Para más información sobre cómo controlar los inicios del nodo, consulte [Aplicación de actualizaciones de kernel y de seguridad en los nodos en AKS][aks-kured].

## <a name="next-steps"></a>Pasos siguientes

En este artículo se indica cómo proteger el clúster de AKS. Para implementar algunas de estas áreas, consulte los artículos siguientes:

* [Integración de Azure Active Directory con AKS][aks-aad]
* [Actualización de un clúster AKS a la última versión de Kubernetes][aks-upgrade]
* [Procesamiento de actualizaciones de seguridad y reinicios del nodo con kured][aks-kured]

<!-- EXTERNAL LINKS -->
[kured]: https://github.com/weaveworks/kured
[k8s-apparmor]: https://kubernetes.io/docs/tutorials/clusters/apparmor/
[seccomp]: https://kubernetes.io/docs/concepts/policy/pod-security-policy/#seccomp
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-exec]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- INTERNAL LINKS -->
[az-aks-get-upgrades]: /cli/azure/aks#az_aks_get_upgrades
[az-aks-upgrade]: /cli/azure/aks#az_aks_upgrade
[aks-supported-versions]: supported-kubernetes-versions.md
[aks-upgrade]: upgrade-cluster.md
[aks-best-practices-identity]: concepts-identity.md
[aks-kured]: node-updates-kured.md
[aks-aad]: ./azure-ad-integration-cli.md
[best-practices-container-image-management]: operator-best-practices-container-image-management.md
[best-practices-pod-security]: developer-best-practices-pod-security.md
[pod-security-contexts]: developer-best-practices-pod-security.md#secure-pod-access-to-resources
[aks-ssh]: ssh.md
[security-center-aks]: ../security-center/defender-for-kubernetes-introduction.md
