---
title: Protección del clúster con directivas de seguridad de pod en Azure Kubernetes Service (AKS)
description: Aprenda a controlar las admisiones de pods mediante PodSecurityPolicy en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/25/2021
ms.openlocfilehash: d70b8e8efbf96e50575845ac88993012fed936d5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767428"
---
# <a name="preview---secure-your-cluster-using-pod-security-policies-in-azure-kubernetes-service-aks"></a>Versión preliminar: Protección del clúster con directivas de seguridad de pod en Azure Kubernetes Service (AKS)

> [!WARNING]
> **La característica descrita en este documento, directiva de seguridad de pods (versión preliminar), comenzará a [dejar de usar](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/) la versión 1.21 de Kubernetes, con su eliminación en la versión 1.25.** A medida que Kubernetes Upstream alcanza este hito, la comunidad de Kubernetes trabajará para documentar alternativas viables. El anuncio de desuso anterior se hizo en el momento en que no había una opción viable para los clientes. Ahora que la comunidad Kubernetes está trabajando en una alternativa, ya no es necesario deshacerse por adelantado de Kubernetes. 
>
> Una vez que la directiva de seguridad de pod (versión preliminar) haya quedado en desuso, deberá deshabilitar la característica en todos los clústeres existentes que la incluyan para realizar futuras actualizaciones de clústeres y seguir recibiendo el soporte técnico de Azure.

Para mejorar la seguridad del clúster de AKS, puede limitar los pods que se pueden programar. Los pods que soliciten recursos que no permita no podrán ejecutarse en el clúster de AKS. Defina este acceso mediante directivas de seguridad de pod. En este artículo se muestra cómo usar las directivas de seguridad de pod para limitar la implementación de pods en AKS.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

Es preciso que esté instalada y configurada la versión 2.0.61 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

### <a name="install-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

Para usar directivas de seguridad de pod, necesitará la versión 0.4.1 de la extensión de la CLI *aks-preview* o una posterior. Instale la extensión de la CLI de Azure *aks-preview* con el comando [az extension add][az-extension-add] y, a continuación, busque las actualizaciones disponibles con el comando [az extension update][az-extension-update]:

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="register-pod-security-policy-feature-provider"></a>Registro del proveedor de características de la directiva de seguridad de pod

**Este documento y esta característica quedarán en desuso el 15 de octubre de 2020.**

