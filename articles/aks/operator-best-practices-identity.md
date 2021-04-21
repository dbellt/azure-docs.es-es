---
title: Procedimientos recomendados para la administración de identidad
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre los procedimientos recomendados del operador de clústeres para saber cómo administrar la autenticación y autorización de los clústeres en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 03/09/2021
ms.author: jpalma
author: palma21
ms.openlocfilehash: de84e3e2a8da3e1b5195978a8a2204fdfa2108d7
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105109"
---
# <a name="best-practices-for-authentication-and-authorization-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para la autenticación y autorización en Azure Kubernetes Service (AKS)

A medida que implementa y mantiene clústeres en Azure Kubernetes Service (AKS), deberá implementar maneras de administrar el acceso a los recursos y servicios. Sin estos controles:
* Las cuentas podrían tener acceso a recursos y servicios innecesarios. 
* Se dificulta el seguimiento del conjunto de credenciales que se usaron para los cambios.

Este artículo de procedimientos recomendados se centra en cómo un operador de clústeres puede administrar el acceso y la identidad de los clústeres de AKS. En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Autenticar a los usuarios de clústeres de AKS con Azure Active Directory.
> * Controlar el acceso a los recursos con el control de acceso basado en roles de Kubernetes (RBAC de Kubernetes).
> * Usar RBAC de Azure para controlar en detalle el acceso al recurso de AKS, a Kubernetes API a escala y a `kubeconfig`.
> * Usar una identidad administrada para autenticar los pods con otros servicios.

## <a name="use-azure-active-directory-azure-ad"></a>Uso de Azure Active Directory (Azure AD)

