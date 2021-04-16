---
title: 'Requisitos previos de la API: Azure Marketplace'
description: Requisitos previos para usar las API de Cloud Partner Portal.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 09/23/2020
ms.openlocfilehash: f5fc77b65f6a83f4f7ca8ed8b8c9294b95307735
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107319"
---
# <a name="api-prerequisites"></a>Requisitos previos de la API

> [!NOTE]
> Las API de Cloud Partner Portal se integran en el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en la [referencia de API de Cloud Partner Portal](cloud-partner-portal-api-overview.md) para asegurarse de que el código siga funcionando después de la transición al Centro de partners. Use las API de CPP solo para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Necesita dos recursos de programación para usar las API de Cloud Partner Portal: una entidad de servicio y un token de acceso de Azure Active Directory (Azure AD).

## <a name="create-service-principal-in-azure-active-directory-tenant"></a>Creación de una entidad de servicio en el inquilino de Azure Active Directory

En primer lugar, deberá crear una entidad de servicio en el inquilino de Azure AD. A este inquilino se le asignará su propio conjunto de permisos de Cloud Partner Portal. Su código llamará a las API mediante este inquilino en lugar de usar sus credenciales personales. Para obtener una explicación completa sobre la creación de una entidad de servicio, consulte [Procedimientos: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio que puedan acceder a los recursos](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="add-service-principal-to-your-account"></a>Adición de la entidad de servicio a su cuenta

Ahora que ha creado la entidad de servicio en el inquilino, puede agregarla como un usuario a su cuenta del portal del Centro de partners. Al igual que un usuario, la entidad de servicio puede ser un propietario o un colaborador en el portal. Para más información, vea **Pasos siguientes** a continuación.

## <a name="next-steps"></a>Pasos siguientes

Vea [Administración de aplicaciones de Azure AD](manage-aad-apps.md).
