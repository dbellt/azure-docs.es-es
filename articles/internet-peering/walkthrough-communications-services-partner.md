---
title: Tutorial de emparejamiento de Internet de Azure para Communication Services
titleSuffix: Azure
description: Tutorial de emparejamiento de Internet de Azure para Communication Services
services: internet-peering
author: gthareja
ms.service: internet-peering
ms.topic: how-to
ms.date: 03/30/2021
ms.author: gatharej
ms.openlocfilehash: 4c4e3e2331ce334fbdd1c273fecd899bcd24612b
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124570"
---
# <a name="azure-internet-peering-for-communications-services-walkthrough"></a>Tutorial de emparejamiento de Internet de Azure para Communication Services

En esta sección se explican los pasos que un proveedor de servicios de comunicación debe seguir para establecer una interconexión directa con Microsoft.

**Proveedores de servicios de comunicación:** son las organizaciones que ofrecen servicios de comunicación (comunicaciones, mensajería, conferencias, etc.) y buscan integrar su infraestructura de servicios de comunicaciones (puerta de enlace de SBC/SIP, etc.) con Azure Communication Services y Microsoft Teams. 

El emparejamiento de Internet de Azure permite que los proveedores de servicios de comunicación establezcan una interconexión directa con Microsoft en cualquiera de sus sitios perimetrales (ubicaciones POP). La lista de todos los sitios públicos perimetrales está disponible en [PeeringDB](https://www.peeringdb.com/net/694).

El emparejamiento de Internet de Azure ofrece una interconexión de alta confiabilidad habilitada para QoS (calidad de servicio) para los servicios de comunicaciones a fin de garantizar servicios centrados en el rendimiento y la calidad.

## <a name="technical-requirements"></a>Requisitos técnicos
Los requisitos técnicos para establecer la interconexión directa con Communication Services son los siguientes:
-   El nodo del mismo nivel debe proporcionar un número de sistema autónomo (ASN), que debe ser público.
-   El nodo del mismo nivel debe tener una interconexión redundante (PNI) en cada ubicación de interconexión para garantizar la redundancia local.
-   El nodo del mismo nivel debe tener redundancia geográfica para garantizar la conmutación por error en caso de errores de sitio en la región o área metropolitana.
-   El nodo del mismo nivel debe tener las sesiones BGP en modo activo-activo para garantizar una alta disponibilidad y una convergencia más rápida. Además, no debe aprovisionarse como primario y copia de seguridad.
-   El nodo del mismo nivel debe mantener una relación 1:1 entre los enrutadores de emparejamiento del mismo nivel y los circuitos de emparejamiento, y no se aplica ninguna limitación de velocidad.
-   El nodo del mismo nivel debe proporcionar y anunciar su propio espacio de direcciones IPv4 enrutable públicamente que usarán los puntos de conexión de servicio de comunicaciones del nodo (por ejemplo, SBC). 
-   El nodo del mismo nivel debe proporcionar detalles de la clase de tráfico y los puntos de conexión hospedados en cada subred anunciada. 
-   El nodo del mismo nivel debe ejecutar BGP a través de la detección de reenvío bidireccional (BFD) para facilitar la convergencia de rutas de menos de un segundo.
-   Todos los prefijos de infraestructura de comunicaciones se registran en Azure Portal y se anuncian con la cadena de comunidad 8075:8007.
-   El nodo del mismo nivel no debe finalizar el emparejamiento en un dispositivo que ejecuta un firewall con estado. 
-   Microsoft configurará todos los vínculos de interconexión como LAG (conjuntos de vínculos) de manera predeterminada, por lo que el nodo del mismo nivel debe admitir LACP (protocolo de control de agregación de vínculos) en los vínculos de interconexión.

## <a name="establishing-direct-interconnect-with-microsoft-for-communications-services"></a>Establecimiento de la interconexión directa con Microsoft para servicios de comunicaciones

Para establecer una interconexión directa con el emparejamiento de Internet de Azure, siga estos pasos:

**1. Asocie el ASN público del nodo del mismo nivel a la suscripción a Azure:**

En caso de que el nodo del mismo nivel ya haya asociado el ASN público a una suscripción a Azure, omita este paso.

[Asociación de un ASN del mismo nivel a una suscripción de Azure mediante el portal - Azure | Microsoft Docs](./howto-subscription-association-portal.md)

El siguiente paso consiste en crear una conexión de emparejamiento directo para Peering Service.

> [!NOTE]
> Una vez aprobada la asociación de ASN, envíenos un correo electrónico a peeringservices@microsoft.com con su ASN y su identificador de suscripción para asociar la suscripción a Communication Services. 

**2. Cree una conexión de emparejamiento directo para Peering Service:**

Siga las instrucciones de [Creación o modificación de un emparejamiento directo mediante el portal](./howto-direct-portal.md).

Asegúrese de que cumple el requisito de alta disponibilidad.

Asegúrese de seleccionar las siguientes opciones en la página "Crear un emparejamiento":

Tipo de emparejamiento:   **Directo**

Red de Microsoft:  **8075**

SKU:        **Premium gratis**


En la página "Conexión de emparejamiento directo", seleccione las siguientes opciones:

Proveedor de direcciones de sesión:   **Microsoft**

Usar para servicios de emparejamiento:   **Habilitado**

> [!NOTE] 
> Omitir el siguiente mensaje mientras selecciona la activación de Peering Service.
> *No lo habilite a menos que se haya puesto en contacto con peering@microsoft.com acerca de ser un proveedor de MAPS.*


  **2a. Use una conexión de emparejamiento directo existente para Peering Service**

Si tiene un emparejamiento directo existente que quisiera usar para admitir Peering Service, puede activarlo en Azure Portal.
1.  Siga las instrucciones de [Conversión de un emparejamiento directo heredado en un recurso de Azure mediante el portal](./howto-legacy-direct-portal.md).
Si es necesario, solicite circuitos adicionales para cumplir el requisito de alta disponibilidad.

2.  Siga los pasos de [Habilitación de Peering Service](./howto-peering-service-portal.md) en un emparejamiento directo mediante el portal.




**3. Registre los prefijos para el enrutamiento optimizado**

Para un enrutamiento optimizado de los prefijos de la infraestructura de servicios de comunicación, debe registrar todos los prefijos con sus interconexiones de emparejamiento.
[Registro de Azure Peering Service: Azure Portal | Microsoft Docs](../peering-service/azure-portal.md)

La clave de prefijo se rellena automáticamente para los asociados de servicios de comunicaciones, por lo que el socio no necesita usar ninguna clave de prefijo para registrarse. 

Asegúrese de que los prefijos que se están registrando se anuncien a través de las interconexiones directas establecidas para la región.


## <a name="faqs"></a>Preguntas más frecuentes:

**P.**  Tengo subredes más pequeñas (< 24) para mis servicios de comunicaciones. ¿Las subredes más pequeñas también se pueden enrutar?

**R.**  Sí, Microsoft Azure Peering Service admite también el enrutamiento de prefijos más pequeños. Asegúrese de registrar los prefijos más pequeños para el enrutamiento y que estos se anuncien a través de las interconexiones.

**P.**  ¿Qué rutas de Microsoft se recibirán a través de estas interconexiones?

**R.** Microsoft anuncia todos los prefijos de servicios públicos de Microsoft a través de estas interconexiones. Esto garantizará que no solo podrá acceder a Communications, sino también a otros servicios en la nube desde la misma interconexión.

**P.**  Necesito establecer el límite de prefijos, ¿cuántas rutas anunciará Microsoft?

**R.** Microsoft anuncia aproximadamente 280 prefijos en Internet y puede aumentar en un 10-15 % más adelante. Por lo tanto, un límite adecuado de 400-500 puede ser una buena opción para configurar como "Número máximo de prefijos"

**P.** ¿Microsoft volverá a anunciar los prefijos del mismo nivel en Internet?

**R.** No.

**P.** ¿Se cobra por este servicio?

**R.** No, aunque se espera que el nodo del mismo nivel se encargue de los costos de conexión entre sitios.

**P.** ¿Cuál es la velocidad de vínculo mínima para una interconexión?

**R.** 10 Gbps.

**P.** ¿El nodo del mismo nivel está enlazado a un acuerdo de nivel de servicio?

**R.** Sí, una vez que el uso alcanza el 40 %, debe comenzar un proceso de aumento de LAG de entre 45 y 60 días.

**P.** ¿Cuál es la ventaja de este servicio frente al emparejamiento directo actual o ExpressRoute?

**R.** La ruta de acceso completa y libre de liquidación está optimizada para el tráfico de voz a través de la WAN de Microsoft, y la convergencia se optimiza para durar menos de un segundo con BFD.

**P.** ¿Cómo se lleva a cabo todo el proceso de incorporación?

**R.** El tiempo variará según el número y la ubicación de los sitios, y en función de si el nodo del mismo nivel está migrando los emparejamientos privados existentes o está configurando un nuevo cableado. El operador debe planear para 3 semanas o más.

**P.** ¿Se pueden usar las API para la incorporación?

**R.** Actualmente no hay compatibilidad con la API, y la configuración debe realizarse a través del portal web.