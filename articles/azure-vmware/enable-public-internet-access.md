---
title: Habilitar el acceso público a Internet en Azure VMware Solution
description: En este artículo se explica cómo usar la funcionalidad de IP pública en Azure Virtual WAN.
ms.topic: how-to
ms.date: 02/04/2021
ms.openlocfilehash: e5d4d73bf65842946e9c0f0c16869779e2e6f787
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777000"
---
# <a name="how-to-use-the-public-ip-functionality-in-azure-vmware-solution"></a>Uso de la funcionalidad de dirección IP pública en Azure VMware Solution

La dirección IP pública es una característica nueva de la conectividad de Azure VMware Solution. Hace que recursos tales como los servidores web, las máquinas virtuales (VM) y los hosts sean accesibles a través de una red pública. 

Puede habilitar el acceso público a Internet de dos maneras. 

- Las aplicaciones se pueden hospedar y publicar en el equilibrador de carga de Application Gateway para el tráfico HTTP/HTTPS.
- Publicado mediante las características de IP públicas de Azure Virtual WAN.

Como parte de la implementación de la nube privada de Azure VMware Solution, al habilitar la funcionalidad de IP pública, los componentes necesarios se crean y habilitan automáticamente:

-  Virtual WAN

-  Centro de Virtual WAN con conectividad a ExpressRoute

-  Servicios de Azure Firewall con IP pública

En este artículo se describe cómo puede usar la funcionalidad de IP pública en Virtual WAN.

## <a name="prerequisites"></a>Requisitos previos

- Entorno de Azure VMware Solution
- Un servidor web que se ejecute en el entorno de Azure VMware Solution.
- Un nuevo intervalo de direcciones IP no superpuestas para la implementación del concentrador de Virtual WAN, normalmente un valor `/24`.

## <a name="reference-architecture"></a>Arquitectura de referencia

:::image type="content" source="media/public-ip-usage/public-ip-architecture-diagram.png" alt-text="Diagrama de la arquitectura de la IP pública" border="false" lightbox="media/public-ip-usage/public-ip-architecture-diagram.png":::

El diagrama de arquitectura muestra un servidor web hospedado en el entorno de Azure VMware Solution que está configurado con direcciones IP privadas de tipo RFC1918.  Este servicio web está disponible en Internet a través de la funcionalidad de dirección IP pública de Virtual WAN.  La dirección IP pública normalmente es una NAT de destino traducida en Azure Firewall. Con las reglas de DNAT, la directiva de firewall traduce las solicitudes de direcciones IP públicas a una dirección privada (webserver) con un puerto.

Las solicitudes de usuario llegan al firewall de una dirección IP pública que, a su vez, se traduce en una dirección IP privada mediante las reglas DNAT de Azure Firewall. El firewall comprueba la tabla NAT y, si la solicitud coincide con una entrada, reenvía el tráfico a la dirección y el puerto traducidos en el entorno de Azure VMware Solution.

El servidor web recibe la solicitud, responde con la información o la página solicitada al firewall y, a continuación, este reenvía la información al usuario en la dirección IP pública.

## <a name="test-case"></a>Caso de prueba
En este escenario, va a publicar el servidor web IIS en Internet. Use la característica de IP pública en Azure VMware Solution para publicar el sitio web en una dirección IP pública.  También va a configurar las reglas NAT en el firewall y accederá a los recursos de Azure VMware Solution (máquinas virtuales con un servidor web) con la dirección IP pública.

>[!TIP]
>Para habilitar el tráfico de salida, debe establecer el valor de Configuración de seguridad > Tráfico de Internet en **Azure Firewall**.

## <a name="deploy-virtual-wan"></a>Implementar Virtual WAN.

1. Inicie sesión en Azure Portal y, a continuación, busque y seleccione **Azure VMware Solution**.

1. Seleccione la nube privada de Azure VMware Solution.

   :::image type="content" source="media/public-ip-usage/avs-private-cloud-resource.png" alt-text="Captura de pantalla de la nube privada de Azure VMware Solution." border="true" lightbox="media/public-ip-usage/avs-private-cloud-resource.png":::

1. En **Administrar**, seleccione **Conectividad**.

   :::image type="content" source="media/public-ip-usage/avs-private-cloud-manage-menu.png" alt-text="Captura de pantalla de la sección de conectividad." border="true" lightbox="media/public-ip-usage/avs-private-cloud-manage-menu.png":::

1. Seleccione la pestaña de **IP pública** y, a continuación, seleccione **Configurar**.

   :::image type="content" source="media/public-ip-usage/connectivity-public-ip-tab.png" alt-text="Captura de pantalla que muestra dónde empezar a configurar la dirección IP pública" border="true" lightbox="media/public-ip-usage/connectivity-public-ip-tab.png":::

1. Acepte los valores predeterminados o cámbielos; a continuación, seleccione **Crear**.

   - Grupo de recursos de Virtual WAN

   - Nombre de Virtual WAN

   - Bloque de direcciones de concentrador virtual (con el nuevo intervalo de direcciones IP no superpuestas)

   - Número de IP públicas (1-100)

Se tarda aproximadamente una hora en completar la implementación de todos los componentes. Esta implementación solo tiene que producirse una vez para admitir todas las IP públicas futuras en este entorno de Azure VMware Solution.  

