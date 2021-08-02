---
title: Establecimiento de la configuración técnica de una oferta de Dynamics 365 for Operations en Microsoft AppSource (Azure Marketplace)
description: Establezca la configuración técnica de una oferta de Dynamics 365 for Operations en Microsoft AppSource (Azure Marketplace).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.date: 05/20/2021
ms.openlocfilehash: bd2a371d702ff7f99cd26408cd40329b8c3d7859
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111971342"
---
# <a name="set-up-dynamics-365-for-operations-offer-technical-configuration"></a>Establecimiento de la configuración técnica de una oferta de Dynamics 365 for Operations

Esta página especifica los detalles técnicos usados para conectarse a la oferta. Esta conexión nos permite aprovisionar su oferta para el cliente final, en caso de que este elija adquirirla.

## <a name="solution-identifier"></a>Identificador de la solución

Proporcione el identificador de la solución (GUID) para la solución. Para encontrar el identificador de la solución:

1. En Microsoft Dynamics Lifecycle Services (LCS), seleccione **Administración de soluciones**.
2. Seleccione la solución y, a continuación, busque el **identificador de la solución** en la **información general del paquete**. Si el identificador está en blanco, seleccione **Editar**, vuelva a publicar el paquete e inténtelo de nuevo.

## <a name="release-version"></a>Versión de lanzamiento

Seleccione la versión de Dynamics 365 for Operations con la que funciona esta solución.

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Venta conjunta con Microsoft**. Para información sobre cómo configurar la venta conjunta con Microsoft (opcional), consulte [Participación de los asociados en la venta conjunta](./co-sell-overview.md). Si no va a configurar la venta conjunta o ha terminado, continúe con la sección **Pasos siguientes** a continuación.

## <a name="test-drive-technical-configuration"></a>Configuración técnica de la versión de prueba

Esta página permite configurar una demostración («versión de prueba») que permite a los clientes probar la oferta antes de adquirirla. Más información en [Qué es la versión de prueba](what-is-test-drive.md).

Para habilitar una versión de prueba, seleccione la pestaña **Habilitar una versión de prueba** y marque la casilla [Configuración de la oferta](dynamics-365-operations-offer-setup.md#test-drive). Para eliminar la versión de prueba de la oferta, desactive esta casilla.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de contenido complementario](dynamics-365-operations-supplemental-content.md)