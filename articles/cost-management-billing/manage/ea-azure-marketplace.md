---
title: Azure Marketplace
description: Describe de qué forma pueden usar Azure Marketplace los clientes de Contrato Enterprise
author: bandersmsft
ms.reviewer: baolcsva
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 09/03/2020
ms.author: banders
ms.openlocfilehash: ce9dff017a796e420586ad191a59c149bed07190
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726833"
---
# <a name="azure-marketplace"></a>Azure Marketplace

En este artículo de qué forma los clientes y asociados de Contrato Enterprise pueden ver los cargos de Marketplace y habilitar las compras en Azure Marketplace.

## <a name="azure-marketplace-for-ea-customers"></a>Azure Marketplace para clientes de EA

Para los clientes directos, los cargos de Azure Marketplace se pueden ver en Azure Enterprise Portal. Las compras y los consumos de Azure Marketplace se facturan fuera del prepago de Azure, en plazos mensuales o trimestrales a pago vencido.

Los clientes indirectos pueden encontrar sus suscripciones a Azure Marketplace en la página **Administrar suscripciones** de Azure Enterprise Portal, pero no se muestran los precios. Los clientes deben ponerse en contacto con su Proveedor de soluciones de licencia (LSP) para obtener información sobre los cargos en Azure Marketplace.

Las nuevas compras periódicas mensuales o anuales de Azure Marketplace se facturan por completo durante el período en que se compran los elementos de Azure Marketplace. Estos artículos se renovarán automáticamente en el siguiente período el mismo día de la compra original.

Los cargos periódicos mensuales existentes se seguirán renovando el primero de cada mes calendario. Los cargos anuales se renovarán en el aniversario de la fecha de compra.

Algunos servicios de revendedores de terceros disponibles en Azure Marketplace ahora consumen el saldo del prepago de Azure del Contrato Enterprise (EA). Antes, estos servicios se facturaban por separado, fuera del prepago de Azure de EA. El prepago de Azure de EA para estos servicios de Azure Marketplace simplifica las compras y la administración de los pagos para los clientes. Para ver una lista completa de los servicios que actualmente consumen prepago de Azure, consulte la [actualización del 6 de marzo de 2018 en el sitio web de Azure](https://azure.microsoft.com/updates/azure-marketplace-third-party-reseller-services-now-use-azure-monetary-commitment/).

### <a name="partners"></a>Asociados

Los LSP pueden descargar una lista de precios de Azure Marketplace en la página de la hoja de precios en Azure Enterprise Portal. Seleccione el vínculo **Lista de precios de Marketplace** en la esquina superior derecha. La lista de precios de Azure Marketplace muestra todos los servicios disponibles y sus precios.

Para descargar la hoja de precios:

1. En Azure Enterprise Portal, vaya a **Informes** > **Hoja de precios**.
1. En la esquina superior derecha, busque el vínculo a la lista de precios de Azure Marketplace debajo de su nombre de usuario.
1. Haga clic con el botón derecho en el vínculo y seleccione **Guardar destino como**.
1. En la ventana **Guardar**, cambie el título del documento a `AzureMarketplacePricelist.zip`, que cambiará el archivo de .xlsx a .zip.
1. Después de completada la descarga, tendrá un archivo ZIP con listas de precios específicas del país o región.
1. Los LSP deben hacer referencia al archivo individual del país para obtener los precios específicos del país o región. Los LSP pueden usar la pestaña **Notificaciones** para estar al tanto de las SKU que son nuevas o retiradas.
1. Los cambios de precio son poco frecuentes. Los LSP reciben notificaciones por correo electrónico de los aumentos de precios y las variaciones en los tipos de cambio (FX) con 30 días de antelación.
1. Los LSP reciben una factura por inscripción, por ISV y por trimestre.

### <a name="enabling-azure-marketplace-purchases"></a>Habilitación de compras en Azure Marketplace

Los administradores de empresa pueden deshabilitar o habilitar las compras de Azure Marketplace para todas las suscripciones de Azure en su inscripción. Si el administrador de empresa deshabilita las compras, y hay suscripciones de Azure que ya tienen suscripciones a Azure Marketplace, esas suscripciones a Azure Marketplace no se cancelarán ni se verán afectadas.

Aunque los clientes pueden convertir sus suscripciones de Azure directas en Azure EA si las asocian a su inscripción en Azure Enterprise Portal, esta acción no convierte automáticamente las suscripciones secundarias.

Para habilitar compras en Azure Marketplace:

1. Inicie sesión en Azure Enterprise Portal como administrador de empresa.
1. Vaya a **Administrar**.
1. En **Enrollment Detail** (Detalles de la inscripción), seleccione el icono de lápiz situado junto al elemento de línea **Azure Marketplace**.
1. Cambie entre **Habilitado/Deshabilitado** o **Solo SKU BYOL o libres**, según corresponda.
1. Seleccione **Guardar**.

> [!NOTE]
> Las opciones BYOL (traiga su propia licencia) y solo gratis limitan la compra y adquisición de SKU de Azure Marketplace a solo SKU BYOL y gratuitas.

### <a name="services-billed-hourly-for-azure-ea"></a>Servicios facturados por horas para Azure EA

Los siguientes servicios se facturan por horas en un Contrato Enterprise, en lugar de la tarifa mensual de MOSP:

- Red de entrega de aplicaciones
- Firewall de aplicaciones web

### <a name="azure-remoteapp"></a>Azure RemoteApp

Si tiene un Contrato Enterprise, paga por Azure RemoteApp según el nivel de precios del Contrato Enterprise. No existen cargos adicionales. El precio estándar incluye 40 horas iniciales. El precio ilimitado cubre 80 horas iniciales. RemoteApp deja de emitir el uso por encima de las 80 horas.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información acerca de los [precios](ea-pricing-overview.md).
- En las [preguntas frecuentes sobre Azure Cost Management + Billing](../cost-management-billing-faq.yml) encontrará una lista de preguntas y respuestas sobre los servicios de Azure Marketplace y el prepago de Azure EA.