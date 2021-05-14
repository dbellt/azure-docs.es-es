---
title: Deshabilitación de la identidad administrada de la cuenta de Azure Automation (versión preliminar)
description: En este artículo se explica cómo deshabilitar y quitar una identidad administrada para una cuenta de Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 04/14/2021
ms.topic: conceptual
ms.openlocfilehash: e17e1afda50d9a0263067a77bf26435f53b4f237
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107519279"
---
# <a name="disable-your-azure-automation-account-managed-identity-preview"></a>Deshabilitación de la identidad administrada de la cuenta de Azure Automation (versión preliminar)

Hay dos maneras de deshabilitar una identidad asignada por el sistema en Azure Automation. Puede completar esta tarea desde Azure Portal o con una plantilla de Resource Manager (ARM).

## <a name="disable-managed-identity-in-the-azure-portal"></a>Deshabilitar la identidad administrada en Azure Portal

Puede deshabilitar la identidad administrada desde Azure Portal independientemente de cómo la haya configurado originalmente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya a la cuenta de Automation y seleccione **Identidad** en **Configuración de la cuenta**.

1. Establezca la opción **Asignado por el sistema** en **Desactivado** y presione **Guardar**. Cuando se le solicite confirmación, presione **Sí**.

La identidad administrada se quitará y ya no tendrá acceso al recurso de destino.

## <a name="disable-using-azure-resource-manager-template"></a>Deshabilitación mediante una plantilla de Resource Manager

Si creó la identidad administrada para la cuenta de Automation mediante una plantilla de Resource Manager, puede deshabilitar la identidad administrada si vuelve a usar esa plantilla y modifica su configuración. Establezca el tipo de la propiedad secundaria del objeto de identidad en **None** como se muestra en el ejemplo siguiente y, a continuación, vuelva a ejecutar la plantilla.

```json
"identity": { 
   "type": "None" 
} 
```

Al quitar una identidad asignada por el sistema con este método, también se elimina de Azure AD. Las identidades asignadas por el sistema también se quitan automáticamente de Azure AD cuando se elimina el recurso de la aplicación al que están asignadas.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de cómo habilitar la identidad administrada en Azure Automation, consulte [Habilitación y uso de la identidad administrada para Automation (versión preliminar)](enable-managed-identity-for-automation.md).

- Para información general acerca de la seguridad de la cuenta de Automation, consulte [Introducción a la autenticación de cuentas de Automation](automation-security-overview.md).