Para crear o actualizar un clúster de AKS para que use directivas de seguridad de pod, habilite primero una marca de características en su suscripción. Para registrar la marca de características *PodSecurityPolicyPreview*, use el comando [az feature register][az-feature-register] tal como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --name PodSecurityPolicyPreview --namespace Microsoft.ContainerService
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/PodSecurityPolicyPreview')].{Name:name,State:properties.state}"
```

Cuando todo esté listo, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="overview-of-pod-security-policies"></a>Introducción a las directivas de seguridad de pod

En los clústeres de Kubernetes se usa un controlador de admisión para interceptar las solicitudes al servidor de API cuando se va a crear un recurso. El controlador de admisión puede *validar* la solicitud del recurso con un conjunto de reglas, o *mutar* el recurso para cambiar los parámetros de implementación.

*PodSecurityPolicy* es un controlador de admisión que valida si una especificación de pod cumple los requisitos definidos. Estos pueden incluir el límite del uso de contenedores con privilegios, del acceso a ciertos tipos de almacenamiento, o del usuario o grupo que puede ejecutar el contenedor. Al intentar implementar un recurso donde las especificaciones de pod no cumplan los requisitos descritos en la directiva de seguridad de pod, la solicitud se deniega. Esta capacidad para controlar qué pods se pueden programar en el clúster de AKS evita algunos puntos vulnerables de seguridad o elevaciones de privilegios.

Al habilitar la directiva de seguridad de pod en el clúster de AKS, se aplican algunas directivas predeterminadas. Estas directivas predeterminadas proporcionan una experiencia de fábrica para definir qué pods que se puede programar. Sin embargo, los usuarios del clúster pueden encontrarse con problemas al implementar los pods hasta que defina sus propias directivas. El método recomendado es el siguiente:

* Creación de un clúster de AKS
* Definición de directivas de seguridad de pod propias
* Habilitación de la característica de directiva de seguridad de pod

Para mostrar cómo las directivas predeterminadas limitan la implementación de pods, en este artículo primero se habilita la característica de las directivas de seguridad de pod y después se crea una directiva personalizada.

### <a name="behavior-changes-between-pod-security-policy-and-azure-policy"></a>El comportamiento cambia entre la directiva de seguridad de pod y Azure Policy

A continuación se muestra un resumen de los cambios de comportamiento entre la directiva de seguridad de pod y Azure Policy.

|Escenario| Directiva de seguridad de pod | Azure Policy |
|---|---|---|
|Instalación|Habilitar la característica de directiva de seguridad de pod |Habilitar el complemento de Azure Policy
|Implementar directivas| Implementar el recurso de la directiva de seguridad de pod| Asigne las directivas de Azure al ámbito de la suscripción o del grupo de recursos. El complemento de Azure Policy es necesario para las aplicaciones de recursos de Kubernetes.
| Directivas predeterminadas | Cuando la directiva de seguridad de pod está habilitada en AKS, se aplican las directivas predeterminadas Con privilegios y Sin restricciones. | No se aplica ninguna directiva predeterminada al habilitar el complemento de Azure Policy. Debe habilitar explícitamente las directivas en Azure Policy.
| Quién puede crear y asignar directivas | El administrador del clúster crea un recurso de directiva de seguridad de pod | Los usuarios deben tener un rol mínimo con permisos de "propietario" o "colaborador de directiva de recursos" en el grupo de recursos del clúster de AKS. - Mediante la API, los usuarios pueden asignar directivas en el ámbito del recurso del clúster de AKS. El usuario debe tener permisos mínimos de "propietario" o "colaborador de directiva de recursos" en el recurso del clúster de AKS. - En Azure Portal, las directivas se pueden asignar en el nivel de grupo de administración, suscripción o grupo de recursos.
| Autorizar directivas| Los usuarios y las cuentas de servicio requieren permisos explícitos para usar las directivas de seguridad de pod. | No se requiere ninguna asignación adicional para autorizar directivas. Una vez que se han asignado las directivas en Azure, todos los usuarios del clúster pueden usar estas directivas.
| Aplicabilidad de las directivas | El usuario administrador está exento del cumplimiento de las directivas de seguridad de pod. | Todos los usuarios (administradores y no administradores) ven las mismas directivas. No hay ninguna grafía especial basada en los usuarios. La aplicación de las directivas se puede excluir en el nivel de espacio de nombres.
| Ámbito de la directiva | Las directivas de seguridad de pod no tienen espacios de nombres | Las plantillas de restricción usadas por Azure Policy no tienen espacios de nombres.
| Acción de denegación/auditoría/mutación | Las directivas de seguridad de pod solo admiten acciones de denegación. La mutación se puede realizar con valores predeterminados en las solicitudes de creación. La validación se puede realizar durante las solicitudes de actualización.| Azure Policy admite las acciones de denegación y de auditoría. La mutación todavía no se admite, pero está planeada.
| Cumplimiento de la directiva de seguridad de pod | No hay visibilidad del cumplimiento de los pods que existían antes de habilitar la directiva de seguridad de pod. Los pods no compatibles creados después de habilitar las directivas de seguridad de pod se deniegan. | Los pods no compatibles que existían antes de aplicar las directivas de Azure se mostrarían en infracciones de directivas. Los pods no compatibles creados después de habilitar las directivas de Azure se deniegan si las directivas se establecen con efecto de denegación.
| Cómo ver las directivas del clúster | `kubectl get psp` | `kubectl get constrainttemplate` : devuelve todas las directivas.
| Directiva de seguridad de pod estándar: con privilegios | De forma predeterminada, se crea un recurso de directiva de seguridad de pod con privilegios al habilitar la característica. | El modo con privilegios no implica ninguna restricción, por tanto, es equivalente a no tener ninguna asignación de Azure Policy.
| [Directiva de seguridad de pod estándar: línea de base/predeterminada](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline-default) | El usuario instala un recurso de línea de base de la directiva de seguridad de pod. | Azure Policy proporciona una [iniciativa de línea de base integrada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) que se asigna a la directiva de seguridad de pod de línea de base.
| [Directiva de seguridad de pod estándar: restringida](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) | El usuario instala un recurso restringido de la directiva de seguridad de pod. | Azure Policy proporciona una [iniciativa restringida integrada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) que se asigna a la directiva de seguridad de pod de restringida.

## <a name="enable-pod-security-policy-on-an-aks-cluster"></a>Habilitación de una directiva de seguridad de pod en un clúster de AKS

Puede habilitar o deshabilitar la directiva de seguridad de pod mediante el comando [az aks update][az-aks-update]. En el siguiente ejemplo se habilita la directiva de seguridad de pod en el clúster denominado *myAKSCluster* del grupo de recursos *myResourceGroup*.

> [!NOTE]
> Para el uso en el mundo real, no habilite la directiva de seguridad de pod hasta que haya definido sus propias directivas personalizadas. En este artículo se habilita la directiva de seguridad de pod como primer paso para mostrar cómo las predeterminadas limitan la implementación de pods.

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --enable-pod-security-policy
```

