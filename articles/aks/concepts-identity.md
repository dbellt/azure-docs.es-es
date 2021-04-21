---
title: 'Conceptos: Acceso e identidad en Azure Kubernetes Service (AKS)'
description: Este artículo contiene más información sobre el acceso y la identidad en Azure Kubernetes Service (AKS), incluida la integración de Azure Active Directory, el control de acceso basado en rol de Kubernetes (RBAC de Kubernetes) y los roles y enlaces.
services: container-service
ms.topic: conceptual
ms.date: 03/24/2021
author: palma21
ms.author: jpalma
ms.openlocfilehash: b10d31cf069bc4f28a1597ec12160fa6ed98b8ce
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789562"
---
# <a name="access-and-identity-options-for-azure-kubernetes-service-aks"></a>Opciones de acceso e identidad en Azure Kubernetes Service (AKS)

Puede autenticar, autorizar, proteger y controlar el acceso a los clústeres de Kubernetes de varias maneras. 
* Mediante el control de acceso basado en rol de Kubernetes (RBAC de Kubernetes), puede conceder acceso a usuarios, grupos y cuentas de servicio únicamente a los recursos que necesitan. 
* Con Azure Kubernetes Service (AKS), puede mejorar aún más la seguridad y la estructura de permisos mediante el uso de Azure Active Directory y Azure RBAC. 

AKS y RBAC de Kubernetes le ayudan a proteger el acceso al clúster y a proporcionar solo los permisos mínimos necesarios a los desarrolladores y operadores.

En este artículo se presentan los conceptos básicos para ayudarle a autenticarse y a asignar permisos en AKS.

## <a name="aks-service-permissions"></a>Permisos del servicio AKS

Al crear un clúster, AKS genera o modifica los recursos necesarios (por ejemplo, máquinas virtuales y NIC) para crear y ejecutar el clúster en nombre del usuario. Esta identidad es distinta del permiso de identidad del clúster, que se crea durante la construcción del clúster.

### <a name="identity-creating-and-operating-the-cluster-permissions"></a>Identidad para crear y administrar los permisos de clúster

Los siguientes permisos son necesarios para la identidad que crea y administra el clúster.

| Permiso | Motivo |
|---|---|
| `Microsoft.Compute/diskEncryptionSets/read` | Se requiere para leer el identificador de conjunto de cifrado de disco. |
| `Microsoft.Compute/proximityPlacementGroups/write` | Necesario para actualizar los grupos con ubicación por proximidad. |
| `Microsoft.Network/applicationGateways/read` <br/> `Microsoft.Network/applicationGateways/write` <br/> `Microsoft.Network/virtualNetworks/subnets/join/action` | Se requiere para configurar las puertas de enlace de aplicaciones y unirse a la subred. |
| `Microsoft.Network/virtualNetworks/subnets/join/action` | Se requiere para configurar el grupo de seguridad de red para la subred cuando se usa una red virtual personalizada.|
| `Microsoft.Network/publicIPAddresses/join/action` <br/> `Microsoft.Network/publicIPPrefixes/join/action` | Se requiere para configurar las direcciones IP públicas salientes en el servicio Standard Load Balancer. |
| `Microsoft.OperationalInsights/workspaces/sharedkeys/read` <br/> `Microsoft.OperationalInsights/workspaces/read` <br/> `Microsoft.OperationsManagement/solutions/write` <br/> `Microsoft.OperationsManagement/solutions/read` <br/> `Microsoft.ManagedIdentity/userAssignedIdentities/assign/action` | Se requiere para crear y actualizar las áreas de trabajo de Log Analytics y la supervisión de Azure para contenedores. |

### <a name="aks-cluster-identity-permissions"></a>Permisos de identidad de clúster de AKS

Los siguientes permisos se usan en la identidad de clúster de AKS, que se crea y se asocia al clúster de AKS. Cada permiso se usa por los motivos que se indican a continuación:

