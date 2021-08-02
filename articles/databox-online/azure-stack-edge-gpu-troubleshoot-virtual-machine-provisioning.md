---
title: Solución de problemas de aprovisionamiento de máquinas virtuales en Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo solucionar problemas que se producen al aprovisionar una nueva máquina virtual en Azure Stack Edge Pro con GPU.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 06/04/2021
ms.author: alkohli
ms.openlocfilehash: 9913fc2e3780d9d6ab91be19913238f8002f281a
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983284"
---
# <a name="troubleshoot-vm-deployment-in-azure-stack-edge-pro-gpu"></a>Solución de problemas de implementación de máquinas virtuales en Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe cómo solucionar errores comunes al implementar máquinas virtuales en un dispositivo GPU de Azure Stack Edge. En el artículo se proporcionan instrucciones para investigar los problemas más comunes que provocan tiempos de espera y errores de aprovisionamiento de máquinas virtuales durante la creación de la interfaz de red y la máquina virtual.

Para diagnosticar cualquier error de aprovisionamiento de máquinas virtuales, revisará los registros de invitado de la máquina virtual con errores. <!--For steps to collect VM guest logs and include them in a Support package, see [Collect guest logs for VMs on Azure Stack Edge Pro](azure-stack-edge-gpu-collect-virtual-machine-guest-logs.md).-->

Para obtener instrucciones sobre los problemas que impiden la carga correcta de una imagen de máquina virtual antes de la implementación de la máquina virtual, consulte [Solución de problemas de cargas de imágenes de máquina virtual en Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-troubleshoot-virtual-machine-image-upload.md).


## <a name="vm-provisioning-timeout"></a>Tiempo de espera de aprovisionamiento de máquinas virtuales

En esta sección se proporciona la solución de problemas de las causas más comunes del tiempo de espera de aprovisionamiento de máquinas virtuales.

Cuando se haya agotado el tiempo de espera de aprovisionamiento de máquinas virtuales, verá el siguiente error: 

![Captura de pantalla del error de Azure Portal que se muestra cuando se ha agotado el tiempo de espera de aprovisionamiento de máquinas virtuales.](./media/azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning/vm-provisioning-timeout-01.png) 

