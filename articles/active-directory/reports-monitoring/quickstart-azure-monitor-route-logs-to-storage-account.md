---
title: 'Tutorial: Archivo de registros de Azure Active Directory en una cuenta de Storage | Microsoft Docs'
description: Aprenda a configurar Azure Diagnostics para insertar los registros de Azure Active Directory en una cuenta de almacenamiento.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 045f94b3-6f12-407a-8e9c-ed13ae7b43a3
ms.service: active-directory
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: cd278968eeb430bf57f279bb8b7e5d4ee9248ce1
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111965722"
---
# <a name="tutorial-archive-azure-ad-logs-to-an-azure-storage-account"></a>Tutorial: Archivo de los registros de Azure AD en una cuenta de Azure Storage

En este tutorial, aprenderá a configurar los valores de diagnóstico de Azure Monitor para enrutar los registros de Azure Active Directory en una cuenta de Azure Storage.

## <a name="prerequisites"></a>Requisitos previos 

Para usar esta característica, necesita:

* Una suscripción de Azure con una cuenta de Azure Storage. Si no tiene ninguna suscripción de Azure, puede [registrarse para obtener una evaluación gratuita](https://azure.microsoft.com/free/).
* Un inquilino de Azure AD.
* Un usuario que sea *administrador global* o *administrador de seguridad* para el inquilino de Azure AD.

## <a name="archive-logs-to-an-azure-storage-account"></a>Archivado de registros en una cuenta de Azure Storage

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

2. Seleccione **Azure Active Directory** > **Actividad** > **Registros de auditoría**. 

3. Seleccione **Exportar configuración**. 

4. En el panel **Configuración de diagnóstico**, realice una de las siguientes acciones:
   * Para cambiar la configuración existente, seleccione **Editar configuración**.
   * Para agregar la nueva configuración, seleccione **Agregar configuración de diagnóstico**.  
     Puede tener un máximo de tres configuraciones. 

     ![Exportar configuración](./media/quickstart-azure-monitor-route-logs-to-storage-account/ExportSettings.png)

5. Escriba un nombre descriptivo para la configuración que le recuerde su propósito (por ejemplo, *Enviar a la cuenta de Azure Storage*). 

6. Active la casilla **Archivar en una cuenta de almacenamiento** y, a continuación, seleccione **Cuenta de almacenamiento**. 

7. Seleccione la suscripción de Azure y la cuenta de almacenamiento a la que desee enrutar los registros.
 
8. Seleccione **Aceptar** para salir de la configuración.

9. Realice alguna de las siguientes acciones o ambas:
    * Para enviar los registros de auditoría a la cuenta de almacenamiento, seleccione la casilla **AuditLogs**. 
    * Para enviar los registros de inicio de sesión a la cuenta de almacenamiento, seleccione la casilla **SignInLogs**.

10. Use el control deslizante para establecer la retención de los datos del registro. De forma predeterminada, este valor es *0*, lo que significa que los registros se conservarán en la cuenta de almacenamiento de forma indefinida. Si establece un valor diferente, los eventos que tengan más días de los seleccionados se eliminarán de forma automática.

11. Seleccione **Guardar** para guardar la configuración.

    ![Configuración de diagnóstico](./media/quickstart-azure-monitor-route-logs-to-storage-account/DiagnosticSettings.png)

12. A los 15 minutos, compruebe que los registros se han insertado en la cuenta de almacenamiento. Vaya a [Azure Portal](https://portal.azure.com), seleccione **Cuentas de almacenamiento**, seleccione la cuenta de almacenamiento que usó anteriormente y seleccione **Blobs**. En **Registros de auditoría**, seleccione **insights-log-audit**. En **Registros de inicio de sesión**, seleccione **insights-log-audit**.

    ![Cuenta de almacenamiento](./media/quickstart-azure-monitor-route-logs-to-storage-account/StorageAccount.png)

## <a name="next-steps"></a>Pasos siguientes

* [Interpretación del esquema de registros de auditoría en Azure Monitor](./overview-reports.md)
* [Interpretación del esquema de registros de inicio de sesión en Azure Monitor](reference-azure-monitor-sign-ins-log-schema.md)
* [Preguntas frecuentes y problemas conocidos](concept-activity-logs-azure-monitor.md#frequently-asked-questions)