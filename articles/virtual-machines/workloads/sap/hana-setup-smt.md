---
title: Configuración de un servidor SMT para SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Configuración de un servidor SMT para SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: hermannd
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ebf0c710a9c3c0358bff280eed2df8485fb991e0
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579588"
---
# <a name="set-up-smt-server-for-suse-linux"></a>Configuración de servidor SMT para SUSE Linux
Las instancias grandes de SAP HANA no tienen conectividad directa a Internet. No es un proceso sencillo registrar una de estas unidades con el proveedor de sistema operativo y descargar y aplicar las actualizaciones. Una solución para SUSE Linux podría ser configurar un servidor SMT en una máquina virtual de Azure. Hospede la máquina virtual en una red virtual de Azure que esté conectada a una instancia grande de HANA. Con este tipo de servidor SMT, la unidad de instancia grande de HANA podría registrar y descargar las actualizaciones. 

Para más información sobre SUSE, consulte [Subscription Management Tool for SLES 12 SP2](https://www.suse.com/documentation/sles-12/pdfdoc/book_smt/book_smt.pdf) (Herramienta de administración de suscripciones para SLES 12 SP2). 

Los requisitos previos para la instalación de un servidor SMT que realice la tarea para SAP HANA (instancias grandes) son:

- Una red virtual de Azure conectada al circuito de ExpressRoute para SAP HANA (instancias grandes).
- Una cuenta de SUSE que esté asociada a una organización. La organización debe tener una suscripción válida de SUSE.

## <a name="install-smt-server-on-an-azure-virtual-machine"></a>Instalación de un servidor SMT en una máquina virtual de Azure

En primer lugar, inicie sesión en [SUSE Customer Center](https://scc.suse.com/) (Centro de servicios al cliente de SUSE).

Vaya a **Organization** > **Organization Credentials** (Organización - Credenciales de organización). En esa sección debería encontrar las credenciales necesarias para configurar el servidor SMT.

A continuación, instale una máquina virtual de SUSE Linux en la red virtual de Azure. Para implementar la máquina virtual, tome una imagen de SLES 12 SP2 de la galería de Azure (seleccione la imagen SUSE BYOS). En el proceso de implementación, no defina un nombre DNS y no use direcciones IP estáticas.

![Captura de pantalla de la implementación de máquina virtual para el servidor SMT](./media/hana-installation/image3_vm_deployment.png)

La máquina virtual implementada es menor y tiene la dirección IP interna 10.34.1.4 en la red virtual de Azure. El nombre de la máquina virtual es *smtserver*. Después de la instalación, se comprobó la conectividad con las unidades de SAP HANA (instancias grandes). En función de cómo haya organizado la resolución de nombres, es posible que tenga que configurar la resolución de las unidades de HANA (instancias grandes) en etc/hosts de la máquina virtual de Azure. 

Agregue un disco a la máquina virtual. Puede usar este disco para almacenar las actualizaciones ya que el propio disco de arranque podría ser demasiado pequeño. En este caso, el disco se montó en /srv/www/htdocs, como se muestra en la captura de pantalla siguiente. Un disco de 100 GB debería ser suficiente.

![Captura de pantalla que muestra el disco agregado en la ventana de PuTTy.](./media/hana-installation/image4_additional_disk_on_smtserver.PNG)

Inicie sesión en las unidades de HANA (instancias grandes), mantenga /etc/hosts y compruebe si puede acceder a la máquina virtual de Azure en la que supuestamente se ejecuta el servidor SMT a través de la red.

Después de la comprobación, inicie sesión en la máquina virtual de Azure que debe ejecutar el servidor SMT. Si usa putty para iniciar sesión en la máquina virtual, ejecute esta secuencia de comandos en la ventana de Bash:

```
cd ~
echo "export NCURSES_NO_UTF8_ACS=1" >> .bashrc
```

Reinicie el bash para activar la configuración. Después, inicie YAST.

Conecte su máquina virtual (smtserver) al sitio de SUSE.

```
smtserver:~ # SUSEConnect -r <registration code> -e s<email address> --url https://scc.suse.com
Registered SLES_SAP 12.2 x86_64
To server: https://scc.suse.com
Using E-Mail: email address
Successfully registered system.
```

Una vez que la máquina virtual esté conectada al sitio de SUSE, instale los paquetes de smt. Use el siguiente comando putty para instalar los paquetes smt.

```
smtserver:~ # zypper in smt
Refreshing service 'SUSE_Linux_Enterprise_Server_for_SAP_Applications_12_SP2_x86_64'.
Loading repository data...
Reading installed packages...
Resolving package dependencies...
```


También puede usar la herramienta YAST para instalar los paquetes smt. En YAST, vaya a **Mantenimiento de software** y busque smt. Seleccione **smt**, que cambia automáticamente a yast2-smt.

![Captura de pantalla de SMT en YAST](./media/hana-installation/image5_smt_in_yast.PNG)


Acepte la selección para la instalación en el servidor SMT. Una vez finalizada la instalación, vaya a la configuración del servidor SMT. Escriba las credenciales de la organización que recuperó anteriormente desde el Centro de servicios al cliente de SUSE. Escriba también el nombre de host de la máquina virtual de Azure y la dirección URL del servidor SMT. En esta demostración, es https:\//smtserver.

![Captura de pantalla del servidor SMT](./media/hana-installation/image6_configuration_of_smtserver1.png)

Ahora, compruebe si funciona la conexión al Centro de servicios al cliente de SUSE. Como se ve en la captura de pantalla siguiente, en el caso del ejemplo, ha funcionado.

![Captura de pantalla de la conexión de prueba al Centro de servicios al cliente de SUSE](./media/hana-installation/image7_test_connect.png)

Una vez que se inicie el programa de instalación de SMT, proporcione una contraseña de base de datos. Dado que es una instalación nueva, debe definir esa contraseña como se muestra en la captura de pantalla siguiente.

![Captura de pantalla de definición de la contraseña de la base de datos](./media/hana-installation/image8_define_db_passwd.PNG)

El siguiente paso consiste en crear un certificado.

![Captura de pantalla de creación de un certificado para el servidor SMT](./media/hana-installation/image9_certificate_creation.PNG)

Al final de la configuración, puede que tarde unos minutos en ejecutarse la comprobación de sincronización. Después de la instalación y configuración del servidor SMT, debería encontrar el repositorio de directorio bajo el punto de montaje /srv/www/htdocs/. También hay algunos subdirectorios en el repositorio. 

Reinicie el servidor SMT y sus servicios relacionados con estos comandos.

```
rcsmt restart
systemctl restart smt.service
systemctl restart apache2
```

## <a name="download-packages-onto-smt-server"></a>Descarga de paquetes al servidor SMT

Después de reiniciar todos los servicios, seleccione los paquetes adecuados en la administración de SMT con Yast. La selección de paquetes depende de la imagen del sistema operativo del servidor de HANA (instancias grandes). La selección de paquetes no depende de la versión de SLES o de la versión de la máquina virtual que ejecuta el servidor SMT. La siguiente captura de pantalla le muestra un ejemplo de la pantalla de selección.

![Captura de pantalla de la selección de paquetes](./media/hana-installation/image10_select_packages.PNG)

A continuación, inicie la copia inicial de los paquetes seleccionados en el servidor SMT que configuró. Esta copia se desencadena en el shell mediante el comando smt-mirror.

![Captura de pantalla de descarga de paquetes al servidor SMT](./media/hana-installation/image11_download_packages.PNG)

Los paquetes se deben copiar en los directorios creados bajo el punto de montaje /srv/www/htdocs. Este proceso puede tardar una hora o más, según la cantidad de paquetes que seleccione. Cuando finalice este proceso, pase a la configuración del cliente SMT. 

## <a name="set-up-the-smt-client-on-hana-large-instance-units"></a>Configurar el cliente SMT en unidades de instancia grande de HANA

En este caso, los clientes son las unidades de HANA (instancias grandes). El programa de instalación del servidor SMT copió el script clientSetup4SMT.sh en la máquina virtual de Azure. Copie ese script en la unidad de instancia grande de HANA que quiere conectar a su servidor SMT. Inicie el script con la opción -h y asígnele como parámetro el nombre del servidor SMT. En este ejemplo, el nombre es *smtserver*.

![Captura de pantalla de configuración del cliente SMT](./media/hana-installation/image12_configure_client.PNG)

Es posible que la carga del certificado desde el servidor por el cliente se realice correctamente, pero como se muestra en la siguiente captura de pantalla, no se pudo realizar el registro.

![Captura de pantalla de error de registro de cliente](./media/hana-installation/image13_registration_failed.PNG)

Si se produce un error en el registro, consulte el [documento de soporte de SUSE](https://www.suse.com/de-de/support/kb/doc/?id=7006024) y ejecute los pasos que se describen.

> [!IMPORTANT] 
> Como nombre del servidor, proporcione el nombre de la máquina virtual, en este caso *smtserver*, sin el nombre de dominio completo. 

Después de ejecutar estos pasos, ejecute el siguiente comando en la unidad de HANA (instancias grandes):

```
SUSEConnect –cleanup
```

> [!Note] 
> Espere unos minutos después de ese paso. Si ejecuta clientSetup4SMT.sh inmediatamente, podría producirse un error.

Si se produce el problema que tuvo que corregir según los pasos descritos en el artículo de SUSE, debe reiniciar clientSetup4SMT.sh en la unidad de HANA (instancias grandes) de nuevo. Ahora debería finalizar correctamente.

![Captura de pantalla de finalización correcta del registro de cliente](./media/hana-installation/image14_finish_client_config.PNG)

Ha configurado el cliente SMT de la unidad de HANA (instancias grandes) para conectarse con el servidor SMT instalado en la máquina virtual de Azure. Ahora puede ejecutar "zypper up" o "zypper in" para instalar actualizaciones del sistema operativo en HANA (instancias grandes) o instalar paquetes adicionales. Solo puede obtener las actualizaciones que descargó antes en el servidor SMT.

## <a name="next-steps"></a>Pasos siguientes
- [Instalación de HANA en HLI](hana-example-installation.md).