Los problemas siguientes son las principales causas de los tiempos de espera de aprovisionamiento de máquinas virtuales:
- La dirección IP que asignó a la máquina virtual ya está en uso. [Más información](#vm-provisioning-timeout)
- La imagen de máquina virtual que usó para implementar la máquina virtual no se preparó correctamente. [Más información](#vm-image-not-prepared-correctly)
- No se pudo establecer contacto con la puerta de enlace predeterminada y el servidor DNS desde la máquina virtual invitada. [Más información](#gateway-dns-server-couldnt-be-reached-from-guest-vm)
- Durante una instalación de `cloud init`, `cloud init` no se ejecutó o hubo problemas mientras se ejecutaba. (Solo máquinas virtuales Linux) [Más información](#cloud-init-issues-linux-vms).
- En el caso de una máquina virtual Linux implementada con una imagen de máquina virtual personalizada, las marcas de aprovisionamiento del archivo /etc/waagent.conf no son correctas. (Solo máquinas virtuales Linux) [Más información](#provisioning-flags-set-incorrectly-linux-vms).

### <a name="ip-assigned-to-the-vm-is-already-in-use"></a>La dirección IP asignada a la máquina virtual ya está en uso

**Descripción del error:** : se asignó a la máquina virtual una dirección IP estática que ya está en uso y no se pudo aprovisionar la máquina virtual. Este error se produce cuando la dirección IP está en uso en la subred en la que se implementa la máquina virtual. Al implementar una máquina virtual mediante Azure Portal, el proceso comprueba si hay una dirección IP existente en el dispositivo, pero no puede comprobar las direcciones IP de otros servicios o máquinas virtuales que también podrían estar en la subred. 

**Solución sugerida:** use una dirección IP estática que no esté en uso o una dirección IP dinámica proporcionada por el servidor DHCP.

Para comprobar si hay una dirección IP duplicada:

- Ejecute los comandos `ping` y Test-NetConnection (`tnc`) desde cualquier dispositivo de la misma red:

  ```
  ping <IP address>
  tnc <IP address>
  tnc <IP address> -CommonTCPPort “RDP”
  ```

Si recibe una respuesta, la dirección IP que asignó a la nueva máquina virtual ya está en uso.

### <a name="vm-image-not-prepared-correctly"></a>Imagen de máquina virtual no preparada correctamente

**Descripción del error:** para preparar una imagen de máquina virtual para su uso en un dispositivo GPU de Azure Stack Edge Pro, debe seguir un flujo de trabajo específico. Debe crear una máquina virtual gen1 en Azure, personalizarla, generalizar el VHD y, luego, descargar el VHD del sistema operativo para esa máquina virtual. La imagen preparada debe ser un VHD gen1 con la extensión de nombre de archivo "vhd" y el tipo fijo.

Para información general sobre los requisitos, consulte [Creación de imágenes de máquina virtual personalizadas para un dispositivo GPU de Azure Stack Edge Pro](azure-stack-edge-gpu-create-virtual-machine-image.md). Para instrucciones sobre cómo resolver los problemas relacionados con las imágenes de máquina virtual, consulte [Solución de problemas de cargas de imágenes de máquina virtual en Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-troubleshoot-virtual-machine-image-upload.md).  

**Solución sugerida:** complete el flujo de trabajo para preparar la imagen de máquina virtual. Consulte uno los artículos siguientes como guía:

* [Flujos de trabajo de imágenes de máquina virtual personalizadas para máquinas virtuales Windows y Linux](azure-stack-edge-gpu-create-virtual-machine-image.md)
* [Preparación de una imagen generalizada a partir de un VHD de Windows](azure-stack-edge-gpu-prepare-windows-vhd-generalized-image.md)
* [Preparación de una imagen generalizada mediante un ISO](azure-stack-edge-gpu-prepare-windows-generalized-image-iso.md)
* [Uso de una imagen especializada para implementar máquinas virtuales](azure-stack-edge-gpu-deploy-virtual-machine-portal.md)


### <a name="gateway-dns-server-couldnt-be-reached-from-guest-vm"></a>No se pudo establecer contacto con la puerta de enlace y el servidor DNS desde la máquina virtual invitada

**Descripción del error:** si no se puede establecer contacto con la puerta de enlace y el servidor DNS predeterminados durante la implementación de la máquina virtual, el aprovisionamiento de la máquina virtual agotará el tiempo de espera y se producirá un error en la operación.

**Solución sugerida:** compruebe que se puede acceder a la puerta de enlace y al servidor DNS predeterminados desde la máquina virtual. Luego, repita la implementación de la máquina virtual.

Para comprobar que se puede acceder a la puerta de enlace y al servidor DNS predeterminados desde la máquina virtual, siga estos pasos:
1. [Conecte a la máquina virtual](azure-stack-edge-gpu-deploy-virtual-machine-portal.md#connect-to-a-vm).
2. Ejecute los comandos siguientes:

   ```
   ping <default gateway IP address>
   ping <DNS server IP address>
   ```

   Para averiguar las direcciones IP de la puerta de enlace y los servidores DNS predeterminados, vaya a la interfaz de usuario local del dispositivo. Seleccione el puerto que le interesa y vea la configuración de red.

   ![Captura de pantalla de la configuración predeterminada de la puerta de enlace y del servidor DNS para un puerto de un dispositivo GPU de Azure Stack Edge Pro.](./media/azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning/gateway-dns-server-settings-01.png) 


### <a name="cloud-init-issues-linux-vms"></a>Problemas de `cloud init` (máquinas virtuales Linux)

**Descripción del error:** `cloud init` no se ejecutó o hubo problemas mientras se ejecutaba. `cloud-init` se usa para personalizar una máquina virtual Linux cuando la máquina virtual arranca por primera vez. Para más información, consulte [Compatibilidad de cloud-init con máquinas virtuales en Azure](../virtual-machines/linux/using-cloud-init.md).

**Soluciones sugeridas:** para encontrar problemas que se produjeron al ejecutar `cloud init`, siga estos pasos:
1. [Conecte a la máquina virtual](azure-stack-edge-gpu-deploy-virtual-machine-portal.md#connect-to-a-vm).
1. Compruebe si hay errores de `cloud init` en los siguientes archivos de registro:

   - /var/log/cloud-init-output.log
   - /var/log/cloud-init.log
   - /var/log/waagent/log 

Para comprobar algunos de los problemas más comunes que impiden que `cloud init` se ejecute correctamente, siga estos pasos:

1. Asegúrese de que la imagen de máquina virtual se basa en `cloud init`. Ejecute el siguiente comando:

   `cloud-init --version`

   El comando debe devolver el número de versión de cloud init. Si la imagen no está basada en `cloud init`, el comando no devolverá la información de la versión.

   Para obtener ayuda con las opciones de `cloud init`, ejecute el siguiente comando:

   `cloud-init --help` 

2. Asegúrese de que la instancia de `cloud init` se puede ejecutar correctamente con el origen de datos establecido en *Azure*. 

   Cuando el origen de datos se establece en *Azure*, la entrada de los registros de *cloud-init* es similar a la siguiente.

   ![Imagen de una entrada de registro de cloud-init para una imagen de máquina virtual con el origen de datos establecido en Azure.](./media/azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning/cloud-init-log-entry-01.png) 

   Si el origen de datos no está establecido en Azure, es posible que tenga que revisar el script de `cloud init`. Para más información, consulte [Profundización en cloud-init](../virtual-machines/linux/cloud-init-deep-dive.md).


### <a name="provisioning-flags-set-incorrectly-linux-vms"></a>Marcas de aprovisionamiento establecidas incorrectamente (máquinas virtuales Linux)

**Descripción del error:** para implementar correctamente una máquina virtual Linux en Azure, el aprovisionamiento debe deshabilitarse en la imagen y habilitarse mediante `cloud init`. Las marcas de aprovisionamiento que establecen estos valores están configuradas correctamente para las imágenes de máquina virtual estándar. Si usa una imagen de máquina virtual personalizada, debe asegurarse de que son correctas. 

**Solución sugerida:** asegúrese de que las marcas de aprovisionamiento del archivo */etc/waagent.conf* tienen los valores siguientes:<!--Move details to "Create a custom VM image" when the 2 active PRs against that article have been merged. Not before Friday release.-->

   | Capacidad                      | Valor obligatorio                |
   |---------------------------------|-------------------------------|
   | Habilitación de aprovisionamiento             | `Provisioning.Enabled=n`      |
   | Confiar en cloud-init para el aprovisionamiento | `Provisioning.UseCloudInit=y` |


##  <a name="network-interface-creation-issues"></a>Problemas de creación de la interfaz de red

En esta sección se proporcionan instrucciones para los problemas que provocan errores en la creación de la interfaz de red durante la implementación de una máquina virtual.


### <a name="nic-creation-timeout"></a>Tiempo de espera de creación de NIC

**Descripción del error:** la creación de la interfaz de red en la máquina virtual no se ha completado dentro del período de tiempo de espera permitido. Este error puede deberse a problemas del servidor DHCP en su entorno. 

Para comprobar si la interfaz de red se creó correctamente, siga estos pasos:

1. En Azure Portal, vaya al recurso Azure Stack Edge del dispositivo (vaya a **Edge Services** > **Virtual machines** [Servicios perimetrales > Máquinas virtuales]). Luego, seleccione **Deployments** (Implementaciones) y vaya a la implementación de máquinas virtuales. 

1. Si no se creó correctamente una interfaz de red, verá el siguiente error.

   ![Captura de pantalla del error que se muestra en el portal cuando no se puede crear la interfaz de red.](./media/azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning/nic-creation-failed-01.png)

**Solución sugerida:** vuelva a crear la máquina virtual y asígnele una dirección IP estática.


## <a name="vm-creation-issues"></a>Problemas de creación de la máquina virtual

En esta sección se tratan los problemas comunes que se producen durante la creación de máquinas virtuales.

### <a name="not-enough-memory-to-create-the-vm"></a>Memoria insuficiente para crear la máquina virtual

**Descripción del error:** cuando no se puede crear la máquina virtual porque no hay memoria suficiente, verá el siguiente error.
 
![Captura de pantalla del error que se muestra en el portal cuando no se puede crear la máquina virtual.](./media/azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning/vm-creation-failed-01.png)

**Solución sugerida:** compruebe la memoria disponible en el dispositivo y elija el tamaño de la máquina virtual en consecuencia. Para más información, consulte [Tamaños de máquina virtual admitidos en Azure Stack Edge](azure-stack-edge-gpu-virtual-machine-sizes.md).

La memoria disponible para la implementación de una máquina virtual está supeditada a varios factores:

- La cantidad de memoria disponible en el dispositivo. Para más información, consulte las especificaciones de memoria y proceso en [Especificaciones técnicas de Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-technical-specifications-compliance.md#compute-and-memory-specifications) y [Especificaciones técnicas de Azure Stack Edge Mini R](azure-stack-edge-mini-r-technical-specifications-compliance.md#compute-memory).

- Si Kubernetes está habilitado, la memoria de proceso necesaria para Kubernetes y las aplicaciones del clúster de Kubernetes.
- La sobrecarga de cada máquina virtual en Hyper-V.

**Soluciones propuestas:**

- Use un tamaño de máquina virtual que requiera menos memoria.
- Detenga las máquinas virtuales que no se utilicen desde el portal antes de implementar la nueva máquina virtual.
- Elimine las máquinas virtuales que ya no se usen.


### <a name="insufficient-number-of-gpus-to-create-gpu-vm"></a>Número insuficiente de GPU para crear una máquina virtual con GPU

Si intenta implementar una máquina virtual en un dispositivo GPU que ya tiene habilitado Kubernetes, no habrá ninguna GPU disponible y no se podrá realizar el aprovisionamiento de la máquina virtual. Aparece este error:

![Captura de pantalla del error que se muestra en el portal cuando no se puede crear una máquina virtual con GPU debido a que no hay GPU disponibles.](./media/azure-stack-edge-gpu-troubleshoot-virtual-machine-provisioning/gpu-vm-creation-failed-01.png)

**Causas posibles:** si Kubernetes está habilitado antes de crear la máquina virtual, usará todas las GPU disponibles y no podrá crear ninguna máquina virtual del tamaño de la GPU. Puede crear tantas máquinas virtuales del tamaño de la GPU como GPU haya disponibles. El dispositivo de Azure Stack Edge puede incluir 1 o 2 GPU.

**Solución sugerida:** para ver las opciones de implementación de máquinas virtuales en un dispositivo de 1 o 2 GPU con Kubernetes configurado, consulte [Máquinas virtuales con GPU y Kubernetes](azure-stack-edge-gpu-deploy-gpu-virtual-machine.md#gpu-vms-and-kubernetes).


## <a name="next-steps"></a>Pasos siguientes

<!-- Remove link while cmdlet issue is fixed. - * [Collect a Support package that includes guest logs for a failed VM](azure-stack-edge-gpu-collect-virtual-machine-guest-logs.md)-->
* [Solución de problemas de errores de instalación de extensiones de GPU](azure-stack-edge-gpu-collect-virtual-machine-guest-logs.md)
* [Solución de problemas con Azure Resource Manager](azure-stack-edge-gpu-troubleshoot-azure-resource-manager.md)

