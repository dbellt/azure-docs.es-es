---
title: Uso de perfiles de Wi-Fi con dispositivos de Azure Stack Edge Mini R
description: Aquí se describe cómo crear perfiles de Wi-Fi para dispositivos de Azure Stack Edge Mini R en redes de empresa y redes personales de alta seguridad.
services: databox
author: v-dalc@microsoft.com
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 03/24/2021
ms.author: alkohli
ms.openlocfilehash: 90c7c238cef104eae78618e51fa4b284adcc8f42
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105050332"
---
# <a name="use-wi-fi-profiles-with-azure-stack-edge-mini-r-devices"></a>Uso de perfiles de Wi-Fi con dispositivos de Azure Stack Edge Mini R

En este artículo se describe cómo usar perfiles de redes inalámbricas (Wi-Fi) con los dispositivos de Azure Stack Edge Mini R.

Esta es la forma de preparar el perfil de Wi-Fi dependiendo del tipo de red inalámbrica:

- En una red personal de acceso protegido Wi-Fi 2 (WPA2), como una red doméstica o una zona Wi-Fi activa, puede descargar y usar un perfil inalámbrico existente con la misma contraseña que usa con otros dispositivos.

- En un entorno empresarial de alta seguridad, tendrá acceso a su dispositivo a través de una red WPA2 de empresa. En este tipo de red, cada equipo cliente tendrá un perfil de Wi-Fi distinto y se autenticará mediante certificados. Deberá ponerse en contacto con el administrador de la red para determinar la configuración necesaria.

Más adelante trataremos los requisitos de perfil para ambos tipos de red.

En cualquier caso, es muy importante asegurarse de que el perfil cumple los requisitos de seguridad de su organización antes de probar o usar el perfil con el dispositivo.

## <a name="about-wi-fi-profiles"></a>Acerca de los perfiles de Wi-Fi

Un perfil de Wi-Fi contiene el SSID (identificador del conjunto de servicios o el **nombre de red**), la clave de contraseña y la información de seguridad necesaria para conectar el dispositivo de Azure Stack Edge Mini R a una red inalámbrica.

En el ejemplo de código siguiente se muestra la configuración básica de un perfil para usarlo con una red inalámbrica típica:

* `SSID` es el nombre de red.

* `name` es el nombre descriptivo de la conexión Wi-Fi. Este es el nombre que los usuarios verán cuando exploren la lista de conexiones disponibles en sus dispositivos.

* El perfil está configurado para conectar automáticamente el equipo a la red inalámbrica cuando está dentro del alcance de la red (`connectionMode` = `auto`).

```html
<?xml version="1.0"?>
<WLANProfile xmlns="http://www.contoso.com/networking/WLAN/profile/v1">
    <name>ContosoWIFICORP</name>
    <SSIDConfig>
        <SSID>
            <hex>1A234561234B5012</hex>
        </SSID>
        <nonBroadcast>false</nonBroadcast>
    </SSIDConfig>
    <connectionType>ESS</connectionType>
    <connectionMode>auto</connectionMode>
    <autoSwitch>false</autoSwitch>
```

