---
title: 'Incorporación de un certificado de entidad de certificación personalizado: Azure API Management | Microsoft Docs'
description: Aprenda a agregar un certificado de entidad de certificación personalizado a Azure API Management. También puede ver instrucciones para eliminar un certificado.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/20/2018
ms.author: apimpm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 51719f23302bfaa036f99e88fcd440d97ca3bc02
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107817835"
---
# <a name="how-to-add-a-custom-ca-certificate-in-azure-api-management"></a>Incorporación de un certificado de entidad de certificación personalizado a Azure API Management

Azure API Management permite instalar los certificados de entidad de certificación en la máquina dentro de los almacenes de certificados intermedio y raíz de confianza. Esta funcionalidad debe usarse si los servicios requieren un certificado de entidad de certificación personalizado.

En este artículo se muestra cómo administrar certificados de entidad de certificación de una instancia del servicio Azure API Management en Azure Portal.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="upload-a-ca-certificate"></a><a name="step1"> </a>Carga de un certificado de entidad de certificación

![Incorporación de certificados de entidad de certificación](media/api-management-howto-ca-certificates/00.png)

Siga los pasos que se describen a continuación para cargar un nuevo certificado de entidad de certificación. Si todavía no ha creado una instancia del servicio API Management, consulte el tutorial [Creación de una instancia del servicio API Management](get-started-create-service-instance.md).

1. Vaya a la instancia del servicio Azure API Management en Azure Portal.

2. Seleccione **Certificados de entidad de certificado de entidad de certificación** en el menú.

3. Haga clic en el botón **+ Agregar**.  

    ![Captura de pantalla que muestra el botón + Agregar para agregar un certificado de CA.](media/api-management-howto-ca-certificates/01.png)  

4. Busque el certificado y decida cuál es el almacén de certificados. Solo se necesita la clave pública, no la contraseña.

    ![Captura de pantalla que muestra cómo buscar el certificado.](media/api-management-howto-ca-certificates/02.png)  

5. Haga clic en **Save**(Guardar). Esta operación puede tardar algunos minutos.

    ![Captura de pantalla que muestra cómo guardar el certificado.](media/api-management-howto-ca-certificates/03.png)  

> [!NOTE]
> Puede cargar un certificado de entidad de certificación mediante el comando de Powershell `New-AzApiManagementSystemCertificate`.

## <a name="delete-a-client-certificate"></a><a name="step1a"> </a>Eliminar un certificado de cliente

Para eliminar un certificado, haga clic en el menú contextual **...** y seleccione **Eliminar** junto a este.

![Eliminación de certificados de entidad de certificación](media/api-management-howto-ca-certificates/04.png)  

[Upload a CA certificate]: #step1
[Delete a CA certificate]: #step1a
