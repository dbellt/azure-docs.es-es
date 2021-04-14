---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/27/2020
ms.author: alkohli
ms.openlocfilehash: fa65a7354112a2b220686372459b348d45832dd9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96465820"
---
Realice los pasos siguientes en la interfaz de usuario web local del dispositivo. Este paso tarda unos 15 minutos, incluida la carga del archivo de configuración de VPN (o el archivo de etiqueta de servicio). 

1. Diríjase a **Configuración > VPN**. Seleccione **Configurar**.

    ![Configuración de la interfaz de usuario local 1](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-1.png)

2. En la hoja **Configurar VPN**:

    - Habilite **Configuración de VPN**.
    - Especifique el **VPN shared secret** (Secreto compartido de VPN). Esta es la clave compartida que proporcionó al crear el recurso de conexión de la VPN de Azure.
    - Proporcione la dirección para **Dirección IP de la puerta de enlace de VPN**. Esta es la dirección IP de la puerta de enlace de red local de Azure.
    - En **Grupo PFS**, seleccione **Ninguno**. 
    - En **Grupo DH**, seleccione **Group2**.
    - Para **IPsec integrity method** (Método de integridad de IPsec), seleccione **SHA256**.
    - Para **IPsec cipher transform constants** (Constantes de transformación de cifrado de IPsec), seleccione **GCMAES256**.
    - Para **IPsec authentication transform constants** (Constantes de transformación de autenticación de IPsec), seleccione **GCMAES256**.
    - Para **IKE encryption method** (Método de cifrado de IKE), seleccione **AES256**.
    - Seleccione **Aplicar**.

        ![Configuración de la interfaz de usuario local 2](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-2.png)

    Para obtener más información sobre los algoritmos criptográficos admitidos, diríjase a [Acerca de los requisitos criptográficos y las instancias de Azure VPN Gateway](../articles/vpn-gateway/vpn-gateway-about-compliance-crypto.md#ipsecike-policy-faq). 

3. Para cargar el archivo de configuración de la ruta de VPN, seleccione **Cargar**. 

    ![Configuración de la interfaz de usuario local 3](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-3.png)

    - Vaya al archivo *json* de etiquetas de servicio que descargó en el sistema local en el paso anterior.
    - Seleccione la región como la región de Azure asociada con el dispositivo, la red virtual y las puertas de enlace.
    - Seleccione **Aplicar**.

        ![Configuración de la interfaz de usuario local 4](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-4.png)
    
    La carga tarda aproximadamente entre 7 y 8 minutos en el dispositivo.

4. Para agregar rutas específicas del cliente, configure los intervalos de direcciones IP a los que se va a tener acceso solo mediante VPN. 

    - En **IP address ranges to be accessed using VPN only** (Intervalos de direcciones IP a los que acceder solo mediante VPN), seleccione **Configurar**.
    - Especifique un intervalo IPv4 válido y seleccione **Agregar**. Repita los pasos para agregar otros intervalos.
    - Seleccione **Aplicar**.

        ![Configuración de la interfaz de usuario local 5](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-5.png)

