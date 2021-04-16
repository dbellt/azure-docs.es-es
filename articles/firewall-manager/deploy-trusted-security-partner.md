---
title: Implementación de un proveedor de asociados de seguridad de Azure Firewall Manager
description: Más información sobre cómo implementar un proveedor de asociados de seguridad de Azure Firewall Manager con el Azure Portal.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: how-to
ms.date: 03/31/2021
ms.author: victorh
ms.openlocfilehash: b8e10eef89df12807cabd96d64d9c7d659f91d6c
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106109516"
---
# <a name="deploy-a-security-partner-provider"></a>Implementación de un proveedor de seguridad asociado

*Los proveedores de seguridad asociados* en Azure Firewall Manager permiten usar las mejores ofertas de seguridad como servicio (SECaaS) de terceros que ya conoce para proteger el acceso a internet para los usuarios.

Para más información acerca de los escenarios admitidos y las directrices de prácticas recomendadas, consulte [¿Qué son los proveedores de seguridad asociados?](trusted-security-partners.md)


Los asociados integrados de seguridad como servicio (SECaaS) de terceros ya están disponibles: 

- **Zscaler**
- **[Check Point](check-point-overview.md)**
- **iboss**

## <a name="deploy-a-third-party-security-provider-in-a-new-hub"></a>Implementación de un proveedor de seguridad de terceros en un nuevo centro de conectividad

Omita esta sección si va a implementar un proveedor de terceros en un centro existente.

1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. En **Búsqueda**, escriba **Firewall Manager** y selecciónelo en **Servicios**.
3. Vaya a **Introducción**. Seleccione **Ver los concentradores virtuales protegidos**.
4. Seleccione **Crear un nuevo concentrador virtual protegido**.
5. Escriba la suscripción y el grupo de recursos, seleccione una región admitida y agregue la información de la virtual WAN y del centro de conectividad. 
6. Seleccione **Incluir VPN Gateway para habilitar los proveedores de seguridad asociados**.
7. Seleccione las **unidades de escala de Gateway** adecuadas para sus requisitos.
8. Seleccione **Siguiente: Azure Firewall**
   > [!NOTE]
   > Los proveedores de seguridad asociados se conectan a su centro con túneles de VPN Gateway. Si elimina el VPN Gateway, se perderán las conexiones con los proveedores de seguridad asociados.