| Permiso | Motivo |
|---|---|
| `Microsoft.ContainerService/managedClusters/*`  <br/> | Necesario para la creación de usuarios y el funcionamiento del clúster
| `Microsoft.Network/loadBalancers/delete` <br/> `Microsoft.Network/loadBalancers/read` <br/> `Microsoft.Network/loadBalancers/write` | Se requiere para configurar el equilibrador de carga para un servicio LoadBalancer. |
| `Microsoft.Network/publicIPAddresses/delete` <br/> `Microsoft.Network/publicIPAddresses/read` <br/> `Microsoft.Network/publicIPAddresses/write` | Se requiere para buscar y configurar direcciones IP públicas para un servicio LoadBalancer. |
| `Microsoft.Network/publicIPAddresses/join/action` | Se requiere para configurar direcciones IP públicas para un servicio LoadBalancer. |
| `Microsoft.Network/networkSecurityGroups/read` <br/> `Microsoft.Network/networkSecurityGroups/write` | Se requiere para crear o eliminar reglas de seguridad para un servicio LoadBalancer. |
| `Microsoft.Compute/disks/delete` <br/> `Microsoft.Compute/disks/read` <br/> `Microsoft.Compute/disks/write` <br/> `Microsoft.Compute/locations/DiskOperations/read` | Se requiere para configurar AzureDisks. |
| `Microsoft.Storage/storageAccounts/delete` <br/> `Microsoft.Storage/storageAccounts/listKeys/action` <br/> `Microsoft.Storage/storageAccounts/read` <br/> `Microsoft.Storage/storageAccounts/write` <br/> `Microsoft.Storage/operations/read` | Se requiere para configurar las cuentas de almacenamiento de AzureFile o AzureDisk. |
| `Microsoft.Network/routeTables/read` <br/> `Microsoft.Network/routeTables/routes/delete` <br/> `Microsoft.Network/routeTables/routes/read` <br/> `Microsoft.Network/routeTables/routes/write` <br/> `Microsoft.Network/routeTables/write` | Se requiere para configurar las tablas de ruta y las rutas para nodos. |
| `Microsoft.Compute/virtualMachines/read` | Se requiere para encontrar información de las máquinas virtuales en un clúster VMAS (por ejemplo, zonas, dominio de error, tamaño y discos de datos). |
| `Microsoft.Compute/virtualMachines/write` | Se requiere para asociar AzureDisks a una máquina virtual en un clúster VMAS. |
| `Microsoft.Compute/virtualMachineScaleSets/read` <br/> `Microsoft.Compute/virtualMachineScaleSets/virtualMachines/read` <br/> `Microsoft.Compute/virtualMachineScaleSets/virtualmachines/instanceView/read` | Se requiere para encontrar información de máquinas virtuales en un conjunto de escalado de máquinas virtuales (por ejemplo, zonas, dominio de error, tamaño y discos de datos). |
| `Microsoft.Network/networkInterfaces/write` | Se requiere para agregar una máquina virtual de un clúster VMAS a un grupo de direcciones de back-end del equilibrador de carga. |
| `Microsoft.Compute/virtualMachineScaleSets/write` | Se requiere para agregar un conjunto de escalado de máquinas virtuales a grupos de direcciones de back-end del equilibrador de carga y a nodos de escalabilidad horizontal en un conjunto de escalado de máquinas virtuales. |
| `Microsoft.Compute/virtualMachineScaleSets/virtualmachines/write` | Se requiere para asociar AzureDisks y agregar una máquina virtual de un conjunto de escalado de máquinas virtuales al equilibrador de carga. |
| `Microsoft.Network/networkInterfaces/read` | Se requiere para buscar grupos de direcciones IP internas y de direcciones de back-end del equilibrador de carga para las máquinas virtuales de un clúster VMAS. |
| `Microsoft.Compute/virtualMachineScaleSets/virtualMachines/networkInterfaces/read` | Se requiere para buscar grupos de direcciones IP internas y de back-end del equilibrador de carga para una máquina virtual de un conjunto de escalado de máquinas virtuales. |
| `Microsoft.Compute/virtualMachineScaleSets/virtualMachines/networkInterfaces/ipconfigurations/publicipaddresses/read` | Se requiere para buscar direcciones IP públicas para una máquina virtual de un conjunto de escalado de máquinas virtuales. |
| `Microsoft.Network/virtualNetworks/read` <br/> `Microsoft.Network/virtualNetworks/subnets/read` | Se requiere para comprobar si existe una subred para el equilibrador de carga interno en otro grupo de recursos. |
| `Microsoft.Compute/snapshots/delete` <br/> `Microsoft.Compute/snapshots/read` <br/> `Microsoft.Compute/snapshots/write` | Se requiere para configurar instantáneas de AzureDisk. |
| `Microsoft.Compute/locations/vmSizes/read` <br/> `Microsoft.Compute/locations/operations/read` | Se requiere para buscar tamaños de máquina virtual y encontrar límites de volumen de AzureDisk. |

