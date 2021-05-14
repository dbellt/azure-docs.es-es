---
title: Consideraciones sobre la topología de red para Application Proxy de Azure Active Directory
description: Consideraciones sobre la topología de red al utilizar Application Proxy de Azure Active Directory.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: a9b09193c94cf6e1674605312f0fbad03af1e7eb
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108186869"
---
# <a name="optimize-traffic-flow-with-azure-active-directory-application-proxy"></a>Optimización del flujo de tráfico con Application Proxy de Azure Active Directory

En este artículo se explica cómo optimizar el flujo de tráfico y se proporcionan consideraciones sobre la topología de red al utilizar Application Proxy de Azure Active Directory (Azure AD) para publicar y acceder a las aplicaciones de forma remota.

## <a name="traffic-flow"></a>Flujo de tráfico

Cuando se publica una aplicación a través del proxy de aplicación de Azure AD, el tráfico de los usuarios a las aplicaciones fluye a través de tres conexiones:

1. El usuario conecta con el punto de conexión público del servicio Azure AD Application Proxy en Azure
1. El servicio Application Proxy se conecta al conector del proxy de aplicación
1. El conector del proxy de aplicación se conecta a la aplicación de destino

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-three-hops.png" alt-text="Diagrama que muestra el flujo de tráfico del usuario a la aplicación de destino." lightbox="./media/application-proxy-network-topology/application-proxy-three-hops.png":::

## <a name="optimize-connector-groups-to-use-closest-application-proxy-cloud-service-preview"></a>Optimización de los grupos de conectores para usar el servicio en la nube de Application Proxy (versión preliminar)

Cuando se suscribe a un inquilino de Azure AD, la región de su inquilino viene determinada por el país o región que especifique. Al habilitar Application Proxy, se eligen las instancias del servicio en la nube de Application Proxy para el inquilino **predeterminadas** en la misma región que el inquilino de Azure AD, o la región más cercana a este.

Por ejemplo, si el país o la región del inquilino de Azure AD es Reino Unido, se asignará que todos los conectores de Application Proxy usen **de manera predeterminada** instancias de servicio de centros de datos europeos. Cuando los usuarios acceden a aplicaciones publicadas, el tráfico pasa por las instancias del servicio en la nube de Application Proxy de esta ubicación.

Si tiene conectores instalados en regiones que no sean la predeterminada, es posible que le convenga cambiar la región para la que se optimiza el grupo de conectores, con el fin de mejorar el rendimiento al acceder a estas aplicaciones. Una vez que se especifica una región para un grupo de conectores, se conectará a los servicios en la nube de Application Proxy de la región designada.

Para optimizar el flujo de tráfico y reducir la latencia de un grupo de conectores, asigne el grupo a la región más cercana. Para asignar una región:

> [!IMPORTANT]
> Para usar esta funcionalidad, los conectores deben utilizar, como mínimo, la versión 1.5.1975.0.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de aplicaciones del directorio que usa el proxy de aplicación. Por ejemplo, si el dominio del inquilino es contoso.com, el administrador debe ser admin@contoso.com o cualquier otro alias de administrador de ese dominio.
1. En la esquina superior derecha, seleccione su nombre de usuario. Compruebe que ha iniciado sesión en el directorio que usa Application Proxy. Si necesita cambiar directorios, seleccione **Cambiar directorio** y elija un directorio que use Application Proxy.
1. En el panel de navegación izquierdo, seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Application proxy**.
1. Seleccione **Nuevo grupo de conectores** y especifique un **nombre** para el grupo de conectores.
1. Después, en **Configuración avanzada**, seleccione la lista desplegable de Optimizar de región específica y seleccione la región más cercana a los conectores.
1. Seleccione **Crear**.
    
    :::image type="content" source="./media/application-proxy-network-topology/geo-routing.png" alt-text="Configurar un nuevo grupo de conectores." lightbox="./media/application-proxy-network-topology/geo-routing.png":::

1. Una vez que se haya creado el nuevo grupo de conectores, puede seleccionar los conectores que se van a asignar a él. 
   - Solo puede mover conectores al grupo de conectores si este está en un grupo de conectores que usa la región predeterminada. Lo mejor es empezar siempre con los conectores colocados en el "grupo predeterminado" y, después, moverlos al grupo de conectores adecuado.
   - La región de un grupo de conectores solo se puede cambiar si **no** hay conectores o aplicaciones asignados a él.
