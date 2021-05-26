---
title: Tutorial para configurar los valores de red de un dispositivo Azure Stack Edge Mini R en Azure Portal
description: El tutorial para implementar Azure Stack Edge Mini R proporciona instrucciones para configurar la red, la red de proceso y los valores del servidor proxy web del dispositivo físico.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: alkohli
ms.openlocfilehash: ae3504b1483dce5678a9d7b53556d9228e53ded0
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109839144"
---
# <a name="tutorial-configure-network-for-azure-stack-edge-mini-r"></a>Tutorial: Configuración de red para Azure Stack Edge Mini R

En este tutorial se describe cómo configurar la red para el dispositivo Azure Stack Edge Mini R con una GPU incorporada mediante la interfaz de usuario web local.

El proceso de conexión puede tardar unos 20 minutos en completarse.

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * Configuración de la red
> * Habilitación de la red de proceso
> * Configurar el proxy web


## <a name="prerequisites"></a>Requisitos previos

Antes de configurar e instalar el dispositivo Azure Stack Edge Mini R, asegúrese de lo siguiente:

* Ha instalado el dispositivo físico tal y como se describe en [Instalación de Azure Stack Edge Mini R](azure-stack-edge-gpu-deploy-install.md).
* Se ha conectado a la interfaz de usuario web local del dispositivo, tal como se detalla en [Conexión a Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-connect.md).


## <a name="configure-network"></a>Configuración de la red

En la página **Introducción** se muestran los distintos valores necesarios para configurar y registrar el dispositivo físico en el servicio Azure Stack Edge. 

Siga estos pasos para configurar la red en el dispositivo.

1. En la interfaz de usuario web local del dispositivo, vaya a la página **Introducción**. 

