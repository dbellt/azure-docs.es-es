---
title: Protección de aplicaciones web en Azure VMware Solution con Azure Application Gateway
description: Configure Azure Application Gateway para exponer de forma segura las aplicaciones web que se ejecutan en Azure VMware Solution.
ms.topic: how-to
ms.date: 02/10/2021
ms.openlocfilehash: 4d31eb24f1cb4ec1e2ead789174189dc99d5310f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108771692"
---
# <a name="protect-web-apps-on-azure-vmware-solution-with-azure-application-gateway"></a>Protección de aplicaciones web en Azure VMware Solution con Azure Application Gateway

[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) es un equilibrador de carga de tráfico web de capa 7 que permite administrar el tráfico a las aplicaciones web. Se ofrece en las versiones 1.0 y 2.0 de Azure VMware Solution. Ambas versiones probadas con aplicaciones web en ejecución en Azure VMware Solution.

Entre estas funcionalidades, cabe destacar: 
- Afinidad de sesión basada en cookies
- Enrutamiento basado en dirección URL
- Firewall de aplicaciones web (WAF)

Para una lista completa de las características, consulte [Características de Azure Application Gateway](../application-gateway/features.md). 

En este artículo se muestra cómo usar Application Gateway delante de una granja de servidores web para proteger una aplicación web que se ejecuta en Azure VMware Solution. 

## <a name="topology"></a>Topología
El diagrama muestra cómo se usa Application Gateway para proteger máquinas virtuales de IaaS de Azure, conjuntos de escalado de máquinas virtuales de Azure o servidores locales. Application Gateway considera las máquinas virtuales de Azure VMware Solution como servidores locales. 

![Diagrama que muestra cómo Application Gateway protege máquinas virtuales de IaaS de Azure, conjuntos de escalado de máquinas virtuales de Azure o servidores locales.](media/protect-azure-vmware-solution-with-application-gateway/app-gateway-protects.png)

> [!IMPORTANT]
> Azure Application Gateway es actualmente el único método admitido para exponer las aplicaciones web que se ejecutan en máquinas virtuales de Azure VMware Solution.

En el diagrama se muestra el escenario de prueba que se usa para validar Application Gateway con aplicaciones web de Azure VMware Solution.

:::image type="content" source="media/hub-spoke/azure-vmware-solution-second-level-traffic-segmentation.png" alt-text="Diagrama que muestra el escenario de prueba que se usa para validar Application Gateway con aplicaciones web de Azure VMware Solution." border="false":::

La instancia de Application Gateway se implementa en el centro de una subred dedicada. Tiene una IP pública de Azure. Se recomienda activar [Azure DDoS Protection estándar](../ddos-protection/ddos-protection-overview.md) para la red virtual. El servidor web se hospeda en una nube privada de Azure VMware Solution detrás de las puertas de enlace NSX T0 y T1. Azure VMware Solution usa [Global Reach de ExpressRoute](../expressroute/expressroute-global-reach.md) para habilitar la comunicación con el centro de conectividad y los sistemas locales.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa.
- Una nube privada de Azure VMware Solution implementada y en ejecución.

## <a name="deployment-and-configuration"></a>Implementación y configuración

1. En Azure Portal, busque **Application Gateway** y seleccione **Crear puerta de enlace de aplicaciones**.

2. Proporcione los detalles básicos como se indica en la ilustración siguiente. A continuación, seleccione **Siguiente: Front-end >** . 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/create-app-gateway.png" alt-text="Captura de pantalla que muestra la página Creación de una puerta de enlace de aplicación en Azure Portal.":::

3. Seleccione el tipo de dirección IP de front-end. Para que sea pública, seleccione una dirección IP pública existente o cree una nueva. Seleccione **Siguiente: Back-end >** .

    > [!NOTE]
    > Solo se admiten SKU estándar y del firewall de aplicaciones web para los servidores front-end privados.

4. Agregue un grupo de back-end de las máquinas virtuales que se ejecutan en la infraestructura de Azure VMware Solution. Proporcione los detalles de los servidores web que se ejecutan en la nube privada de Azure VMware Solution y seleccione **Agregar**.  Después, seleccione **Next: Configuración >** .

5. En la pestaña **Configuración**, seleccione **Agregar una regla de enrutamiento**.

6. En la pestaña **Cliente de escucha**, proporcione los detalles del cliente de escucha. Si se selecciona HTTPS, debe proporcionar un certificado, ya sea de un archivo PFX o un certificado existente de Azure Key Vault. 