>[!TIP]
>Puede supervisar el estado en el área de **Notificación**. 

## <a name="view-and-add-public-ip-addresses"></a>Ver y agregar direcciones IP públicas

Para comprobar y agregar más direcciones IP públicas, siga los pasos que se indican a continuación.

1. En Azure Portal, busque y seleccione **Firewall**.

1. Seleccione un firewall implementado y, a continuación, seleccione **Visit Azure Firewall Manager to configure and manage this firewall** (Visite Azure Firewall Manager para configurar y administrar este firewall).

   :::image type="content" source="media/public-ip-usage/configure-manage-deployed-firewall.png" alt-text="Captura de pantalla que muestra la opción para configurar y administrar el firewall" border="true" lightbox="media/public-ip-usage/configure-manage-deployed-firewall.png":::

1. Seleccione la opción de los **Centros virtuales protegidos** y, en la lista, seleccione un centro virtual.

   :::image type="content" source="media/public-ip-usage/select-virtual-hub.png" alt-text="Captura de pantalla de Firewall Manager" lightbox="media/public-ip-usage/select-virtual-hub.png":::

1. En la página del centro virtual, seleccione la **Configuración de IP pública** y, para agregar más direcciones IP públicas, seleccione **Agregar**. 

   :::image type="content" source="media/public-ip-usage/virtual-hub-page-public-ip-configuration.png" alt-text="Captura de pantalla que muestra cómo agregar una configuración de IP pública en Firewall Manager" border="true" lightbox="media/public-ip-usage/virtual-hub-page-public-ip-configuration.png":::

1. Proporcione el número de direcciones IP necesarias y seleccione **Agregar**.

   :::image type="content" source="media/public-ip-usage/add-number-of-ip-addresses-required.png" alt-text="Captura de pantalla para agregar un número especificado de configuraciones de dirección IP pública" border="true":::


## <a name="create-firewall-policies"></a>Creación de directivas de firewall

Una vez que se implementan todos los componentes, puede verlos en el grupo de recursos agregado. El siguiente paso es agregar una directiva de firewall.

1. En Azure Portal, busque y seleccione **Firewall**.

1. Seleccione un firewall implementado y, a continuación, seleccione **Visit Azure Firewall Manager to configure and manage this firewall** (Visite Azure Firewall Manager para configurar y administrar este firewall).

   :::image type="content" source="media/public-ip-usage/configure-manage-deployed-firewall.png" alt-text="Captura de pantalla que muestra la opción para configurar y administrar el firewall" border="true" lightbox="media/public-ip-usage/configure-manage-deployed-firewall.png":::

1. Seleccione **Directivas de Azure Firewall** y, a continuación, seleccione **Crear una directiva de Azure Firewall**.

   :::image type="content" source="media/public-ip-usage/create-firewall-policy.png" alt-text="Captura de pantalla que muestra cómo crear una directiva de firewall en Firewall Manager" border="true" lightbox="media/public-ip-usage/create-firewall-policy.png":::

1. En la pestaña **Conceptos básicos**, proporcione los detalles necesarios y seleccione **Siguiente: Configuración de DNS**. 

1. En la pestaña **DNS**, seleccione **Deshabilitar** y luego **Siguiente: Las reglas**.

1. Seleccione **Agregar una colección de reglas**, proporcione los detalles siguientes y seleccione **Agregar** y luego **Siguiente: Inteligencia sobre amenazas**.

   -  Name
   -  Tipo de colección de reglas: DNAT
   -  Prioridad
   -  Acción de la colección de reglas: permitir
   -  Nombre de la regla
   -  Tipo de origen: **IPaddress**
   -  Origen: **\***
   -  Protocolo **TCP**
   -  Puerto de destino: **80**
   -  Tipo de destino: **Dirección IP**
   -  Destino: **Dirección IP pública**
   -  Dirección traducida: **dirección IP privada del servidor web de Azure VMware Solution**
   -  Puerto traducido: **puerto del servidor web de Azure VMware Solution**

1. Deje el valor predeterminado y seleccione **Siguiente: Centros de conectividad**.

1. Seleccione **Asociar centros virtuales**.

1. Seleccione un centro de la lista y seleccione **Agregar**.

   :::image type="content" source="media/public-ip-usage/secure-hubs-with-azure-firewall-polcy.png" alt-text="Captura de pantalla que muestra los centros seleccionados que se convertirán en centros virtuales protegidos." border="true" lightbox="media/public-ip-usage/secure-hubs-with-azure-firewall-polcy.png":::

1. Seleccione **Siguiente: Etiquetas**. 

1. (Opcional) Cree pares de nombre y valor para clasificar los recursos. 

1. Seleccione **Siguiente: revisar y crear** y, a continuación, seleccione **Crear**.

## <a name="limitations"></a>Limitaciones

Puede tener 100 direcciones IP públicas por nube privada.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha visto cómo usar la funcionalidad de IP pública de Azure VMware Solution, puede que quiera obtener información sobre:

- Uso de direcciones IP públicas con [Azure Virtual WAN](../virtual-wan/virtual-wan-about.md).
- [Creación de un túnel IPSec en Azure VMware Solution](create-ipsec-tunnel.md).