### <a name="additional-cluster-identity-permissions"></a>Permisos adicionales de identidad de clúster

Al crear un clúster con atributos específicos, necesitará los siguientes permisos adicionales para la identidad del clúster. Dado que estos permisos no se asignan automáticamente, debe agregarlos a la identidad del clúster después de su creación.

| Permiso | Motivo |
|---|---|
| `Microsoft.Network/networkSecurityGroups/write` <br/> `Microsoft.Network/networkSecurityGroups/read` | Se requiere cuando se usa un grupo de seguridad de red en otro grupo de recursos. Se requiere para configurar reglas de seguridad para un servicio LoadBalancer. |
| `Microsoft.Network/virtualNetworks/subnets/read` <br/> `Microsoft.Network/virtualNetworks/subnets/join/action` | Se requiere cando se usa una subred en otro grupo de recursos (por ejemplo, una red virtual personalizada). |
| `Microsoft.Network/routeTables/routes/read` <br/> `Microsoft.Network/routeTables/routes/write` | Se requiere cuando se usa una subred asociada a una tabla de rutas en otro grupo de recursos (por ejemplo, una red virtual personalizada con una tabla de rutas personalizada). Se requiere para comprobar si ya existe una subred para la subred en el otro grupo de recursos. |
| `Microsoft.Network/virtualNetworks/subnets/read` | Se requiere cuando se usa un equilibrador de carga interno en otro grupo de recursos. Se requiere para comprobar si ya existe una subred para el equilibrador de carga interno en el grupo de recursos. |
| `Microsoft.Network/privatednszones/*` | Se requiere cuando se usa una subred en otro grupo de recursos (por ejemplo, una red privateDNSZone personalizada). |

## <a name="kubernetes-rbac"></a>RBAC de Kubernetes

RBAC de Kubernetes proporciona un filtrado detallado de las acciones del usuario. Con este mecanismo de control:
* Asigna a usuarios o grupos de usuarios permiso para crear o modificar recursos, o ver registros de cargas de trabajo de aplicaciones en ejecución. 
* Puede limitar los permisos a un único espacio de nombres o a todo el clúster de AKS. 
* Crea *roles* para definir permisos y, después, asigna esos roles a usuarios mediante *enlaces de rol*.

Para obtener más información, consulte [Uso de la autorización de RBAC de Kubernetes][kubernetes-rbac].

### <a name="roles-and-clusterroles"></a>Roles y ClusterRoles

#### <a name="roles"></a>Roles
Antes de asignar permisos a los usuarios con RBAC de Kubernetes, definirá los permisos de usuario como un *rol*. Conceda permisos dentro de un espacio de nombres mediante roles. 

> [!NOTE]
> Los roles de Kubernetes *conceden* permisos; no los *deniegan*.

