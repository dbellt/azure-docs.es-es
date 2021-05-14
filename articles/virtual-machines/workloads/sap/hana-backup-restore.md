---
title: Copia de seguridad y restauración de HANA en SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Realización de la copia de seguridad y restauración de HANA en SAP HANA en Azure (instancias grandes)
services: virtual-machines-linux
documentationcenter: ''
author: Ajayan1008
manager: gwallace
editor: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/16/2020
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b4b7ac968564c6c2e734fe48cca27c617eb82533
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109737283"
---
# <a name="backup-and-restore-of-sap-hana-on-hana-large-instances"></a>Copia de seguridad y restauración de SAP HANA en instancias grandes de HANA

>[!IMPORTANT]
>Este artículo no sustituye a la documentación de administración de SAP HANA ni a las notas de SAP. Se espera que el lector tenga un conocimiento sólido de la administración y operaciones de SAP HANA, sobre todo en lo relativo a la copia de seguridad, restauración, alta disponibilidad y recuperación ante desastres. En este artículo se muestran capturas de pantalla de SAP HANA Studio. El contenido, la estructura y la naturaleza de las pantallas de las herramientas de administración de SAP y de las herramientas en sí pueden cambiar de una versión a otra de SAP HANA.

Es importante que practique los pasos y procesos que se realizan en su entorno y con sus versiones de HANA. Algunos de los procesos descritos en este artículo se simplificaron para mejorar la comprensión general. No están diseñados para utilizarse como pasos detallados para posibles manuales de uso. Si quiere crear manuales de operaciones de sus configuraciones, pruebe y ejecute sus procesos y, después, documente aquellos relacionados con sus configuraciones concretas. 

Uno de los aspectos más importantes del funcionamiento de las bases de datos es protegerlas de eventos catastróficos. La causa de estos eventos pueden deberse a cualquier cosa, desde desastres naturales a simples errores del usuario.

La copia de seguridad de una base de datos, con la capacidad de realizar la restauración a cualquier momento dado (como antes de que alguien eliminara datos críticos), posibilita la restauración a un estado que sea lo más parecido posible al que había antes de la interrupción.

Se deben realizar dos tipos de copias de seguridad para obtener la capacidad de restauración:

- Copias de seguridad de bases de datos: completas, incrementales o diferenciales
- Copias de seguridad del registro de transacciones

Además de las copias de seguridad completas de la base de datos que se realizan en un nivel de aplicación, puede realizar copias de seguridad con instantáneas de almacenamiento. Las instantáneas de almacenamiento no reemplazan las copias de seguridad del registro de transacciones. Las copias de seguridad del registro de transacciones siguen siendo importantes para restaurar la base de datos a un momento dado o para eliminar las transacciones ya confirmadas de los registros. Aun así, las instantáneas de almacenamiento pueden acelerar la recuperación al proporcionar rápidamente una imagen de puesta al día de la base de datos. 

SAP HANA en Azure (instancias grandes) ofrece dos opciones de copia de seguridad y restauración:

- **Hágalo usted mismo (DIY).** Después de asegurarse de que hay suficiente espacio en disco, realice copias de seguridad completas tanto de las bases de datos como del registro mediante uno de los siguientes métodos de copia de seguridad de disco. Puede hacer copias de seguridad directamente en los volúmenes conectados a las unidades de HANA (instancias grandes) o en los recursos compartidos de NFS configurados en una máquina virtual (VM) de Azure. En el último caso, los clientes configuran una VM Linux en Azure, adjuntan Azure Storage a dicha VM y comparten el almacenamiento a través de un servidor NFS configurado en esa máquina virtual. Si realiza la copia de seguridad en los volúmenes que están adjuntos directamente a las unidades de HANA (instancias grandes), copie las copias de seguridad a una cuenta de almacenamiento de Azure. Puede hacerlo después de configurar una máquina virtual de Azure que exporte los recursos compartidos de NFS que se basan en Azure Storage. También puede usar un almacén de Azure Backup o el almacenamiento en frío de Azure. 

   Otra opción consiste en usar una herramienta de protección de datos de terceros para almacenar las copias de seguridad una vez que se copian en una cuenta Azure Storage. Esta opción de copia de seguridad DIY también podría ser necesaria para aquellos datos que necesita almacenar durante períodos más largos de tiempo con fines de cumplimiento y auditorías. En todos los casos, las copias de seguridad se copian en recursos compartidos NFS que se representan a través de una VM y Azure Storage.

- **Funcionalidad de copia de seguridad y restauración de infraestructura.** También puede usar la funcionalidad de copia de seguridad y restauración que proporciona la infraestructura subyacente de SAP HANA en Azure (instancias grandes). Esta opción cumple con la necesidad de copias de seguridad y restauraciones rápidas. En el resto de esta sección se trata la funcionalidad de copia de seguridad y restauración que se ofrece con las instancias grandes de HANA. Esta sección también trata sobre la relación que la copia de seguridad y la restauración tienen con la funcionalidad de recuperación ante desastres que proporciona HANA (instancias grandes).

> [!NOTE]
>   La tecnología de instantáneas que utiliza la infraestructura subyacente de HANA (instancias grandes ) depende de las instantáneas de SAP HANA. En este momento, las instantáneas de SAP HANA no funcionan en conjunción con varios inquilinos de los contenedores de base de datos multiinquilino de SAP HANA. Si solo se implementa un inquilino, las instantáneas de SAP HANA funcionan y este método puede usarse.

## <a name="use-storage-snapshots-of-sap-hana-on-azure-large-instances"></a>Uso de instantáneas de almacenamiento de SAP HANA en Azure (instancias grandes)

La infraestructura de almacenamiento subyacente de SAP HANA en Azure (instancias grandes) admite instantáneas de almacenamiento de volúmenes. Se admite tanto la copia de seguridad como la restauración de volúmenes, pero es preciso tener en cuenta lo siguiente:

- En lugar de las copias de seguridad de bases de datos completas, se toman instantáneas del volumen de almacenamiento con frecuencia.
- Cuando una instantánea se desencadena sobre los volúmenes /hana/data y /hana/shared (que incluye a /usr/sap), la tecnología de la instantánea inicia una instantánea de SAP HANA antes de ejecutar la instantánea de almacenamiento. Esta instantánea de SAP HANA es el punto de configuración de las posibles restauraciones del registro después de la recuperación de la instantánea de almacenamiento. Para que una instantánea de HANA se realice correctamente se necesita una instancia de HANA activa. En un escenario HSR, la instantánea de almacenamiento no se admite en el nodo secundario actual en el que no se puede realizar la instantánea de HANA.
- Después de que la instantánea de almacenamiento se ejecuta correctamente, se elimina la instantánea de SAP HANA.
- Las copias de seguridad del registro de transacciones se realizan con frecuencia y se almacenan en el volumen hana/logbackups o en Azure. Puede desencadenar el volumen /hana/logbackups que contiene las copias de seguridad del registro de transacciones para tomar una instantánea por separado. En ese caso, no es preciso ejecutar una instantánea de HANA.
- Si debe restaurar una base de datos a un punto determinado en el tiempo debido a una interrupción de producción, solicite que el soporte técnico de Microsoft Azure o SAP HANA en Azure realicen la restauración a una instantánea de almacenamiento determinada. Un ejemplo es una restauración planeada de un sistema de espacio aislado a su estado original.
- La instantánea de SAP HANA que se incluye en la instantánea de almacenamiento es un punto de desplazamiento para aplicar las copias de seguridad del registro de transacciones que se ejecutaron y almacenaron después de que se tomara la instantánea de almacenamiento.
- Estas copias de seguridad del registro de transacciones se realizan para restaurar la base de datos a un momento dado.

Puede realizar instantáneas de almacenamiento dirigidas a tres clases de volúmenes:

- Una instantánea combinada en /hana/data y /hana/shared, que incluye /usr/sap. Esta instantánea requiere la creación de una instantánea de SAP HANA como preparación para la instantánea de almacenamiento. La instantánea de SAP HANA garantiza que la base de datos esté en un estado coherente desde un punto de vista de almacenamiento, lo cual es un punto a considerar para el proceso de restauración.
- Una instantánea independiente en /hana/logbackups.
- Una partición del sistema operativo.

Para obtener los scripts y la documentación de instantáneas más recientes, consulte [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). Cuando descargue el paquete de scripts de instantánea de [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md), obtendrá tres archivos. Uno de los archivos está documentado en un archivo PDF para la funcionalidad proporcionada. Después de descargar el conjunto de herramientas, siga las instrucciones que aparecen en "Get the snapshot tools" (obtención de las herramientas de instantánea).

## <a name="storage-snapshot-considerations"></a>Consideraciones de las instantáneas de almacenamiento

>[!NOTE]
>Las instantáneas de almacenamiento consumen espacio de almacenamiento que está asignado a las unidades de HANA (instancias grandes). Tenga en cuenta los siguientes aspectos de la programación de instantáneas de almacenamiento y del número de instantáneas que va a conservar. 

Mecánica concreta de las instantáneas de almacenamiento de SAP HANA en Azure (instancias grandes):

- Una instantánea de almacenamiento concreta en el momento en que se toma consume poco espacio de almacenamiento.
- Cuando el contenido de datos cambia y el contenido de los archivos de datos de SAP HANA cambia en el volumen de almacenamiento, la instantánea debe almacenar el contenido del bloque original y los cambios de datos.
- El resultado es que la instantánea de almacenamiento aumenta de tamaño. Cuanto más tiempo dure la instantánea, mayor será la instantánea de almacenamiento.
- Cuantos más cambios se realicen en el volumen de la base de datos de SAP HANA a lo largo de la vigencia de una instantánea de almacenamiento, mayor será el consumo de espacio de dicha instantánea.

