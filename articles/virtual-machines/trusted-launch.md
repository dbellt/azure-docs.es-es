---
title: 'Versión preliminar: inicio seguro para máquinas virtuales de Azure'
description: Obtenga información acerca del inicio seguro para máquinas virtuales de Azure.
author: khyewei
ms.author: khwei
ms.service: virtual-machines
ms.subservice: trusted-launch
ms.topic: conceptual
ms.date: 02/26/2021
ms.reviewer: cynthn
ms.custom: template-concept; references_regions
ms.openlocfilehash: 2483a207344894489992f38d08b37126d7eded22
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110652244"
---
# <a name="trusted-launch-for-azure-virtual-machines-preview"></a>Inicio seguro para máquinas virtuales de Azure (versión preliminar)

Azure ofrece el inicio seguro como una manera continua de mejorar la seguridad de las máquinas virtuales de [generación 2](generation-2.md). El inicio seguro protege frente a técnicas de ataque persistentes y avanzadas. El inicio seguro se compone de varias tecnologías de infraestructura coordinadas que se pueden habilitar de manera independiente. Cada tecnología proporciona otro nivel de defensa contra amenazas sofisticadas.

> [!IMPORTANT]
> El inicio seguro requiere la creación de nuevas máquinas virtuales. No se puede habilitar el inicio seguro en las máquinas virtuales existentes que se crearon inicialmente sin él.
>
> El inicio seguro está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. 
>
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="benefits"></a>Ventajas 

- Implemente máquinas virtuales de forma segura con los cargadores de arranque comprobados, los kernels del sistema operativo y los controladores.
- Proteja de forma segura claves, certificados y secretos en las máquinas virtuales.
- Obtenga información y confianza de la integridad de toda la cadena de arranque.
- Asegúrese de que las cargas de trabajo sean de confianza y comprobables.

## <a name="public-preview-limitations"></a>Limitaciones de la vista preliminar pública

**Compatibilidad con tamaño**:
- Serie B
- Serie Dav4, serie Dasv4
- Serie DCsv2
- Serie Dv4, serie Dsv4, serie Dsv3, serie Dsv2
- Serie Ddv4, serie Ddsv4
- Serie Fsv2
- Serie Eav4, serie Easv4
- Serie Ev4, serie Esv4, serie Esv3
- Serie Edv4, serie Edsv4
- Serie Lsv2

**Compatibilidad con SO**:
- RedHat Enterprise Linux 8.3
- SLES 15 SP2
- Ubuntu 20.04 LTS
- Ubuntu 18.04 LTS
- Windows Server 2019
- Windows Server 2016
- Windows 10 Pro
- Windows 10 Enterprise
- Sesión múltiple de Windows 10 Enterprise

**Regiones**: 
- Centro de EE. UU.
- Este de EE. UU. 2
- Centro-sur de EE. UU.
- Norte de Europa
- Oeste de Europa

**Precios**: no hay coste adicional en los precios de las máquinas virtuales existentes.

**Las siguientes características no se admiten en esta versión preliminar**:
- Backup
- Azure Site Recovery
- Galería de imágenes compartidas
- Disco de sistema operativo efímero
- Disco compartido
- Imagen administrada
- Azure Dedicated Host 

## <a name="secure-boot"></a>Arranque seguro

La raíz del inicio seguro es el arranque seguro para la máquina virtual. Este modo, que se implementa en el firmware de la plataforma, protege frente a la instalación de rootkits y kits de arranque basados en malware. El arranque seguro funciona para asegurarse de que solo se puedan arrancar los sistemas operativos y controladores firmados. Establece una "raíz de confianza" para la pila de software en la máquina virtual. Con el arranque seguro habilitado, todos los componentes de arranque del sistema operativo (cargador de arranque, kernel y controladores de kernel) deben estar firmados por editores de confianza. Tanto Windows como las distribuciones de Linux seleccionadas admiten el arranque seguro. Si el arranque seguro no puede autenticar si la imagen está firmada por un editor de confianza, no se permitirá el arranque de la máquina virtual. Para obtener más información, consulta [Arranque seguro](/windows-hardware/design/device-experiences/oem-secure-boot).

