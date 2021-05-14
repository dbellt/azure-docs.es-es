---
title: Copia de seguridad y restauración de Active Directory
description: Aprenda a realizar copias de seguridad y a restaurar controladores de dominio de Active Directory.
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: 8bc6458895965d4c37667e0cff3051a4e4e8288e
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107898214"
---
# <a name="back-up-and-restore-active-directory-domain-controllers"></a>Copia de seguridad y restauración de controladores de dominio de Active Directory

La realización de copias de seguridad de Active Directory y la garantía de unas restauraciones correctas en casos de daños, riesgos o desastres es una parte fundamental del mantenimiento de Active Directory.

En este artículo se describen los procedimientos adecuados para realizar copias de seguridad y restaurar controladores de dominio de Active Directory con Azure Backup, tanto si son máquinas virtuales de Azure como servidores locales. Describe un escenario en el que necesita restaurar un controlador de dominio completo al estado que tenía en el momento de la copia de seguridad. Para ver qué escenario de restauración le resulta más adecuado, consulte [este artículo](/windows-server/identity/ad-ds/manage/ad-forest-recovery-determine-how-to-recover).  

>[!NOTE]
> En este artículo no se explica cómo restaurar los elementos de [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md). Para más información sobre cómo restaurar usuarios de Azure Active Directory, consulte [este artículo](../active-directory/fundamentals/active-directory-users-restore.md).

## <a name="best-practices"></a>Procedimientos recomendados

- Asegúrese de que se haya realizado una copia de seguridad de al menos un controlador de dominio. Si realiza una copia de seguridad de más de un controlador de dominio, asegúrese de que se haya realizado una copia de seguridad de todos los que tengan roles [FSMO (operación de maestro único flexible)](/windows-server/identity/ad-ds/plan/planning-operations-master-role-placement).

- Realice copias de seguridad de Active Directory con frecuencia. La fecha de la copia de seguridad nunca debe ser anterior a la duración del marcador de exclusión (TSL), ya que los objetos anteriores a TSL se "extinguirán" y ya no se consideran válidos.
  - El TSL predeterminado, para los dominios creados en Windows Server 2003 SP2 y versiones posteriores, es de 180 días.
  - Puede comprobar el TSL configurado mediante el siguiente script de PowerShell:

    ```powershell
    (Get-ADObject $('CN=Directory Service,CN=Windows NT,CN=Services,{0}' -f (Get-ADRootDSE).configurationNamingContext) -Properties tombstoneLifetime).tombstoneLifetime
    ```

- Tenga un plan de recuperación ante desastres claro que incluya instrucciones sobre cómo restaurar los controladores de dominio. Si desea prepararse para restaurar un bosque de Active Directory, lea la [Guía de recuperación de bosques de Active Directory](/windows-server/identity/ad-ds/manage/ad-forest-recovery-guide).

- Si necesita restaurar un controlador de dominio y le queda un controlador de dominio en funcionamiento en el dominio, puede crear un nuevo servidor en lugar de restaurar desde la copia de seguridad. Agregue el rol de servidor de **Active Directory Domain Services** al nuevo servidor para convertirlo en un controlador de dominio en el dominio existente. A continuación, los datos del Active Directory se replicarán en el nuevo servidor. Para quitar el controlador de dominio anterior de Active Directory, siga los pasos de [este artículo](/windows-server/identity/ad-ds/deploy/ad-ds-metadata-cleanup) para realizar la limpieza de los metadatos.

