---
title: Configuración de los componentes de red NSX en Azure VMware Solution
description: Aprenda a usar la consola de Azure VMware Solution para configurar los segmentos de red NSX-T.
ms.topic: how-to
ms.date: 02/16/2021
ms.openlocfilehash: a4702efe8ea26234d074f421d2e6eab871eba5f5
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105733145"
---
# <a name="configure-nsx-network-components-in-azure-vmware-solution"></a>Configuración de los componentes de red NSX en Azure VMware Solution

Una nube privada de Azure VMware Solution se incluye de forma predeterminada con NSX-T como red definida por software (SDDC). Viene ya aprovisionada con una puerta de enlace NSX-T de nivel 0 en modo **activo-activo** y una puerta de enlace NSX-T predeterminada de nivel 1 en modo activo-en espera.  Estas puertas de enlace permiten conectar los segmentos (conmutadores lógicos) y proporcionan conectividad horizontal de derecha a izquierda y vertical de arriba abajo. 

Después de implementar la nube privada de Azure VMware Solution, puede configurar los objetos NSX-T necesarios desde la consola de Azure VMware Solution en **Workload Networking** (Red de la carga de trabajo).  La consola presenta la vista simplificada de las operaciones NSX-T que un administrador de VMware necesita diariamente y está dirigida a los usuarios que no están familiarizados con NSX-T.  

Tendrá cuatro opciones para configurar los componentes de NSX-T en la consola de Azure VMware Solution:
- **Segments** (Segmentos): cree segmentos que se muestren en NSX-T Manager y vCenter.
- **DHCP**: cree un servidor DHCP o una retransmisión DHCP si planea usar DHCP.
- **Port mirroring** (Creación de reflejo del puerto): cree la creación de reflejo del puerto para ayudar a solucionar los problemas de red.
- **DNS**: cree un reenviador DNS para enviar solicitudes DNS a un servidor DNS designado y resolverlas.  

>[!NOTE]
>Seguirá teniendo acceso a la consola de NSX-T Manager, donde puede usar la configuración avanzada mencionada y otras características de NSX-T. 
 
:::image type="content" source="media/configure-nsx-network-components-azure-portal/nsxt-workload-networking.png" alt-text="Captura de pantalla que muestra cuatro opciones de la consola de Azure VMware Solution para configurar NSX-T.":::

## <a name="prerequisites"></a>Prerrequisitos
Las máquinas virtuales (VM) creadas en la nube privada de Azure VMware Solution, o migradas a esta, se deben asociar a un segmento. 

## <a name="create-an-nsx-t-segment-in-the-azure-portal"></a>Creación de un segmento de NSX-T en Azure Portal
Puede crear y configurar un segmento de NSX-T desde la consola de Azure VMware Solution en Azure Portal.  Estos segmentos están conectados a la puerta de enlace predeterminada de nivel 1, y las cargas de trabajo de estos segmentos obtienen la conectividad horizontal de derecha a izquierda y vertical de arriba abajo. Una vez creado el segmento, se muestra en NSX-T Manager y vCenter.