SAP HANA en Azure (instancias grandes) incluye tamaños de volumen fijos para los volúmenes de datos y de registro de SAP HANA. Realizar instantáneas de esos volúmenes consume el espacio del volumen. Necesita:

- Determinar cuándo se deben programar las instantáneas de almacenamiento.
- Supervisar el consumo de espacio de los volúmenes de almacenamiento. 
- Administrar el número de instantáneas que se almacenan. 

Puede deshabilitar las instantáneas de almacenamiento cuando importa masas de datos o realiza otros cambios importantes en la base de datos de HANA. 


En las secciones siguientes se proporciona información para realizar estas instantáneas y se incluyen recomendaciones generales:

- Aunque el hardware puede admitir 255 instantáneas por volumen, es conveniente no acercarse a ese número. La recomendación es usar 250 o menos.
- Antes de realizar instantáneas de almacenamiento, supervise y realice un seguimiento del espacio disponible.
- Reduzca el número de instantáneas del almacenamiento en función del espacio libre. Puede reducir el número de instantáneas que mantiene, o ampliar los volúmenes. Puede solicitar más almacenamiento en unidades de un terabyte.
- Durante actividades como mover datos a SAP HANA con herramientas de migración de la plataforma SAP (R3load) o restaurar bases de datos de SAP HANA a partir de copias de seguridad, deshabilite las instantáneas de almacenamiento en el volumen /hana/data. 
- Durante reorganizaciones mayores de tablas de SAP HANA, evite realizar instantáneas de mantenimiento cuando sea posible.
- Las instantáneas de almacenamiento son un requisito previo para sacar provecho de las funcionalidades de recuperación ante desastres de SAP HANA en Azure (instancias grandes).

## <a name="prerequisites-for-using-self-service-storage-snapshots"></a>Requisitos previos para usar instantáneas de almacenamiento de autoservicio

Para asegurarse de que el script de instantáneas se ejecuta correctamente, asegúrese de que Perl está instalado en el sistema operativo Linux en el servidor de HANA (instancias grandes). Perl viene preinstalado en la unidad de HANA (instancias grandes). Para comprobar la versión de Perl, utilice la siguiente línea de comandos:

`perl -v`

![La clave pública se copia ejecutando este comando](./media/hana-overview-high-availability-disaster-recovery/perl_screen.png)


## <a name="set-up-storage-snapshots"></a>Configuración de instantáneas de almacenamiento

Para configurar las instantáneas de almacenamiento con HANA (instancias grandes), siga estos pasos.
1. Asegúrese de que Perl está instalado en el sistema operativo Linux en el servidor de instancias grandes de HANA.
1. Modifique /etc/ssh/ssh\_config para agregar la línea _MACs hmac-sha1_.
1. Cree una cuenta de usuario de copia de seguridad de SAP HANA en el nodo maestro de cada instancia de SAP HANA que ejecute, si procede.
1. Instale el cliente HDB de SAP HANA en todos los servidores de instancias grandes de SAP HANA.
1. En el primer servidor de instancias grandes de SAP HANA de cada región, cree una clave pública para acceder a la infraestructura de almacenamiento subyacente que controla la creación de instantáneas.
1. Copie los scripts y el archivo de configuración de [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md) a la ubicación de **hdbsql** en la instalación de SAP HANA.
1. Modifique el archivo *HANABackupDetails.txt* tanto como sea necesario para ajustarse a las especificaciones apropiadas del cliente.

Obtenga los scripts de instantáneas más recientes y la documentación en [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md). Para obtener los pasos enumerados anteriormente, consulte [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

### <a name="consideration-for-mcod-scenarios"></a>Consideraciones sobre los escenarios MCOD
Si ejecuta un [escenario MCOD](https://launchpad.support.sap.com/#/notes/1681092) con varias instancias de SAP HANA en una unidad HANA (instancias grandes), puede tener volúmenes de almacenamiento independientes aprovisionados para cada una de las instancias de SAP HANA. Para obtener más información sobre MDC y otras consideraciones, consulte "Important things to remember" (cosas importantes que debe recordar) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).
 

### <a name="step-1-install-the-sap-hana-hdb-client"></a>Paso 1: Instalar el cliente de HDB de SAP HANA

El sistema operativo Linux instalado en SAP HANA en Azure (instancias grandes) incluye las carpetas y los scripts necesarios para ejecutar las instantáneas de almacenamiento de SAP HANA para la realización de copias de seguridad y para la recuperación ante desastres. Busque las versiones más recientes en [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md). 

Es responsabilidad suya instalar el cliente de HDB de SAP HANA en las unidades de HANA (instancia grande) mientras instala SAP HANA.

### <a name="step-2-change-the-etcsshssh_config"></a>Paso 2: Cambiar /etcetera/ssh/ssh\_config

Este paso se describe en "Enable communication with storage" (habilitación de la comunicación con el almacenamiento) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).


### <a name="step-3-create-a-public-key"></a>Paso 3: Crear una clave pública

Para habilitar el acceso a las interfaces de las instantáneas de almacenamiento del inquilino de HANA (instancia grande), establezca un procedimiento de inicio de sesión a través de una clave pública. 

En el primer servidor de SAP HANA en Azure (instancias grandes) del inquilino, cree una clave pública para acceder a la infraestructura de almacenamiento. Con una clave pública, no se necesita una contraseña para iniciar sesión en las interfaces de la instantánea de almacenamiento. Tampoco es necesario mantener las credenciales de contraseña con una clave pública. 

Para generar una clave pública, consulte "Enable communication with storage" (habilitación de la comunicación con el almacenamiento) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).


### <a name="step-4-create-an-sap-hana-user-account"></a>Paso 4: Crear una cuenta de usuario de SAP HANA

Para iniciar la creación de instantáneas de SAP HANA, cree una cuenta de usuario en SAP HANA que los scripts de instantánea de almacenamiento puedan usar. Cree una cuenta de usuario de SAP HANA en SAP HANA Studio para esta finalidad. El usuario debe crearse en SYSTEMDB y *no* en la base de datos de SID para MDC. En el entorno de contenedor único, el usuario se crea en la base de datos de inquilinos. Esta cuenta debe tener los privilegios **Backup Admin** (Administración de copias de seguridad) y **Catalog Read** (Lectura de catálogos). 

Para configurar y usar una cuenta de usuario, consulte "Enable communication with SAP HANA" (habilitación de la comunicación con SAP HANA) en [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).


### <a name="step-5-authorize-the-sap-hana-user-account"></a>Paso 5: Autorizar la cuenta de usuario de SAP HANA

En este paso se autoriza a la cuenta de usuario de SAP HANA que creó, de manera que los scripts no necesitan enviar contraseñas en tiempo de ejecución. El comando de SAP HANA `hdbuserstore` permite la creación de una clave de usuario de SAP HANA. Esta clave se almacena en uno o más nodos de SAP HANA. La clave de usuario permite que el usuario acceda a SAP HANA sin tener que administrar contraseñas mediante el proceso de scripting. El proceso de scripting se describe más adelante en este mismo artículo.

>[!IMPORTANT]
>Ejecute estos comandos de configuración con el mismo contexto de usuario con el que se ejecutan los comandos de instantánea. De lo contrario, los comandos de instantánea no funcionarán correctamente.


### <a name="step-6-get-the-snapshot-scripts-configure-the-snapshots-and-test-the-configuration-and-connectivity"></a>Paso 6: Obtener los scripts de instantánea, configurar las instantáneas y probar la configuración y la conectividad

Descargue la versión más reciente de los scripts de [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md). Puede cambiar la manera en que se instalan los scripts con la versión 4.1 de los scripts. Para obtener más información, consulte "Enable communication with SAP HANA" (habilitación de la comunicación con SAP HANA) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

Para ver la secuencia exacta de comandos, consulte "Easy installation of snapshot tools (default)" (instalación fácil de las herramientas de instantáneas [manera predeterminada]) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure). Se recomienda usar la instalación predeterminada. 

