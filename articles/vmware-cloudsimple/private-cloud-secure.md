---
title: 'Azure VMware Solutions by CloudSimple: nube privada segura'
description: Se describe cómo proteger la nube privada de Azure VMware Solutions by CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 2cf73cfcad43f34bc3977ab7818bb9623d56c4aa
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177267"
---
# <a name="how-to-secure-your-private-cloud-environment"></a>Cómo proteger el entorno de nube privada

Establezca el control de acceso basado en rol (RBAC) en el servicio CloudSimple, en el portal de CloudSimple y en la nube privada de Azure.  Los usuarios, grupos y roles necesarios para acceder a la instancia de vCenter de la nube privada se especifican mediante SSO de VMware.  

## <a name="azure-rbac-for-cloudsimple-service"></a>Azure RBAC en el servicio CloudSimple

La creación del servicio CloudSimple requiere el rol **Propietario** o **Colaborador** en la suscripción de Azure.  Todos los propietarios y colaboradores pueden crear un servicio CloudSimple y acceder al portal de CloudSimple de forma predeterminada para crear y administrar nubes privadas.  Solo se puede crear un servicio CloudSimple por región.  Para restringir el acceso a solo algunos administradores, realice el procedimiento que se indica a continuación.

1. Cree un servicio CloudSimple en un **grupo de recursos** nuevo en Azure Portal.
2. Especifique Azure RBAC en el grupo de recursos.
3. Adquiera nodos y use el mismo grupo de recursos que el servicio CloudSimple.

Solo los usuarios que tienen privilegios de **Propietario** o **Colaborador** en el grupo de recursos verán el servicio CloudSimple e iniciarán el portal de CloudSimple.

Para obtener más información, consulte [¿Qué es el control de acceso basado en roles (RBAC) de Azure?](../role-based-access-control/overview.md)

## <a name="rbac-for-private-cloud-vcenter"></a>RBAC para vCenter de nube privada

Cuando se crea una nube privada, se crea un usuario predeterminado `CloudOwner@cloudsimple.local` en el dominio de SSO de vCenter.  El usuario CloudOwner tiene privilegios para administrar vCenter. Se irán agregando más orígenes de identidades al SSO de vCenter para permitir el acceso a distintos usuarios.  Se configurarán roles y grupos predefinidos en vCenter para usarlos para agregar más usuarios.

### <a name="add-new-users-to-vcenter"></a>Adición de nuevos usuarios a vCenter

1. [Escale privilegios](escalate-private-cloud-privileges.md) para el usuario **CloudOwner\@cloudsimple.local** en la nube privada.
2. Inicie sesión en vCenter con **CloudOwner\@cloudsimple.local**.
3. [Agregue usuarios de inicio de sesión único de vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-72BFF98C-C530-4C50-BF31-B5779D2A4BBB.html).
4. Agregue usuarios a [grupos de inicio de sesión único de vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-CDEA6F32-7581-4615-8572-E0B44C11D80D.html).

Para más información sobre los roles y grupos predefinidos, vea el artículo [Modelo de permisos de la nube privada de VMware vCenter de CloudSimple](learn-private-cloud-permissions.md).

### <a name="add-new-identity-sources"></a>Adición de nuevos orígenes de identidades

Se pueden agregar más proveedores de identidades relativos al dominio de SSO de vCenter de la nube privada.  Los proveedores de identidades facilitan la característica de autenticación y los grupos de SSO de vCenter proporcionan autorización a los usuarios.

* [Use Active Directory como proveedor de identidades](set-vcenter-identity.md) en la instancia de vCenter de nube privada.
* [Use Active AD como proveedor de identidades](azure-ad.md) en la instancia de vCenter de nube privada.

1. [Escale privilegios](escalate-private-cloud-privileges.md) para el usuario **CloudOwner\@cloudsimple.local** en la nube privada.
2. Inicie sesión en vCenter con **CloudOwner\@cloudsimple.local**.
3. Agregue usuarios desde el proveedor de identidades a los [grupos de inicio de sesión único de vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-CDEA6F32-7581-4615-8572-E0B44C11D80D.html).

## <a name="secure-network-on-your-private-cloud-environment"></a>Protección de red en el entorno de nube privada

La seguridad de red del entorno de nube privada se controla mediante la protección del acceso a la red y el control del tráfico de red entre los recursos.

### <a name="access-to-private-cloud-resources"></a>Acceso a los recursos de nube privada

El acceso a los recursos y a la instancia de vCenter de nube privada se realiza a través de una conexión de red segura:

* **[Conexión ExpressRoute](on-premises-connection.md)** . ExpressRoute proporciona una conexión segura de baja latencia y ancho de banda elevado desde el entorno local.  Mediante esta conexión, los servicios locales, las redes y los usuarios tienen acceso a la instancia de vCenter de nube privada.
* **[Puerta de enlace de VPN de sitio a sitio](vpn-gateway.md)** . Las VPN de sitio a sitio proporcionan acceso a los recursos de nube privada desde el entorno local a través de un túnel seguro.  Hay que especificar qué redes locales pueden enviar y recibir tráfico de red a la nube privada.
* **[Puerta de enlace de VPN de punto a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway)** . Use una conexión VPN de punto a sitio para acceder de forma remota rápida a la instancia de vCenter de nube privada.

### <a name="control-network-traffic-in-private-cloud"></a>Control del tráfico de red en la nube privada

Las reglas y tablas de firewall controlan el tráfico de red en la nube privada.  La tabla de firewall permite controlar el tráfico de red entre una red o una dirección IP de origen y una red o dirección IP de destino en función de la combinación de reglas definida en la tabla en cuestión.

1. Cree una [tabla de firewall](firewall.md#add-a-new-firewall-table).
2. [Agregue reglas](firewall.md#create-a-firewall-rule) a la tabla de firewall.
3. [Asocie una tabla de firewall a una VLAN o una subred](firewall.md#attach-vlans-subnet).
