---
title: Arquitectura de conectividad
titleSuffix: Azure SQL Managed Instance
description: Obtenga información sobre la arquitectura de comunicación y conectividad de Azure SQL Managed Instance y sobre la forma en que los componentes dirigen el tráfico a una instancia administrada.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: fasttrack-edit
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 04/24/2021
ms.openlocfilehash: 99ec792937a85122fc3e6886309f1f9b24eecafc
ms.sourcegitcommit: aaba99b8b1c545ad5d19f400bcc2d30d59c63f39
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108007527"
---
# <a name="connectivity-architecture-for-azure-sql-managed-instance"></a>Arquitectura de conectividad de Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este artículo se explica la comunicación en Azure SQL Managed Instance. También se describe la arquitectura de conectividad y cómo los componentes dirigen el tráfico a una instancia administrada.  

SQL Managed Instance se coloca dentro de la red virtual de Azure y de la subred dedicada a las instancias administradas. Esta implementación proporciona:

- Una dirección IP privada segura.
- La posibilidad de conectar una red local a SQL Managed Instance.
- La posibilidad de conectar SQL Managed Instance a un servidor vinculado o a otro almacén de datos local.
- La posibilidad de conectar SQL Managed Instance a recursos de Azure.

## <a name="communication-overview"></a>Información general sobre la comunicación

En el siguiente diagrama se muestran las entidades que se conectan a SQL Managed Instance. También se muestran los recursos que deben comunicarse con una instancia administrada. El proceso de comunicación de la parte inferior del diagrama representa las aplicaciones y las herramientas del cliente que se conectan a SQL Managed Instance como orígenes de datos.  

![Entidades de la arquitectura de conectividad](./media/connectivity-architecture-overview/connectivityarch001.png)

SQL Managed Instance es una oferta de plataforma como servicio (PaaS). Azure emplea agentes automatizados (administración, implementación y mantenimiento) para administrar este servicio en función de flujos de datos de telemetría. Dado que Azure es responsable de la administración, los clientes no pueden acceder a las máquinas del clúster virtual de Instancia administrada de SQL mediante el Protocolo de escritorio remoto (RDP).

Algunas operaciones iniciadas por los usuarios finales o las aplicaciones podrían requerir que SQL Managed Instance interactuara con la plataforma. Un caso es la creación de una base de datos de Instancia administrada de SQL. Este recurso se expone mediante Azure Portal, PowerShell, la CLI de Azure y la API REST.

SQL Managed Instance depende de servicios de Azure como Azure Storage para las copias de seguridad, Azure Event Hubs para la telemetría, Azure Active Directory (Azure AD) para la autenticación, Azure Key Vault para el Cifrado de datos transparente (TDE) y un par de servicios de plataforma de Azure que proporcionan características de seguridad y compatibilidad. SQL Managed Instance crea conexiones a estos servicios.

Todas las comunicaciones se cifran y firman mediante certificados. Para comprobar la confiabilidad de las partes en comunicación, SQL Managed Instance comprueba constantemente estos certificados por medio de listas de revocación de certificados. Si los certificados se revocan, SQL Managed Instance cierra la conexión para proteger los datos.

## <a name="high-level-connectivity-architecture"></a>Arquitectura de conectividad de alto nivel

En general, SQL Managed Instance es un conjunto de componentes del servicio. Estos componentes se hospedan en un conjunto dedicado de máquinas virtuales aisladas que se ejecutan dentro de la subred de red virtual del cliente. Estas máquinas forman un clúster virtual.

Un clúster virtual puede hospedar varias instancias administradas. El clúster se expande y contrae automáticamente si es necesario cuando el cliente cambia el número de instancias aprovisionadas en la subred.

Las aplicaciones de los clientes pueden conectarse a SQL Managed Instance y pueden consultar y actualizar las bases de datos dentro de la red virtual, la red virtual emparejada o la red conectada mediante VPN o Azure ExpressRoute. Esta red debe usar un punto de conexión y una dirección IP privada.  

![Diagrama de la arquitectura de conectividad](./media/connectivity-architecture-overview/connectivityarch002.png)

Los servicios de implementación y administración de Azure se ejecutan fuera de la red virtual. SQL Managed Instance y los servicios de Azure se conectan a través de los puntos de conexión que tienen direcciones IP públicas. Cuando SQL Managed Instance crea una conexión de salida, en el extremo receptor parece que procede de esta dirección IP pública debido a la traducción de direcciones de red (NAT).

