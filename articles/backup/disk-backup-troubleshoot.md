---
title: Solución de problemas de errores de copia de seguridad en Azure Disk Backup
description: Obtenga información sobre cómo solucionar errores de copia de seguridad en Azure Disk Backup
ms.topic: conceptual
ms.date: 06/08/2021
ms.openlocfilehash: a749c87949d53781dc810148d01cc5d179d70f77
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111754122"
---
# <a name="troubleshooting-backup-failures-in-azure-disk-backup"></a>Solución de problemas de errores de copia de seguridad en Azure Disk Backup

En este artículo se proporciona información para la solución de problemas de copias de seguridad y de restauración de Azure Disk. Para obtener más información sobre la disponibilidad de la región de [Azure Disk Backup](disk-backup-overview.md), los escenarios admitidos y las limitaciones, consulte la [matriz de compatibilidad](disk-backup-support-matrix.md).

## <a name="common-issues-faced-with-azure-disk-backup"></a>Problemas comunes de Azure Disk Backup

### <a name="error-code-usererrorsnapshotrgsubscriptionmismatch"></a>Código de error: UserErrorSnapshotRGSubscriptionMismatch

Mensaje de error: Suscripción no válida seleccionada en el almacén de datos de instantáneas.

Acción recomendada: Los discos y las instantáneas de disco se almacenan en la misma suscripción. Puede elegir cualquier grupo de recursos para almacenar las instantáneas de disco dentro de la suscripción. Seleccione la misma suscripción que la del disco de origen. Para obtener más información, consulte la [matriz de compatibilidad](disk-backup-support-matrix.md).

### <a name="error-code-usererrorsnapshotrgnotfound"></a>Código de error: UserErrorSnapshotRGNotFound

Mensaje de error: No se pudo realizar la operación porque el grupo de recursos del almacén de datos de instantáneas no existe.