>[!NOTE]
>Si tiene previsto usar DHCP, deberá [configurar un servidor DHCP o una retransmisión DHCP](#create-a-dhcp-server-or-dhcp-relay-in-the-azure-portal) antes de crear y configurar un segmento de NSX-T.

1. En la nube privada de Azure VMware Solution, en **Workload Networking** (Red de la carga de trabajo), seleccione **Segments** > **Add** (Segmentos > Agregar). Proporcione los detalles del nuevo segmento lógico y seleccione **Aceptar**.

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/add-new-nsxt-segment.png" alt-text="Captura de pantalla que muestra cómo agregar un nuevo segmento.":::
   
   - **Segment name** (Nombre del segmento): el nombre del conmutador lógico que está visible en vCenter.
   - **Subnet gateway** (Puerta de enlace de subred): dirección IP de la puerta de enlace de la subred del conmutador lógico con una máscara de subred. Las máquinas virtuales están conectadas a un conmutador lógico y todas las máquinas virtuales que se conectan a este conmutador pertenecen a la misma subred.  Además, todas las máquinas virtuales conectadas a este segmento lógico deben llevar una dirección IP del mismo segmento.
   - **DHCP** (opcional): intervalos DHCP para un segmento lógico. Para consumir DHCP en segmentos, se debe configurar un [servidor DHCP o una retransmisión DHCP](#create-a-dhcp-server-or-dhcp-relay-in-the-azure-portal).  
   - **Connected gateway** (Puerta de enlace conectada) -  *: se selecciona de forma predeterminada y es de solo lectura.*  Puerta de enlace de nivel 1 y tipo de información del segmento. 
      - **T1**: nombre de la puerta de enlace de nivel 1 en NSX-T Manager. Una nube privada de Azure VMware Solution incluye una puerta de enlace NSX-T de nivel 0 en modo activo-activo y una puerta de enlace NSX-T predeterminada de nivel 1 en modo activo-en espera.  Los segmentos creados a través de la consola de Azure VMware Solution solo se conectan a la puerta de enlace de nivel 1 predeterminada, y las cargas de trabajo de estos segmentos obtienen la conectividad horizontal de derecha a izquierda y vertical de arriba abajo. Solo puede crear más puertas de enlace de nivel 1 mediante NSX-T Manager. Las puertas de enlace de nivel 1 creadas desde la consola de NSX-T Manager no son visibles en la consola de Azure VMware Solution. 
      - **Type** (Tipo): segmento de superposición compatible con Azure VMware Solution.

   El segmento ahora está visible en la consola de Azure VMware Solution, NSX-T Manager y vCenter.

## <a name="create-a-dhcp-server-or-dhcp-relay-in-the-azure-portal"></a>Creación de un servidor DHCP o una retransmisión DHCP en Azure Portal
Puede crear un servidor o una retransmisión DHCP directamente desde la consola de Azure VMware Solution en Azure Portal. El servidor o la retransmisión DHCP se conectan a la puerta de enlace de nivel 1, que se crea al implementar Azure VMware Solution. Todos los segmentos en los que dio intervalos DHCP formarán parte de este DHCP.  Después de crear un servidor DHCP o una retransmisión DHCP, debe definir una subred o un intervalo en el nivel de segmento para consumirlo.

1. En la nube privada de Azure VMware Solution, en **Workload Networking** (Red de la carga de trabajo), seleccione **DHCP** > **Add** (DHCP > Agregar).

2. Seleccione **DHCP Server** (Servidor DHCP) o **DHCP Relay** (Retransmisión DHCP) y, luego, proporcione un nombre para el servidor o la retransmisión y tres direcciones IP. 

   >[!NOTE]
   >En el caso de la retransmisión DHCP, solo se necesita una dirección IP para una configuración correcta.

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/add-dhcp-server-relay.png" alt-text="Captura de pantalla que muestra cómo agregar un servidor DHCP o una retransmisión DHCP en Azure VMware Solution.":::

4. Complete la configuración de DHCP; para ello [proporcione intervalos DHCP en los segmentos lógicos](#create-an-nsx-t-segment-in-the-azure-portal) y, luego, seleccione **OK** (Aceptar).
 
## <a name="configure-port-mirroring-in-the-azure-portal"></a>Configuración de la creación de reflejo del puerto en Azure Portal
Puede configurar la creación de reflejo del puerto para supervisar el tráfico de red que supone el reenvío de una copia de cada paquete de un puerto de conmutación de red a otro. Esta opción coloca un analizador de protocolos en el puerto que recibe los datos reflejados. Se analiza el tráfico desde un origen, una máquina virtual o un grupo de máquinas virtuales y, luego, se envía a un destino definido. 

Para configurar la creación de reflejo del puerto en la consola de Azure VMware Solution, deberá hacer lo siguiente:

* [Paso 1. Cree máquinas virtuales o grupos de máquinas virtuales de origen y destino](#step-1-create-source-and-destination-vms-or-vm-groups): el grupo de origen tiene una sola máquina virtual o varias máquinas virtuales donde se refleja el tráfico.

* [Paso 2. Cree un perfil de creación de reflejo del puerto](#step-2-create-a-port-mirroring-profile): definirá la dirección del tráfico de los grupos de máquinas virtuales de origen y destino.

### <a name="step-1-create-source-and-destination-vms-or-vm-groups"></a>Paso 1. Creación de máquinas virtuales o grupos de máquinas virtuales de origen y destino

En este paso, creará un grupo de máquinas virtuales de origen y un grupo de máquinas virtuales de destino.

1. En la nube privada de Azure VMware Solution, en **Workload Networking** (Red de la carga de trabajo), seleccione **Port mirroring** > **VM groups** > **Add** (Creación de reflejo del puerto > Grupos de máquinas virtuales > Agregar).

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/add-port-mirroring-vm-groups.png" alt-text="Captura de pantalla que muestra cómo crear un grupo de máquinas virtuales para la creación de reflejo del puerto.":::

1. Proporcione un nombre para el nuevo grupo de máquinas virtuales, seleccione las máquinas virtuales que desee de la lista y, luego, haga clic en **OK** (Aceptar).

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/add-vm-group.png" alt-text="Captura de pantalla que muestra la lista de máquinas virtuales que se van a agregar al grupo de máquinas virtuales.":::

1. Repita estos pasos para crear el grupo de máquinas virtuales de destino.

### <a name="step-2-create-a-port-mirroring-profile"></a>Paso 2. Creación de un perfil de creación de reflejo del puerto

En este paso, definirá un perfil para la dirección del tráfico de los grupos de máquinas virtuales de origen y destino.

>[!NOTE]
>Asegúrese de que tiene creados los grupos de máquinas virtuales de origen y destino.

1. Seleccione **Port mirroring** > **Add** (Creación de reflejo del puerto > Agregar) y, luego, proporcione:

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/add-port-mirroring-profile.png" alt-text="Captura de pantalla que muestra la información necesaria para el perfil de creación de reflejo del puerto.":::

   - **Port mirroring name** (Nombre de la creación de reflejo del puerto): nombre descriptivo del perfil.
   - **Direction** (Dirección): seleccione entrada, salida o bidireccional.
   - **Source** (Origen): seleccione el grupo de máquinas virtuales de origen.
   - **Destination** (Destino): seleccione el grupo de máquinas virtuales de destino.
   - **Description** (Descripción): escriba una descripción para la creación de reflejo del puerto.

1. Seleccione **OK** (Aceptar) para completar el perfil. 

   El perfil y los grupos de máquinas virtuales están visibles en la consola de Azure VMware Solution.

## <a name="configure-a-dns-forwarder-in-the-azure-portal"></a>Configuración de un reenviador DNS en Azure Portal
Va a configurar un reenviador DNS en el que solicitudes DNS específicas se reenvían a un servidor DNS designado para resolverlas.  Un reenviador DNS se asocia con una **zona DNS predeterminada** y con hasta tres **zonas FQDN**.

>[!TIP]
>También puede usar la [consola de NSX-T Manager para configurar un reenviador de DNS](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/2.5/administration/GUID-A0172881-BB25-4992-A499-14F9BE3BE7F2.html).

Para configurar un reenviador DNS en la consola de Azure VMware Solution, deberá hacer lo siguiente:

* [Paso 1. Configure una zona DNS predeterminada y una zona FQDN](#step-1-configure-a-default-dns-zone-and-fqdn-zone): cuando se recibe una consulta de DNS, un reenviador DNS compara el nombre de dominio con los nombres de dominio de la zona DNS del nombre de dominio completo. 

* [Paso 2. Configure el servicio DNS](#step-2-configure-dns-service): configurará el servicio de reenvío DNS.

### <a name="step-1-configure-a-default-dns-zone-and-fqdn-zone"></a>Paso 1. Configuración de una zona DNS predeterminada y una zona FQDN
Configurará una zona DNS predeterminada y una zona FQDN para enviar consultas DNS al servidor que precede en la cadena.  Cuando se recibe una consulta DNS, el reenviador DNS compara el nombre de dominio de la consulta con los nombres de dominio de las zonas DNS del nombre de dominio completo. Si se encuentra una coincidencia, la consulta se reenvía a los servidores DNS especificados en la zona DNS del nombre de dominio completo. Si no se encuentra ninguna coincidencia, la consulta se reenvía a los servidores DNS especificados en la zona DNS predeterminada.

>[!NOTE]
>Debe definirse una zona DNS predeterminada antes de configurar una zona FQDN. 

1. En la nube privada de Azure VMware Solution, en **Workload Networking** (Red de la carga de trabajo), seleccione **DHCP** > **DNS Zones** > **Add** (DHCP > Zonas DNS > Agregar).

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/nsxt-workload-networking-dns-zones.png" alt-text="Captura de pantalla que muestra cómo agregar zonas DNS y un servicio DNS.":::

1. Seleccione **Default DNS zone** (Zona DNS predeterminada) y proporcione:

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/nsxt-workload-networking-configure-dns-zones.png" alt-text="Captura de pantalla que muestra cómo agregar una zona DNS predeterminada.":::

   1. Un nombre para la zona DNS.

   1. Hasta tres direcciones IP de servidor DNS en el formato **8.8.8.8**.

1. Seleccione **FQDN zone** (Zona FQDN) y proporcione:

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/nsxt-workload-networking-configure-fqdn-zone.png" alt-text="Captura de pantalla que muestra cómo agregar una zona FQDN.":::

   1. Un nombre para la zona DNS.

   1. El dominio FQDN.

   1. Hasta tres direcciones IP de servidor DNS en el formato **8.8.8.8**.

1. Seleccione **OK** (Aceptar) para terminar de agregar la zona DNS predeterminada y el servicio DNS.

### <a name="step-2-configure-dns-service"></a>Paso 2. Configuración del servicio DNS

1. Seleccione la pestaña **Servicio DNS** y elija **Agregar**. Proporcione los detalles y seleccione **Aceptar**.

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/nsxt-workload-networking-configure-dns-service.png" alt-text="Captura de pantalla que muestra la información necesaria para el servicio DNS.":::

   >[!TIP]
   >La **puerta de enlace de nivel 1** está seleccionada de forma predeterminada y refleja la puerta de enlace que se crea al implementar Azure VMware Solution.

   El servicio DNS se ha agregado correctamente.

   :::image type="content" source="media/configure-nsx-network-components-azure-portal/nsxt-workload-networking-configure-dns-service-success.png" alt-text="Captura de pantalla que muestra que el servicio DNS se ha agregado correctamente.":::

