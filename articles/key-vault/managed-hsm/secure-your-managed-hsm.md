---
title: 'Protección del acceso a un HSM administrado: Azure Key Vault Managed HSM'
description: Aprenda a proteger el acceso a los dispositivos HSM administrados mediante Azure RBAC y RBAC local de HSM administrado.
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 07602bfefbbd38538973c799e2306d21b7a840f1
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111411996"
---
# <a name="secure-access-to-your-managed-hsms"></a>Protección del acceso a los HSM administrados

Azure Key Vault Managed HSM es un servicio en la nube que protege las claves de cifrado. Dado que estos datos son confidenciales y críticos para la empresa, debe proteger el acceso a los HSM administrados de modo que solo se admita el acceso de las aplicaciones y los usuarios autorizados. En este artículo se proporciona información general sobre el modelo de control de acceso del HSM administrado. Se explican la autenticación y la autorización, y se describe cómo proteger el acceso a los HSM administrados.

Este tutorial le lleva por un ejemplo sencillo que muestra cómo lograr la separación de tareas y el control de acceso mediante Azure RBAC y RBAC local de HSM administrado. Consulte [Control de acceso de Managed HSM](access-control.md) para obtener información sobre el modelo de control de acceso de Managed HSM.

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este artículo, debe tener los siguientes elementos:

* Una suscripción a Microsoft Azure. Si no tiene una, puede registrarse para una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial).
* La CLI de Azure, versión 2.12.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).
* Un HSM administrado en la suscripción. Consulte [Quickstart: Aprovisionamiento y activación de un HSM administrado mediante la CLI de Azure](quick-create-cli.md) para aprovisionar y activar un HSM administrado.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para iniciar sesión en Azure mediante la CLI puede escribir:

```azurecli
az login
```

Para más información sobre las opciones de inicio de sesión mediante la CLI, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

## <a name="example"></a>Ejemplo

En este ejemplo, se desarrolla una aplicación que utiliza una clave RSA de 2048 bits para las operaciones de firma. La aplicación se ejecuta en una máquina virtual de Azure con una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md). La clave RSA que se usa para firmar se almacena en el HSM administrado.

Hemos identificado los siguientes roles que administran, implementan y auditan la aplicación:

- **Equipo de seguridad**: el personal de TI de la oficina del CSO (director de seguridad) o colaboradores equivalentes. El equipo de seguridad es responsable de la protección adecuada de las claves. Claves RSA o EC para la firma y claves RSA o AES para el cifrado de datos.
- **Desarrolladores y operadores**: personal que desarrolla la aplicación y la implementa en Azure. Los miembros de este equipo no forman parte del personal de seguridad. No deben tener acceso a información confidencial, como las claves RSA. Solo la aplicación que implementan debe tener acceso a esta información confidencial.
- **Auditores**: este rol es para los colaboradores que no sean miembros del personal de TI general o de desarrollo. Deben revisar el uso y mantenimiento de los certificados, las claves y los secretos para garantizar el cumplimiento de los estándares de seguridad.

Hay otro rol que está fuera del ámbito de la aplicación: el administrador de la suscripción o del grupo de recursos. El administrador de la suscripción configura los permisos de acceso iniciales del equipo de seguridad. Conceden acceso al equipo de seguridad mediante el uso de un grupo de recursos que tiene los recursos requeridos por la aplicación.

Es necesario autorizar las siguientes operaciones para los roles:

**Equipo de seguridad**
- Crear el HSM administrado.
- Descargar el dominio de seguridad del HSM administrado (para la recuperación ante desastres)
- Active el registro.
- Generar o importar las claves.
- Crear las copias de seguridad del HSM administrado para la recuperación ante desastres.
- Establecer RBAC local del HSM administrado para conceder permisos a usuarios y aplicaciones para operaciones específicas.
- Rotar las claves periódicamente.

**Desarrolladores y operadores**
- Obtener la referencia (identificador URI de la clave) de la clave RSA que se usa para firmar del equipo de seguridad.
- Desarrollar e implementar la aplicación que accede a la clave mediante programación.

**Auditores**
- Revisar las fechas de expiración de las claves para asegurarse de que las claves están actualizadas.
- Supervisar las asignaciones de roles para asegurarse de que solo los usuarios o las aplicaciones autorizados pueden acceder a las claves.
- Revisar los registros del HSM administrado para confirmar el uso adecuado de las claves en cumplimiento de los estándares de seguridad de datos.

En la tabla siguiente se resumen las asignaciones de roles a equipos y recursos para acceder al HSM administrado.

| Role | Rol del plano de administración | Rol del plano de datos |
| --- | --- | --- |
| Equipo de seguridad | Managed HSM Contributor | Managed HSM Administrator |
| Desarrolladores y operadores | None | None |
| Auditores | None | Managed HSM Crypto Auditor |
| Identidad administrada de la máquina virtual utilizada por la aplicación| None | Managed HSM Crypto User |
| Identidad administrada de la cuenta de almacenamiento utilizada por la aplicación| None| Managed HSM Service Encryption |

