---
title: Configuración de la conmutación por error entre varios puntos de conexión con Traffic Manager
titleSuffix: Azure Content Delivery Network
description: Obtenga información sobre cómo configurar la conmutación por error en varios puntos de conexión de Azure Content Delivery Network mediante Azure Traffic Manager.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: how-to
ms.date: 10/08/2020
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: a003becba0bc1e42d8fe0c0c5b199402a430a8e1
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105034765"
---
# <a name="failover-across-multiple-endpoints-with-azure-traffic-manager"></a>Configuración de la conmutación por error entre varios puntos de conexión con Traffic Manager

Al configurar Azure Content Delivery Network (CDN), puede seleccionar el proveedor óptimo y el plan de tarifa que mejor se ajuste a sus necesidades. 

Azure CDN, con su infraestructura distribuida globalmente, crea de forma predeterminada redundancia local y geográfica, así como equilibrio de carga global para mejorar el rendimiento y disponibilidad del servicio. 

Si una ubicación no está disponible para servir contenido, las solicitudes se enrutan automáticamente a otra ubicación. El punto de presencia óptimo (POP) se usa para atender cada solicitud de cliente. El enrutamiento automático se basa en factores como la ubicación de la solicitud y la carga del servidor.
 
Si tiene varios perfiles de CDN, puede mejorar aún más la disponibilidad y el rendimiento con Azure Traffic Manager. 

Use Azure Traffic Manager con Azure CDN para equilibrar la carga entre varios puntos de conexión de CDN para realizar lo siguiente:
 
* Conmutación por error
* Equilibrio de carga geográfica 

En un escenario de conmutación por error típico, todas las solicitudes de cliente se dirigen al perfil de CDN principal. 

Si el perfil no está disponible, las solicitudes se dirigen al perfil secundario.  Asimismo, las solicitudes se reanudan en el perfil principal cuando vuelve a estar en línea.

Al usar Azure Traffic Manager de este modo se garantiza que la aplicación web esté disponible siempre. 

En este artículo se proporcionan instrucciones y un ejemplo de cómo configurar la conmutación por error con perfiles desde: 

* **Azure CDN estándar de Verizon**
* **Azure CDN Estándar de Akamai**

También se admite **Azure CDN de Microsoft**.

## <a name="create-azure-cdn-profiles"></a>Crear perfiles de Azure CDN
Cree dos o más perfiles de Azure CDN y puntos de conexión con diferentes proveedores.

