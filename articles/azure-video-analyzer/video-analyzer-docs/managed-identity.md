---
title: Identidades administradas con Azure Video Analyzer
description: En este tema se explica cómo usar identidades administradas con Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: how-to
ms.date: 05/04/2021
ms.openlocfilehash: 8d9558319168e2a94c81f764c9b19f70dc904f79
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386799"
---
# <a name="managed-identity"></a>Identidad administrada

Un desafío común para los desarrolladores es la administración de secretos y credenciales para proteger la comunicación entre distintos servicios. En Azure, las identidades administradas eliminan la necesidad de que los desarrolladores administren credenciales, al proporcionar una identidad para el recurso de Azure en Azure Active Directory (Azure AD) y usarla para obtener tokens de Azure AD.

Al crear una cuenta de Azure Video Analyzer, tiene que asociar una cuenta de Azure Storage a ella. Si usa Video Analyzer para grabar vídeo en directo desde una cámara, los datos se almacenan como blobs en un contenedor de la cuenta de almacenamiento. Debe usar una identidad administrada para conceder a la cuenta de Video Analyzer el acceso adecuado a la cuenta de almacenamiento, tal como se muestra a continuación.


## <a name="user-assigned-managed-identity-for-video-analyzer"></a>Identidad administrada asignada por el usuario para Video Analyzer

1. Cree una [identidad administrada asignada por el usuario](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

1. Creación de una cuenta de Azure Storage

   [!INCLUDE [the video analyzer account and storage account must be in the same subscription and region](./includes/note-account-storage-same-subscription.md)]

1. Agregue los roles [Colaborador de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor) y [Lector](../../role-based-access-control/built-in-roles.md#reader) de la cuenta de almacenamiento anterior a la identidad administrada.

1. Cree la cuenta de Video Analyzer; para ello, proporcione la identidad administrada asignada por el usuario y la cuenta de almacenamiento como valores para las propiedades pertinentes.

De este modo, Video Analyzer puede acceder a la cuenta de almacenamiento en su nombre mediante la identidad administrada.

Consulte [este](create-video-analyzer-account.md) artículo para obtener un ejemplo del uso de Azure Portal para realizar lo anterior.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las funciones de las identidades administradas y su uso con las aplicaciones de Azure, consulte [Identidades administradas de Azure AD](../../active-directory/managed-identities-azure-resources/overview.md).
