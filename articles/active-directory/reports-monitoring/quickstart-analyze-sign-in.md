---
title: Análisis de los inicios de sesión con el registro de inicios de sesión de Azure AD
description: En este inicio rápido aprenderá a usar el registro de inicios de sesión para determinar el motivo del error al iniciar sesión en Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: report-monitor
ms.topic: quickstart
ms.date: 06/03/2021
ms.author: markvi
author: MarkusVi
manager: mtillman
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: beaab0517c1bd0d3aa06cc31cf6c087896f7d52b
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111442183"
---
# <a name="quickstart-analyze-sign-ins-with-the-azure-ad-sign-ins-log"></a>Inicio rápido: Análisis de los inicios de sesión con el registro de inicios de sesión de Azure AD 

Con la información del registro de inicios de sesión de Azure AD puede averiguar qué ha ocurrido si se produjo un error en el inicio de sesión de un usuario. En este inicio rápido se muestra cómo encontrar el inicio de sesión con error mediante el registro de inicios de sesión.


## <a name="prerequisites"></a>Requisitos previos

Para completar el escenario en este inicio rápido, necesita:

- **Acceso a un inquilino de Azure AD**: si no tiene acceso a uno, consulte [Cree su cuenta gratuita de Azure hoy mismo](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- **Una cuenta de prueba denominada Isabella Simonsen**: si no sabe cómo crear una cuenta de prueba, consulte [Agregar usuarios basados en la nube](../fundamentals/add-users-azure-active-directory.md#add-a-new-user).

## <a name="perform-a-failed-sign-in"></a>Creación de un inicio de sesión con errores

El objetivo de este paso es crear un registro de un inicio de sesión con error en el registro de inicios de sesión de Azure AD.

**Para este paso:**

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como Isabella Simonsen con una contraseña incorrecta.

2. Espere 5 minutos para asegurarse de que la entrada aparece en el registro de inicios de sesión. Para más información, consulte [Informes de actividad](reference-reports-latencies.md#activity-reports).



## <a name="find-the-failed-sign-in"></a>Identificación del inicio de sesión con errores

En esta sección se proporcionan los pasos para analizar un inicio de sesión con errores:

- **Filtrado de los inicios de sesión**: quite todos los registros que no sean pertinentes para el análisis. Por ejemplo, establezca un filtro para mostrar solo los registros de un usuario específico.
- **Búsqueda de información adicional sobre el error**: además de la información que puede encontrar en el registro de inicios de sesión, también puede buscar el error mediante la [herramienta de búsqueda de errores de inicio de sesión](https://login.microsoftonline.com/error). Esta herramienta puede proporcionarle información adicional sobre un error de inicio de sesión. 


**Para revisar el inicio de sesión con errores:**

1. Vaya al [registro de inicios de sesión](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns).

2. Para mostrar solo los registros de Isabella Simonsen:

    a. En la barra de herramientas, haga clic en **Agregar filtros**.
    
    ![Incorporación de un filtro de usuario](./media/quickstart-analyze-sign-in/add-filters.png)   

    b. En la lista **Elija un campo**, seleccione **Usuario** y, a continuación, haga clic **en Aplicar**.

    c. En el cuadro de texto **Nombre de usuario**, escriba **Isabella Simonsen** y haga clic en **Aplicar**.

    d. En la barra de herramientas, haga clic en **Actualizar**.

3. Para analizar el problema, haga clic en **Solución de problemas y soporte técnico**.

    ![Adición de filtro](./media/quickstart-analyze-sign-in/troubleshooting-and-support.png)   

4. Copie el **código de error de inicio de sesión**.

    ![Código de error de inicio de sesión](./media/quickstart-analyze-sign-in/sign-in-error-code.png)   


5. Pegue el código de error en el cuadro de texto de la [herramienta de búsqueda de errores de inicio de sesión](https://login.microsoftonline.com/error) y haga clic en **Enviar**.

Revise el resultado de la herramienta y determine si proporciona información adicional.

![Herramienta de búsqueda de códigos de error](./media/concept-all-sign-ins/error-code-lookup-tool.png)


## <a name="additional-tests"></a>Pruebas adicionales

Ahora que sabe cómo buscar una entrada en el registro de inicios de sesión por nombre, también debe intentar buscar el registro mediante los filtros siguientes:

- **Fecha**: intente buscar el usuario Isabella mediante **Inicio** y **Fin**.

    ![Filtro de fecha](./media/quickstart-analyze-sign-in/start-and-end-filter.png)

- **Estado**: intente encontrar el usuario Isabella mediante **Estado: Error**.

    ![Estado de error](./media/quickstart-analyze-sign-in/status-failure.png)




## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el usuario de prueba. Si no sabe cómo eliminar un usuario de Azure AD, consulte [Eliminación de usuarios desde Azure AD](../fundamentals/add-users-azure-active-directory.md#delete-a-user).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [¿Qué son los informes de Azure Active Directory?](overview-reports.md)