1. Cree dos perfiles de CDN:
    * **Azure CDN estándar de Verizon**
    * **Azure CDN Estándar de Akamai** 

    Cree los perfiles siguiendo los pasos descritos en [Creación de un nuevo perfil de CDN](cdn-create-new-endpoint.md#create-a-new-cdn-profile).
 
   ![Varios perfiles de CDN](./media/cdn-traffic-manager/cdn-multiple-profiles.png)

2. En cada uno de los perfiles nuevos, cree al menos un punto de conexión siguiendo los pasos descritos en [Crear un nuevo extremo de CDN](cdn-create-new-endpoint.md#create-a-new-cdn-endpoint).

## <a name="create-traffic-manager-profile"></a>Creación de un perfil de Traffic Manager
Cree un perfil de Azure Traffic Manager y configure el equilibrio de carga entre los puntos de conexión de CDN. 

1. Cree un perfil de Azure Traffic Manager siguiendo los pasos descritos en [Crear un perfil de Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md). 

    * En el **Método de enrutamiento**, seleccione **Prioridad**.

2. Agregue los puntos de conexión de CDN a su perfil de Traffic Manager siguiendo los pasos descritos en [Incorporación de puntos de conexión de Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md#add-traffic-manager-endpoints).

    * En **Tipo**, seleccione **Puntos de conexión externos**.
    * En **Prioridad**, escriba un número.

    Por ejemplo, cree **cdndemo101akamai.azureedge.net** con una prioridad de **1** y **cdndemo101verizon.azureedge.net** con una prioridad de **2**.

   ![Puntos de conexión de CDN y Traffic Manager](./media/cdn-traffic-manager/cdn-traffic-manager-endpoints.png)


## <a name="configure-custom-domain-on-azure-cdn-and-azure-traffic-manager"></a>Configuración de un dominio personalizado en Azure CDN y Azure Traffic Manager
Después de configurar los perfiles de CDN y Traffic Manager, siga estos pasos para agregar la asignación de DNS y registrar el dominio personalizado en los puntos de conexión de CDN. En este ejemplo, el nombre de dominio personalizado es **cdndemo101.dustydogpetcare.online**.

1. Vaya al sitio web para el proveedor del dominio personalizado, como GoDaddy y cree dos entradas CNAME de DNS. 

    a. Para la primera entrada CNAME, asigne su dominio personalizado, con el subdominio cdnverify, al punto de conexión de CDN. Esta entrada es un paso necesario para registrar el dominio personalizado en el punto de conexión de CDN que agregó a Traffic Manager en el paso 2.

      Por ejemplo: 

      `cdnverify.cdndemo101.dustydogpetcare.online  CNAME  cdnverify.cdndemo101akamai.azureedge.net`  

    b. Para la segunda entrada de CNAME, asigne su dominio personalizado, sin el subdominio cdnverify, al punto de conexión de CDN. Esta entrada asigna el dominio personalizado a Traffic Manager. 

      Por ejemplo: 
      
      `cdndemo101.dustydogpetcare.online  CNAME  cdndemo101.trafficmanager.net`   

    > [!NOTE]
    > Si el dominio está actualmente activo y no se puede interrumpir, realice este paso al final. Compruebe que los puntos de conexión de CDN y los dominios de Traffic Manager estén activos antes de actualizar el DNS de dominio personalizado a Traffic Manager.
    >
   
    > [!NOTE]
    > Para implementar este escenario de conmutación por error, ambos puntos de conexión deben estar en diferentes perfiles, y los diferentes perfiles deben ser de diferentes proveedores de CDN para evitar conflictos de nombres de dominio.
    > 

2.  En el perfil de Azure CDN, seleccione el primer punto de conexión de CDN (Akamai). Seleccione **Agregar dominio personalizado** y especifique **cdndemo101.dustydogpetcare.online**. Compruebe que la marca de verificación para validar el dominio personalizado sea de color verde. 

    Azure CDN usa el subdominio **cdnverify** para validar la asignación de DNS para completar este proceso de registro. Para más información, consulte [Creación de un registro DNS de CNAME](cdn-map-content-to-custom-domain.md#create-a-cname-dns-record). En este paso se habilita a Azure CDN para que reconozca el dominio personalizado de modo que pueda responder a sus solicitudes.
    
    > [!NOTE]
    > Para habilitar TLS en una instancia de **Azure CDN desde perfiles de Akamai**, debe aplicar directamente cname de dominio personalizado al punto de conexión. cdnverify para habilitar TLS aún no se admite. 
    >

3.  Vuelva al sitio web del proveedor de dominios de su dominio personalizado. Actualice la primera asignación DNS que creó. Asigne el dominio personalizado al segundo punto de conexión de CDN.
                             
    Por ejemplo: 

    `cdnverify.cdndemo101.dustydogpetcare.online  CNAME  cdnverify.cdndemo101verizon.azureedge.net`  

4. En el perfil de Azure CDN, seleccione el segundo punto de conexión de CDN (Verizon) y repita el paso 2. Seleccione **Agregar dominio personalizado** y escriba **cdndemo101.dustydogpetcare.online**.
 
Después de completar estos pasos, el servicio de varias CDN con capacidades de conmutación por error estará configurado con Azure Traffic Manager. 

Puede acceder a las direcciones URL de prueba desde el dominio personalizado. 

Para probar la funcionalidad, deshabilite el punto de conexión de CDN principal y compruebe que la solicitud se mueve correctamente al punto de conexión de CDN secundario. 

## <a name="next-steps"></a>Pasos siguientes
También puede configurar otros métodos de enrutamiento, como el geográfico, para equilibrar la carga entre distintos puntos de conexión de CDN. 

Para obtener más información, consulte [Configuración del método de enrutamiento de tráfico geográfico con Traffic Manager](../traffic-manager/traffic-manager-configure-geographic-routing-method.md).
