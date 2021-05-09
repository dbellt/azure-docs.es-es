---
title: 'Azure VMware Solution by CloudSimple: establecimiento de una nube privada como sitio de recuperación ante desastres mediante VMware Site Recovery Manager'
description: Se describe cómo configurar la nube privada CloudSimple como sitio de recuperación ante desastres mediante VMware Site Recovery Manager.
author: shortpatti
ms.author: v-patsho
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 38e10de5644655c52b1a1a4399d5180eb8b5861d
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176925"
---
# <a name="set-up-private-cloud-as-a-disaster-recovery-target-with-vmware-site-recovery-manager"></a>Configuración de una nube privada como destino de recuperación ante desastres con VMware Site Recovery Manager

Puede usar la nube privada de CloudSimple como un sitio de recuperación ante desastres (DR) para las cargas de trabajo locales de VMware.

La solución de recuperación ante desastres se basa en vSphere Replication y VMware Site Recovery Manager (SRM). Se puede seguir un enfoque similar para habilitar la nube privada como sitio primario protegido por el sitio de recuperación local.

La solución de CloudSimple:

* Elimina la necesidad de configurar un centro de datos específico para la recuperación ante desastres.
* Permite aprovechar las ubicaciones de Azure en que se implementa CloudSimple para lograr resistencia geográfica internacional.
* Ofrece una opción para reducir los costos de implementación y el costo total de propiedad a fin de establecer la recuperación ante desastres.

La solución de CloudSimple requiere que haga lo siguiente:

* Instale, configure y administre vSphere Replication y SRM en su nube privada.
* Proporcione sus propias licencias para SRM cuando la nube privada sea el sitio protegido. No necesita ninguna licencia de SRM adicional para el sitio de CloudSimple cuando se usa como sitio de recuperación.

Con esta solución, tiene control total sobre vSphere Replication y SRM. La UI y las interfaces conocidas de la API y la CLI permiten el uso de los scripts y las herramientas existentes.

![Implementación del Site Recovery Manager](media/srm-deployment.png)

Puede usar cualquier versión de vRA y SRM que sea compatible con la nube privada y los entornos locales. En los ejemplos de esta guía se usan vRA 6.5 y SRM 6.5. Estas versiones son compatibles con vSphere 6.5, cuya versión también es compatible con CloudSimple.

## <a name="deploy-the-solution"></a>Implementación de la solución

En las secciones siguientes se describe cómo implementar una solución de recuperación ante desastres con SRM en la nube privada.