Para obtener más información acerca de la configuración del perfil de Wi-Fi, consulte **Perfil de empresa** en [Adición de configuraciones de Wi-Fi para Windows 10 y dispositivos más recientes](/mem/intune/configuration/wi-fi-settings-windows#enterprise-profile), y consulte [Configuración del perfil de Wi-Fi de Cisco](azure-stack-edge-mini-r-manage-wifi.md#configure-cisco-wi-fi-profile).

Para habilitar las conexiones inalámbricas en un dispositivo de Azure Stack Edge Mini R, configure el puerto Wi-Fi en el dispositivo y, a continuación, agregue los perfiles de Wi-Fi al dispositivo. En una red de empresa, también debe cargar los certificados en el dispositivo. A continuación, puede conectarse a una red inalámbrica desde la interfaz de usuario web local del dispositivo. Para obtener más información, consulte [Administración de la conectividad inalámbrica en su instancia de Azure Stack Edge Mini R](./azure-stack-edge-mini-r-manage-wifi.md).

## <a name="profile-for-wpa2---personal-network"></a>Perfil de red WPA2 personal

En una red personal de acceso protegido Wi-Fi 2 (WPA2), como una red doméstica o una zona Wi-Fi activa pública, varios dispositivos pueden usar el mismo perfil y la misma contraseña. En la red doméstica, el teléfono móvil y el portátil usan el mismo perfil inalámbrico y contraseña para conectarse a la red.

Por ejemplo, un cliente de Windows 10 puede generar un perfil en tiempo de ejecución. Cuando inicie sesión en la red inalámbrica, se le pedirá la contraseña Wi-Fi y, una vez que la proporcione, estará conectado. No se necesita ningún certificado en este entorno.

En este tipo de red, es posible que pueda exportar un perfil de Wi-Fi desde el equipo portátil y, a continuación, agregarlo al dispositivo de Azure Stack Edge Mini R. Para obtener instrucciones, consulte a continuación [Exportación de un perfil de Wi-Fi](#export-a-wi-fi-profile).

> [!IMPORTANT]
> Antes de crear un perfil de Wi-Fi para el dispositivo de Azure Stack Edge Mini R, póngase en contacto con el administrador de red para averiguar los requisitos de seguridad de la organización para redes inalámbricas. No debe probar ni usar ningún perfil de Wi-Fi en el dispositivo hasta que sepa que la red inalámbrica cumple tales requisitos.

## <a name="profiles-for-wpa2---enterprise-network"></a>Perfiles para red WPA2 de empresa

En una red de empresa de acceso protegido inalámbrico 2 (WPA2), deberá contactar con el administrador de la red para obtener el perfil de Wi-Fi necesario y el certificado para conectar el dispositivo de Azure Stack Edge Mini R a la red.

En el caso de las redes de alta seguridad, el dispositivo de Azure puede usar un Protocolo de autenticación extensible protegido (PEAP) con el Protocolo de autenticación extensible y Seguridad de la capa de transporte (EAP-TLS). Como los protocolos PEAP con EAP-TLS usan la autenticación de la máquina, el cliente y el servidor usan certificados para comprobar sus identidades entre sí.

> [!NOTE]
> * La autenticación de un usuario que usa la versión 2 del Protocolo de autenticación por desafío mutuo de Microsoft PEAP (PEAP MSCHAPv2) no se admite en dispositivos de Azure Stack Edge Mini R.
> * La autenticación EAP-TLS es necesaria para obtener acceso a la funcionalidad de Azure Stack Edge Mini R. Una conexión inalámbrica que configure con Active Directory no funcionará.

El administrador de red generará un perfil de Wi-Fi único y un certificado de cliente para cada equipo. El administrador de red decide si se debe usar un certificado independiente o un certificado compartido para cada dispositivo.

Si trabaja en más de una ubicación física en el área de trabajo, es posible que el administrador de red tenga que proporcionarle más de un perfil de Wi-Fi específico del sitio para las conexiones inalámbricas.

En una red empresarial, se recomienda no cambiar la configuración de los perfiles de Wi-Fi que proporciona el administrador de red. El único ajuste que puede querer hacer es la configuración de la conexión automática. Para obtener más información, consulte [Perfil básico](/mem/intune/configuration/wi-fi-settings-windows#basic-profile) en la configuración de Wi-Fi de Windows 10 y en los dispositivos más recientes.

En un entorno empresarial de alta seguridad, es posible que pueda usar un perfil de red inalámbrica existente como plantilla:

* Puede descargar el perfil de red inalámbrica corporativa desde el equipo de trabajo. Para obtener instrucciones, consulte a continuación [Exportación de un perfil de Wi-Fi](#export-a-wi-fi-profile).

* Si otras personas de su organización ya están conectadas a sus dispositivos de Azure Stack Edge Mini R a través de una red inalámbrica, pueden descargar el perfil de Wi-Fi desde su dispositivo. Para obtener instrucciones, consulte [Descarga del perfil de Wi-Fi](azure-stack-edge-mini-r-manage-wifi.md#download-wi-fi-profile).

## <a name="export-a-wi-fi-profile"></a>Exportar un perfil de Wi-Fi

Para exportar un perfil para la interfaz de Wi-Fi en el equipo, siga estos pasos:

1. Para ver los perfiles inalámbricos en el equipo, vaya al menú **Inicio**, abra el **símbolo del sistema** (cmd.exe) y escriba este comando:

   `netsh wlan show profiles`

   El resultado tendrá un aspecto similar al siguiente:

   ```dos
   Profiles on interface Wi-Fi:

   Group policy profiles (read only)
   ---------------------------------
       <None>

   User profiles
   -------------
       All User Profile     : ContosoCORP
       All User Profile     : ContosoFTINET
       All User Profile     : GusIS2809
       All User Profile     : GusGuests
       All User Profile     : SeaTacGUEST
       All User Profile     : Boat
   ```

2. Para exportar un perfil, escriba el siguiente comando:

   `netsh wlan export profile name=”<profileName>” folder=”<path>\<profileName>"`

   Por ejemplo, el siguiente comando guarda el perfil ContosoFTINET en formato XML en la carpeta Descargas para el usuario llamado `gusp`.

   ```dos
   C:\Users\gusp>netsh wlan export profile name="ContosoFTINET" folder=c:Downloads

   Interface profile "ContosoFTINET" is saved in file "c:Downloads\ContosoFTINET.xml" successfully.
   ```

## <a name="add-certificate-wi-fi-profile-to-device"></a>Adición del certificado y el perfil Wi-Fi al dispositivo

Cuando tenga los perfiles de Wi-Fi y los certificados que necesite, siga estos pasos para configurar el dispositivo de Azure Stack Edge Mini R para las conexiones inalámbricas:

1. En el caso de una red WPA2 de empresa, cargue los certificados necesarios en el dispositivo siguiendo las instrucciones de la [Carga de certificados](./azure-stack-edge-gpu-manage-certificates.md#upload-certificates).

1. Cargue los perfiles de Wi-Fi en el dispositivo Mini R y, a continuación, conéctese a él siguiendo las instrucciones que se indican en [Adición y conexión del perfil Wi-Fi](./azure-stack-edge-mini-r-manage-wifi.md#add-connect-to-wi-fi-profile).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [Cómo configurar una red para Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy.md).
- Obtenga información sobre [Cómo administrar una red Wi-Fi para Azure Stack Edge Mini R](azure-stack-edge-mini-r-manage-wifi.md).
