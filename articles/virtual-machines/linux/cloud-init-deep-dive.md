---
title: Reconocimiento de cloud-init
description: Profundización para comprender el aprovisionamiento de una VM de Azure con cloud-init.
author: danielsollondon
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.subservice: cloud-init
ms.openlocfilehash: 63bc821648348c2936d437fef7fdd89314fad3c5
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783232"
---
# <a name="diving-deeper-into-cloud-init"></a>Profundizar en cloud-init
Para obtener más información acerca de [cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) o solucionar problemas en un nivel más profundo, debe entender cómo funciona. En este documento se resaltan las partes importantes y se explican las características específicas de Azure.

Cuando cloud-init se incluye en una imagen generalizada y se crea una VM a partir de esa imagen, procesará las configuraciones y se ejecutará en 5 fases durante el arranque inicial. Estas fases son importantes, ya que le muestran en qué punto cloud-init va a aplicar las configuraciones. 


## <a name="understand-cloud-init-configuration"></a>Descripción de la configuración de cloud-init
La configuración de una VM para que se ejecute en una plataforma significa que cloud-init debe aplicar varias configuraciones, como un consumidor de imágenes, las principales configuraciones con las que va a interactuar son `User data` (customData), que admite varios formatos, estos se documentan [aquí](https://cloudinit.readthedocs.io/en/latest/topics/format.html#user-data-formats). También tiene la posibilidad de agregar y ejecutar scripts (/var/lib/cloud/scripts) para una configuración adicional. A continuación se describe con más detalle.

Algunas configuraciones ya están simuladas mediante "bake" en las imágenes de Azure Marketplace que se incluyen en cloud-init, como:

1. **Origen de datos en la nube**: cloud-init contiene código que puede interactuar con plataformas en la nube, que se denominan "datasources". Cuando se crea una VM a partir de una imagen de cloud-init en [Azure](https://cloudinit.readthedocs.io/en/latest/topics/datasources/azure.html#azure), cloud-init carga el origen de datos de Azure, que interactuará con los puntos de conexión de metadatos de Azure para obtener la configuración específica de la máquina virtual.
2. **Configuración de runtime** (/run/cloud-init)
3. **Configuración de imagen** (/etc/Cloud), como `/etc/cloud/cloud.cfg`, `/etc/cloud/cloud.cfg.d/*.cfg`. Un ejemplo de dónde se usa en Azure, es común que las imágenes del sistema operativo Linux con cloud-init tengan una directiva de Azure DataSource que indique a cloud-init qué datasource debe usar, esto ahorra tiempo de cloud-init:

   ```bash
   /etc/cloud/cloud.cfg.d# cat 90_dpkg.cfg
   # to update this file, run dpkg-reconfigure cloud-init
   datasource_list: [ Azure ]
   ```


## <a name="cloud-init-boot-stages-processing-configuration"></a>Fases de arranque de cloud-init (configuración de procesamiento)

Al aprovisionar con cloud-init, hay cinco fases de arranque, que procesan la configuración y se muestran en los registros.

1. [Fase generador](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#generator): El generador del sistema cloud-init inicia y determina que cloud-init debe incluirse en los objetivos de arranque y, si es así, habilita cloud-init. 

2. [Fase local de cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#local): Aquí cloud-init buscará el datasource local de "Azure", que permitirá a cloud-init interactuar con Azure, y aplicar una configuración de red, incluido el respaldo.

3. [Fase inicial de cloud-init (red)](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#network): Las redes deben estar en línea y se debe generar la información de NIC y de la tabla de rutas. En esta fase, se ejecutarán los módulos enumerados en `cloud_init_modules` /etc/cloud/cloud.cfg. Se montará la VM en Azure, se formateará el disco efímero, se fijará el nombre del host, junto con otras tareas.

   Estos son algunos de los `cloud_init_modules`:
   
   ```bash
   - migrator
   - seed_random
   - bootcmd
   - write-files
   - growpart
   - resizefs
   - disk_setup
   - mounts
   - set_hostname
   - update_hostname
   - ssh
   ```
   
   Después de esta fase, cloud-init indicará a la plataforma Azure que la VM se ha aprovisionado correctamente. Es posible que se haya producido un error en algunos módulos, no todos los errores de módulo provocarán un error de aprovisionamiento.

4. [Fase de creación de Cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#config): En esta fase, se ejecutarán los módulos de `cloud_config_modules` definidos y enumerados en /etc/Cloud/Cloud.cfg.


5. [Fase final de cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#final): En esta fase final, se ejecutarán los módulos de `cloud_final_modules`, enumerados en /etc/Cloud/Cloud.cfg. Aquí están los módulos que deben ejecutarse al final del proceso de arranque, como la instalación de paquetes y la ejecución de scripts, etc. 

   -   Durante esta fase, puede ejecutar los scripts colocándolos en los directorios en `/var/lib/cloud/scripts`:
   - `per-boot`: los scripts dentro de este directorio, se ejecutan en cada reinicio
   - `per-instance`: los scripts de este directorio se ejecutan cuando se inicia por primera vez una nueva instancia
   - `per-once`: los scripts dentro de este directorio se ejecutan solo una vez

## <a name="next-steps"></a>Pasos siguientes

[Solución de problemas de cloud-init](cloud-init-troubleshooting.md).
