---
title: Solución de problemas de certificación de máquina virtual para Azure Marketplace
description: Solución de problemas comunes relacionados con la prueba y certificación de las imágenes de máquina virtual para Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: troubleshooting
author: mathapli
ms.author: mathapli
ms.date: 01/18/2021
ms.openlocfilehash: f5bfea74785c32a514128dc8462891e1db3aca29
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111811549"
---
# <a name="troubleshoot-virtual-machine-certification"></a>Solución de problemas de certificación de máquina virtual

Cuando publica la imagen de máquina virtual (VM) en Azure Marketplace, el equipo de Azure la valida para garantizar que se puede arrancar, que es segura y que es compatible con Azure. Si la imagen de la máquina virtual produce un error en cualquiera de las pruebas de alta calidad, no se publicará. Recibirá un mensaje de error que describe el problema.

En este artículo se explican los mensajes de error comunes durante la publicación de imágenes de máquina virtual, junto con soluciones relacionadas.

> [!NOTE]
> Si tiene preguntas sobre este artículo o comentarios sobre mejoras, póngase en contacto con el [soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport).

## <a name="vm-extension-failure"></a>Error de la extensión de máquina virtual

Compruebe si la imagen es compatible con las extensiones de máquina virtual.

Para habilitar las extensiones de máquina virtual:

1. Seleccione la máquina virtual Linux.
1. Vaya a **Configuración de diagnóstico**.
1. Habilite las matrices base mediante la actualización de la **cuenta de almacenamiento**.
1. Seleccione **Guardar**.

   ![Captura de pantalla que muestra cómo habilitar la supervisión de nivel de invitado.](./media/create-vm/vm-certification-issues-solutions-1.png)

Para comprobar que las extensiones de máquina virtual están activadas correctamente:

1. Vaya a la pestaña **Extensiones de máquina virtual** y, a continuación, verifique el estado de la **Extensión de diagnósticos de Linux**.
1. Compruebe el estado de aprovisionamiento.

   - Si el estado es *Aprovisionamiento realizado correctamente*, el caso de prueba de las extensiones se ha realizado correctamente.  
   - Si el estado es *Error de aprovisionamiento*, el caso de prueba de extensiones ha dado error y es necesario configurar la marca reforzada.

   ![Captura de pantalla que muestra que el aprovisionamiento se ha realizado correctamente.](./media/create-vm/vm-certification-issues-solutions-2.png)

   Si se produce un error en la extensión de máquina virtual, consulte [Uso de la extensión de diagnósticos de Linux para supervisar métricas y registros](../virtual-machines/extensions/diagnostics-linux.md) para habilitarla. Si no desea habilitar la extensión de máquina virtual, póngase en contacto con el equipo de soporte técnico y pídale que la deshabilite.

## <a name="vm-provisioning-issue"></a>Problema de aprovisionamiento de máquinas virtuales

Asegúrese de que ha seguido el proceso de aprovisionamiento de la máquina virtual rigurosamente antes de enviar la oferta. Para ver el formato JSON y aprovisionar la máquina virtual, consulte [Prueba de una imagen de máquina virtual](azure-vm-image-test.md).

Entre los problemas de aprovisionamiento, se incluyen los siguientes escenarios:

|Escenario|Error|Motivo|Solución|
|---|---|---|---|
|1|Disco duro virtual (VHD) no válido:|Si el valor de cookie especificado en el pie de página del VHD no es correcto, el VHD se considerará no válido.|Vuelva a crear la imagen y envíe la solicitud.|
|2|Tipo de blob no válido|La máquina virtual no se pudo aprovisionar porque el bloque utilizado es de tipo blob en lugar de tipo página.|Vuelva a crear la imagen y envíe la solicitud.|
|3|Tiempo de espera de aprovisionamiento agotado o no generalizado correctamente|Hay un problema con la generalización de máquina virtual.|Vuelva a crear la imagen con generalización y envíe la solicitud.|
|

