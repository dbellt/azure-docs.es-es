---
title: 'Alta disponibilidad y equilibrio de carga: Application Proxy de Azure Active Directory'
description: Funcionamiento de la distribución del tráfico con la implementación de Application Proxy. Incluye sugerencias para optimizar el rendimiento de los conectores y usar el equilibrio de carga para los servidores back-end.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: conceptual
ms.date: 04/29/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 8dbff6b53133bace6643b21c8faa706ef817f1de
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108286283"
---
# <a name="high-availability-and-load-balancing-of-your-application-proxy-connectors-and-applications"></a>Alta disponibilidad y equilibrio de carga de los conectores y las aplicaciones de Application Proxy

En este artículo se explica cómo funciona la distribución del tráfico con la implementación de Application Proxy. Analizaremos:

- Cómo se distribuye el tráfico entre los usuarios y los conectores, junto con sugerencias para optimizar el rendimiento de los conectores.

- Cómo fluye el tráfico entre los conectores y los servidores de aplicaciones de back-end, con recomendaciones para el equilibrio de carga entre varios servidores back-end.

## <a name="traffic-distribution-across-connectors"></a>Distribución del tráfico entre conectores

Los conectores establecen sus conexiones en función de principios de alta disponibilidad. No hay ninguna garantía de que el tráfico se distribuya siempre uniformemente entre los conectores y no hay tampoco afinidad de sesión. Sin embargo, el uso varía y las solicitudes se envían aleatoriamente a las instancias de servicio de Application Proxy. Como resultado, el tráfico se distribuye por lo general de manera casi uniforme entre los conectores. En el diagrama y los pasos siguientes se muestra cómo se establecen las conexiones entre los usuarios y los conectores.

![Diagrama que muestra las conexiones entre los usuarios y los conectores](media/application-proxy-high-availability-load-balancing/application-proxy-connections.png)

1. Un usuario de un dispositivo cliente intenta acceder a una aplicación local publicada a través de Application Proxy.
2. La solicitud pasa por una instancia de Azure Load Balancer para determinar qué instancia de servicio de Application Proxy debe llevarla a cabo. Hay decenas de instancias disponibles por región para aceptar la solicitud. Este método ayuda a distribuir uniformemente el tráfico entre las instancias de servicio.
3. La solicitud se envía a [Service Bus](../../service-bus-messaging/index.yml).
4. Service Bus señala a un conector disponible. A continuación, el conector recoge la solicitud de Service Bus.
   - En el paso 2, las solicitudes van a diferentes instancias de servicio de Application Proxy, por lo que es más probable que las conexiones se realicen con conectores diferentes. Como resultado, los conectores se usan de manera casi uniforme dentro del grupo.
5. El conector pasa la solicitud al servidor back-end de la aplicación. A continuación, la aplicación envía la respuesta de vuelta al conector.
6. El conector completa la respuesta abriendo una conexión de salida a la instancia de servicio desde donde llegó la solicitud. Después, esta conexión se cierra inmediatamente. De forma predeterminada, cada conector está limitado a 200 conexiones de salida simultáneas.
7. A continuación, la respuesta se devuelve al cliente desde la instancia de servicio.
8. Las solicitudes posteriores de la misma conexión repiten los pasos anteriores.

Una aplicación suele tener muchos recursos y abre varias conexiones cuando se carga. Cada conexión sigue los pasos anteriores para su asignación a una instancia de servicio; si todavía no se ha emparejado con un conector, se selecciona un nuevo conector disponible.


## <a name="best-practices-for-high-availability-of-connectors"></a>Procedimientos recomendados para la alta disponibilidad de los conectores

- Debido al modo en que se distribuye el tráfico entre los conectores para lograr una alta disponibilidad, es esencial que un grupo de conectores cuente siempre al menos con dos conectores. Se prefieren tres conectores, para proporcionar más búfer entre los conectores. Para determinar el número correcto de conectores que necesita, siga la documentación de planeamiento de capacidad.

- Sitúe los conectores en diferentes conexiones de salida, para evitar un único punto de error. Si los conectores usan la misma conexión de salida, un problema de red en ella puede afectar a todos los conectores que la usan.

- Evite forzar el reinicio de los conectores cuando se conectan a aplicaciones de producción. Esto podría afectar negativamente a la distribución del tráfico entre los conectores. El reinicio de los conectores hace que haya menos conectores disponibles y fuerza las conexiones con el conector disponible restante. El resultado es un uso inicialmente desigual de los conectores.

- Evite todo tipo de inspección insertada en las comunicaciones TLS salientes entre los conectores y Azure. Este tipo de inspección insertada provoca la degradación en el flujo de la comunicación.

- Asegúrese de mantener las actualizaciones automáticas en ejecución para los conectores. Si se está ejecutando el servicio Application Proxy Connector Updater, los conectores se actualizan automáticamente y reciben las últimas actualizaciones de software. Si no ve el servicio Connector Updater en el servidor, debe volver a instalar el conector con el fin de obtener las actualizaciones.

## <a name="traffic-flow-between-connectors-and-back-end-application-servers"></a>Flujo de tráfico entre los conectores y los servidores de aplicaciones de back-end