## <a name="default-aks-policies"></a>Directivas predeterminadas de AKS

Cuando se habilita la directiva de seguridad pod, AKS crea una directiva predeterminada denominada *privileged*. No modifique ni elimine la directiva predeterminada. En su lugar, cree sus propias directivas que definan la configuración que desee controlar. Primero echemos un vistazo a qué son estas directivas predeterminadas y a cómo afectan a la implementación de pods.

Para ver las directivas disponibles, use el comando [kubectl get psp][kubectl-get], tal como se muestra en el siguiente ejemplo.

```console
$ kubectl get psp

NAME         PRIV    CAPS   SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
privileged   true    *      RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
```

La directiva de seguridad de pod *provileged* se aplica a cualquier usuario que se autentique en el clúster de AKS. Esta asignación se controla mediante ClusterRoles y ClusterRoleBindings. Use el comando [kubectl get rolebindings][kubectl-get] y busque el enlace *default:privileged:* en el espacio de nombres *kube-system*:

```console
kubectl get rolebindings default:privileged -n kube-system -o yaml
```

Como se muestra en la siguiente salida reducida, el valor de ClusterRole *psp:privileged* se asigna a cualquier usuario *system:authenticated*. Esta capacidad proporciona un nivel básico de privilegio sin tener directivas propias definidas.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  [...]
  name: default:privileged
  [...]
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp:privileged
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

Es importante entender cómo interactúan estas directivas predeterminadas con las solicitudes de usuario para programar los pods antes de empezar a crear directivas de seguridad de pod propias. En las siguientes secciones programaremos algunos pods para ver estas directivas predeterminadas en acción.

## <a name="create-a-test-user-in-an-aks-cluster"></a>Creación de un usuario de prueba en un clúster de AKS

De forma predeterminada, al usar el comando [az aks get-credentials][az-aks-get-credentials], las credenciales del *administrador* del clúster de AKS se agregan a la configuración de `kubectl`. El usuario administrador está exento del cumplimiento de las directivas de seguridad de pod. Si usa la integración de Azure Active Directory para los clústeres de AKS, podría iniciar sesión con las credenciales de un usuario sin derechos de administrador para ver el cumplimiento de directivas en acción. En este artículo vamos a crear y a usar una cuenta de usuario de prueba en el clúster de AKS.

Cree un espacio de nombres de ejemplo denominado *psp-aks* para probar los recursos con el comando [kubectl create namespace][kubectl-create]. A continuación, cree una cuenta de servicio denominada *nonadmin-user* con el comando [kubectl create serviceaccount][kubectl-create]:

```console
kubectl create namespace psp-aks
kubectl create serviceaccount --namespace psp-aks nonadmin-user
```

Ahora cree un RoleBinding para *nonadmin-user* para que realice acciones básicas en el espacio de nombres mediante el comando [kubectl create rolebinding][kubectl-create]:

```console
kubectl create rolebinding \
    --namespace psp-aks \
    psp-aks-editor \
    --clusterrole=edit \
    --serviceaccount=psp-aks:nonadmin-user
```

### <a name="create-alias-commands-for-admin-and-non-admin-user"></a>Creación de comandos de alias para los usuarios administrador y no administrador

Para resaltar la diferencia entre el usuario de administrador normal al usar `kubectl` y el usuario sin derechos de administrador que creó en los pasos anteriores, cree dos alias de la línea de comandos:

* El alias **kubectl-admin** es para el usuario administrador normal, para el espacio de nombres *psp-aks*.
* El alias **kubectl nonadminuser** es para el *usuario no administrador* creado en el paso anterior, para el espacio de nombres *psp-aks*.

Cree estos dos alias tal y como se muestra en los siguientes comandos:

```console
alias kubectl-admin='kubectl --namespace psp-aks'
alias kubectl-nonadminuser='kubectl --as=system:serviceaccount:psp-aks:nonadmin-user --namespace psp-aks'
```

## <a name="test-the-creation-of-a-privileged-pod"></a>Prueba de la creación de un pod con privilegios

Vamos a probar primero lo que sucede cuando se programa un pod con el contexto de seguridad `privileged: true`. Este contexto de seguridad eleva los privilegios del pod. En la sección anterior donde se mostraban las directivas de seguridad de pod predeterminadas de AKS, la directiva *privilege* debería denegar esta solicitud.