9. Si desea implementar Azure Firewall para filtrar el tráfico privado junto con el proveedor de servicios externo para filtrar el tráfico de Internet, seleccione una directiva para Azure Firewall. Vea los [escenarios compatibles](trusted-security-partners.md#key-scenarios).
10. Si solo desea implementar un proveedor de seguridad de terceros en el centro de conectividad, seleccione **Azure Firewall: Habilitada/deshabilitada** para establecerla como **deshabilitada**. 
11. Seleccione **Siguiente: Proveedores de seguridad asociados**.
12. Establezca **Proveedores de seguridad asociados** en **Habilitado**. 
13. Seleccione un socio. 
14. Seleccione **Siguiente: Review + create** (Revisar y crear). 
15. Revise el contenido y, luego seleccione **Crear**.

La implementación de la puerta de enlace VPN puede durar más de 30 minutos.

Para comprobar que se ha creado el centro de conectividad, vaya a Azure Firewall Manager-> Centros de conectividad protegidos. Seleccione la página del centro de conectividad->Información general para mostrar el nombre del socio y el estado como **Conexión de seguridad pendiente**.

Una vez que se crea el centro de conectividad y se configure el socio de seguridad, continúe con la conexión al proveedor de seguridad al centro de conectividad.

## <a name="deploy-a-third-party-security-provider-in-an-existing-hub"></a>Implementación de un proveedor de seguridad de terceros en un nuevo centro de conectividad

También puede seleccionar un centro de conectividad existente en una Virtual WAN y convertirlo en un *centro virtual protegido*.

1. En **Introducción**, seleccione **Ver concentradores virtuales protegidos**.
2. Seleccione **Convertir los concentradores existentes**.
3. Seleccione una suscripción y un centro de conectividad existente. Siga los pasos restantes para implementar un proveedor de terceros en un nuevo centro de conectividad.

Recuerde que se debe implementar una puerta de enlace de VPN para convertir un centro de conectividad existente en un centro protegido con proveedores externos.

## <a name="configure-third-party-security-providers-to-connect-to-a-secured-hub"></a>Configuración de proveedores de seguridad de terceros para conectarse a un centro de conectividad protegido

Para configurar túneles en VPN Gateway del centro de conectividad virtual, los proveedores de terceros necesitan derechos de acceso al centro. Para ello, asocie una entidad de servicio a la suscripción o grupo de recursos, y conceda derechos de acceso. A continuación, debe proporcionar estas credenciales a terceros mediante el portal.

### <a name="create-and-authorize-a-service-principal"></a>Crear y autorizar una entidad de servicio

1. Crear una entidad de servicio de Azure Active Directory (AD): Puede omitir la dirección URL de redireccionamiento. 

   [Cómo: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)
2. Agregue los derechos de acceso y el ámbito de la entidad de servicio.
   [Cómo: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)

   > [!NOTE]
   > Puede limitar el acceso solo a su grupo de recursos para un control más granular.

### <a name="visit-partner-portal"></a>Visite el portal asociado

1. Siga las instrucciones proporcionadas por el asociado para completar la instalación. Esto incluye el envío de información de AAD para detectar y conectarse al concentrador, actualizar las directivas de salida y comprobar el estado de la conectividad y los registros.

   - [ZScaler: Configure una integración Virtual WAN de Microsoft Azure](https://help.zscaler.com/zia/configuring-microsoft-azure-virtual-wan-integration).
   - [Check Point: Configure una integración Virtual WAN de Microsoft Azure](https://sc1.checkpoint.com/documents/Infinity_Portal/WebAdminGuides/EN/CloudGuard-Connect-Azure-Virtual-WAN/Default.htm).
   - [iboss: Configure una integración Virtual WAN de Microsoft Azure](https://www.iboss.com/blog/securing-microsoft-azure-with-iboss-saas-network-security). 
   
2. Puede ver el estado de creación del túnel en el portal de la Azure Virtual WAN en Azure. Una vez que los túneles se muestren **conectados** tanto en Azure como en el portal asociado, continúe con los siguientes pasos para configurar las rutas en la selección de las sucursales y VNets que deberán enviar el tráfico de Internet al asociado.

## <a name="configure-security-with-firewall-manager"></a>Configuración de la seguridad con Firewall Manager

1. Examine Azure Firewall Manager -> Centros de conectividad seguros. 
2. Seleccione un centro de conectividad. El estado del centro de conectividad debe mostrar ahora **Aprovisionado** en lugar de **Conexión de seguridad pendiente**.

   Asegúrese de que el proveedor de terceros pueda conectarse al centro de conectividad. El estado de los túneles de la puerta de enlace VPN deben ser **Conectados**. Este estado refleja mejor el estado de la conexión entre el centro de conectividad y el socio de terceros, en comparación con el estado anterior.
3. Seleccione el centro de conectividad y vaya a **Opciones de configuración de seguridad**.

   Cuando se implementa un proveedor externo en el centro de conectividad, este se convierte en un *centro virtual protegido*. Esto garantiza que el proveedor de terceros anuncia una ruta 0.0.0.0.0/0 (predeterminada) hacia el centro de conectividad. Sin embargo, las conexiones de VNet y los sitios conectados al centro de conectividad no reciben esta ruta a menos que elija qué conexiones deben recibir esta ruta predeterminada.
4. Configure la seguridad de WAN virtual estableciendo el **tráfico de Internet** a través de Azure Firewall y el **tráfico privado** mediante un asociado de seguridad de confianza. Esto protege automáticamente las conexiones individuales en la WAN virtual.

   :::image type="content" source="media/deploy-trusted-security-partner/security-configuration.png" alt-text="Configuración de seguridad":::
5. Además, si su organización usa intervalos de direcciones IP públicas en redes virtuales y sucursales, debe especificar los prefijos IP explícitamente mediante **prefijos de tráfico privados**. Los prefijos de direcciones IP públicas se pueden especificar individualmente o como agregados.

## <a name="branch-or-vnet-internet-traffic-via-third-party-service"></a>Tráfico de Internet de sucursales o VNet a través de un servicio de terceros

A continuación, puede comprobar si las máquinas virtuales VNet o la sucursal pueden tener acceso a Internet y validar que el tráfico fluye hacia el servicio de terceros.

Una vez finalizados los pasos de configuración de la ruta, las máquinas virtuales VNet y las sucursales reciben una ruta de servicio de 0/0 a terceros. No puede RDP o SSH en estas máquinas virtuales. Para iniciar sesión, puede implementar el servicio [Azure Bastion](../bastion/bastion-overview.md) en una red VNet emparejada.

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Protección de una red en la nube con Azure Firewall Manager mediante Azure Portal](secure-cloud-network.md)