1. A continuación, asigne el grupo de conectores a sus aplicaciones. Al acceder a las aplicaciones, el tráfico ahora debería ir al servicio en la nube de Application Proxy que se encuentre en la región para la que está optimizado el grupo de conectores.

## <a name="considerations-for-reducing-latency"></a>Consideraciones para reducir la latencia

Todas las soluciones de proxy introducirán latencia en la conexión de red. Independientemente del proxy o solución VPN que elija como solución de acceso remoto, siempre se incluirá un conjunto de servidores que habilita la conexión hacia su la red corporativa.

Las organizaciones suelen incluir puntos de conexión de servidor en su red perimetral. Sin embargo, con el proxy de la aplicación de Azure AD, el tráfico fluye a través del servicio de proxy en la nube, mientras que los conectores residen en la red corporativa. No se requiere red perimetral.

Las secciones siguientes contienen sugerencias adicionales para ayudar a reducir la latencia aún más. 

### <a name="connector-placement"></a>Colocación del conector

Proxy de aplicación elige automáticamente la ubicación de las instancias en función de la ubicación del inquilino. Sin embargo, corresponde al usuario decidir dónde instalar el conector, con lo que tiene la posibilidad de definir las características de la latencia del tráfico de su red.

Al configurar el servicio de proxy de aplicación, formule las siguientes preguntas:

- ¿Dónde se encuentra la aplicación?
- ¿Dónde se encuentran la mayoría de los usuarios que acceden a la aplicación?
- ¿Dónde se encuentra la instancia del proxy de aplicación?
- ¿Ya tiene configurada una conexión de red dedicada a centros de datos de Azure (como ExpressRoute o una VPN similar)?

El conector tiene que comunicarse con Azure y con sus aplicaciones (pasos 2 y 3 del diagrama de flujo del tráfico); por eso, la ubicación del conector afecta a la latencia de esas dos conexiones. Al evaluar la posición del conector, tenga en cuenta lo siguiente:

- Si desea utilizar la delegación limitada de Kerberos (KCD) para el inicio de sesión único, el conector necesita una línea de visión a un centro de datos. Además, el servidor de conector debe estar unido al dominio.  
- Si no está seguro, instale el conector más cerca de la aplicación.

### <a name="general-approach-to-minimize-latency"></a>Método general para minimizar la latencia

Puede probar a minimizar la latencia del tráfico de extremo a extremo optimizando cada conexión de red. Cada conexión se puede optimizar mediante uno de estos modos:

- Reducir la distancia entre los dos extremos del salto.
- Elegir la red adecuada por la que pasar. Por ejemplo, puede ser más rápido pasar por una red privada en lugar de la red pública de Internet debido a los vínculos dedicados.

Si tiene un vínculo de VPN o ExpressRoute dedicado entre Azure y la red corporativa, puede que desee usarlo.

## <a name="focus-your-optimization-strategy"></a>Centrar la estrategia de optimización

Se puede hacer poco para controlar la conexión entre los usuarios y el servicio Aplication Proxy. Los usuarios pueden tener acceso a las aplicaciones desde una red doméstica, una cafetería u otro país o región. En cambio, puede optimizar las conexiones del servicio Application Proxy a los conectores del proxy de aplicación con las aplicaciones. Es aconsejable incorporar los siguientes patrones a su entorno.

### <a name="pattern-1-put-the-connector-close-to-the-application"></a>Patrón 1: colocar el conector cerca de la aplicación

Coloque el conector cerca de la aplicación de destino en la red del cliente. Esta configuración reduce al mínimo el paso 3 en el diagrama de la topología, porque el conector y la aplicación son similares.

Si el conector necesita una línea de visión al controlador de dominio, este modelo es ventajoso. La mayoría de nuestros clientes lo usa, puesto que funciona bien para la mayoría de los escenarios. Este modelo se puede combinar con el 2 para optimizar el tráfico entre el servicio y el conector.

### <a name="pattern-2-take-advantage-of-expressroute-with-microsoft-peering"></a>Patrón 2: sacar partido de ExpressRoute con emparejamiento de Microsoft

Si tiene una configuración de ExpressRoute con emparejamiento de Microsoft, puede usar la conexión de ExpressRoute más rápida para el tráfico entre el proxy de la aplicación y el conector. El conector sigue en su red; cierre la aplicación.

### <a name="pattern-3-take-advantage-of-expressroute-with-private-peering"></a>Patrón 3: sacar partido de ExpressRoute con el emparejamiento privado