7. Seleccione la pestaña **Destinos de back-end** y seleccione el grupo de back-end que creó anteriormente. En el campo **Configuración HTTP**, seleccione **Agregar nueva**.

8. Configure los parámetros para la configuración HTTP. Seleccione **Agregar**.

9. Si desea configurar reglas basadas en rutas de acceso, seleccione **Agregar varios destinos para crear una regla basada en ruta de acceso**. 

10. Agregue una regla basada en ruta de acceso y seleccione **Agregar**. Repita para agregar más reglas basadas en rutas de acceso. 

11. Cuando haya terminado de agregar reglas basadas en rutas de acceso, seleccione **Agregar** de nuevo. A continuación, seleccione **Siguiente: Etiquetas >** . 

12. Agregue etiquetas y, a continuación, seleccione **Siguiente: Revisar y crear >** .

13. Se ejecutará una validación en la instancia de Application Gateway. Si es correcta, seleccione **Crear** para implementar.

## <a name="configuration-examples"></a>Ejemplos de configuración

Ahora vamos a configurar Application Gateway con máquinas virtuales de Azure VMware Solution como grupos de back-end para estos casos de uso: 

- [Hospedaje de varios sitios](#hosting-multiple-sites)
- [Enrutamiento por dirección URL](#routing-by-url)

### <a name="hosting-multiple-sites"></a>Hospedaje de varios sitios

En este procedimiento se muestra cómo definir grupos de direcciones de back-end mediante máquinas virtuales que se ejecutan en una nube privada de Azure VMware Solution en una puerta de enlace de aplicación existente. 

>[!NOTE]
>En este procedimiento se presupone que tiene varios dominios, por lo que usaremos ejemplos de www.contoso.com y www.fabrikam.com.


1. En la nube privada, cree dos grupos distintos de máquinas virtuales. Uno representa a Contoso y el segundo, a Fabrikam. 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool.png" alt-text="Captura de pantalla que muestra un resumen de los detalles de un servidor web en el cliente de vSphere.":::

    Hemos usado Windows Server 2016 con el rol Internet Information Services (IIS) instalado. Una vez instaladas las máquinas virtuales, ejecute los comandos de PowerShell siguientes para configurar IIS en cada una de ellas. 

    ```powershell
    Install-WindowsFeature -Name Web-Server
    Add-Content -Path C:\inetpub\wwwroot\Default.htm -Value $($env:computername)
    ```

2. En una instancia de Application Gateway existente, seleccione **Grupos de back-end** en el menú de la izquierda, seleccione **Agregar** y escriba los detalles de los grupos nuevos. Seleccione **Agregar** en el panel derecho.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-02.png" alt-text="Captura de pantalla de la página Grupos de back-end para agregar grupos de back-end." lightbox="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-02.png":::

3. En la sección **Clientes de escucha**, cree un nuevo cliente de escucha para cada sitio web. Especifique los detalles de cada cliente de escucha y seleccione **Agregar**.

4. A la izquierda, seleccione **Configuración HTTP** y seleccione **Agregar** en el panel de la izquierda. Complete los detalles para crear una nueva configuración HTTP y seleccione **Guardar**.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-03.png" alt-text="Captura de pantalla de la página Configuración HTTP para crear una nueva configuración HTTP." lightbox="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-03.png":::

5. Cree las reglas en la sección **Reglas** del menú de la izquierda. Asocie cada regla con el cliente de escucha correspondiente. Seleccione **Agregar**.

6. Configure el grupo de back-end y la configuración HTTP correspondientes. Seleccione **Agregar**.

7. Pruebe la conexión. Abra el explorador que prefiera y vaya a los distintos sitios web hospedados en su entorno de Azure VMware Solution, como por ejemplo http://www.fabrikam.com.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-07.png" alt-text="Captura de pantalla de la página del explorador que muestra que la prueba de la conexión se completa correctamente.":::

### <a name="routing-by-url"></a>Enrutamiento por dirección URL

En los pasos siguientes se definen grupos de direcciones de back-end mediante máquinas virtuales que se ejecutan en una nube privada de Azure VMware Solution. La nube privada está en una puerta de enlace de aplicación existente. A continuación, creará reglas de enrutamiento que garanticen que el tráfico web llega a los servidores adecuados de los grupos.

1. En la nube privada, cree un grupo de máquinas virtuales para representar la granja de servidores web. 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool.png" alt-text="Captura de pantalla de la página del cliente de VMSphere que muestra el resumen de otra VM.":::

    Se ha usado Windows Server 2016 con el rol de IIS instalado para ilustrar este tutorial. Una vez instaladas las máquinas virtuales, ejecute los comandos de PowerShell siguientes para configurar IIS para cada tutorial de VM. 

    La primera máquina virtual, contoso-web-01, hospedará el sitio web principal.

    ```powershell
    Install-WindowsFeature -Name Web-Server
    Add-Content -Path C:\inetpub\wwwroot\Default.htm -Value $($env:computername)
    ```

    La segunda máquina virtual, contoso-web-02, hospedará el sitio de imágenes.
 
    ```powershell
    Install-WindowsFeature -Name Web-Server
    New-Item -Path "C:\inetpub\wwwroot\" -Name "images" -ItemType "directory"
    Add-Content -Path C:\inetpub\wwwroot\images\test.htm -Value $($env:computername)
    ```

    La tercera máquina virtual, contoso-web-03, hospedará el sitio de vídeos.

    ```powershell
    Install-WindowsFeature -Name Web-Server
    New-Item -Path "C:\inetpub\wwwroot\" -Name "video" -ItemType "directory"
    Add-Content -Path C:\inetpub\wwwroot\video\test.htm -Value $($env:computername)
    ```

2. Agregue tres nuevos grupos de back-end en una instancia de Application Gateway existente. 

   1. Seleccione los **grupos back-end** en el menú de la izquierda. 
   1. Seleccione **Agregar** y escriba los detalles del primer grupo, **contoso-web**. 
   1. Agregue una máquina virtual como destino. 
   1. Seleccione **Agregar**. 
   1. Repita este proceso para **contoso-images** y **contoso-video**, agregando una máquina virtual única como destino. 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-02.png" alt-text="Captura de pantalla de la página Grupos de back-end que muestra la incorporación de tres grupos de back-end nuevos." lightbox="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-02.png":::

3. En la sección **Clientes de escucha**, cree un nuevo cliente de escucha de tipo Básico mediante el puerto 8080.

4. En el panel de navegación izquierdo, seleccione **Configuración HTTP** y seleccione **Agregar** en el panel izquierdo. Complete los detalles para crear una nueva configuración HTTP y seleccione **Guardar**.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-04.png" alt-text="Captura de pantalla de la página Agregar configuración HTTP que muestra la configuración de los valores HTTP.":::

5. Cree las reglas en la sección **Reglas** del menú de la izquierda. Asocie cada regla con el cliente de escucha creado anteriormente. A continuación, configure el grupo de back-end principal y la configuración HTTP. Seleccione **Agregar**.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-07.png" alt-text="Captura de pantalla de la página Agregar una regla de enrutamiento para configurar las reglas de enrutamiento a un destino de back-end.":::

6. Pruebe la configuración. Acceda a la puerta de enlace de aplicación en Azure Portal y copie la IP pública en la sección **Información general**. 

   1. Abra una ventana nueva del explorador y escriba la dirección URL `http://<app-gw-ip-address>:8080`. 

      :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-08.png" alt-text="Captura de pantalla de la página del explorador que muestra que la prueba de la configuración se completa correctamente.":::

   1. Cambie la dirección URL a `http://<app-gw-ip-address>:8080/images/test.htm`.

      :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-09.png" alt-text="Captura de pantalla de otra prueba correcta con la dirección URL nueva.":::

   1. Vuelva a cambiar la dirección URL a `http://<app-gw-ip-address>:8080/video/test.htm`.

      :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-10.png" alt-text="Captura de pantalla de una prueba correcta con la dirección URL final.":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha leído sobre el uso de Application Gateway para proteger una aplicación web que se ejecuta en Azure VMware Solution, se recomienda que aprenda sobre:

- [Configuración de Azure Application Gateway para distintos escenarios](../application-gateway/configuration-overview.md).
- [Implementación de Traffic Manager para equilibrar las cargas de trabajo de Azure VMware Solution](deploy-traffic-manager-balance-workloads.md).
- [Integración de Azure NetApp Files con cargas de trabajo basadas en Azure VMware Solution](netapp-files-with-azure-vmware-solution.md).
- [Protección de los recursos de Azure en redes virtuales](../ddos-protection/ddos-protection-overview.md).