2. Si se necesita una actualización de día cero, puede hacerlo aquí mediante la configuración de un puerto de datos con una conexión con cable. Para más instrucciones sobre cómo configurar una conexión con cable para este dispositivo, consulte [Cableado del dispositivo](azure-stack-edge-mini-r-deploy-install.md#cable-the-device). Una vez finalizada la actualización, puede quitar la conexión con cable.

3. Cree certificados para la red Wi-Fi y la cadena de firma. Tanto la cadena de firma como el certificado para la red Wi-Fi deben tener el formato DER con una extensión de archivo *.cer*. Para obtener instrucciones, consulte [Uso de certificados con el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-manage-certificates.md).

4. En la interfaz de usuario web local, vaya a **Get started** (Comenzar). En el icono **Security** (Seguridad), seleccione **Certificates** (Certificados) y, a continuación, seleccione **Configure** (Configurar). 

   [![Página "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/get-started-1.png)](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/get-started-1.png#lightbox)

   1. Select **+ Add certificate** (+ Agregar certificado). 
    
      [![Página "Certificates" (Certificados) de la interfaz de usuario web local 1](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-cert-1.png)](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-cert-1.png#lightbox)

   2. Cargue la cadena de firma y seleccione **Apply** (Aplicar).

      ![Página 2 de "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-cert-2.png)

   3. Repita el procedimiento con el certificado para la red Wi-Fi. 

      ![Página 3 de "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-cert-4.png)

   4. Los nuevos certificados se deberían mostrar en la página **Certificates** (Certificados). 

      [![Página 4 de "Certificados" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-cert-5.png)](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-cert-5.png#lightbox)

   5. Vuelva a **Get started** (Comenzar).

5. En el icono **Red**, seleccione **Configurar**.  

   En el dispositivo físico hay cinco interfaces de red. PUERTO 1 y PUERTO 2 son interfaces de red de 1 Gbps. El PUERTO 3 y el PUERTO 4 son interfaces de red de 10 Gbps. El quinto puerto es el puerto Wi-Fi. 

   [![Página "Network settings" (Configuración de red) de la interfaz de usuario web local 1](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-1.png)](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-1.png#lightbox)    

   Seleccione el puerto Wi-Fi y establezca la configuración del puerto. 

   > [!IMPORTANT]
   > Se recomienda que configure una dirección IP estática para el puerto Wi-Fi.  


   ![Página 2 "Configuración de red" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-2.png)

   La página **Network** (Red)se actualiza después de aplicar la configuración del puerto Wi-Fi.

   ![Página "Network settings" (Configuración de red) de la interfaz de usuario web local 3](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/configure-wifi-4.png)

6. Seleccione **Add Wi-Fi profile** (Agregar perfil de Wi-Fi) y cargue el perfil de Wi-Fi. 

   !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 1](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-1.png)

   Un perfil de red inalámbrica contiene el SSID (nombre de red), la clave de contraseña y la información de seguridad para poder conectarse a una red inalámbrica. Puede obtener el perfil de Wi-Fi de su entorno del administrador de red.

   !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 2](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-2.png)

   Una vez agregado el perfil, se actualiza la lista de perfiles de Wi-Fi para reflejar el nuevo perfil. El perfil debe mostrar el campo **Connection status** (Estado de la conexión) como **Disconnected** (Desconectado). 

   !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 3](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-3.png)

7. Una vez que el perfil de red inalámbrica se haya cargado correctamente, conéctese a este perfil. Seleccione **Connect to Wi-Fi profile** (Conectarse a perfil de Wi-Fi). 

   !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 4](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-4.png)

8. Seleccione el perfil de Wi-Fi que agregó en el paso anterior y haga clic en **Aplicar**. 

   !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 5](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-5.png)

   El campo **Connection status** (Estado de la conexión) se debe actualizar a **Conectado** (Conectado). La intensidad de la señal se actualiza para indicar la calidad de la señal. 

   !["Port WiFi Network settings" (Configuración de red del puerto Wi-Fi) de la interfaz de usuario web local 6](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/add-wifi-profile-6.png)

   > [!NOTE]
   > Para transferir grandes cantidades de datos, recomendamos que use una conexión con cable en lugar de la red inalámbrica. 

9. Desconecte el puerto 1 del dispositivo del equipo portátil. 

10. Al configurar la red, tenga en cuenta lo siguiente:

    - Si DHCP está habilitado en el entorno, las interfaces de red se configuran automáticamente. Se asignan automáticamente una dirección IP, la subred, la puerta de enlace y DNS.
    - Si DHCP no está habilitado, puede asignar direcciones IP estáticas si es necesario.
    - Puede configurar la interfaz de red como IPv4.
    - Azure Stack Edge no admite la agregación de vínculos ni la formación de equipos en las tarjetas de interfaz de red (NIC).
    - El número de serie de cualquier puerto corresponde al número de serie del nodo. Para un dispositivo de la serie K, solo se muestra un número de serie.

     > [!NOTE]
     > Se recomienda no cambiar la dirección IP local de la interfaz de red de estática a DHCP, a menos que se tenga otra dirección IP para conectarse al dispositivo. Si se usara una interfaz de red y cambiara a DHCP, no habría forma de determinar la dirección DHCP. Si desea cambiar a una dirección DHCP, espere a que el dispositivo se haya registrado con el servicio y, después, realice el cambio. Después, puede ver las direcciones IP de todos los adaptadores de las **propiedades del dispositivo** de su servicio en Azure Portal.

Después de configurar y aplicar los valores de red, seleccione **Next: Compute** (Siguiente: Proceso) para configurar la red de proceso.

## <a name="enable-compute-network"></a>Habilitación de la red de proceso

Siga estos pasos para habilitar el proceso y configurar la red de proceso. 


1. En la página **Proceso**, seleccione la interfaz de red que desea habilitar para el proceso. 

   ![Página Proceso en la interfaz de usuario local 2](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/compute-network-1.png)

1. En el cuadro de diálogo **Configuración de red**, seleccione **Habilitar**. Cuando se habilita el proceso, se crea un conmutador virtual en el dispositivo en esa interfaz de red. El conmutador virtual se usa para la infraestructura de proceso en el dispositivo. 
    
1. Asigne las **IP de nodo de Kubernetes**. Estas direcciones IP estáticas son para la máquina virtual de proceso.  

   En el caso de un dispositivo con *n* nodos, se proporciona un rango contiguo de un mínimo de *n + 1* direcciones IPv4 (o más) para la máquina virtual de proceso mediante las direcciones IP inicial y final. Dado que Azure Stack Edge es un dispositivo de 1 nodo, se proporciona un mínimo de 2 direcciones IPv4 contiguas.

   > [!IMPORTANT]
   > Kubernetes en Azure Stack Edge utiliza la subred 172.27.0.0/16 para los pods y la subred 172.28.0.0/16 para el servicio. Asegúrese de que no están en uso en la red. Si estas subredes ya están en uso en la red, ejecute el cmdlet `Set-HcsKubeClusterNetworkInfo` desde la interfaz de PowerShell del dispositivo para cambiar estas subredes. Para más información, consulte [Cambio de las subredes de pods y de servicio de Kubernetes](azure-stack-edge-gpu-connect-powershell-interface.md#change-kubernetes-pod-and-service-subnets).

1. Asigne las **IP del servicio externo de Kubernetes**. También hay direcciones IP de equilibrio de carga. Estas direcciones IP contiguas son para los servicios que desea exponer fuera del clúster de Kubernetes, y debe especificar el rango de direcciones IP estáticas en función del número de servicios expuestos. 
    
   > [!IMPORTANT]
   > Le recomendamos que especifique un mínimo de una dirección IP para que el servicio del centro de Azure Stack Edge Mini R acceda a los módulos de proceso. Opcionalmente, puede especificar direcciones IP adicionales para otros módulos de servicios o IoT Edge (1 por servicio o módulo) a los que es necesario acceder desde fuera del clúster. Las direcciones IP de servicio se pueden actualizar más adelante. 
    
1. Seleccione **Aplicar**.

   ![Página Proceso en la interfaz de usuario local 3](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/compute-network-3.png)

1. La configuración tarda algunos minutos en aplicarse, y es posible que tenga que actualizar el explorador. Puede ver que el puerto especificado está habilitado para el proceso. 
 
   ![Página Proceso en la interfaz de usuario local 4](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/compute-network-4.png)

   Seleccione **Siguiente: Proxy web** para configurar el proxy web.  

  
## <a name="configure-web-proxy"></a>Configurar el proxy web

Es una configuración opcional.

> [!IMPORTANT]
> No se admiten los archivos de configuración automática del proxy (PAC). Los archivos de PAC definen el número de exploradores web y otros agentes de usuario que pueden elegir automáticamente el servidor proxy (método de acceso) adecuado para obtener una dirección URL determinada. Los proxies que intentan interceptar y leer todo el tráfico (y luego vuelven a firmar todo con su propia certificación) no son compatibles, ya que el certificado del proxy no es de confianza. Normalmente, los servidores proxy transparentes funcionan bien con Azure Stack Edge Mini R. No se admiten los servidores proxy web no transparentes.


1. En la página **Configuración de proxy web**, realice los siguientes pasos:

   1. En el cuadro **URL de proxy web** , escriba la dirección URL en este formato: `http://host-IP address or FQDN:Port number`. No se admiten direcciones URL HTTPS.

   2. Para validar y aplicar la configuración del proxy web configurado, seleccione **Aplicar**.

   ![Página "Configuración de proxy web" de la interfaz de usuario web local](./media/azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy/web-proxy-1.png)<!--UI text update is needed to remove NTLM from instruction text.-->

2. Una vez aplicada la configuración, seleccione **Siguiente: Device** (IoT Workbench: dispositivo).


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos previos
> * Configuración de la red
> * Habilitación de la red de proceso
> * Configurar el proxy web


Para aprender a configurar el dispositivo Azure Stack Edge Mini R, consulte:

> [!div class="nextstepaction"]
> [Configuración de las opciones de dispositivo](./azure-stack-edge-mini-r-deploy-set-up-device-update-time.md)