Para actualizar de la versión 3.x a la 4.1, consulte "Actualización de una instalación existente" en [Herramientas de instantáneas de Microsoft para SAP HANA en Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). Para desinstalar el conjunto de herramientas 4.1, consulte "Desinstalación de las herramientas de instantáneas" en [Herramientas de instantáneas de Microsoft para SAP HANA en Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

No olvide seguir los pasos descritos en "Complete setup of snapshot tools" (configuración completa de las herramientas de instantáneas) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

La finalidad de los distintos scripts y archivos instalados se describe en "What are these snapshot tools?" (¿cuáles son estas herramientas de instantáneas?). en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

Antes de configurar las herramientas de instantáneas, asegúrese también de configurar correctamente las ubicaciones y opciones de la copia de seguridad de HANA. Para obtener más información, consulte "SAP HANA Configuration" (configuración de SAP HANA) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

La configuración del conjunto de herramientas de instantáneas se describe en "Config file - HANABackupCustomerDetails.txt" (archivo de configuración: HANABackupCustomerDetails.txt) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

#### <a name="test-connectivity-with-sap-hana"></a>Prueba de conectividad con SAP HANA

Después de colocar todos los datos de configuración en el archivo *HANABackupCustomerDetails.txt* compruebe si las configuraciones son correctas para los datos de la instancia de HANA. Use el script `testHANAConnection`, que es independiente de una configuración de escalado vertical o escalado horizontal de SAP HANA.

Para obtener más información, consulte "Check connectivity with SAP HANA - testHANAConnection" (comprobación de la conectividad con SAP HANA: testHANAConnection) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

#### <a name="test-storage-connectivity"></a>Pruebas de conectividad con el almacenamiento

El paso siguiente es comprobar la conectividad con el almacenamiento según los datos que proporciona en el archivo de configuración *HANABackupCustomerDetails.txt*. A continuación, ejecute una instantánea de prueba. Debe realizar esta prueba antes de ejecutar el comando `azure_hana_backup`. Para obtener la secuencia de comandos de esta prueba, consulte "Check connectivity with storage - testStorageSnapshotConnection" (comprobación de la conectividad con el almacenamiento: testStorageSnapshotConnection) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

Después de un inicio de sesión correcto en las interfaces de la máquina virtual de almacenamiento, el script pasa a la fase 2 y crea una instantánea de prueba. Aquí se muestra el resultado de una configuración de escalado horizontal de tres nodos de SAP HANA.

Si la instantánea de prueba se ejecuta correctamente con el script, puede programar las instantáneas de almacenamiento reales. Si el resultado no es satisfactorio, investigue los problemas antes de continuar. Se debe conservar la instantánea de prueba hasta que se hayan realizado las primeras instantáneas reales.


### <a name="step-7-perform-snapshots"></a>Paso 7: Realizar instantáneas

Cuando finalicen los pasos de preparación, puede empezar a configurar y programar las instantáneas de almacenamiento reales. El script que se va a programar funciona con configuraciones de escalado vertical o escalado horizontal de SAP HANA. Para ejecutar el script de copia de seguridad de forma periódica, prográmelo mediante la utilidad cron. 

Para ver la sintaxis y funcionalidad exactas de los comandos, consulte "Perform snapshot backup - azure_hana_backup" (creación de una copia de seguridad de instantánea: azure_hana_backup) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure). 

Cuando se ejecuta el script `azure_hana_backup`, realiza las tres fases siguientes para crear la instantánea de almacenamiento:

1. Ejecuta una instantánea de SAP HANA.
1. Ejecuta una instantánea de almacenamiento.
1. Quita la instantánea de SAP HANA que se creó antes de ejecutar la instantánea de almacenamiento.

Para ejecutar el script, llámelo desde la carpeta ejecutable de HDB en la que se copió. 

El período de retención se administra con el número de instantáneas que se envían como parámetro al ejecutar el script. La cantidad de tiempo que cubren las instantáneas de almacenamiento es una función del período de ejecución y del número de instantáneas que se envían como parámetro al ejecutar el script. 

Si el número de instantáneas que se conserva supera el número en el parámetro de la llamada del script, se eliminará la instantánea de almacenamiento más antigua con la misma etiqueta antes de ejecutar una nueva instantánea. El número que proporcione como último parámetro de la llamada es el que puede usar para controlar el número de instantáneas que se conservan. Con este número también puede controlar indirectamente el espacio en disco usado para las instantáneas. 


## <a name="snapshot-strategies"></a>Estrategias de instantánea
La frecuencia de las instantáneas de los distintos tipos depende de si usa, o no, la funcionalidad de recuperación ante desastres de HANA (instancias grandes). Esta funcionalidad confía en instantáneas de almacenamiento, lo que podría requerir recomendaciones especiales en cuanto a la frecuencia y los períodos de ejecución de las instantáneas de almacenamiento. 

En las consideraciones y recomendaciones siguientes, se supone que *no* se usa la funcionalidad de recuperación ante desastres que ofrece HANA (instancias grandes). En su lugar, las instantáneas de almacenamiento se usan como medio para tener copias de seguridad y poder proporcionar la recuperación a un momento dado durante los últimos 30 días. Dadas las limitaciones del número de instantáneas y del espacio, tenga en cuenta los siguientes requisitos:

- El tiempo de recuperación para una recuperación a un momento dado.
- El espacio utilizado.
- Los objetivos de punto de recuperación y tiempo de recuperación para una posible recuperación en caso de desastre.
- La ejecución eventual de copias de seguridad completas de la base de datos de HANA en discos. Cada vez que se realiza una copia de seguridad completa de la base de datos en discos o la interfaz de **backint**, se produce un error en la ejecución de las instantáneas de almacenamiento. Si planea ejecutar copias de seguridad de la base de datos completa sobre las instantáneas del almacenamiento, asegúrese de que la ejecución de dichas instantáneas está deshabilitada durante el proceso.
- El número de instantáneas por volumen, que está limitado a 250.

<!-- backint is term for a SAP HANA interface and not a spelling error not spelling errors -->

Si no usa la funcionalidad de recuperación ante desastres de HANA (instancias grandes), el período de la instantánea es menos frecuente. En dichos casos, realice las instantáneas combinadas en /hana/data y /hana/shared (incluye que /usr/sap) en períodos de 12 o 24 horas. Conserve las instantáneas durante un mes. Lo mismo ocurre para las instantáneas del volumen de copia de seguridad del registro de transacciones. Aun así, la ejecución de las copias de seguridad del registro de transacciones de SAP HANA en el volumen de la copia de seguridad del registro tiene lugar en períodos de cinco a quince minutos.

Las instantáneas de almacenamiento programadas se realizan mejor mediante cron. Utilice el mismo script para todas las copias de seguridad y para sus necesidades de recuperación ante desastres. Modifique las entradas de script para que coincidan con los distintos tiempos de copia de seguridad solicitados. Todas estas instantáneas se programan de forma diferente en cron en función de su tiempo de ejecución. Puede ser cada hora, cada 12 horas, a diario o semanalmente. 

El siguiente ejemplo muestra una programación de cron en /etc/crontab:
```
00 1-23 * * * ./azure_hana_backup --type=hana --prefix=hourlyhana --frequency=15min --retention=46
10 00 * * *  ./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=28
00,05,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup --type=logs --prefix=regularlogback --frequency=3min --retention=28
22 12 * * *  ./azure_hana_backup --type=logs --prefix=dailylogback --frequncy=3min --retention=28
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeI --prefix=dailyboot --frequncy=15min --retention=28
```
En el ejemplo anterior, una instantánea combinada por hora abarca los volúmenes que contienen las ubicaciones /hana/data y /hana/shared (que incluye /usr/sap). Use este tipo de instantánea para una agilizar la recuperación a un momento dado en los dos últimos días. También hay una instantánea diaria en esos volúmenes. Por tanto, tiene dos días de cobertura con instantáneas por hora, más cuatro semanas de cobertura con instantáneas diarias. La copia de seguridad del volumen de la copia de seguridad del registro de transacciones también se realiza a diario. Estas copias de seguridad se conservan durante cuatro semanas. 

Como se ve en la tercera línea de crontab, la copia de seguridad del registro de transacciones de HANA está programada para ejecutarse cada cinco minutos. Las horas de inicio de los distintos trabajos de cron que ejecutan las instantáneas de almacenamiento están escalonadas. De este modo, las instantáneas no se ejecutan todas a la vez en un momento determinado. 

En el ejemplo siguiente, cada hora se realiza una instantánea combinada que abarca los volúmenes que contienen las ubicaciones /hana/data y /hana/shared/SID (que incluye /usr/sap). Las instantáneas se conservan durante dos días. Las instantáneas de los volúmenes de copia de seguridad del registro de transacciones se ejecutan cada cinco minutos y se conservan durante cuatro horas. Como antes, la copia de seguridad del archivo de registro de transacciones de HANA está programada para ejecutarse cada cinco minutos. 

La instantánea del volumen de la copia de seguridad del registro de transacciones se realiza con un retraso de 2 minutos una vez que se ha iniciado la copia de seguridad del registro de transacciones. En circunstancias normales, la copia de seguridad del registro de transacciones de SAP HANA debería finalizar en esos dos minutos. Como antes, una instantánea de almacenamiento crea una copia de seguridad al día del volumen que contiene el LUN de arranque y se conserva durante cuatro semanas.

```
10 0-23 * * * ./azure_hana_backup --type=hana ==prefix=hourlyhana --frequency=15min --retention=48
0,5,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup --type=logs --prefix=regularlogback --frequency=3min --retention=28
2,7,12,17,22,27,32,37,42,47,52,57 * * * *  ./azure_hana_backup --type=logs --prefix=logback --frequency=3min --retention=48
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeII --prefix=dailyboot --frequency=15min --retention=28
```

En el siguiente gráfico se ilustran las secuencias del ejemplo anterior. No está incluido el LUN de arranque.

![Relación entre copias de seguridad e instantáneas](./media/hana-overview-high-availability-disaster-recovery/backup_snapshot_updated0921.PNG)

SAP HANA realiza escrituras normales en el volumen /hana/log para documentar los cambios confirmados en la base de datos. De manera regular, SAP HANA escribe un punto de retorno en el volumen /hana/data. Tal como se especifica en crontab, se ejecuta una copia de seguridad del registro de transacciones de SAP HANA cada cinco minutos. 

También se ve que una instantánea de SAP HANA se ejecuta cada hora como resultado del desencadenamiento de una instantánea de almacenamiento combinada sobre los volúmenes /hana/data y /hana/shared/SID. Una vez que la instantánea de HANA se realiza correctamente, se ejecuta la instantánea de almacenamiento combinada. Tal como se indica en crontab, la instantánea de almacenamiento del volumen /hana/logbackup se ejecuta cada cinco minutos, aproximadamente dos minutos después de la copia de seguridad del registro de transacciones de HANA.

> 

