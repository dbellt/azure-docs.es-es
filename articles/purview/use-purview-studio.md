---
title: Uso de Azure Purview Studio
description: En este artículo se describe cómo utilizar Azure Purview Studio.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 04/02/2021
ms.openlocfilehash: ba22c322d47d8738b1d607597d6f93b8b8456616
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106283876"
---
# <a name="use-purview-studio"></a>Uso de Purview Studio

En este artículo se proporciona información general sobre algunas de las características principales de Azure Purview.

## <a name="prerequisites"></a>Requisitos previos

* Ya se ha creado una cuenta de Purview activa en Azure Portal y el usuario tiene permisos de acceso a Purview Studio.

## <a name="launch-purview-account"></a>Inicio de la cuenta de Purview

* Para iniciar su cuenta de Purview, vaya a Purview accounts (Cuentas de Purview) en Azure Portal, seleccione la cuenta que desea iniciar e iníciela.

  :::image type="content" source="./media/use-purview-studio/launch-from-portal.png" alt-text="Captura de pantalla de la opción seleccionada para iniciar el catálogo de la cuenta de Azure Purview.":::

* Otra manera de iniciar la cuenta de Purview es ir a `https://web.purview.azure.com`, seleccionar **Azure Active Directory** y un nombre de cuenta para iniciarla.

## <a name="home-page"></a>Página de inicio

**Página principal** es la página de inicio del cliente de Azure Purview.

:::image type="content" source="./media/use-purview-studio/purview-homepage.png" alt-text="Captura de pantalla de la página principal.":::

En la lista siguiente se resumen las principales características de la **página principal**. Cada número de la lista corresponde a un número resaltado en la captura de pantalla anterior.

1. Nombre descriptivo del catálogo. Puede establecer el nombre del catálogo en **Management Center** > **Account information** (Centro de administración > Información de la cuenta).

2. El análisis de catálogo muestra el número de:

   * Usuarios, grupos y aplicaciones
   * Orígenes de datos
   * Recursos
   * Términos del glosario

3. El cuadro de búsqueda permite buscar recursos en el catálogo de datos.

4. Los botones de acceso rápido proporcionan acceso a las funciones de la aplicación que se usan con frecuencia. Los botones que se presentan dependen del rol asignado a su cuenta de usuario.

   * Para el *administrador de datos*, los botones son **Knowledge Center** (Centro de conocimientos), **Browse Assets** (Examinar los activos), **Manage Glossary** (Administrar glosario) y **View Insights** (Ver información).
   * Para el *lector de datos*, los botones destacados son **Knowledge Center**, (Centro de conocimientos), **Browse Assets** (Examinar los activos), **View Glossary** (Ver glosario) y **View Insights** (Ver información).
   * Para *Administrador de orígenes de datos* + *Administrador de datos*, los botones destacados son **Knowledge Center** (Centro de conocimientos), **Register Data Sources** (Registrar orígenes de datos), **Browse Assets** (Examinar los recursos) y **Manage Glossary** (Administrar glosario).
   * Para *Administrador de orígenes de datos* + *Lector de datos*, los botones destacados son **Knowledge Center** (Centro de conocimientos), **Register Data Sources** (Registrar orígenes de datos), **Browse Assets** (Examinar los recursos) y **View Glossary** (Ver glosario).

5. La barra de navegación izquierda ayuda a localizar las páginas principales de la aplicación. Los botones que se presentan dependen del rol asignado a su cuenta de usuario.

   * Para el *administrador de datos*, los botones son **Home** (Página principal), **Glossary** (Glosario), **Insights** (Información) y **Management center** (Centro de administración).
   * Para el *lector de datos*, los botones son **Home** (Página principal), **Glossary** (Glosario), **Insights** (Información) y **Management center** (Centro de administración).
   * Para el *administrador de orígenes de datos* o el *lector o administrador de datos*, los botones son **Home** (Inicio), **Sources** (Orígenes), **Glossary** (Glosario), **Insights** (Información) y **Management Center** (Centro de administración).
  
6. En la pestaña **Recently accessed** (Accedido recientemente) se muestra una lista de los recursos de datos a los que se ha accedido recientemente. Para información sobre el acceso a los recursos, consulte [Búsqueda en el catálogo de datos](how-to-search-catalog.md) y [Examen por tipo de recurso](how-to-browse-catalog.md#browse-experience).  La pestaña **My items** (Mis elementos) es una lista de los recursos de datos que pertenecen al usuario que ha iniciado sesión.
7. **Useful Links** (Vínculos útiles) contiene vínculos al estado de la región, la documentación, los precios, información general y el estado de Purview.
8. La barra de navegación superior contiene información sobre las secciones siguientes: notas de la versión, actualizaciones, cambio de cuenta de Purview, notificaciones, ayuda y comentarios.

## <a name="knowledge-center"></a>Centro de conocimientos

El Centro de conocimientos es donde puede encontrar todos los vídeos y tutoriales relacionados con Purview.

## <a name="guided-tours"></a>Visitas guiadas

Los UX de Azure Purview Studio tendrán visitas guiadas para ver la información general de la página. Para iniciar la visita guiada seleccione **Help** (Ayuda) en la barra superior y seleccione **Guided tour** (Recorrido guiado).

:::image type="content" source="./media/use-purview-studio/guided-tour.png" alt-text="Captura de pantalla de la visita guiada.":::

> [!Important]
> El rol de *administrador de orígenes de datos* no tiene acceso a Purview Studio.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Incorporación de una entidad de seguridad](tutorial-scan-data.md)
