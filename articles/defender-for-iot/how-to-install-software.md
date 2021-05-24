---
title: Instalación de Defender para IoT
description: Obtenga información sobre cómo instalar un sensor y la consola de administración local para Azure Defender para IoT.
ms.date: 05/04/2021
ms.topic: how-to
ms.openlocfilehash: 2bbf9440dbae8c50569e18cccef7a2a0a2a7782a
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108735393"
---
# <a name="defender-for-iot-installation"></a>Instalación de Defender para IoT

En este artículo se describe cómo instalar los siguientes elementos de Azure Defender para IoT:

- **Sensor**: los sensores de Defender para IoT recopilan el tráfico de ICS mediante la supervisión pasiva (sin agente). Los sensores, pasivos y no invasivos, tienen un impacto nulo sobre el rendimiento en las redes y los dispositivos de OT e IoT. El sensor se conecta a un puerto SPAN o a un TAP de red e inicia inmediatamente la supervisión de la red. Las detecciones se ven en la consola del sensor. Allí puede verlas, investigarlas y analizarlas en un mapa de red, un inventario de dispositivos y una amplia variedad de informes. Algunos ejemplos son los informes de evaluación de riesgos, las consultas de minería de datos y los vectores de ataque. Obtenga más información sobre las funcionalidades del sensor en la [Guía de usuario del sensor de Defender para IoT (descarga directa)](./getting-started.md).

- **Consola de administración local**: la consola de administración local le permite llevar a cabo la administración de dispositivos, riesgos y vulnerabilidades. También puede usarla para llevar a cabo la supervisión de amenazas y la respuesta a incidentes en toda la empresa. Ofrece una vista unificada de todos los dispositivos de red e indicadores de riesgo y alertas clave de OT e IoT detectados en las instalaciones en las que los sensores están implementados. Use la consola de administración local para ver y administrar sensores en redes aisladas.

En este artículo se trata la siguiente información sobre la instalación:

  - **Hardware:** detalles de los dispositivos físicos Dell y HPE.

  - **Software:** instalación del software de la consola de administración local y del sensor.

  - **Aplicaciones virtuales:** detalles sobre la máquina virtual e instalación del software.

Después de la instalación, conecte el sensor a la red.

## <a name="about-defender-for-iot-appliances"></a>Acerca de los dispositivos de Defender para IoT 

En las secciones siguientes se proporciona información sobre los dispositivos de sensor de Defender para IoT y sobre el dispositivo para la consola de administración local de Defender para IoT.

### <a name="physical-appliances"></a>Dispositivos físicos

El sensor de dispositivo de Azure Defender para IoT se conecta a un puerto SPAN o a un TAP de red e inmediatamente comienza a recopilar el tráfico de ICS mediante supervisión pasiva (sin agente). Este proceso no tiene ningún impacto en las redes y dispositivos OT, porque no se coloca en la ruta de acceso de datos y no examina de manera activa los dispositivos OT.

Están disponibles los siguientes dispositivos de montaje en bastidor:

| **Tipo de implementación** | **Corporativos** | **Empresa** | **SMB** | **Line** |
|--|--|--|--|--|
| **Modelo** | HPE ProLiant DL360 | Dell PowerEdge R340 XL | HPE ProLiant DL20 | HPE ProLiant DL20 |
| **Puertos de supervisión** | hasta 15 RJ45 u 8 OPT | hasta 9 RJ45 o 6 OPT | hasta 8 RJ45 o 6 OPT | 4 RJ45 |
| **Ancho de banda máximo\*** | 3 GB/s | 1 GB/s | 1 GB/s | 100 MB/s |
| **Número máximo de dispositivos protegidos** | 30,000 | 10 000 | 15,000 | 1,000 |

*La capacidad de ancho de banda máxima puede variar en función de la distribución de protocolos.

### <a name="virtual-appliances"></a>Dispositivos virtuales

Están disponibles las siguientes aplicaciones virtuales:

| **Tipo de implementación** | **Empresa** | **SMB** | **Line** |
|--|--|--|--|
| **Descripción** | Aplicación virtual para implementaciones Enterprise | Aplicación virtual para implementaciones de SMB | Aplicación virtual para implementaciones en línea |
| **Ancho de banda máximo\*** | 150 MB/s | 15 MB/s | 3 MB/s |
| **Número máximo de dispositivos protegidos** | 3,000 | 300 | 100 |
| **Tipo de implementación** | Enterprise | SMB | Línea |
| **Descripción** | Aplicación virtual para implementaciones Enterprise | Aplicación virtual para implementaciones de SMB | Aplicación virtual para implementaciones en línea |

*La capacidad de ancho de banda máxima puede variar en función de la distribución de protocolos.

### <a name="hardware-specifications-for-the-on-premises-management-console"></a>Especificaciones de hardware para la consola de administración local

 | Elemento | Descripción |
 |----|--|
 **Descripción** | En una arquitectura de varios niveles, la consola de administración local ofrece visibilidad y control en todos los sitios distribuidos geográficamente. Se integra con pilas de seguridad de SOC, entre las que se incluyen SIEM, sistemas de vales, firewalls de última generación, plataformas de acceso remoto seguras y el espacio aislado de malware de ICS para Defender para IoT. |
 **Tipo de implementación** | Enterprise |
 **Tipo de dispositivo**  | Dell R340, máquina virtual |
 **Número de sensores administrados** | Sin límite |

## <a name="prepare-for-the-installation"></a>Preparación de la instalación

### <a name="access-the-iso-installation-image"></a>Acceso a la imagen de instalación ISO

Se puede acceder a la imagen de instalación desde el portal de Defender para IoT.

Para acceder al archivo, haga lo siguiente:

1. Inicie sesión en su cuenta de Defender para IoT.

1. Vaya a la página **Sensor de red** o **Consola de administración local** y seleccione la versión que quiere descargar.

### <a name="install-from-dvd"></a>Instalación desde DVD

Antes de la instalación, asegúrese de tener lo siguiente:

- Una unidad de DVD portátil con conexión mediante USB.

- Una imagen de instalador ISO.

Para instalarlo:

1. Grabe la imagen en un DVD o prepare un disco en una memoria USB. Conecte una unidad de DVD portátil al equipo, haga clic con el botón secundario en la imagen ISO y seleccione **Grabar en el disco**.

1. Conecte el DVD o el disco en una memoria USB y configure el dispositivo para que arranque desde un DVD o memoria USB.

### <a name="install-from-disk-on-a-key"></a>Instalación desde un disco en memoria USB

Antes de la instalación, asegúrese de tener lo siguiente:

  - Rufus instalado.
  
  - Un disco en una memoria USB versión 3.0 de USB y posteriores. El tamaño mínimo es 4 GB.

  - Un archivo de imagen de instalador ISO.

El disco en memoria USB se borrará en este proceso.

Para preparar un disco en memoria USB:

1. Ejecute Rufus y seleccione **SENSOR ISO**.

1. Conecte el disco en una memoria USB al panel frontal.

1. Configure el BIOS del servidor para que arranque desde la USB.

## <a name="dell-poweredger340xl-installation"></a>Instalación de Dell PowerEdgeR340XL

