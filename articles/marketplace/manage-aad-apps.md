---
title: 'Incorporación y administración de aplicaciones de Azure AD: Azure Marketplace'
description: Aprenda a agregar y administrar aplicaciones de Azure AD para el programa de Marketplace comercial en el Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 04/06/2021
author: varsha-sarah
ms.author: vavargh
ms.custom: contperf-fy21q2
ms.openlocfilehash: 77f6b2ec71df63da1fe5ff52f948ded9502c5edb
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110076283"
---
# <a name="add-and-manage-azure-ad-applications"></a>Agregar y administrar aplicaciones de Azure AD

**Roles adecuados**

- Propietario
- Manager

Puede permitir que las aplicaciones o los servicios que forman parte de la instancia de Azure Active Directory (Azure AD) de la empresa accedan a su cuenta del Centro de partners.

## <a name="add-existing-azure-ad-applications"></a>Agregar aplicaciones de Azure AD existentes

Para agregar aplicaciones que ya existen en la instancia de Azure Active Directory de la empresa:

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar aplicaciones de Azure AD**.
1. Seleccione una o varias aplicaciones de Azure AD en la lista que aparece. Puede usar el cuadro de búsqueda para buscar aplicaciones de Azure AD concretas. Si selecciona más de una aplicación de Azure AD para agregar a la cuenta del Centro de partners, debe asignarles el mismo rol o conjunto de permisos personalizados. Para agregar varias aplicaciones de Azure AD con roles y permisos diferentes, repita estos pasos para cada rol o conjunto de permisos personalizados.
1. Cuando termine de seleccionar las aplicaciones de Azure AD, seleccione **Agregar selección**.
1. En la sección **Roles**, especifique los roles o permisos personalizados de las aplicaciones de Azure AD seleccionadas.
1. Seleccione **Guardar**.

## <a name="add-new-azure-ad-applications"></a>Agregar nuevas aplicaciones de Azure AD

Si desea conceder acceso al Centro de partners a una nueva cuenta de aplicación de Azure AD, puede crear una en la sección **Usuarios**. Esto creará una nueva cuenta en la cuenta profesional de la empresa (inquilino de Azure AD), no solo en la cuenta del Centro de partners. Si se utiliza principalmente esta aplicación de Azure AD para la autenticación en el Centro de partners y no necesita usuarios para acceder a ella directamente, puede especificar cualquier dirección válida como **URL de respuesta** y como **URI de id. de aplicación**, siempre y cuando ninguna otra aplicación de Azure AD use esos valores en el directorio.

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione **Agregar aplicaciones de Azure AD**.
1. En la siguiente página, seleccione **New Azure AD application** (Nueva aplicación de Azure AD).
1. Especifique la **URL de respuesta** de la nueva aplicación de Azure AD. Esta es la dirección URL en la que los usuarios pueden iniciar sesión y usar la aplicación de Azure AD (también se denomina a veces como URL de la aplicación o URL de inicio de sesión). La *URL de respuesta* no puede tener más de 256 caracteres y debe ser única dentro del directorio.
1. Especifique el **URI de id. de aplicación** de la nueva aplicación de Azure AD. Se trata de un identificador lógico para la aplicación de Azure AD que se presenta cuando se envía una solicitud de inicio de sesión único a Azure AD. El *URI de id. de aplicación* debe ser exclusivo para cada aplicación de Azure AD del directorio. No puede tener más de 256 caracteres. Para más información sobre él, consulte [Integración de aplicaciones con Azure Active Directory](/azure/active-directory/develop/howto-modify-supported-accounts#change-the-application-registration-to-support-different-accounts).
1. En la sección **Roles**, especifique los roles o permisos personalizados de la aplicación de Azure AD.
1. Seleccione **Guardar**.

Después de agregar o crear una aplicación de Azure AD, puede volver a la sección **Usuarios** y seleccionar el nombre de la aplicación para revisar la configuración de la aplicación, incluido el identificador del inquilino y del cliente, la URL de respuesta y el URI de identificador de aplicación.

## <a name="remove-an-azure-ad-application"></a>Eliminación de una aplicación de Azure AD

Para quitar una aplicación de la cuenta profesional (inquilino de Azure AD), vaya a **Usuarios** (en **Configuración de la cuenta**), seleccione la aplicación que desee eliminar mediante la casilla de verificación de la columna situada más a la derecha y, a continuación, elija **Quitar** en el listado de acciones disponibles. Aparecerá una ventana emergente para que confirme que desea eliminar las aplicaciones seleccionadas.

## <a name="manage-keys-for-an-azure-ad-application"></a>Administración de claves de una aplicación de Azure AD

Si la aplicación de Azure AD lee y escribe datos en Microsoft Azure AD, necesitará una clave. Puede crear claves para una aplicación de Azure AD mediante la edición de su información en el Centro de partners. También puede eliminar las claves que ya no son necesarias.

1. En la página **Usuarios** (en **Configuración de la cuenta**), seleccione el nombre de la aplicación de Azure AD. Verá todas las claves activas de la aplicación de Azure AD, incluida la fecha de creación de la clave y cuándo expirará.
1. Para eliminar una clave que ya no es necesaria, seleccione **Eliminar**.
1. Para agregar una nueva clave, seleccione **Agregar nueva clave**.
1. Verá una pantalla que muestra el **id. de cliente** y los **valores de clave**. Asegúrese de imprimir o copiar esta información, ya que no podrá acceder a ella nuevamente después de salir de esta página.
1. Si desea crear más claves, seleccione **Add another key** (Agregar otra clave).