Otra área clave donde la alta disponibilidad es un factor es la conexión entre los conectores y los servidores back-end. Cuando se publica una aplicación a través de Azure AD Application Proxy, el tráfico de los usuarios a las aplicaciones fluye a través de tres saltos:

1. El usuario se conecta al punto de conexión público del servicio Azure AD Application Proxy en Azure. La conexión se establece entre la dirección IP (pública) del cliente de origen y la dirección IP del punto de conexión de Application Proxy.
2. El conector de Application Proxy extrae la solicitud HTTP del cliente del servicio de Application Proxy.
3. El conector de Application Proxy se conecta a la aplicación de destino. El conector utiliza su propia dirección IP para establecer la conexión.

![Diagrama del usuario que se conecta a una aplicación a través de Application Proxy](media/application-proxy-high-availability-load-balancing/application-proxy-three-hops.png)

### <a name="x-forwarded-for-header-field-considerations"></a>Consideraciones sobre el campo de encabezado X-Forwarded-For
En algunas situaciones (como la auditoría, el equilibrio de carga, etc.), se debe cumplir el requisito de compartir la dirección IP de origen del cliente externo con el entorno local. Para solucionarlo, el conector de Azure AD Application Proxy agrega el campo de encabezado X-Forwarded-For con la dirección IP (pública) del cliente de origen a la solicitud HTTP. El dispositivo de red adecuado (equilibrador de carga, firewall), el servidor web o la aplicación de back-end pueden leer y usar la información.

## <a name="best-practices-for-load-balancing-among-multiple-app-servers"></a>Procedimientos recomendados para el equilibrio de carga entre varios servidores de aplicaciones
Cuando el grupo de conectores que se asigna a la aplicación de Application Proxy tiene dos o más conectores y se está ejecutando la aplicación web de back-end en varios servidores (granja de servidores), se necesita una buena estrategia de equilibrio de carga. Esta estrategia debe garantizar que los servidores recogen las solicitudes de cliente de manera uniforme e impiden el uso excesivo o insuficiente de los servidores de la granja.
### <a name="scenario-1-back-end-application-does-not-require-session-persistence"></a>Escenario 1: La aplicación de back-end no requiere la persistencia de la sesión
El escenario más sencillo es aquel en el que la aplicación web de back-end no necesita permanencia de la sesión (persistencia de la sesión). Cualquier solicitud del usuario se puede controlar mediante cualquier instancia de aplicación de back-end de la granja de servidores. Puede usar un equilibrador de carga de nivel 4 y configurarlo sin afinidad. Entre las opciones se incluyen el equilibrio de carga de red de Microsoft y Azure Load Balancer o un equilibrador de carga de otro proveedor. También se puede configurar un DNS round robin.
### <a name="scenario-2-back-end-application-requires-session-persistence"></a>Escenario 2: La aplicación de back-end requiere la persistencia de la sesión
En este escenario, la aplicación web de back-end requiere la permanencia de la sesión (persistencia de la sesión) durante la sesión autenticada. Todas las solicitudes del usuario deben controlarse mediante la instancia de la aplicación de back-end que se ejecuta en el mismo servidor de la granja de servidores.
Este escenario puede resultar más complicado, porque por lo general el cliente establece varias conexiones con el servicio Application Proxy. Las solicitudes en distintas conexiones pueden llegar a diferentes conectores y servidores de la granja. Dado que cada conector utiliza su propia dirección IP para esta comunicación, el equilibrador de carga no puede garantizar la permanencia de la sesión en función de la dirección IP de los conectores. Tampoco se puede usar la afinidad de la IP de origen.
Estas son algunas opciones para el escenario 2:

- Opción 1: basar la persistencia de la sesión en una cookie de sesión establecida por el equilibrador de carga. Se recomienda esta opción porque permite que la carga se extienda de manera más uniforme entre los servidores back-end. Requiere un equilibrador de carga de nivel 7 con esta capacidad, que puede controlar el tráfico HTTP y finalizar la conexión TLS. Puede usar Azure Application Gateway (Afinidad de sesión) o un equilibrador de carga de otro proveedor.

- Opción 2: basar la persistencia de la sesión en el campo de encabezado X-Forwarded-For. Esta opción requiere un equilibrador de carga de nivel 7 con esta capacidad, que puede controlar el tráfico HTTP y finalizar la conexión TLS.  

- Opción 3: configurar la aplicación de back-end para que no requiera la persistencia de la sesión.

Consulte la documentación del proveedor de software para conocer los requisitos de equilibrio de carga de la aplicación de back-end.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación del proxy de la aplicación](application-proxy-add-on-premises-application.md)
- [Habilitar el inicio de sesión único](application-proxy-configure-single-sign-on-with-kcd.md)
- [Habilitar el acceso condicional](../manage-apps/application-proxy-integrate-with-sharepoint-server.md)
- [Solucionar los problemas que tiene con el Proxy de aplicación](application-proxy-troubleshoot.md)
- [Aprenda cómo la arquitectura de Azure AD admite alta disponibilidad](../fundamentals/active-directory-architecture.md)