Si tiene una configuración de VPN o ExpressRoute dedicada con emparejamiento privado entre Azure y la red corporativa, tiene otra opción. En esta configuración, la red virtual de Azure normalmente se considera una extensión de la red corporativa. Así pues, puede instalar el conector en el centro de datos de Azure y aun así cumplir los requisitos de latencia baja de la conexión del conector a la aplicación.

La latencia no se ve comprometida porque el tráfico está fluyendo por una red dedicada. También obtiene una mejor latencia del conector al servicio de proxy de la aplicación, ya que el conector está instalado en un centro de datos Azure cerca de la ubicación del inquilino de Azure AD.

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-expressroute-private.png" alt-text="Diagrama que muestra el conector instalado en un centro de datos de Azure" lightbox="./media/application-proxy-network-topology/application-proxy-expressroute-private.png":::

### <a name="other-approaches"></a>Otros enfoques

Aunque el centro de atención de este artículo es la selección de la ubicación del conector, también puede cambiar la ubicación de la aplicación para obtener mejores características de latencia.

Las organizaciones están cambiando sus redes a entornos hospedados cada vez con más frecuencia. Esto les permite colocar sus aplicaciones en un entorno hospedado que también forma parte de su red corporativa y seguir estando dentro del dominio. En este caso, los patrones analizados en las secciones anteriores se pueden aplicar a la nueva ubicación de la aplicación. Si se plantea esta opción, consulte [Servicios de dominio de Azure Active Directory](../../active-directory-domain-services/overview.md).

Además, considere organizar los conectores con [grupos de conectores](application-proxy-connector-groups.md) para las aplicaciones de destino que se encuentren en ubicaciones y redes distintas.

## <a name="common-use-cases"></a>Casos de uso comunes

En esta sección, se abordan algunos de los escenarios habituales. Suponga que el inquilino de Azure AD (y, por tanto, el punto de conexión de servicio del proxy) se encuentra en Estados Unidos. Los aspectos descritos en estos casos de uso también se aplican a otras regiones de todo el mundo.

Para estos escenarios, se llama "salto" a cada una de las conexiones y les asignamos un número para facilitar el análisis:

- **Salto 1**: usuario al servicio Application Proxy
- **Salto 2:** el servicio Application Proxy se conecta al conector del proxy de aplicación
- **Salto 3:** el conector del proxy de aplicación se conecta a la aplicación de destino 

### <a name="use-case-1"></a>Caso de uso 1

**Escenario**: la aplicación se encuentra en la red de una organización en Estados Unidos con usuarios en la misma región. No hay ExpressRoute ni VPN entre el centro de datos de Azure y la red corporativa.

**Recomendación:** siga el patrón 1 explicado en la sección anterior. Para mejorar la latencia, considere la posibilidad de usar ExpressRoute, si es necesario.

Se trata de un modelo simple. Puede optimizar el salto 3 colocando el conector cerca de la aplicación. Se trata además de una opción natural, ya que el conector se instala normalmente con línea de visión hacia la aplicación y el centro de datos para realizar operaciones de KCD.

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-pattern1.png" alt-text="Diagrama que muestra que los usuarios, el proxy, el conector y la aplicación están en EE. UU." lightbox="./media/application-proxy-network-topology/application-proxy-pattern1.png":::

### <a name="use-case-2"></a>Caso de uso 2

**Escenario**: la aplicación se encuentra en la red de una organización en Estados Unidos con usuarios dispersos por todo el mundo. No hay ExpressRoute ni VPN entre el centro de datos de Azure y la red corporativa.

**Recomendación:** siga el patrón 1 explicado en la sección anterior.

Nuevamente, el patrón más común es optimizar el salto 3, donde el conector se coloca cerca de la aplicación. El salto 3 no es costoso por lo general si todo se encuentra dentro de la misma región. Sin embargo, el salto 1 puede resultar más costoso dependiendo de dónde esté el usuario, ya que los usuarios de todo el mundo tendrán que acceder a la instancia del proxy de aplicación en Estados Unidos. Merece la pena mencionar que cualquier solución de proxy tendrá similares características con respecto a los usuarios que se reparten por todo el mundo.

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-pattern2.png" alt-text="Los usuarios están distribuidos globalmente, pero todo lo demás está en EE. UU." lightbox="./media/application-proxy-network-topology/application-proxy-pattern2.png":::