1. [Verificar que las versiones de los productos de VMware son compatibles](#verify-that-vmware-product-versions-are-compatible)
2. [Calcular el tamaño del entorno de recuperación ante desastres](#estimate-the-size-of-your-dr-environment)
3. [Crear una nube privada para su entorno](#create-a-private-cloud-for-your-environment)
4. [Configurar redes de la nube privada para la solución de SRM](#set-up-private-cloud-networking-for-the-srm-solution)
5. [Configurar una conexión VPN de sitio a sitio entre la red local y la nube privada y abrir los puertos necesarios](#set-up-a-site-to-site-vpn-connection-between-your-on-premises-network-and-the-private-cloud-and-open-required-ports)
6. [Configurar los servicios de infraestructura en la nube privada](#set-up-infrastructure-services-in-your-private-cloud)
7. [Instalar vSphere Replication Appliance en el entorno local](#install-vsphere-replication-appliance-in-your-on-premises-environment)
8. [Instalar vSphere Replication Appliance en el entorno de la nube privada](#install-vsphere-replication-appliance-in-your-private-cloud-environment)
9. [Instalar el servidor de SRM en el entorno local](#install-srm-server-in-your-on-premises-environment)
10. [Instalar el servidor de SRM en la nube privada](#install-srm-server-in-your-private-cloud)

### <a name="verify-that-vmware-product-versions-are-compatible"></a>Verificar que las versiones de los productos de VMware son compatibles

Las configuraciones de esta guía están sujetas a los siguientes requisitos de compatibilidad:

* Debe implementarse la misma versión de SRM en la nube privada y en el entorno local.
* Debe implementarse la misma versión de vSphere Replication en la nube privada y en el entorno local.
* Las versiones de Platform Services Controller (PSC) de la nube privada y el entorno local deben ser compatibles.
* Las versiones de vCenter en la nube privada y el entorno local deben ser compatibles.
* Las versiones de SRM y vSphere Replication deben ser compatibles entre sí y con las versiones de PSC y vCenter.

Para obtener vínculos a la documentación de VMware correspondiente e información sobre compatibilidad, vaya a la documentación de [VMware Site Recovery Manager](https://docs.vmware.com/en/Site-Recovery-Manager/index.html).

Para averiguar las versiones de vCenter y PSC de su nube privada, abra el portal de CloudSimple. Vaya a **Recursos**, seleccione la nube privada y haga clic en la pestaña **Red de administración de vSphere**.

![Versiones de vCenter y PSC en la nube privada](media/srm-resources.png)

### <a name="estimate-the-size-of-your-dr-environment"></a>Calcular el tamaño del entorno de recuperación ante desastres

1. Verifique que la configuración local identificada se encuentre dentro de los límites admitidos. En el caso de SRM 6.5, los límites se documentan en el artículo de Knowledge Base de VMware sobre los [límites operativos de Site Recovery Manager 6.5](https://kb.vmware.com/s/article/2147110).
2. Asegúrese de que tiene suficiente ancho de banda de red para satisfacer los requisitos de tamaño de la carga de trabajo y RPO. En el artículo de Knowledge Base de VMware sobre cómo [calcular los requisitos de ancho de banda para vSphere Replication](https://docs.vmware.com/en/vSphere-Replication/6.5/com.vmware.vsphere.replication-admin.doc/GUID-4A34D0C9-8CC1-46C4-96FF-3BF7583D3C4F.html) se proporcionan instrucciones acerca de los límites del ancho de banda.
3. Use la herramienta CloudSimple Sizer para calcular los recursos que se necesitan en el sitio de recuperación ante desastres para proteger su entorno local.

### <a name="create-a-private-cloud-for-your-environment"></a>Crear una nube privada para su entorno

Cree una nube privada desde el portal de CloudSimple siguiendo las instrucciones y las recomendaciones de ajuste de tamaño que se proporcionan en el artículo [Crear una nube privada](create-private-cloud.md).

### <a name="set-up-private-cloud-networking-for-the-srm-solution"></a>Configurar redes de la nube privada para la solución de SRM

Acceda al portal de CloudSimple para configurar las redes de la nube privada para la solución de SRM.

Cree una VLAN para la red de la solución de SRM y asígnele un valor de CIDR de subred. Para obtener instrucciones, consulte el artículo [Creación y administración de subredes y VLAN.](create-vlan-subnet.md)

### <a name="set-up-a-site-to-site-vpn-connection-between-your-on-premises-network-and-the-private-cloud-and-open-required-ports"></a>Configurar una conexión VPN de sitio a sitio entre la red local y la nube privada y abrir los puertos necesarios

Puede configurar una conectividad de sitio a sitio entre la red local y la nube privada. Para obtener instrucciones, consulte [Configuración de una conexión VPN en la nube privada de CloudSimple](set-up-vpn.md).

### <a name="set-up-infrastructure-services-in-your-private-cloud"></a>Configuración de los servicios de infraestructura en la nube privada

Configure los servicios de infraestructura en la nube privada para facilitar la administración de las cargas de trabajo y las herramientas.

Puede agregar un proveedor de identidades externo tal y como se describe en el artículo [Uso de Azure AD como proveedor de identidades para vCenter en la nube privada de CloudSimple](azure-ad.md) si quiere realizar alguna de las acciones siguientes:

* Identificar a los usuarios de su instancia local de Active Directory (AD) en la nube privada.
* Configurar una instancia de AD en la nube privada para todos los usuarios.
* Usar Azure AD.

Para proporcionar servicios de resolución de nombres, administración de direcciones IP y búsqueda de direcciones IP para las cargas de trabajo de la nube privada, configure un servidor DHCP y DNS tal y como se describe en el artículo [Configuración de aplicaciones y cargas de trabajo de DNS y DHCP en la nube privada de CloudSimple](dns-dhcp-setup.md).

Los hosts y las máquinas virtuales de administración usan el dominio *.cloudsimple.io en la nube privada. Para resolver las solicitudes a este dominio, configure el reenvío de DNS en el servidor DNS tal y como se describe en el artículo [Creación de un reenviador condicional](on-premises-dns-setup.md#create-a-conditional-forwarder).

### <a name="install-vsphere-replication-appliance-in-your-on-premises-environment"></a>Instalar vSphere Replication Appliance en el entorno local

Para instalar vSphere Replication Appliance (vRA) en el entorno local, siga la documentación de VMware. Para realizar la instalación, debe realizar estos pasos de alto nivel:

1. Prepare los servidores locales para la instalación de vRA.

2. Implemente vRA en su entorno local mediante el uso de OVF en el archivo ISO de VR desde vmware.com. En el caso de vRA 6.5, puede encontrar información pertinente en [este blog de VMware](https://blogs.vmware.com/virtualblocks/2017/01/20/vr-65-ovf-choices).

3. Registre su vRA local con el inicio de sesión único de vCenter en el sitio local. Para obtener instrucciones detalladas sobre la replicación de vSphere 6.5, consulte el documento [Instalación y configuración de VMware vSphere Replication 6.5](https://docs.vmware.com/en/vSphere-Replication/6.5/vsphere-replication-65-install.pdf) de VMware.

## <a name="install-vsphere-replication-appliance-in-your-private-cloud-environment"></a>Instalar vSphere Replication Appliance en el entorno de la nube privada

Antes de comenzar, verifique que dispone de lo siguiente:

* Disponibilidad de IP desde las subredes del entorno local a la subred de administración de la nube privada
* Disponibilidad de IP desde la subred de replicación del entorno local de vSphere a la subred de la solución de SRM de la nube privada

Para obtener instrucciones, consulte [Configuración de una conexión VPN en la nube privada de CloudSimple](set-up-vpn.md). Los pasos son similares a los de la instalación local.

CloudSimple recomienda el uso de FQDN en lugar de direcciones IP durante la instalación de vRA y SRM. Para averiguar el FQDN de vCenter y PSC de su nube privada, abra el portal de CloudSimple. Vaya a **Recursos**, seleccione la nube privada y haga clic en la pestaña **Red de administración de vSphere**.

![Búsqueda del FQDN de vCenter y PSC en la nube privada](media/srm-resources.png)

CloudSimple requiere que no instale vRA y SRM con el usuario "cloudowner" predeterminado, sino que, en su lugar, cree un nuevo usuario. Esto permite garantizar un tiempo de actividad y una disponibilidad elevados para el entorno de vCenter de la nube privada. Sin embargo, el usuario "cloudowner" predeterminado de la instancia de vCenter de la nube privada no tiene privilegios suficientes para crear un nuevo usuario con privilegios administrativos.

Antes de instalar vRA y SRM, debe escalar los privilegios de vCenter del usuario cloudowner y, a continuación, crear un usuario con privilegios administrativos en el dominio de SSO de vCenter. Para obtener detalles sobre el modelo predeterminado de permisos y usuarios de la nube privada, consulte [Información sobre el modelo de permisos de la nube privada](learn-private-cloud-permissions.md).

Para realizar la instalación, debe realizar estos pasos de alto nivel:

1. [Escale privilegios](escalate-private-cloud-privileges.md).
2. Cree un usuario en la nube privada para la instalación de SRM y vSphere Replication. Se explica más abajo en la sección [Interfaz de usuario de vCenter: crear un usuario en la nube privada para la instalación de vRA y SRM](#vcenter-ui-create-a-user-in-private-cloud-for-vra-and-srm-installation).
3. Prepare los servidores de la nube privada para la instalación de vRA.
4. Implemente vRA en su nube privada mediante el uso de OVF en el archivo ISO de VR desde vmware.com. En el caso de vRA 6.5, puede encontrar información pertinente en [este blog de VMware](https://blogs.vmware.com/virtualblocks/2017/01/20/vr-65-ovf-choices).
5. Configure las reglas del firewall para vRA. Se explica más abajo en la sección [Portal de CloudSimple: configurar las reglas de firewall para vRA](#cloudsimple-portal-configure-firewall-rules-for-vra).
6. Registre la instancia de vRA de la nube privada con el inicio de sesión único de vCenter en el sitio de la nube privada.
7. Configure las conexiones de vSphere Replication entre los dos dispositivos. Asegúrese de que los puertos necesarios están abiertos en los firewalls. Consulte [este artículo de Knowledge Base de VMware](https://kb.vmware.com/s/article/2087769) para obtener una lista de números de puerto que deben estar abiertos para vSphere Replication 6.5.

Para obtener instrucciones detalladas sobre la instalación de vSphere Replication 6.5, consulte el documento [Instalación y configuración de VMware vSphere Replication 6.5](https://docs.vmware.com/en/vSphere-Replication/6.5/vsphere-replication-65-install.pdf) de VMware.

#### <a name="vcenter-ui-create-a-user-in-private-cloud-for-vra-and-srm-installation"></a>Interfaz de usuario de vCenter: crear un usuario en la nube privada para la instalación de vRA y SRM

Inicie sesión en vCenter con las credenciales del usuario cloudowner después de escalar los privilegios desde el portal de CloudSimple.

Cree un nuevo usuario, `srm-soln-admin`, en vCenter y agréguelo al grupo de administradores de vCenter.
Cierre sesión de vCenter con el usuario cloudowner e iníciela con el usuario *srm-soln-admin*.

#### <a name="cloudsimple-portal-configure-firewall-rules-for-vra"></a>Portal de CloudSimple: Configuración de las reglas de firewall para vRA

Configure reglas de firewall tal y como se describe en [Configuración de tablas y reglas de firewall](firewall.md) para abrir puertos para habilitar la comunicación entre:

* vRA en la red de la solución de SRM y los hosts de vCenter y ESXi de la red de administración.
* Dispositivos de vRA en los dos sitios.

Consulte este [artículo de Knowledge Base de VMware](https://kb.vmware.com/s/article/2087769) para obtener una lista de números de puerto que deben estar abiertos para vSphere Replication 6.5.

### <a name="install-srm-server-in-your-on-premises-environment"></a>Instalar el servidor de SRM en el entorno local

Antes de empezar, verifique lo siguiente:

* vSphere Replication Appliance está instalado en los entornos de nube privada y local.
* Las aplicaciones de vSphere Replication Appliance de ambos sitios están conectadas entre sí.
* Ha revisado la información de VMware sobre los requisitos previos y los procedimientos recomendados. En el caso de SRM 6.5, puede consultar el documento [Requisitos previos y prácticas recomendadas para SRM 6.5](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-BB0C03E4-72BE-4C74-96C3-97AC6911B6B8.html) de VMware.

Siga la documentación de VMware para realizar la instalación del servidor de SRM en el modelo de implementación "topología de dos sitios con una instancia de vCenter por Platform Services Controller" como se describe en este [documento de VMware](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-F474543A-88C5-4030-BB86-F7CC51DADE22.html). Las instrucciones de instalación de SRM 6.5 están disponibles en el documento [Instalar Site Recovery Manager](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html) de VMware.

### <a name="install-srm-server-in-your-private-cloud"></a>Instalar el servidor de SRM en la nube privada

Antes de empezar, verifique lo siguiente:

* vSphere Replication Appliance está instalado en los entornos de nube privada y local.
* Las aplicaciones de vSphere Replication Appliance de ambos sitios están conectadas entre sí.
* Ha revisado la información de VMware sobre los requisitos previos y los procedimientos recomendados. En el caso de SRM 6.5, puede consultar el artículo [Requisitos previos y prácticas recomendadas para instalar Site Recovery Manager 6.5 Server](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-BB0C03E4-72BE-4C74-96C3-97AC6911B6B8.html).

En los pasos siguientes se describe la instalación de SRM de la nube privada.

1. [Interfaz de usuario de vCenter: instalar SSMS](#vcenter-ui-install-srm)
2. [Portal de CloudSimple: configurar las reglas de firewall para SRM](#cloudsimple-portal-configure-firewall-rules-for-srm)
3. [Interfaz de usuario de vCenter: configurar SRM](#vcenter-ui-configure-srm)
4. [Portal de CloudSimple: anular el escalado de privilegios](#cloudsimple-portal-de-escalate-privileges)

#### <a name="vcenter-ui-install-srm"></a>Interfaz de usuario de vCenter: instalar SSMS

Después de iniciar sesión en vCenter con las credenciales srm-soln-admin, siga la documentación de VMware para realizar la instalación del servidor de SRM en el modelo de implementación "topología de dos sitios con una instancia de vCenter por Platform Services Controller" como se describe en este [documento de VMware](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-F474543A-88C5-4030-BB86-F7CC51DADE22.html). Las instrucciones de instalación de SRM 6.5 están disponibles en el documento [Instalar Site Recovery Manager](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html) de VMware.

#### <a name="cloudsimple-portal-configure-firewall-rules-for-srm"></a>Portal de CloudSimple: configurar las reglas de firewall para SRM

Configure reglas de firewall tal y como se describe en [Configuración de tablas y reglas de firewall](firewall.md) para permitir la comunicación entre:

El servidor de SRM y vCenter o PSC en la nube privada.
Los servidores de SRM en ambos sitios

Consulte [este artículo de Knowledge Base de VMware](https://kb.vmware.com/s/article/2087769) para obtener una lista de números de puerto que deben estar abiertos para vSphere Replication 6.5.

#### <a name="vcenter-ui-configure-srm"></a>Interfaz de usuario de vCenter: configurar SRM

Después de instalar SRM en la nube privada, realice las siguientes tareas, tal como se describe en las secciones de la guía de instalación y configuración de VMware Site Recovery Manager. En el caso de SRM 6.5, las instrucciones están disponibles en el documento [Instalar Site Recovery Manager](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html) de VMware.

1. Conecte las instancias de Site Recovery Manager Server en los sitios protegidos y de recuperación.
2. Establezca una conexión de cliente con la instancia remota de Site Recovery Manager Server.
3. Instale la clave de licencia de Site Recovery Manager.

#### <a name="cloudsimple-portal-de-escalate-privileges"></a>Portal de CloudSimple: Anular el escalado de privilegios

Para anular el escalado de privilegios, consulte [Anular el escalado de privilegios](escalate-private-cloud-privileges.md#de-escalate-privileges).

## <a name="ongoing-management-of-your-srm-solution"></a>Administración continua de la solución de SRM

Tiene control total sobre el software de SRM y vSphere Replication en su entorno de nube privada y se espera que realice la administración del ciclo de vida del software necesaria. Asegúrese de que cualquier versión nueva del software sea compatible con PSC y la instancia de vCenter de la nube privada antes de actualizar vSphere Replication o SRM.

> [!NOTE]
> Actualmente, CloudSimple está explorando opciones para ofrecer un servicio de recuperación ante desastres administrado. 

## <a name="multiple-replication-configuration"></a>Configuración de varias replicaciones

 [Tanto la replicación basada en matrices como las tecnologías de replicación de vSphere se pueden usar junto con SRM](https://blogs.vmware.com/virtualblocks/2017/06/22/srm-array-based-replication-vs-vsphere-replication) al mismo tiempo. Sin embargo, se deben aplicar a un conjunto independiente de máquinas virtuales (una máquina virtual determinada puede protegerse mediante la replicación basada en matrices o la replicación de vSphere, pero no ambas). Además, el sitio de CloudSimple se puede configurar como un sitio de recuperación para varios sitios protegidos. Consulte [Opciones de varios sitios de SRM](https://blogs.vmware.com/virtualblocks/2016/07/28/srm-multisite/) para obtener información sobre las configuraciones de varios sitios.

## <a name="references"></a>Referencias

* [Documentación de VMware Site Recovery Manager](https://docs.vmware.com/en/Site-Recovery-Manager/index.html)
* [Límites operativos para Site Recovery Manager 6.5](https://kb.vmware.com/s/article/2147110)
* [Cálculo de los requisitos de ancho de banda para vSphere Replication](https://docs.vmware.com/en/vSphere-Replication/6.5/com.vmware.vsphere.replication-admin.doc/GUID-4A34D0C9-8CC1-46C4-96FF-3BF7583D3C4F.html)
* [Opciones de OVF al implementar vSphere Replication 6.5](https://blogs.vmware.com/virtualblocks/2017/01/20/vr-65-ovf-choices/)
* [Instalación y configuración de VMware vSphere Replication 6.5](https://docs.vmware.com/en/vSphere-Replication/6.5/vsphere-replication-65-install.pdf)
* [Requisitos previos y prácticas recomendadas para SRM 6.5](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-BB0C03E4-72BE-4C74-96C3-97AC6911B6B8.html)
* [Site Recovery Manager en una topología de dos sitios con una instancia de vCenter Server por Platform Services Controller](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-F474543A-88C5-4030-BB86-F7CC51DADE22.html)
* [Guía de instalación y configuración de VMware Site Recovery Manager 6.5](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html)
* [Blog de VMware sobre SRM con la replicación basada en matrices frente a la replicación de vSphere](https://blogs.vmware.com/virtualblocks/2017/06/22/srm-array-based-replication-vs-vsphere-replication)
* [Blog de VMware sobre opciones de varios sitios de SRM](https://blogs.vmware.com/virtualblocks/2016/07/28/srm-multisite)
* [Números de puerto que deben estar abiertos para vSphere Replication 5.8.x, 6.x y 8](https://kb.vmware.com/s/article/2147112)
