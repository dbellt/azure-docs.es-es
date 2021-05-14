---
title: 'Trabajo de forma remota: Consideraciones sobre el trabajo remoto en la aplicación virtual de red (NVA)'
titleSuffix: Azure VPN Gateway
description: Conozca los aspectos que debe tener en cuenta al trabajar con aplicaciones virtuales de red (NVA) en Azure durante la pandemia de COVID-19.
services: vpn-gateway
author: scottnap
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: scottnap
ms.openlocfilehash: fd0c1f836abc60a53168098cc1bcce72ded2a120
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108229205"
---
# <a name="working-remotely-network-virtual-appliance-nva-considerations-for-remote-work"></a>Trabajo de forma remota: Consideraciones sobre el trabajo remoto en la aplicación virtual de red (NVA)

>[!NOTE]
>En este artículo se describe cómo puede aprovechar las aplicaciones virtuales de red, Azure, la red de Microsoft y el ecosistema de asociados de Azure para trabajar de forma remota y mitigar los problemas de red que está experimentando debido a la crisis provocada por la COVID-19.
>

Algunos clientes de Azure usan aplicaciones virtuales de red (NVA) de terceros de Azure Marketplace para proporcionar servicios críticos como VPN de punto a sitio para los empleados que trabajan desde casa durante la epidemia de COVID-19. En este artículo se describe una orientación de alto nivel que se debe tener en cuenta al aprovechar las NVA en Azure para proporcionar soluciones de acceso remoto.

## <a name="nva-performance-considerations"></a>Consideraciones sobre el rendimiento de las NVA

Todos los proveedores importantes de NVA en Azure Marketplace deben tener recomendaciones sobre el tamaño de VM y el número de instancias que se usarán al implementar sus soluciones.  Aunque casi todos los proveedores de NVA le permitirán elegir cualquier tamaño que esté disponible para usted en una región determinada, es muy importante que siga las recomendaciones de los proveedores con respecto a los tamaños de instancia de VM de Azure, ya que estas recomendaciones son los tamaños de VM en los que el proveedor ha realizado pruebas de rendimiento con Azure.  

### <a name="consider-the-following"></a>Considere lo siguiente

- **Capacidad y número de usuarios simultáneos**: este número es especialmente importante para los usuarios de VPN de punto a sitio, ya que cada usuario conectado creará un túnel cifrado (VPN SSL o IPSec).  
- **Rendimiento agregado**: el ancho de banda agregado que necesitará para incluir el número de usuarios necesarios a los cuales deberá proporcionar acceso remoto.
- **El tamaño de VM que necesitará**: siempre debe usar los tamaños de VM que recomienda el proveedor de NVA.  En el caso de una VPN de punto a sitio, si tiene muchas conexiones de usuario simultáneas, debe usar tamaños de VM más grandes, como máquinas virtuales de las [series Dv2 y DSv2](../virtual-machines/dv2-dsv2-series.md "Series Dv2 y Dsv2"). Estas máquinas virtuales tienden a tener más CPU virtuales y pueden manejar más sesiones de VPN simultáneas.  Además de tener más núcleos virtuales, los tamaños de máquina virtual más grandes de Azure tienen más capacidad de ancho de banda agregada que los tamaños de máquina virtual más pequeños.
    > **Importante:** Cada proveedor utiliza los recursos de manera diferente.  Si no está claro qué tamaños de instancia debe utilizar para dar cabida a la carga de usuarios estimada, debe ponerse en contacto directamente con el proveedor de software y solicitarle una recomendación.
