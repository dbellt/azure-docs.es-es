---
title: Habilitación de Azure Disk Encryption en máquinas virtuales Windows
description: En este artículo se proporcionan las instrucciones necesarias para habilitar Microsoft Azure Disk Encryption en las máquinas virtuales Windows.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 10/05/2019
ms.custom: seodec18
ms.openlocfilehash: e283ff2de003146c8228d36843f00ca8e4faced9
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111748578"
---
# <a name="azure-disk-encryption-for-windows-vms"></a>Azure Disk Encryption para máquinas virtuales Windows

Azure Disk Encryption ayuda a custodiar y proteger sus datos con el fin de satisfacer los compromisos de cumplimiento y seguridad de su organización. Usa la característica [BitLocker](https://en.wikipedia.org/wiki/BitLocker) de Windows para proporcionar cifrado de volumen al SO y los discos de datos de las máquinas virtuales (VM) de Azure y se integra con [Azure Key Vault](../../key-vault/index.yml) para ayudar a controlar y administrar las claves de cifrado de disco y los secretos.

Azure Disk Encryption es resistente a zona, de la misma manera que Virtual Machines. Para obtener detalles, vea [Servicios de Azure compatibles con Availability Zones](../../availability-zones/az-region.md).

Si utiliza [Azure Security Center](../../security-center/index.yml), se le alertará si tiene VM que no estén cifradas. Estas alertas se muestran con gravedad alta y se recomienda cifrar estas máquinas virtuales.

![Alerta de cifrado de discos en Azure Security Center](../media/disk-encryption/security-center-disk-encryption-fig1.png)

> [!WARNING]
> - Si ya ha usado Azure Disk Encryption con Azure AD para cifrar una VM, debe seguir usando esta opción para cifrar la VM. Para más información, consulte [Azure Disk Encryption con Azure AD (versión anterior)](disk-encryption-overview-aad.md). 
> - Algunas de las recomendaciones pueden provocar un aumento del uso de datos, de la red o de recursos de proceso, lo que incrementará los costes de las licencias o suscripciones. Para crear recursos en Azure en las regiones admitidas, debe tener una suscripción válida de Azure activa.

Para obtener información sobre los aspectos básicos de Azure Disk Encryption para Windows en unos minutos, vea [Inicio rápido: Creación y cifrado de una máquina virtual Windows con la CLI de Azure](disk-encryption-cli-quickstart.md) o [Inicio rápido: Creación y cifrado de una máquina virtual Windows en Azure con PowerShell](disk-encryption-powershell-quickstart.md).

## <a name="supported-vms-and-operating-systems"></a>VM y sistemas operativos compatibles

### <a name="supported-vms"></a>VM admitidas

Las máquinas virtuales Windows están disponibles en una [variedad de tamaños](../sizes-general.md). Azure Disk Encryption es compatible con las máquinas virtuales de Gen1 y Gen2. Azure Disk Encryption también está disponible para las VM con almacenamiento Premium.

Azure Disk Encryption no está disponible en [VM básicas o de serie A](https://azure.microsoft.com/pricing/details/virtual-machines/series/) ni en las máquinas virtuales que tengan menos de 2 GB de memoria.  Azure Disk Encryption tampoco está disponible en las imágenes de VM sin discos temporales (Dv4, Dsv4, Ev4 y Esv4).  Consulte [Tamaños de máquina virtual de Azure sin disco temporal local](../azure-vms-no-temp-disk.md).  Para ver más excepciones, consulte [Azure Disk Encryption: escenarios no admitidos](disk-encryption-windows.md#unsupported-scenarios).

### <a name="supported-operating-systems"></a>Sistemas operativos admitidos

- Cliente Windows: Windows 8 y versiones posteriores.
- Windows Server: Windows Server 2008 R2 y versiones posteriores.  
 
> [!NOTE]
> Windows Server 2008 R2 requiere la instalación de .NET Framework 4.5 para el cifrado; instálelo desde Windows Update con la actualización opcional Microsoft .NET Framework 4.5.2 para sistemas basados en Windows Server 2008 R2 x64 ([KB2901983](https://www.catalog.update.microsoft.com/Search.aspx?q=KB2901983)).  
>  
> Windows Server 2012 R2 Core y Windows Server 2016 Core requieren la instalación del componente bdehdcfg en la máquina virtual para el cifrado.


## <a name="networking-requirements"></a>Requisitos de red
Para habilitar Azure Disk Encryption, las máquinas virtuales deben cumplir los siguientes requisitos de configuración de los puntos de conexión de red:
  - Para que un token se conecte a un almacén de claves, la máquina virtual Windows debe poder conectarse a un punto de conexión de Azure Active Directory, \[login.microsoftonline.com\].
  - Para escribir las claves de cifrado en el almacén de claves, la máquina virtual Windows debe poder conectarse al punto de conexión del almacén de claves.
  - La máquina virtual Windows debe poder conectarse al punto de conexión de Azure Storage que hospeda el repositorio de extensiones de Azure y la cuenta de Azure Storage que hospeda los archivos del VHD.
  -  Si su directiva de seguridad limita el acceso desde máquinas virtuales de Azure a Internet, puede resolver el URI anterior y configurar una regla concreta para permitir la conectividad de salida para las direcciones IP. Para más información, consulte [Azure Key Vault detrás de un firewall](../../key-vault/general/access-behind-firewall.md).    


## <a name="group-policy-requirements"></a>Requisitos de la directiva de grupo

Azure Disk Encryption usa el protector de claves externas de BitLocker para las máquinas virtuales Windows. Para las máquinas virtuales unidas en un dominio, no cree ninguna directiva de grupo que exija protectores de TPM. Para obtener información acerca de la directiva de grupo para "Permitir BitLocker sin un TPM compatible", consulte la [Referencia de la directiva de grupo de BitLocker](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings#bkmk-unlockpol1).

La directiva de BitLocker en máquinas virtuales de unión a un dominio con directivas de grupo personalizadas debe incluir la siguiente configuración: [Configuración de almacenamiento de usuario de información de recuperación de BitLocker -> Permitir clave de recuperación de 256 bits](/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings). Azure Disk Encryption presentará un error cuando la configuración de la directiva de grupo personalizada para BitLocker sea incompatible. En máquinas que no tengan la configuración de directiva correcta, puede que sea necesario aplicar la nueva directiva, forzar la nueva directiva a actualizarse (gpupdate.exe /force) y luego reiniciar.

> [!WARNING]
> Azure Disk Encryption **no almacena las claves de recuperación**. Si la opción de seguridad [Inicio de sesión interactivo: umbral de bloqueo de cuenta del equipo](/windows/security/threat-protection/security-policy-settings/interactive-logon-machine-account-lockout-threshold) está habilitada, las máquinas solo se pueden recuperar si se proporciona una clave de recuperación a través de la consola serie. Puede encontrar instrucciones para asegurarse de que las directivas de recuperación adecuadas están habilitadas en el [plan de la guía de recuperación de BitLocker](/windows/security/information-protection/bitlocker/bitlocker-recovery-guide-plan).

Azure Disk Encryption producirá un error si la directiva de grupo de nivel de dominio bloquea el algoritmo AES-CBC, que BitLocker usa.

## <a name="encryption-key-storage-requirements"></a>Requisitos de almacenamiento de la clave de cifrado  

Azure Disk Encryption requiere Azure Key Vault para controlar y administrar las claves y los secretos de cifrado de discos. El almacén de claves y las máquinas virtuales deben residir en la misma región y suscripción de Azure.

Para obtener más información, consulte [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md).

## <a name="terminology"></a>Terminología
En la siguiente tabla se definen algunos de los términos comunes que se usan en la documentación de cifrado de disco de Azure:

| Terminología | Definición |
| --- | --- |
| Azure Key Vault | Key Vault es un servicio de administración de claves criptográficas basado en módulos de seguridad de hardware validados por el Estándar federal de procesamiento de información (FIPS). Estos estándares ayudan a proteger las claves criptográficas y los secretos confidenciales. Para obtener más información, consulte la documentación de [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) y [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md). |
| Azure CLI | [La CLI de Azure](/cli/azure/install-azure-cli) está optimizada para administrar recursos de Azure desde la línea de comandos.|
| BitLocker |[BitLocker](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831713(v=ws.11)) es una tecnología de cifrado de volúmenes de Windows reconocida por el sector que se usa para habilitar el cifrado de disco en VM Windows. |
| Clave de cifrado de claves (KEK) | La clave asimétrica (RSA 2048) que puede usar para proteger o encapsular el secreto. Puede proporcionar una clave protegida mediante módulos de seguridad de hardware (HSM) o una clave protegida mediante software. Para obtener más información, consulte la documentación de [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) y [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md). |
| Cmdlets de PowerShell | Para más información, consulte [Cmdlets de Azure PowerShell](/powershell/azure/). |

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Creación y cifrado de una máquina virtual Windows con la CLI de Azure](disk-encryption-cli-quickstart.md)
- [Inicio rápido: Creación y cifrado de una máquina virtual Windows en Azure con PowerShell](disk-encryption-powershell-quickstart.md)
- [Escenarios de Azure Disk Encryption en máquinas virtuales Windows](disk-encryption-windows.md)
- [Script de la CLI de requisitos previos de Azure Disk Encryption](https://github.com/ejarvi/ade-cli-getting-started) 
- [Script de PowerShell de requisitos previos de Azure Disk Encryption](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md)
