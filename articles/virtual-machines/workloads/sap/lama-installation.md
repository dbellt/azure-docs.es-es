---
title: Conector de SAP LaMa para Azure | Microsoft Docs
description: Administración de sistemas SAP en Azure con SAP LaMa
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: timlt
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/29/2019
ms.author: sedusch
ms.custom: subject-rbac-steps
ms.openlocfilehash: 9eca2fe92109bcd91fe5943e53d1e18734401984
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111814366"
---
# <a name="sap-lama-connector-for-azure"></a>Conector de SAP LaMa para Azure

[1877727]:https://launchpad.support.sap.com/#/notes/1877727
[2343511]:https://launchpad.support.sap.com/#/notes/2343511
[2350235]:https://launchpad.support.sap.com/#/notes/2350235
[2562184]:https://launchpad.support.sap.com/#/notes/2562184
[2628497]:https://launchpad.support.sap.com/#/notes/2628497
[2445033]:https://launchpad.support.sap.com/#/notes/2445033
[2815988]:https://launchpad.support.sap.com/#/notes/2815988
[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png
[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[hana-ops-guide]:hana-vm-operations.md

> [!NOTE]
> Declaración general de soporte técnico: si necesita soporte técnico para SAP LaMa o el conector de Azure, abra siempre una incidencia con SAP sobre el componente BC-VCM-LVM-HYPERV.

Muchos clientes usan SAP LaMa para operar y supervisar su entorno de SAP. A partir de SAP LaMa 3.0 SP05, se suministra con un conector de Azure de forma predeterminada. Puede usar este conector para desasignar e iniciar máquinas virtuales o copiar, reubicar y eliminar discos administrados. Con estas operaciones básicas, puede reubicar, copiar, clonar y actualizar los sistemas SAP mediante SAP LaMa.

En esta guía se describe cómo configurar el conector de Azure para SAP LaMa, crear las máquinas virtuales que pueden usarse para instalar sistemas SAP adaptables y cómo configurar estas máquinas.

> [!NOTE]
> El conector solo está disponible con la edición Enterprise de SAP LaMa.

## <a name="resources"></a>Recursos

Las siguientes notas de SAP están relacionadas con el tema de SAP LaMa en Azure:

| Número de nota | Título |
| --- | --- |
| [2343511] |Conector de Microsoft Azure para SAP Landscape Management (LaMa) |
| [2350235] |SAP Landscape Management 3.0, edición Enterprise |

Visite también el [portal de ayuda de SAP para SAP LaMa](https://help.sap.com/viewer/p/SAP_LANDSCAPE_MANAGEMENT_ENTERPRISE).

## <a name="general-remarks"></a>Observaciones generales

* Asegúrese de habilitar *Automatic Mountpoint Creation* (Creación automática de puntos de montaje) Setup -> Settings -> Engine (Configuración -> Configuración -> Motor).  
  Si SAP LaMa monta volúmenes mediante las extensiones adaptables de SAP en una máquina virtual, debe haber un punto de montaje si esta opción no está habilitada.

* Use una subred independiente y no use direcciones IP dinámicas para evitar el "robo" de direcciones IP al implementar nuevas máquinas virtuales sin que las instancias de SAP estén preparadas.  
  Si usa la asignación de direcciones IP dinámicas en la subred, que se usa también en SAP LaMa, la preparación de un sistema SAP con SAP LaMa podría dar error. Si un sistema SAP no está preparado, las direcciones IP no están reservadas y podrían asignarse a otras máquinas virtuales.

* Si inicia sesión en hosts administrados, asegúrese de no bloquear los sistemas de archivos desde los que se van a montar.  
  Si inicia sesión en una máquina virtual Linux y cambia el directorio de trabajo a un directorio de un punto de montaje, por ejemplo /usr/sap/AH1/ASCS00/exe, el volumen no se puede desmontar y la reubicación o la cancelación de la preparación darán error.

* Asegúrese de deshabilitar CLOUD_NETCONFIG_MANAGE en las máquinas virtuales SUSE SLES Linux. Para más información, consulte [SUSE KB 7023633](https://www.suse.com/support/kb/doc/?id=7023633).

## <a name="set-up-azure-connector-for-sap-lama"></a>Configuración del conector de Azure para SAP LaMa

El conector de Azure se distribuye a partir de SAP LaMa 3.0 SP05. Se recomienda instalar siempre el paquete de compatibilidad y la revisión más recientes para SAP LaMa 3.0.

El conector de Azure usa la API de Azure Resource Manager para administrar los recursos de Azure. SAP LaMa puede usar una entidad de servicio o una identidad administrada para autenticarse en esta API. Si SAP LaMa se ejecuta en una VM de Azure, se recomienda usar una identidad administrada, tal como se describe en el capítulo [Uso de una identidad administrada para obtener acceso a la API de Azure](lama-installation.md#af65832e-6469-4d69-9db5-0ed09eac126d). Si desea usar una entidad de servicio, siga los pasos descritos en el capítulo [Uso de una entidad de servicio para obtener acceso a la API de Azure](lama-installation.md#913c222a-3754-487f-9c89-983c82da641e).

### <a name="use-a-service-principal-to-get-access-to-the-azure-api"></a><a name="913c222a-3754-487f-9c89-983c82da641e"></a>Uso de una entidad de servicio para obtener acceso a la API de Azure

El conector de Azure puede usar una entidad de servicio para la autorización en Microsoft Azure. Siga estos pasos para crear una entidad de servicio para SAP Landscape Management (LaMa).

1. Vaya a https://portal.azure.com.
1. Abra la hoja Azure Active Directory
1. Haga clic en Registros de aplicaciones.
1. Haga clic en Nuevo registro.
1. Escriba un nombre y haga clic en Registrar.
1. Seleccione la nueva aplicación y haga clic en Certificados y secretos en la pestaña Configuración.
1. Cree un secreto de cliente, escriba una descripción para una nueva clave, seleccione cuándo debe expirar el secreto y haga clic en Guardar.
1. Anote el valor. Se usa como contraseña para la entidad de servicio
1. Anote el identificador de la aplicación. Se usa como nombre de usuario de la entidad de servicio.

La entidad de servicio no tiene permiso para tener acceso a los recursos de Azure de forma predeterminada.
Asigne el rol Colaborador a la entidad de servicio en el ámbito del grupo de recursos para todos los grupos de recursos que contengan sistemas SAP que SAP LaMa deba administrar.

Para asignar roles, consulte [Asignación de roles de Azure mediante Azure Portal](../../../role-based-access-control/role-assignments-portal.md).

### <a name="use-a-managed-identity-to-get-access-to-the-azure-api"></a><a name="af65832e-6469-4d69-9db5-0ed09eac126d"></a>Uso de una identidad administrada para obtener acceso a la API de Azure

Para poder usar una identidad administrada, la instancia de SAP LaMa debe ejecutarse en una VM de Azure que tenga una identidad asignada por el usuario o por el sistema. Para más información sobre las identidades administradas, consulte [¿Qué es Managed Identities for Azure Resources?](../../../active-directory/managed-identities-azure-resources/overview.md) y [Configurar identidades administradas para recursos de Azure en una VM mediante Azure Portal](../../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md).

La identidad administrada no tiene permiso para tener acceso a los recursos de Azure de forma predeterminada.
Asigne el rol Colaborador a la identidad de máquina virtual en el ámbito del grupo de recursos para todos los grupos de recursos que contengan sistemas SAP que SAP LaMa deba administrar.

Para asignar roles, consulte [Asignación de roles de Azure mediante Azure Portal](../../../role-based-access-control/role-assignments-portal.md).

En la configuración del conector de Azure para SAP LaMa, seleccione "Usar identidad administrada" para habilitar el uso de Identidad administrada. Si desea usar una identidad asignada por el sistema, asegúrese de dejar vacío el campo Nombre de usuario. Si desea usar una identidad asignada por el usuario, escriba el identificador de la identidad asignada por el usuario en el campo Nombre de usuario.

### <a name="create-a-new-connector-in-sap-lama"></a>Creación de un conector en SAP LaMa

Abra el sitio web de SAP LaMa y navegue hasta la infraestructura. Vaya a la pestaña Cloud Managers (Administradores de nube) y haga clic en Add (Agregar). Seleccione el adaptador para la nube de Microsoft Azure y haga clic en Next (Siguiente). Escriba la siguiente información:

* Etiqueta: elija un nombre para la instancia del conector.
* Nombre de usuario: Identificador de la aplicación de la entidad de servicio o identificador de la identidad asignada por el usuario de la máquina virtual. Para obtener más información, consulte [Uso de una identidad asignada por el sistema o el usuario].
* Contraseña: Clave o contraseña de la entidad de servicio. Puede dejar este campo vacío si usa una identidad asignada por el usuario o el sistema.
* Dirección URL: mantenga el valor predeterminado `https://management.azure.com/`.
* Intervalo de supervisión (segundos): debe ser al menos 300.
* Use Managed Identity: SAP LaMa puede usar una identidad asignada por el usuario o el sistema para autenticarse en la API de Azure. Consulte el capítulo [Uso de una identidad administrada para obtener acceso a la API de Azure](lama-installation.md#af65832e-6469-4d69-9db5-0ed09eac126d) de esta guía.
* Identificador de suscripción: Identificador de suscripción de Azure
* Identificador del inquilino de Azure Active Directory: el identificador del inquilino de Active Directory.
* Host de proxy: nombre de host del proxy si SAP LaMa necesita un proxy para conectarse a Internet.
* Puerto de proxy: el puerto TCP del proxy.
* Change Storage Type to save costs: Habilite esta opción si el adaptador de Azure debe cambiar el tipo de almacenamiento de Managed Disks para ahorrar costos cuando los discos no estén en uso. En el caso de los discos de datos a los que se hace referencia en una configuración de instancia de SAP, el adaptador cambiará el tipo de disco a Standard Storage durante la reversión de la preparación de una instancia y de nuevo al tipo de almacenamiento original durante la preparación de la instancia. Si detiene una máquina virtual en SAP LaMa, el adaptador cambiará el tipo de almacenamiento de todos los discos conectados, incluido el disco del sistema operativo, a Standard Storage. Si inicia una máquina virtual en SAP LaMa, el adaptador volverá a cambiar el tipo de almacenamiento al tipo de almacenamiento original.

Haga clic en Test Configuration (Probar configuración) para validar la entrada. Verá un mensaje que dice:

Conexión correcta: la conexión a la nube de Microsoft se ha realizado correctamente. Se han encontrado 7 grupos de recursos (solo 10 grupos solicitados)

en la parte inferior del sitio web.

## <a name="provision-a-new-adaptive-sap-system"></a>Aprovisionamiento de un nuevo sistema SAP adaptable

Puede implementar manualmente una nueva máquina virtual o usar una de las plantillas de Azure del [repositorio de inicio rápido](https://github.com/Azure/azure-quickstart-templates). Este repositorio contiene plantillas para [ASCS de SAP NetWeaver](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-ascs), [servidores de aplicaciones de SAP NetWeaver](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-apps) y la [base de datos](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-database). También puede usar estas plantillas para aprovisionar nuevos hosts como parte de la copia o la clonación de un sistema, etc.

Se recomienda usar una subred independiente para todas las máquinas virtuales que quiera administrar con SAP LaMa y no usar direcciones IP dinámicas para impedir el "robo" de direcciones IP al implementar nuevas máquinas virtuales cuando las instancias de SAP no están preparadas.

> [!NOTE]
> Si es posible, quite todas las extensiones de máquina virtual, ya que podrían provocar tiempos de ejecución largos por desconectar discos de una máquina virtual.

Asegúrese de que el usuario \<hanasid>adm, \<sapsid>adm y el grupo sapsys existen en el equipo de destino con el mismo identificador y GID o use LDAP. Habilite e inicie el servidor NFS en las máquinas virtuales que se deben usar para ejecutar (A)SCS de SAP NetWeaver.

### <a name="manual-deployment"></a>Implementación manual

SAP LaMa se comunica con la máquina virtual mediante el agente de host SAP. Si implementa las máquinas virtuales manualmente o sin usar la plantilla de Azure Resource Manager desde el repositorio de inicio rápido, asegúrese de instalar el agente de host SAP y las extensiones adaptables de SAP más recientes. Para más información sobre los niveles de revisión necesarios para Azure, consulte la nota de SAP [2343511].

#### <a name="manual-deployment-of-a-linux-virtual-machine"></a>Implementación manual de una máquina virtual Linux

Cree una máquina virtual con uno de los sistemas operativos admitidos que se indican en la nota de SAP [2343511]. Agregue configuraciones IP adicionales para las instancias de SAP. Cada instancia necesita al menos una dirección IP y debe instalarse con un nombre de host virtual.

La instancia de ASCS de SAP NetWeaver necesita discos para /sapmnt/\<SAPSID>, /usr/sap/\<SAPSID>, /usr/sap/trans y /usr/sap/\<sapsid>adm. Los servidores de aplicaciones de SAP NetWeaver no necesitan discos adicionales. Todo lo relacionado con la instancia de SAP se debe almacenar en ASCS y exportarse por medio de NFS. En caso contrario, actualmente no resulta posible agregar servidores de aplicaciones adicionales mediante SAP LaMa.

![ASCS de SAP NetWeaver en Linux](media/lama/sap-lama-ascs-app-linux.png)

#### <a name="manual-deployment-for-sap-hana"></a>Implementación manual de SAP HANA

Cree una máquina virtual con uno de los sistemas operativos admitidos para SAP HANA que se indican en la nota de SAP [2343511]. Agregue una configuración IP adicional para SAP HANA y una por inquilino de HANA.

SAP HANA necesita discos para /hana/shared, /hana/backup, /hana/data, and /hana/log

![SAP HANA en Linux](media/lama/sap-lama-db-hana.png)

#### <a name="manual-deployment-for-oracle-database-on-linux"></a>Implementación manual para Oracle Database en Linux

Cree una máquina virtual con uno de los sistemas operativos admitidos para bases de datos Oracle que se indican en la nota de SAP [2343511]. Agregue una configuración IP adicional para la base de datos Oracle.

La base de datos Oracle necesita discos para /oracle, /home/oraod1 y /home/oracle

![Diagrama que muestra una base de datos de Oracle en Linux y los discos que necesita.](media/lama/sap-lama-db-ora-lnx.png)

#### <a name="manual-deployment-for-microsoft-sql-server"></a>Implementación manual para Microsoft SQL Server

Cree una máquina virtual con uno de los sistemas operativos admitidos para Microsoft SQL Server que se indican en la nota de SAP [2343511]. Agregue una configuración IP adicional para la instancia de SQL Server.

El servidor de base de datos de SQL Server necesita discos para los datos de base de datos y archivos de registro y discos para c:\usr\sap.

![Oracle Database en Linux](media/lama/sap-lama-db-sql.png)

Asegúrese de instalar un controlador ODBC de Microsoft compatible para SQL Server en una máquina virtual que quiera usar para reubicar un servidor de aplicaciones de SAP NetWeaver o como destino de copia o clonación de un sistema.

SAP LaMa no puede reubicar SQL Server por sí mismo, así que una máquina virtual que quiera usar para reubicar una instancia de base de datos o como destino de copia o clonación de un sistema necesita tener instalado SQL Server.

### <a name="deploy-virtual-machine-using-an-azure-template"></a>Implementación de una máquina virtual con una plantilla de Azure

Descargue los siguientes archivos disponibles más recientes del [catálogo de software de SAP](https://support.sap.com/swdc) para el sistema operativo de las máquinas virtuales:

1. SAPCAR 7.21
1. SAP HOST AGENT 7.21
1. SAP ADAPTIVE EXTENSION 1.0 EXT

Además, descargue los siguientes componentes del [Centro de descarga de Microsoft](https://www.microsoft.com/download).

1. Paquete redistribuible de Microsoft Visual C++ 2010 (x64) (solo Windows)
1. Microsoft ODBC Driver for SQL Server (solo SQL Server)

Los componentes son necesarios para implementar la plantilla. La manera más fácil de conseguir que estén disponibles para la plantilla es cargarlos en una cuenta de almacenamiento de Azure y crear una firma de acceso compartido (SAS).

Las plantillas tienen los siguientes parámetros:

* sapSystemId: el identificador del sistema SAP. Se usa para crear el diseño del disco (por ejemplo, /usr/sap/\<sapsid>).

* computerName: el nombre del equipo de la nueva máquina virtual. Este parámetro se usa también en SAP LaMa. Cuando se usa esta plantilla para aprovisionar una nueva máquina virtual como parte de una copia del sistema, SAP LaMa espera hasta que es posible la comunicación con el host que tiene este nombre de equipo.

* osType: el tipo del sistema operativo que quiere implementar.

* dbtype: el tipo de la base de datos. Este parámetro se usa para determinar cuántas configuraciones IP adicionales es necesario agregar y cómo debe ser el diseño de disco.

* sapSystemSize: el tamaño del sistema SAP que quiere implementar. Se usa para determinar el tamaño y el tipo de la instancia de máquina virtual.

* adminUsername: nombre de usuario de la máquina virtual.

* adminPassword: contraseña de la máquina virtual. También puede proporcionar una clave SSH pública.

* sshKeyData: clave SSH pública para las máquinas virtuales. Solo se admite en sistemas operativos Linux.

* subnetId: el identificador de la subred que quiere usar.

* deployEmptyTarget: puede implementar un destino vacío si quiere usar la máquina virtual como destino para la reubicación de una instancia o similar. En este caso, no se asocian discos adicionales o configuraciones IP adicionales.

* sapcarLocation: la ubicación de la aplicación sapcar que coincide con el sistema operativo implementado. sapcar se usa para extraer los archivos que se proporcionan en otros parámetros.

* sapHostAgentArchiveLocation: la ubicación del archivo de agente de host SAP. El agente de host SAP se implementa como parte de esta implementación de plantilla.

* sapacExtLocation: la ubicación de las extensiones adaptables de SAP. En la nota de SAP [2343511] se indica el nivel de revisión mínima necesario para Azure.

* vcRedistLocation: la ubicación del entorno en tiempo de ejecución de VC que es necesaria para instalar las extensiones adaptables de SAP. Este parámetro solo es necesario para Windows.

* odbcDriverLocation: la ubicación del controlador ODBC que quiere instalar. Solo se admite el controlador ODBC de Microsoft para SQL Server.

* sapadmPassword: la contraseña del usuario sapadm.

* sapadmId: el identificador de usuario de Linux del usuario sapadm. No es necesario en Windows.

* sapsysGid: el identificador de grupo de Linux del grupo sapsys. No es necesario en Windows.

* _artifactsLocation: el identificador URI base, donde se encuentran los artefactos que necesita esta plantilla. Cuando la plantilla se implementa mediante los scripts complementarios, se usa una ubicación privada en la suscripción y este valor se genera automáticamente. Solo es necesario si no se implementa la plantilla desde GitHub.

* _artifactsLocationSasToken: el valor de sasToken necesario para acceder a _artifactsLocation. Cuando la plantilla se implementa mediante los scripts complementarios, se genera automáticamente un valor de sasToken. Solo es necesario si no se implementa la plantilla desde GitHub.

### <a name="sap-hana"></a>SAP HANA

En los ejemplos siguientes, se da por hecho que instala SAP HANA con el identificador del sistema HN1 y el sistema SAP NetWeaver con el identificador del sistema AH1. Los nombres de host virtuales son hn1-db para la instancia de HANA, ah1-db para el inquilino de HANA usado por el sistema SAP NetWeaver, ah1-ascs para el ASCS de SAP NetWeaver y ah1-di-0 para el primer servidor de aplicaciones de SAP NetWeaver.

#### <a name="install-sap-netweaver-ascs-for-sap-hana-using-azure-managed-disks"></a>Instalación de ASCS de SAP NetWeaver para SAP HANA mediante Azure Managed Disks

Antes de iniciar SAP Software Provisioning Manager (SWPM), debe montar la dirección IP del nombre de host virtual de ASCS. El método recomendado es usar sapacext. Si monta la dirección IP con sapacext, asegúrese de volver a montar la dirección IP después de reiniciar.

![Logotipo de Linux.][Logo_Linux] Linux

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-ascs -n 255.255.255.128
```

![Logotipo de Windows.][Logo_Windows] Windows

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h ah1-ascs -n 255.255.255.128
```

Ejecute SWPM y use *ah1-ascs* para *ASCS Instance Host Name* (Nombre de host de la instancia de ASCS).

![Logotipo de Linux.][Logo_Linux] Linux  
Agregue el siguiente parámetro de perfil al perfil de agente de host SAP, que se encuentra en /usr/sap/hostctrl/exe/host_profile. Para más información, consulte la nota de SAP [2628497].
```
acosprep/nfs_paths=/home/ah1adm,/usr/sap/trans,/sapmnt/AH1,/usr/sap/AH1
```

#### <a name="install-sap-netweaver-ascs-for-sap-hana-on-azure-netappfiles-anf-beta"></a>Instalación de SAP NetWeaver ASCS para SAP HANA en Azure NetAppFiles (ANF) BETA

> [!NOTE]
> Esta funcionalidad todavía no está disponible. Para obtener más información, consulte la Nota de SAP [2815988] (solo visible para los clientes de vista previa).
Abrir un incidente de SAP en el componente BC-VCM-LVM-HYPERV y solicitar unirse al adaptador de almacenamiento de LaMa para Azure NetApp Files (versión preliminar).

ANF proporciona NFS para Azure. En el contexto de SAP LaMa, se simplifica la creación de las instancias de ABAP Central Services (ASCS) y la posterior instalación de los servidores de aplicaciones. Anteriormente, la instancia de ASCS tenía que actuar como servidor NFS y había que agregar el parámetro acosprep/nfs_paths al host_profile de SAP Hostagent.

#### <a name="anf-is-currently-available-in-these-regions"></a>ANF está disponible actualmente en estas regiones:

Este de Australia, Centro de EE. UU., Este de EE. UU., Este de EE. UU. 2, Norte de Europa, Centro-sur de EE. UU., Oeste de Europa, Oeste de EE. UU. 2.

#### <a name="network-requirements"></a>Requisitos de red

ANF requiere una subred delegada que debe formar parte de la misma red virtual que los servidores SAP. Este es un ejemplo de este tipo de configuración:
En esta pantalla se muestra la creación de la red virtual y la primera subred:

![SAP LaMa Creación de una red virtual para Azure ANF ](media/lama/sap-lama-createvn-50.png)

En el paso siguiente se crea la subred delegada para Microsoft.NetApp/volumes.

![SAP LaMa Agregar subred delegada ](media/lama/sap-lama-addsubnet-50.png)

![SAP LaMa Lista de subredes ](media/lama/sap-lama-subnets.png)

Ahora es necesario crear una cuenta de NetApp en Azure Portal:

![SAP LaMa Crear cuenta de NetApp ](media/lama/sap-lama-create-netappaccount-50.png)

![SAP LaMa Cuenta de NetApp creada ](media/lama/sap-lama-netappaccount.png)

Dentro de la cuenta de NetApp, el grupo de capacidad especifica el tamaño y el tipo de discos para cada grupo:

![SAP LaMa Crear grupo de capacidad de NetApp ](media/lama/sap-lama-capacitypool-50.png)

![SAP LaMa Grupo de capacidad de NetApp creado ](media/lama/sap-lama-capacitypool-list.png)

Ahora se pueden definir los volúmenes NFS. Dado que habrá volúmenes para varios sistemas en un grupo, se debe elegir un esquema de nomenclatura autodescriptivo. Agregar el SID ayuda a agrupar los volúmenes relacionados. En el caso de la instancia de ASCS y AS, se necesitan los siguientes montajes: */sapmnt/\<SID\>* , */usr/sap/\<SID\>* y */home/\<sid\>adm*. De forma opcional, se necesita */usr/sap/trans* para el directorio central de transporte, que se utiliza, como mínimo, en todos los sistemas de un escenario.

> [!NOTE]
> Durante la fase BETA, el nombre de los volúmenes debe ser único dentro de la suscripción.

![SAP LaMa Crear un volumen 1 ](media/lama/sap-lama-createvolume-80.png)

![SAP LaMa Crear un volumen 2 ](media/lama/sap-lama-createvolume2-80.png)

![SAP LaMa Crear un volumen 3 ](media/lama/sap-lama-createvolume3-80.png)

Estos pasos deben repetirse también para los demás volúmenes.

![SAP LaMa Lista de volúmenes creados ](media/lama/sap-lama-volumes.png)

Ahora estos volúmenes deben montarse en los sistemas donde se realizará la instalación inicial con SAP SWPM.

En primer lugar, es necesario crear los puntos de montaje. En este caso, el SID es AN1, por lo que deben ejecutarse los siguientes comandos:

```bash
mkdir -p /home/an1adm
mkdir -p /sapmnt/AN1
mkdir -p /usr/sap/AN1
mkdir -p /usr/sap/trans
```
Después, los volúmenes de ANF se montarán con los siguientes comandos:

```bash
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/an1-home-sidadm /home/an1adm
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/an1-sapmnt-sid /sapmnt/AN1
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/an1-usr-sap-sid /usr/sap/AN1
# sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp 9.9.9.132:/global-usr-sap-trans /usr/sap/trans
```
Los comandos de montaje también se pueden derivar del portal. Los puntos de montaje locales deben ajustarse.

Use el comando df -h para comprobar.

![SAP LaMa Nivel de sistema operativo de puntos de montaje ](media/lama/sap-lama-mounts.png)

Ahora se debe realizar la instalación con SWPM.

Se deben realizar los mismos pasos para, al menos, una instancia de AS.

Después de la instalación correcta, el sistema debe detectarse dentro de SAP LaMa.

Los puntos de montaje deben tener un aspecto similar al siguiente para la instancia de ASCS y la de AS:

![Puntos de montaje de SAP en LaMa ](media/lama/sap-lama-ascs.png) (Este es un ejemplo. Las direcciones IP y la ruta de acceso de exportación son diferentes de las que se usaron antes)


#### <a name="install-sap-hana"></a>Instalación de SAP HANA

Si instala SAP HANA mediante la herramienta de línea de comandos hdblcm, use el parámetro --hostname para proporcionar un nombre de host virtual. Deberá agregar la dirección IP del nombre de host virtual de la base de datos a una interfaz de red. El método recomendado es usar sapacext. Si monta la dirección IP con sapacext, asegúrese de volver a montar la dirección IP después de reiniciar.

Agregue otro nombre de host virtual y otra dirección IP para el nombre que usarán los servidores de aplicaciones para conectar con el inquilino de HANA.

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h hn1-db -n 255.255.255.128
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-db -n 255.255.255.128
```

Ejecute la instalación de la instancia de base de datos de SWPM en la máquina virtual del servidor de aplicaciones, no en la máquina virtual de HANA. Use *ah1-db* para *Database Host* (Host de base de datos) del cuadro de diálogo *Database for SAP System* (Base de datos para el sistema SAP).

#### <a name="install-sap-netweaver-application-server-for-sap-hana"></a>Instalación del servidor de aplicaciones de SAP NetWeaver para SAP HANA

Antes de iniciar SAP Software Provisioning Manager (SWPM), debe montar la dirección IP del nombre de host virtual del servidor de aplicaciones. El método recomendado es usar sapacext. Si monta la dirección IP con sapacext, asegúrese de volver a montar la dirección IP después de reiniciar.

![Logotipo de Linux.][Logo_Linux] Linux

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-di-0 -n 255.255.255.128
```

![Logotipo de Windows.][Logo_Windows] Windows

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h ah1-di-0 -n 255.255.255.128
```

Se recomienda usar el parámetro de perfil de SAP NetWeaver dbs/hdb/hdb_use_ident para establecer la identidad que se usa para encontrar la clave en userstore de HDB. Puede agregar este parámetro manualmente después de la instalación de la instancia de base de datos con SWPM o ejecutar SWPM con

```bash
# from https://blogs.sap.com/2015/04/14/sap-hana-client-software-different-ways-to-set-the-connectivity-data/
/sapdb/DVDs/IM_LINUX_X86_64/sapinst HDB_USE_IDENT=SYSTEM_COO
```

Si lo establece manualmente, también debe crear nuevas entradas userstore de HDB.

```bash
# run as <sapsid>adm
/usr/sap/AH1/hdbclient/hdbuserstore LIST
# reuse the port that was listed from the command above, in this example 35041
/usr/sap/AH1/hdbclient/hdbuserstore SET DEFAULT ah1-db:35041@AH1 SAPABAP1 <password>
```

Use *ah1-di-0* para *PAS Instance Host Name* (Nombre de host de la instancia de PAS) del cuadro de diálogo *Primary Application Server Instance* (Instancia principal del servidor de aplicaciones).

#### <a name="post-installation-steps-for-sap-hana"></a>Pasos posteriores a la instalación para SAP HANA

Asegúrese de realizar la copia de seguridad de SYSTEMDB y de todas las bases de datos de inquilino antes de intentar realizar una copia del inquilino, un traslado de inquilinos o de crear una replicación del sistema.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

En los ejemplos siguientes, se da por hecho que instala el sistema SAP NetWeaver con el identificador del sistema AH1. Los nombres de host virtuales son as1-db para la instancia de SQL Server que usa el sistema SAP NetWeaver, as1-ascs para el ASCS de SAP NetWeaver y ah1-di-0 para el primer servidor de aplicaciones de SAP NetWeaver.

#### <a name="install-sap-netweaver-ascs-for-sql-server"></a>Instalación de ASCS de SAP NetWeaver para SQL Server

Antes de iniciar SAP Software Provisioning Manager (SWPM), debe montar la dirección IP del nombre de host virtual de ASCS. El método recomendado es usar sapacext. Si monta la dirección IP con sapacext, asegúrese de volver a montar la dirección IP después de reiniciar.

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-ascs -n 255.255.255.128
```

Ejecute SWPM y use *as1-ascs* para *ASCS Instance Host Name* (Nombre de host de la instancia de ASCS).

#### <a name="install-sql-server"></a>Instalación de SQL Server

Deberá agregar la dirección IP del nombre de host virtual de la base de datos a una interfaz de red. El método recomendado es usar sapacext. Si monta la dirección IP con sapacext, asegúrese de volver a montar la dirección IP después de reiniciar.

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-db -n 255.255.255.128
```

Ejecute la instalación de la instancia de base de datos de SWPM en la máquina virtual de SQL Server. Use SAPINST_USE_HOSTNAME=*as1-db* para invalidar el nombre de host usado para conectar con SQL Server. Si implementó la máquina virtual mediante la plantilla de Azure Resource Manager, asegúrese de establecer el directorio usado para los archivos de datos de la base de datos en *C:\sql\data* y el archivo de registro de base de datos en *C:\sql\log*.

Asegúrese de que el usuario *NT AUTHORITY\SYSTEM* tenga acceso a SQL Server y tenga el rol de servidor *sysadmin*. Para más información, consulte las notas de SAP [1877727] y [2562184].

#### <a name="install-sap-netweaver-application-server"></a>Instalación del servidor de aplicaciones de SAP NetWeaver

Antes de iniciar SAP Software Provisioning Manager (SWPM), debe montar la dirección IP del nombre de host virtual del servidor de aplicaciones. El método recomendado es usar sapacext. Si monta la dirección IP con sapacext, asegúrese de volver a montar la dirección IP después de reiniciar.

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-di-0 -n 255.255.255.128
```

Use *as1-di-0* para *PAS Instance Host Name* (Nombre de host de la instancia de PAS) del cuadro de diálogo *Primary Application Server Instance* (Instancia principal del servidor de aplicaciones).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="errors-and-warnings-during-discover"></a>Errores y advertencias durante la detección

* Se denegó el permiso SELECT.
  * [Microsoft][ODBC SQL Server Driver][SQL Server]El permiso SELECT se denegó en el objeto 'log_shipping_primary_databases', database 'msdb', schema 'dbo'. [SOAPFaultException]  
  El permiso SELECT se denegó en el objeto 'log_shipping_primary_databases', database 'msdb', schema 'dbo'.
  * Solución  
    Asegúrese de que *NT AUTHORITY\SYSTEM* pueda acceder a SQL Server. Consulte la nota de SAP [2562184].


### <a name="errors-and-warnings-for-instance-validation"></a>Errores y advertencias para la validación de instancias

* Se generó una excepción en la validación de userstore de HDB.  
  * Consulte el Visor de registros.  
    com.sap.nw.lm.aci.monitor.api.validation.RuntimeValidationException: excepción de validador con el identificador "RuntimeHDBConnectionValidator" (validación: "VALIDATION_HDB_USERSTORE"): No se pudo recuperar el valor de hdbuserstore.  
    userstore de HANA no está en la ubicación correcta.
  * Solución  
    Asegúrese de que /usr/sap/AH1/hdbclient/install/installation.ini sea correcto.

### <a name="errors-and-warnings-during-a-system-copy"></a>Errores y advertencias durante una copia del sistema

* Se produjo un error al validar el paso de aprovisionamiento del sistema.
  * Ocasionado por: com.sap.nw.lm.aci.engine.base.api.util.exception.HAOperationException Calling '/usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r' | /usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r
  * Solución  
    Realice una copia de seguridad de todas las bases de datos del sistema HANA de origen.

* Paso *Start* (Inicio) de copia del sistema de la instancia de base de datos.
  * Error en la operación del agente de host "000D3A282BC91EE8A1D76CF1F92E2944" (OperationException. FaultCode: "127". Mensaje: "Error al ejecutar el comando. : [Microsoft][ODBC SQL Server Driver][SQL Server]El usuario ni tiene permiso para modificar la base de datos "AS2", la base de datos no existe o no se encuentra en un estado que permita comprobaciones de acceso").
  * Solución  
    Asegúrese de que *NT AUTHORITY\SYSTEM* pueda acceder a SQL Server. Consulte la nota de SAP [2562184].

### <a name="errors-and-warnings-during-a-system-clone"></a>Errores y advertencias durante una clonación del sistema

* Se produjo un error al intentar registrar el agente de instancia en el paso *Forced Register and Start Instance Agent* (Registro e inicio forzosos del agente de instancia) del servidor de aplicaciones o ASCS
  * Se produjo un error al intentar registrar el agente de instancia. (RemoteException: "No se pudo cargar los datos de la instancia del perfil '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0':  No se puede acceder al perfil '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0': No se encontró el archivo o directorio").
  * Solución  
   Asegúrese de que el recurso compartido de sapmnt en ASCS/SCS tenga acceso completo para SAP_AS1_GlobalAdmin.

* Error en el paso *Enable Startup Protection for Clone* (Habilitar protección de inicio para clon)
  * No se pudo abrir el archivo "\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0" Causa: No se encontró el archivo o directorio.
  * Solución  
    La cuenta de equipo del servidor de aplicaciones necesita acceso de escritura al perfil.

### <a name="errors-and-warnings-during-create-system-replication"></a>Errores y advertencias durante la creación de la replicación del sistema

* Excepción al hacer clic en Crear replicación del sistema
  * Ocasionado por: com.sap.nw.lm.aci.engine.base.api.util.exception.HAOperationException Calling '/usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r' | /usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r
  * Solución  
    Pruebe si se puede ejecutar sapacext como `<hanasid`> adm.

* Error cuando la copia completa no está habilitada en el paso de almacenamiento
  * Se produjo un error al informar de un mensaje de atributo de contexto para la ruta de acceso IStorageCopyData.storageVolumeCopyList:1 y el campo targetStorageSystemId
  * Solución  
    Omita las advertencias del paso e inténtelo de nuevo. Este problema se corregirá en un nuevo paquete o revisión de soporte técnico de SAP LaMa.

### <a name="errors-and-warnings-during-relocate"></a>Errores y advertencias durante la reubicación

* La ruta de acceso "/usr/sap/AH1" no se permite para reexportaciones de nfs.
  * Consulte la nota de SAP [2628497] para más información.
  * Solución  
    Agregue exportaciones de ASCS al perfil de agente de host de ASCS. (Consulte la nota de SAP [2628497]).

* Función no implementada al reubicar ASCS
  * Salida del comando: exportfs: host:/usr/sap/AX1: función no implementada
  * Solución  
    Asegúrese de que el servicio del servidor NFS está habilitado en la máquina virtual de destino de reubicación.

### <a name="errors-and-warnings-during-application-server-installation"></a>Errores y advertencias durante la instalación del servidor de aplicaciones

* Error al ejecutar el paso SAPinst: getProfileDir
  * ERROR: (Último error notificado por el paso: se produjo la excepción ESAPinstException en la llamada del módulo: El validador del paso "|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_readProfileDir|ind|ind|ind|ind|readProfile|0|getProfileDir" notificó un error: El nodo \\\as1-ascs\sapmnt\AS1\SYS\profile no existe. Inicie SAPinst en modo interactivo para resolver este problema)
  * Solución  
    Asegúrese de que se está ejecutando SWPM con un usuario que tenga acceso al perfil. Este usuario se puede configurar en el Asistente para instalación del servidor de aplicaciones.

* Error al ejecutar el paso SAPinst: askUnicode
  * ERROR: (Último error notificado por el paso: se produjo la excepción ESAPinstException en la llamada del módulo: El validador del paso "|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_getUnicode|ind|ind|ind|ind|unicode|0|askUnicode" notificó un error: Inicie SAPinst en modo interactivo para resolver este problema)
  * Solución  
    Si usa un kernel de SAP reciente, SWPM ya no puede determinar si el sistema es un sistema Unicode con el servidor de mensajes de ASCS. Consulte la nota de SAP [2445033] para más información.  
    Este problema se corregirá en un nuevo paquete o revisión de soporte técnico de SAP LaMa.  
    Para solucionar este problema, establezca el parámetro del perfil OS_UNICODE=uc en el perfil predeterminado de su sistema SAP.

* Error al ejecutar el paso SAPinst: dCheckGivenServer
  * Error al ejecutar el paso SAPinst: dCheckGivenServer" version="1.0" ERROR: (Último error notificado por el paso: \<p> El usuario canceló la instalación. \</p>
  * Solución  
    Asegúrese de que se está ejecutando SWPM con un usuario que tenga acceso al perfil. Este usuario se puede configurar en el Asistente para instalación del servidor de aplicaciones.

* Error al ejecutar el paso SAPinst: checkClient
  * Error al ejecutar el paso SAPinst: checkClient" version="1.0" ERROR: (Último error notificado por el paso: \<p> El usuario canceló la instalación. \</p>)
  * Solución  
    Asegúrese de que el controlador ODBC de Microsoft para SQL Server está instalado en la máquina virtual en la que quiere instalar el servidor de aplicaciones.

* Error al ejecutar el paso SAPinst: copyScripts
  * Último error notificado por el paso: error en la llamada del sistema. DETALLES: Error 13 (0x0000000d) (Permiso denegado) en la ejecución de la llamada del sistema "fopenU" con el parámetro (\\\as1-ascs/sapmnt/AS1/SYS/exe/uc/NTAMD64/strdbs.cmd, w), línea (494) en el archivo (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/filesystem/syxxcfstrm2.cpp), seguimiento de la pila:  
  CThrThread.cpp: 85: CThrThread::threadFunction()  
  CSiServiceSet.cpp: 63: CSiServiceSet::executeService()  
  CSiStepExecute.cpp: 913: CSiStepExecute::execute()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  iaxxcfile.cpp: 183: iastring CIaOsFileConnect::callMemberFunction(iastring const& name, args_t const& args)  
  iaxxcfile.cpp: 1849: iastring CIaOsFileConnect::newFileStream(args_t const& _args)  
  iaxxbfile.cpp: 773: CIaOsFile::newFileStream_impl(4)  
  syxxcfile.cpp: 233: CSyFileImpl::openStream(ISyFile::eFileOpenMode)  
  syxxcfstrm.cpp: 29: CSyFileStreamImpl::CSyFileStreamImpl(CSyFileStream*,iastring,ISyFile::eFileOpenMode)  
  syxxcfstrm.cpp: 265: CSyFileStreamImpl::open()  
  syxxcfstrm2.cpp: 58: CSyFileStream2Impl::CSyFileStream2Impl(const CSyPath & \\\aw1-ascs/sapmnt/AW1/SYS/exe/uc/NTAMD64/strdbs.cmd, 0x4)  
  syxxcfstrm2.cpp: 456: CSyFileStream2Impl::open()
  * Solución  
    Asegúrese de que se está ejecutando SWPM con un usuario que tenga acceso al perfil. Este usuario se puede configurar en el Asistente para instalación del servidor de aplicaciones.

* Error al ejecutar el paso SAPinst: askPasswords
  * Último error notificado por el paso: error en la llamada del sistema. DETALLES: Error 5 (0x00000005) (Acceso denegado) en la ejecución de la llamada del sistema "NetValidatePasswordPolicy" con el parámetro (...), línea (359) en el archivo (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/account/synxcaccmg.cpp), seguimiento de la pila:  
  CThrThread.cpp: 85: CThrThread::threadFunction()  
  CSiServiceSet.cpp: 63: CSiServiceSet::executeService()  
  CSiStepExecute.cpp: 913: CSiStepExecute::execute()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  CSiStepExecute.cpp: 764: CSiStepExecute::invokeDialog()  
  DarkModeGuiEngine.cpp: 56: DarkModeGuiEngine::showDialogCalledByJs()  
  DarkModeDialog.cpp: 85: DarkModeDialog::submit()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  iaxxcaccount.cpp: 107: iastring CIaOsAccountConnect::callMemberFunction(iastring const& name, args_t const& args)  
  iaxxcaccount.cpp: 1186: iastring CIaOsAccountConnect::validatePasswordPolicy(args_t const& _args)  
  iaxxbaccount.cpp: 430: CIaOsAccount::validatePasswordPolicy_impl()  
  synxcaccmg.cpp: 297: ISyAccountMgt::PasswordValidationMessage CSyAccountMgtImpl::validatePasswordPolicy(saponazure,*****) const )
  * Solución  
    Asegúrese de agregar una regla de host en el paso *Isolation* (Aislamiento) para permitir la comunicación de la máquina virtual al controlador de dominio.

## <a name="next-steps"></a>Pasos siguientes
* [SAP HANA en la guía de operaciones de Azure][hana-ops-guide]
* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
