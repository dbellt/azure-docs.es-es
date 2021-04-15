---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 04/01/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c9568c51fe3a59dba83a66fc35c8d9112ade167f
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106178282"
---
Puede implementar perfiles para clientes VPN de Azure (Windows 10) mediante Microsoft Intune. Este artículo le ayuda a crear un perfil de Intune mediante una configuración personalizada.

> [!NOTE]
>* Este artículo se aplica a la implementación de perfiles que utilizan Azure Active Directory solo para la autenticación.


## <a name="prerequisites"></a>Requisitos previos

* Los dispositivos ya están inscritos con MDM de Intune.
* El cliente VPN de Azure para Windows 10 ya está implementado en el equipo cliente.
* Solo se admite la versión 19H2 o posterior de Windows.

## <a name="modify-xml"></a><a name="xml"></a>Modificación de XML

En los pasos siguientes, usamos un XML de ejemplo con un perfil OMA-URI personalizado para Intune con la siguiente configuración:

* Conexión automática activada
* Detección de redes de confianza habilitada.

Para ver otras opciones compatibles, consulte el artículo sobre [CSP de VPNv2](/windows/client-management/mdm/vpnv2-csp).

1. Descargue el perfil de VPN del Azure Portal y extraiga el archivo *azurevpnconfig.xml* del paquete.
1. Copie y pegue el texto siguiente en un nuevo archivo de editor de texto.

   ```xml-interactive
    <VPNProfile>
      <!--<EdpModeId>corp.contoso.com</EdpModeId>-->
      <RememberCredentials>true</RememberCredentials>
      <AlwaysOn>true</AlwaysOn>
      <TrustedNetworkDetection>contoso.com,test.corp.contoso.com</TrustedNetworkDetection>
      <DeviceTunnel>false</DeviceTunnel>
      <RegisterDNS>false</RegisterDNS>
      <PluginProfile>
        <ServerUrlList>azuregateway-7cee0077-d553-4323-87df-069c331f58cb-053dd0f6af02.vpn.azure.com</ServerUrlList> 
        <CustomConfiguration>

        </CustomConfiguration>
        <PluginPackageFamilyName>Microsoft.AzureVpn_8wekyb3d8bbwe</PluginPackageFamilyName>
      </PluginProfile>
    </VPNProfile>
   ```
1. Modifique la entrada entre ```<ServerUrlList>``` y ```</ServerUrlList>``` con la entrada del perfil descargado (azurevpnconfig.xml). Cambie el FQDN "TrustedNetworkDetection" para que se ajuste a su entorno.
1. Abra el perfil descargado de Azure (azurevpnconfig.xml) y copie todo el contenido en el Portapapeles; para ello, resalte el texto y presione (Ctrl) + C. 
1. Pegue el texto copiado del paso anterior en el archivo que creó en el paso 2 entre las etiquetas ```<CustomConfiguration>  </CustomConfiguration>```. Guarde el archivo con la extensión xml.
1. Anote el valor de las etiquetas ```<name>  </name>```. Se trata del nombre del perfil. Necesitará este nombre al crear el perfil en Intune. Cierre el archivo y recuerde la ubicación en la que se guarda.

## <a name="create-intune-profile"></a>Creación del perfil de Intune

En esta sección, creará un perfil de Microsoft Intune con una configuración personalizada.

1. Inicie sesión en Intune y navegue a **Dispositivos-> Perfiles de configuración**. Seleccione **+ Crear perfil**.

   :::image type="content" source="./media/vpn-gateway-virtual-wan-vpn-profile-intune/configuration-profile.png" alt-text="Perfiles de configuración":::
1. En **Plataforma**, seleccione **Windows 10 y versiones posteriores**. Como **Perfil**, seleccione **Personalizado**. Seleccione **Crear**.
1. Asigne un nombre y una descripción al perfil y, luego, seleccione **Siguiente**.
1. En la pestaña **Opciones de configuración**, seleccione **Agregar**.

    * **Nombre:** escriba un nombre para la configuración.
    * **Descripción:** descripción opcional.
    * **OMA-URI:** ```./User/Vendor/MSFT/VPNv2/<name of your connection>/ProfileXML``` (esta información se puede encontrar en el archivo azurevpnconfig.xml de la etiqueta <name></name>).
    * **Tipo de datos:** string (archivo XML).

   Seleccione el icono de carpeta y elija el archivo que guardó en el paso 6 de los pasos de [XML](#xml). Seleccione **Agregar**.

   :::image type="content" source="./media/vpn-gateway-virtual-wan-vpn-profile-intune/configuration-settings.png" alt-text="Parámetros de configuración" lightbox="./media/vpn-gateway-virtual-wan-vpn-profile-intune/configuration-settings.png":::
1. Seleccione **Next** (Siguiente).
1. En **Asignaciones**, seleccione el grupo en el que quiere insertar la configuración. Después, seleccione **Siguiente**.
1. Las reglas de aplicabilidad son opcionales. Defina las reglas si es necesario y luego seleccione **Siguiente**.
1. En la página **Revisar y crear**, seleccione **Crear**.

    :::image type="content" source="./media/vpn-gateway-virtual-wan-vpn-profile-intune/create-profile.png" alt-text="Creación de perfil":::
1. El perfil personalizado ya se ha creado. Para conocer los pasos de Microsoft Intune para implementar este perfil, consulte [Asignación de perfiles de usuario y de dispositivo](/mem/intune/configuration/device-profile-assign).