>[!NOTE]
>Azure Backup no incluye la restauración de nivel de elemento para Active Directory. Si desea restaurar los objetos eliminados y puede acceder a un controlador de dominio, utilice la [Papelera de reciclaje de Active Directory](/windows-server/identity/ad-ds/get-started/adac/introduction-to-active-directory-administrative-center-enhancements--level-100-#ad_recycle_bin_mgmt). Si ese método no está disponible, puede utilizar la copia de seguridad del controlador de dominio para restaurar los objetos eliminados con la herramienta **ntdsutil.exe**, tal como se explica [aquí](https://support.microsoft.com/help/840001/how-to-restore-deleted-user-accounts-and-their-group-memberships-in-ac).
>
>Para más información sobre cómo realizar una restauración autoritativa de SYSVOL, consulte [este artículo](/windows-server/identity/ad-ds/manage/ad-forest-recovery-authoritative-recovery-sysvol).

## <a name="backing-up-azure-vm-domain-controllers"></a>Copia de seguridad de controladores de dominio de máquinas virtuales de Azure

Si el controlador de dominio es una máquina virtual de Azure, puede realizar una copia de seguridad del servidor mediante [Copia de seguridad de la máquina virtual de Azure](backup-azure-vms-introduction.md).

Lea acerca de las [consideraciones operativas para los controladores de dominio virtualizados](/windows-server/identity/ad-ds/get-started/virtual-dc/virtualized-domain-controllers-hyper-v#operational-considerations-for-virtualized-domain-controllers) para garantizar unas copias de seguridad (y futuras restauraciones) correctas de los controladores de dominio de máquinas virtuales de Azure.

## <a name="backing-up-on-premises-domain-controllers"></a>Copia de seguridad de controladores de dominio locales

Para realizar una copia de seguridad de un controlador de dominio local, debe realizar una copia de seguridad de los datos de estado del sistema del servidor.

- Si utiliza MARS, siga [estas instrucciones](backup-azure-system-state.md).
- Si utiliza MABS (Azure Backup Server), siga [estas instrucciones](backup-mabs-system-state-and-bmr.md).

>[!NOTE]
> No se admite la restauración de controladores de dominio locales (ya sea desde el estado del sistema o desde las máquinas virtuales) a la nube de Azure. Si desea tener la opción de conmutación por error de un entorno de Active Directory local a Azure, considere la posibilidad de utilizar [Azure Site Recovery](../site-recovery/site-recovery-active-directory.md).

## <a name="restoring-active-directory"></a>Restauración de Active Directory

Los datos de Active Directory se pueden restaurar de una de las dos maneras siguientes: **autoritativa** o **no autoritativa**. En una restauración autoritativa, los datos de Active Directory restaurados invalidarán los datos encontrados en los otros controladores de dominio del bosque.

Sin embargo, en este escenario se va a recompilar un controlador de dominio en un dominio existente, por lo que se debe realizar una restauración **no autoritativa**.

Durante la restauración, el servidor se iniciará en modo de restauración de servicios de directorio (DSRM). Deberá proporcionar la contraseña de administrador para el modo de restauración de servicios de directorio.

>[!NOTE]
>Si ha olvidado la contraseña de DSRM, puede restablecerla mediante [estas instrucciones](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754363(v=ws.11)).

### <a name="restoring-azure-vm-domain-controllers"></a>Restauración de controladores de dominio de máquinas virtuales de Azure

Para restaurar un controlador de dominio de una máquina virtual de Azure, consulte el artículo sobre [restauración de máquinas virtuales de controlador de dominio](backup-azure-arm-restore-vms.md#restore-domain-controller-vms).

Si va a restaurar una única máquina virtual de controlador de dominio o varias máquinas virtuales de controlador de dominio en un solo dominio, restáurelas como cualquier otra máquina virtual. El modo de restauración de servicios de directorio (DSRM) también está disponible, de modo que todos los escenarios de recuperación de Active Directory son viables.

Si necesita restaurar una única máquina virtual de controlador de dominio en una configuración de varios dominios, restaure los discos y cree una máquina virtual [mediante PowerShell](backup-azure-vms-automation.md#restore-the-disks).

Si está restaurando el último controlador de dominio que queda en el dominio o restaurando varios dominios en un único bosque, se recomienda una [recuperación del bosque](/windows-server/identity/ad-ds/manage/ad-forest-recovery-single-domain-in-multidomain-recovery).

>[!NOTE]
> Los controladores de dominio virtualizados, desde Windows 2012 y versiones posteriores, utilizan [medidas de seguridad basadas en la virtualización](/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100#virtualization-based-safeguards). Con estas medidas de seguridad, Active Directory entiende si la máquina virtual restaurada es un controlador de dominio y realiza los pasos necesarios para restaurar los datos de Active Directory.

### <a name="restoring-on-premises-domain-controllers"></a>Restauración de controladores de dominio locales

Para restaurar un controlador de dominio local, siga las instrucciones para restaurar el estado del sistema a Windows Server, mediante la guía de [consideraciones especiales para la recuperación del estado del sistema en un controlador de dominio](backup-azure-restore-system-state.md#special-considerations-for-system-state-recovery-on-a-domain-controller).

## <a name="next-steps"></a>Pasos siguientes

- [Matriz de compatibilidad para Azure Backup](backup-support-matrix.md)
