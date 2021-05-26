---
title: 'Tutorial: Configuración de los valores de red de un dispositivo Azure Stack Edge Pro R en Azure Portal | Microsoft Docs'
description: El tutorial para implementar Azure Stack Edge Pro R proporciona instrucciones para configurar la red, la red de proceso y los valores del servidor proxy web para el dispositivo físico.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 05/11/2021
ms.author: alkohli
ms.openlocfilehash: 609e12e2c39dcbc6b0213f7619541a83dc6ffde8
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109839561"
---
# <a name="tutorial-configure-network-for-azure-stack-edge-pro-r"></a>Tutorial: Configuración de la red para Azure Stack Edge Pro R

En este tutorial se describe cómo configurar la red para el dispositivo Azure Stack Edge Pro R mediante la interfaz de usuario web local.

El proceso de conexión puede tardar unos 20 minutos en completarse.

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos
> * Configuración de la red
> * Habilitación de la red de proceso
> * Configurar el proxy web


## <a name="prerequisites"></a>Requisitos previos

Antes de configurar e instalar el dispositivo Azure Stack Edge Pro R, asegúrese de lo siguiente:

* Ha instalado el dispositivo físico tal y como se describe en [Instalación de Azure Stack Edge Pro R](azure-stack-edge-gpu-deploy-install.md).
* Se ha conectado a la interfaz de usuario web local del dispositivo, tal como se detalla en [Conexión a Azure Stack Edge Pro R](azure-stack-edge-gpu-deploy-connect.md).


## <a name="configure-network"></a>Configuración de la red

En la página **Introducción** se muestran los distintos valores necesarios para configurar y registrar el dispositivo físico en el servicio Azure Stack Edge. 

Siga estos pasos para configurar la red en el dispositivo.

1. En la interfaz de usuario web local del dispositivo, vaya a la página **Introducción**. 

2. En el icono **Network** (Red), seleccione **Configure** (Configurar) para ir a la página **Network** (Red). 
    
    <!--![Local web UI "Network settings" tile](./media/azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy/network-1.png)-->

    En el dispositivo físico hay cuatro interfaces de red. PUERTO 1 y PUERTO 2 son interfaces de red de 1 Gbps. El PUERTO 3 y el PUERTO 4 son interfaces de red de 10/25 Gbps. El PUERTO 1 se configura automáticamente como puerto solo de administración y del PUERTO 2 al PUERTO 4 son los puertos de datos. La página **Network** (Red) es tal como se muestra a continuación.
    
    ![Página "Configuración de red" de la interfaz de usuario web local](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/network-2.png)

   
3. Para cambiar la configuración de red, seleccione un puerto y, en el panel derecho que aparece, modifique la dirección IP, la subred, la puerta de enlace, el DNS principal y el DNS secundario. 

    - Si selecciona el puerto 1, puede ver que está preconfigurado como estático. 

        !["Configuración del puerto 1 de la red" de la interfaz de usuario web local](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/network-3.png)

    - Si selecciona el puerto 2, el puerto 3 o el puerto 4, se configuran todos como DHCP de forma predeterminada.

        !["Configuración del puerto 3 de la red" de la interfaz de usuario web local](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/network-4.png)

    Al configurar la red, tenga en cuenta lo siguiente:

   * Si DHCP está habilitado en el entorno, las interfaces de red se configuran automáticamente. Se asignan automáticamente una dirección IP, la subred, la puerta de enlace y DNS.
   * Si DHCP no está habilitado, puede asignar direcciones IP estáticas si es necesario.
   * Puede configurar la interfaz de red como IPv4.
   * Azure Stack Edge no admite la agregación de vínculos ni la formación de equipos en las tarjetas de interfaz de red (NIC).
   * El número de serie de cualquier puerto corresponde al número de serie del nodo.
    <!--* On the 25-Gbps interfaces, you can set the RDMA (Remote Direct Access Memory) mode to iWarp or RoCE (RDMA over Converged Ethernet). Where low latencies are the primary requirement and scalability is not a concern, use RoCE. When latency is a key requirement, but ease-of-use and scalability are also high priorities, iWARP is the best candidate.-->
    Una vez configurada la red del dispositivo, la página se actualiza tal como se muestra a continuación.

    ![Página 2 "Configuración de red" de la interfaz de usuario web local](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/network-2a.png)<!--change-->


     >[!NOTE]
     >
     > * Se recomienda no cambiar la dirección IP local de la interfaz de red de estática a DHCP, a menos que se tenga otra dirección IP para conectarse al dispositivo. Si se usara una interfaz de red y cambiara a DHCP, no habría forma de determinar la dirección DHCP. Si desea cambiar a una dirección DHCP, espere a que el dispositivo se haya activado con el servicio y, después, realice el cambio. Después, puede ver las direcciones IP de todos los adaptadores de las **propiedades del dispositivo** de su servicio en Azure Portal.


    Después de configurar y aplicar los valores de red, seleccione **Next: Compute** (Siguiente: Proceso) para configurar la red de proceso.