Los tres roles de equipo necesitan acceder a otros recursos junto con los permisos del HSM administrado. Para implementar las máquinas virtuales o la característica Web Apps de Azure App Service, los desarrolladores y operadores también necesitan acceso `Contributor` a dichos tipos de recursos. Los auditores necesitan acceso de lectura a la cuenta de almacenamiento donde se almacenan los registros del HSM administrado.

Para asignar los roles del plano de administración (RBAC de Azure), puede usar Azure Portal o cualquiera de las otras interfaces de administración, como la CLI de Azure o Azure PowerShell. Para asignar los roles del plano de datos del HSM administrado, debe usar la CLI de Azure.

Los fragmentos de código de la CLI de Azure de esta sección se crean con los siguientes supuestos:

- El administrador de Azure Active Directory ha creado grupos de seguridad para representar los tres roles: Contoso Security Team, Contoso App DevOps y Contoso App Auditors. El administrador ha agregado usuarios a sus respectivos grupos.
- Todos los recursos se encuentran en el grupo de recursos **ContosoAppRG**.
- Los registros del HSM administrado se almacenan en la cuenta de almacenamiento **contosologstorage**.
- El HSM administrado **ContosoMHSM** y la cuenta de almacenamiento **contosologstorage** están en la misma ubicación de Azure.

El administrador de la suscripción asigna el rol `Managed HSM Contributor` al equipo de seguridad. Este rol permite al equipo de seguridad administrar los HSM administrados existentes y crear otros nuevos. Si hay HSM administrados existentes, se les debe asignar el rol "Administrador de Managed HSM" para poder administrarlos.

```azurecli-interactive
# This role assignment allows Contoso Security Team to create new Managed HSMs
az role assignment create --assignee-object-id $(az ad group show -g 'Contoso Security Team' --query 'objectId' -o tsv) --assignee-principal-type Group --role "Managed HSM Contributor"

# This role assignment allows Contoso Security Team to become administrator of existing managed HSM
az keyvault role assignment create  --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso Security Team' --query 'objectId' -o tsv) --scope / --role "Managed HSM Administrator"
```

El equipo de seguridad configura el registro y asigna roles a los auditores y a la aplicación de la máquina virtual.

```azurecli-interactive
# Enable logging
hsmresource=$(az keyvault show --hsm-name ContosoMHSM --query id -o tsv)
storageresource=$(az storage account show --name contosologstorage --query id -o tsv)
az monitor diagnostic-settings create --name MHSM-Diagnostics --resource $hsmresource --logs    '[{"category": "AuditEvent","enabled": true}]' --storage-account $storageresource

# Assign the "Crypto Auditor" role to Contoso App Auditors group. It only allows them to read.
az keyvault role assignment create  --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso App Auditors' --query 'objectId' -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Grant the "Crypto User" role to the VM's managed identity. It allows to create and use keys. 
# However it cannot permanently delete (purge) keys
az keyvault role assignment create  --hsm-name ContosoMHSM --assignee $(az vm identity show --name "vmname" --resource-group "ContosoAppRG" --query objectId -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Assign "Managed HSM Crypto Service Encryption User" role to the Storage account ID
storage_account_principal=$(az storage account show --id $storageresource --query identity.principalId -o tsv)
# (if no identity exists), then assign a new one
[ "$storage_account_principal" ] || storage_account_principal=$(az storage account update --assign-identity --id $storageresource)

az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Service Encryption User" --assignee $storage_account_principal
```

En este tutorial solo se muestran las acciones relacionadas con el control de acceso de la mayor parte. Otras acciones y operaciones relacionadas con la implementación de la aplicación en la máquina virtual, la activación del cifrado con la clave administrada por el cliente para una cuenta de almacenamiento o la creación de HSM administrados no se muestran aquí para mantener el ejemplo centrado en el control de acceso y la administración de roles.

En el ejemplo se describe un escenario sencillo. Los escenarios reales pueden ser más complejos. Puede ajustar los permisos para el almacén de claves según sus necesidades. Se supone que el equipo de seguridad proporciona las referencias de clave y secreto (identificadores URI y huellas digitales), que el personal de DevOps utiliza en sus aplicaciones. Los desarrolladores y operadores no requieren ningún acceso al plano de datos. Nos centramos en cómo proteger el almacén de claves. Debe darse una consideración parecida a la protección de [las máquinas virtuales](https://azure.microsoft.com/services/virtual-machines/security/), [las cuentas de almacenamiento](../../storage/blobs/security-recommendations.md) y otros recursos de Azure.

## <a name="resources"></a>Recursos

- [Documentación de Azure RBAC](../../role-based-access-control/overview.md)
- [RBAC de Azure: roles integrados](../../role-based-access-control/built-in-roles.md)
- [Administración de RBAC de Azure con la CLI de Azure](../../role-based-access-control/role-assignments-cli.md)

## <a name="next-steps"></a>Pasos siguientes

Para seguir un tutorial de introducción para un administrador, consulte [¿Qué es Managed HSM?](overview.md).

Para más información sobre el registro de uso del registro de Managed HSM, consulte [Registro de Managed HSM](logging.md).