> [!NOTE]
> Para obtener más información sobre la generalización de máquinas virtuales, consulte:
> - [Documentación de Linux](azure-vm-create-using-approved-base.md#generalize-the-image)
> - [Documentación de Windows](../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep)

## <a name="vhd-specifications"></a>Especificaciones del disco duro virtual

### <a name="conectix-cookie-and-other-vhd-specifications"></a>Cookies de Conectix y otras especificaciones del disco duro virtual

La cadena "conectix" forma parte de la especificación de VHD. Se define como la cookie de 8 bytes en el pie de página de VHD que identifica al creador del archivo. Todos los archivos VHD creados por Microsoft tienen esta cookie.

Un blob con formato VHD debe tener un pie de página de 512 bytes con este formato:

|Campos de pie de disco duro|Tamaño (bytes)|
|---|---|
Cookie|8
Características|4
Versión del formato de archivo|4
Desplazamiento de los datos|8
Marca de tiempo|4
Aplicación de creación|4
Versión de la creación|4
Sistema operativo host de la creación|4
Tamaño original|8
Tamaño actual|8
Geometría del disco|4
Tipo de disco|4
Suma de comprobación|4
Id. único|16
Estado guardado|1
Reservada|427
|

### <a name="vhd-specifications"></a>Especificaciones del disco duro virtual

Para garantizar una experiencia de publicación sin problemas, asegúrese de que el disco duro virtual cumple los siguientes criterios:

- La cookie contiene la cadena "conectix".
- El tipo de disco es fijo.
- El tamaño virtual del disco duro virtual es de al menos 20 MB.
- El disco duro virtual está alineado. El tamaño virtual debe ser un múltiplo de 1 MB.
- La longitud de blob de VHD es igual al tamaño virtual más la longitud del pie de página de VHD (512).

Descargue la [especificación de VHD](https://www.microsoft.com/download/details.aspx?id=23850).

## <a name="software-compliance-for-windows"></a>Compatibilidad de software para Windows

Si se rechaza su solicitud de imagen de Windows debido a un problema de cumplimiento de software, es posible que haya creado una imagen de Windows con una instancia de SQL Server instalada. En su lugar, debe utilizar la imagen base de la versión de SQL Server pertinente de Azure Marketplace.

No cree su propia imagen de Windows con SQL Server instalado. Use las imágenes base de SQL aprobadas (Enterprise/Standard/web) de Azure Marketplace.

Si intenta instalar Visual Studio o cualquier producto con licencia de Office, póngase en contacto con el equipo de soporte técnico para obtener aprobación previamente.

Para más información sobre seleccionar una base aprobada, consulte [Creación de una máquina virtual desde una base aprobada](azure-vm-create-using-approved-base.md).

## <a name="toolkit-test-case-execution-failed"></a>Error de ejecución del caso de prueba del kit de herramientas

El kit de herramientas de certificación de Microsoft le puede ayudarle a ejecutar casos de prueba y comprobar que su disco duro virtual o imagen es compatible con el entorno de Azure.

Descargue el [kit de herramientas de certificación de Microsoft](azure-vm-image-test.md).

### <a name="linux-test-cases"></a>Casos de prueba de Linux

En la tabla siguiente se enumeran los casos de prueba de Linux que ejecutará el kit de herramientas. La validación de prueba se indica en la descripción.

|Escenario|Caso de prueba|Descripción|
|---|---|---|
|1|Historial de Bash|Los archivos de historial de Bash deben borrarse antes de que cree la imagen de máquina virtual.|
|2|Versión del agente Linux|Se debe instalar el agente Linux de Azure 2.2.41 o más reciente.|
|3|Parámetros de kernel obligatorios|Comprueba que se han configurado los siguientes parámetros de kernel: <br>console=ttyS0<br>earlyprintk=ttyS0<br>rootdelay=300 |
|4|Partición de intercambio en el disco del sistema operativo|Comprueba que no se crean particiones de intercambio en el disco del sistema operativo.|
|5|Partición raíz en el disco del sistema operativo|Crear una sola partición raíz para el disco de SO.|
|6|Versión de OpenSSL|La versión de OpenSSL debe ser v0.9.8 o más reciente.|
|7|Versión de Python|Se recomienda usar la versión de Python 2.6 o más reciente.|
|8|ClientAliveInterval|Establece ClientAliveInterval en 180. En cuanto al requisito de la aplicación, se puede establecer de 30 a 235. Si va a habilitar SSH para sus usuarios finales, este valor debe establecerse tal y como se ha explicado.|
|9|Arquitectura del SO|Solo se admiten los sistemas operativos de 64 bits.|
|10|Actualización automática|Determina si la actualización automática del agente Linux está habilitada.|
|

### <a name="common-test-case-errors"></a>Errores comunes de casos de prueba

Consulte en la tabla siguiente los errores comunes que pueden aparecer al ejecutar casos de prueba:

| Escenario | Caso de prueba | Error | Solución |
| --- | --- | --- | --- |
| 1 | Caso de prueba de la versión del agente Linux | La versión mínima del agente Linux es 2.2.41 o más reciente. Este requisito es obligatorio desde el 1 de mayo de 2020. | Actualice la versión del agente Linux. Debe ser 2.241 o posterior. Para obtener más información, visite la [página de actualización de la versión del agente Linux](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support). |
| 2 | Caso de prueba del historial de Bash | Se produce un error si el tamaño del historial de Bash de la imagen enviada es superior a 1 kilobyte (KB). El tamaño está limitado a 1 KB para garantizar que cualquier archivo de historial de Bash no contiene información potencialmente confidencial. | Monte el disco duro virtual en otra máquina virtual que funcione y realice cambios para reducir el tamaño a 1 KB o menos. Por ejemplo, elimine los archivos de historial de `.bash`. |
| 3 | Caso de prueba del parámetro de kernel obligatorio | Recibirá este error si el valor de `console` no se ha establecido en `ttyS0`. Haga una comprobación al ejecutar el comando siguiente: <br /> `cat /proc/cmdline` | Establezca el valor de `console` en `ttyS0` y vuelva a enviar la solicitud. |
| 4 | Caso de prueba de ClientAliveInterval | Si el resultado del kit de herramientas genera un resultado de error para este caso de prueba, el valor de `ClientAliveInterval` no es adecuado. | Establezca `ClientAliveInterval` en un valor igual o inferior a 235 y, a continuación, vuelva a enviar la solicitud. |
|

### <a name="windows-test-cases"></a>Casos de prueba de Windows

En la tabla siguiente se enumeran los casos de prueba de Windows que ejecutará el kit de herramientas, junto con una descripción de la validación de prueba:

|Escenario |Casos de prueba|Descripción|
|---|---|---|
|1|Arquitectura del SO|Azure solo admite los sistemas operativos de 64 bits.|
|2|Dependencia de la cuenta de usuario|La ejecución de aplicaciones no debería depender de la cuenta de administrador.|
|3|Clúster de conmutación por error|La característica de clústeres de conmutación por error de Windows Server aún no se admite. La aplicación no debería depender de esta característica.|
|4|IPV6|IPv6 aún no se admite en el entorno de Azure. La aplicación no debería depender de esta característica.|
|5|DHCP|El rol de servidor del protocolo de configuración dinámica de host aún no se admite. La aplicación no debería depender de esta característica.|
|6|Hyper-V|El rol de servidor de Hyper-V aún no se admite. La aplicación no debería depender de esta característica.|
|7|Acceso remoto|El rol de servidor de acceso remoto (acceso directo) aún no se admite. La aplicación no debería depender de esta característica.|
|8|Rights Management Services|Rights Management Services. El rol de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|9|Servicios de implementación de Windows|Servicios de implementación de Windows. El rol de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|10|Cifrado BitLocker de unidades|El Cifrado de unidad BitLocker no se admite en el disco duro del sistema operativo, pero se podría utilizar en discos de datos.|
|11|Servidor de nombres de almacenamiento de Internet|La característica Servidor de nombres de almacenamiento de Internet aún no se admite. La aplicación no debería depender de esta característica.|
|12|E/S de múltiples rutas|E/S de múltiples rutas. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|13|Equilibrio de carga de red|Equilibrio de carga de red. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|14|Protocolo de resolución de nombres de mismo nivel|Protocolo de resolución de nombres de mismo nivel. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|15|Servicios SNMP|Todavía no se admite la característica servicios de Protocolo simple de administración de redes (SNMP). La aplicación no debería depender de esta característica.|
|16|Servicio de nombres Internet de Windows|Servicio de nombres Internet de Windows. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|17|Servicio WLAN|Servicio WLAN. Esta característica de servidor aún no se admite. La aplicación no debería depender de esta característica.|
|

Si se producen errores con los casos de prueba mencionados, consulte la columna de **Descripción** en la tabla para ver la solución. Para más información, póngase en contacto con el equipo de soporte técnico.

## <a name="data-disk-size-verification"></a>Comprobación del tamaño del disco de datos

No se aprobarán las solicitudes de disco de datos con un tamaño superior a 1023 gigabytes (GB). Esta regla se aplica a Windows y Linux.

Vuelva a enviar la solicitud con un tamaño que sea inferior o igual a 1023 GB.

## <a name="os-disk-size-validation"></a>Validación del tamaño del disco del sistema operativo

Consulte las siguientes reglas para conocer las limitaciones del tamaño del disco del sistema operativo. Al enviar una solicitud, compruebe que el tamaño del disco del sistema operativo cumple el requisito para Linux o Windows.

|SO|Tamaño recomendado del disco duro virtual|
|---|---|
|Linux|De 1 GB a 1023 GB|
|Windows|De 30 a 250 GB|
|

Como las máquinas virtuales permiten acceder al sistema operativo subyacente, asegúrese de que el tamaño del disco duro virtual sea lo suficientemente grande. Los discos no se pueden expandir sin tiempo de inactividad. Use un tamaño de disco de 30 GB a 50 GB.

|Tamaño de VHD|Tamaño real ocupado|Solución|
|---|---|---|
|>500 tebibytes (TiB)|N/D|Póngase en contacto con el equipo de soporte técnico para obtener una aprobación extraordinaria.|
|De 250 a 500 TiB|diferencia de > 200 gibibytes (GiB) del tamaño del blob|Póngase en contacto con el equipo de soporte técnico para obtener una aprobación extraordinaria.|
|

> [!NOTE]
> Los discos de mayor tamaño tienen costos más elevados y provocarán un retraso durante el proceso de configuración y replicación. Debido a este retraso y al costo, el equipo de soporte técnico podría solicitar una justificación para la aprobación extraordinaria.

## <a name="wannacry-patch-verification-test-for-windows"></a>Prueba de comprobación de la revisión de WannaCry para Windows

Para evitar un posible ataque relacionado con el virus WannaCry, asegúrese de que todas las solicitudes de imagen de Windows estén actualizadas con la revisión más reciente.

Puede comprobar la versión del archivo de imagen en `C:\windows\system32\drivers\srv.sys` o `srv2.sys`.

En la tabla siguiente se muestra la versión mínima revisada de Windows Server:

|SO|Versión|
|---|---|
|Windows Server 2008 R2|6.1.7601.23689|
|Windows Server 2012|6.2.9200.22099|
|Windows Server 2012 R2|6.3.9600.18604|
|Windows Server 2016|10.0.14393.953|
|Windows Server 2019|N/D|
|

> [!NOTE]
> WindowsServer2019 no tiene requisitos de versión obligatoria.

## <a name="sack-vulnerability-patch-verification"></a>Comprobación de la revisión para la vulnerabilidad SACK

Al enviar una imagen de Linux, es posible que se rechace su solicitud debido a problemas con la versión del kernel.

Actualice el kernel con una versión aprobada y vuelva a enviar la solicitud. Puede encontrar la versión aprobada de kernel en la tabla siguiente. El número de versión debe ser igual o superior al número que se muestra a continuación.

Si su imagen no tiene instalada una de las siguientes versiones de kernel, actualícela con las revisiones correctas. Solicite la aprobación correspondiente del equipo de soporte técnico después de actualizar la imagen con estas revisiones necesarias:

- CVE-2019-11477
- CVE-2019-11478
- CVE-2019-11479

|Familia de SO|Versión|Kernel|
|---|---|---|
|Ubuntu|14.04 LTS|4.4.0-151| 
||14.04 LTS|4.15.0-1049-\*-azure|
||16.04 LTS|4.15.0-1049|
||18.04 LTS|4.18.0-1023|
||18.04 LTS|5.0.0-1025|
||18.10|4.18.0-1023|
||19.04|5.0.0-1010|
||19.04|5.3.0-1004|
|RHEL y Cent OS|6.10|2.6.32-754.15.3|
||7.2|3.10.0-327.79.2|
||7.3|3.10.0-514.66.2|
||7.4|3.10.0-693.50.3|
||7.5|3.10.0-862.34.2|
||7.6|3.10.0-957.21.3|
||7,7|3.10.0-1062.1.1|
||8.0|4.18.0-80.4.2|
||8.1|4.18.0-147|
||"7-RAW" (7.6)||
||"7-LVM" (7.6)|3.10.0-957.21.3|
||RHEL-SAP 7.4|TBD|
||RHEL-SAP 7.5|TBD|
|SLES|SLES11SP4 (incluido SAP)|3.0.101-108.95.2|
||SLES12SP1 para SAP|3.12.74-60.64.115.1|
||SLES12SP2 para SAP|4.4.121-92.114.1|
||SLES12SP3|4.4180-4.31.1 (kernel-azure)|
||SLES12SP3 para SAP|4.4.180-94.97.1|
||SLES12SP4|4.12.14-6.15.2 (kernel-azure)|
||SLES12SP4 para SAP|4.12.14-95.19.1|
||SLES15|4.12.14-5.30.1 (kernel-azure)|
||SLES15 para SAP|4.12.14-5.30.1 (kernel-azure)|
||SLES15SP1|4.12.14-5.30.1 (kernel-azure)|
|Oracle|6.10|UEK2 2.6.39-400.312.2<br>UEK3 3.8.13-118.35.2<br>RHCK 2.6.32-754.15.3 
||7.0-7.5|UEK3 3.8.13-118.35.2<br>UEK4 4.1.12-124.28.3<br>RHCK sigue a RHEL (más arriba)|
||7.6|RHCK 3.10.0-957.21.3<br>UEK5 4.14.35-1902.2.0|
|CoreOS Stable 2079.6.0|4.19.43\*|
||Beta 2135.3.1|4.19.50\*|
||Alpha 2163.2.1|4.19.50\*|
|Debian|jessie (seguridad)|3.16.68-2|
||jessie (backports)|4.9.168-1+deb9u3|
||stretch (seguridad)|4.9.168-1+deb9u3|
||Debian GNU/Linux 10 (buster)|Debian 6.3.0-18+deb9u1|
||buster, sid (stretch backports)|4.19.37-5|
|

## <a name="image-size-should-be-in-multiples-of-megabytes"></a>El tamaño de la imagen debe expresarse en múltiplos de megabytes

En Azure, todos los discos duros virtuales deben tener un tamaño virtual expresado en múltiplos de 1 megabyte (MB). Si el disco duro virtual no tiene el tamaño recomendado, es posible que la solicitud se rechace.

Siga las instrucciones al convertir de un disco sin procesar a VHD. Asegúrese de que el tamaño del disco sin procesar sea un múltiplo de 1 MB. Para más información, consulte [Información para las distribuciones no aprobadas](../virtual-machines/linux/create-upload-generic.md).

## <a name="vm-access-denied"></a>Acceso denegado a la máquina virtual

Un problema de _acceso denegado_ para ejecutar un caso de prueba en la máquina virtual podría ser debido a que los privilegios no son suficientes.

Compruebe que haya habilitado el acceso adecuado para la cuenta en la que se ejecutan los casos de prueba automáticos. Habilite el acceso para ejecutar casos de prueba si no está habilitado. Si no desea habilitar el acceso, podría compartir los resultados de los casos de prueba automáticos con el equipo de soporte técnico.

Para enviar la solicitud con la imagen SSH deshabilitada para el proceso de certificación:

1. Ejecute la [herramienta de prueba de certificación más reciente para máquinas virtuales de Azure](https://aka.ms/AzureCertificationTestTool) en la imagen.

2. Presente una [incidencia de soporte técnico](https://aka.ms/marketplacepublishersupport). Asegúrese de adjuntar el informe del kit de herramientas y proporcionar los detalles de la oferta:
   - Nombre de la oferta
   - Nombre de publicador
   - Id. de plan o SKU y versión

3. Vuelva a enviar la solicitud de certificación.

## <a name="download-failure"></a>Error de descarga

Consulte la tabla siguiente para ver los problemas que surgen al descargar la imagen de máquina virtual con una dirección URL de firma de acceso compartido (SAS).

|Error|Motivo|Solución|
|---|---|---|
|Blob no encontrado|El disco duro virtual podría eliminarse o moverse de la ubicación especificada.|| 
|Blob en uso|Otro proceso interno utiliza el disco duro virtual.|El disco duro virtual debe estar en uso cuando lo descargue con una dirección URL de SAS.|
|Dirección URL de SAS no válida|La dirección URL de SAS asociada al disco duro virtual es incorrecta.|Obtenga la dirección URL de SAS correcta.|
|Firma incorrecta|La dirección URL de SAS asociada al disco duro virtual es incorrecta.|Obtenga la dirección URL de SAS correcta.|
|Encabezado condicional HTTP|La dirección URL no es válida.|Obtenga la dirección URL de SAS correcta.|
|El nombre del disco duro virtual no es válido|Compruebe si hay algún carácter especial, como un signo de porcentaje `%` o comillas `"`, en el nombre del disco duro virtual.|Cambie el nombre del archivo del disco duro virtual y quite los caracteres especiales.|
|

## <a name="vm-images-must-have-1mb-free-space"></a>Las imágenes de máquina virtual deben tener 1 MB de espacio libre.

Si [crea su propia imagen](azure-vm-create-using-own-image.md), asegúrese de que los primeros 2048 sectores (1 MB) del disco del sistema operativo están vacíos. De lo contrario, se producirá un error en la publicación. Este requisito solo se aplica al disco del sistema operativo (no a discos de datos). Si crea la imagen [a partir de una base aprobada](azure-vm-create-using-approved-base.md), puede omitir este requisito.

### <a name="how-to-keep-1-mb-free-space-at-the-start-on-an-empty-vhd-2048-sectors-each-sector-of-512-bytes"></a>Cómo mantener 1 MB de espacio libre al principio en un VHD vacío (2048 sectores, cada sector de 512 bytes)

Estos pasos se aplican a Linux solamente.

1. Cree cualquier tipo de máquina virtual de Linux, como Ubuntu, Cent OS u otro. Rellene los campos obligatorios y haga clic en **Siguiente: Discos >** .

   ![Captura de pantalla que muestra la página Crear una máquina virtual con el botón "Siguiente: Discos" resaltado.](./media/create-vm/vm-certification-issues-solutions-15.png)

1. Cree un disco no administrado para la máquina virtual.

   Use los valores predeterminados o especifique cualquier valor para los campos como NIC, NSG y la dirección IP pública.

   ![Imagen de captura de pantalla de la página "Discos de datos" en el flujo de creación de una máquina virtual.](./media/create-vm/vm-certification-issues-solutions-16.png)

1. Después de crear la máquina virtual, seleccione **Discos** en el panel izquierdo.

   ![Captura de pantalla en la que muestra cómo seleccionar discos para una máquina virtual.](./media/create-vm/vm-certification-issues-solutions-17.png)

1. Conecte el VHD como un disco de datos a la VM para crear la tabla de particiones.

   1. Seleccione **Agregar disco de datos** > **Blob existente**.

      ![Captura de pantalla en la que se muestra cómo agregar un disco de datos al VHD.](./media/create-vm/vm-certification-issues-solutions-18.png)

   1. Busque la cuenta de almacenamiento de VHD.
   1. Seleccione **Contenedor** y, luego, seleccione el VHD.
   1. Seleccione **Aceptar**.

      ![Captura de pantalla de la página Asociar disco no administrado.](./media/create-vm/vm-certification-issues-solutions-19.png)

      El VHD se agregará como disco de datos LUN 0.

   1. Reinicie la VM.

1. Después de reiniciar la VM, inicie sesión en ella con Putty u otro cliente, y ejecute el comando `sudo  -i` para obtener acceso raíz.

   ![Captura de pantalla de la línea de comandos del cliente Putty que muestra el comando sudo -i.](./media/create-vm/vm-certification-issues-solutions-20.png)

1. Cree una partición en el VHD.

   1. Escriba comando `fdisk /dev/sdb`.
   1. Para ver la lista de particiones existente del disco duro virtual, escriba `p`.
   1. Escriba `d` para eliminar todas las particiones existentes disponibles en el disco duro virtual. Puede omitir este paso, si no es necesario.

      ![Captura de pantalla de la línea de comandos del cliente Putty que muestra los comandos para eliminar las particiones existentes.](./media/create-vm/vm-certification-issues-solutions-21.png)

   1. Escriba `n` para crear una nueva partición y seleccione `p` para la partición primaria.

   1. Escriba 2048 como valor del _primer sector_. Puede dejar el _último sector_  como el valor predeterminado.

      >[!IMPORTANT]
      >Los datos existentes se borrarán hasta llegar a 2048 sectores (cada sector de 512 bytes). Haga copia de seguridad del disco duro virtual antes de crear una nueva partición.

      ![Captura de pantalla de la línea de comandos del cliente Putty que muestra los comandos y la salida de los datos borrados.](./media/create-vm/vm-certification-issues-solutions-22.png)

   1. Escriba `w` para confirmar la creación de la partición.

      ![Captura de pantalla de la línea de comandos del cliente Putty que muestra los comandos para crear una partición.](./media/create-vm/vm-certification-issues-solutions-23.png)

   1. Para comprobar la tabla de particiones, ejecute el comando `n fdisk /dev/sdb` y escriba `p`. Verá que la partición se crea con el valor de desplazamiento 2048.

      ![Captura de pantalla de la línea de comandos del cliente Putty que muestra los comandos para crear un desplazamiento de 2048.](./media/create-vm/vm-certification-issues-solutions-24.png)

1. Desconecte el disco duro virtual de la VM y elimínela.

### <a name="how-to-keep-1-mb-free-space-by-moving-existing-data-on-vhd-2048-sectors-each-sector-of-512-bytes"></a>Cómo mantener 1 MB de espacio libre moviendo los datos existentes en el VHD (2048 sectores, cada sector de 512 bytes)

Estos pasos se aplican a Linux solamente.

1. Cree cualquier tipo de máquina virtual de Linux, como Ubuntu, Cent OS u otro. Rellene los campos obligatorios y haga clic en **Siguiente: Discos >** .

   ![Captura de pantalla que muestra la página Crear una máquina virtual con el botón "Siguiente: Discos" resaltado.](./media/create-vm/vm-certification-issues-solutions-15.png)

1. Cree un disco no administrado para la máquina virtual.

   ![Imagen de captura de pantalla de la página "Discos de datos" en el flujo de creación de una máquina virtual.](./media/create-vm/vm-certification-issues-solutions-16.png)

   Use los valores predeterminados o especifique cualquier valor para los campos como NIC, NSG y la dirección IP pública.

1. Después de crear la máquina virtual, seleccione **Discos** en el panel izquierdo.

   ![Captura de pantalla en la que muestra cómo seleccionar discos para una máquina virtual.](./media/create-vm/vm-certification-issues-solutions-17.png)

1. Conecte el VHD como un disco de datos a la VM para crear la tabla de particiones.

   1. Conecte el VHD como un disco de datos a la VM para crear la tabla de particiones.

   1. Seleccione **Agregar disco de datos** > **Blob existente**.

      ![Captura de pantalla en la que se muestra cómo agregar un disco de datos al VHD.](./media/create-vm/vm-certification-issues-solutions-18.png)

   1. Busque la cuenta de almacenamiento de VHD.
   1. Seleccione **Contenedor** y, luego, seleccione el VHD.
   1. Seleccione **Aceptar**.

      ![Captura de pantalla de la página Asociar disco no administrado.](./media/create-vm/vm-certification-issues-solutions-19.png)

      El VHD se agregará como disco de datos LUN 0.

   1. Reinicie la VM.

1. Inicie sesión en la máquina virtual con Putty u otro cliente y ejecute el comando `sudo  -i` para obtener el acceso raíz.

   ![Captura de pantalla de la línea de comandos del cliente Putty que muestra el inicio de sesión y el comando sudo -i.](./media/create-vm/vm-certification-issues-solutions-20.png)

1. Ejecute el comando `echo '+1M,' | sfdisk --move-data /dev/sdc -N 1`.

   ![Captura de pantalla de la línea de comandos del cliente Putty que muestra la ejecución del comando sudo -i.](./media/create-vm/vm-certification-issues-solutions-25.png)

   >[!NOTE]
   >Este comando puede tardar un poco en completarse porque depende del tamaño del disco.

1. Desconecte el disco duro virtual de la VM y elimínela.


## <a name="default-credentials"></a>Credenciales predeterminadas

No envíe nunca las credenciales predeterminadas con el VHD. Agregar credenciales predeterminadas hace que el disco duro virtual sea más vulnerable a las amenazas de seguridad. En lugar de ello, cree sus propias credenciales al enviar el disco duro virtual.
  
## <a name="datadisk-mapped-incorrectly"></a>DataDisk se asignó incorrectamente

Se puede producir un problema de asignación cuando se envía una solicitud con varios discos de datos que no están en secuencia. Por ejemplo, el orden de numeración de tres discos de datos debe ser *0, 1, 2*. Cualquier otro orden se interpreta como un problema de asignación.

Vuelva a enviar la solicitud con una secuencia adecuada de los discos de datos.

## <a name="incorrect-os-mapping"></a>Asignación de SO incorrecta

Cuando se crea una imagen, se podría asignar a una etiqueta de sistema operativo incorrecta. Por ejemplo, cuando selecciona **Windows** como parte del nombre del sistema operativo al crear la imagen, el disco del sistema operativo debe instalarse solo con Windows. El mismo requisito se aplica a Linux.

## <a name="vm-not-generalized"></a>Máquina virtual no generalizada

Si se van a volver a usar todas las imágenes tomadas de Azure Marketplace, el disco duro virtual del sistema operativo debe generalizarse.

- En el caso de **Linux**, el siguiente proceso generaliza una máquina virtual Linux y la vuelve a implementar como una máquina virtual independiente.

  En la ventana de SSH, escriba el siguiente comando: `sudo waagent -deprovision+user`.

- Para **Windows**, puede generalizar las imágenes de Windows mediante `sysreptool`.

  Para obtener más información acerca de la herramienta `sysreptool`, consulte [Información general de la preparación del sistema (Sysprep)](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

## <a name="datadisk-errors"></a>Errores de DataDisk

Consulte la tabla siguiente a fin de obtener soluciones para los errores relativos al disco de datos:

|Error|Motivo|Solución|
|---|---|---|
|`DataDisk- InvalidUrl:`|Este error puede producirse debido a un número no válido para el número de unidad lógica (LUN) al enviar la oferta.|Compruebe que la secuencia de números de LUN del disco de datos se encuentra en el Centro de partners.|
|`DataDisk- NotFound:`|Este error podría deberse a que un disco de datos no se encuentra en una dirección URL de SAS especificada.|Compruebe que el disco de datos se encuentra en la dirección URL de SAS especificada.|

## <a name="remote-access-issue"></a>Problema de acceso remoto

Recibirá este error si la opción de Protocolo de escritorio remoto (RDP) no está habilitada para la imagen de Windows.

Habilite el acceso RDP para imágenes de Windows antes de enviarlas.

## <a name="bash-history-failed"></a>Error en el historial de Bash

Si el tamaño del historial de Bash de la imagen enviada es superior a 1 kilobyte (KB), se mostrará un error. El tamaño está restringido a 1 KB para impedir que el archivo contenga información potencialmente confidencial.

Para eliminar el historial de Bash:

1. Implemente la máquina virtual y seleccione la opción **Ejecutar comando** en Azure Portal.

   ![Captura de pantalla de Azure Portal con la opción "Ejecutar comando" en el panel izquierdo.](./media/create-vm/vm-certification-issues-solutions-3.png)

1. Seleccione la primera opción **RunShellScript** y ejecute el comando: `cat /dev/null > ~/.bash_history && history -c`.

   ![Captura de pantalla de la página "Ejecutar script de comando" en Azure Portal.](./media/create-vm/vm-certification-issues-solutions-4.png)

1. Después de que el comando se ejecute correctamente, reinicie la máquina virtual.

1. Generalice la máquina virtual, tome la imagen VHD y detenga la máquina virtual.

1. Vuelva a enviar la imagen generalizada.

## <a name="request-an-exception-on-vm-images-for-select-tests"></a>Solicitud de una excepción en imágenes de máquina virtual para pruebas seleccionadas

Los publicadores pueden solicitar excepciones para algunas pruebas realizadas durante la certificación de la máquina virtual. Las excepciones se proporcionan en raros casos cuando un publicador proporciona evidencias para admitir la solicitud. El equipo de certificación se reserva el derecho a denegar o aprobar excepciones en cualquier momento.

En esta sección se describen los escenarios generales en los que los publicadores solicitan una excepción y cómo solicitar una.

### <a name="scenarios-for-exception"></a>Escenarios para la excepción

Normalmente, los publicadores solicitan excepciones en los casos siguientes:

- **Excepción para uno o varios casos de prueba**. Póngase en contacto con el [soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport) para solicitar excepciones para casos de prueba.

- **Máquinas virtuales bloqueadas/sin acceso raíz**. Algunos publicadores tienen escenarios donde es necesario bloquear las máquinas virtuales porque tienen software, como firewalls, instalado en la máquina virtual. En este caso, descargue la [herramienta de prueba certificada](https://aka.ms/AzureCertificationTestTool) y envíe el informe al [soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport).

- **Plantillas personalizadas**. Algunos publicadores publican imágenes de máquinas virtuales que requieren una plantilla de Azure Resource Manager (ARM) personalizada para implementar las máquinas virtuales. En este caso, envíe las plantillas personalizadas al [soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport) para que el equipo de certificación pueda usarlas para la validación.

### <a name="information-to-provide-for-exception-scenarios"></a>Información que se debe proporcionar para escenarios de excepción

Póngase en contacto con el [soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport) para solicitar una excepción en uno de los escenarios e incluya la siguiente información:

- **Id. de editor**. Escriba el id. del publicador del portal central de partners.
- **Id. o nombre de la oferta**. Escriba el id. o el nombre de la oferta.
- **SKU o id. de plan**. Escriba el id. de plan o SKU de la oferta de VM.
- **Versión**. Escriba la versión de la oferta de máquina virtual que requiere una excepción.
- **Tipo de excepción**. Elija entre pruebas, máquina virtual bloqueada, plantillas personalizadas.
- **Motivo de la solicitud**. Incluya el motivo de la solicitud de excepción, además de cualquier información sobre las exenciones de prueba.
- **Escala de tiempo**. Escriba la fecha de finalización de la excepción.
- **Datos adjuntos**. Documentos de evidencia importantes adjuntos:

  - En el caso de las máquinas virtuales bloqueadas, adjunte el informe de prueba.
  - En el caso de las plantillas personalizadas, proporcione la plantilla de Resource Manager personalizada como datos adjuntos.

  Si no incluye estos datos adjuntos, se denegará la solicitud.

## <a name="address-a-vulnerability-or-an-exploit-in-a-vm-offer"></a>Resolución de una vulnerabilidad o vulnerabilidad de seguridad en una oferta de máquina virtual

En esta sección se describe cómo proporcionar una nueva imagen de máquina virtual cuando se detecta una vulnerabilidad de seguridad con una de las imágenes de máquina virtual. Solo se aplica a las ofertas de máquina virtual de Azure publicadas en Azure Marketplace.

> [!NOTE]
> No se puede quitar la última imagen de la máquina virtual de un plan y no se puede detener la venta del último plan para una oferta.

Realice alguna de las siguientes acciones:

- Si tiene una nueva imagen de máquina virtual para reemplazar la imagen de máquina virtual vulnerable, consulte [Cómo proporcionar una imagen de máquina virtual fija](#provide-a-fixed-vm-image) a continuación.
- Si no tiene una nueva imagen de máquina virtual para reemplazar la única imagen de máquina virtual en un plan y si ha terminado con el plan, puede [detener la venta del plan](update-existing-offer.md#stop-selling-an-offer-or-plan).
- Si no tiene previsto reemplazar la única imagen de máquina virtual en la oferta, le recomendamos que [detenga la venta de la oferta](update-existing-offer.md#stop-selling-an-offer-or-plan).

### <a name="provide-a-fixed-vm-image"></a>Cómo proporcionar una imagen de máquina virtual fija

Para proporcionar una imagen de máquina virtual fija para reemplazar una imagen con una vulnerabilidad o vulnerabilidad de seguridad:

1. Proporcione una nueva imagen de máquina virtual para solucionar la vulnerabilidad de seguridad.
1. Quite la imagen de máquina virtual con vulnerabilidad de seguridad.
1. Vuelva a publicar la oferta.

#### <a name="provide-a-new-vm-image-to-address-the-security-vulnerability-or-exploit"></a>Proporcione una nueva imagen de máquina virtual para solucionar la vulnerabilidad de seguridad.

Para completar estos pasos, prepare los recursos técnicos para la imagen de máquina virtual que quiere agregar. Para más información, consulte [Creación de una máquina virtual con una base aprobada](azure-vm-create-using-approved-base.md) o [Creación de una máquina virtual con su propia imagen](azure-vm-create-using-own-image.md) y [Generación de una URI de SAS para la imagen de máquina virtual](azure-vm-get-sas-uri.md).

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
1. En el panel izquierdo, seleccione **Marketplace comercial** > **Información general**.
1. En la columna **Alias de la oferta**, seleccione la oferta.
1. En la pestaña **Información general del plan**, en la columna **Nombre**, seleccione el plan adecuado.
1. En la pestaña **Configuración técnica**, en **Imágenes de VM**, seleccione **+ Agregar imagen de VM**.

   > [!NOTE]
   > Solo puede agregar una imagen de máquina virtual a un plan a la vez. Para agregar varias imágenes de máquina virtual, publique la primera de ellas antes de agregar la siguiente imagen de máquina virtual.

1. En los cuadros que aparecen, proporcione una nueva versión de disco y la imagen de máquina virtual.
1. Seleccione **Guardar borrador**.

A continuación, quite la imagen de máquina virtual con la vulnerabilidad de seguridad.

#### <a name="remove-the-vm-image-with-the-security-vulnerability-or-exploit"></a>Quite la imagen de máquina virtual con la vulnerabilidad de seguridad.

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el panel izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la columna **Alias de la oferta**, seleccione la oferta.
4. En la pestaña **Información general del plan**, en la columna **Nombre**, seleccione el plan adecuado.
5. En la pestaña **Configuración técnica**, en **Imágenes de VM**, junto a la imagen de VM que quiera quitar, seleccione **Quitar imagen de VM**.
6. En el cuadro de diálogo, seleccione **Conectar**.
7. Seleccione **Guardar borrador**.

A continuación, vuelva a publicar la oferta.

#### <a name="republish-the-offer"></a>Republicación de la oferta.

1. Seleccione **Revisar y publicar**.
2. Si necesita proporcionar información al equipo de certificación, agréguelo al cuadro **Notas para la certificación**.
3. Seleccione **Publicar**.

Para completar el proceso de publicación, consulte [Revisión y publicación de ofertas](review-publish-offer.md).

### <a name="vm-images-with-limited-access-or-requiring-custom-templates"></a>Imágenes de máquina virtual con acceso limitado o que necesitan plantillas personalizadas

#### <a name="locked-down-or-ssh-disabled-offer"></a>Oferta de SSH bloqueado (o) deshabilitado

Las imágenes que se publican con SSH deshabilitado (para Linux) o RDP deshabilitado (para Windows) se tratan como máquinas virtuales bloqueadas. Hay escenarios empresariales especiales debido a que los publicadores solo permiten el acceso restringido a pocos usuarios, o bien a ninguno.

Durante las comprobaciones de validación, es posible que las máquinas virtuales bloqueadas no permitan la ejecución de determinados comandos de certificación.

#### <a name="custom-templates"></a>Plantillas personalizadas

En general, todas las imágenes que se publican en ofertas de máquinas virtuales únicas seguirán la plantilla de ARM estándar para la implementación. Pero hay escenarios en los que el publicador puede exigir personalización mientras se implementan las máquinas virtuales (por ejemplo, que se configuren varias NIC).
    
En función de los escenarios siguientes (no exhaustivos), los publicadores usarán plantillas personalizadas para implementar la máquina virtual:

- La máquina virtual necesita subredes de red adicionales.
- Metadatos adicionales que se van a insertar en la plantilla de ARM.
- Comandos que son requisitos previos para la ejecución de la plantilla de ARM.

### <a name="vm-extensions"></a>Extensiones de máquina virtual   

Las extensiones de máquina virtual (VM) de Azure son aplicaciones pequeñas que realizan tareas de automatización y configuración posterior a la implementación en máquinas virtuales de Azure. Por ejemplo, si una máquina virtual necesita que se instale software, protección antivirus o ejecutar un script en ella, se puede usar una extensión de máquina virtual. 

Las validaciones de extensión de máquina virtual Linux necesitan lo siguiente para formar parte de la imagen:

- Versión del Agente Linux de Azure superior a la 2.2.41
- Versión de Python superior a la 2.8 

Para obtener más información, visite [Extensión de máquina virtual](../virtual-machines/extensions/diagnostics-linux.md).
     
## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las propiedades de una oferta de máquina virtual](azure-vm-create-properties.md)
- [Activación de Marketplace Rewards](marketplace-rewards.md)
- Si tiene preguntas o comentarios sobre mejoras, póngase en contacto con el [equipo de soporte técnico del Centro de partners](https://aka.ms/marketplacepublishersupport).
