---
title: Autenticación del clúster de AKS a Azure Container Registry en un inquilino de AD diferente
description: Configure una entidad de servicio de un clúster de AKS con permisos para acceder a Azure Container Registry en un inquilino de AD diferente.
ms.topic: article
author: dlepow
ms.author: danlep
ms.date: 05/21/2021
ms.openlocfilehash: 154ef93ca89c6d117d23a80986fe30ef2e14a426
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111442223"
---
# <a name="pull-images-from-a-container-registry-to-an-aks-cluster-in-a-different-azure-ad-tenant"></a>Extracción de imágenes de un registro de contenedor a un clúster de AKS en un inquilino de Azure AD diferente

En algunos casos, es posible que tenga el clúster de Azure AKS en un inquilino de Azure Active Directory (Azure AD) y la instancia de Azure Container Registry en otro inquilino. En este artículo encontrará una guía por los pasos necesarios para habilitar la autenticación entre inquilinos mediante la credencial de la entidad de servicio de AKS para extraer del registro de contenedor.

## <a name="scenario-overview"></a>Información general de escenario
Las suposiciones para este ejemplo son:

* El clúster de AKS está en el **Inquilino A** y la instancia de Azure Container Registry está en el **Inquilino B**. 
* El clúster de AKS se configura con autenticación de entidad de servicio en el **Inquilino A**. Obtenga más información sobre cómo crear y usar una [entidad de servicio para un clúster de AKS](../aks/kubernetes-service-principal.md).

Necesita al menos el rol Colaborador en la suscripción del clúster de AKS, y el rol Propietario en la suscripción del registro de contenedor.

Seguirá los pasos a continuación:

* Cree una aplicación multiinquilino (entidad de servicio) en el **Inquilino A**. 
* Aprovisione la aplicación en el **Inquilino B**.
* Configure la entidad de servicio para extraer del registro en el **Inquilino B**.
* Actualice el clúster de AKS en el **Inquilino A** para autenticarse con la nueva entidad de servicio.


## <a name="step-by-step-instructions"></a>Instrucciones paso a paso

### <a name="step-1-create-multitenant-azure-ad-application"></a>Paso 1: Crear una aplicación multiinquilino de Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) en el **Inquilino A**.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones > + Nuevo registro**.
1. En **Tipos de cuenta compatibles**, seleccione **Cuentas en cualquier directorio organizativo**.
1. En **URI de redirección**, escriba *https://www.microsoft.com* .
1. Seleccione **Registrar**.
1. En la página **Información general**, tome nota del **Id. de aplicación (cliente)** . Se usará en los pasos 2 y 4.

    :::image type="content" source="media/authenticate-kubernetes-cross-tenant/service-principal-overview.png" alt-text="Id. de aplicación de la entidad de servicio":::
1. En **Certificados y secretos**, en **Secretos de cliente**, seleccione **+ Nuevo secreto de cliente**.
1. Escriba una **Descripción**, como *Contraseña*, y seleccione **Agregar**.
1. En **Secretos de cliente**, tome nota del valor del secreto de cliente. Se usa para actualizar la entidad de servicio del clúster de AKS en el paso 4.

    :::image type="content" source="media/authenticate-kubernetes-cross-tenant/configure-client-secret.png" alt-text="Configuración del secreto de cliente":::
### <a name="step-2-provision-the-service-principal-in-the-acr-tenant"></a>Paso 2: Aprovisionar la entidad de servicio en el inquilino de ACR

1. Abra el vínculo siguiente con una cuenta de administrador en el **Inquilino B**. Cuando se le indique, inserte el **Id. del Inquilino B** y el **Id. de aplicación** (Id. de cliente) de la aplicación multiinquilino.

    ```console
    https://login.microsoftonline.com/<Tenant B ID>/oauth2/authorize?client_id=<Multitenant application ID>&response_type=code&redirect_uri=<redirect url>
    ```
1. Seleccione **Consentimiento en nombre de la organización** y, luego, **Aceptar**. 
    
    :::image type="content" source="media/authenticate-kubernetes-cross-tenant/multitenant-app-consent.png" alt-text="Concesión de acceso de inquilino a la aplicación":::
 

### <a name="step-3-grant-service-principal-permission-to-pull-from-registry"></a>Paso 3: Conceder permiso a la entidad de servicio para extraer del registro

En el **Inquilino B**, asigne el rol AcrPull a la entidad de servicio, con ámbito en el registro de contenedor de destino. Puede usar [Azure Portal](../role-based-access-control/role-assignments-portal.md) u otras herramientas para asignar el rol. Para ver los pasos de ejemplo con la CLI de Azure, consulte [Autenticación de Azure Container Registry con entidades de servicio](container-registry-auth-service-principal.md#use-an-existing-service-principal).

:::image type="content" source="media/authenticate-kubernetes-cross-tenant/multitenant-app-acr-pull.png" alt-text="Asignación del rol AcrPull a una aplicación multiinquilino":::

### <a name="step-4-update-aks-with-the-azure-ad-application-secret"></a>Paso 4: Actualizar AKS con el secreto de aplicación de Azure AD

Use el identificador de la aplicación multiinquilino (cliente) y el secreto de cliente recopilados en el paso 1 para [actualizar la credencial de la entidad de servicio de AKS](../aks/update-credentials.md#update-aks-cluster-with-new-service-principal-credentials).

La actualización de la entidad de servicio puede tardar varios minutos.

## <a name="next-steps"></a>Pasos siguientes

* Más información en [Autenticación de Azure Container Registry con entidades de servicio](container-registry-auth-service-principal.md)
* Más información sobre los secretos de extracción de imágenes en la [documentación de Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod)
- Más información sobre [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md)


