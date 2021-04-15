---
title: Recuperación ante desastres mediante Azure DNS y Traffic Manager | Microsoft Docs
description: Introducción a las soluciones de recuperación ante desastres con Azure DNS y Traffic Manager.
services: dns
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/06/2021
ms.author: kumud
ms.openlocfilehash: 077e92b67f0cf6dac673cc870b7ff8c86fbe60dd
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106551295"
---
# <a name="disaster-recovery-using-azure-dns-and-traffic-manager"></a>Recuperación ante desastres mediante Azure DNS y Traffic Manager

La recuperación ante desastres se ocupa de recuperarse tras una pérdida grave de funcionalidad de la aplicación. Para elegir una solución de recuperación ante desastres, los propietarios del negocio y la tecnología deben, en primer lugar, determinar el nivel de funcionalidad que se necesita durante un desastre, por ejemplo: disponible, parcialmente disponible con funcionalidad reducida, disponibilidad diferida o totalmente disponible.
La mayoría de los clientes de empresa eligen una arquitectura de varias regiones para lograr resistencia frente a una conmutación por error en el nivel de aplicación o en el de infraestructura. Los clientes pueden elegir varios métodos para lograr alta disponibilidad y conmutación por error mediante una arquitectura con redundancia. Estos son algunos de los enfoques frecuentes:

- **Activo/pasivo con espera pasiva**: en esta solución de conmutación por error, las máquinas virtuales y otros dispositivos que se ejecutan en la región en espera no están activos hasta que se necesite la conmutación por error. Sin embargo, el entorno de producción se ha replicado en forma de copias de seguridad, imágenes de máquinas virtuales o plantillas de Resource Manager en otra región. Este mecanismo de conmutación por error es rentable pero tarda más tiempo en llevar a cabo una conmutación por error completa.
 
    ![Activo/pasivo con espera pasiva.](./media/disaster-recovery-dns-traffic-manager/active-passive-with-cold-standby.png)
    
    *Figura: configuración de recuperación ante desastres activo/pasivo con espera pasiva*

- **Activo/pasivo con luz piloto**: en esta solución de conmutación por error, el entorno en espera se establece con una configuración mínima. La configuración tiene únicamente en ejecución los servicios necesarios para admitir un conjunto mínimo y crítico de aplicaciones. En su formato nativo, este escenario solo puede ejecutar una funcionalidad mínima, pero puede escalar verticalmente y generar más servicios para realizar la mayor parte de la carga de producción si se produce una conmutación por error.
    
    ![Activo/pasivo con luz piloto](./media/disaster-recovery-dns-traffic-manager/active-passive-with-pilot-light.png)
    
    *Figura: configuración de recuperación ante desastres activo/pasivo con luz piloto*

- **Activo/pasivo con estado de espera semiactiva**: en esta solución de conmutación por error, la región en espera está activa previamente y está lista para tomar la carga de base, el escalado automático está activado y todas las instancias están en ejecución. Esta solución no se ha escalado para asumir la carga de producción completa, pero es funcional y todos los servicios están activos y en ejecución. Esta solución es una versión aumentada del enfoque de luz piloto.
    
    ![Activo/pasivo con estado de espera semiactiva](./media/disaster-recovery-dns-traffic-manager/active-passive-with-warm-standby.png)
    
    *Figura: configuración de recuperación ante desastres activo/pasivo con estado de espera semiactiva*
    
Para más información sobre la conmutación por error y la alta disponibilidad, consulte [Recuperación ante desastres para aplicaciones de Azure](/azure/architecture/resiliency/disaster-recovery-azure-applications).


## <a name="planning-your-disaster-recovery-architecture"></a>Planeamiento de la arquitectura de recuperación ante desastres

Hay dos aspectos técnicos en la configuración de la arquitectura de recuperación ante desastres:
-  El uso de un mecanismo de implementación para replicar instancias, datos y configuraciones entre entornos principales y en espera. Este tipo de recuperación ante desastres se puede realizar de forma nativa con Azure Site Recovery y mediante dispositivos o servicios de asociados de Microsoft Azure como Veritas o NetApp. 
- El desarrollo de una solución para desviar el tráfico de red y el tráfico web desde el sitio principal al sitio en espera. Este tipo de recuperación ante desastres se puede lograr con Azure DNS, Azure Traffic Manager (DNS) o equilibradores de carga globales de otros fabricantes.

Este artículo se limita a los enfoques de redireccionamiento del tráfico de red y del tráfico web. Para instrucciones sobre la configuración de Azure Site Recovery, consulte la [Documentación de Azure Site Recovery](../site-recovery/index.yml).
DNS es uno de los mecanismos más eficaces para desviar el tráfico de red porque a menudo es global y externo al centro de datos y está aislado de los errores del nivel de zona de disponibilidad (AZ) o región. Se puede usar un mecanismo de conmutación por error basado en DNS y en Azure dos servicios DNS pueden lograr lo mismo de algún modo: Azure DNS (DNS autoritativo) y Azure Traffic Manager (enrutamiento de tráfico inteligente basado en DNS). 