> **Guía de procedimientos recomendados** 
> 
> Implementación de clústeres de AKS con integración de Azure AD. Mediante Azure AD se centraliza el componente de administración de identidades. Cualquier cambio en el estado del grupo o cuenta del usuario se actualiza automáticamente al acceder al clúster de AKS. Establezca el ámbito de los usuarios o grupos en la cantidad mínima de permisos mediante [roles, ClusterRoles o enlaces](#use-kubernetes-role-based-access-control-kubernetes-rbac).

Los desarrolladores del clúster de Kubernetes y propietarios de aplicaciones necesitan acceder a diferentes recursos. Kubernetes carece de una solución de administración de identidades para controlar los recursos con los que los usuarios pueden interactuar. En su lugar, el usuario suele integrar el clúster con una solución de identidades existente. Introduzca Azure AD: una solución de administración de identidades para el ámbito empresarial que se integra con los clústeres de AKS.

Con los clústeres integrados con Azure AD en AKS, podrá crear *roles* o *ClusterRoles* que definan los permisos de acceso a los recursos. A continuación, *enlaza* los roles a usuarios o grupos de Azure AD. Más información sobre estos RBAC de Kubernetes en [la sección siguiente](#use-kubernetes-role-based-access-control-kubernetes-rbac). En el diagrama siguiente se muestran la integración de Azure AD y la forma de controlar el acceso a los recursos:

![Autenticación a nivel de clúster para la integración de Azure Active Directory con AKS](media/operator-best-practices-identity/cluster-level-authentication-flow.png)

1. El desarrollador se autentica con Azure AD.
1. El punto de conexión de emisión de tokens de Azure AD emite el token de acceso.
1. El desarrollador realiza una acción con el token de Azure AD, como `kubectl create pod`.
1. Kubernetes valida el token con Azure AD y recupera las pertenencias a grupos del desarrollador.
1. Se aplican las directivas del clúster y de RBAC de Kubernetes.
1. La solicitud del desarrollador es correcta en función de la validación anterior de pertenencia a grupos de Azure AD, y las directivas y RBAC de Kubernetes.

Para crear un clúster de AKS que use Azure AD, consulte [Integración de Azure Active Directory con AKS][aks-aad].

## <a name="use-kubernetes-role-based-access-control-kubernetes-rbac"></a>Uso del control de acceso basado en roles de Kubernetes (RBAC de Kubernetes)

> **Guía de procedimientos recomendados**
> 
> Defina permisos de usuario o grupo para agrupar recursos con RBAC de Kubernetes. Cree roles y enlaces que asignen la mínima cantidad de permisos necesarios. Use la integración de Azure AD para actualizar automáticamente cualquier cambio de estado de usuario o pertenencia a grupos y mantener el acceso a los recursos del clúster actual.

En Kubernetes se proporciona control de acceso pormenorizado a los recursos del clúster. Los permisos se definen en el nivel de clúster, así como en espacios de nombres específicos. Determine qué recursos se pueden administrar y con qué permisos. A continuación, aplique estos roles a los usuarios o grupos con un enlace. Para obtener más información sobre los *roles*, *ClusterRoles* y *enlaces*, consulte [Opciones de acceso e identidad en Azure Kubernetes Service (AKS)][aks-concepts-identity].

Por ejemplo, puede crear un rol con acceso total a los recursos del espacio de nombres denominado *finance-app*, tal y como se muestra en el siguiente ejemplo de manifiesto de YAML:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-app-full-access-role
  namespace: finance-app
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
```

A continuación, cree un elemento RoleBinding y enlace el usuario de Azure AD *developer1\@contoso.com* con RoleBinding, como se muestra en el siguiente manifiesto de YAML:

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-app-full-access-role-binding
  namespace: finance-app
subjects:
- kind: User
  name: developer1@contoso.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: finance-app-full-access-role
  apiGroup: rbac.authorization.k8s.io
```

Cuando *developer1\@contoso.com* se autentica en el clúster de AKS, se tienen los permisos completos sobre los recursos en el espacio de nombres *finance-app*. De esta forma, separa y controla el acceso a los recursos de forma lógica. Use RBAC de Kubernetes junto con la integración de Azure AD.

Para ver cómo usar los grupos de Azure AD para controlar el acceso a los recursos de Kubernetes mediante RBAC de Kubernetes, consulte [Administración del acceso a recursos de clúster mediante el control de acceso basado en roles y las identidades de Azure Active Directory en Azure Kubernetes Service][azure-ad-rbac].

## <a name="use-azure-rbac"></a>Uso de Azure RBAC 
> **Guía de procedimientos recomendados** 
> 
> Use RBAC de Azure para definir los permisos mínimos de los usuarios o grupos de los recursos de AKS en una o más suscripciones.

Hay dos niveles de acceso necesarios para operar completamente un clúster de AKS: 
1. Acceda al recurso de AKS en la suscripción de Azure. 

    Este nivel de acceso permite:
      * Controlar el escalado o la actualización del clúster mediante las API de AKS.
      * Extraer `kubeconfig`.

    Para ver cómo controlar el acceso al recurso de AKS y a `kubeconfig`, consulte [Limitación del acceso al archivo de configuración del clúster](control-kubeconfig-access.md).

2. Acceso al API de Kubernetes. 
    
    Este nivel de acceso se controla mediante:
    * [RBAC de Kubernetes](#use-kubernetes-role-based-access-control-kubernetes-rbac) (tradicionalmente), o bien, 
    * mediante la integración de RBAC de Azure con AKS para la autorización de Kubernetes.

    Para ver cómo conceder permisos pormenorizados a Kubernetes API con RBAC de Azure, consulte [Uso de RBAC de Azure para la autorización de Kubernetes](manage-azure-rbac.md).

## <a name="use-pod-managed-identities"></a>Uso de identidades administradas de pod

> **Guía de procedimientos recomendados** 
> 
> No use credenciales fijas dentro de pods o imágenes de contenedor, ya que corren riesgo de exposición o abuso. En su lugar, use *identidades de pod* para solicitar acceso automáticamente mediante una solución central de identidades de Azure AD. 

> [!NOTE]
> Las identidades de pod están pensadas para su uso únicamente con pods de Linux y las imágenes de contenedor. La compatibilidad de las identidades administradas de pod con los contenedores de Windows va a estar disponible próximamente.

Para acceder a otros servicios de Azure, como Cosmos DB, Key Vault o Blob Storage, el pod necesita credenciales de acceso. Puede definir las credenciales de acceso con la imagen de contenedor o insertarlas como secreto de Kubernetes. En cualquier caso, tendría que crearlas y asignarlas manualmente. A menudo, las credenciales se reutilizan en los pods y no se rotan de forma periódica.

Con las identidades administradas de pod para los recursos de Azure, podrá solicitar acceso automáticamente a los servicios por medio de Azure AD. Las identidades administradas de pod están actualmente en versión preliminar para AKS. Consulte la documentación de [Uso de identidades administradas del pod de Azure Active Directory en Azure Kubernetes Service (versión preliminar)] (https://docs.microsoft.com/azure/aks/use-azure-ad-pod-identity) ) para comenzar. 

En lugar de definir manualmente las credenciales de los pods, las identidades administradas por pods solicitan un token de acceso en tiempo real y lo usan para acceder solo a sus servicios asignados. En AKS hay dos componentes que controlan las operaciones para permitir que los pods usen identidades administradas:

* **El servidor de identidad de administración de nodos (NMI)** es un pod que se ejecuta como DaemonSet en cada nodo del clúster de AKS. El servidor de NMI escucha las solicitudes de pods para servicios de Azure.
* **El proveedor de recursos de Azure** consulta al servidor de API de Kubernetes y busca una asignación de identidad de Azure que corresponda a un pod.

Cuando los pods solicitan acceso a un servicio de Azure, las reglas de red redirigen el tráfico al servidor NMI. 
1. El servidor NMI:
    * Identifica los pods que solicitan acceso a los servicios de Azure en función de su dirección remota.
    * Consulta al proveedor de recursos de Azure. 
1. El proveedor de recursos de Azure las asignaciones de identidad de Azure en el clúster de AKS.
1. El servidor NMI solicita un token de acceso de Azure AD en función de la asignación de identidad del pod. 
1. Azure AD proporciona acceso al servidor de NMI, que se devuelve al pod. 
    * El pod puede usar este token de acceso para, después, solicitar acceso a los servicios de Azure.

En el ejemplo siguiente, un desarrollador crea un pod que usa una identidad administrada para solicitar acceso a Azure SQL Database:

![Las identidades de pod permiten que un pod solicite acceso a otros servicios automáticamente.](media/operator-best-practices-identity/pod-identities.png)

1. El operador de los clústeres crea una cuenta de servicio para asignar identidades cuando los pods solicitan acceso a los servicios.
1. El servidor de NMI se implementa para retransmitir las solicitudes de los pods, y del proveedor de recursos de Azure, de tokens de acceso a Azure AD.
1. Un desarrollador implementa un pod con una identidad administrada que solicita un token de acceso a través del servidor de NMI.
1. El token se devuelve al pod y se utiliza para obtener acceso a Azure SQL Database.

> [!NOTE]
> Las identidades administradas de pod están actualmente en estado de versión preliminar.

Para usar identidades administradas de pod, vea [Uso de identidades administradas del pod de Azure Active Directory en Azure Kubernetes Service (versión preliminar)]( https://docs.microsoft.com/azure/aks/use-azure-ad-pod-identity).

## <a name="next-steps"></a>Pasos siguientes

Este artículo de procedimientos recomendados se centra en la autenticación y autorización para el clúster y los recursos. Para implementar algunos de estos procedimientos recomendados, consulte los artículos siguientes:

* [Integración de Azure Active Directory con AKS][aks-aad]
* [Uso de identidades administradas del pod de Azure Active Directory en Azure Kubernetes Service (versión preliminar)]( https://docs.microsoft.com/azure/aks/use-azure-ad-pod-identity)

Para obtener más información acerca de las operaciones de clúster en AKS, consulte los siguientes procedimientos recomendados:

* [Aislamiento de clúster y de multiinquilinato][aks-best-practices-cluster-isolation]
* [Características básicas del programador de Kubernetes][aks-best-practices-scheduler]
* [Características avanzadas del programador de Kubernetes][aks-best-practices-advanced-scheduler]

<!-- EXTERNAL LINKS -->
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity

<!-- INTERNAL LINKS -->
[aks-concepts-identity]: concepts-identity.md
[aks-aad]: azure-ad-integration-cli.md
[managed-identities:]: ../active-directory/managed-identities-azure-resources/overview.md
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[azure-ad-rbac]: azure-ad-rbac.md