Antes de instalar el software en el dispositivo Dell, debe cambiar la configuración del BIOS del dispositivo:

  - El [panel frontal de Dell PowerEdge R340](#dell-poweredge-r340-front-panel) y el [panel posterior de Dell PowerEdge R340](#dell-poweredge-r340-back-panel) contiene la descripción de los paneles frontal y posterior, junto con la información necesaria para la instalación, como los controladores y los puertos.

  - La [configuración de BIOS de Dell](#dell-bios-configuration) proporciona información acerca de cómo conectarse a la interfaz de administración de dispositivos de Dell y configurar el BIOS.

  - [Instalación de software (Dell R340)](#software-installation-dell-r340) describe el procedimiento necesario para instalar el software del sensor de Defender para IoT.

### <a name="dell-poweredge-r340xl-requirements"></a>Requisitos de Dell PowerEdge R340XL 

Para instalar el dispositivo Dell PowerEdge R340XL, necesitará lo siguiente:

- Licencia Enterprise para el controlador integrado de acceso remoto de Dell (iDrac)

- Archivo XML de configuración del BIOS

- Versiones de firmware del servidor:

  - Versión del BIOS 2.1.6

  - Versión de iDrac 3.23.23.23

### <a name="dell-poweredge-r340-front-panel"></a>Panel frontal de Dell PowerEdge R340

:::image type="content" source="media/tutorial-install-components/view-of-dell-poweredge-r340-front-panel.jpg" alt-text="Panel frontal de Dell PowerEdge R340.":::

 1. Panel de control izquierdo 
 1. Unidad óptica (opcional) 
 1. Panel de control derecho 
 1. Etiqueta de información 
 1. Unidades  

### <a name="dell-poweredge-r340-back-panel"></a>Panel posterior de Dell PowerEdge R340

:::image type="content" source="media/tutorial-install-components/view-of-dell-poweredge-r340-back-panel.jpg" alt-text="Panel posterior de Dell PowerEdge R340.":::

1. Puerto serie 
1. Puerto de adaptador de red (Gb 1) 
1. Puerto de adaptador de red (Gb 1) 
1. PCIe de media altura 
1. Ranura de tarjeta de expansión PCIe de altura completa 
1. Unidad de fuente de alimentación 1 
1. Unidad de fuente de alimentación 2 
1. Identificación del sistema 
1. Botón de puerto del cable indicador de estado del sistema (CMA) 
1. Puerto USB 3.0 (2) 
1. Puerto de red dedicado iDRAC9 
1. Puerto VGA 

### <a name="dell-bios-configuration"></a>Configuración del BIOS de Dell

La configuración del BIOS de Dell es necesaria para modificar el dispositivo Dell para que funcione con el software.

La configuración del BIOS se lleva a cabo a través de una configuración predefinida. Se puede acceder al archivo desde el [Centro de ayuda](https://help.cyberx-labs.com/).

Importe el archivo de configuración al dispositivo Dell. Antes de usar el archivo de configuración, debe establecer la comunicación entre el dispositivo de Dell y el equipo de administración.

Un iDRAC integrado con el controlador del ciclo de vida (LC) administra el dispositivo Dell. El LC está integrado en todos los servidores Dell PowerEdge y proporciona una funcionalidad que le ayuda a implementar, actualizar, supervisar y dar mantenimiento a sus dispositivos Dell PowerEdge.

Para establecer la comunicación entre el dispositivo Dell y el equipo de administración, debe definir la dirección IP de iDRAC y la dirección IP del equipo de administración en la misma subred.

Cuando se establece la conexión, el BIOS se puede configurar.

Para configurar el BIOS de Dell, realice lo siguiente:

1. [Configuración de las direcciones IP de iDRAC](#configure-idrac-ip-address)

1. [Importación del archivo de configuración del BIOS](#import-the-bios-configuration-file)

#### <a name="configure-idrac-ip-address"></a>Configuración de las direcciones IP de iDRAC

1. Encienda el sensor.

1. Si el sistema operativo ya está instalado, presione la tecla F2 para especificar la configuración del BIOS.

1. Seleccione **iDRAC Settings** (Configuración de iDRAC).

1. Seleccione **Network** (Red).

   > [!NOTE]
   > Durante la instalación, debe configurar la dirección IP y la contraseña de iDRAC predeterminadas que se mencionan en los siguientes pasos. Después de la instalación, deberá cambiar estas definiciones.

1. Cambie la dirección IPv4 estática en **10.100.100.250**.

1. Cambie la máscara de subred estática en **255.255.255.0**.

   :::image type="content" source="media/tutorial-install-components/idrac-network-settings-screen-v2.png" alt-text="Captura de pantalla que muestra la máscara de subred estática":::.

1. Seleccione **Volver** > **Finalizar**.

#### <a name="import-the-bios-configuration-file"></a>Importación del archivo de configuración del BIOS

En esta sección se explica cómo configurar el BIOS mediante el archivo de configuración.

1. Conecte un equipo con una dirección IP preconfigurada estática **10.100.100.200** al puerto de **iDRAC**.

   :::image type="content" source="media/tutorial-install-components/idrac-port.png" alt-text="Captura de pantalla del puerto de dirección IP preconfigurada.":::

1. Abra un explorador web y escriba **10.100.100.250** para conectarse a la interfaz web de iDRAC.

1. Inicie sesión con los privilegios de administrador predeterminados de Dell:

   - Nombre de usuario: **root**

   - Contraseña: **calvin**

1. Las credenciales del dispositivo son:

   - Nombre de usuario: **XXX**

   - Contraseña: **XXX**

     Se inicia la operación de importación del perfil de servidor.

     > [!NOTE]
     > Antes de importar el archivo, asegúrese de cumplir con lo siguiente:
     > - Es el único usuario que está conectado actualmente a iDRAC.
     > - El sistema no está en el menú del BIOS.

1. Diríjase a **Configuration** > **Server Configuration Profile** (Configuración > Perfil de configuración del servidor). Establezca los siguientes parámetros:

   :::image type="content" source="media/tutorial-install-components/configuration-screen.png" alt-text="Captura de pantalla que muestra la configuración del perfil de servidor.":::

   | Parámetro | Configuración |
   |--|--|
   | Tipo de ubicación | Seleccione **Local**. |
   | Ruta del archivo | Seleccione **Choose File** (Elegir archivo) y agregue el archivo XML de configuración. |
   | Componentes de importación | Seleccione **BIOS, NIC, RAID**. |
   | Tiempo máximo de espera | Seleccione **20 minutes** (20 minutos). |

1. Seleccione **Import** (Importar).

1. Para supervisar el proceso, diríjase a **Maintenance** > **Job Queue** (Mantenimiento > Cola de trabajos).

   :::image type="content" source="media/tutorial-install-components/view-the-job-queue.png" alt-text="Captura de pantalla que muestra la cola de trabajos":::.

#### <a name="manually-configuring-bios"></a>Configuración manual del BIOS 

Debe configurar manualmente el BIOS del dispositivo si se cumple lo siguiente:

- No compró el dispositivo en Arrow.

- Tiene un dispositivo, pero tiene acceso al archivo de configuración XML.

Después de acceder al BIOS, diríjase a **Device Settings** (Configuración del dispositivo).

Para realizar la configuración manualmente:

1. Acceda al BIOS del dispositivo directamente con un teclado y una pantalla, o bien use iDRAC.

   - Si no se trata de un dispositivo de Defender para IoT, abra un explorador web y vaya a la dirección IP que se configuró antes. Inicie sesión con los privilegios de administrador predeterminados de Dell. Use **root** como nombre de usuario y **calvin** como contraseña.

   - Si se trata de un dispositivo de Defender para IoT, inicie sesión con **XXX** como nombre de usuario y **XXX** como contraseña.

1. Después de acceder al BIOS, diríjase a **Device Settings** (Configuración del dispositivo).

1. Para elegir la configuración controlada por RAID, seleccione **Integrated RAID controller 1: Dell PERC\<PERC H330 Adapter\> Configuration Utility** (Controlador RAID integrado 1: utilidad de configuración de Dell PERC <adaptador PERC H330>).

1. Seleccione **Configuration Management** (administración de configuración).

1. Seleccione **Create Virtual Disk** (crear disco virtual).

1. En el campo **Select RAID Level** (Seleccionar nivel de RAID), seleccione **RAID5**. En el campo **Virtual Disk Name** (nombre de disco virtual), escriba **ROOT** y seleccione **Physical Disks** (discos físicos).

1. Seleccione **Check All** (comprobar todos) y después seleccione **Apply Changes** (aplicar cambios).

1. Seleccione **Aceptar**.

1. Desplácese hacia abajo y seleccione **Create Virtual Disk** (crear disco virtual).

1. Marque la casilla **Confirm** (confirmar) y, a continuación, seleccione **Yes** (sí).

1. Seleccione **Aceptar**.

1. Vuelva a la pantalla principal y seleccione **System BIOS** (BIOS del sistema).

1. Seleccione **Boot Settings** (configuración de arranque).

1. En la opción **Boot Mode** (modo de arranque), seleccione **BIOS**.

1. Seleccione **Back** (atrás) y, a continuación, seleccione **Finish** (finalizar) para salir de la configuración del BIOS.

### <a name="software-installation-dell-r340"></a>Instalación de software (Dell R340)

El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

Para instalarlo:

1. Compruebe que el soporte físico de la versión esté montado en el dispositivo de una de las siguientes maneras:

   - Conecte el CD o el disco en memoria USB con la versión.

   - Monte la imagen ISO mediante iDRAC. Después de iniciar sesión en iDRAC, seleccione la consola virtual y, a continuación, seleccione **Virtual Media** (soporte físico virtual).

1. En la sección **Map CD/DVD** (asignar CD/DVD), seleccione **Choose File** (elegir archivo).

1. Elija el archivo de imagen ISO correspondiente a esta versión en el cuadro de diálogo que se abrirá.

1. Seleccione el botón **Map Device** (asignar dispositivo).

   :::image type="content" source="media/tutorial-install-components/mapped-device-on-virtual-media-screen-v2.png" alt-text="Captura de pantalla que muestra u dispositivo asignado.":::

1. El soporte físico está montado. Seleccione **Cerrar**.

1. Inicie el dispositivo. Cuando use iDRAC, puede reiniciar los servidores al seleccionar el botón **Consul Control** (control de Consul). A continuación, en **Keyboard Macros** (macros de teclado), seleccione el botón **Apply** (aplicar) que iniciará la secuencia de teclas Ctrl+Alt+Suprimir.

1. Seleccione **English** (inglés).

1. Seleccione **SENSOR-RELEASE-\<version\> Enterprise**.

   :::image type="content" source="media/tutorial-install-components/sensor-version-select-screen-v2.png" alt-text="Seleccione la versión del sensor y el tipo de empresa.":::   

1. Defina el perfil de la aplicación y las propiedades de la red:

   :::image type="content" source="media/tutorial-install-components/appliance-profile-screen-v2.png" alt-text="Captura de pantalla que muestra el perfil del dispositivo y las propiedades de la red.":::   

   | Parámetro | Configuración |
   |--|--|
   | **Perfil de hardware** | **enterprise** |
   | **Interfaz de administración** | **eno1** |
   | **Parámetros de red (proporcionados por el cliente)** | - |
   |**dirección IP de la red de administración:** | - |
   | **máscara de subred:** | - |
   | **nombre de host del dispositivo:** | - |
   | **DNS:** | - |
   | **dirección IP de la puerta de enlace predeterminada:** | - |
   | **interfaces de entrada:** |  El sistema genera la lista de interfaces de entrada automáticamente. Para reflejar las interfaces de entrada, copie todos los elementos que se muestran en la lista separados con comas. No es necesario configurar la interfaz de puente. Esta opción solo se usa para casos de uso especiales. |

1. Después de unos diez minutos, se muestran los dos conjuntos de credenciales. Uno es para un usuario **CyberX** y otro es para un usuario **Support**.  

1. Guarde el id. de dispositivo y las contraseñas. Necesitará estas credenciales para acceder a la plataforma la primera vez que la use.

1. Seleccione **Entrar** para continuar.

## <a name="hpe-proliant-dl20-installation"></a>Instalación de HPE ProLiant DL20

En esta sección se describe el proceso de instalación de HPE ProLiant DL20, que incluye los pasos siguientes:

  - Habilite el acceso remoto y actualice la contraseña de administrador predeterminada.
  - Configure el BIOS y las opciones de RAID.
  - Instale el software.

### <a name="about-the-installation"></a>Acerca de la instalación

  - Se pueden instalar aplicaciones de Enterprise y SMB. El proceso de instalación es idéntico para ambos tipos de dispositivo, excepto para la configuración de matriz.
  - Se especifica un usuario administrativo predeterminado. Se recomienda cambiar la contraseña durante el proceso de configuración de la red.
  - Durante el proceso de configuración de la red, configurará el puerto iLO en el puerto de red 1.
  - El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

### <a name="hpe-proliant-dl20-front-panel"></a>Panel frontal del HPE ProLiant DL20

:::image type="content" source="media/tutorial-install-components/hpe-proliant-dl20-front-panel-v2.png" alt-text="Panel frontal del HPE ProLiant DL20.":::

### <a name="hpe-proliant-dl20-back-panel"></a>Panel posterior del HPE ProLiant DL20

:::image type="content" source="media/tutorial-install-components/hpe-proliant-dl20-back-panel-v2.png" alt-text="Panel posterior del HPE ProLiant DL20.":::

### <a name="enable-remote-access-and-update-the-password"></a>Habilitación del acceso remoto y actualización de la contraseña

Use el siguiente procedimiento para configurar las opciones de red y actualizar la contraseña predeterminada.

Para habilitar y actualizar la contraseña, haga lo siguiente:

1. Conecte una pantalla y un teclado al dispositivo HP, enciéndalo y presione **F9**.

    :::image type="content" source="media/tutorial-install-components/hpe-proliant-screen-v2.png" alt-text="Captura de pantalla en la que se muestra la ventana HPE ProLiant.":::

1. Diríjase a **System Utilities** > **System Configuration** > **iLO 5 Configuration Utility** > **Network Options** (Utilidades del sistema > Configuración del sistema > Utilidad de configuración iLO 5 > Opciones de red).

    :::image type="content" source="media/tutorial-install-components/system-configuration-window-v2.png" alt-text="Captura de pantalla que muestra la ventana Configuración del sistema.":::

    1.  Seleccione **Shared Network Port-LOM** (puerto de red compartido-LOM) en el campo **Network Interface Adapter** (adaptador de interfaz de red).
    
    1.  Deshabilite el DHCP.
    
    1.  Escriba la dirección IP, la máscara de subred y la dirección IP de puerta de enlace.

1. Seleccione **F10: Save** (F10: guardar).

1. Seleccione **Esc** para volver a la **utilidad de configuración iLO 5** y después seleccione **User Management** (administración de usuarios).

1. Seleccione **Edit/Remove User** (editar/quitar usuario). El administrador es el único usuario predeterminado definido. 

1. Cambie la contraseña predeterminada y seleccione **F10: Save** (F10: guardar).

### <a name="configure-the-hpe-bios"></a>Configuración del BIOS de HPE

En el procedimiento siguiente se describe cómo configurar el BIOS de HPE para las aplicaciones de Enterprise y SMB.

Para configurar el BIOS de HPE, haga lo siguiente:

1. Seleccione **System Utilities** > **System Configuration** > **BIOS/Platform Configuration (RBSU)** (utilidades del sistema > configuración del sistema > configuración de la plataforma/BIOS [RBSU]).

1. En el formulario **BIOS/Platform Configuration (RBSU)** (configuración de la plataforma/BIOS [RBSU]), seleccione **Boot Options** (opciones de arranque).

1. Cambie el **Boot Mode** (modo de arranque) en **Legacy BIOS Mode** (modo de BIOS heredado) y después seleccione **F10: Save** (F10: guardar).

1. Seleccione **Esc** dos veces para cerrar el formulario **System Configuration** (configuración del sistema).

#### <a name="for-the-enterprise-appliance"></a>Para el dispositivo Enterprise, haga lo siguiente

1. Seleccione **Embedded RAID 1: HPE Smart Array P408i-a SR Gen 10** > **Array Configuration** > **Create Array** (RAID integrado 1: HPE Smart Array P408i-a SR Gen 10 > configuración de matriz > crear matriz).

1. En el formulario **Create Array** (crear matriz), seleccione todas las opciones. Hay tres opciones disponibles para el dispositivo **Enterprise**.

#### <a name="for-the-smb-appliance"></a>Para el dispositivo SMB, haga lo siguiente

1. Seleccione **Embedded RAID 1: HPE Smart Array P208i-a SR Gen 10** > **Array Configuration** > **Create Array** (RAID incrustado 1: HPE Smart Array P208i-a SR Gen 10 > configuración de matriz > crear matriz).

1. Seleccione **Proceed to Next Form** (continuar con el siguiente formulario).

1. En el formulario **Set RAID Level** (establecer nivel de RAID), establezca el nivel en **RAID 5** para las implementaciones Enterprise y **RAID 1** para las implementaciones de SMB.

1. Seleccione **Proceed to Next Form** (continuar con el siguiente formulario).

1. En el formulario **Logical Drive Label** (etiqueta de unidad lógica) escriba **Logical Drive 1**.

1. Seleccione **Submit Changes** (enviar cambios).

1. En el formulario **Submit** (enviar), seleccione **Back to Main Menu** (volver al menú principal).

1. Seleccione **F10: Save** (F10: guardar) y después presione **Esc** dos veces.

1. En la ventana **System Utilities** (utilidades del sistema), seleccione **One-Time Boot Menu** (menú de arranque único).

1. En el formulario **One-Time Boot Menu** (menú de arranque único), seleccione **Legacy BIOS One-Time Boot Menu** (menú de arranque único del BIOS heredado).

1. Aparecen las ventanas **Booting in Legacy** (arranque en modo heredado) y **Boot Override** (invalidación de arranque). Elija una opción de invalidación de arranque; por ejemplo, en un shell de CD-ROM, USB, HDD o UEFI.

    :::image type="content" source="media/tutorial-install-components/boot-override-window-one-v2.png" alt-text="Captura de pantalla que muestra la primera ventana de invalidación de arranque.":::

    :::image type="content" source="media/tutorial-install-components/boot-override-window-two-v2.png" alt-text="Captura de pantalla que muestra la segunda ventana de invalidación de arranque.":::
### <a name="software-installation-hpe-proliant-dl20-appliance"></a>Instalación de software (dispositivo HPE ProLiant DL20)

El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

Para instalar el software, haga lo siguiente:

1. Conecte la pantalla y el teclado al dispositivo y, a continuación, conéctese a la CLI.

1. Conecte un CD externo o disco en memoria USB con la imagen ISO que descargó de la página **Actualizaciones** en el portal de Defender para IoT.

1. Inicie el dispositivo.

1. Seleccione **English** (inglés).

    :::image type="content" source="media/tutorial-install-components/select-english-screen.png" alt-text="Selección de inglés en la ventana de la CLI.":::

1. Seleccione **SENSOR-RELEASE-<version> Enterprise**.

    :::image type="content" source="media/tutorial-install-components/sensor-version-select-screen-v2.png" alt-text="Captura de pantalla de la pantalla para seleccionar una versión.":::

1. En el Asistente de instalación, defina el perfil de hardware y las propiedades de la red:

    :::image type="content" source="media/tutorial-install-components/installation-wizard-screen-v2.png" alt-text="Captura de pantalla que muestra al Asistente de instalación.":::

    | Parámetro | Configuración |
    | ----------| ------------- |
    | **Perfil de hardware** | Seleccione **Enterprise** u **Office** para las implementaciones de SMB. |
    | **Interfaz de administración** | **eno2** |
    | **Parámetros de red predeterminados (normalmente, el cliente especifica los parámetros)** | **dirección IP de la red de administración:** <br/> <br/>**nombre de host del dispositivo:** <br/>**DNS:** <br/>**la dirección IP de puerta de enlace predeterminada:**|
    | **interfaces de entrada:** | El sistema genera la lista de interfaces de entrada automáticamente.<br/><br/>Para reflejar las interfaces de entrada, copie todos los elementos que se muestran en la lista separados con comas: **eno5, eno3, eno1, eno6, eno4**.<br/><br/>**Para el HPE DL20: no incluya eno1, enp1s0f4u4 (interfaces iLO)**<br/><br/>**PUENTE**: no es necesario configurar la interfaz de puente. Esta opción solo se usa para casos de uso especiales. Presione **Entrar** para continuar. |

1. Después de unos diez minutos, se muestran los dos conjuntos de credenciales. Uno es para un usuario **CyberX** y otro es para un usuario **Support**.

1. Guarde el id. de dispositivo y las contraseñas. Necesitará las credenciales para acceder a la plataforma la primera vez.

1. Seleccione **Entrar** para continuar.

## <a name="hpe-proliant-dl360-installation"></a>Instalación de HPE ProLiant DL360

  - Se especifica un usuario administrativo predeterminado. Se recomienda cambiar la contraseña durante la configuración de la red.

  - Durante la configuración de la red, configurará el puerto iLO.

  - El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

### <a name="hpe-proliant-dl360-front-panel"></a>Panel frontal del HPE ProLiant DL360

:::image type="content" source="media/tutorial-install-components/hpe-proliant-dl360-front-panel.png" alt-text="Panel frontal del HPE ProLiant DL360.":::

### <a name="hpe-proliant-dl360-back-panel"></a>Panel posterior del HPE ProLiant DL360

:::image type="content" source="media/tutorial-install-components/hpe-proliant-dl360-back-panel.png" alt-text="Panel posterior del HPE ProLiant DL360.":::

### <a name="enable-remote-access-and-update-the-password"></a>Habilitación del acceso remoto y actualización de la contraseña

Consulte las secciones anteriores para saber sobre la instalación de HPE ProLiant DL20:

  - "Habilitación del acceso remoto y actualización de la contraseña"

  - "Configuración del BIOS de HPE"

La configuración de Enterprise es idéntica.

> [!Note]
> En el formulario de matriz, compruebe que haya seleccionado todas las opciones.

### <a name="ilo-remote-installation-from-a-virtual-drive"></a>Instalación remota de iLO (desde una unidad virtual)

En este procedimiento se describe la instalación de iLO desde una unidad virtual.

Para instalarlo:

1. Inicie sesión en la consola de iLO y, a continuación, haga clic con el botón secundario en la pantalla de servidores.

1. Seleccione **HTML5 Console** (consola HTML5).

1. En la consola, seleccione el icono de CD y elija la opción CD/DVD.

1. Seleccione **Local ISO file** (archivo ISO local).

1. En el cuadro de diálogo, elija el archivo ISO pertinente.

1. Vaya al icono de la izquierda, seleccione **Power** (inicio/apagado) y seleccione **Reset** (restablecer).

1. El dispositivo se reiniciará y ejecutará el proceso de instalación del sensor.

### <a name="software-installation-hpe-dl360"></a>Instalación de software (HPE DL360)

El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

Para instalarlo:

1. Conecte la pantalla y el teclado al dispositivo y, a continuación, conéctese a la CLI.

1. Conecte un CD externo o disco en memoria USB con la imagen ISO que descargó de la página **Actualizaciones** en el portal de Defender para IoT.

1. Inicie el dispositivo.

1. Seleccione **English** (inglés).

1. Seleccione **SENSOR-RELEASE-<version> Enterprise**.

    :::image type="content" source="media/tutorial-install-components/sensor-version-select-screen-v2.png" alt-text="Captura de pantalla que muestra la selección de la versión.":::

1. En el Asistente de instalación, defina el perfil de dispositivo y las propiedades de la red.

    :::image type="content" source="media/tutorial-install-components/installation-wizard-screen-v2.png" alt-text="Captura de pantalla que muestra al Asistente de instalación.":::

    | Parámetro | Configuración |
    | ----------| ------------- |
    | **Perfil de hardware** | Seleccione **corporate**. |
    | **Interfaz de administración** | **eno2** |
    | **Parámetros de red predeterminados (proporcionados por el cliente)** | **dirección IP de la red de administración:** <br/>**máscara de subred:** <br/>**nombre de host del dispositivo:** <br/>**DNS:** <br/>**la dirección IP de puerta de enlace predeterminada:**|
    | **interfaces de entrada:**  | El sistema genera una lista de interfaces de entrada automáticamente.<br/><br/>Para reflejar las interfaces de entrada, copie todos los elementos que se muestran en la lista separados con comas.<br/><br/> No tiene que configurar la interfaz de puente. Esta opción solo se usa para casos de uso especiales. |

1. Después de unos diez minutos, se muestran los dos conjuntos de credenciales. Uno es para un usuario **CyberX** y otro es para un usuario **support**.

1. Guarde el id. de dispositivo y las contraseñas. Necesitará estas credenciales para acceder a la plataforma la primera vez.

1. Seleccione **Entrar** para continuar.

## <a name="hp-edgeline-300-installation"></a>Instalación de HP EdgeLine 300

•   Se especifica un usuario administrativo predeterminado. Se recomienda cambiar la contraseña durante la configuración de la red.

•   El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

### <a name="hp-edgeline-300-back-panel"></a>Panel posterior de HP EdgeLine 300

:::image type="content" source="media/tutorial-install-components/edgeline-el300-panel.png" alt-text="Vista del panel posterior de EL300":::

### <a name="enable-remote-access"></a>Habilitación del acceso remoto

1. Escriba la dirección IP de iSM en el explorador web.

1. Inicie sesión con el nombre de usuario y la contraseña predeterminados que se encuentran en el dispositivo.

1. Vaya a **Wired and Wireless Network** (Red cableada e inalámbrica)  > **IPV4**.

    :::image type="content" source="media/tutorial-install-components/wired-and-wireless.png" alt-text="Vaya a las secciones resaltadas.":::

1. Deshabilite el **botón de alternancia de DHCP**.

1. Configure las direcciones IPv4 como tales:
    - **Dirección IPV4**: `192.168.1.125`
    - **Máscara de subred IPv4**: `255.255.255.0`
    - **Puerta de enlace IPv4**: `192.168.1.1`

1. Seleccione **Aplicar**.

1. Cierre sesión y reinicie el dispositivo.

### <a name="configure-the-bios"></a>Configuración del BIOS

En el procedimiento siguiente se describe cómo configurar el BIOS del dispositivo HP EL300.

Para configurar el BIOS:

1. Encienda el dispositivo y presione **F9** para entrar en el BIOS.

1. Seleccione **Advanced** (Opciones avanzadas) y desplácese hacia abajo hasta **CSM Support** (Compatibilidad con CSM).

    :::image type="content" source="media/tutorial-install-components/csm-support.png" alt-text="Habilite la compatibilidad con CSM para abrir el menú adicional.":::

1. Presione **ENTRAR** para habilitar la compatibilidad con CSM.

1. Vaya a **Storage** (Almacenamiento) y presione **+/-** para cambiarlo a Legacy (Heredado).

1. Vaya a **Video** (Vídeo) y presione **+/-** para cambiarlo a Legacy (Heredado).

    :::image type="content" source="media/tutorial-install-components/storage-and-video.png" alt-text="Navegación a almacenamiento y vídeo para cambiarlos a la configuración heredada.":::

1. Vaya a **Boot** (Arranque)  > **Boot mode select** (Selección de modo de arranque).

1. Presione **+/-** para cambiarlo a Legacy (Heredado).

    :::image type="content" source="media/tutorial-install-components/boot-mode.png" alt-text="Cambio de la selección de modo de arranque a la configuración heredada.":::

1. Vaya a **Save & Exit** (Guardar y salir).

1. Seleccione **Save Changes and Exit** (Guardar cambios y salir).

    :::image type="content" source="media/tutorial-install-components/save-and-exit.png" alt-text="Guarde los cambios y salga del sistema.":::

1. Seleccione **Yes** (Sí), y el dispositivo se reiniciará.

1. Presione **F11** para entrar en el **menú de arranque**.

1. Seleccione el dispositivo con la imagen del sensor. Ya sea **DVD** o **USB**.

1. Seleccione el idioma.

1. Seleccione **sensor-10.0.3.12-62a2a3f724 Office: 4 CPUS, 8GB RAM, 100GB STORAGE**.

    :::image type="content" source="media/tutorial-install-components/sensor-select-screen.png" alt-text="Seleccione la versión del sensor como se muestra.":::

1. En el Asistente de instalación, defina el perfil de dispositivo y las propiedades de la red:

    :::image type="content" source="media/tutorial-install-components/appliance-parameters.png" alt-text="Defina el perfil del dispositivo y las configuraciones de la red con los parámetros siguientes.":::

    | Parámetro | Configuración |
    |--|--|
    | **configure hardware profile** | **office** |
    | **configure management network interface** | **enp3s0** <br />o bien <br />**valor posible** |
    | **configure management network IP address:** | **dirección IP proporcionada por el cliente** |
    | **configure subnet mask:** | **dirección IP proporcionada por el cliente** |
    | **configure DNS:** | **dirección IP proporcionada por el cliente** |
    | **configure default gateway IP address:** | **dirección IP proporcionada por el cliente** |
    | **configure input interface(s)** | **enp4s0** <br />o bien <br />**valor posible** |
    | **configure bridge interface(s)** | N/D |

1. Acepte la configuración y, para continuar, escriba `Y`.

## <a name="sensor-installation-for-the-virtual-appliance"></a>Instalación del sensor para el aplicación virtual

Puede implementar la máquina virtual para el sensor de Defender para IoT en las siguientes arquitecturas:

| Architecture | Especificaciones | Uso | Comentarios |
|---|---|---|---|
| **Empresa** | CPU: 8<br/>Memoria: 32 GB DE RAM<br/>HDD: 1800 GB | Entorno de producción | Valor predeterminado y más común |
| **Pequeñas empresas** | CPU: 4 <br/>Memoria: 8 GB RAM<br/>HDD: 500 GB | Entornos de producción pequeños o de pruebas | -  |
| **Office** | CPU: 4<br/>Memoria: 8 GB RAM<br/>HDD: 100 GB | Entornos de prueba pequeños | -  |

### <a name="prerequisites"></a>Requisitos previos

La consola de administración local admite las opciones de implementación de Hyper-V y VMware. Antes de iniciar la instalación, asegúrese de disponer de los siguientes elementos:

  - VMware (ESXi 5.5 o posterior) o el hipervisor de Hyper-V (Windows 10 Pro o Enterprise) instalados y en funcionamiento

  - Recursos de hardware disponibles para la máquina virtual

  - Archivo de instalación ISO para el sensor de Azure Defender para IoT

Asegúrese de que el hipervisor esté en ejecución.

### <a name="create-the-virtual-machine-esxi"></a>Creación de la máquina virtual (ESXi)

1. Inicie sesión en la máquina ESXi, elija el **almacén de datos** pertinente y seleccione **Explorador del almacén de datos**.

1. **Cargue** la imagen y seleccione **Cerrar**.

1. Vaya a **Virtual Machines** (máquinas virtuales) y, a continuación, seleccione **Create/Register VM** (crear/registrar máquina virtual).

1. Seleccione **Create new virtual machine** (crear nueva máquina virtual) y después seleccione **Next** (siguiente).

1. Agregue un nombre de sensor y elija lo siguiente:

   - Compatibilidad: **&lt;última versión de ESXi&gt;**

   - Familia del SO invitado: **Linux**

   - Versión del SO invitado: **Ubuntu Linux (solo 64 bits)**

1. Seleccione **Next** (Siguiente).

1. Elija el almacén de datos pertinente y seleccione **Next** (siguiente).

1. Cambie los parámetros del hardware virtual según la arquitectura necesaria.

1. En **CD/DVD Drive 1** (unidad de CD/DVD 1), seleccione **Datastore ISO file** (archivo ISO de almacén de datos) y elija el archivo ISO que cargó anteriormente.

1. Seleccione **Siguiente** > **Finalizar**.

### <a name="create-the-virtual-machine-hyper-v"></a>Creación de la máquina virtual (Hyper-V)

En este procedimiento se describe cómo crear una máquina virtual mediante Hyper-V.

Para crear una máquina virtual, haga lo siguiente:

1. Cree un disco virtual en el administrador de Hyper-V.

1. Seleccione **format = VHDX**.

1. Seleccione **type = Dynamic Expanding**.

1. Escriba el nombre y la ubicación del VHD.

1. Escriba el tamaño requerido (según la arquitectura).   

1. Revise el resumen y seleccione **Finalizar**.

1. En el menú **Actions** (acciones), cree una nueva máquina virtual.

1. Escriba un nombre para la máquina virtual.

1. Seleccione **Specify Generation** > **Generation 1** (especificar generación > generación 1).

1. Especifique la asignación de memoria (de acuerdo con la arquitectura) y marque la casilla de la memoria dinámica.

1. Configure el adaptador de red según la topología de red del servidor.

1. Conecte el VHDX creado previamente a la máquina virtual.

1. Revise el resumen y seleccione **Finalizar**.

1. Haga clic con el botón secundario en la máquina virtual nueva y seleccione **Settings** (configuración).

1. Seleccione **Add Hardware** (agregar hardware) y agregue un nuevo adaptador de red.

1. Seleccione el conmutador virtual que se conectará a la red de administración del sensor.

1. Asignar recursos de CPU (según la arquitectura).

1. Conecte la imagen ISO de la consola de administración a una unidad de DVD virtual.

1. Inicie la máquina virtual.

2. En el menú **Actions** (acciones), seleccione **Connect** (conectar) para continuar con la instalación del software.

### <a name="software-installation-esxi-and-hyper-v"></a>Instalación del software (ESXi e Hyper-V)

En esta sección se describe la instalación del software de ESXi e Hyper-V.

Para instalarlo:

1. Abra la consola de la máquina virtual.

1. La máquina virtual se iniciará a partir de la imagen ISO y aparecerá la pantalla de selección de idioma. Seleccione **English** (inglés).

1. Seleccione la arquitectura requerida.

1. Defina el perfil de la aplicación y las propiedades de la red:

    | Parámetro | Configuración |
    | ----------| ------------- |
    | **Perfil de hardware** | &lt;arquitectura requerida&gt; |
    | **Interfaz de administración** | **ens192** |
    | **Parámetros de red (proporcionados por el cliente)** | **dirección IP de la red de administración:** <br/>**máscara de subred:** <br/>**nombre de host del dispositivo:** <br/>**DNS:** <br/>**puerta de enlace predeterminada:** <br/>**interfaces de entrada:**|
    | **interfaces de puente:** | no es necesario configurar la interfaz de puente. Esta opción solo es para casos de uso especiales. |

1. Escriba **Y** para aceptar la configuración.

1. Las credenciales de inicio de sesión se generan y presentan automáticamente. Copie el nombre de usuario y la contraseña en un lugar seguro, ya que se necesitarán para el inicio de sesión y administración.

      - **Support**: usuario administrativo para la administración de usuarios.

      - **CyberX**: equivalente de usuario raíz para acceder al dispositivo.

1. El dispositivo se reinicia.

1. Acceda a la consola de administración mediante la dirección IP configurada previamente: `https://ip_address`.

    :::image type="content" source="media/tutorial-install-components/defender-for-iot-sign-in-screen.png" alt-text="Captura de pantalla que muestra el acceso a la consola de administración.":::

## <a name="on-premises-management-console-installation"></a>instalación de la consola de administración local

Antes de instalar el software en el dispositivo, debe cambiar la configuración del BIOS del dispositivo:

### <a name="bios-configuration"></a>Configuración de BIOS

Para configurar el BIOS del dispositivo:

1. [Habilite el acceso remoto y actualice la contraseña](#enable-remote-access-and-update-the-password).

1. [Configure el BIOS](#configure-the-hpe-bios).

### <a name="software-installation"></a>Instalación de software

El proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces. 

Durante el proceso de instalación, puede agregar una NIC secundaria. Si decide no instalar la NIC secundaria durante la instalación, puede [agregar una NIC secundaria](#add-a-secondary-nic) más adelante. 

Para instalar el software, haga lo siguiente:

1. Seleccione el idioma que prefiera para el proceso de instalación.

   :::image type="content" source="media/tutorial-install-components/on-prem-language-select.png" alt-text="Seleccione el idioma que prefiera para el proceso de instalación.":::     

1. Seleccione **MANAGEMENT-RELEASE-\<version\>\<deployment type\>** .

   :::image type="content" source="media/tutorial-install-components/on-prem-install-screen.png" alt-text="Seleccione la versión.":::   

1. En el Asistente de instalación, defina las propiedades de la red:

   :::image type="content" source="media/tutorial-install-components/on-prem-first-steps-install.png" alt-text="Captura de pantalla que muestra el perfil del dispositivo.":::   

   | Parámetro | Configuración |
   |--|--|
   | **configure management network interface** | Para Dell: **eth0, eth1** <br /> Para HP: **enu1, enu2** <br /> o bien <br />**valor posible** |
   | **configure management network IP address:** | **dirección IP proporcionada por el cliente** |
   | **configure subnet mask:** | **dirección IP proporcionada por el cliente** |
   | **configure DNS:** | **dirección IP proporcionada por el cliente** |
   | **configure default gateway IP address:** | **dirección IP proporcionada por el cliente** |
   
1. **(Opcional)** Si quiere instalar una tarjeta de interfaz de red (NIC) secundaria, defina el siguiente perfil de dispositivo y las propiedades de la red:

    :::image type="content" source="media/tutorial-install-components/on-prem-secondary-nic-install.png" alt-text="Captura de pantalla que muestra las preguntas de instalación de la NIC secundaria.":::

   | Parámetro | Configuración |
   |--|--|
   | **configure sensor monitoring interface (Optional):** | **eth1** o **valor posible** |
   | **configure an IP address for the sensor monitoring interface:** | **dirección IP proporcionada por el cliente** |
   | **configure a subnet mask for the sensor monitoring interface:** | **dirección IP proporcionada por el cliente** |

1. Acepte la configuración y, para continuar, escriba `Y`. 

1. Después de unos diez minutos, se muestran los dos conjuntos de credenciales. Uno es para un usuario **CyberX** y otro es para un usuario **Support**.

   :::image type="content" source="media/tutorial-install-components/credentials-screen.png" alt-text="Copie estas credenciales, ya que no se volverán a presentar.":::  

   Guarde los nombres de usuarios y las contraseñas, necesitará estas credenciales para acceder a la plataforma la primera vez que la use.

1. Seleccione **Entrar** para continuar.

Para obtener información sobre cómo buscar el puerto físico en el dispositivo, consulte [Búsqueda del puerto](#find-your-port).

### <a name="add-a-secondary-nic"></a>Adición de una NIC secundaria

Puede mejorar la seguridad de la consola de administración local agregando una NIC secundaria. Al agregar una NIC secundaria, tendrá una dedicada para los usuarios, y la otra admitirá la configuración de una puerta de enlace para redes enrutadas. La segunda NIC está dedicada a todos los sensores conectados dentro de un intervalo de direcciones IP.

Ambas NIC tienen habilitada la interfaz de usuario (UI). Cuando el enrutamiento no es necesario, todas las características que son compatibles con la interfaz de usuario estarán disponibles en la NIC secundaria. La alta disponibilidad se ejecutará en la NIC secundaria.

Si decide no implementar una NIC secundaria, todas las características estarán disponibles a través de la NIC principal. 

Si ya ha configurado la consola de administración local y quiere agregarle una NIC secundaria, siga estos pasos:

1. Use el comando de reconfiguración de red:

    ```bash
    sudo cyberx-management-network-reconfigure
    ```

1. Escriba las siguientes respuestas a las siguientes preguntas:

    :::image type="content" source="media/tutorial-install-components/network-reconfig-command.png" alt-text="Escriba las siguientes respuestas para configurar el dispositivo.":::

    | Parámetros | Respuesta para escribir |
    |--|--|
    | **Management Network IP address** | `N` |
    | **Máscara de subred** | `N` |
    | **DNS** | `N` |
    | **Default gateway IP Address** | `N` |
    | **Sensor monitoring interface (Optional. Applicable when sensors are on a different network segment. For more information, see the Installation instructions)**| `Y`, **seleccione un valor posible** |
    | **An IP address for the sensor monitoring interface (accessible by the sensors)** | `Y`, **dirección IP proporcionada por el cliente**|
    | **A subnet mask for the sensor monitoring interface (accessible by the sensors)** | `Y`, **dirección IP proporcionada por el cliente** |
    | **Nombre de host** | **proporcionado por el cliente** |

1. Revise todas las opciones y escriba `Y` para aceptar los cambios. El sistema se reinicia.

### <a name="find-your-port"></a>Búsqueda del puerto

Si tiene problemas para localizar el puerto físico en el dispositivo, puede usar el siguiente comando:

```bash
sudo ethtool -p <port value> <time-in-seconds>
```

Este comando hará que la luz del puerto parpadee durante el período de tiempo especificado. Por ejemplo, si escribe `sudo ethtool -p eno1 120`, el puerto eno1 parpadeará durante 2 minutos, lo que le permitirá encontrar el puerto en la parte posterior del dispositivo. 

## <a name="virtual-appliance-on-premises-management-console-installation"></a>Aplicación virtual: instalación de la consola de administración local

La máquina virtual de la consola de administración local es compatible con las siguientes arquitecturas:

| Architecture | Especificaciones | Uso | 
|--|--|--|
| Enterprise <br/>(valor predeterminado y más común) | CPU: 8 <br/>Memoria: 32 GB DE RAM<br/> HDD: 1,8 TB | Entornos de producción de gran tamaño | 
| Pequeña | CPU: 4 <br/> Memoria: 8 GB RAM<br/> HDD: 500 GB | Entornos de producción de gran tamaño |
| Office | CPU: 4 <br/>Memoria: 8 GB RAM <br/> HDD: 100 GB | Entornos de prueba pequeños | 
   
### <a name="prerequisites"></a>Requisitos previos

La consola de administración local admite las opciones de implementación de Hyper-V y VMware. Antes de empezar la instalación, compruebe lo siguiente:

- VMware (ESXi 5.5 o posterior) o el hipervisor de Hyper-V (Windows 10 Pro o Enterprise) están instalados y en funcionamiento.

- Los recursos de hardware están disponibles para la máquina virtual.

- Tiene el archivo de instalación ISO para la consola de administración local.
    
- El hipervisor está en ejecución.

### <a name="create-the-virtual-machine-esxi"></a>Creación de la máquina virtual (ESXi)

Para crear la máquina virtual (ESXi), haga lo siguiente:

1. Inicie sesión en la máquina ESXi, elija el **almacén de datos** pertinente y seleccione **Explorador del almacén de datos**.

1. Cargue la imagen y seleccione **Close** (cerrar).

1. Vaya a **Virtual Machines** (máquinas virtuales).

1. Seleccione **Create/Register VM** (crear/registrar máquina virtual).

1. Seleccione **Create new virtual machine** (crear nueva máquina virtual) y seleccione **Next** (siguiente).

1. Agregue un nombre de sensor y elija lo siguiente:

   - Compatibilidad: \<latest ESXi version>

   - Familia del SO invitado: Linux

   - Versión del SO invitado: Ubuntu Linux (solo 64 bits)

1. Seleccione **Next** (Siguiente).

1. Elija el almacén de datos pertinente y seleccione **Next** (siguiente).

1. Cambie los parámetros del hardware virtual según la arquitectura necesaria.

1. En **CD/DVD Drive 1** (unidad de CD/DVD 1), seleccione **Datastore ISO file** (archivo ISO de almacén de datos) y elija el archivo ISO que cargó anteriormente.

1. Seleccione **Siguiente** > **Finalizar**.

### <a name="create-the-virtual-machine-hyper-v"></a>Creación de la máquina virtual (Hyper-V)

Para crear una máquina virtual mediante Hyper-V, haga lo siguiente:

1. Cree un disco virtual en el administrador de Hyper-V.

1. Seleccione el formato **VHDX**.

1. Seleccione **Next** (Siguiente).

1. Seleccione el tipo **Dynamic expanding** (expansión dinámica).

1. Seleccione **Next** (Siguiente).

1. Escriba el nombre y la ubicación del VHD.

1. Seleccione **Next** (Siguiente).

1. Escriba el tamaño requerido (según la arquitectura).

1. Seleccione **Next** (Siguiente).

1. Revise el resumen y seleccione **Finalizar**.

1. En el menú **Actions** (acciones), cree una nueva máquina virtual.

1. Seleccione **Next** (Siguiente).

1. Escriba un nombre para la máquina virtual.

1. Seleccione **Next** (Siguiente).

1. Seleccione **Generation** (generaci+on) y establézcalo en **Generation 1**.

1. Seleccione **Next** (Siguiente).

1. Especifique la asignación de memoria (de acuerdo con la arquitectura) y marque la casilla de la memoria dinámica.

1. Seleccione **Next** (Siguiente).

1. Configure el adaptador de red según la topología de red del servidor.

1. Seleccione **Next** (Siguiente).

1. Conecte el VHDX creado previamente a la máquina virtual.

1. Seleccione **Next** (Siguiente).

1. Revise el resumen y seleccione **Finalizar**.

1. Haga clic con el botón secundario en la máquina virtual nueva y después seleccione **Settings** (configuración).

1. Seleccione **Add Hardware** (agregar hardware) y agregue un nuevo adaptador en **Network Adapter** (adaptador de red).

1. En **Virtual Switch** (conmutador virtual), seleccione el conmutador que se conectará a la red de administración del sensor.

1. Asignar recursos de CPU (según la arquitectura).

1. Conecte la imagen ISO de la consola de administración a una unidad de DVD virtual.

1. Inicie la máquina virtual.

1. En el menú **Actions** (acciones), seleccione **Connect** (conectar) para continuar con la instalación del software.

### <a name="software-installation-esxi-and-hyper-v"></a>Instalación del software (ESXi e Hyper-V)

Al iniciar la máquina virtual, se iniciará el proceso de instalación desde la imagen ISO.

Para instalar el software, haga lo siguiente:

1. Seleccione **English** (inglés).

1. Seleccione la arquitectura necesaria para la implementación.

1. Defina la interfaz de red para la red de administración del sensor: interfaz, IP, subred, servidor DNS y puerta de enlace predeterminada.

1. Las credenciales de inicio de sesión se generan automáticamente. Guarde el nombre de usuarios y la contraseña, necesitará estas credenciales para acceder a la plataforma la primera vez que la use.

   A continuación, el dispositivo se reiniciará.

1. Acceda a la consola de administración mediante la dirección IP configurada previamente: `<https://ip_address>`.

    :::image type="content" source="media/tutorial-install-components/defender-for-iot-management-console-sign-in-screen.png" alt-text="Captura de pantalla que muestra la pantalla de inicio de sesión de la consola de administración.":::

## <a name="legacy-devices"></a>Dispositivos heredados

En esta sección se describen los dispositivos que ya no están disponibles para la compra, pero que siguen siendo compatibles con Azure Defender para IoT.

### <a name="nuvo-5006lp-installation"></a>Instalación de Nuvo 5006LP

En esta sección se proporciona el procedimiento de instalación de Nuvo 5006LP. Antes de instalar el software en el dispositivo Nuvo 5006LP, debe cambiar la configuración del BIOS del dispositivo. 

#### <a name="nuvo-5006lp-front-panel"></a>Panel frontal del Nuvo 5006LP

:::image type="content" source="media/tutorial-install-components/nuvo5006lp_frontpanel.png" alt-text="Vista del panel frontal del dispositivo Nuvo 5006LP.":::

1. Botón de encendido, indicador de alimentación
1. Conectores de vídeo DVI
1. Conectores de vídeo HDMI
1. Conectores de vídeo VGA
1. Control de encendido y apagado remoto y salida del indicador LED de estado
1. Botón Restablecer
1. Adaptador de red de administración
1. Puertos para recibir datos reflejados

#### <a name="nuvo-back-panel"></a>Panel posterior de Nuvo

:::image type="content" source="media/tutorial-install-components/nuvo5006lp_backpanel.png" alt-text="Vista del panel posterior de Nuvo 5006LP.":::

1. Ranura de tarjeta SIM
1. Micrófono y altavoces
1. Puertos COM
1. Conectores USB
1. Puerto de alimentación de corriente continua (DC IN)

#### <a name="configure-the-nuvo-5006lp-bios"></a>Configuración del BIOS de Nuvo 5006LP

El procedimiento siguiente describe cómo configurar el BIOS de Nuvo 5006LP. Asegúrese de que el sistema operativo se haya instalado previamente en el dispositivo.

Para configurar el BIOS:

1. Encienda el dispositivo.

1. Presione **F2** para entrar en la configuración del BIOS.

1. Vaya a **Power** (Encendido) y cambie Power On after Power Failure (Encender después de un error de encendido) a S0-Power On (S0-Encendido).

    :::image type="content" source="media/tutorial-install-components/nuvo-power-on.png" alt-text="Cambie el dispositivo Nuvo 5006 para que se encienda después de un error de alimentación.":::

1. Vaya a **Boot** (Arranque) y asegúrese de que **PXE Boot to LAN** (Arranque PXE a LAN) esté establecido en **Disabled** (Deshabilitado).

1. Presione **F10** para guardar y, a continuación, seleccione **Exit** (Salir). 

#### <a name="software-installation-nuvo-5006lp"></a>Instalación de software (Nuvo 5006LP)

Este proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

1. Conecte el CD externo o el disco en memoria USB con la imagen ISO.

1. Arranque el dispositivo.

1. Seleccione **English** (inglés).

1. Seleccione **XSENSE-RELEASE-<version> Office...** .

    :::image type="content" source="media/tutorial-install-components/sensor-version-select-screen-v2.png" alt-text="Seleccione la versión del sensor que se va a instalar.":::

1. Defina la arquitectura de la aplicación y las propiedades de la red:

    :::image type="content" source="media/tutorial-install-components/nuvo-profile-appliance.png" alt-text="Defina la arquitectura y las propiedades de red de Nuvo.":::

    | Parámetro | Configuración |
    | ----------| ------------- |
    | **Perfil de hardware** | Seleccione **Office**. |
    | **Interfaz de administración** | **eth0** |
    | **Management Network IP address** | **dirección IP proporcionada por el cliente** | 
    | **Management subnet mask** | **dirección IP proporcionada por el cliente** | 
    | **DNS** | **dirección IP proporcionada por el cliente** |
    | **Default gateway IP Address** | **0.0.0.0** | 
    | **Input interface** | El sistema genera automáticamente la lista de interfaces de entrada. <br />Para reflejar las interfaces de entrada, copie todos los elementos que se muestran en la lista separados con comas. |
    | **Bridge interface** | - |

1. Acepte la configuración y, para continuar, escriba `Y`.

Después de aproximadamente 10 minutos, las credenciales de inicio de sesión se generan automáticamente. Guarde el nombre de usuarios y la contraseña, necesitará estas credenciales para acceder a la plataforma la primera vez que la use.

### <a name="fitlet2-mini-sensor-installation"></a>Instalación del mini sensor Fitlet2

En esta sección se proporciona el procedimiento de instalación de Fitlet2. Antes de instalar el software en el dispositivo Fitlet, debe cambiar la configuración del BIOS del dispositivo.

#### <a name="fitlet2-front-panel"></a>Panel frontal de Fitlet2 

:::image type="content" source="media/tutorial-install-components/fitlet-front-panel.png" alt-text="Vista del panel frontal de Fitlet2.":::

#### <a name="fitlet2-back-panel"></a>Panel posterior de Fitlet2

:::image type="content" source="media/tutorial-install-components/fitlet2-back-panel.png" alt-text="Vista del panel posterior de Fitlet2.":::

#### <a name="configure-the-fitlet2-bios"></a>Configuración del BIOS de Fitlet2

1. Encienda el dispositivo.

1. Vaya a **Main** > **OS Selection** (Principal > Selección del sistema operativo).

1. Presione **+/-** para seleccionar **Linux**.

    :::image type="content" source="media/tutorial-install-components/fitlet-linux.png" alt-text="Establezca el sistema operativo en Linux para el dispositivo Fitlet2.":::

1. Compruebe que la fecha y hora del sistema se actualizan con la fecha y hora de instalación.

1. Vaya a **Advanced** (Avanzado) y seleccione **ACPI Settings** (Configuración de ACPI).

1. Seleccione **Enable Hibernation** (Habilitar hibernación) y presione **+/-** para seleccionar **Disabled** (Deshabilitado).

    :::image type="content" source="media/tutorial-install-components/disable-hibernation.png" alt-text="Deshabilitar el modo de hibernación en el dispositivo Fitlet2.":::

1. Presione **Esc**.

1. Vaya a **Advanced** > **TPM Configuration** (Avanzado > Configuración de TPM).

1. Seleccione **fTPM** y presione **+/-** para seleccionar **Disabled** (Deshabilitado).

1. Presione **Esc**.

1. Vaya a **CPU Configuration (Configuración de CPU)**  > **VT-d**.

1. Presione **+/-** para seleccionar **Enabled** (Habilitado).

1. Vaya a **CSM Configuration** > **CSM Support** (Configuración de CSM > Compatibilidad con CSM).

1. Presione **+/-** para seleccionar **Enabled** (Habilitado).
1. Vaya a **Advanced** > **Boot option filter [Legacy only]** (Avanzado > Filtro de opciones de arranque [Solo heredadas] y cambie el valor de los siguientes campos a **Legacy** (Heredado):
    - Red
    - Storage
    - Vídeo
    - Otros dispositivos PCI

    :::image type="content" source="media/tutorial-install-components/legacy-only.png" alt-text="Establecer todos los campos en Heredado.":::

1. Presione **Esc**.

1. Vaya a **Security** > **Secure Boot Customization** (Seguridad > Personalización del arranque seguro).

1. Presione **+/-** para seleccionar **Disabled** (Deshabilitado).

1. Presione **Esc**.

1. Vaya a **Boot** > **Boot mode** (Arranque > Modo de arranque) y seleccione **Legacy** (Heredado).

1. Seleccione **Boot Option #1 – [USB CD/DVD]** (Opción de arranque nº 1: [USB CD/DVD]).
 
1. Seleccione **Save & Exit** (Guardar y salir).

#### <a name="software-installation-fitlet2"></a>Instalación del software (Fitlet2)

Este proceso de instalación tarda aproximadamente 20 minutos. Después de la instalación, el sistema se reinicia varias veces.

1. Conecte el CD externo o el disco en memoria USB con la imagen ISO.

1. Arranque el dispositivo.

1. Seleccione **English** (inglés).

1. Seleccione **XSENSE-RELEASE-<version> Office...** .

    :::image type="content" source="media/tutorial-install-components/sensor-version-select-screen-v2.png" alt-text="Seleccione la versión del sensor que se va a instalar.":::

    > [!Note]
    > No seleccione Ruggedized (Resistente).

1. Defina la arquitectura de la aplicación y las propiedades de la red:

    :::image type="content" source="media/tutorial-install-components/nuvo-profile-appliance.png" alt-text="Defina la arquitectura y las propiedades de red de Nuvo.":::

    | Parámetro | Configuración |
    | ----------| ------------- |
    | **Perfil de hardware** | Seleccione **Office**. |
    | **Interfaz de administración** | **em1** |
    | **Management Network IP address** | **dirección IP proporcionada por el cliente** | 
    | **Management subnet mask** | **dirección IP proporcionada por el cliente** | 
    | **DNS** | **dirección IP proporcionada por el cliente** |
    | **Default gateway IP Address** | **0.0.0.0** | 
    | **Input interface** | El sistema genera automáticamente la lista de interfaces de entrada. <br />Para reflejar las interfaces de entrada, copie todos los elementos que se muestran en la lista separados con comas. |
    | **Bridge interface** | - |

1. Acepte la configuración y, para continuar, escriba `Y`.

Después de aproximadamente 10 minutos, las credenciales de inicio de sesión se generan automáticamente. Guarde el nombre de usuarios y la contraseña, necesitará estas credenciales para acceder a la plataforma la primera vez que la use.

## <a name="post-installation-validation"></a>Validación después de la instalación

Para validar la instalación de un dispositivo físico, debe realizar muchas pruebas. El mismo proceso de validación se realiza a todos los tipos de dispositivos.

Realice dicha validación mediante la GUI o la CLI. La validación está disponible para los usuarios **Support** y **CyberX**.

La validación después de la instalación debe incluir las siguientes pruebas:

  - **Prueba de comprobación**: compruebe que el sistema se está ejecutando.

  - **Versión**: compruebe que la versión sea correcta.

  - **ifconfig**: compruebe que todas las interfaces de entrada configuradas durante el proceso de instalación se estén ejecutando.

### <a name="checking-system-health-by-using-the-gui"></a>Comprobación del mantenimiento del sistema mediante la GUI

:::image type="content" source="media/tutorial-install-components/system-health-check-screen.png" alt-text="Captura de pantalla que muestra la comprobación del mantenimiento del sistema.":::

#### <a name="sanity"></a>Comprobación

- **Dispositivo**: ejecuta la comprobación del dispositivo. Puede realizar la misma comprobación mediante el comando de la CLI `system-sanity`.

- **Versión**: muestra la versión del dispositivo.

- **Propiedades de red**: muestra los parámetros de red del sensor.

#### <a name="redis"></a>Redis

- **Memoria**: proporciona un panorama general del uso de memoria; por ejemplo, cuánta memoria se usó y cuánta quedó disponible.

- **Clave más larga**: muestra las claves más largas que pueden causar un uso intensivo de la memoria.

#### <a name="system"></a>Sistema

- **Registro principal**: proporciona las últimas 500 filas del registro principal, lo que le permite ver las filas de registro recientes sin exportar todo el registro del sistema.

- **Administrador de tareas**: traduce las tareas que aparecen en la tabla de procesos a los siguientes niveles: 
  
  - Nivel persistente (Redis) 
  - Nivel de efectivo (SQL)

- **Estadísticas de red**: muestra las estadísticas de la red.

- **TOP**: muestra la tabla de procesos. Se trata de un comando de Linux que proporciona una vista dinámica en tiempo real del sistema en ejecución.

- **Comprobación de memoria de copia de seguridad**: proporciona el estado de la memoria de copia de seguridad y comprueba lo siguiente:
  - La ubicación de la carpeta de copia de seguridad. 
  - El tamaño de la carpeta de copia de seguridad.
  - Las limitaciones de la carpeta de copia de seguridad.
  - Cuándo se produjo la última copia de seguridad.
  - La cantidad de espacio disponible para los archivos de copia de seguridad adicionales.

- **ifconfig**: muestra los parámetros de las interfaces físicas del dispositivo.

- **CyberX nload**: muestra el ancho de banda y el tráfico mediante pruebas de seis segundos.

- **Errores del registro principal**: muestra los errores del archivo de registro principal.

Para acceder a la herramienta, haga lo siguiente:

1. Inicie sesión en el sensor con las credenciales del usuario **Support**.

1. Seleccione **System Statistics** (estadísticas del sistema) en la ventana **System Settings** (configuración del sistema).

    :::image type="icon" source="media/tutorial-install-components/system-statistics-icon.png" border="false":::

### <a name="checking-system-health-by-using-the-cli"></a>Comprobación del mantenimiento del sistema mediante la CLI

**Prueba 1: Comprobación**

Compruebe que el sistema está en funcionamiento:

1. Conéctese a la CLI con el terminal de Linux (por ejemplo, PuTTy) y el usuario **Support**.

1. Escriba `system sanity`.

1. Compruebe que todos los servicios están en verde (en ejecución).

    :::image type="content" source="media/tutorial-install-components/support-screen.png" alt-text="Captura de pantalla que muestra los servicios en ejecución.":::

1. Compruebe que se muestre el mensaje **System is UP! (prod)** (el sistema funciona [producción]) en la parte inferior.

**Prueba 2: Comprobación de versión**

Compruebe que se está usando la versión correcta:

1. Conéctese a la CLI con el terminal de Linux (por ejemplo, PuTTy) y el usuario **Support**.

1. Escriba `system version`.

1. Compruebe que se muestre la versión correcta.

**Prueba 3: Validación de la red**

Compruebe que todas las interfaces de entrada configuradas durante el proceso de instalación se estén ejecutando:

1. Conéctese a la CLI con el terminal de Linux (por ejemplo, PuTTy) y el usuario **Support**.

1. Escriba `network list` (el equivalente del comando de Linux `ifconfig`).

1. Compruebe que se muestren las interfaces de entrada necesarias. Por ejemplo, si están instalados dos adaptadores de red de cuatro puertos de cobre, debe haber diez interfaces en la lista.

    :::image type="content" source="media/tutorial-install-components/interface-list-screen.png" alt-text="Captura de pantalla que muestra la lista de interfaces.":::

**Prueba 4: Acceso de administración a la interfaz de usuario**

Compruebe que puede acceder a la GUI web de la consola:

1. Conecte un portátil con un cable Ethernet al puerto de administración (**Gb1**).

1. Defina la dirección del adaptador de red del portátil para que esté en el mismo intervalo que el dispositivo.

    :::image type="content" source="media/tutorial-install-components/access-to-ui.png" alt-text="Captura de pantalla que muestra el acceso de administración a la interfaz de usuario.":::

1. Haga ping a la dirección IP del dispositivo desde el equipo portátil para comprobar la conectividad (valor predeterminado: 10.100.10.1).

1. Abra el explorador web Chrome en el equipo portátil y escriba la dirección IP del dispositivo.

1. En la ventana **La conexión no es privada**, seleccione **Avanzado** y continúe.

1. La prueba se realiza correctamente cuando aparece la pantalla de inicio de sesión de Defender para IoT.

   :::image type="content" source="media/tutorial-install-components/defender-for-iot-sign-in-screen.png" alt-text="Captura de pantalla que muestra el acceso a la consola de administración.":::

## <a name="troubleshooting"></a>Solución de problemas

### <a name="you-cant-connect-by-using-a-web-interface"></a>No es posible conectarse mediante una interfaz web

1. Compruebe que el equipo que está intentando conectar esté en la misma red que el dispositivo.

1. Compruebe que la red de GUI esté conectada al puerto de administración.

1. Haga ping en la dirección IP del dispositivo. Si no se encuentra un ping:

   1. Conecte un monitor y un teclado al dispositivo.

   1. Use el usuario y la contraseña de **Support** para iniciar sesión.

   1. Use el comando `network list` para ver la dirección IP actual.

      :::image type="content" source="media/tutorial-install-components/network-list.png" alt-text="Captura de pantalla que muestra la lista de redes.":::

1. Si los parámetros de red no están configurados correctamente, use el procedimiento siguiente para cambiarlos:

   1. use el comando `network edit-settings`.

   1. Para cambiar la dirección IP de la red de administración, seleccione **Y**.

   1. Para cambiar la máscara de subred, seleccione **Y**.

   1. Para cambiar el DNS, seleccione **Y**.

   1. Para cambiar la dirección IP de puerta de enlace predeterminada, seleccione **Y**.

   1. Para el cambio de la interfaz de entrada (solo para sensor), seleccione **N**.

   1. Seleccione **Y** para aplicar los cambios.

1. Después del reinicio, conéctese con las credenciales del usuario de soporte técnico y use el comando `network list` para comprobar que los parámetros se hayan cambiado.

1. Intente hacer ping y volver a conectarse desde la GUI.

### <a name="the-appliance-isnt-responding"></a>El dispositivo no responde

1. Conecte un monitor y un teclado al dispositivo, o use PuTTY para conectarse de forma remota a la CLI.

1. Use las credenciales del usuario **Support** para iniciar sesión.

1. Use el comando `system sanity` y compruebe que todos los procesos se están ejecutando.

    :::image type="content" source="media/tutorial-install-components/system-sanity-screen.png" alt-text="Captura de pantalla que muestra el comando de comprobación del sistema.":::

Para cualquier otra incidencia, póngase en contacto con [Soporte técnico de Microsoft](https://support.microsoft.com/en-us/supportforbusiness/productselection?sapId=82c88f35-1b8e-f274-ec11-c6efdd6dd099).

## <a name="appendix-a-mirroring-port-on-vswitch-esxi"></a>Apéndice A: Puerto de creación de reflejo en vSwitch (ESXi)

### <a name="configure-a-span-port-on-an-existing-vswitch"></a>Configuración de un puerto SPAN en una instancia de vSwitch existente

Una instancia de vSwitch no tiene funcionalidades de creación de reflejo, pero se puede usar una solución alternativa para implementar un puerto SPAN.

Para configurar un puerto SPAN, haga lo siguiente:

1. Abra las propiedades de vSwitch.

1. Seleccione **Agregar**.

1. Seleccione **Virtual Machine** > **Next** (máquina virtual > siguiente).

1. Inserte una etiqueta de red **SPAN Network**, seleccione **VLAN ID** > **All** (id. de VLAN > todos) y, después, seleccione **Next** (siguiente).

1. Seleccione **Finalizar**.

1. Seleccione **SPAN Network** > **Edit* (red SPAN > editar).

1. Seleccione **Security** (seguridad) y compruebe que la directiva **Promiscuous Mode** (modo promiscuo) esté establecida en el modo **Accept** (aceptar).

1. Seleccione **OK** (aceptar) y, después, seleccione **Close** (cerrar) para cerrar las propiedades de vSwitch.

1. Abras la propiedades de **XSense VM**.

1. En **Network Adapter 2** (adaptador de red 2), seleccione la red **SPAN**.

1. Seleccione **Aceptar**.

1. Conéctese al sensor y compruebe que la creación de reflejos funcione.

## <a name="appendix-b-access-sensors-from-the-on-premises-management-console"></a>Apéndice B: Acceso a los sensores desde la consola de administración local

Puede mejorar la seguridad del sistema al impedir el acceso de usuario directo al sensor. En su lugar, use la tunelización de proxy para permitir que los usuarios accedan al sensor desde la consola de administración local con una sola regla de firewall. Esta técnica reduce la posibilidad del acceso no autorizado al entorno de red más allá del sensor. La experiencia del usuario al iniciar sesión en el sensor seguirá siendo la misma.

:::image type="content" source="media/tutorial-install-components/sensor-system-graph.png" alt-text="Captura de pantalla que muestra el acceso al sensor.":::

Para habilitar la tunelización, haga lo siguiente:

1. Inicie sesión en la CLI de la consola de administración local con las credenciales de usuario **CyberX** o **Support**.

1. Escriba `sudo cyberx-management-tunnel-enable`.

1. Seleccione **Entrar**.

1. Escriba `--port 10000`.

### <a name="next-steps"></a>Pasos siguientes

[Configuración de la red](how-to-set-up-your-network.md)