El tráfico de administración fluye a través de la red virtual del cliente. Esto significa que los elementos de la infraestructura de la red virtual pueden dañar el tráfico de administración al provocar que la instancia genere un error y deje de estar disponible.

> [!IMPORTANT]
> Para mejorar la experiencia del cliente y la disponibilidad del servicio, Azure aplica una directiva de intención de red a los elementos de la infraestructura de la red virtual de Azure. La directiva puede afectar al modo en que funciona SQL Managed Instance. Este mecanismo de plataforma comunica de forma transparente los requisitos de red a los usuarios. El objetivo principal de la directiva es evitar errores de configuración de red y garantizar el funcionamiento normal de Instancia administrada de SQL. Cuando se elimina una instancia administrada, también se quita la directiva de intención de red.

## <a name="virtual-cluster-connectivity-architecture"></a>Arquitectura de conectividad del clúster virtual

Vamos a profundizar más en la arquitectura de conectividad de SQL Managed Instance. En el siguiente diagrama se muestra el diseño conceptual del clúster virtual.

![Arquitectura de conectividad del clúster virtual](./media/connectivity-architecture-overview/connectivityarch003.png)

Los clientes se conectan a SQL Managed Instance mediante un nombre de host que tiene el formato `<mi_name>.<dns_zone>.database.windows.net`. Este nombre de host se resuelve en una dirección IP privada, aunque se registra en la zona DNS (Sistema de nombres de dominio) pública y puede resolverse públicamente. `zone-id` se genera automáticamente al crear el clúster. Si un clúster recién creado hospeda una instancia administrada secundaria, comparte su identificador de zona con el clúster principal. Para más información, consulte [Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada](../database/auto-failover-group-overview.md#enabling-geo-replication-between-managed-instances-and-their-vnets).

Esta dirección IP privada pertenece al equilibrador de carga interno de SQL Managed Instance. El equilibrador de carga dirige el tráfico a la puerta de enlace de SQL Managed Instance. Como se pueden ejecutar varias instancias administradas dentro del mismo clúster, la puerta de enlace usa el nombre de host de SQL Managed Instance para dirigir el tráfico al servicio correcto del motor de SQL.

Los servicios de administración e implementación se conectan a SQL Managed Instance mediante un [punto de conexión de administración](#management-endpoint) que se asigna a un equilibrador de carga externo. El tráfico se enruta a los nodos únicamente si se recibe en un conjunto predefinido de puertos que solo usan los componentes de administración de SQL Managed Instance. Un firewall integrado en los nodos se configura para permitir el tráfico solo desde intervalos IP de Microsoft. Los certificados autentican mutuamente toda la comunicación entre los componentes de administración y el plano de administración.

## <a name="management-endpoint"></a>Administración de puntos de conexión

Azure administra SQL Managed Instance mediante un punto de conexión de administración. Este punto de conexión está dentro del clúster virtual de una instancia. El punto de conexión de administración está protegido por un firewall integrado en el nivel de red. En el nivel de aplicación, está protegido por la comprobación mutua del certificado. Para buscar la dirección IP del punto de conexión, consulte [Determinación de la dirección IP del punto de conexión de administración](management-endpoint-find-ip-address.md).

Cuando las conexiones se inician dentro de SQL Managed Instance (como en las copias de seguridad y los registros de auditoría), el tráfico parece iniciarse desde la dirección IP pública del punto de conexión de administración. Se puede limitar el acceso a los servicios públicos de SQL Managed Instance mediante el establecimiento de reglas de firewall que permitan únicamente la dirección IP de SQL Managed Instance. Para obtener más información, consulte [Comprobación del firewall integrado de SQL Managed Instance](management-endpoint-verify-built-in-firewall.md).

> [!NOTE]
> El tráfico que se dirige a servicios de Azure que están dentro de la región de SQL Managed Instance se optimiza y, por ese motivo, no se dirige mediante NAT a la dirección IP pública del punto de conexión de administración. Así que, si necesita usar reglas de firewall basadas en IP, normalmente para el almacenamiento, el servicio debe estar en una región distinta de la de SQL Managed Instance.

## <a name="service-aided-subnet-configuration"></a>Configuración de subred asistida por servicio

Para cumplir los requisitos de manejabilidad y seguridad de los clientes, SQL Managed Instance cambia de una configuración de subred manual a una asistida por servicio.

Con la configuración de subred asistida por servicio, el usuario tiene control total sobre el tráfico de datos (TDS) a la vez que SQL Managed Instance asume la responsabilidad de garantizar el flujo ininterrumpido de tráfico de administración para cumplir el Acuerdo de Nivel de Servicio.

La configuración de la subred asistida por servicio se basa en la característica de [delegación de subred](../../virtual-network/subnet-delegation-overview.md) de la red virtual para proporcionar administración automática de la configuración de red y habilitar puntos de conexión de servicio. 

Los puntos de conexión de servicio podrían usarse para configurar reglas de firewall de red virtual en cuentas de almacenamiento que conservan copias de seguridad y registros de auditoría. Incluso si tienen habilitados los puntos de conexión del servicio, se recomienda que los clientes utilicen el [vínculo privado](../../private-link/private-link-overview.md) que añade más seguridad a los puntos de conexión del servicio.

> [!IMPORTANT]
> Debido a las características de configuración del plano de control, la configuración de subred asistida por servicio no habilitaría los puntos de conexión del servicio en nubes nacionales. 

### <a name="network-requirements"></a>Requisitos de red

Implemente SQL Managed Instance en una subred dedicada dentro de la red virtual. La subred debe tener estas características:

- **Subred dedicada:** la subred de SQL Managed Instance no puede contener ningún otro servicio en la nube asociado a ella ni puede ser una subred de puerta de enlace. La subred no puede contener ningún recurso, a excepción de SQL Managed Instance, ni se le pueden agregar posteriormente otros tipos de recursos.
- **Delegación de subred:** la subred de SQL Managed Instance debe delegarse en el proveedor de recursos `Microsoft.Sql/managedInstances`.
- **Grupo de seguridad de red (NSG):** se debe asociar un grupo de seguridad de red a la subred de SQL Managed Instance. Puede usar un NSG para controlar el acceso al punto de conexión de datos de SQL Managed Instance mediante el filtrado del tráfico en el puerto 1433 y en los puertos 11000 a 11999 cuando SQL Managed Instance se configura para conexiones de redirección. El servicio aprovisionará y mantendrá automáticamente las [reglas](#mandatory-inbound-security-rules-with-service-aided-subnet-configuration) actuales necesarias para permitir el flujo ininterrumpido de tráfico de administración.
- **Tabla de rutas definida por el usuario (UDR):** se debe asociar una tabla UDR a la subred de SQL Managed Instance. Puede agregar entradas a la tabla de rutas para enrutar el tráfico que tiene intervalos IP privados locales como destino a través de una puerta de enlace de red virtual o de un dispositivo de red virtual (NVA). El servicio aprovisionará y mantendrá automáticamente las [entradas](#mandatory-user-defined-routes-with-service-aided-subnet-configuration) actuales necesarias para permitir el flujo ininterrumpido de tráfico de administración.
- **Suficientes direcciones IP:** la subred de SQL Managed Instance debe tener al menos 32 direcciones IP. Para obtener más información, consulte [Determinación del tamaño de la subred de SQL Managed Instance](vnet-subnet-determine-size.md). Puede implementar instancias administradas en [la red existente](vnet-existing-add-subnet.md) después de configurarla para satisfacer [los requisitos de red de SQL Managed Instance](#network-requirements). De lo contrario, cree [una red y una subred](virtual-network-subnet-create-arm-template.md).

> [!IMPORTANT]
> Cuando se crea una instancia administrada, se aplica una directiva de intención de red en la subred para evitar cambios no compatibles con la configuración de red. Después de quitar la última instancia de la subred, también se quitará la directiva de intención de red. Las reglas siguientes son solo para fines informativos y no se deben implementar mediante la plantilla de ARM/PowerShell/CLI. Si quiere usar la plantilla oficial más reciente, siempre puede [recuperarla del portal](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

### <a name="mandatory-inbound-security-rules-with-service-aided-subnet-configuration"></a>Reglas de seguridad de entrada obligatorias con la configuración de subred asistida por servicio
Estas reglas son necesarias para garantizar el flujo de tráfico de administración de entrada. Consulte el [párrafo anterior](#high-level-connectivity-architecture) para más información sobre la arquitectura de conectividad y el tráfico de administración.

| Nombre       |Port                        |Protocolo|Source           |Destination|Acción|
|------------|----------------------------|--------|-----------------|-----------|------|
|management  |9000, 9003, 1438, 1440, 1452|TCP     |SqlManagement    |MI SUBNET  |Allow |
|            |9000, 9003                  |TCP     |CorpnetSaw       |MI SUBNET  |Allow |
|            |9000, 9003                  |TCP     |CorpnetPublic    |MI SUBNET  |Allow |
|mi_subnet   |Any                         |Any     |MI SUBNET        |MI SUBNET  |Allow |
|health_probe|Any                         |Any     |AzureLoadBalancer|MI SUBNET  |Allow |

### <a name="mandatory-outbound-security-rules-with-service-aided-subnet-configuration"></a>Reglas de seguridad de entrada obligatorias con la configuración de subred asistida por servicio
Estas reglas son necesarias para garantizar el flujo del tráfico de administración de salida. Consulte el [párrafo anterior](#high-level-connectivity-architecture) para más información sobre la arquitectura de conectividad y el tráfico de administración.

| Nombre       |Port          |Protocolo|Source           |Destination|Acción|
|------------|--------------|--------|-----------------|-----------|------|
|management  |443, 12000    |TCP     |MI SUBNET        |AzureCloud |Allow |
|mi_subnet   |Any           |Any     |MI SUBNET        |MI SUBNET  |Allow |

### <a name="mandatory-user-defined-routes-with-service-aided-subnet-configuration"></a>Rutas obligatorias definidas por el usuario con configuración de subred asistida por servicios
Estas rutas son necesarias para garantizar que el tráfico de administración se enruta directamente a un destino. Consulte el [párrafo anterior](#high-level-connectivity-architecture) para más información sobre la arquitectura de conectividad y el tráfico de administración.

|Nombre|Prefijo de dirección|Próximo salto|
|----|--------------|-------|
|subnet-to-vnetlocal|MI SUBNET|Virtual network|
|mi-azurecloud-REGION-internet|AzureCloud.REGION|Internet|
|mi-azurecloud-REGION_PAIR-internet|AzureCloud.REGION_PAIR|Internet|
|mi-azuremonitor-internet|AzureMonitor|Internet|
|mi-corpnetpublic-internet|CorpNetPublic|Internet|
|mi-corpnetsaw-internet|CorpNetSaw|Internet|
|mi-eventhub-REGION-internet|EventHub.REGION|Internet|
|mi-eventhub-REGION_PAIR-internet|EventHub.REGION_PAIR|Internet|
|mi-sqlmanagement-internet|SqlManagement|Internet|
|mi-storage-internet|Storage|Internet|
|mi-storage-REGION-internet|Storage.REGION|Internet|
|mi-storage-REGION_PAIR-internet|Storage.REGION_PAIR|Internet|
|mi-azureactivedirectory-internet|AzureActiveDirectory|Internet|
||||

\* MI SUBNET se refiere al intervalo de direcciones IP de la subred con el formato x.x.x.x/y. Puede encontrar esta información en Portal de Azure, en las propiedades de subred.

\** Si la dirección de destino es para uno de los servicios de Azure, Azure enruta el tráfico directamente al servicio a través de su red troncal, en lugar de enrutarlo a Internet. El tráfico entre los servicios de Azure no atraviesa Internet, independientemente de la región de Azure en que exista la red virtual o de la región de Azure en que se implementa una instancia del servicio de Azure. Para obtener más detalles, consulte la [página de documentación de UDR](../../virtual-network/virtual-networks-udr-overview.md).

Además, puede agregar entradas a la tabla de rutas para enrutar el tráfico que tiene intervalos IP privados locales como destino a través de una puerta de enlace de red virtual o de un dispositivo de red virtual (NVA).

Si la red virtual incluye un DNS personalizado, el servidor DNS personalizado debe ser capaz de resolver los registros DNS públicos. El uso de características adicionales, como la autenticación de Azure AD, puede requerir resolver FQDN adicionales. Para más información, consulte [Configuración de un DNS personalizado ](custom-dns-configure.md).

### <a name="networking-constraints"></a>Restricciones de redes

**TLS 1.2 se aplica en las conexiones salientes**: En enero 2020, Microsoft aplicó TLS 1.2 para el tráfico entre servicios en todos los servicios de Azure. En el caso de Instancia administrada de Azure SQL, esto supuso la aplicación de TLS 1.2 en las conexiones salientes usadas para la replicación y las conexiones del servidor vinculado a SQL Server. Si usa versiones de SQL Server anteriores a 2016 con SQL Managed Instance, asegúrese de que se hayan aplicado las [actualizaciones específicas de TLS 1.2](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server).

Las siguientes características de red virtual *no se admiten* actualmente con SQL Managed Instance:

- **Emparejamiento de Microsoft**: la habilitación del [emparejamiento de Microsoft](../../expressroute/expressroute-faqs.md#microsoft-peering) en circuitos de ExpressRoute emparejados directamente o de manera transitiva con la red virtual en la que reside SQL Managed Instance afecta al flujo de tráfico entre los componentes de SQL Managed Instance dentro de la red virtual y los servicios de los que depende, causando problemas de disponibilidad. Se prevé que se produzcan errores en las implementaciones de SQL Managed Instance en la red virtual con el emparejamiento de Microsoft habilitado.
- **Emparejamiento global de redes virtuales**: la conectividad de [emparejamiento de redes virtuales](../../virtual-network/virtual-network-peering-overview.md) entre regiones de Azure no funciona con instancias de SQL Managed Instance colocadas en subredes creadas antes del 22/9/2020.
- **AzurePlatformDNS**: el uso de la [etiqueta de servicio](../../virtual-network/service-tags-overview.md) AzurePlatformDNS para bloquear la resolución DNS de la plataforma hace que SQL Managed Instance no esté disponible. Aunque SQL Managed Instance admite DNS definida por el cliente para la resolución DNS dentro del motor, existe una dependencia en la DNS de la plataforma para las operaciones de plataforma.
- **NAT Gateway**: el uso de [Azure Virtual Network NAT](../../virtual-network/nat-overview.md) para controlar la conectividad saliente con una dirección IP pública específica, representaría SQL Managed Instance como no disponible. Actualmente, el servicio SQL Managed Instance está limitado al uso de un equilibrador de carga básico que no proporciona coexistencia de flujos entrantes y salientes con Virtual Network NAT.
- **IPv6 para Azure Virtual Network**: Se espera que se produzca un error en la implementación de Azure SQL Managed Instance en [redes virtuales IPv4/IPv6 de pila dual](../../virtual-network/ipv6-overview.md). Al asociar el grupo de seguridad de red (NSG) o la tabla de rutas (UDR) que contiene los prefijos de dirección IPv6 a la subred de SQL Managed Instance, o al agregar prefijos de direcciones IPv6 a NSG o UDR que ya está asociado a la subred de instancia administrada, se representaría la instancia de SQL Managed Instance no disponible. Se espera que las implementaciones de Azure SQL Managed Instance en una subred con NSG y UDR que ya tienen prefijos IPv6.
- **Zonas privadas de Azure DNS con un nombre reservado para servicios de Microsoft**: esta es una lista de los nombres reservados: windows.net, database.windows.net, core.windows.net, blob.core.windows.net, table.core.windows.net, management.core.windows.net, monitoring.core.windows.net, queue.core.windows.net, graph.windows.net, login.microsoftonline.com, login.windows.net, servicebus.windows.net, vault.azure.net. La implementación de SQL Managed Instance en una red virtual con una [zona privada de Azure DNS](../../dns/private-dns-privatednszone.md) asociada con un nombre reservado para servicios de Microsoft producirá un error. La asociación de la zona privada de Azure DNS con un nombre reservado con una red virtual que contiene Managed Instance representaría SQL Managed Instance como no disponible. Siga la [configuración de DNS del punto de conexión privado de Azure](../../private-link/private-endpoint-dns.md) para conocer la configuración apropiada de Private Link.
- **Directivas de punto de conexión de servicio de Azure Storage**: se producirá un error al implementar SQL Managed Instance en una subred que tenga [directivas de punto de conexión de servicio](../../virtual-network/virtual-network-service-endpoint-policies-overview.md) asociadas. Las directivas de punto de conexión de servicio no se pueden asociar a una subred que hospeda Managed Instance.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, vea  [¿Qué es Instancia administrada de Azure SQL?](sql-managed-instance-paas-overview.md)
- Aprenda a [configurar una nueva red virtual de Azure](virtual-network-subnet-create-arm-template.md) o una [red virtual de Azure existente](vnet-existing-add-subnet.md) donde pueda implementar SQL Managed Instance.
- [Calcule el tamaño de la subred](vnet-subnet-determine-size.md) donde quiere implementar SQL Managed Instance.
- Aprenda a crear una instancia administrada:
  - Desde [Azure Portal](instance-create-quickstart.md).
  - Mediante [PowerShell](scripts/create-configure-managed-instance-powershell.md).
  - Mediante [una plantilla de Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-sqlmi-new-vnet/).
  - Mediante [una plantilla de Azure Resource Manager (JumpBox con SSMS incluido)](https://azure.microsoft.com/resources/templates/201-sqlmi-new-vnet-w-jumpbox/).
