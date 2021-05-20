---
title: Sincronizar la hora en las máquinas virtuales de Linux en Azure
description: Sincronice la hora de las máquinas virtuales Linux.
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/30/2021
ms.author: cynthn
ms.openlocfilehash: c50e39db804a18d50f4a6fb594209cc015515a8c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754746"
---
# <a name="time-sync-for-linux-vms-in-azure"></a>Sincronizar la hora en las máquinas virtuales de Linux en Azure

La sincronización de la hora es importante para la seguridad y la correlación de eventos. A veces se usa en la implementación de transacciones distribuidas. La precisión de hora entre varios sistemas de equipos se logra mediante la sincronización. Además, la sincronización puede verse afectada por varias cosas, incluyendo las operaciones de reinicio y el tráfico de red entre el origen de la hora y el equipo encargado de capturar la hora. 

Por ello, Azure cuenta con la infraestructura que ejecuta Windows Server 2016. Windows Server 2016 ha mejorado los algoritmos que se usan para corregir la hora y la condición del reloj local, y así poder sincronizarse con la hora UTC.  La característica de hora precisa de Windows Server 2016 ha mejorado en gran medida la manera en que el servicio VMICTimeSync rige las máquinas virtuales con el host para obtener la hora exacta. Las mejoras incluyen una hora inicial más precisa en el inicio o la restauración de la máquina virtual, así como la interrupción de la corrección de la latencia. 