>[!IMPORTANT]
> El uso de instantáneas de almacenamiento para las copias de seguridad de SAP HANA solo resulta valioso cuando las instantáneas se realizan junto con las copias de seguridad del registro de transacciones de SAP HANA. Es preciso que dichas copias de seguridad del registro de transacciones cubran los períodos entre las instantáneas de almacenamiento. 

Si ha adquirido un compromiso a los usuarios de una recuperación a un momento dado de 30 días, necesita lo siguiente:

- Acceda a una instantánea de almacenamiento combinada en /hana/data y /hana/shared/SID que tenga una antigüedad de 30 días en casos extremos. 
- Tenga copias de seguridad del registro de transacciones contiguas que cubran el tiempo entre cualquiera de las instantáneas del almacenamiento combinado. Por tanto, la instantánea de mayor antigüedad del volumen de copia de seguridad del registro de transacciones debe tener 30 días. No es este el caso si copia las copias de seguridad del registro de transacciones en otro recurso compartido NFS ubicado en Azure Storage. En ese caso, podría extraer las copias de seguridad del registro de transacciones anteriores de dicho recurso compartido.

Para beneficiarse de las instantáneas de almacenamiento y la eventual replicación de almacenamiento de las copias de seguridad del registro de transacciones, cambie la ubicación en la que SAP HANA escribe las copias de seguridad del registro de transacciones. Puede realizar este cambio en HANA Studio. 

Aunque SAP HANA crea automáticamente copias de seguridad de segmentos de registros completos, especifique un intervalo de copia de seguridad de registros para que sea determinista. Esto debe hacerse así especialmente cuando se usa la opción de recuperación ante desastres, ya que lo habitual es que desee ejecutar las copias de seguridad del registro de transacciones con un período determinista. En el caso siguiente, se estableció un intervalo de copia de seguridad de registros de 15 minutos.

![Programe los registros de copia de seguridad de SAP HANA en SAP HANA Studio](./media/hana-overview-high-availability-disaster-recovery/image5-schedule-backup.png)

También puede optar por que las copias de seguridad se realicen con una frecuencia superior a 15 minutos. A menudo este valor se usa en conjunción con la funcionalidad de recuperación ante desastres de HANA (instancias grandes). Algunos clientes realizan copias de seguridad del registro de transacciones cada 5 minutos.

Si nunca se realizó una copia de seguridad de la base de datos, el último paso es crear una copia de seguridad en archivo para crear una entrada de copia de seguridad individual que debe existir en el catálogo de copias de seguridad. En caso contrario, SAP HANA no puede iniciar las copias de seguridad del registro de transacciones especificadas.

![Realizar una copia de seguridad en archivo para crear una entrada de copia de seguridad individual](./media/hana-overview-high-availability-disaster-recovery/image6-make-backup.png)


Después de ejecutar correctamente las primeras instantáneas de almacenamiento, puede eliminar la instantánea de prueba que se ejecutó en el paso 6. Para obtener más información, consulte "Remove test snapshots - removeTestStorageSnapshot" (eliminación de las instantáneas de prueba: removeTestStorageSnapshot) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure). 


### <a name="monitor-the-number-and-size-of-snapshots-on-the-disk-volume"></a>Supervisión del número y tamaño de instantáneas en el volumen de disco

En un volumen de almacenamiento concreto se puede supervisar tanto el número de instantáneas como el consumo de almacenamiento de dichas instantáneas. El comando `ls` no muestra los archivos o el directorio de instantáneas. El comando `du` del sistema operativo Linux muestra detalles sobre esas instantáneas de almacenamiento debido a que se almacenan en los mismos volúmenes. Utilice el comando con las siguientes opciones:

- `du –sh .snapshot`: esta opción proporciona el total de todas las instantáneas del directorio de instantáneas.
- `du –sh --max-depth=1`: esta opción enumera todas las instantáneas que se guardan en la carpeta **.snapshot** y el tamaño de cada una de ellas.
- `du –hc`: esta opción proporciona el tamaño total que usan todas las instantáneas.

Utilice estos comandos para asegurarse de que las instantáneas tomadas y almacenadas no consumen todo el almacenamiento de los volúmenes.

>[!NOTE]
>Las instantáneas del LUN de arranque no son visibles con los comandos anteriores.

### <a name="get-details-of-snapshots"></a>Obtención de detalles de las instantáneas
Para obtener más detalles sobre las instantáneas, utilice el script `azure_hana_snapshot_details`. Puede ejecutar este script en cualquier ubicación si hay un servidor activo en la ubicación de recuperación ante desastres. El script proporciona la siguiente salida desglosada por cada volumen que contiene instantáneas: 
   * El tamaño de las instantáneas totales de un volumen
   * Los siguientes detalles de cada instantánea del volumen: 
      - Nombre de la instantánea 
      - Hora de creación 
      - Tamaño de la instantánea
      - Frecuencia de la instantánea
      - Identificador de copia de seguridad de HANA asociado con esa instantánea, si procede

Para conocer la sintaxis del comando y sus salidas, consulte "List snapshots - azure_hana_snapshot_details" (enumeración de instantáneas: azure_hana_snapshot_details) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure). 



### <a name="reduce-the-number-of-snapshots-on-a-server"></a>Reducción del número de instantáneas en un servidor

Como se explicó anteriormente, es posible reducir el número de instantáneas en etiquetas concretas almacenadas. Los dos últimos parámetros del comando para iniciar una instantánea son la etiqueta y el número de instantáneas que se desean conservar.

```
./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=28
```

En el ejemplo anterior, la etiqueta de instantánea es **dailyhana**. El número de instantáneas con esta etiqueta que va a conservarse es **28**. A medida que responda al consumo de espacio en el disco es posible que desee reducir el número de instantáneas que se almacenan. Una manera sencilla de reducir el número de instantáneas a 15, por ejemplo, es ejecutar el script con el último parámetro establecido en **15**:

```
./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=15
```

Si ejecuta el script con la configuración anterior, el número de instantáneas se reduce a 15, y en él se incluye a la nueva instantánea de almacenamiento. Se conservan las 15 instantáneas más recientes y se eliminan las 15 más antiguas.

 >[!NOTE]
 > Este script reduce el número de instantáneas solo si hay instantáneas que tienen más de una hora de antigüedad. Este script no elimina las instantáneas que tienen menos de una hora de antigüedad. Estas restricciones están relacionadas con la funcionalidad opcional de recuperación ante desastres que se ofrece.

Si ya no quiere conservar un conjunto de instantáneas con el prefijo de copia de seguridad **dailyhana** en los ejemplos de sintaxis, puede ejecutar el script con **0** como número de retención. Todas las instantáneas que coinciden con esa etiqueta se eliminan. La eliminación de todas las instantáneas puede afectar a las funcionalidades de recuperación ante desastres de HANA (instancias grandes).

Otra opción para eliminar instantáneas específicas es usar el script `azure_hana_snapshot_delete`. Este script está diseñado para eliminar una instantánea o un conjunto de instantáneas, ya sea mediante el identificador de la copia de seguridad de HANA como aparece en HANA Studio, o bien a través del nombre mismo de la instantánea. Actualmente, el identificador de la copia de seguridad solo está asociado a las instantáneas creadas para el tipo de instantánea **hana**. Las copias de seguridad de las instantáneas del tipo **logs** y **boot** no realizan una instantánea de SAP HANA, por lo que no se puede encontrar un identificador de copia de seguridad para dichas instantáneas. Si se escribe el nombre de la instantánea, busca todas las instantáneas en los distintos volúmenes que coincidan con el nombre de instantánea que se escribió. 

<!-- hana, logs and boot are no spelling errors as Acrolinx indicates, but terms of parameter values -->

