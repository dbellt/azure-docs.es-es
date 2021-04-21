---
title: 'Tutorial: implementación y configuración de VMware HCX'
description: Aprenda a implementar y configurar una solución VMware HCX para la nube privada de Azure VMware Solution.
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: 9ee451ce4be59c51299d66e4042ed344026100cf
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107011007"
---
# <a name="deploy-and-configure-vmware-hcx"></a>Implementación y configuración de VMware HCX

En este artículo se muestra cómo implementar y configurar el conector local de VMware HCX para la nube privada de Azure VMware Solution. Con VMware HCX, es posible migrar las cargas de trabajo de VMware a Azure VMware Solution y a otros sitios conectados mediante varios tipos de migración. Dado que Azure VMware Solution implementa y configura el Administrador de la nube de HCX, debe descargar, activar y configurar el conector de HCX en el centro de datos de VMware local.

El conector avanzado de VMware HCX está implementado previamente en Azure VMware Solution. Admite hasta tres conexiones de sitio (de local a la nube o de nube a nube). Si necesita más de tres conexiones de sitio, envíe una [solicitud de soporte técnico](https://portal.azure.com/#create/Microsoft.Support) para habilitar el complemento [VMware HCX Enterprise](https://cloud.vmware.com/community/2019/08/08/introducing-hcx-enterprise/).  

>[!TIP]
>Aunque la herramienta VMware Configuration Maximum describe que el número máximo de pares de sitios entre On-Premises Connector y Cloud Manager es 25, la licencia lo limita a 3 para HCX Advanced y 10 para HCX Enterprise Edition.

>[!NOTE]
>VMware HCX Enterprise está disponible con Azure VMware Solution como un servicio en versión preliminar. Es gratuito y está sujeto a los términos y condiciones de un servicio en versión preliminar. Una vez que el servicio VMware HCX Enterprise esté disponible de forma general, recibirá un aviso de que en 30 días va a cambiar la facturación. También tendrá la opción de desactivar o no participar en el servicio. El cambio de HCx Enterprise a HCX Advanced se pude hacer sin tener que volver a realizar la implementación, pero tendrá que registrar una incidencia de soporte técnico para que esa acción tenga lugar. Si planea cambiar a una versión anterior, asegúrese de que no hay migraciones programadas o de que no se están utilizando características como RAV o MON.

Revise primero las secciones [Antes de empezar](#before-you-begin), [Requisitos de versión de software](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html) y [Requisitos previos](#prerequisites). 

A continuación, examinaremos todos los procedimientos necesarios para:

> [!div class="checklist"]
> * Descargar el archivo OVA del conector de VMware HCX.
> * Implementar el archivo OVA (conector de VMware HCX) de VMware HCX local.
> * Activar el conector de VMware HCX.
> * Emparejar el conector de VMware HCX local con el Administrador de la nube de HCX de Azure VMware Solution.
> * Configurar la interconexión (perfil de red, perfil de proceso y malla de servicios).
> * Para completar la instalación, compruebe el estado del dispositivo y valide que la migración es posible.

Una vez finalizado, realice los pasos siguientes recomendados al final de este artículo.  

## <a name="before-you-begin"></a>Antes de empezar

Al preparar la implementación, se recomienda que revise la siguiente documentación de VMware:

* [Guía de usuario de VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-E456F078-22BE-494B-8E4B-076EF33A9CF4.html)
* [Migración de máquinas virtuales con VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g)
* [Consideraciones de implementación en producción de VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/install-checklist/GUID-C0A0E820-D5D0-4A3D-AD8E-EEAA3229F325.html)
* [Serie de blogs de VMware: migración a la nube](https://blogs.vmware.com/vsphere/2019/10/cloud-migration-series-part-2.html) 
* [Puertos de red necesarios para VMware HCX](https://ports.vmware.com/home/VMware-HCX)


## <a name="prerequisites"></a>Requisitos previos

Si tiene previsto usar VMware HCX Enterprise, asegúrese de haber solicitado la activación mediante los canales de soporte técnico de Azure VMware Solution.


### <a name="on-premises-vsphere-environment"></a>Entorno de vSphere local

Asegúrese de que el entorno de vSphere local (entorno de origen) cumpla los [requisitos mínimos](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html). 

### <a name="network-and-ports"></a>Red y puertos

* [Global Reach de Azure ExpressRoute](tutorial-expressroute-global-reach-private-cloud.md) se configura entre los circuitos de ExpressRoute de nube privada de Azure VMware Solution y del entorno local.

* [Todos los puertos necesarios](https://ports.vmware.com/home/VMware-HCX) están abiertos para realizar la comunicación entre los componentes locales y el entorno local de Azure VMware Solution.

### <a name="ip-addresses"></a>Direcciones IP

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]
   
## <a name="download-the-vmware-hcx-connector-ova"></a>Descarga del archivo OVA del conector de VMware HCX

Antes de implementar el dispositivo virtual en la instancia local de vCenter, debe descargar el archivo OVA del conector de VMware HCX.  

1. En Azure Portal, seleccione la nube privada de Azure VMware Solution. 

1. Seleccione **Administrar** > **Conectividad** y, luego, la pestaña **HCX** para identificar la dirección IP del Administrador HCX de Azure VMware Solution. 

   :::image type="content" source="media/tutorial-vmware-hcx/find-hcx-ip-address.png" alt-text="Captura de pantalla de la dirección IP de VMware HCX." lightbox="media/tutorial-vmware-hcx/find-hcx-ip-address.png":::

1. Seleccione **Administrar** > **Identidad**. 

   Se muestran las direcciones URL y las credenciales de usuario para vCenter y NSX-T Manager de la nube privada.

   > [!TIP]
   > La contraseña de vCenter se definió al configurar la nube privada. Se trata de la misma contraseña que utilizará para iniciar sesión en Azure VMware Solution HCX Manager. Puede seleccionar **Generate a new password** (Generar una contraseña nueva) para generar contraseñas de vCenter y NSX-T.

   :::image type="content" source="media/tutorial-access-private-cloud/ss4-display-identity.png" alt-text="Mostrar las direcciones URL y las credenciales del administrador de NSX y vCenter de la nube privada" border="true":::.


1. Abra una ventana del explorador e inicie sesión en el Administrador de HCX de Azure VMware Solution en el puerto 443 en `https://x.x.x.9` con las credenciales de usuario **cloudadmin\@vsphere.local**.

1. Seleccione **Administración** > **Actualizaciones del sistema** y, a continuación, **Request Download Link** (Vínculo de descarga de solicitud).

1. Seleccione la opción que prefiera para descargar el archivo OVA del conector de VMware HCX.

## <a name="deploy-the-vmware-hcx-connector-ova-on-premises"></a>Implementación del archivo OVA del conector de VMware HCX local

1. En la instancia local de vCenter, seleccione una [plantilla OVF](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html) para implementar VMware HCX Connector. 

   > [!TIP]
   > Va a seleccionar el archivo OVA que ha descargado en el paso anterior.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-ovf-template.png" alt-text="Captura de pantalla de la búsqueda de una plantilla OVF." lightbox="media/tutorial-vmware-hcx/select-ovf-template.png":::


1. Seleccione un nombre y una ubicación y, luego, un recurso o clúster en el que va a implementar el conector de VMware HCX. A continuación, revise los detalles y los recursos necesarios y seleccione **Siguiente**.  

1. Revise los términos de la licencia. Si está de acuerdo con ellos, seleccione el almacenamiento y la red necesarios y, a continuación, seleccione **Next** (Siguiente).

1. Seleccione almacenamiento y, después, **Siguiente**.

1. Elija el segmento de red de administración de VMware HCX que ha definido anteriormente en la sección de [requisitos previos de las direcciones IP](#ip-addresses).  Después, seleccione **Siguiente**.

1. En **Plantilla personalizada**, escriba toda la información necesaria y seleccione **Siguiente**. 

   :::image type="content" source="media/tutorial-vmware-hcx/customize-template.png" alt-text="Captura de pantalla de los cuadros para personalizar una plantilla." lightbox="media/tutorial-vmware-hcx/customize-template.png":::

1. Verifique la configuración y seleccione **Finalizar** para implementar el archivo OVA de VMware HCX Connector.
   
   > [!IMPORTANT]
   > Tendrá que activar la aplicación virtual manualmente.  Tras encenderlo, espere entre 10 y 15 minutos antes de continuar en el paso siguiente.

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Implementación del dispositivo HCX](https://www.youtube.com/embed/UKmSTYrL6AY). 


## <a name="activate-vmware-hcx"></a>Activación de VMware HCX

Después de implementar el archivo OVA del conector de VMware HCX en el entorno local e iniciar el dispositivo, está listo para realizar la activación. En primer lugar, debe obtener una clave de licencia en el portal de Azure VMware Solution.

1. En el portal de Azure VMware Solution, vaya a **Manage** > **Connectivity** (Administrar > Conectividad), seleccione la pestaña **HCX** y, a continuación, seleccione **Add** (Agregar).

1. Use las credenciales del **administrador** para iniciar sesión en el administrador de VMware HCX local en `https://HCXManagerIP:9443`. 

   > [!TIP]
   > Ha definido la contraseña de usuario de **admin** durante la implementación del archivo OVA del Administrador de VMware HCX.

   > [!IMPORTANT]
   > Asegúrese de incluir el número de puerto `9443` con la dirección IP de VMware HCX Manager.

1. En **Licencias**, escriba la clave para **HCX Advanced Key** (Clave de HCX Advanced) y seleccione **Activar**.  
   
    > [!NOTE]
    > VMware HCX Manager debe tener acceso abierto a Internet o un proxy configurado.

1. En la **Datacenter Location** (Ubicación del centro de datos), proporcione la ubicación más cercana donde va a instalar el administrador de VMware HCX. Después, seleccione **Continuar**.

1. En **Nombre del sistema**, modifique el nombre o acepte el valor predeterminado y seleccione **Continuar**.
   
1. Seleccione **Yes, Continue** (Sí, continuar).

1. En **Connect your vCenter** (Conectar vCenter), proporcione el FQDN o la dirección IP del servidor de vCenter y las credenciales adecuadas y, a continuación, seleccione **Continuar**.
   
   > [!TIP]
   > El servidor vCenter es donde ha implementado el conector de VMware HCX en el centro de datos.

1. En **Configure SSO/PSC** (Configurar SSO/PSC), proporcione el FQDN o la dirección IP de Platform Services Controller y, a continuación, seleccione **Continue** (Continuar).
   
   > [!NOTE]
   > Normalmente, suele coincidir con la dirección IP o el FQDN de la instancia de vCenter.

1. Compruebe que toda la información especificada es correcta y seleccione **Reiniciar**.
    
   > [!NOTE]
   > Verá que el proceso se ralentiza después de reiniciar y antes de que se le solicite realizar el siguiente paso.

Una vez que se reinicien los servicios, verá que vCenter se muestra en verde en la pantalla que aparece. Tanto vCenter como SSO deben tener los parámetros de configuración adecuados, que deben ser los mismos que los de la pantalla anterior.

:::image type="content" source="media/tutorial-vmware-hcx/activation-done.png" alt-text="Captura de pantalla del panel con el estado de vCenter en color verde." lightbox="media/tutorial-vmware-hcx/activation-done.png":::  

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Activación de HCX](https://www.youtube.com/embed/PnVg6SZkQsY?rel=0&amp;vq=hd720).

   > [!IMPORTANT]
   > Tanto si usa HCX Advanced como HCX Enterprise, es posible que tenga que instalar la revisión del [artículo de KB 81558](https://kb.vmware.com/s/article/81558) de VMware. 

## <a name="configure-the-vmware-hcx-connector"></a>Configuración del conector de VMware HCX

Ya está listo para agregar un emparejamiento de sitios, crear un perfil de red y de proceso y habilitar servicios como la migración, la extensión de red o la recuperación ante desastres. 

### <a name="add-a-site-pairing"></a>Agregar un emparejamiento de sitios

Puede conectar o emparejar VMware HCX Cloud Manager en Azure VMware Solution con la instancia de VMware HCX Connector del centro de datos. 

1. Inicie sesión en la instancia local de vCenter y en **Inicio**, seleccione **HCX**.

1. En **Infrastructure** (Infraestructura), seleccione **Site Pairing** (Emparejamiento de sitios) y, a continuación, seleccione la opción **Connect To Remote Site** (Conectar con el sitio remoto) que se encuentra en medio de la pantalla. 

1. Escriba la dirección IP o la dirección URL del Administrador de la nube de HCX de Azure VMware Solution anotada anteriormente `https://x.x.x.9`, el nombre de usuario cloudadmin\@vsphere.local de Azure VMware Solution y la contraseña. A continuación, seleccione **Conectar**.

   > [!NOTE]
   > Para establecer correctamente un emparejamiento de sitios:
   > * VMware HCX Connector debe poder enrutarse a la dirección IP de HCX Cloud Manager a través del puerto 443.
   >
   > * Utilice la misma contraseña que usó para iniciar sesión en vCenter. Recuerde que definió esta contraseña en la pantalla de implementación inicial.

   Verá una pantalla que muestra cómo su instancia de VMware HCX Cloud Manager en Azure VMware Solution y su instancia local de VMware HCX Connector están conectadas (emparejadas).

   :::image type="content" source="media/tutorial-vmware-hcx/site-pairing-complete.png" alt-text="Captura de pantalla que muestra el emparejamiento de HCX Manager en Azure VMware Solution y VMware HCX Connector.":::

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Emparejamiento de sitios de HCX](https://www.youtube.com/embed/jXOmYUnbWZY?rel=0&amp;vq=hd720).

### <a name="create-network-profiles"></a>Creación de perfiles de red

El conector de VMware HCX implementa un subconjunto de aplicaciones virtuales (automatizadas) que requieren varios segmentos IP. Al crear los perfiles de red, utiliza los segmentos IP que ha identificado durante la [fase de preparación y planeación previas a la implementación de los segmentos de red de VMware HCX](production-ready-deployment-steps.md#vmware-hcx-network-segments).

Va a crear cuatro perfiles de red:

   - Administración
   - vMotion
   - Replicación
   - Vínculo superior

1. En **Infrastructure** (Infraestructura), seleccione **Interconnect** > **Multi-Site Service Mesh** > **Network Profiles** > **Create Network Profile** (Interconectar > Malla de servicios de varios sitios > Perfiles de red > Crear perfil de red).

   :::image type="content" source="media/tutorial-vmware-hcx/network-profile-start.png" alt-text="Captura de pantalla de las selecciones para empezar a crear un perfil de red." lightbox="media/tutorial-vmware-hcx/network-profile-start.png":::

1. En cada perfil de red, seleccione la red y el grupo de puertos, proporcione un nombre y cree el grupo de direcciones IP del segmento. Seleccione **Crear**. 

   :::image type="content" source="media/tutorial-vmware-hcx/example-configurations-network-profile.png" alt-text="Captura de pantalla de los detalles de un nuevo perfil de red.":::

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Perfil de red de HCX](https://www.youtube.com/embed/O0rU4jtXUxc).


### <a name="create-a-compute-profile"></a>Creación de un perfil de proceso

1. En **Infraestructura**, seleccione **Interconnect (Interconectar)**  > **Compute Profiles (Perfiles de proceso)**  > **Create Compute Profile (Crear perfiles de proceso)** .

   :::image type="content" source="media/tutorial-vmware-hcx/compute-profile-create.png" alt-text="Captura de pantalla que muestra las selecciones para empezar a crear un perfil de proceso." lightbox="media/tutorial-vmware-hcx/compute-profile-create.png":::

1. Escriba un nombre para el perfil y seleccione **Continuar**.  

   :::image type="content" source="media/tutorial-vmware-hcx/name-compute-profile.png" alt-text="Captura de pantalla que muestra la entrada de un nombre de perfil de proceso y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/name-compute-profile.png":::

1. Seleccione los servicios que se van a habilitar, como la migración, la extensión de red o la recuperación ante desastres y, a continuación, seleccione **Continue** (Continuar).
  
   > [!NOTE]
   > Por lo general, no es necesario cambiar nada.

1. En **Select Service Resources** (Seleccionar recursos de servicio), seleccione uno o más recursos de servicio (clústeres) para habilitar los servicios de VMware HCX seleccionados.  

1. Cuando vea los clústeres en el centro de datos local, seleccione **Continuar**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-service-resource.png" alt-text="Captura de pantalla que muestra los recursos de servicio seleccionados y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/select-service-resource.png":::

1. En **Select Datastore** (Seleccionar almacén de datos), seleccione el recurso de almacenamiento del almacén de datos para implementar los dispositivos de interconexión de VMware HCX. Después, seleccione **Continuar**.

   Cuando se seleccionan varios recursos, VMware HCX usa el primero seleccionado hasta que se agota su capacidad.   

   :::image type="content" source="media/tutorial-vmware-hcx/deployment-resources-and-reservations.png" alt-text="Captura de pantalla que muestra un recurso de almacenamiento de datos seleccionado y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/deployment-resources-and-reservations.png":::  

1. En **Select Management Network Profile** (Seleccionar perfil de red de administración), seleccione el perfil de red de administración que creó en los pasos anteriores. Después, seleccione **Continuar**.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-management-network-profile.png" alt-text="Captura de pantalla que muestra la selección de un perfil de red de administración y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/select-management-network-profile.png":::

1. En **Select Uplink Network Profile** (Seleccionar perfil de red de vínculo superior), seleccione el perfil de red de vínculo superior que ha creado en el procedimiento anterior. Después, seleccione **Continuar**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-uplink-network-profile.png" alt-text="Captura de pantalla que muestra la selección de un perfil de red de vínculo superior y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/select-uplink-network-profile.png":::

1. En **Select vMotion Network Profile** (Seleccionar perfil de red de vMotion), seleccione el perfil de red que creó en los pasos anteriores. Después, seleccione **Continuar**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-vmotion-network-profile.png" alt-text="Captura de pantalla que muestra la selección de un perfil de red de vMotion y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/select-vmotion-network-profile.png":::

1. En **Select vSphere Replication Network Profile** (Seleccionar perfil de red de replicación de vSphere), seleccione el perfil de red que creó en los pasos anteriores. Después, seleccione **Continuar**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-replication-network-profile.png" alt-text="Captura de pantalla que muestra la selección de un perfil de red de replicación y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/select-replication-network-profile.png":::

1. En **Select Distributed Switches for Network Extensions** (Seleccionar los conmutadores distribuidos para las extensiones de red), seleccione los conmutadores que contienen las máquinas virtuales que se van a migrar a Azure VMware Solution en una red extendida de nivel 2. Después, seleccione **Continuar**.

   > [!NOTE]
   > Si no va a migrar máquinas virtuales en redes extendidas de nivel 2 (L2), puede omitir este paso.
   
   :::image type=" content" source="media/tutorial-vmware-hcx/select-layer-2-distributed-virtual-switch.png" alt-text="Captura de pantalla que muestra la selección de los conmutadores virtuales distribuidos y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/select-layer-2-distributed-virtual-switch.png":::

1. Revise las reglas de conexión y seleccione **Continue** (Continuar).  

   :::image type="content" source="media/tutorial-vmware-hcx/review-connection-rules.png" alt-text="Captura de pantalla que muestra las reglas de conexión y el botón Continue (Continuar)." lightbox="media/tutorial-vmware-hcx/review-connection-rules.png":::

1. Seleccione **Finish** (Finalizar) para crear el perfil de proceso.


   :::image type="content" source="media/tutorial-vmware-hcx/compute-profile-done.png" alt-text="Captura de pantalla que muestra información del perfil de proceso." lightbox="media/tutorial-vmware-hcx/compute-profile-done.png":::

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Perfil de proceso](https://www.youtube.com/embed/e02hsChI3b8).

### <a name="create-a-service-mesh"></a>Creación de una malla de servicios

Este es el momento de configurar una malla de servicios entre el entorno local y la nube privada de Azure VMware Solution.



> [!NOTE]
> Para establecer correctamente una malla de servicios con Azure VMware Solution:
>
> * Los puertos UDP 500/4500 están abiertos entre las direcciones del perfil de red de "vínculo superior" de la instancia local de VMware HCX Connector y las direcciones del perfil de red de "vínculo superior" de Azure VMware Solution HCX Cloud.
>
> * Asegúrese de revisar los [puertos necesarios de VMware HCX](https://ports.vmware.com/home/VMware-HCX).

1. En la opción **Infrastructure** (Infraestructura), seleccione **Interconnect** > **Service Mesh** > **Create Service Mesh** (Interconexión > Malla de servicios > Crear malla de servicios).    

   :::image type="content" source="media/tutorial-vmware-hcx/create-service-mesh.png" alt-text="Captura de pantalla de las selecciones para empezar a crear una malla de servicios." lightbox="media/tutorial-vmware-hcx/create-service-mesh.png":::

1. Revise los sitios que se han rellenado previamente y, a continuación, seleccione **Continuar**. 

   > [!NOTE]
   > Si esta es la primera configuración de la malla de servicios, no tendrá que modificar esta pantalla.  

1. Seleccione los perfiles de proceso de origen y remoto en las listas desplegables y, a continuación, seleccione **Continue** (Continuar).  

   Las selecciones definen los recursos donde las VM pueden consumir los servicios de VMware HCX.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-compute-profile-source.png" alt-text="Captura de pantalla que muestra la selección del perfil de proceso de origen." lightbox="media/tutorial-vmware-hcx/select-compute-profile-source.png":::

   :::image type="content" source="media/tutorial-vmware-hcx/select-compute-profile-remote.png" alt-text="Captura de pantalla que muestra la selección del perfil de proceso remoto." lightbox="media/tutorial-vmware-hcx/select-compute-profile-remote.png":::

1. Revise los servicios que se van a habilitar y, a continuación, seleccione **Continue** (Continuar).  

1. En **Advanced Configuration - Override Uplink Network profiles** (Configuración avanzada: Invalidar los perfiles de red de vínculo superior), seleccione **Continue** (Continuar).  

   Los perfiles de red de vínculo superior se conectan a la red a través de la cual se puede acceder a los dispositivos de interconexión del sitio remoto.  
  
1. En **Advanced Configuration - Network Extension Appliance Scale Out** (Configuración avanzada: escalabilidad horizontal del dispositivo de extensión de red), revise la información y seleccione **Continuar**. 

   Puede tener hasta ocho VLAN por dispositivo. También puede implementar otro dispositivo para agregar ocho VLAN más. Además, debe disponer de espacio de IP para los dispositivos adicionales. Es una IP por dispositivo.  Para más información, consulte los [límites de configuración de VMware HCX](https://configmax.vmware.com/guest?vmwareproduct=VMware%20HCX&release=VMware%20HCX&categories=41-0,42-0,43-0,44-0,45-0).
   
   :::image type="content" source="media/tutorial-vmware-hcx/extend-networks-increase-vlan.png" alt-text="Captura de pantalla que muestra dónde hay que aumentar el número de VLAN." lightbox="media/tutorial-vmware-hcx/extend-networks-increase-vlan.png":::

1. En **Advanced Configuration - Traffic Engineering** (Configuración avanzada: ingeniería de tráfico), revise y realice las modificaciones que considere necesarias y, a continuación, seleccione **Continue** (Continuar).

1. Revise la vista previa de la topología y seleccione **Continue** (Continuar).

1. A continuación, escriba un nombre descriptivo para esta malla de servicios y seleccione **Finalizar** para terminar el proceso.  

1. Seleccione **Ver tareas** para supervisar la implementación. 

   :::image type="content" source="media/tutorial-vmware-hcx/monitor-service-mesh.png" alt-text="Captura de pantalla que muestra el botón para ver las tareas.":::

   Una vez que la implementación de la malla de servicios finalice correctamente, verá los servicios en color verde.

   :::image type="content" source="media/tutorial-vmware-hcx/service-mesh-green.png" alt-text="Captura de pantalla que muestra indicadores verdes en los servicios." lightbox="media/tutorial-vmware-hcx/service-mesh-green.png":::

1. Para comprobar el estado de la malla de servicios, compruebe el estado del dispositivo. 

1. Seleccione **Interconnect** > **Appliances** (Interconectar > Dispositivos).

   :::image type="content" source="media/tutorial-vmware-hcx/interconnect-appliance-state.png" alt-text="Captura de pantalla que muestra las selecciones para comprobar el estado del dispositivo." lightbox="media/tutorial-vmware-hcx/interconnect-appliance-state.png":::

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Malla de servicios](https://www.youtube.com/embed/COY3oIws108).

### <a name="optional-create-a-network-extension"></a>(Opcional) Creación de una extensión de red

Si quiere extender las redes de su entorno local a Azure VMware Solution, siga estos pasos:

1. En **Servicios**, seleccione **Extensión de red** > **Create a Network Extension** (Crear una extensión de red).

   :::image type="content" source="media/tutorial-vmware-hcx/create-network-extension.png" alt-text="Captura de pantalla que muestra las selecciones para empezar a crear una extensión de red." lightbox="media/tutorial-vmware-hcx/create-network-extension.png":::

1. Seleccione cada una de las redes que quiere extender a Azure VMware Solution y, a continuación, seleccione **Next** (Siguiente).

   :::image type="content" source="media/tutorial-vmware-hcx/select-extend-networks.png" alt-text="Captura de pantalla que muestra la selección de una red.":::

1. Escriba la dirección IP de puerta de enlace local para cada una de las redes que va a extender y, a continuación, seleccione **Enviar**. 

   :::image type="content" source="media/tutorial-vmware-hcx/extend-networks-gateway.png" alt-text="Captura de pantalla que muestra la entrada de una dirección IP de puerta de enlace.":::

   La extensión de red tardará unos minutos en finalizar. Cuando lo haga, verá que el estado cambia a **Extension complete** (Extensión completada).

   :::image type="content" source="media/tutorial-vmware-hcx/extension-complete.png" alt-text="Captura de pantalla que muestra el estado de extensión completada." lightbox="media/tutorial-vmware-hcx/extension-complete.png":::

Puede encontrar información de un extremo a otro de este procedimiento en el vídeo [Azure VMware Solution: Extensión de red](https://www.youtube.com/embed/gYR0nftKui0).


## <a name="next-steps"></a>Pasos siguientes

Si el estado del túnel de interconexión de PCX es **ACTIVO** y está en color verde, está listo para migrar y proteger las máquinas virtuales de Azure VMware Solution mediante VMware HCX. Azure VMware Solution admite migraciones de cargas de trabajo (con o sin una extensión de red). Todavía puede migrar cargas de trabajo en el entorno de vSphere, junto con la creación local de redes y la implementación de máquinas virtuales en esas redes.  

Para más información sobre el uso de HCX, consulte la documentación técnica de VMware:

* [Documentación de VMware HCX](https://docs.vmware.com/en/VMware-HCX/index.html)
* [Migración de máquinas virtuales con VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g)
* [Puertos necesarios de HCX](https://ports.vmware.com/home/VMware-HCX)
* [Configuración de un servidor proxy HCX antes de aprobar la clave de licencia](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-920242B3-71A3-4B24-9ACF-B20345244AB2.html?hWord=N4IghgNiBcIA4CcD2APAngAgBIGEAaIAvkA)