Es importante entender algunos conceptos sobre DNS que se utilizan ampliamente para explicar las soluciones proporcionadas en este artículo:
- **Registro A de DNS**: los registros A son punteros que apuntan un dominio a una dirección IPv4. 
- **CNAME o nombre canónico**: este tipo de registro se utiliza para apuntar a otro registro DNS. CNAME no responde con una dirección IP, sino con el puntero al registro que contiene la dirección IP. 
- **Enrutamiento ponderado**: se puede elegir asociar un peso a los puntos de conexión de servicio y, a continuación, distribuir el tráfico en función de los pesos asignados. Este método de enrutamiento es uno de los cuatro mecanismos de enrutamiento de tráfico disponibles en Traffic Manager. Para más información, consulte [Método de enrutamiento ponderado](../traffic-manager/traffic-manager-routing-methods.md#weighted).
- **Enrutamiento por prioridad**: el enrutamiento por prioridad se basa en las comprobaciones de mantenimiento de los puntos de conexión. De forma predeterminada, Azure Traffic Manager envía todo el tráfico al punto de conexión de prioridad más alta y, tras un error o un desastre, Traffic Manager enruta el tráfico al punto de conexión secundario. Para más información, consulte [Método de enrutamiento por prioridad](../traffic-manager/traffic-manager-routing-methods.md#priority-traffic-routing-method).

## <a name="manual-failover-using-azure-dns"></a>Conmutación por error manual con Azure DNS
La solución de conmutación por error manual de Azure DNS para la recuperación ante desastres utiliza el mecanismo de DNS estándar para realizar la conmutación por error al sitio de copia de seguridad. La opción manual mediante Azure DNS funciona mejor cuando se utiliza junto con el enfoque de espera pasiva o de luz piloto. 

![Conmutación por error manual con Azure DNS](./media/disaster-recovery-dns-traffic-manager/manual-failover-using-dns.png)

*Figura: Conmutación por error manual con Azure DNS*

Los supuestos realizados para la solución son:
- Los puntos de conexión principal y secundario tienen direcciones IP estáticas que no cambian con frecuencia. Supongamos que la dirección IP del sitio principal es 100.168.124.44 y la dirección IP del sitio secundario es 100.168.124.43.
- Existe una zona de Azure DNS tanto para el sitio principal como para el secundario. Supongamos que el punto de conexión para el sitio principal es prod.contoso.com y para el sitio de copia de seguridad es dr.contoso.com. También existe un registro DNS para la aplicación principal, conocido como www\.contoso.com.   
- El TTL es igual o inferior al del Acuerdo de Nivel de Servicio de RTO establecido en la organización. Por ejemplo, si una empresa establece el RTO de la respuesta ante desastres de aplicaciones en 60 minutos, el TTL debe ser inferior a 60 minutos, preferiblemente el menor posible. 
  Puede configurar Azure DNS para la conmutación por error manual como se indica a continuación:
- Creación de una zona DNS
- Creación de registros de zona DNS
- Actualización del registro CNAME

### <a name="step-1-create-a-dns"></a>Paso 1: Creación de una zona DNS
Cree una zona DNS (por ejemplo, www\.contoso.com) como se muestra a continuación:

![Creación de una zona DNS en Azure](./media/disaster-recovery-dns-traffic-manager/create-dns-zone.png)

*Figura: creación de una zona DNS en Azure*

### <a name="step-2-create-dns-zone-records"></a>Paso 2: creación de registros de zona DNS

Dentro de esta zona debe crear tres registros (por ejemplo: www\.contoso.com, prod.contoso.com y dr.contoso.com), como se muestra a continuación.

![Creación de registros de zona DNS](./media/disaster-recovery-dns-traffic-manager/create-dns-zone-records.png)

*Figura: creación de registros de zona DNS en Azure*

En este escenario, el sitio www\.contoso.com tiene un valor de TTL de 30 minutos, muy inferior al RTO indicado, y apunta al sitio de producción prod.contoso.com. Esta es la configuración durante las operaciones normales del negocio. El TTL de prod.contoso.com y dr.contoso.com se ha establecido en 300 segundos o 5 minutos. Puede usar un servicio de supervisión de Azure, como Azure Monitor o Azure App Insights, o cualquier solución de supervisión de asociados, como Dynatrace. Incluso puede usar soluciones domésticas que puedan supervisar o detectar errores de nivel de infraestructura virtual o de aplicación.

### <a name="step-3-update-the-cname-record"></a>Paso 3: actualización del registro CNAME

Una vez que se detecta un error, cambie el valor del registro para que apunte a dr.contoso.com como se muestra a continuación:
       
![Actualización del registro CNAME](./media/disaster-recovery-dns-traffic-manager/update-cname-record.png)

*Figura: actualización del registro CNAME en Azure*

En un plazo de 30 minutos, durante los cuales la mayoría de los sistemas de resolución actualizarán el archivo de zona almacenado en caché, cualquier consulta a www\.contoso.com se redirigirá a dr.contoso.com.
También puede ejecutar el siguiente comando de la CLI de Azure para cambiar el valor de CNAME:
 ```azurecli
   az network dns record-set cname set-record \
   --resource-group 123 \
   --zone-name contoso.com \
   --record-set-name www \
   --cname dr.contoso.com
```
Este paso se puede ejecutar manualmente o mediante automatización. Se puede realizar manualmente mediante la consola o mediante la CLI de Azure. La API y el SDK de Azure se pueden utilizar para automatizar la actualización de CNAME para que no sea necesaria ninguna intervención manual. La automatización se puede crear mediante Azure Functions, en una aplicación de supervisión de terceros o incluso desde el entorno local.

### <a name="how-manual-failover-works-using-azure-dns"></a>Funcionamiento de la conmutación por error manual con Azure DNS
Puesto que el servidor DNS está fuera de la zona de conmutación por error o zona de desastre, está aislado frente a los tiempos de inactividad. Esto permite al usuario diseñar un escenario de conmutación por error simple que es rentable y funcionará ininterrumpidamente suponiendo que el operador tenga conectividad de red durante el desastre y pueda realizar el cambio. Si la solución consiste en un script, es preciso asegurarse de que el servidor o el servicio que ejecuta el script está aislado del problema que afecta al entorno de producción. Además, tenga en cuenta que se estableció un valor de TTL bajo en la zona para que ningún sistema de resolución en todo el mundo mantenga el punto de conexión almacenado en memoria caché durante mucho tiempo y los clientes puedan acceder al sitio dentro del RTO. En los casos de espera pasiva y de luz piloto, dado que pueden ser necesarias algunas actividades de puesta en actividad y otras actividades administrativas, se debe dar tiempo suficiente antes de realizar el cambio.

## <a name="automatic-failover-using-azure-traffic-manager"></a>Conmutación por error automática con Azure Traffic Manager
Cuando hay arquitecturas complejas y varios conjuntos de recursos capaces de realizar la misma función, puede configurar Azure Traffic Manager (basado en DNS) para comprobar el mantenimiento de los recursos y enrutar el tráfico desde el recurso que no es correcto hacia el recurso correcto. En el ejemplo siguiente, la región primaria y la región secundaria tienen una implementación completa. Esta implementación incluye los servicios en la nube y una base de datos sincronizada. 

![Conmutación por error automática con Azure Traffic Manager](./media/disaster-recovery-dns-traffic-manager/automatic-failover-using-traffic-manager.png)

*Figura: conmutación por error automática con Azure Traffic Manager*

Sin embargo, la región principal es la única que controla activamente las solicitudes de red de los usuarios. La región secundaria no pasa a ser la activa hasta que la región principal experimenta una interrupción del servicio. En ese caso, todas las solicitudes de red nuevas se enrutan a la región secundaria. Puesto que la copia de seguridad de la base de datos es casi instantánea, los equilibradores de carga tienen direcciones IP cuyo mantenimiento se puede comprobar y las instancias están siempre en ejecución, esta topología proporciona una opción para alcanzar un RTO bajo y una conmutación por error sin ninguna intervención manual. La región de conmutación por error secundaria debe estar lista para empezar a trabajar inmediatamente después de que se produzca un error en la región principal.
Este escenario es ideal para el uso de Azure Traffic Manager, que dispone de sondeos integrados para varios tipos de comprobaciones de mantenimiento incluidas HTTP, HTTPS y TCP. Azure Traffic Manager también tiene un motor de reglas que se puede configurar para conmutar por error cuando se produce un error, como se describe a continuación. Veamos la solución siguiente con Traffic Manager:
- El cliente tiene el punto de conexión de la región 1, conocido como prod.contoso.com, con la dirección IP estática 100.168.124.44 y un punto de conexión en la región 2, conocido como dr.contoso.com, con la dirección IP estática 100.168.124.43. 
-   Cada uno de estos entornos tiene como front-end una propiedad de cara al público; por ejemplo, un equilibrador de carga. El equilibrador de carga se puede configurar para que tenga un punto de conexión basado en DNS o un nombre de dominio completo (FQDN), como se muestra a continuación.
-   Todas las instancias en la región 2 tienen replicación casi en tiempo real con la región 1. Además, las imágenes de las máquinas están actualizadas y todo el software y los datos de configuración tienen aplicadas las actualizaciones y están en consonancia con la región 1.  
-   El escalado automático está preconfigurado con antelación. 

Los pasos seguidos para configurar la conmutación por error con Azure Traffic Manager son los siguientes:
1. Creación de un nuevo perfil de Azure Traffic Manager
2. Creación de puntos de conexión en el perfil de Traffic Manager
3. Configuración de la comprobación de mantenimiento y de la conmutación por error

### <a name="step-1-create-a-new-azure-traffic-manager-profile"></a>Paso 1: creación de un nuevo perfil de Azure Traffic Manager
Cree un nuevo perfil de Azure Traffic Manager con el nombre contoso123 y seleccione el método de enrutamiento por prioridad. Si tiene un grupo de recursos existente y desea asociarlo, puede seleccionar un grupo de recursos existente; de lo contrario, cree un nuevo grupo de recursos.

![Creación de un perfil de Traffic Manager](./media/disaster-recovery-dns-traffic-manager/create-traffic-manager-profile.png)

*Ilustración: creación de un perfil de Traffic Manager*

### <a name="step-2-create-endpoints-within-the-traffic-manager-profile"></a>Paso 2: creación de puntos de conexión en el perfil de Traffic Manager

En este paso, creará los puntos de conexión que apuntan a los sitios de producción y de recuperación ante desastres. A continuación, elija el **Tipo** como un punto de conexión externo, pero si el recurso está hospedado en Azure, también puede elegir **Punto de conexión de Azure**. Si elige **Punto de conexión de Azure**, a continuación, seleccione un **Recurso de destino**, que puede ser una instancia de **App Service** o una **Dirección IP pública** que es asignada por Azure. La prioridad se establece en **1**, ya que es el servicio principal para la región 1.
Del mismo modo, cree también el punto de conexión de recuperación ante desastres en Traffic Manager.

![Creación de puntos de conexión de recuperación ante desastres](./media/disaster-recovery-dns-traffic-manager/create-disaster-recovery-endpoint.png)

*Figura: creación de puntos de conexión de recuperación ante desastres*

### <a name="step-3-set-up-health-check-and-failover-configuration"></a>Paso 3: configuración de la comprobación de mantenimiento y de la conmutación por error

En este paso, se establece el TTL de DNS en 10 segundos, que es respetado por la mayoría de los sistemas de resolución recursivos orientados a Internet. Esta configuración significa que ningún sistema de resolución DNS almacenará en memoria caché la información durante más de 10 segundos. Para la configuración de supervisión del punto de conexión, la ruta de acceso actualmente se establece en / o raíz, aunque puede personalizar la configuración del punto de conexión para que evalúe una ruta de acceso; por ejemplo, prod.contoso.com/index. En el ejemplo siguiente se muestra **https** como el protocolo de sondeo. No obstante, también puede elegir **http** o **tcp**. La elección del protocolo depende de la aplicación final. El intervalo de sondeo se establece en 10 segundos, lo que permite un sondeo rápido y los reintentos se establecen en 3. Como resultado, Traffic Manager conmutará por error al segundo punto de conexión si tres intervalos consecutivos registran un error. La siguiente fórmula define el tiempo total de una conmutación por error automática: Tiempo de una conmutación por error = TTL + Reintentos * Intervalo de sondeo. En este caso, el valor es 10 + 3 * 10 = 40 segundos (máximo).
Si los reintentos se establecen en 1 y el valor de TTL se establece en 10 segundos, el tiempo de la recuperación por error es 10 + 1 * 10 = 20 segundos. Establezca los reintentos en un valor mayor que **1** para eliminar las posibilidades de conmutaciones por error debidas a falsos positivos o a cualquier señal en la red de escasa relevancia. 


![Configuración de la comprobación de mantenimiento](./media/disaster-recovery-dns-traffic-manager/set-up-health-check.png)

*Figura: configuración de la comprobación de mantenimiento y de la conmutación por error*

### <a name="how-automatic-failover-works-using-traffic-manager"></a>Funcionamiento de la conmutación por error automática con Traffic Manager

Durante un desastre, se sondea el punto de conexión principal y el estado cambia a **degradado** y el sitio de recuperación ante desastres permanece **En línea**. De forma predeterminada, Traffic Manager envía todo el tráfico al punto de conexión principal (prioridad más alta). Si el punto de conexión principal aparece degradado, Traffic Manager enruta el tráfico al segundo punto de conexión mientras no funcione correctamente. Es posible configurar más puntos de conexión en Traffic Manager que pueden actuar como puntos de conexión de conmutación por error adicionales o como equilibradores de carga para compartir la carga entre puntos de conexión.

## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md).
- Obtenga más información sobre [Azure DNS](../dns/dns-overview.md).