Cree un archivo denominado `nginx-privileged.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
      securityContext:
        privileged: true
```

Cree el pod mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl-nonadminuser apply -f nginx-privileged.yaml
```

El pod no se puede programar, como se muestra en la salida del ejemplo siguiente:

```console
$ kubectl-nonadminuser apply -f nginx-privileged.yaml

Error from server (Forbidden): error when creating "nginx-privileged.yaml": pods "nginx-privileged" is forbidden: unable to validate against any pod security policy: []
```

El pod no llega a la fase de programación, por lo que no hay recursos que eliminar para continuar.

## <a name="test-creation-of-an-unprivileged-pod"></a>Prueba de la creación de un pod sin privilegios

En el ejemplo anterior, la especificación del pod solicitaba el escalado de privilegios. Esta solicitud la deniega la directiva de seguridad de pod *privilege* predeterminada, por lo que no se puede programar el pod. Vamos a probar ahora a ejecutar ese mismo pod NGINX sin la solicitud de elevación de privilegios.

Cree un archivo denominado `nginx-unprivileged.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
```

Cree el pod mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl-nonadminuser apply -f nginx-unprivileged.yaml
```

El pod no se puede programar, como se muestra en la salida del ejemplo siguiente:

```console
$ kubectl-nonadminuser apply -f nginx-unprivileged.yaml

Error from server (Forbidden): error when creating "nginx-unprivileged.yaml": pods "nginx-unprivileged" is forbidden: unable to validate against any pod security policy: []
```

El pod no llega a la fase de programación, por lo que no hay recursos que eliminar para continuar.

## <a name="test-creation-of-a-pod-with-a-specific-user-context"></a>Creación de prueba de un pod con un contexto de usuario específico

En el ejemplo anterior la imagen de contenedor intentó automáticamente utilizar la raíz para enlazar NGINX al puerto 80. Esta solicitud la denegó la directiva de seguridad de pod *privilege* predeterminada, por lo que no se puede iniciar el pod. Vamos a probar ahora a ejecutar ese mismo pod NGINX con un contexto de usuario específico, como `runAsUser: 2000`.

Cree un archivo denominado `nginx-unprivileged-nonroot.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged-nonroot
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
      securityContext:
        runAsUser: 2000
```

Cree el pod mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl-nonadminuser apply -f nginx-unprivileged-nonroot.yaml
```

El pod no se puede programar, como se muestra en la salida del ejemplo siguiente:

```console
$ kubectl-nonadminuser apply -f nginx-unprivileged-nonroot.yaml

Error from server (Forbidden): error when creating "nginx-unprivileged-nonroot.yaml": pods "nginx-unprivileged-nonroot" is forbidden: unable to validate against any pod security policy: []
```

El pod no llega a la fase de programación, por lo que no hay recursos que eliminar para continuar.

## <a name="create-a-custom-pod-security-policy"></a>Creación de una directiva de seguridad de pod personalizada

Ahora que hemos visto el comportamiento de las directivas de seguridad de pod predeterminadas, vamos a proporcionar una forma para que el *usuario no administrador* pueda programar pods.

Vamos a crear una directiva para rechazar los pods que soliciten acceso con privilegios. Otras opciones, como *runAsUser* o los *volúmenes* permitidos, no se restringen explícitamente. Este tipo de directiva deniega una solicitud de acceso con privilegios, pero permite al clúster ejecutar los pods solicitados.

Cree un archivo denominado `psp-deny-privileged.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp-deny-privileged
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
```

Cree la directiva mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f psp-deny-privileged.yaml
```

Para ver las directivas disponibles, use el comando [kubectl get psp][kubectl-get], tal como se muestra en el siguiente ejemplo. Compare la directiva *psp-deny-privileged* con el valor de la directiva *privilege* predeterminada que aplicó en los ejemplos anteriores para crear un pod. Su directiva solo deniega el uso del escalado *PRIV*. No hay ninguna restricción para el usuario o grupo relativa a la directiva *psp-deny-privileged*.

```console
$ kubectl get psp

NAME                  PRIV    CAPS   SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
privileged            true    *      RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *
psp-deny-privileged   false          RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *          
```

## <a name="allow-user-account-to-use-the-custom-pod-security-policy"></a>Permiso par que la cuenta de usuario use la directiva de seguridad de pod personalizada