> [!NOTE]
> Para obtener información general rápida del servicio Horario de Windows, eche un vistazo a este [vídeo de información general de alto nivel](https://aka.ms/WS2016TimeVideo).
>
> Para obtener más información, consulte el artículo [Hora de Windows Server 2016 precisa](/windows-server/networking/windows-time-service/accurate-time). 

## <a name="overview"></a>Información general

La precisión del reloj de un equipo se mide en función de lo cerca que esté ese reloj de la hora universal coordinada (UTC) estándar. La UTC se define mediante una muestra internacional de relojes atómicos precisos que solo se retrasan un segundo en 300 años. Aún así, es necesario tener hardware especializado para leer la UTC directamente. En cambio, los servidores horarios se sincronizan con la UTC y se accede a ellos desde otros equipos para proporcionar escalabilidad y robustez. Cada equipo tiene un servicio de sincronización de hora en ejecución que sabe qué servidores horarios debe usar y que comprueba periódicamente si el reloj del equipo necesita ser corregido ajustándolo si fuera necesario. 

Los hosts de Azure se sincronizan con los servidores horarios internos de Microsoft que ajustan su hora a partir de los dispositivos Stratum 1 que pertenecen a Microsoft, y que cuentan con antenas GPS. Las máquinas virtuales en Azure pueden depender de su host para proporcionar la hora exacta (*hora del host*) a la máquina virtual, o la máquina virtual puede obtener directamente la hora de un servidor horario, o de una combinación de ambas opciones. 

En el hardware independiente, el sistema operativo Linux solo lee el reloj del hardware del host en el arranque. Después de eso, el reloj se mantiene gracias al temporizador de interrupción en el kernel de Linux. En esta configuración, el reloj dejará de ser preciso con el tiempo. En las nuevas distribuciones de Linux en Azure, las máquinas virtuales pueden usar el proveedor VMICTimeSync que está incluido en los servicios de integración de Linux (LIS), para consultar con más frecuencia actualizaciones del reloj desde el host.

Las interacciones de la máquina virtual con el host también pueden afectar al reloj. Durante [el mantenimiento de conservación de la memoria](../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot) las máquinas virtuales se detienen hasta 30 segundos. Por ejemplo, antes de que comience el proceso de mantenimiento que dura 28 segundos, el reloj de la máquina virtual muestra las 10:00:00 A.M. Cuando se reanuda la máquina virtual, el reloj de la misma todavía muestra las 10:00:00 A.M., por lo que tendría 28 segundos de retraso. Para corregir este problema, el servicio VMICTimeSync supervisa lo que sucede en el host y actualiza el reloj de hora del día en las máquinas virtuales de Linux para compensar este retraso.

Por supuesto, si la sincronización de la hora no funciona, el reloj de la máquina virtual solo acumularía errores. Cuando solo hay una máquina virtual, este efecto puede no ser significativo a menos que la carga de trabajo requiera un cronometraje altamente preciso. Aún así, en la mayoría de los casos hay varias máquinas virtuales conectadas entre sí que usan la hora para realizar el seguimiento de las transacciones, por lo que esta hora debe ser consistente en toda la implementación. Cuando la hora de las máquinas virtuales es diferente, puede ver los siguientes efectos:

- Se produce un error en la autenticación. Los protocolos de seguridad como Kerberos o la tecnología dependiente de los certificados confían en que la hora sea consistente en todos los sistemas.
- Es muy difícil averiguar qué ha ocurrido en un sistema si los registros (u otros datos) tienen horas diferentes. Parecería que el mismo evento ocurrió en diferentes momentos, lo que dificulta la correlación.
- Si el reloj está atrasado, es posible que la facturación no se calcule correctamente.


## <a name="configuration-options"></a>Opciones de configuración

La sincronización de hora requiere que se ejecute un servicio de sincronización de hora en la máquina virtual de Linux, además de una fuente de información de hora precisa con la que sincronizar.
Normalmente, se usa ntpd o chronyd como servicio de sincronización de hora, aunque hay otros servicios de sincronización de hora de código abierto que también se pueden usar.
El origen de información de hora precisa puede ser el host de Azure o un servicio de hora externo al que se accede a través de la red pública de Internet.
Por sí mismo, el servicio VMICTimeSync no proporciona sincronización de hora continua entre el host de Azure y una máquina virtual de Linux, excepto después de pausas para el mantenimiento del host, como se describió anteriormente. 

Históricamente, la mayoría de imágenes de Azure Marketplace con Linux se han configurado de una de estas dos maneras:
- No se está ejecutando ningún servicio de sincronización de hora de manera predeterminada.
- ntpd se ejecuta como servicio de sincronización de hora y se sincroniza con un origen de hora NTP externo al que se accede a través de la red. Por ejemplo, las imágenes de Marketplace de Ubuntu 18.04 LTS usan **ntp.ubuntu.com**.

Para confirmar que ntpd se está sincronizando correctamente, ejecute el comando `ntpq -p`.

A partir de principios de 2021, las imágenes de Azure Marketplace más actuales con Linux se cambian para usar chronyd como servicio de sincronización de hora, y chronyd está configurado para sincronizarse con el host de Azure en lugar de con un origen de hora NTP externo. La hora del host de Azure suele ser el mejor origen de hora con el que sincronizarse, ya que se mantiene de forma muy precisa y confiable, y es accesible sin los retrasos de red variables inherentes al acceso a un origen de hora NTP externo a través de la red pública de Internet.

VMICTimeSync se usa en paralelo y proporciona dos funciones:
- Actualiza inmediatamente el reloj de la hora del día de la máquina virtual de Linux después de un evento de mantenimiento del host.
- Crea instancias de un origen de reloj de hardware del Protocolo de tiempo de precisión (PTP) IEEE 1588 como un dispositivo /dev/ptp que proporciona la hora del día precisa del host de Azure.  Chronyd se puede configurar para sincronizarse con este origen de hora (que es la configuración predeterminada en las imágenes de Linux más nuevas). Las distribuciones de Linux con la versión de kernel 4.11 o posterior (o la versión 3.10.0-693 o posterior para RHEL/CentOS 7) admiten el dispositivo /dev/ptp.  En versiones anteriores del kernel que no admiten /dev/ptp para la hora del host de Azure, solo es posible la sincronización con un origen de hora externo.

Por supuesto, se puede cambiar la configuración predeterminada. Una imagen anterior configurada para usar ntpd y un origen de hora externo se puede cambiar para usar chronyd y el dispositivo /dev/ptp para la hora del host de Azure.  De forma similar, una imagen que usa la hora del host de Azure a través de un dispositivo /dev/ptp se puede configurar para usar un origen de hora NTP externo si lo requiere la aplicación o la carga de trabajo.


## <a name="tools-and-resources"></a>Herramientas y recursos

Hay algunos comandos básicos para comprobar la configuración de la sincronización horaria. En la documentación de la distribución de Linux encontrará más detalles sobre la mejor manera de configurar la sincronización horaria para esa distribución en cuestión.

### <a name="integration-services"></a>Servicios de integración

Compruebe si el servicio de integración (hv_utils) está cargado.

```bash
lsmod | grep hv_utils
```
Verá algo parecido a esto:

```
hv_utils               24418  0
hv_vmbus              397185  7 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
```

### <a name="check-for-ptp-clock-source"></a>Comprobación del origen del reloj de PTP

Con las versiones más recientes de Linux, un origen del reloj del protocolo de tiempo de precisión (PTP) está disponible como parte del proveedor VMICTimeSync.
En versiones anteriores de Red Hat Enterprise Linux o CentOS 7.x, los [servicios de integración de Linux](https://github.com/LIS/lis-next) se pueden descargar y usar para instalar el controlador actualizado. Cuando el origen del reloj de PTP esté disponible, el dispositivo Linux tendrá el formato /dev/ptp *x*. 

Consulte los orígenes de reloj del PTP que están disponibles.

```bash
ls /sys/class/ptp
```

En este ejemplo, el valor devuelto es *ptp0*, así que lo usamos para comprobar el nombre del reloj. Para comprobar el dispositivo, compruebe también el nombre del reloj.

```bash
cat /sys/class/ptp/ptp0/clock_name
```

Esto debe devolver `hyperv`, lo que significa que es el host de Azure.

En máquinas virtuales de Linux con redes aceleradas habilitadas, es posible que aparezcan varios dispositivos PTP porque el controlador mlx5 de Mellanox también crea un dispositivo /dev/ptp.
Dado que el orden de inicialización puede ser diferente cada vez que se inicia Linux, el dispositivo PTP correspondiente al host de Azure puede ser /dev/ptp0 o puede ser /dev/ptp1, lo que dificulta la configuración de chronyd con el origen de reloj correcto. Para solucionar este problema, las imágenes de Linux más recientes tienen una regla udev que crea el symlink /dev/ptp_hyperv a cualquier entrada /dev/ptp correspondiente al host de Azure. Chrony debe configurarse para usar este symlink en lugar de /dev/ptp0 o /dev/ptp1.

### <a name="chrony"></a>chrony

En Ubuntu 19.10 y versiones posteriores, Red Hat Enterprise Linux y CentOS 8.x, [chrony](https://chrony.tuxfamily.org/) se ha configurado para usar un reloj de origen del PTP. En lugar de chrony, las versiones anteriores de Linux usan el demonio del Protocolo de tiempo de red (ntpd), que no admite orígenes de PTP. Para habilitar PTP en esas versiones, chrony se debe instalar y configurar manualmente (en chrony.conf) usando la siguiente instrucción:

```bash
refclock PHC /dev/ptp0 poll 3 dpoll -2 offset 0
```
Según lo anterior, si el symlink /dev/ptp_hyperv está disponible, úselo en lugar de /dev/ptp0 para evitar confusiones con el dispositivo /dev/ptp creado por el controlador mlx5 de Mellanox.

Para más información sobre Ubuntu y NTP, vea [Sincronización de hora](https://ubuntu.com/server/docs/network-ntp).

Para más información sobre Red Hat y NTP, vea [Configurar NTP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_ntpd#s1-Configure_NTP). 

Para más información sobre chrony, vea [Usar chrony](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite#sect-Using_chrony).

Si los orígenes de chrony y VMICTimeSync están habilitados simultáneamente, puede marcar el que **prefiera**; al hacer esto, el otro origen se establecerá como secundario. Como los servicios de NTP no actualizan el reloj para grandes distorsiones, excepto después de un largo período, VMICTimeSync recuperará el reloj de los eventos de la máquina virtual en pausa mucho más rápido que las herramientas basadas en NTP.

De forma predeterminada, chronyd acelera o ralentiza el reloj del sistema para corregir cualquier desfase de tiempo. Si el desfase es demasiado grande, chrony no podrá corregirlo. Para solucionarlo, el parámetro `makestep` en **/etc/chrony.conf** se puede cambiar para forzar una sincronización de hora si el desplazamiento supera el umbral especificado.

 ```bash
makestep 1.0 -1
```

En este caso, chrony forzará una actualización de hora si el desfase es superior a 1 segundo. Reinicie el servicio chronyd para aplicar el cambio:

```bash
systemctl restart chronyd
```

### <a name="systemd"></a>systemd 

En las versiones de SUSE y Ubuntu anteriores a 19.10, la sincronización de hora se configura con [systemd](https://www.freedesktop.org/wiki/Software/systemd/). Para más información sobre Ubuntu, vea [Sincronización de hora](https://help.ubuntu.com/lts/serverguide/NTP.html). Para más información sobre SUSE, vea la sección 4.5.8 en [Notas de la versión de SUSE Linux Enterprise Server 12 SP3](https://www.suse.com/releasenotes/x86_64/SUSE-SLES/12-SP3/#InfraPackArch.ArchIndependent.SystemsManagement).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte el artículo [Hora de Windows Server 2016 precisa](/windows-server/networking/windows-time-service/accurate-time).


