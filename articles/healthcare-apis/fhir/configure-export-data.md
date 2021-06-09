---
title: Definición de la configuración de exportación en Azure API for FHIR
description: En este artículo se describe cómo definir la configuración de exportación en Azure API for FHIR
author: CaitlinV39
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 5/11/2021
ms.author: cavoeg
ms.openlocfilehash: b245fcaecde13c4a751926e9b7bd639a43b36701
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111809053"
---
# <a name="configure-export-setting-and-set-up-the-storage-account"></a>Definición de la configuración de la exportación y de la configuración de la cuenta de almacenamiento

Azure API for FHIR admite el comando $export, que le permite exportar los datos de la cuenta de Azure API for FHIR a una cuenta de almacenamiento.

Para configurar la exportación en Azure API for FHIR es necesario realizar tres pasos:

1. Habilite Managed Identity en Azure API for FHIR Service.
2. Creación de una cuenta de almacenamiento de Azure (si no se ha hecho antes) y asignación de permisos Azure API for FHIR a la cuenta de almacenamiento.
3. Al seleccionar la cuenta de almacenamiento en Azure API for FHIR cuenta de almacenamiento de exportación.

## <a name="enabling-managed-identity-on-azure-api-for-fhir"></a>Habilitación de la identidad administrada en Azure API for FHIR

El primer paso para configurar Azure API for FHIR para la exportación consiste en habilitar la identidad administrada en todo el sistema en el servicio. Para más información sobre las identidades administradas en Azure, consulte [Acerca de las identidades administradas para recursos de Azure.](../../active-directory/managed-identities-azure-resources/overview.md)

Para ello, vaya al servicio de Azure API for FHIR y seleccione **Identidad.** Al cambiar el estado a **On** se habilitará la identidad administrada en Azure API for FHIR Service.

![Habilitación de una entidad administrada](media/export-data/fhir-mi-enabled.png)

Ahora, puede pasar al paso siguiente creando una cuenta de almacenamiento y asignando permiso a nuestro servicio.

## <a name="adding-permission-to-storage-account"></a>Concesión de permisos a la cuenta de almacenamiento

El siguiente paso en la exportación de datos consiste en asignar permiso Azure API for FHIR servicio para escribir en la cuenta de almacenamiento.

Después de crear una cuenta de almacenamiento, vaya a la Access Control **(IAM)** de la cuenta de almacenamiento y, a continuación, **seleccione Agregar asignación de roles.** 

Para más información sobre la asignación de roles en Azure Portal, consulte [Roles integrados de Azure.](../../role-based-access-control/role-assignments-portal.md)

Aquí es donde agregará el rol Colaborador de datos de [Storage Blob a](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor) nuestro nombre de servicio y, a continuación, seleccione **Guardar.**

![Página Agregar asignación de roles](../../../includes/role-based-access-control/media/add-role-assignment-page.png)

Ahora está listo para seleccionar la cuenta de almacenamiento en Azure API for FHIR como una cuenta de almacenamiento predeterminada para $export.

## <a name="selecting-the-storage-account-for-export"></a>Selección de la cuenta de almacenamiento para el comando $export

El último paso consiste en asignar la cuenta de almacenamiento de Azure a la Azure API for FHIR que se usará para exportar los datos. Para ello, vaya a **Integración** en Azure API for FHIR servicio y seleccione la cuenta de almacenamiento.

![Almacenamiento de las exportaciones de FHIR](media/export-data/fhir-export-storage.png)

Una vez completado este último paso, ya está listo para exportar los datos mediante $export comando.

> [!Note]
> Solo las cuentas de almacenamiento de la misma suscripción que para Azure API for FHIR pueden registrarse como destino para $export operaciones.

Para obtener más información sobre cómo configurar la base de datos, el control de acceso, habilitar el registro de diagnóstico y usar encabezados personalizados para agregar datos a los registros de auditoría, consulte:

>[!div class="nextstepaction"]
>[Configuración adicional](azure-api-for-fhir-additional-settings.md)