- **Número de instancias**: si espera tener un gran número de usuarios y conexiones, existen límites con respecto a lo que puede conseguir al escalar de manera vertical los tamaños de las instancias de NVA.  Considere la posibilidad de implementar varias instancias de VM.
- **VPN de IPSec frente a VPN SSL**: en general, las implementaciones de VPN de IPSec funcionan mejor que las implementaciones de VPN SSL.  
- **Licencias**: asegúrese de que las licencias de software que adquirió para la solución de NVA cubran el crecimiento repentino que puede experimentar durante la epidemia de COVID-19.  Muchos programas de licencias de NVA limitan el número de conexiones o el ancho de banda que admite la solución.
- **Redes aceleradas**: considere una solución de NVA que tenga compatibilidad con las redes aceleradas.  Accelerated Networking habilita la virtualización de E/S de raíz única (SR-IOV) en una máquina virtual (VM), lo que mejora significativamente su rendimiento en la red. Este método de alto rendimiento omite el host de la ruta de acceso de datos, lo que reduce la latencia, la inestabilidad y la utilización de la CPU para usarse con las cargas de trabajo de red más exigentes en los tipos de máquina virtual admitidos. Las redes aceleradas son compatibles con la mayoría de los tamaños de instancia de uso general y optimizados para procesos de dos o más vCPU.

## <a name="monitoring-resources"></a>Supervisión de recursos

Cada solución de NVA tiene sus propias herramientas y recursos para supervisar el rendimiento de su NVA.  Consulte la documentación de los proveedores para asegurarse de entender las limitaciones de rendimiento y de poder detectar cuándo la NVA está cerca de alcanzar su capacidad.  Además, puede consultar Azure Monitor Network Insights y ver información básica sobre el rendimiento de las aplicaciones virtuales de red como:

- Uso de CPU
- Red interna
- Red interna
- Flujos de entrada
- Flujos de salida

## <a name="next-steps"></a>Pasos siguientes

La mayoría de los asociados de NVA importantes han publicado guías sobre cómo escalar debido al crecimiento inesperado y repentino durante la pandemia de COVID-19. Estos son algunos vínculos útiles a recursos de los asociados.

[Barracuda Enable Work from home while securing your data during COVID-19](https://www.barracuda.com/covid-19/work-from-home "Permita trabajar desde casa a la vez que protege sus datos durante la COVID-19") (Barracuda: Permita trabajar desde casa a la vez que protege sus datos durante la COVID-19)

[Check Point protege los recursos remotos durante la crisis del coronavirus](https://www.checkpoint.com/solutions/secure-remote-workforce-during-coronavirus/ "Protección de los recursos remotos durante la crisis del coronavirus")

[Cisco AnyConnect Implementation and Performance/Scaling Reference for COVID-19 Preparation](https://www.cisco.com/c/en/us/support/docs/security/anyconnect-secure-mobility-client/215331-anyconnect-implementation-and-performanc.html "Cisco: Implementación de AnyConnect y referencia de rendimiento y escalado para la preparación ante la COVID-19") (Cisco: Implementación de AnyConnect y referencia de rendimiento y escalado para la preparación ante la COVID-19)

[Citrix COVID-19 Response Support Center](https://www.citrix.com/support/covid-19-coronavirus.html "Citrix: Centro de apoyo a la respuesta ante la COVID-19") (Citrix: Centro de apoyo a la respuesta ante la COVID-19)

[F5 Guidance to Address the Dramatic Increase in Remote Workers](https://www.f5.com/business-continuity "F5: Guía para abordar el aumento drástico de trabajadores remotos") (F5: Guía para abordar el aumento drástico de trabajadores remotos)

[Fortinet COVID-19 Updates for Customers and Partners](https://www.fortinet.com/covid-19.html "Fortinet: Actualizaciones sobre la COVID-19 para clientes y asociados") (Fortinet: Actualizaciones sobre la COVID-19 para clientes y asociados)

[Palo Alto Networks COVID-19 Response Center](https://live.paloaltonetworks.com/t5/COVID-19-Response-Center/ct-p/COVID-19_Response_Center "Palo Alto Networks: Centro de respuesta frente a la COVID-19") (Palo Alto Networks: Centro de respuesta frente a la COVID-19)

[Kemp habilita el trabajo remoto y la experiencia de la aplicación AlwaysOn para la continuidad empresarial](https://kemptechnologies.com/remote-work-always-on-application-experience-business-continuity/ "Kemp habilita el trabajo remoto y la experiencia de la aplicación AlwaysOn para la continuidad empresarial")