Para conceder permisos en todo el clúster o a recursos de clúster fuera de un espacio de nombres determinado, puede usar en su lugar *ClusterRoles*.

#### <a name="clusterroles"></a>ClusterRoles

Un objeto ClusterRole concede y aplica permisos a los recursos de todo el clúster, no a un espacio de nombres específico.

### <a name="rolebindings-and-clusterrolebindings"></a>RoleBindings y ClusterRoleBindings

Tras definir los roles para conceder permisos a los recursos, asigne esos permisos de RBAC de Kubernetes con un *RoleBinding*. Si el clúster de AKS [se integra con Azure Active Directory (Azure AD)](#azure-ad-integration), los RoleBindings conceden permisos a los usuarios de Azure AD para que realicen acciones en el clúster. Consulte cómo se hace en [Administración del acceso a recursos de clúster mediante el control de acceso basado en roles de Kubernetes y las identidades de Azure Active Directory](azure-ad-rbac.md).

#### <a name="rolebindings"></a>RoleBindings

Asigne roles a los usuarios de un espacio de nombres determinado mediante RoleBindings. Con RoleBindings, puede separar lógicamente un único clúster de AKS, de modo que solo se permita a los usuarios acceder a los recursos de la aplicación en su espacio de nombres asignado. 

Para enlazar roles en todo el clúster o para recursos de clúster fuera de un espacio de nombres determinado, use en su lugar *ClusterRoleBindings*.

#### <a name="clusterrolebinding"></a>ClusterRoleBinding

Con un objeto ClusterRoleBinding, enlaza roles a usuarios y los aplica a los recursos de todo el clúster, no a un espacio de nombres determinado. Este enfoque le permite conceder acceso para los administradores o ingenieros de soporte técnico a todos los recursos del clúster de AKS.


> [!NOTE]
> Microsoft/AKS realiza todas las acciones de clúster con el consentimiento del usuario en el rol de Kubernetes integrado `aks-service` y el enlace de roles integrado `aks-service-rolebinding`. 
> 
> Este rol permite a AKS solucionar problemas de clústeres, pero no puede modificar permisos ni crear roles, enlaces de roles ni otras acciones de privilegios elevados. El acceso a roles solo se habilita en incidencias de soporte técnico activos con acceso Just-in-Time (JIT). Obtenga más información sobre [Directivas de soporte técnico de AKS](support-policies.md).


### <a name="kubernetes-service-accounts"></a>Cuentas de servicio de Kubernetes

Las *cuentas de servicio* son uno de los tipos de usuario principales en Kubernetes. La API de Kubernetes contiene y administra cuentas de servicio. Las credenciales de las cuentas de servicio se almacenan como secretos de Kubernetes que pueden usar los pods autorizados para comunicarse con el servidor de API. La mayoría de las solicitudes de API proporcionan un token de autenticación para una cuenta de servicio o una cuenta de usuario normal.

Las cuentas de usuario normales permiten un acceso más tradicional para administradores o desarrolladores humanos, no solo a servicios y procesos. Aunque Kubernetes no proporciona una solución de administración de identidades para almacenar contraseñas y cuentas de usuario normales, puede integrar soluciones de identidades externas en Kubernetes. Para los clústeres de AKS, esta solución de identidades integrada es Azure AD.

Para más información sobre las opciones de identidad en Kubernetes, consulte la [autenticación Kubernetes][kubernetes-authentication].

## <a name="azure-ad-integration"></a>Integración de Azure AD

Mejore la seguridad del clúster de AKS con la integración de Azure AD. Con la experiencia de varias décadas de administración de identidades empresariales, Azure AD es un servicio multiinquilino de administración de identidades y de directorios basado en la nube que combina los servicios de directorio principales, la administración del acceso de las aplicaciones y la protección de identidades. Con Azure AD, puede integrar identidades locales en clústeres de AKS para proporcionar un único origen para la seguridad y administración de cuentas.

![Integración de Azure Active Directory con clústeres de AKS](media/concepts-identity/aad-integration.png)

Con los clústeres de AKS integrados en Azure AD, puede conceder a los usuarios o grupos acceso a los recursos de Kubernetes de un espacio de nombres o del clúster. 

1. Para obtener un contexto de configuración de `kubectl`, el usuario ejecuta el comando [az aks get-credentials][az-aks-get-credentials]. 
1. Cuando un usuario interactúa con el clúster de AKS con `kubectl`, se le pide que inicie sesión con sus credenciales de Azure AD. 

Este enfoque proporciona un único origen para la administración de cuentas de usuario y de las credenciales de contraseña. El usuario solo puede acceder a los recursos como defina el administrador de clústeres.

La autenticación de Azure AD se proporciona a los clústeres de AKS con OpenID Connect. OpenID Connect es una capa de identidad creada basándose en el protocolo OAuth 2.0. Puede encontrar más información sobre OpenID Connect en la [documentación de OpenID Connect][openid-connect]. Dentro del clúster de Kubernetes, se usa [la autenticación de token de webhook][webhook-token-docs] para verificar los tokens de autenticación. La autenticación de token de webhook se configura y administra como parte del clúster de AKS.

### <a name="webhook-and-api-server"></a>Webhook y servidor de API

![Flujo de autenticación del servidor de API y webhook](media/concepts-identity/auth-flow.png)

Como se muestra en el gráfico anterior, el servidor de API llama al servidor de webhook de AKS y realiza los pasos siguientes:

1. `kubectl` usa la aplicación cliente de Azure AD para el inicio de sesión de los usuarios con el [flujo de concesión de autorización de dispositivos OAuth 2.0](../active-directory/develop/v2-oauth2-device-code.md).
2. Azure AD proporciona un access_token, id_token y un refresh_token.
3. El usuario realiza una solicitud a `kubectl` con un access_token de `kubeconfig`.
4. `kubectl` envía access_token al servidor de API.
5. El servidor de API se configura con el servidor de autenticación de Webhook para realizar la validación.
6. El servidor de autenticación de webhook confirma que la firma JSON Web Token es válida mediante la comprobación de la clave de firma pública de Azure AD.
7. La aplicación del servidor usa credenciales proporcionadas por el usuario para consultar la pertenencia a grupos del usuario que ha iniciado sesión desde la MS Graph API.
8. Se envía una respuesta al servidor de API con información del usuario, como la notificación del nombre principal de usuario (UPN) del token de acceso y la pertenencia al grupo del usuario en función del identificador de objeto.
9. La API realiza una decisión de autorización basada en el rol Kubernetes/RoleBinding.
10. Una vez autorizado, el servidor de API devuelve una respuesta a `kubectl`.
11. `kubectl` envía comentarios al usuario.
 
Obtenga información sobre cómo integrar AKS con Azure AD con nuestra [guía paso a paso de integración de Azure AD administrada por AKS](managed-aad.md).

## <a name="azure-role-based-access-control"></a>Control de acceso basado en roles de Azure

El control de acceso basado en rol (RBAC) de Azure es un sistema de autorización basado en [Azure Resource Manager](../azure-resource-manager/management/overview.md) que proporciona administración de acceso específico a los recursos de Azure.

| Sistema RBAC | Descripción |
|---|---|
| RBAC de Kubernetes | Diseñado para funcionar en recursos de Kubernetes dentro del clúster de AKS. |
| Azure RBAC | Diseñado para funcionar en recursos dentro de la suscripción de Azure. |

Con el control de acceso basado en rol de Azure, puede crear una *definición de rol* que describe los permisos que se aplicarán. A continuación, asigne esta definición de rol a un usuario o grupo mediante una *asignación de roles* para un *ámbito* determinado. El ámbito puede ser un recurso individual, un grupo de recursos o toda la suscripción.

Para obtener más información, consulte [¿Qué es el control de acceso basado en roles (RBAC) de Azure?][azure-rbac]

Hay dos niveles de acceso necesarios para operar completamente un clúster de AKS: 
* [Acceso al recurso de AKS en la suscripción de Azure](#azure-rbac-to-authorize-access-to-the-aks-resource) 
  * Controle el escalado o la actualización del clúster mediante las API de AKS.
  * Extraiga `kubeconfig`.
* Acceso al API de Kubernetes. Este acceso se controla mediante:
  * [RBAC de Kubernetes](#kubernetes-rbac) (tradicionalmente).
  * [Integración de RBAC de Azure con AKS para la autorización de Kubernetes](#azure-rbac-for-kubernetes-authorization-preview).

### <a name="azure-rbac-to-authorize-access-to-the-aks-resource"></a>Azure RBAC para autorizar el acceso al recurso de AKS

Con Azure RBAC, puede proporcionar a los usuarios (o identidades) acceso granular a los recursos de AKS en una o varias suscripciones. Por ejemplo, puede usar el [rol de colaborador de Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-contributor-role) para escalar y actualizar el clúster. Mientras, otro usuario con el [rol de administrador de clúster de Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-admin-role) solo tiene permiso para extraer el `kubeconfig` de administración.

Como alternativa, puede dar a su usuario el rol de [colaborador](../role-based-access-control/built-in-roles.md#contributor) general. Con el rol de colaborador general, los usuarios tienen los permisos anteriores y pueden realizar todas las acciones posibles en el recurso de AKS, excepto la administración de permisos.

[Use Azure RBAC para definir el acceso al archivo de configuración de Kubernetes en AKS](control-kubeconfig-access.md).

### <a name="azure-rbac-for-kubernetes-authorization-preview"></a>Azure RBAC para la autorización de Kubernetes (versión preliminar)

Con la integración de Azure RBAC, AKS utilizará un servidor de webhooks de autorización de Kubernetes, lo que le permitirá administrar los permisos y asignaciones de los recursos del clúster de Kubernetes integrados en Azure AD utilizando la definición y la asignación de roles de Azure.

![Azure RBAC para el flujo de autorización de Kubernetes](media/concepts-identity/azure-rbac-k8s-authz-flow.png)

Como se muestra en el diagrama anterior, al usar la integración de Azure RBAC, todas las solicitudes a la API de Kubernetes seguirán el mismo flujo de autenticación que se explica en la [sección de integración de Azure Active Directory](#azure-ad-integration). 

Si la identidad que realiza la solicitud existe en Azure AD, Azure colaborará con RBAC de Kubernetes para autorizar la solicitud. Si la identidad existe fuera de Azure AD (es decir, una cuenta de servicio de Kubernetes), la autorización usará al RBAC de Kubernetes normal.

En este escenario, usará mecanismos y API de RBAC de Azure para asignar roles integrados a los usuarios o crear roles personalizados, tal como lo haría con los roles de Kubernetes. 

Con esta característica, no solo concede permisos a los usuarios para el recurso de AKS entre suscripciones, sino que también configura el rol y los permisos dentro de cada uno de esos clústeres que controlan el acceso a la API de Kubernetes. Por ejemplo, puede conceder el rol `Azure Kubernetes Service RBAC Viewer` en el ámbito de la suscripción. El destinatario del rol podrá enumerar y obtener todos los objetos de Kubernetes de todos los clústeres sin modificarlos.

> [!IMPORTANT]
> Debe habilitar Azure RBAC para la autorización de Kubernetes antes de usar esta característica. Para obtener información detallada e instrucciones paso a paso, siga nuestra guía paso a paso [Uso de Azure RBAC para la autorización de Kubernetes](manage-azure-rbac.md).

#### <a name="built-in-roles"></a>Roles integrados

AKS proporciona los siguientes cuatro roles integrados. Son similares a los [roles integrados de Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles) con algunas diferencias, como la compatibilidad con CRDs. Consulte la lista completa de acciones permitidas por cada [rol integrado de Azure](../role-based-access-control/built-in-roles.md).

| Role                                | Descripción  |
|-------------------------------------|--------------|
| Visor de Azure Kubernetes Service RBAC  | Permite el acceso de solo lectura para ver la mayoría de los objetos en un espacio de nombres. <br> No permite la visualización de roles o enlaces de roles.<br> No permite la visualización de `Secrets`. Leer el contenido de `Secrets` permite el acceso a las credenciales de `ServiceAccount` en el espacio de nombres, lo que permitiría el acceso a la API como cualquier `ServiceAccount` en el espacio de nombres (una forma de elevación de privilegios).  |
| Escritor de Azure Kubernetes Service RBAC | Permite el acceso de lectura y escritura para ver la mayoría de los objetos en un espacio de nombres. <br> No permite la visualización o modificación de roles o enlaces de roles. <br> Permite acceder a `Secrets` y ejecutar pods como cualquier ServiceAccount en el espacio de nombres, por lo que se puede usar para obtener los niveles de acceso de la API de cualquier ServiceAccount en el espacio de nombres. |
| Administrador de Azure Kubernetes Service RBAC  | Permite el acceso de administrador, diseñado para su concesión dentro de un espacio de nombres. <br> Permite el acceso de lectura y escritura a la mayoría de los recursos de un espacio de nombres (o ámbito de clúster), incluida la capacidad de crear roles y enlaces de roles dentro del espacio de nombres. <br> No permite el acceso de escritura a la cuota de recursos o al espacio de nombres en sí. |
| Administrador de clúster de Azure Kubernetes Service RBAC  | Permite el acceso de superusuario para realizar cualquier acción en cualquier recurso. <br> Proporciona control total sobre todos los recursos del clúster y en todos los espacios de nombres. |


## <a name="summary"></a>Resumen

Vea en la tabla un resumen rápido de cómo los usuarios pueden autenticarse en Kubernetes cuando la integración de Azure AD está habilitada. En todos los casos, la secuencia de comandos del usuario es:
1. Ejecute `az login` para autenticarse en Azure.
1. Ejecute `az aks get-credentials` para descargar las credenciales del clúster en `.kube/config`.
1. Ejecute comandos `kubectl`. 
   * El primer comando puede desencadenar la autenticación basada en explorador para autenticarse en el clúster, tal y como se describe en la tabla siguiente.

En Azure Portal, puede encontrar lo siguiente:
* La *concesión de roles* (concesión de roles de Azure RBAC) a la que se hace referencia en la segunda columna se muestra en la pestaña **Control de acceso**. 
* El grupo de Azure AD de administración del clúster se muestra en la pestaña **Configuración**.
  * También se encuentra con el nombre de parámetro `--aad-admin-group-object-ids` en la CLI de Azure.


| Descripción        | Concesión de rol requerida| Grupos de administración de clústeres de Azure AD | Cuándo se usa |
| -------------------|------------|----------------------------|-------------|
| Inicio de sesión de administrador heredado con certificado de cliente| **Rol de administrador de Azure Kubernetes**. Este rol permite usar `az aks get-credentials` con la marca `--admin`, que descarga un [certificado de administrador de clústeres heredado (no de Azure AD)](control-kubeconfig-access.md) en `.kube/config` del usuario. Este es el único propósito del "rol de administrador de Azure Kubernetes".|N/D|Si está bloqueado de forma permanente porque no tiene acceso a un grupo válido de Azure AD con acceso al clúster.| 
| Azure AD con (Cluster)RoleBindings manual| **Rol de usuario de Azure Kubernetes**. El rol "Usuario" permite usar `az aks get-credentials` sin la marca `--admin`. (Este es el único propósito del "rol de usuario de Azure Kubernetes"). El resultado, en un clúster habilitado para Azure AD, es la descarga de [una entrada vacía](control-kubeconfig-access.md) en `.kube/config`, que desencadena la autenticación basada en explorador cuando se usa por primera vez en `kubectl`.| El usuario no se encuentra en ninguno de estos grupos. Dado que el usuario no está en ningún grupo de administración de clústeres, cualquier RoleBindings o ClusterRoleBindings configurado por los administradores de clústeres controlarán por completo sus derechos. (Cluster)RoleBindings [designa los usuarios de Azure AD o los grupos de Azure AD](azure-ad-rbac.md) como sus `subjects`. Si no se ha configurado ningún enlace de este tipo, el usuario no podrá ejecutar ningún comando `kubectl`.|Si desea tener un control de acceso específico y no usa RBAC de Azure para la autorización de Kubernetes. Tenga en cuenta que el usuario que configura los enlaces debe iniciar sesión con uno de los otros métodos enumerados en esta tabla.|
| Azure AD por miembro de grupo de administración| Lo mismo que antes.|El usuario es miembro de uno de los grupos que se indican aquí. AKS genera automáticamente un elemento ClusterRoleBinding que enlaza todos los grupos indicados al rol `cluster-admin` de Kubernetes. De este modo, los usuarios de estos grupos pueden ejecutar todos los comandos `kubectl` como `cluster-admin`.|Si desea conceder de forma práctica a los usuarios derechos de administrador completos y _no_ usan RBAC de Azure para la autorización de Kubernetes.|
| Azure AD con RBAC de Azure para la autorización de Kubernetes|Dos roles: <br> En primer lugar, el **rol de usuario de Azure Kubernetes**. <br> En segundo lugar, uno de los roles **RBAC** de "Azure Kubernetes Service" indicados anteriormente, o su propia alternativa personalizada.|El campo de roles de administrador de la pestaña Configuración es irrelevante cuando está habilitada la autorización de RBAC de Azure para Kubernetes.|Usa RBAC de Azure para la autorización de Kubernetes. Este enfoque proporciona un control específico, sin necesidad de configurar RoleBindings o ClusterRoleBindings.|

## <a name="next-steps"></a>Pasos siguientes

- Para empezar a trabajar con Azure AD y el RBAC de Kubernetes, consulte [Integración de Azure Active Directory con AKS][aks-aad].
- Para conocer los procedimientos recomendados asociados, consulte [Procedimientos recomendados para la autenticación y la autorización en AKS][operator-best-practices-identity].
- Para empezar a trabajar con Azure RBAC para la autorización de Kubernetes, consulte [usar Azure RBAC para autorizar el acceso en el clúster de Azure Kubernetes Service (AKS)](manage-azure-rbac.md).
- Para empezar a proteger el archivo `kubeconfig`, consulte [Limitación del acceso al archivo de configuración de clúster](control-kubeconfig-access.md).

Para obtener más información sobre los conceptos básicos de Kubernetes y AKS, consulte los artículos siguientes:

- [Clústeres y cargas de trabajo de Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Seguridad de Kubernetes/AKS][aks-concepts-security]
- [Redes virtuales de Kubernetes/AKS][aks-concepts-network]
- [Almacenamiento de Kubernetes/AKS][aks-concepts-storage]
- [Escala de Kubernetes/AKS][aks-concepts-scale]

<!-- LINKS - External -->
[kubernetes-authentication]: https://kubernetes.io/docs/reference/access-authn-authz/authentication
[webhook-token-docs]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubernetes-rbac]: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

<!-- LINKS - Internal -->
[openid-connect]: ../active-directory/develop/v2-protocols-oidc.md
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[azure-rbac]: ../role-based-access-control/overview.md
[aks-aad]: managed-aad.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[operator-best-practices-identity]: operator-best-practices-identity.md
[upgrade-per-cluster]: ../azure-monitor/containers/container-insights-update-metrics.md#upgrade-per-cluster-using-azure-cli