Acción recomendada: Cree el grupo de recursos y proporcione los permisos necesarios en el mismo. Para obtener más información, consulte la [configuración de copias de seguridad](backup-managed-disks.md#configure-backup).

### <a name="error-code-usererrormanageddisknotfound"></a>Código de error: UserErrorManagedDiskNotFound

Mensaje de error: No se pudo realizar la operación porque el disco administrado ya no existe.

Acción recomendada: Seguirán produciéndose errores en las copias de seguridad si el disco de origen se elimina o se mueve a otra ubicación. Use el punto de restauración existente para restaurar el disco si este se ha eliminado por error. Si el disco se mueve a una ubicación diferente, configure la copia de seguridad del mismo.

### <a name="error-code-usererrornotenoughpermissionondisk"></a>Código de error: UserErrorNotEnoughPermissionOnDisk

Mensaje de error: El servicio Azure Backup requiere permisos adicionales en el disco para realizar esta operación.

Acción recomendada: Conceda a la identidad administrada del almacén de Backup los permisos adecuados en el disco. Consulte [la documentación](backup-managed-disks.md) para comprender qué permisos son necesarios para la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrornotenoughpermissiononsnapshotrg"></a>Código de error: UserErrorNotEnoughPermissionOnSnapshotRG

Mensaje de error: El servicio Azure Backup requiere permisos adicionales en el grupo de recursos del almacén de datos de instantáneas para realizar esta operación.

Acción recomendada: Conceda a la identidad administrada del almacén de Backup los permisos adecuados en el grupo de recursos del almacén de datos de instantáneas. El grupo de recursos del almacén de datos de instantáneas es la ubicación donde se almacenan las instantáneas de disco. Consulte [la documentación](backup-managed-disks.md) para comprender qué grupo de recursos va a usar, qué permisos necesita la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrordiskbackupdiskormsipermissionsnotpresent"></a>Código de error: UserErrorDiskBackupDiskOrMSIPermissionsNotPresent

Mensaje de error: El disco no es válido o el servicio Azure Backup requiere permisos adicionales en el disco para realizar esta operación.

Acción recomendada: Seguirán produciéndose errores en las copias de seguridad si el disco de origen se elimina o se mueve a otra ubicación. Use el punto de restauración existente para restaurar el disco si este se ha eliminado por error. Si el disco se mueve a una ubicación diferente, configure la copia de seguridad del mismo. Si el disco no se elimina ni se mueve de sitio, conceda a la identidad administrada del almacén de Backup los permisos adecuados en el disco. Consulte [la documentación](backup-managed-disks.md) para comprender qué permisos son necesarios para la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrordiskbackupsnapshotrgormsipermissionsnotpresent"></a>Código de error: UserErrorDiskBackupSnapshotRGOrMSIPermissionsNotPresent

Mensaje de error: No se pudo realizar la operación porque el grupo de recursos del almacén de datos de instantáneas ya no existe. O bien, el servicio Azure Backup requiere permisos adicionales en el grupo de recursos del almacén de datos de instantáneas para realizar esta operación.

Acción recomendada: Cree un grupo de recursos y conceda a la identidad administrada del almacén de Backup los permisos adecuados en el grupo de recursos del almacén de datos de instantáneas. El grupo de recursos del almacén de datos de instantáneas es la ubicación donde se almacenan las instantáneas de disco. Consulte [la documentación](backup-managed-disks.md) para comprender qué grupo de recursos va a usar, qué permisos necesita la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrordiskbackupauthorizationfailed"></a>Código de error: UserErrorDiskBackupAuthorizationFailed

Mensaje de error: La identidad administrada del almacén de Backup no tiene los permisos necesarios para realizar esta operación.

Acción recomendada: Conceda a la identidad administrada del almacén de Backup los permisos adecuados en el disco del que se va a realizar la copia de seguridad y en el grupo de recursos del almacén de datos de instantáneas donde se almacenan las instantáneas. Consulte [la documentación](backup-managed-disks.md) para comprender qué permisos son necesarios para la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrorsnapshotrgormsipermissionsnotpresent"></a>Código de error: UserErrorSnapshotRGOrMSIPermissionsNotPresent

Mensaje de error: No se pudo realizar la operación porque el grupo de recursos del almacén de datos de instantáneas ya no existe. O bien, el servicio Azure Backup requiere permisos adicionales en el grupo de recursos del almacén de datos de instantáneas para realizar esta operación.

Acción recomendada: Cree un grupo de recursos y conceda a la identidad administrada del almacén de Backup los permisos adecuados en el grupo de recursos del almacén de datos de instantáneas. El grupo de recursos del almacén de datos de instantáneas es la ubicación donde se almacenan las instantáneas. Consulte [la documentación](backup-managed-disks.md) para comprender qué grupo de recursos va a usar, qué permisos necesita la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererroroperationalstoreparametersnotprovided"></a>Código de error: UserErrorOperationalStoreParametersNotProvided

Mensaje de error: No se pudo realizar la operación porque el parámetro del grupo de recursos del almacén de datos de instantáneas no se ha proporcionado.

Acción recomendada: Proporcione el parámetro del grupo de recursos del almacén de datos de instantáneas. El grupo de recursos del almacén de datos de instantáneas es la ubicación donde se almacenan las instantáneas de disco. Consulte [la documentación](backup-managed-disks.md) para más información.

### <a name="error-code-usererrorinvalidoperationalstoreresourcegroup"></a>Código de error: UserErrorInvalidOperationalStoreResourceGroup

Mensaje de error: El grupo de recursos del almacén de datos de instantáneas proporcionado no es válido.

Acción recomendada: Proporcione un grupo de recursos válido para el almacén de datos de instantáneas. El grupo de recursos del almacén de datos de instantáneas es la ubicación donde se almacenan las instantáneas de disco. Consulte [la documentación](backup-managed-disks.md) para más información.

### <a name="error-code-usererrordiskbackupdisktypenotsupported"></a>Código de error: UserErrorDiskBackupDiskTypeNotSupported

Mensaje de error: El tipo de disco no es compatible.

Acción recomendada: Consulte la [matriz de compatibilidad](disk-backup-support-matrix.md) para ver los escenarios que no se admiten y las limitaciones.

### <a name="error-code-usererrorsamenamediskalreadyexists"></a>Código de error: UserErrorSameNameDiskAlreadyExists

Mensaje de error: No se pudo restaurar porque ya existe un disco con el mismo nombre en el grupo de recursos de destino seleccionado.

Acción recomendada: Proporcione un nombre de disco diferente para la restauración. Para obtener más información, consulte [Restauración en Azure Managed Disks](restore-managed-disks.md).

### <a name="error-code-usererrorrestoretargetrgnotfound"></a>Código de error: UserErrorRestoreTargetRGNotFound

Mensaje de error: No se pudo realizar la operación porque el grupo de recursos de destino no existe.

Acción recomendada: Proporcione un grupo de recursos válido para realizar la restauración. Para obtener más información, consulte [Restauración en Azure Managed Disks](restore-managed-disks.md).

### <a name="error-code-usererrornotenoughpermissionontargetrg"></a>Código de error: UserErrorNotEnoughPermissionOnTargetRG

Mensaje de error: El servicio Azure Backup requiere permisos adicionales en el grupo de recursos de destino para realizar esta operación.

Acción recomendada: Conceda a la identidad administrada del almacén de Backup los permisos adecuados en el grupo de recursos de destino. El grupo de recursos de destino es la ubicación seleccionada donde se restaurará el disco. Consulte la [documentación de restauración](restore-managed-disks.md) para comprender qué permisos son necesarios para la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrorsubscriptiondiskquotalimitreached"></a>Código de error: UserErrorSubscriptionDiskQuotaLimitReached

Mensaje de error: No se pudo realizar la operación porque se ha alcanzado el límite máximo de la cuota de disco en la suscripción.

Acción recomendada: Consulte la [documentación referente a los límites del servicio y la suscripción de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md) o póngase en contacto con el Soporte técnico de Microsoft para obtener más instrucciones.

### <a name="error-code-usererrordiskbackuprestorergormsipermissionsnotpresent"></a>Código de error: UserErrorDiskBackupRestoreRGOrMSIPermissionsNotPresent

Mensaje de error: No se pudo realizar la operación porque el grupo de recursos de destino no existe. O bien, el servicio Azure Backup requiere permisos adicionales en el grupo de recursos de destino para realizar esta operación.

Acción recomendada: Proporcione un grupo de recursos válido para realizar la restauración y conceda a la identidad administrada del almacén de Backup los permisos adecuados en el grupo de recursos de destino. El grupo de recursos de destino es la ubicación seleccionada donde se restaurará el disco. Consulte la [documentación de restauración](restore-managed-disks.md) para comprender qué permisos son necesarios para la identidad administrada del almacén de Backup y cómo proporcionarla.

### <a name="error-code-usererrordeskeyvaultkeydisabled"></a>Código de error: UserErrorDESKeyVaultKeyDisabled

Mensaje de error: La clave del almacén de claves que se usó en el conjunto de cifrado de disco no está habilitada.

Acción recomendada: Habilite la clave del almacén de claves que se usa en el conjunto de cifrado de disco. Consulte las limitaciones en la [matriz de compatibilidad](disk-backup-support-matrix.md).

### <a name="error-code-usererrormsipermissionsnotpresentondes"></a>Código de error: UserErrorMSIPermissionsNotPresentOnDES

Mensaje de error: El servicio Azure Backup necesita permiso para obtener acceso al conjunto de cifrado de disco que se usa con el disco.

Acción recomendada: Proporcione acceso de lectura a la identidad administrada del almacén de Backup en el conjunto de cifrado de disco (DES).

### <a name="error-code-usererrordeskeyvaultkeynotavailable"></a>Código de error: UserErrorDESKeyVaultKeyNotAvailable

Mensaje de error: La clave del almacén de claves que se usó en el conjunto de cifrado de disco no está disponible.

Acción recomendada: Asegúrese de que la clave del almacén de claves que se usó en el conjunto de cifrado de disco no esté deshabilitada o se haya eliminado.

### <a name="error-code-usererrordisksnapshotnotfound"></a>Código de error: UserErrorDiskSnapshotNotFound

Mensaje de error: La instantánea de disco de este punto de restauración se ha eliminado.

Acción recomendada: Las instantáneas se almacenan en el grupo de recursos del almacén de datos de instantáneas de la suscripción. Es posible que la instantánea relacionada con el punto de restauración seleccionado se haya eliminado o movido del grupo de recursos. Considere la posibilidad de usar otro punto de recuperación para realizar la restauración. Asimismo, siga las directrices recomendadas para elegir el grupo de recursos de instantáneas que se menciona en la [documentación de restauración](restore-managed-disks.md).

### <a name="error-code-usererrorsnapshotmetadatanotfound"></a>Código de error: UserErrorSnapshotMetadataNotFound

Mensaje de error: Los metadatos de la instantánea de disco de este punto de restauración se han eliminado.

Acción recomendada: Considere la posibilidad de usar otro punto de recuperación para realizar la restauración. Para obtener más información, consulte la [documentación de restauración](restore-managed-disks.md).

### <a name="error-code-backupagentpluginhostvalidateprotectionerror"></a>Código de error: BackupAgentPluginHostValidateProtectionError

Mensaje de error: La característica de copia de seguridad de disco todavía no está disponible en la región del almacén de Azure Back en el que se está intentando configurar la protección.

Acción recomendada: el almacén de copia de seguridad debe estar en una región admitida. Para ver la disponibilidad de las regiones, consulte la [matriz de compatibilidad](disk-backup-support-matrix.md).

### <a name="error-code-usererrordppdatasourcealreadyhasbackupinstance"></a>Código de error: UserErrorDppDatasourceAlreadyHasBackupInstance

Mensaje de error: El disco en el que está intentando configurar la copia de seguridad ya está protegido. El disco ya está asociado a una instancia de copia de seguridad en un almacén de Azure Back.

Acción recomendada: Este disco ya está asociado a una instancia de copia de seguridad en un almacén de Azure Back. Si desea volver a proteger este disco, elimine la instancia de copia de seguridad del almacén de Azure Back en el que está protegido actualmente y vuelva a proteger el disco en cualquier otro almacén.

### <a name="error-code-usererrordppdatasourcealreadyprotected"></a>Código de error: UserErrorDppDatasourceAlreadyProtected

Mensaje de error: El disco en el que está intentando configurar la copia de seguridad ya está protegido. El disco ya está asociado a una instancia de copia de seguridad en un almacén de Azure Back.

Acción recomendada: Este disco ya está asociado a una instancia de copia de seguridad en un almacén de Azure Back. Si desea volver a proteger este disco, elimine la instancia de copia de seguridad del almacén de Azure Back en el que está protegido actualmente y vuelva a proteger el disco en cualquier otro almacén.

### <a name="error-code-usererrormaxconcurrentoperationlimitreached"></a>Código de error: UserErrorMaxConcurrentOperationLimitReached

Mensaje de error: No se puede iniciar la operación porque se ha alcanzado el número máximo de copias de seguridad simultáneas permitidas.

Acción recomendada: Espere hasta que se complete la copia de seguridad en ejecución anterior.

### <a name="error-code-usererrormissingsubscriptionregistration"></a>Código de error: UserErrorMissingSubscriptionRegistration

Mensaje de error: la suscripción no está registrada para usar el espacio de nombres "Microsoft.Compute".

Acción recomendada: el proveedor de recursos necesario no se ha registrado para la suscripción. Registre ambos espacios de nombres (_Microsoft.Compute_ y _Microsoft.Storage_) mediante los pasos descritos en la [solución 3](../azure-resource-manager/templates/error-register-resource-provider.md#solution-3---azure-portal).

## <a name="next-steps"></a>Pasos siguientes

[Matriz de compatibilidad de Azure Disk Backup](disk-backup-support-matrix.md)