Para obtener más información sobre el script, consulte "Delete a snapshot - azure_hana_snapshot_delete" (eliminación de una instantánea: azure_hana_snapshot_delete) en [Microsoft snapshot tools for SAP HANA on Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (herramientas de instantáneas de Microsoft para SAP HANA en Azure).

Ejecute el script como usuario **raíz**.

>[!IMPORTANT]
>Si hay datos que existen solo en la instantánea que planea eliminar, cuando la instantánea se elimine se perderán para siempre.


## <a name="file-level-restore-from-a-storage-snapshot"></a>Restauración de nivel de archivo desde una instantánea de almacenamiento

<!-- hana, logs and boot are no spelling errors as Acrolinx indicates, but terms of parameter values -->
En el caso de los tipos de instantánea **hana** y **logs**, es posible acceder a las instantáneas directamente en los volúmenes del directorio **.snapshot**. Hay un subdirectorio para cada una de las instantáneas. Copie cada archivo en el estado en que estaba al momento de realizar la instantánea desde ese subdirectorio a la estructura de directorios real. 

En la versión actual del script, *no* se proporciona ningún script para la restauración de autoservicio de las instantáneas. La restauración de instantáneas puede realizarse como parte de los scripts de recuperación ante desastres de autoservicio en el sitio de recuperación ante desastres durante la conmutación por error. Para restaurar la instantánea deseada de entre las instantáneas disponibles existentes, debe ponerse en contacto con el equipo de operaciones de Microsoft mediante la apertura de una solicitud de servicio.

>[!NOTE]
>Una sola restauración de archivos no funciona con las instantáneas del LUN de arranque, que independientemente del tipo de unidades de HANA (instancias grandes). El directorio **.snapshot** no se expone en el LUN de arranque. 
 

## <a name="recover-to-the-most-recent-hana-snapshot"></a>Recuperación a la instantánea más reciente de HANA

En un escenario de bajada de producción, el proceso de recuperación a partir de una instantánea de almacenamiento se puede iniciar como un incidente de cliente en el Soporte técnico de Microsoft Azure. Sería una cuestión muy urgente si los datos se eliminaran en un sistema de producción y la única forma de volver a obtenerlos fuera a través de la restauración de la base de datos de producción.

En otra situación, una recuperación a un momento dado podría ser poco urgente y se podría planear con días de antelación. Para planearla, se puede usar SAP HANA en Azure, en lugar de generar una marca de alta prioridad. Por ejemplo, puede planear la actualización del software de SAP mediante la aplicación de un nuevo paquete de mejora. A continuación, necesita volver a una instantánea que representa el estado antes de la actualización del paquete mejora.

Antes de enviar la solicitud, necesita prepararse. Después, el equipo de SAP HANA en Azure puede encargarse de la solicitud y proporcionar los volúmenes restaurados. A partir de ese momento, restaurará la base de datos de HANA con las instantáneas.

Para conocer las posibilidades de restaurar una instantánea con el nuevo conjunto de herramientas, consulte "How to restore a snapshot" (procedimiento para restaurar una instantánea) en [Manual recovery guide for SAP HANA on Azure from a storage snapshot](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (guía de restauración manual para SAP HANA en Azure a partir de una instantánea de almacenamiento).

Como preparación para la solicitud, siga estos pasos.

1. Decida qué instantánea desea restaurar. Salvo que se indique lo contrario, solo se restaura el volumen de datos/HANA. 

1. Cierre la instancia de HANA.

   ![Cerrar la instancia de HANA](./media/hana-overview-high-availability-disaster-recovery/image7-shutdown-hana.png)

1. Desmonte los volúmenes de datos de todos los nodos de la base de datos de HANA. Si los volúmenes de datos siguen montados en el sistema operativo, se produce un error en la restauración de la instantánea.

   ![Desmontar los volúmenes de datos de todos los nodos de la base de datos de HANA](./media/hana-overview-high-availability-disaster-recovery/image8-unmount-data-volumes.png)

1. Abra una solicitud de soporte técnico de Azure e incluya instrucciones acerca de la restauración de una instantánea concreta:

   - Durante la restauración: SAP HANA en Azure Service podría pedirle que asista a una llamada de conferencia para coordinar, comprobar y confirmar que se restaure la instantánea de almacenamiento correcta. 

   - Después de la restauración: SAP HANA en Azure Service le enviará una notificación cuando la instantánea del almacenamiento esté restaurada.

1. Una vez que el proceso de restauración se haya completado, vuelva a montar todos los volúmenes de datos.

   ![Volver a montar todos los volúmenes de datos](./media/hana-overview-high-availability-disaster-recovery/image9-remount-data-volumes.png)



Otra posibilidad de restaurar, por ejemplo, archivos de datos de SAP HANA a partir de una instantánea de almacenamiento se documentó en el paso 7 de [Manual recovery guide for SAP HANA on Azure from a storage snapshot](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (guía de restauración manual para SAP HANA en Azure a partir de una instantánea de almacenamiento).

Para realizar una restauración a partir de una copia de seguridad de instantánea, consulte [Manual recovery guide for SAP HANA on Azure from a storage snapshot](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (guía de restauración manual para SAP HANA en Azure a partir de una instantánea de almacenamiento). 

>[!Note]
>Si las operaciones de Microsoft restauraron su instantánea, no es necesario realizar el paso 7.


### <a name="recover-to-another-point-in-time"></a>Recuperación a otro momento dado
Para realizar una restauración a un momento dado, consulte "Recover the database to the following point in time" (recuperación de la base de datos al siguiente momento dado) en [Manual recovery guide for SAP HANA on Azure from a storage snapshot](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md) (guía de restauración manual para SAP HANA en Azure a partir de una instantánea de almacenamiento). 





## <a name="snapcenter-integration-in-sap-hana-large-instances"></a>Integración de SnapCenter en instancias grandes de SAP HANA

En esta sección se describe cómo pueden los clientes usar el software SnapCenter de NetApp para tomar una instantánea, hacer una copia de seguridad de bases de datos de SAP HANA hospedadas en HANA (instancias grandes) de Microsoft Azure y restaurarlas. 

SnapCenter ofrece soluciones para escenarios como copia de seguridad y recuperación, recuperación ante desastres (DR) con replicación de almacenamiento asincrónica, replicación del sistema y clonación del sistema. Integrado con SAP HANA (instancias grandes) en Azure, los clientes ahora pueden usar SnapCenter para las operaciones de copia de seguridad y recuperación.

Para obtener referencias adicionales, consulte NetApp TR-4614 y TR-4646 en SnapCenter.

- [Copia de seguridad y restauración de SAP HANA con SnapCenter (TR-4614)](https://www.netapp.com/us/media/tr-4614.pdf)
- [Recuperación ante desastres con replicación de almacenamiento de SAP HANA (TR-4646)](https://www.netapp.com/us/media/tr-4646.pdf)
- [HSR de SAP HANA con SnapCenter (TR-4719)](https://www.netapp.com/us/media/tr-4719.pdf)
- [Clonación de SAP desde SnapCenter (TR-4667)](https://www.netapp.com/us/media/tr-4667.pdf)

### <a name="system-requirements-and-prerequisites"></a>Requisitos del sistema y requisitos previos

Para ejecutar SnapCenter en Azure HLI, los requisitos del sistema incluyen:
* SnapCenter Server en Azure Windows 2016 o posterior con 4-vCPU, 16 GB de RAM y un mínimo de 650 GB de almacenamiento SSD prémium administrado.
* Sistema SAP HANA (instancias grandes) con 1,5 TB – 24 TB de RAM. Se recomienda usar dos sistemas de SAP HANA (instancias grandes) para las operaciones y pruebas de clonación.

Los pasos para integrar SnapCenter en SAP HANA son los siguientes: 

1. Genere una solicitud de incidencia de soporte técnico para comunicar la clave pública generada por el usuario al equipo de Microsoft Ops. Esto es necesario para configurar el usuario de SnapCenter de modo que tenga acceso al sistema de almacenamiento.
1. Cree una máquina virtual en la red virtual que tenga acceso a HLI; esta máquina virtual se usa para SnapCenter. 
1. Descargue e instale SnapCenter. 
1. Lleve a cabo operaciones de copia de seguridad y recuperación. 

### <a name="create-a-support-ticket-for-user-role-storage-setup"></a>Creación de una incidencia de soporte técnico para la configuración del almacenamiento del rol de usuario

1. Abra Azure Portal y vaya a la página **Suscripciones**. Una vez en la página "Suscripciones", seleccione su suscripción de SAP HANA, destacada en rojo a continuación.

   :::image type="content" source="./media/snapcenter/create-support-case-for-user-role-storage-setup.png" alt-text="Creación de incidencia de soporte técnico para la configuración del almacenamiento de usuarios":::

1. En la página de la suscripción de SAP HANA, seleccione la subpágina **Grupos de recursos**.

   :::image type="content" source="./media/snapcenter/solution-lab-subscription-resource-groups.png" alt-text="Grupo de recursos de suscripción de laboratorio de soluciones" lightbox="./media/snapcenter/solution-lab-subscription-resource-groups.png":::

1. Seleccione un grupo de recursos adecuado en una región.

   :::image type="content" source="./media/snapcenter/select-appropriate-resource-group-in-region.png" alt-text="Selección del grupo de recursos adecuado en una región" lightbox="./media/snapcenter/select-appropriate-resource-group-in-region.png":::

1. Seleccione una entrada de SKU correspondiente a SAP HANA en Azure Storage.

   :::image type="content" source="./media/snapcenter/select-sku-entry-corresponding-to-sap-hana.png" alt-text="Selección de la entrada de SKU correspondiente a SAP HANA" lightbox="./media/snapcenter/select-sku-entry-corresponding-to-sap-hana.png":::

1. Abra una **nueva incidencia de soporte técnico**, destacada en rojo.

   :::image type="content" source="./media/snapcenter/open-new-support-ticket-request.png" alt-text="Apertura de la solicitud de incidencia de soporte técnico":::

1. En la pestaña **Básico**, especifique la siguiente información:

   * **Tipo de problema**: Requisitos previos técnicos
   * **Subscription** (Suscripción): Su suscripción
   * **Servicio:** Instancia grande de SAP HANA
   * **Recurso:** Su grupo de recursos
   * **Resumen:** Proporcionar la clave pública generada por el usuario
   * **Tipo de problema**: Instalación y configuración
   * **Subtipo de problema:** Configuración de SnapCenter para HLI


1. En la **descripción** de la incidencia de soporte técnico, en la pestaña **Detalles**, proporcione la siguiente información: 
   
   * Configuración de SnapCenter para HLI
   * Su clave pública para el usuario de SnapCenter (SnapCenter.pem); consulte el ejemplo de creación de clave pública siguiente.

     :::image type="content" source="./media/snapcenter/new-support-request-details.png" alt-text="Nueva solicitud de soporte técnico - pestaña Detalles" lightbox="./media/snapcenter/new-support-request-details.png":::

1. Seleccione **Revisar y crear** para revisar la incidencia de soporte técnico. 

1. Genere un certificado para el nombre de usuario de SnapCenter en la instancia grande de HANA o en cualquier servidor Linux.

   SnapCenter requiere un nombre de usuario y una contraseña para acceder a la máquina virtual de almacenamiento (SVM) y crear instantáneas de la base de datos de HANA. Microsoft usa la clave pública para que permitir que usted (el cliente) establezca la contraseña para tener acceso al sistema de almacenamiento.

   ```bash
   openssl req -x509 -nodes -days 1095 -newkey rsa:2048 -keyout snapcenter.key -out snapcenter.pem -subj "/C=US/ST=WA/L=BEL/O=NetApp/CN=snapcenter"
   Generating a 2048 bit RSA private key
   ................................................................................................................................................+++++
   ...............................+++++
   writing new private key to 'snapcenter.key'
   -----

   sollabsjct31:~ # ls -l cl25*
   -rw-r--r-- 1 root root 1704 Jul 22 09:59 snapcenter.key
   -rw-r--r-- 1 root root 1253 Jul 22 09:59 snapcenter.pem

   ```

1. Adjunte el archivo snapcenter.pem a la incidencia de soporte técnico y, a continuación, seleccione **Crear**.

   Una vez que se envía el certificado de clave pública, Microsoft configura el nombre de usuario de SnapCenter para el inquilino junto con la dirección IP de SVM.   

1. Después de recibir la dirección IP de SVM, establezca una contraseña para acceder a SVM, que usted controla.

   El siguiente es un ejemplo de la llamada de REST (documentación) desde la instancia grande de HANA o la máquina virtual de la red virtual, que tiene acceso a un entorno de instancia grande de HANA y que se usará para establecer la contraseña.

   ```bash
   curl --cert snapcenter.pem --key snapcenter.key -X POST -k "https://10.0.40.11/api/security/authentication/password" -d '{"name":"snapcenter","password":"test1234"}'
   ```

   Asegúrese de que no hay ninguna variable de proxy activa en el sistema HANA DB.

   ```bash
   sollabsjct31:/tmp # unset http_proxy
   sollabsjct31:/tmp # unset https_proxy
   ```

### <a name="download-and-install-snapcenter"></a>Descarga e instalación de SnapCenter
Ahora que el nombre de usuario está configurado para el acceso de SnapCenter al sistema de almacenamiento, usará el nombre de usuario de SnapCenter para configurar SnapCenter una vez instalado. 

Antes de instalar SnapCenter, revise [Copia de seguridad y restauración de SAP HANA con SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf) para definir la estrategia de copia de seguridad. 

1. Inicie sesión en [NetApp](https://mysupport.netapp.com) para [descargar](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fmysupport.netapp.com%2Fsite%2Fproducts%2Fall%2Fdetails%2Fsnapcenter%2Fdownloads-tab&data=02%7C01%7Cmadhukan%40microsoft.com%7Ca53f5e2f245a4e36933008d816efbb54%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637284566603265503&sdata=TOANWNYoAr1q5z1opu70%2FUDPHjluvovqR9AKplYpcpk%3D&reserved=0) la versión más reciente de SnapCenter.

1. Instale SnapCenter en la máquina virtual de Azure en Windows.

   El instalador comprueba los requisitos previos de la máquina virtual. 

   >[!IMPORTANT]
   >Preste atención al tamaño de la máquina virtual, especialmente en entornos de mayor tamaño.

1. Configure las credenciales de usuario para SnapCenter. De forma predeterminada, utiliza las credenciales de usuario de Windows que se usan para instalar la aplicación. 

   :::image type="content" source="media/snapcenter/installation-user-inputs-dialog.png" alt-text="Cuadro de diálogo de entradas del usuario de la instalación"::: 

1. Cuando inicie la sesión, guarde la exención de seguridad y se iniciará la GUI.

1. Inicie sesión en SnapCenter en la máquina virtual (https://snapcenter-vm:8146) con las credenciales de Windows para configurar el entorno.


### <a name="set-up-the-storage-system"></a>Configuración del sistema de almacenamiento

1. En SnapCenter, seleccione **Storage System** (Sistema de almacenamiento) y, después, seleccione **+ New** (Nuevo). 

   :::image type="content" source="./media/snapcenter/snapcenter-storage-connections-window.png" alt-text="Conexiones de almacenamiento SnapCenter" lightbox="./media/snapcenter/snapcenter-storage-connections-window.png":::

   El valor predeterminado es un SVM por inquilino. Si un cliente tiene varios inquilinos o HLI en varias regiones, se recomienda configurar todos los SVM en SnapCenter.

1. En Add Storage System (Agregar sistema de almacenamiento), proporcione la información del sistema de almacenamiento que desea agregar, el nombre de usuario y la contraseña de SnapCenter y, a continuación, seleccione **Submit** (Enviar).

   :::image type="content" source="./media/snapcenter/new-storage-connection.png" alt-text="Nueva conexión de almacenamiento":::

   >[!NOTE]
   >El valor predeterminado es un SVM por inquilino.  Si hay varios inquilinos, se recomienda configurar todos los SVM aquí en SnapCenter. 

1. En SnapCenter, seleccione **Hosts** y seleccione **+Add** (Agregar) para configurar el complemento de HANA y los hosts de HANA DB.  La versión más reciente de SnapCenter detecta automáticamente la base de datos de HANA en el host.

   :::image type="content" source="media/snapcenter/managed-hosts-new-host.png" alt-text="En SnapCenter, seleccione Hosts y, a continuación, seleccione Add (Agregar)" lightbox="media/snapcenter/managed-hosts-new-host.png":::.

1. Proporcione la información para el nuevo host:
   1. Seleccione el sistema operativo para el tipo de host.
   1. Escriba el nombre de host de la máquina virtual de SnapCenter.
   1. Proporcione las credenciales que desea usar.
   1. Seleccione las opciones **Microsoft Windows** and **SAP HANA** y, luego, seleccione **Submit** (Enviar).

   :::image type="content" source="media/snapcenter/add-new-host-operating-system-credentials.png" alt-text="Información para el nuevo host":::

   >[!IMPORTANT]
   >Antes de poder instalar el primer nodo, SnapCenter permite a un usuario no raíz instalar complementos en la base de datos.  Para obtener información sobre cómo habilitar un usuario no raíz, consulte [Adición de un usuario no raíz y configuración de los privilegios de sudo](https://library.netapp.com/ecmdocs/ECMLP2590889/html/GUID-A3EEB5FC-242B-4C2C-B407-510E48A8F131.html).

1. Revise los detalles del host y seleccione **Submit** (Enviar) para instalar el complemento en el servidor SnapCenter.

1. Una vez instalado el complemento, en SnapCenter, seleccione **Hosts** y, después, seleccione **+ Add** (Agregar) para agregar un nodo de HANA.

   :::image type="content" source="media/snapcenter/add-hana-node.png" alt-text="Adición de un nodo de HANA" lightbox="media/snapcenter/add-hana-node.png":::

1. Proporcione la información para el nodo de HANA:
   1. Seleccione el sistema operativo para el tipo de host.
   1. Escriba el nombre de host o la dirección IP de HANA DB.
   1. Seleccione **+** para agregar las credenciales configuradas en el sistema operativo host de HANA DB y, a continuación, seleccione **OK** (Aceptar).
   1. Seleccione **SAP HANA** y, a continuación, seleccione **Submit** (Enviar).

   :::image type="content" source="media/snapcenter/add-hana-node-details.png" alt-text="Detalles del nodo de SAP HANA":::

1. Confirme la huella digital y seleccione **Confirm and Submit** (Confirmar y enviar).

   :::image type="content" source="media/snapcenter/confirm-submit-fingerprint.png" alt-text="Confirmación y envío de huella digital":::

1. En el nodo de HANA, en la base de datos del sistema, seleccione **Security**(Seguridad) > **Users**(Usuarios) > **SNAPCENTER** para crear el usuario de SnapCenter.

   :::image type="content" source="media/snapcenter/create-snapcenter-user-hana-system-db.png" alt-text="Creación del usuario de SnapCenter en HANA (system db)":::



### <a name="auto-discovery"></a>Detección automática
SnapCenter 4.3 habilita la función de detección automática de forma predeterminada.  La detección automática no se admite para las instancias de HANA con la replicación del sistema de HANA (HSR) configurada. Debe agregar manualmente la instancia al servidor de SnapCenter.


### <a name="hana-setup-manual"></a>Instalación de HANA (manual)
Si configuró HSR, debe configurar el sistema manualmente.  

1. En SnapCenter, seleccione **Resources** (Recursos) y **SAP HANA** (en la parte superior) y, a continuación, seleccione **+Add SAP HANA Database** (+Agregar base de datos de SAP HANA) (a la derecha).

   :::image type="content" source="media/snapcenter/manual-hana-setup.png" alt-text="Configuración manual de HANA" lightbox="media/snapcenter/manual-hana-setup.png":::

1. Especifique los detalles de los recursos del usuario administrador de HANA configurado en el host de Linux o en el host donde están instalados los complementos. La copia de seguridad se administrará desde el complemento en el sistema Linux.

   :::image type="content" source="media/snapcenter/provide-resource-details-sap-hana-database.png" alt-text="Especifique los detalles de recursos del usuario administrador de HANA configurado en el host de Linux":::.

1. Seleccione el volumen de datos para el que necesita tomar instantáneas, seleccione **Save** (Guardar) y, después, seleccione **Finish** (Finalizar).

   :::image type="content" source="media/snapcenter/provide-storage-footprint.png" alt-text="Seleccione el volumen de datos para el que necesita tomar instantáneas, seleccione Save (Guardar) y, a continuación, seleccione Finish (Finalizar).":::

### <a name="create-a-snapshot-policy"></a>Creación de una directiva de instantánea

Antes de usar SnapCenter para realizar una copia de seguridad de los recursos de base de datos de SAP HANA, debe crear una directiva de copia de seguridad para el recurso o el grupo de recursos del que desea realizar la copia de seguridad. Durante el proceso de creación de una directiva de instantáneas, se le ofrecerá la opción de configurar comandos anteriores o posteriores y claves SSL especiales. Para obtener información sobre cómo crear una directiva de instantáneas, vea [Creación de directivas de copia de seguridad para bases de datos SAP HANA](http://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-sap-hana%2FGUID-246C0810-4F0B-4BF7-9A35-B729AD69954A.html).

1. En SnapCenter, seleccione **Resources** (Recursos) y, a continuación, seleccione una base de datos.

   :::image type="content" source="media/snapcenter/select-database-create-policy.png" alt-text="En SnapCenter, seleccione Resources (Recursos) y, después, seleccione una base de datos.":::

1. Siga el flujo de trabajo del Asistente para configuración para configurar el programador de instantáneas.

   :::image type="content" source="media/snapcenter/follow-workflow-configuration-wizard.png" alt-text="Siga el flujo de trabajo del Asistente para configuración para configurar el programador de instantáneas" lightbox="media/snapcenter/follow-workflow-configuration-wizard.png":::.

1. Proporcione las opciones para configurar comandos anteriores o posteriores y claves SSL especiales.  En este ejemplo, no se usa ninguna configuración especial.

   :::image type="content" source="media/snapcenter/configuration-options-pre-post-commands.png" alt-text="Proporcione las opciones para configurar comandos anteriores o posteriores y claves SSL especiales" lightbox="media/snapcenter/configuration-options-pre-post-commands.png":::.

1. Seleccione **Add** (Agregar) para crear una directiva de instantáneas, que también se puede usar para otras bases de datos de HANA. 

   :::image type="content" source="media/snapcenter/select-one-or-more-policies.png" alt-text="Seleccione Add (Agregar) para crear una directiva de instantáneas, que también se puede usar para otras bases de datos de HANA":::.

1. Escriba el nombre de la directiva y una descripción.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy.png" alt-text="Escriba el nombre de la directiva y una descripción":::.


1. Seleccione el tipo y la frecuencia de la copia de seguridad.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-settings.png" alt-text="Seleccione el tipo y la frecuencia de la copia de seguridad":::.

1. Establezca los parámetros de **On demand backup retention settings** (Configuración de la retención de copia de seguridad a petición).  En nuestro ejemplo, vamos a configurar la retención en la conservación de tres copias de instantáneas.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-retention-settings.png" alt-text="Establezca los parámetros de On demand backup retention settings (Configuración de la retención de copia de seguridad a petición)":::.

1. Configure los parámetros de **Hourly retention settings** (Configuración de la retención por horas). 

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-hourly-retention-settings.png" alt-text="Configure los parámetros de Hourly retention settings (Configuración de la retención por horas)":::.

1. Si se configura una instalación de SnapMirror, seleccione **Update SnapMirror after creating a local SnapShot copy** (Actualizar SnapMirror después de crear una copia de SnapShot local).

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-snapmirror.png" alt-text="Si se requiere SnapMirror, seleccione Update SnapMirror after creating a local SnapShot copy (Actualizar SnapMirror después de crear una copia de SnapShot local)":::.

1. Seleccione **Finish** (Finalizar) para revisar el resumen de la nueva directiva de copia de seguridad. 
1. En **Configure Schedule** (Configurar programación), seleccione **Add** (Agregar).

   :::image type="content" source="media/snapcenter/configure-schedules-for-selected-policies.png" alt-text="En Configure Schedule (Configurar programación), seleccione Add (Agregar)":::.

1. Seleccione los valores de **Start date** (Fecha de inicio), **Expires on** (Fecha de caducidad) y la frecuencia.

   :::image type="content" source="media/snapcenter/add-schedules-for-policy.png" alt-text="Seleccione los valores de Start date (Fecha de inicio), Expires on (Fecha de caducidad) y la frecuencia.":::

1. Proporcione los detalles del correo electrónico para las notificaciones.

   :::image type="content" source="media/snapcenter/backup-policy-notification-settings.png" alt-text="Proporcione los detalles del correo electrónico para las notificaciones.":::

1.  Seleccione **Finish** (Finalizar) para crear la directiva de copia de seguridad.

### <a name="disable-ems-message-to-netapp-autosupport"></a>Deshabilitación del mensaje de EMS para el servicio de soporte técnico automático de NetApp
De forma predeterminada, la recopilación de datos de EMS está habilitada y se ejecuta cada siete días después de la fecha de instalación.  Puede deshabilitar la recopilación de datos con el cmdlet de PowerShell `Disable-SmDataCollectionEms`.

1. En PowerShell, establezca una sesión con SnapCenter.

   ```powershell
   Open-SmConnection
   ```

1. Inicie sesión con sus credenciales.
1. Deshabilite la recopilación de mensajes de EMS.

   ```powershell
   Disable-SmCollectionEms
   ```

### <a name="restore-database-after-crash"></a>Restauración de la base de datos tras bloqueo
Puede usar SnapCenter para restaurar la base de datos.  En esta sección, trataremos los pasos de alto nivel, pero para obtener más información, consulte [Copia de seguridad y restauración de SAP HANA con SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf).


1. Detenga la base de datos y elimine todos los archivos de base de datos.

   ```
   su - h31adm
   > sapcontrol -nr 00 -function StopSystem
   StopSystem
   OK
   > sapcontrol -nr 00 -function GetProcessList
   OK
   name, description, dispstatus, textstatus, starttime, elapsedtime, pid
   hdbdaemon, HDB Daemon, GRAY, Stopped, , , 35902
 
   ```

1. Desmonte el volumen de la base de datos.

   ```bash
   unmount /hana/data/H31/mnt00001
   ```


1. Restaure los archivos de base de datos a través de SnapCenter.  Seleccione la base de datos y, a continuación, seleccione **Restore** (Restaurar).  

   :::image type="content" source="media/snapcenter/restore-database-via-snapcenter.png" alt-text="Seleccione la base de datos y, a continuación, seleccione Restore (Restaurar)" lightbox="media/snapcenter/restore-database-via-snapcenter.png":::.

1. Seleccione el tipo de restauración.  En nuestro ejemplo, vamos a restaurar el recurso completo. 

   :::image type="content" source="media/snapcenter/restore-database-select-restore-type.png" alt-text="Seleccione el tipo de restauración":::.

   >[!NOTE]
   >Con una configuración predeterminada, no es necesario especificar comandos para realizar una restauración local desde la instantánea en el disco. 

   >[!TIP]
   >Si desea restaurar un LUN determinado dentro del volumen, seleccione **File Level** (Nivel de archivo).

1. Siga el flujo de trabajo a través del Asistente para configuración.
   
   SnapCenter restaura los datos en la ubicación original para que pueda iniciar el proceso de restauración en HANA. Además, dado que SnapCenter no puede modificar el catálogo de copias de seguridad (la base de datos está inactiva), se muestra una advertencia.

   :::image type="content" source="media/snapcenter/restore-database-job-details-warning.png" alt-text="Dado que SnapCenter no puede modificar el catálogo de copias de seguridad, se muestra una advertencia":::.

1. Como se restauran todos los archivos de base de datos, inicie el proceso de restauración en HANA. En HANA Studio, en **Systems** (Sistemas), haga clic con el botón secundario en la base de datos del sistema y seleccione **Backup and Recovery** (Copia de seguridad y recuperación) > **Recover System Database** (Recuperar base de datos del sistema).

   :::image type="content" source="media/snapcenter/hana-studio-backup-recovery.png" alt-text="Inicie el proceso de restauración en HANA":::.

1. Seleccione un punto de recuperación.

   :::image type="content" source="media/snapcenter/restore-database-select-recovery-type.png" alt-text="Seleccione el tipo de recuperación":::.

1. Seleccione la ubicación del catálogo de copias de seguridad.

   :::image type="content" source="media/snapcenter/restore-database-select-location-backup-catalog.png" alt-text="Seleccione la ubicación del catálogo de copias de seguridad":::.

1. Seleccione una copia de seguridad para recuperar la base de datos de SAP HANA.

   :::image type="content" source="media/snapcenter/restore-database-select-backup.png" alt-text="Seleccione una copia de seguridad para recuperar la base de datos de SAP HANA":::.

   Una vez recuperada la base de datos, aparece un mensaje con un sello **Recovered to Time** (Recuperado a la hora) y **Recovered to Log Position** (Recuperado a la posición del registro).

1. En **Systems** (Sistemas), haga clic con el botón secundario en la base de datos del sistema y seleccione **Backup and Recovery** (Copia de seguridad y recuperación) > **Recover Tenant Database** (Recuperar base de datos del inquilino).
1. Siga el flujo de trabajo del Asistente para completar la recuperación de la base de datos del inquilino. 

Para obtener más información sobre cómo restaurar una base de datos, consulte [Copia de seguridad y restauración de SAP HANA con SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf).


### <a name="non-database-backups"></a>Copias de seguridad que no son de base de datos
Puede restaurar volúmenes que no sean de datos, por ejemplo, un recurso compartido de archivos de red (/hana/shared) o una copia de seguridad del sistema operativo.  Para obtener más información sobre la restauración de un volumen que no es de datos, consulte [Copia de seguridad y restauración de SAP HANA con SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf).

### <a name="sap-hana-system-cloning"></a>Clonación del sistema SAP HANA

Para clonar, debe tener instalada la misma versión de HANA que la base de datos de origen. El SID y el identificador pueden ser diferentes. 

:::image type="content" source="media/snapcenter/system-cloning-diagram.png" alt-text="Clonación del sistema SAP HANA" lightbox="media/snapcenter/system-cloning-diagram.png" border="false":::

1. Cree un almacén de usuario de base de datos de HANA para la base de datos H34 desde /usr/sap/H34/HDB40.

   ```
   hdbuserstore set H34KEY sollabsjct34:34013 system manager
   ```
 
1. Deshabilite el firewall.

   ```bash
   systemctl disable SuSEfirewall2
   systemctl stop  SuSEfirewall2
   ```

1. Instale el SDK de Java.

   ```bash
   zypper in java-1_8_0-openjdk
   ```

1. En SnapCenter, agregue el host de destino en el que se montará el clon. Para obtener más información, consulte [Adición de hosts e instalación de paquetes de complementos en hosts remotos](http://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-sap-hana%2FGUID-246C0810-4F0B-4BF7-9A35-B729AD69954A.html).
   1. Proporcione la información de las credenciales de ejecución que desea agregar. 
   1. Seleccione el sistema operativo host y escriba la información del host.
   1. En **Plug-ins to install** (Complementos para instalar), seleccione la versión, escriba la ruta de instalación y seleccione **SAP HANA**.
   1. Seleccione **Validate** (Validar) para ejecutar las comprobaciones previas a la instalación.

1. Detenga HANA y desmonte el volumen de datos antiguo.  Montará el clon desde SnapCenter.  

   ```bash
   sapcontrol -nr 40 -function StopSystem
   umount /hana/data/H34/mnt00001

   ```
 1. Cree los archivos de script de configuración y de shell para el destino.
 
    ```bash
    mkdir /NetApp
    chmod 777 /NetApp
    cd NetApp
    chmod 777 sc-system-refresh-H34.cfg
    chmod 777 sc-system-refresh.sh

    ```

    >[!TIP]
    >Puede copiar los scripts desde [Clonación de SAP desde SnapCenter](https://www.netapp.com/us/media/tr-4667.pdf).

1. Modifique el archivo de configuración. 

   ```bash
   vi sc-system-refresh-H34.cfg
   ```

   * HANA_ARCHITECTURE="MDC_single_tenant"
   * KEY="H34KEY"
   * TIME_OUT_START=18
   * TIME_OUT_STOP=18
   * INSTANCENO="40"
   * STORAGE="10.250.101.33"

1. Modifique el archivo de script de shell.

   ```bash
   vi sc-system-refresh.sh
   ```  

   * VERBOSE=NO
   * MY_NAME="`basename $0`"
   * BASE_SCRIPT_DIR="`dirname $0`"
   * MOUNT_OPTIONS="rw,vers=4,hard,timeo=600,rsize=1048576,wsize=1048576,intr,noatime,nolock"

1. Inicie el clon desde un proceso de copia de seguridad. Seleccione el host para crear el clon. 

   >[!NOTE]
   >Para obtener más información, vea [Clonación desde una copia de seguridad](https://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-cpi%2FGUID-F6E7FF73-0183-4B9F-8156-8D7DA17A8555.html).

1. En **Scripts**, proporcione lo siguiente:

   * **Comando de montaje:** /NetApp/sc-system-refresh.sh mount H34 %hana_data_h31_mnt00001_t250_vol_Clone
   * **Comando posterior a la clonación:** /NetApp/sc-system-refresh.sh recover H34

1. Deshabilite (bloquee) el montaje automático en /etc/fstab, ya que no es necesario el volumen de datos de la base de datos preinstalada. 

   ```bash
   vi /etc/fstab
   ```

### <a name="delete-a-clone"></a>Eliminación de un clon

Puede eliminar un clon si ya no es necesario. Para más información, consulte [Eliminación de clones](https://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-cpi%2FGUID-F6E7FF73-0183-4B9F-8156-8D7DA17A8555.html).

Los comandos usados para ejecutar antes de la eliminación de clones son:
* **Eliminación anterior a la clonación:** /NetApp/sc-system-refresh.sh shut down H34
* **Desmontaje:** /NetApp/sc-system-refresh.sh umount H34

Estos comandos permiten a SnapCenter apagar la base de datos, desmontar el volumen y eliminar la entrada fstab.  Después, se elimina FlexClone. 

### <a name="cloning-database-logfile"></a>Clonación del archivo de registro de la base de datos

```   
20190502025323###sollabsjct34###sc-system-refresh.sh: Adding entry in /etc/fstab.
20190502025323###sollabsjct34###sc-system-refresh.sh: 10.250.101.31:/Sc21186309-ee57-41a3-8584-8210297f791d /hana/data/H34/mnt00001 nfs rw,vers=4,hard,timeo=600,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
20190502025323###sollabsjct34###sc-system-refresh.sh: Mounting data volume.
20190502025323###sollabsjct34###sc-system-refresh.sh: mount /hana/data/H34/mnt00001
20190502025323###sollabsjct34###sc-system-refresh.sh: Data volume mounted successfully.
20190502025323###sollabsjct34###sc-system-refresh.sh: chown -R h34adm:sapsys /hana/data/H34/mnt00001
20190502025333###sollabsjct34###sc-system-refresh.sh: Recover system database.
20190502025333###sollabsjct34###sc-system-refresh.sh: /usr/sap/H34/HDB40/exe/Python/bin/python /usr/sap/H34/HDB40/exe/python_support/recoverSys.py --command "RECOVER DATA USING SNAPSHOT CLEAR LOG"
[140278542735104, 0.005] >> starting recoverSys (at Thu May  2 02:53:33 2019)
[140278542735104, 0.005] args: ()
[140278542735104, 0.005] keys: {'command': 'RECOVER DATA USING SNAPSHOT CLEAR LOG'}
recoverSys started: ============2019-05-02 02:53:33 ============
testing master: sollabsjct34
sollabsjct34 is master
shutdown database, timeout is 120
stop system
stop system: sollabsjct34
stopping system: 2019-05-02 02:53:33
stopped system: 2019-05-02 02:53:33
creating file recoverInstance.sql
restart database
restart master nameserver: 2019-05-02 02:53:38
start system: sollabsjct34
2019-05-02T02:53:59-07:00  P010976      16a77f6c8a2 INFO    RECOVERY state of service: nameserver, sollabsjct34:34001, volume: 1, RecoveryPrepared
recoverSys finished successfully: 2019-05-02 02:54:00
[140278542735104, 26.490] 0
[140278542735104, 26.490] << ending recoverSys, rc = 0 (RC_TEST_OK), after 26.485 secs
20190502025400###sollabsjct34###sc-system-refresh.sh: Wait until SAP HANA database is started ....
20190502025400###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025410###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025420###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025430###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025440###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025451###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502025451###sollabsjct34###sc-system-refresh.sh: SAP HANA database is started.
20190502025451###sollabsjct34###sc-system-refresh.sh: Recover tenant database H34.
20190502025451###sollabsjct34###sc-system-refresh.sh: /usr/sap/H34/SYS/exe/hdb/hdbsql -U H34KEY RECOVER DATA FOR H34 USING SNAPSHOT CLEAR LOG
0 rows affected (overall time 69.584135 sec; server time 69.582835 sec)
20190502025600###sollabsjct34###sc-system-refresh.sh: Checking availability of Indexserver for tenant H34.
20190502025601###sollabsjct34###sc-system-refresh.sh: Recovery of tenant database H34 succesfully finished.
20190502025601###sollabsjct34###sc-system-refresh.sh: Status: GREEN
Deleting the DB Clone – Logfile
20190502030312###sollabsjct34###sc-system-refresh.sh: Stopping HANA database.
20190502030312###sollabsjct34###sc-system-refresh.sh: sapcontrol -nr 40 -function StopSystem HDB

02.05.2019 03:03:12
StopSystem
OK
20190502030312###sollabsjct34###sc-system-refresh.sh: Wait until SAP HANA database is stopped ....
20190502030312###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502030322###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502030332###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502030342###sollabsjct34###sc-system-refresh.sh: Status:  GRAY
20190502030342###sollabsjct34###sc-system-refresh.sh: SAP HANA database is stopped.
20190502030347###sollabsjct34###sc-system-refresh.sh: Unmounting data volume.
20190502030347###sollabsjct34###sc-system-refresh.sh: Junction path: Sc21186309-ee57-41a3-8584-8210297f791d
20190502030347###sollabsjct34###sc-system-refresh.sh: umount /hana/data/H34/mnt00001
20190502030347###sollabsjct34###sc-system-refresh.sh: Deleting /etc/fstab entry.
20190502030347###sollabsjct34###sc-system-refresh.sh: Data volume unmounted successfully.

```

### <a name="uninstall-snapcenter-plug-ins-package-for-linux"></a>Desinstalación del paquete de complementos de SnapCenter para Linux

Puede desinstalar el paquete de complementos de Linux desde la línea de comandos. Dado que la implementación automática espera un sistema nuevo, es fácil desinstalar el complemento.  

>[!NOTE]
>Es posible que tenga que desinstalar manualmente una versión anterior del complemento. 

Desinstale los complementos.

```bash
cd /opt/NetApp/snapcenter/spl/installation/plugins
./uninstall
```

Ahora puede instalar el complemento HANA más reciente en el nuevo nodo seleccionando **SUBMIT** (ENVIAR) en SnapCenter. 




## <a name="next-steps"></a>Pasos siguientes
- Consulte [Principios y preparación de la recuperación ante desastres](hana-concept-preparation.md).
