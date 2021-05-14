---
title: 'Configuración técnica de una oferta de Dynamics 365 for Customer Engagement & Power Apps en Microsoft AppSource: Azure Marketplace'
description: Realice la configuración técnica de una oferta de Dynamics 365 for Customer Engagement & Power Apps en Microsoft AppSource (Azure Marketplace).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.custom: references_regions
author: vamahtan
ms.author: vamahtan
ms.date: 04/29/2021
ms.openlocfilehash: 06b08ec2a28af36299f583bc9f8aa5604c347593
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319060"
---
# <a name="set-up-dynamics-365-for-customer-engagement--power-apps-offer-technical-configuration"></a>Configuración técnica de una oferta de Dynamics 365 for Customer Engagement & Power Apps

Esta página especifica los detalles técnicos usados para conectarse a la oferta. Esta conexión permite aprovisionar la oferta para los clientes que deciden adquirirla.

## <a name="offer-information"></a>Información de la oferta

El **modelo de licencia básica** determina cómo se asignan los clientes a la aplicación en el centro de administración de CRM. Realice una de las siguientes acciones:

- Seleccione **Recurso** en el caso de las licencias basadas en instancias.
- Seleccione **Usuario** si las licencias se asignan una por inquilino o si ha optado por administrar las licencias de la aplicación mediante Microsoft.

La casilla **Requiere salida de sitio a sitio y acceso al almacén seguro de CRM** permite configurar el acceso de salida del almacenamiento seguro de CRM o de servidor a servidor (S2S). Esta característica requiere una atención especial por parte del equipo de Dynamics 365 durante la fase de certificación. Microsoft se pondrá en contacto con usted para completar los pasos adicionales para admitir esta característica.

Deje **Dirección URL de configuración de aplicación** en blanco; se usará en un futuro.

## <a name="crm-package"></a>Paquete de CRM

En el cuadro **Dirección URL de la ubicación del paquete**, escriba la dirección URL de la cuenta de Azure Blob Storage que contiene el archivo .zip del paquete de CRM cargado. Incluya una clave SAS de solo lectura para permitir que Microsoft pueda recoger el paquete para su comprobación.

> [!IMPORTANT]
> Para evitar un bloque de publicación, asegúrese de que la fecha de expiración de la dirección URL del almacenamiento de blobs no haya expirado. Se puede revisar la fecha accediendo a la directiva. Se recomienda que el **Tiempo de expiración** sea al menos dentro de un mes.

Seleccione el cuadro **There is more than one CRM package in my package file** (Hay más de un paquete de CRM en el archivo de paquete), si procede. Si es así, asegúrese de incluir todos los paquetes en el archivo ZIP.

Para obtener información detallada sobre cómo compilar el paquete y actualizar su estructura, consulte [Paso 3: Crear un paquete AppSource para la aplicación](/powerapps/developer/common-data-service/create-package-app-appsource).

## <a name="crm-package-availability"></a>Disponibilidad del paquete de CRM

Seleccione **+ Agregar región** para especificar las regiones geográficas en las que el paquete de CRM estará disponible para los clientes. La implementación en las siguientes regiones soberanas requiere un permiso y una validación especiales durante el proceso de certificación: [Alemania](../germany/index.yml), la [nube del US Gov](../azure-government/documentation-government-welcome.md) y TIP.

De manera predeterminada, la **dirección URL de configuración de aplicación** que especificó anteriormente se usará para cada región. Si lo prefiere, puede especificar una dirección URL de configuración de aplicación independiente para una o varias regiones específicas.

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Venta conjunta con Microsoft**. Para obtener información sobre la configuración de la venta conjunta con Microsoft (opcional), vea [Información general sobre la venta conjunta de los equipos de ventas de Microsoft y los asociados](marketplace-co-sell.md). Si no va a configurar la venta conjunta o ha terminado, continúe con la sección **Pasos siguientes** a continuación.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de contenido complementario](dynamics-365-customer-engage-supplemental-content.md)