## <a name="vtpm"></a>vTPM

El inicio seguro también presenta vTPM para las máquinas virtuales de Azure. Esta es una versión virtualizada de un [módulo de plataforma segura](/windows/security/information-protection/tpm/trusted-platform-module-overview) de hardware, compatible con la especificación TPM2.0. Sirve como almacén seguro dedicado para claves y medidas. El inicio seguro proporciona la máquina virtual con su propia instancia de TPM dedicada, que se ejecuta en un entorno seguro fuera del alcance de cualquier máquina virtual. vTPM permite la [atestación](/windows/security/information-protection/tpm/tpm-fundamentals#measured-boot-with-support-for-attestation) midiendo la cadena de arranque completa de la máquina virtual (UEFI, SO, sistema y controladores). 

El inicio seguro usa vTPM para realizar la atestación remota por parte de la nube. Se usa para las comprobaciones de estado de la plataforma y para tomar decisiones basadas en confianza. Como comprobación de estado, el inicio seguro puede certificar criptográficamente que la máquina virtual se ha arrancado correctamente. Si se produce un error en el proceso, posiblemente porque la máquina virtual ejecuta un componente no autorizado, Azure Security Center emitirá alertas de integridad. Las alertas incluyen detalles sobre los componentes que no superaron las comprobaciones de integridad.

## <a name="virtualization-based-security"></a>Seguridad de virtualización

La [seguridad basada en virtualización](/windows-hardware/design/device-experiences/oem-vbs) (SBV) utiliza el hipervisor para crear una región de memoria segura y aislada. Windows usa estas regiones para ejecutar varias soluciones de seguridad con mayor protección contra vulnerabilidades y vulnerabilidades de seguridad malintencionadas. El inicio seguro le permite habilitar la integridad de código de hipervisor (HVCI) y Credential Guard de Windows Defender.

HVCI es una mitigación del sistema eficaz que protege los procesos del modo kernel de Windows contra la inyección y la ejecución de código malintencionado o no comprobado. Comprueba los controladores y los archivos binarios del modo kernel antes de que se ejecuten, evitando que los archivos sin firmar se carguen en la memoria. Esto garantiza que el código ejecutable no se puede modificar una vez se permita su caga. Para obtener más información sobre VBS y HVCI, consulte [Seguridad basada en virtualización (SBV) e integridad de código aplicada por el hipervisor (HVCI)](https://techcommunity.microsoft.com/t5/windows-insider-program/virtualization-based-security-vbs-and-hypervisor-enforced-code/m-p/240571).

Con el inicio seguro y SBV, puede habilitar Credential Guard de Windows Defender. Esta característica aísla y protege los secretos para que solo el software del sistema con privilegios pueda acceder a ellos. Ayuda a evitar el acceso no autorizado a secretos y ataques de robo de credenciales, como ataques Pass-the-Hash (PtH). Para obtener más información, consulte [Credential Guard](/windows/security/identity-protection/credential-guard/credential-guard).


## <a name="security-center-integration"></a>Integración en Security Center

El inicio seguro se integra con Azure Security Center para asegurarse de que las máquinas virtuales estén configuradas correctamente. Azure Security Center evaluará continuamente las máquinas virtuales compatibles y emitirá recomendaciones pertinentes.

- **Recomendación para habilitar el arranque seguro**: esta recomendación solo se aplica a las máquinas virtuales que admiten el inicio seguro. Azure Security Center identificará las máquinas virtuales que pueden habilitar el arranque seguro, pero que lo han deshabilitado. Emitirá una recomendación de gravedad baja para habilitarlo.
- **Recomendación para habilitar vTPM**: si la máquina virtual tiene vTPM habilitado, Azure Security Center puede usarla para realizar la atestación de invitados e identificar patrones de amenazas avanzados. Si Azure Security Center identifica las máquinas virtuales que admiten el inicio seguro y tienen deshabilitado vTPM, emitirá una recomendación de gravedad baja para habilitarlo. 
- **Evaluación del estado de atestación**: si la máquina virtual tiene vTPM habilitado, una extensión de Azure Security Center puede validar de forma remota que la máquina virtual se ha arrancado de forma correcta. Esto se conoce como atestación remota. Azure Security Center emite una evaluación, que indica el estado de la atestación remota.

## <a name="azure-defender-integration"></a>Integración de Azure Defender

Si las máquinas virtuales están configuradas correctamente con el inicio seguro, Azure Defender puede detectarlas y enviarle una alerta de los problemas de estado de la máquina virtual.

- **Alerta de error de atestación de VM**: Azure Defender realizará periódicamente la atestación en las máquinas virtuales. Esto también ocurre después de que la máquina virtual se arranque. Si se produce un error en la atestación, se desencadena una alerta de gravedad media.
    La atestación de la máquina virtual puede producir un error por las razones siguientes:
    - La información atestada, que incluye un registro de arranque, se desvía de una línea base de confianza. Esto puede indicar que los módulos que no son de confianza se han cargado y que el sistema operativo puede estar en peligro.
    - No se pudo comprobar que la oferta de atestación se originó en el vTPM de la máquina virtual atestada. Esto puede indicar que el malware está presente y puede estar interceptando tráfico a vTPM.
    - La extensión de atestación en la máquina virtual no responde. Esto puede indicar un ataque por denegación de servicio por malware o un administrador del sistema operativo.

    > [!NOTE]
    >  Esta alerta está disponible para las máquinas virtuales con vTPM habilitado y la extensión de atestación instalada. El arranque seguro debe estar habilitado para que se supere la atestación. Se producirá un error en la atestación si el arranque seguro está deshabilitado. Si debe deshabilitar el arranque seguro, puede suprimir esta alerta para evitar falsos positivos.

- **Alerta para el módulo de kernel de Linux que no es de confianza**: para el inicio seguro con arranque seguro habilitado, es posible que una máquina virtual arranque incluso si se produce un error en la validación de un controlador de kernel y se prohíbe la carga. Si esto ocurre, Azure Defender emitirá una alerta de gravedad baja. Aunque no hay ninguna amenaza inmediata, ya que no se ha cargado el controlador que no es de confianza, se deben investigar estos eventos. Tenga en cuenta lo siguiente.
    - ¿Qué controlador de kernel produjo un error? ¿Estoy familiarizado con este controlador y espero que se cargue?
    - ¿Es esta la versión exacta del controlador que se espera? ¿Los archivos binarios del controlador están intactos? Si se trata de un controlador de terceros, ¿el proveedor pasó las pruebas de cumplimiento del sistema operativo para que se firme?


## <a name="faq"></a>Preguntas más frecuentes

Preguntas más frecuentes sobre el inicio seguro.

### <a name="why-should-i-use-trusted-launch-what-does-trusted-launch-guard-against"></a>¿Por qué debo usar el inicio seguro? ¿Para qué sirve la protección de inicio seguro?

El inicio seguro protege contra los kits de arranque, los rootkits y el malware de nivel de kernel. Estos sofisticados tipos de malware se ejecutan en modo kernel y permanecen ocultos para los usuarios. Por ejemplo:
- Rootkits de firmware: estos kits sobrescriben el firmware del BIOS de la máquina virtual, por lo que el rootkit puede iniciarse antes que el sistema operativo. 
- Kits de arranque: estos kits reemplazan el cargador de arranque del sistema operativo para que la máquina virtual cargue el kit de arranque antes que el sistema operativo.
- Rootkits de kernel: estos kits reemplazan una parte del kernel del sistema operativo para que el rootkit pueda iniciarse automáticamente cuando se carga el sistema operativo.
- Rootkits de controladores: se supone que estos kits son uno de los controladores de confianza que utiliza el sistema operativo para comunicarse con los componentes de la máquina virtual.

### <a name="what-are-the-differences-between-secure-boot-and-measured-boot"></a>¿Cuáles son las diferencias entre el arranque seguro y el arranque medido?

En la cadena de arranque seguro, cada paso del proceso de arranque comprueba una firma criptográfica de los pasos siguientes. Por ejemplo, el BIOS comprobará una firma en el cargador y el cargador comprobará las firmas de todos los objetos de kernel que cargue, etc. Si alguno de los objetos está en peligro, la firma no coincidirá y la máquina virtual no arrancará. Para obtener más información, consulta [Arranque seguro](/windows-hardware/design/device-experiences/oem-secure-boot). El arranque medido no detiene el proceso de arranque, mide o calcula el hash de los siguientes objetos de la cadena y almacena los valores hash en los registros de configuración de la plataforma (PCR) en vTPM. Los registros de arranque medidos se utilizan para la supervisión de la integridad de arranque.

### <a name="what-happens-when-an-integrity-fault-is-detected"></a>¿Qué ocurre cuando se detecta un error de integridad?

El inicio seguro para máquinas virtuales de Azure se supervisa para detectar amenazas avanzadas. Si se detectan estas amenazas, se desencadenará una alerta. Las alertas solo están disponibles en el [nivel Estándar](../security-center/security-center-pricing.md) de Azure Security Center.
Azure Security Center realiza la atestación periódicamente. Si se produce un error en la atestación, se desencadena una alerta de gravedad media. La atestación del inicio seguro puede producir un error por las razones siguientes: 
- La información atestada, que incluye un registro de la base de cálculo de confianza (TCB), se desvía de una línea base de confianza (como cuando el arranque seguro está habilitado). Esto puede indicar que los módulos que no son de confianza se han cargado y que el sistema operativo puede estar en peligro.
- No se pudo comprobar que la oferta de atestación se originó en el vTPM de la máquina virtual atestada. Esto puede indicar que el malware está presente y puede estar interceptando tráfico a vTPM. 
- La extensión de atestación en la máquina virtual no responde. Esto puede indicar un ataque por denegación de servicio por malware o un administrador del sistema operativo.

  
### <a name="how-does-trusted-launch-compared-to-hyper-v-shielded-vm"></a>¿Cómo se compara el inicio seguro con la máquina virtual blindada de Hyper-V?
Actualmente, la máquina virtual blindada de Hyper-V solo está disponible en Hyper-V. Normalmente, la [máquina virtual blindada de Hyper-V](/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-and-shielded-vms) se implementa junto con el tejido protegido. Un tejido protegido consta de un Servicio de protección de host (HGS), uno o más hosts protegidos y un conjunto de máquinas virtuales blindadas. Las máquinas virtuales blindadas de Hyper-V están destinadas a usarse en tejidos en los que los datos y el estado de la máquina virtual deben estar protegidos de los administradores del tejido y del software que no es de confianza y que podrían estar ejecutándose en los hosts de Hyper-V. Por otro lado, el inicio seguro se puede implementar como una máquina virtual independiente o conjuntos de escalado de máquinas virtuales en Azure sin necesidad de implementación y administración adicionales de HGS. Las demás características de inicio seguro se pueden habilitar con un simple cambio en el código de implementación o una casilla en Azure Portal.  

### <a name="how-can-i-convert-existing-vms-to-trusted-launch"></a>¿Cómo puedo convertir las máquinas virtuales existentes en un inicio seguro?
En el caso de las máquinas virtuales de generación 2, la ruta de acceso de migración para convertir en un inicio seguro está destinada a disponibilidad general (GA).

## <a name="next-steps"></a>Pasos siguientes

Implementación de una [máquina virtual con el inicio seguro mediante el portal](trusted-launch-portal.md).