## <a name="enable-compute-network"></a>Habilitación de la red de proceso

Siga estos pasos para habilitar el proceso y configurar la red de proceso. 

1. En la página **Proceso**, seleccione la interfaz de red que desea habilitar para el proceso. 

    ![Página Proceso en la interfaz de usuario local 2](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/compute-network-2.png)

1. En el cuadro de diálogo **Configuración de red**, seleccione **Habilitar**. Cuando se habilita el proceso, se crea un conmutador virtual en el dispositivo en esa interfaz de red. El conmutador virtual se usa para la infraestructura de proceso en el dispositivo. 
    
1. Asigne las **IP de nodo de Kubernetes**. Estas direcciones IP estáticas son para la máquina virtual de proceso.  

    En el caso de un dispositivo con *n* nodos, se proporciona un rango contiguo de un mínimo de *n + 1* direcciones IPv4 (o más) para la máquina virtual de proceso mediante las direcciones IP inicial y final. Dado que Azure Stack Edge es un dispositivo de 1 nodo, se proporciona un mínimo de 2 direcciones IPv4 contiguas. Estas direcciones IP deben estar en la misma red en la que habilitó el proceso y se creó el conmutador virtual.

    > [!IMPORTANT]
    > Kubernetes en Azure Stack Edge utiliza la subred 172.27.0.0/16 para los pods y la subred 172.28.0.0/16 para el servicio. Asegúrese de que no están en uso en la red. Si estas subredes ya están en uso en la red, ejecute el cmdlet `Set-HcsKubeClusterNetworkInfo` desde la interfaz de PowerShell del dispositivo para cambiar estas subredes. Para más información, consulte [Cambio de las subredes de pods y de servicio de Kubernetes](azure-stack-edge-gpu-connect-powershell-interface.md#change-kubernetes-pod-and-service-subnets).


1. Asigne las **IP del servicio externo de Kubernetes**. También son las direcciones IP de equilibrio de carga. Estas direcciones IP contiguas son para los servicios que desea exponer fuera del clúster de Kubernetes y debe especificar el rango de direcciones IP estáticas en función del número de servicios expuestos. 
    
    > [!IMPORTANT]
    > Le recomendamos que especifique un mínimo de una dirección IP para que el servicio del centro de Azure Stack Edge Pro R acceda a los módulos de proceso. Opcionalmente, puede especificar direcciones IP adicionales para otros módulos de servicios o IoT Edge (1 por servicio o módulo) a los que es necesario acceder desde fuera del clúster. Las direcciones IP de servicio se pueden actualizar más adelante. 
    
1. Seleccione **Aplicar**.

    ![Página Proceso en la interfaz de usuario local 3](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/compute-network-3.png)

1. La configuración tarda un par de minutos en aplicarse y puede que tenga que actualizar el explorador. Puede ver que el puerto especificado está habilitado para el proceso. 
 
    ![Página Proceso en la interfaz de usuario local 4](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/compute-network-4.png)

    Seleccione **Siguiente: Proxy web** para configurar el proxy web.  

  
## <a name="configure-web-proxy"></a>Configurar el proxy web

Es una configuración opcional.

> [!IMPORTANT]
> No se admiten los archivos de configuración automática del proxy (PAC). Los archivos de PAC definen el número de exploradores web y otros agentes de usuario que pueden elegir automáticamente el servidor proxy (método de acceso) adecuado para obtener una dirección URL determinada. Los proxies que intentan interceptar y leer todo el tráfico (y luego vuelven a firmar todo con su propia certificación) no son compatibles, ya que el certificado del proxy no es de confianza. Normalmente, los servidores proxy transparentes funcionan bien con Azure Stack Edge Pro R. No se admiten los servidores proxy web no transparentes.


1. En la página **Configuración de proxy web**, realice los siguientes pasos:

   1. En el cuadro **URL de proxy web** , escriba la dirección URL en este formato: `http://host-IP address or FQDN:Port number`. No se admiten direcciones URL HTTPS.

   2. Para validar y aplicar la configuración del proxy web configurado, seleccione **Aplicar**.
    
   ![Página 2 "Configuración de proxy web" de la interfaz de usuario web local](./media/azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy/web-proxy-2.png)<!--UI text update for instruction text is needed.-->

2. Una vez aplicada la configuración, seleccione **Siguiente: Device** (IoT Workbench: dispositivo).


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió sobre lo siguiente:

> [!div class="checklist"]
> * Requisitos previos
> * Configuración de la red
> * Habilitación de la red de proceso
> * Configurar el proxy web


Para aprender a configurar el dispositivo Azure Stack Edge Pro R, consulte:

> [!div class="nextstepaction"]
> [Configuración de las opciones de dispositivo](./azure-stack-edge-pro-r-deploy-set-up-device-update-time.md)
