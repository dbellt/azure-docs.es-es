---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 11/12/2019
ms.author: cynthn;kareni
ms.custom: include file
ms.openlocfilehash: a9146099951aba223a7b201c1613e1ec0ba617d4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96509222"
---
**Última actualización del documento**: 12 de noviembre de 2019, 10:00 PST.

La divulgación de una [nueva clase de vulnerabilidades de CPU](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002), conocidas como ataques de canal lateral de ejecución especulativa, han generado preguntas de los clientes que buscan mayor claridad.  

Microsoft ha implementado mitigaciones en todos nuestros servicios en la nube. La infraestructura que ejecuta Azure y aísla las cargas de trabajo de los clientes entre sí está protegida. Esto significa que un atacante potencial que use la misma infraestructura no puede atacar su aplicación mediante estas vulnerabilidades.

Azure utiliza un [mantenimiento con conservación de memoria](../articles/virtual-machines/maintenance-and-updates.md?bc=%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json%252c%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json%253ftoc%253d%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#maintenance-that-doesnt-require-a-reboot) siempre que es posible para minimizar el impacto en el cliente y eliminar la necesidad de reinicios. Azure seguirá utilizando estos métodos al realizar actualizaciones del host en todo el sistema y proteger a los clientes.

Encontrará más información acerca de cómo integrar la seguridad en todos los aspectos de Azure en el sitio de [documentación de Azure Security Center](../articles/security/index.yml). 

> [!NOTE] 
> Desde que este documento se publicó por primera vez, se han divulgado varias variantes de esta clase de vulnerabilidad. Microsoft continúa realizando grandes esfuerzos por proteger a nuestros clientes y proporcionarles orientación. Esta página se actualizará a medida que se publiquen más correcciones. 
> 
> El 12 de noviembre de 2019, [Intel publicó](https://software.intel.com/security-software-guidance/insights/deep-dive-intel-transactional-synchronization-extensions-intel-tsx-asynchronous-abort) un aviso técnico sobre la vulnerabilidad anulación asincrónica de transacciones (TAA) de Intel® Extensiones de sincronización transaccional (Intel® TSX) al que se le asignó [CVE-2019-11135](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-11135). Esta vulnerabilidad afecta a los procesadores Intel® Core® y los procesadores Intel® Xeon®.  Microsoft Azure ha publicado actualizaciones del sistema operativo y está implementando nuevo microcódigo, a medida que Intel reveló lo anterior, en toda nuestra flota para proteger a nuestros clientes contra estos nuevos puntos vulnerables.   Azure está trabajando estrechamente con Intel para probar y validar el nuevo microcódigo antes de su lanzamiento oficial en la plataforma. 
>
> **Los clientes que ejecutan código que no es de confianza dentro de su máquina virtual** deben tomar medidas para protegerse frente a estos puntos vulnerables; para ello, lea a continuación para obtener instrucciones acerca de todas las vulnerabilidades de canal lateral de ejecución especulativa (ADV de Avisos de Microsoft [180002](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002), [180018](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/adv180018) y [190013](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV190013)).
>
> Otros clientes deben evaluar estos puntos vulnerables desde la perspectiva de la defensa en profundidad y considerar las implicaciones en la seguridad y el rendimiento de la configuración que han elegido.
> 



## <a name="keeping-your-operating-systems-up-to-date"></a>Mantener actualizados los sistemas operativos

Aunque no se necesita una actualización del sistema operativo para aislar de otros clientes las aplicaciones que se ejecutan en Azure, mantener el software actualizado siempre es un procedimiento recomendado. Los últimos paquetes acumulativos de actualizaciones de seguridad para Windows contienen mitigaciones para varias vulnerabilidades de canal lateral de ejecución especulativa. De forma similar, las distribuciones de Linux han publicado varias actualizaciones para corregir estas vulnerabilidades. Estas son las acciones recomendadas para actualizar el sistema operativo:

| Oferta | Acción recomendada  |
|----------|---------------------|
| Azure Cloud Services  | Habilite las [actualizaciones automáticas](../articles/cloud-services/cloud-services-how-to-configure-portal.md) o asegúrese de que está ejecutando la versión más reciente del sistema operativo invitado. |
| Máquinas virtuales Linux en Azure | Instale las actualizaciones de su proveedor de sistema operativo. Para más información, consulte [Linux](#linux) más adelante en este documento. |
| Máquinas virtuales Windows en Azure  | Instale las últimas actualizaciones acumulativas de seguridad.
| Otros servicios PaaS de Azure | No es necesaria ninguna acción para los clientes que usan estos servicios. Azure mantiene actualizadas las versiones de sistema operativo de forma automática. |

## <a name="additional-guidance-if-you-are-running-untrusted-code"></a>Instrucciones adicionales si está ejecutando código que no es de confianza 

Los clientes que permiten a los usuarios que no son de confianza ejecutar código arbitrario tal vez deseen implementar algunas características de seguridad adicionales en las máquinas virtuales o en los servicios en la nube de Azure. Estas características protegen frente a los vectores de divulgación entre procesos que describen varias vulnerabilidades de ejecución especulativa.

Escenarios de ejemplo donde se recomiendan las características de seguridad adicionales:

- Permite la ejecución de código que no es de confianza en la máquina virtual.  
    - *Por ejemplo, permite que uno de sus clientes cargue un archivo binario o un script que, a continuación, se ejecutará en la aplicación*. 
- Permite iniciar sesión en la máquina virtual con cuentas con privilegios reducidos a usuarios que no son de confianza.   
    - *Por ejemplo, permite iniciar sesión en una de las máquinas virtuales mediante escritorio remoto o SSH a un usuario con privilegios reducidos*.  
- Permite acceder a las máquinas virtuales implementadas mediante virtualización anidada a usuarios que no son de confianza.  
    - *Por ejemplo, controla el host de Hyper-V, pero asigna las máquinas virtuales a usuarios que no son de confianza*. 

Los clientes que no implementan un escenario relacionado con código que no es de confianza no necesitan habilitar estas características de seguridad adicionales. 

## <a name="enabling-additional-security"></a>Habilitación de la seguridad adicional 

Puede habilitar características de seguridad adicionales en la máquina virtual o en el servicio en la nube si ejecuta código que no sea de confianza. Al mismo tiempo, asegúrese de que el sistema operativo está actualizado para habilitar las características de seguridad dentro de la máquina virtual o el servicio en la nube

### <a name="windows"></a>Windows 

El sistema operativo de destino debe estar actualizado para habilitar estas características de seguridad adicionales. Aunque hay habilitadas numerosas mitigaciones del canal lateral de ejecución especulativa de forma predeterminada, las características adicionales que se describen aquí deben habilitarse manualmente y pueden afectar al rendimiento. 


**Paso 1: deshabilite la característica de Hyper-Threading en la máquina virtual**. Los clientes que ejecutan código que no es de confianza en una máquina virtual con Hyper-Threading deben deshabilitar esta característica o pasar a un tamaño de máquina virtual que no sea de Hyper-Threading. Remítase a [este documento](../articles/virtual-machines/acu.md) para obtener una lista de tamaños de máquina virtual de hyper-threading (donde la proporción de vCPU a Core es 2:1). Para comprobar si la máquina virtual tiene habilitada la característica de hyper-threading, consulte el siguiente script mediante la línea de comandos de Windows desde la máquina virtual.

Escriba `wmic` para especificar la interfaz interactiva. A continuación, escriba lo siguiente para ver la cantidad de procesadores físicos y lógicos en la máquina virtual.

```console
CPU Get NumberOfCores,NumberOfLogicalProcessors /Format:List
```

Si el número de procesadores lógicos es mayor que el de procesadores físicos (núcleos), la característica de hyper-threading está habilitada.  Si está ejecutando una máquina virtual de hyper-threading, [póngase en contacto con el soporte técnico de Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) para deshabilitar la característica de hyper-threading.  Una vez que el hyper-threading esté deshabilitado, **se requerirá un reinicio completo de la máquina virtual**. Consulte [Número de núcleos](#core-count) para saber por qué se reduce el número de núcleos de máquina virtual.


**Paso 2**: a la vez que el paso 1, siga las instrucciones descritas en [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) para comprobar que las protecciones se habilitan con el módulo de PowerShell [SpeculationControl](https://aka.ms/SpeculationControlPS).

> [!NOTE]
> Si descargó este módulo anteriormente, deberá instalar la versión más reciente.
>


La salida del script de PowerShell debe tener los siguientes valores para validar las protecciones habilitadas frente a estos puntos vulnerables:

```
Windows OS support for branch target injection mitigation is enabled: True
Windows OS support for kernel VA shadow is enabled: True
Windows OS support for speculative store bypass disable is enabled system-wide: False
Windows OS support for L1 terminal fault mitigation is enabled: True
Windows OS support for MDS mitigation is enabled: True
Windows OS support for TAA mitigation is enabled: True
```

Si el resultado muestra `MDS mitigation is enabled: False`, [póngase en contacto con el soporte técnico de Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) para conocer las opciones de mitigación disponibles.



**Paso 3**: para habilitar la compatibilidad del sistema operativo con la sesión concurrente de direcciones virtuales del kernel (KVAS) y la inserción de destino de rama (BTI), siga las instrucciones descritas en [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) para habilitar las protecciones con las claves del Registro de `Session Manager`. Es necesario reiniciar.


**Paso 4**: para las implementaciones que usan la [virtualización anidada](../articles/virtual-machines/windows/nested-virtualization.md) (D3 y E3 solo), estas instrucciones se aplican en la máquina virtual que se usa como host de Hyper-V.

1.  Siga las instrucciones descritas en el artículo [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) para habilitar las protecciones mediante las claves del Registro `MinVmVersionForCpuBasedMitigations`.
2.  Establezca el tipo de programador del hipervisor en `Core` mediante las instrucciones descritas [aquí](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-scheduler-types).


### <a name="linux"></a>Linux

<a name="linux"></a>Se requiere que el sistema operativo de destino esté completamente actualizado para habilitar el conjunto de características de seguridad adicionales. Algunas de las mitigaciones estarán habilitadas de forma predeterminada. La siguiente sección describe las características que están desactivadas de forma predeterminada y sujetas a la compatibilidad de hardware (microcódigo). La habilitación de estas características puede afectar al rendimiento. Consulte la documentación del proveedor de su sistema operativo para más instrucciones


**Paso 1: deshabilite la característica de Hyper-Threading en la máquina virtual**. Los clientes que ejecutan código que no es de confianza en una máquina virtual con Hyper-Threading deben deshabilitar esta característica o pasar a una máquina virtual sin Hyper-Threading.  Remítase a [este documento](../articles/virtual-machines/acu.md) para obtener una lista de tamaños de máquina virtual de hyper-threading (donde la proporción de vCPU a Core es 2:1). Para comprobar si está ejecutando una máquina virtual de hyper-threading, ejecute el comando `lscpu` en la máquina virtual de Linux. 

Si `Thread(s) per core = 2`, el hyper-threading se ha habilitado. 

Si `Thread(s) per core = 1`, el hyper-threading se ha deshabilitado. 

 
Salida de ejemplo para una máquina virtual con hyper-threading habilitado: 

```console
CPU Architecture:      x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                8
On-line CPU(s) list:   0-7
Thread(s) per core:    2
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1

```

Si está ejecutando una máquina virtual de hyper-threading, [póngase en contacto con el soporte técnico de Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) para deshabilitar la característica de hyper-threading.  Una vez que el hyper-threading esté deshabilitado, **se requerirá un reinicio completo de la máquina virtual**. Consulte [Número de núcleos](#core-count) para saber por qué se reduce el número de núcleos de máquina virtual.



**Paso 2**: para mitigar cualquiera de las vulnerabilidades siguientes de canal lateral de ejecución especulativa, consulte la documentación del proveedor de su sistema operativo:   
 
- [RedHat y CentOS](https://access.redhat.com/security/vulnerabilities) 
- [SUSE](https://www.suse.com/support/kb/?doctype%5B%5D=DT_SUSESDB_PSDB_1_1&startIndex=1&maxIndex=0) 
- [Ubuntu](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/) 


### <a name="core-count"></a>Recuento de núcleos

Cuando se crea una máquina virtual de hyper-threading, Azure asigna dos subprocesos por núcleo, que se denominan vCPU. Cuando el hyper-threading está deshabilitado, Azure quita un subproceso y usa los núcleos de subprocesos únicos (núcleos físicos). La proporción de vCPU y CPU es 2:1, de modo que, una vez que el hyper-threading se deshabilite, el número de CPU en la máquina virtual aparecerá reducido a la mitad. Por ejemplo, una máquina virtual D8_v3 es una máquina virtual de hyper-threading que se ejecuta en 8 vCPU (2 subprocesos por núcleo, por 4 núcleos).  Cuando el hyper-threading se deshabilite, las CPU caerán hasta 4 núcleos físicos con 1 subproceso por núcleo. 

## <a name="next-steps"></a>Pasos siguientes

Este artículo proporciona instrucciones para los siguientes ataques de canal lateral de ejecución especulativa que afectan a muchos de los procesadores modernos:

[Spectre Meltdown](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV180002):
- CVE-2017-5715: inserción de destino de rama (BTI)  
- CVE-2017-5754: aislamiento de tabla de páginas de kernel (KPTI)
- CVE-2018-3639: omisión especulativa de almacén (KPTI) 
- [CVE-2019-1125](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-1125): información del kernel de Windows, variante de Spectre variante 1
 
[Error de terminal L1 (L1TF)](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV180018):
- CVE-2018-3615: extensiones de protección de software de Intel (Intel SGX)
- CVE-2018-3620: sistemas operativos (SO) y modo de administración del sistema (SMM)
- CVE-2018-3646: afecta a Virtual Machine Manager (VMM)

[Muestreo de datos de microarquitectura](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190013): 
- CVE-2019-11091: Memoria sin caché de muestreo de datos de microarquitectura (MDSUM)
- CVE-2018-12126: Muestreo de datos de búfer de almacén de microarquitectura (MSBDS)
- CVE-2018-12127: Muestreo de datos de puerto de carga microarquitectura (MLPDS)
- CVE-2018-12130: Muestreo de datos de búfer de relleno de microarquitectura (MFBDS)

Anulación asincrónica de transacciones de Extensiones de sincronización transaccional (Intel® TSX):  
- [CVE-2019-11135](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-11135): asincrónica de transacciones de TSX (TAA)