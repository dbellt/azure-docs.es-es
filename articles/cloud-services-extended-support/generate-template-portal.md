---
title: Habilitación de plantilla de ARM para Cloud Services (soporte extendido) mediante Azure Portal
description: Genere y descargue la plantilla de ARM y el archivo de parámetros para Cloud Services (soporte extendido) mediante Azure Portal
ms.topic: how-to
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachandw
ms.date: 03/07/2021
ms.custom: ''
ms.openlocfilehash: a4f206d68df3cd8dd4dd5b1b411d316e7aacde92
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077126"
---
# <a name="generate-arm-template-for-cloud-services-extended-support-using-the-azure-portal"></a>Habilitación de plantilla de ARM para Cloud Services (soporte extendido) mediante Azure Portal

En este artículo se explica cómo descargar la plantilla de ARM y el archivo de parámetros desde [Azure Portal](https://portal.azure.com) para el servicio en la nube. La plantilla de ARM y el archivo de parámetros se pueden usar en implementaciones a través de PowerShell para crear o actualizar un servicio en la nube.

## <a name="get-arm-template-via-portal"></a>Obtención de una plantilla de ARM mediante el portal

  1. Vaya a Azure Portal y [cree un nuevo servicio en la nube](deploy-portal.md). Agregue los archivos de configuración del servicio en la nube, paquetes y definiciones. 
    :::image type="content" source="media/deploy-portal-4.png" alt-text="Imagen que muestra la sección de carga de la pestaña Datos básicos durante la creación":::
  
  2. Una vez completados todos los campos, vaya a la pestaña Revisar y crear para validar la configuración de implementación y haga clic en **Descargar una plantilla para la automatización** para crear la instancia de Cloud Services (soporte extendido).
    :::image type="content" source="media/download-template-portal-1.png" alt-text="Imagen que muestra la descarga de la plantilla en el servicio en la nube (soporte extendido) en Azure Portal.":::
  
  3. Descargue la plantilla y los archivos de parámetros. 
    :::image type="content" source="media/generate-template-portal-3.png" alt-text="Imagen que muestra la descarga del archivo de plantilla en Azure Portal.":::
  
  4. Copie el URI de SAS del paquete y el URI de SAS de configuración de la pestaña Revisar y crear y agréguelos al archivo parameter.json. Estos archivos ahora se pueden usar para crear un nuevo servicio en la nube a través de PowerShell.
    :::image type="content" source="media/download-template-portal-2.png" alt-text="La imagen muestra los parámetros del URI de SAS del paquete y del URI de SAS de configuración en Azure Portal.":::
  
## <a name="next-steps"></a>Pasos siguientes 
- Vea las [preguntas más frecuentes](faq.md) sobre Cloud Services (soporte extendido).
- Implementación de un servicio en la nube (soporte extendido) mediante [Azure Portal](deploy-portal.md)
  