En el paso anterior ha creado una directiva de seguridad de pod para rechazar los pods que soliciten acceso con privilegios. Para permitir el uso de la directiva, cree un *rol* o *ClusterRole*. Asocie uno de estos roles mediante *RoleBinding* o *ClusterRoleBinding*.

Para este ejemplo, cree un ClusterRole que le permita *usar* la directiva *psp-deny-privileged* que creó en el paso anterior. Cree un archivo denominado `psp-deny-privileged-clusterrole.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-deny-privileged-clusterrole
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - psp-deny-privileged
  verbs:
  - use
```

Cree el ClusterRole mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f psp-deny-privileged-clusterrole.yaml
```

Ahora cree un ClusterRoleBinding para usar el ClusterRole que creó en el paso anterior. Cree un archivo denominado `psp-deny-privileged-clusterrolebinding.yaml` y pegue el siguiente manifiesto de YAML:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: psp-deny-privileged-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-deny-privileged-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts
```

Cree un ClusterRoleBinding mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f psp-deny-privileged-clusterrolebinding.yaml
```

> [!NOTE]
> En el primer paso de este artículo se ha habilitado la característica de directiva de seguridad de pod en el clúster de AKS. El método recomendado era habilitar solo la característica de directiva de seguridad de pod una vez definidas las directivas propias. Esta es la fase en la que habilitaría la característica de directiva de seguridad de pod. Se han definido una o varias directivas personalizadas y las cuentas de usuario se han asociado a esas directivas. Ahora puede habilitar la característica de directiva de seguridad de pod sin problemas y reducir los problemas causados por las directivas predeterminadas.

## <a name="test-the-creation-of-an-unprivileged-pod-again"></a>Segunda prueba de creación de un pod sin privilegios

Con la directiva de seguridad de pod personalizada aplicado y un enlace para la cuenta de usuario para que use la directiva, vamos a intentar volver a crear un pod sin privilegios. Use el mismo manifiesto `nginx-privileged.yaml` para crear el pod mediante el comando [kubectl apply][kubectl-apply]:

```console
kubectl-nonadminuser apply -f nginx-unprivileged.yaml
```

Se ha programado correctamente el pod. Al activar el estado del pod mediante el comando [kubectl get pods][kubectl-get], el pod tiene el estado *En ejecución*:

```
$ kubectl-nonadminuser get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          7m14s
```

En este ejemplo se muestra cómo crear directivas de seguridad de pod personalizadas para definir el acceso al clúster de AKS para distintos usuarios o grupos. Las directivas de AKS predeterminadas proporcionan controles estrictos sobre lo que los pods pueden ejecutar, por lo tanto, cree sus propias directivas personalizadas para definir las restricciones que necesite correctamente.

Elimine el pod NGIX sin privilegios mediante el comando [kubectl delete][kubectl-delete] y especifique el nombre del manifiesto de YAML:

```console
kubectl-nonadminuser delete -f nginx-unprivileged.yaml
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Par deshabilitar la directiva de seguridad de pod, use el comando [az aks update][az-aks-update] de nuevo. En el siguiente ejemplo se deshabilita la directiva de seguridad de pod en el clúster denominado *myAKSCluster* del grupo de recursos *myResourceGroup*:

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --disable-pod-security-policy
```

A continuación, elimine ClusterRole y ClusterRoleBinding:

```console
kubectl delete -f psp-deny-privileged-clusterrolebinding.yaml
kubectl delete -f psp-deny-privileged-clusterrole.yaml
```

Elimine la directiva de seguridad mediante el comando [kubectl delete][kubectl-delete] y especifique el nombre del manifiesto de YAML:

```console
kubectl delete -f psp-deny-privileged.yaml
```

Por último, elimine el espacio de nombres *psp-aks*:

```console
kubectl delete namespace psp-aks
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha explicado cómo crear una directiva de seguridad de pod para evitar el acceso con privilegios. Hay una gran cantidad de características que puede aplicar una directiva, como el tipo de volumen o el usuario RunAs. Para más información sobre las opciones disponibles, consulte los [documentos de referencia de la directiva de seguridad de pod de Kubernetes][kubernetes-policy-reference].

Para más información sobre la limitación del tráfico del pod, consulte [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)][network-policies].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[kubernetes-policy-reference]: https://kubernetes.io/docs/concepts/policy/pod-security-policy/#policy-reference
<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policies]: use-network-policies.md
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-extension-add]: /cli/azure/extension#az_extension_add
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[policy-samples]: ./policy-reference.md#microsoftcontainerservice
[azure-policy-add-on]: ../governance/policy/concepts/policy-for-kubernetes.md