### <a name="use-case-3"></a>Caso de uso 3

**Escenario**: la aplicación se encuentra en una red de una organización en Estados Unidos. Hay una instancia de ExpressRoute con emparejamiento de Microsoft entre Azure y la red corporativa.

**Recomendación:** siga los modelos 1 y 2 explicados en la sección anterior.

En primer lugar, coloque el conector lo más próximo posible a la aplicación. Entonces, el sistema utilizará automáticamente ExpressRoute para el salto 2.

Si el vínculo de ExpressRoute usa emparejamiento de Microsoft, el tráfico entre el proxy y el conector pasará por ese vínculo. El salto 2 tiene la latencia optimizada.

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-pattern3.png" alt-text="Diagrama que muestra ExpressRoute entre el proxy y el conector" lightbox="./media/application-proxy-network-topology/application-proxy-pattern3.png":::

### <a name="use-case-4"></a>Caso de uso 4

**Escenario**: la aplicación se encuentra en una red de una organización en Estados Unidos. Hay una instancia de ExpressRoute con emparejamiento privado entre Azure y la red corporativa.

**Recomendación:** siga el modelo 3 explicado en la sección anterior.

Coloque el conector del centro de datos de Azure que esté conectado a la red corporativa a través del emparejamiento privado de ExpressRoute.

El conector puede colocarse en el centro de datos de Azure. Puesto que el conector todavía tiene una línea de visión hacia la aplicación y el centro de datos a través de la red privada, el salto 3 se mantiene optimizado. Además, el salto 2 se optimiza aún más.

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-pattern4.png" alt-text="Conector de Azure Datacenter, ExpressRoute entre el conector y la aplicación" lightbox="./media/application-proxy-network-topology/application-proxy-pattern4.png":::

### <a name="use-case-5"></a>Caso de uso 5

**Escenario:** la aplicación se encuentra en una red de la organización en Europa, la región predeterminada del inquilino es EE. UU. y la mayor parte de los usuarios están en Europa.

**Recomendación:** coloque el conector cerca de la aplicación. Actualice el grupo de conectores para que esté optimizado para usarlo en las instancias de servicio de Application Proxy de Europa. Para ver los pasos, consulte la sección [Optimización de los grupos de conectores para usar el servicio en la nube de Application Proxy más cercano](application-proxy-network-topology.md#optimize-connector-groups-to-use-closest-application-proxy-cloud-service-preview).

Puesto que los usuarios de Europa acceden a una instancia de Application Proxy que resulta estar en la misma región, el salto 1 no es caro. El salto 3 se optimiza. Es aconsejable usar ExpressRoute para optimizar el salto 2.

### <a name="use-case-6"></a>Caso de uso 6

**Escenario:** la aplicación se encuentra en una red de la organización en Europa, la región predeterminada del inquilino es EE. UU. y la mayor parte de los usuarios están en EE. UU.

**Recomendación:** coloque el conector cerca de la aplicación. Actualice el grupo de conectores para que esté optimizado para usarlo en las instancias de servicio de Application Proxy de Europa. Para ver los pasos, consulte la sección [Optimización de los grupos de conectores para usar el servicio en la nube de Application Proxy más cercano](application-proxy-network-topology.md#optimize-connector-groups-to-use-closest-application-proxy-cloud-service-preview). El salto 1 puede ser más caro, ya que todos los usuarios de EE. UU. deben acceder a la instancia de Application Proxy de Europa.

También puede considerar la utilización de alguna otra variante en esta situación. Si la mayoría de los usuarios de la organización están en Estados Unidos, lo más probable es que la red también se extienda a Estados Unidos. Coloque el conector en EE. UU., siga usando la región de EE. UU. predeterminada para los grupos de conectores y utilice la línea de red corporativa interna dedicada a la aplicación en Europa. De esta forma se optimizan los saltos 2 y 3.

:::image type="content" source="./media/application-proxy-network-topology/application-proxy-pattern5c.png" alt-text="Diagrama que muestra los usuarios, el proxy y el conector en EE. UU., y la aplicación en Europa." lightbox="./media/application-proxy-network-topology/application-proxy-pattern5c.png":::

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación del proxy de la aplicación](application-proxy-add-on-premises-application.md)
- [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
- [Habilitar el acceso condicional](../manage-apps/application-proxy-integrate-with-sharepoint-server.md)
- [Solucionar los problemas que tiene con el Proxy de aplicación](application-proxy-troubleshoot.md)
