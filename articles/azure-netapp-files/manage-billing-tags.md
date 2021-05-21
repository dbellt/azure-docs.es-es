---
title: Administración de la facturación de Azure NetApp Files mediante etiquetas | Microsoft Docs
description: Se describe cómo administrar la facturación de Azure NetApp Files mediante etiquetas.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/06/2021
ms.author: b-juche
ms.openlocfilehash: b0efd8c580b0d4f1ae94d4dfc655bb351e084a67
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109480911"
---
# <a name="manage-billing-by-using-capacity-pool-billing-tags"></a>Administración de la facturación mediante etiquetas de facturación del grupo de capacidad

Las etiquetas son pares de nombre y valor que permiten clasificar los recursos y ver la facturación consolidada. Puede aplicar la misma etiqueta a varios recursos y grupo de recursos.  Consulte [Uso de etiquetas para organizar los recursos de Azure y la jerarquía de administración](../azure-resource-manager/management/tag-resources.md) para más información sobre las etiquetas.  

El uso de etiquetas le ayuda a administrar la facturación y los gastos de Azure NetApp Files. Por ejemplo, su empresa podría tener solo una suscripción de Azure, pero varios departamentos que usan recursos de Azure e incurren en gastos. Puede etiquetar los recursos con los nombres de los departamentos en el nivel de grupo de capacidad. Las etiquetas correspondientes se mostrarán en la factura para ayudarle a ver los gastos en los que incurre cada departamento.   

Las etiquetas de facturación se asignan en el nivel de grupo de capacidad, no en el nivel de volumen.

## <a name="steps"></a>Pasos

1. Para agregar o editar una etiqueta en un grupo de capacidad, vaya al **grupo de capacidad** y seleccione **Etiquetas**.   

2. Rellene el par **Nombre** y **Valor**.  Haga clic en **Aplicar**.

    > [!IMPORTANT] 
    > Los datos de etiqueta se replican globalmente. Por lo tanto, no use nombres de etiqueta ni valores que puedan poner en peligro la seguridad de los recursos. Por ejemplo, no use nombres de etiqueta que contengan información personal o confidencial. 

      ![Instantánea que muestra la ventana Etiquetas de un grupo de capacidad.](../media/azure-netapp-files/billing-tags-capacity-pool.png)

3. Puede mostrar y descargar información sobre los recursos etiquetados mediante el portal de [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md): 
    1. Haga clic en **Análisis de costos** y seleccione la vista **Costo por recurso**.    
      [ ![Captura de pantalla que muestra el análisis de costos de Azure Cost Management](../media/azure-netapp-files/cost-analysis.png) ](../media/azure-netapp-files/cost-analysis.png#lightbox)  

    2. Para descargar una factura, seleccione **Facturas** y, a continuación, el botón **Descargar**.   
      [ ![Captura de pantalla que muestra las facturas de Azure Cost Management](../media/azure-netapp-files/azure-cost-invoices.png) ](../media/azure-netapp-files/azure-cost-invoices.png#lightbox)  

    1. En la ventana Descargar que aparece, descargue los detalles de uso. El archivo `csv` descargado incluirá etiquetas de facturación del grupo de capacidad para los recursos correspondientes.   
       ![Instantánea que muestra la ventana Descargar de Azure Cost Management.](../media/azure-netapp-files/invoice-download.png)   

       [ ![Captura de pantalla que muestra la hoja de cálculo descargada.](../media/azure-netapp-files/spreadsheet-download.png) ](../media/azure-netapp-files/spreadsheet-download.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

[Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md) 