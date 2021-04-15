---
title: 'Programa Azure Certified Device: Tutorial sobre la creación de un proyecto'
description: Guía para crear un proyecto en el portal de Azure Certified Device
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: tutorial
ms.date: 03/01/2021
ms.custom: template-tutorial
ms.openlocfilehash: 11d72847209a2e706d4aa32d38af1b2c8af3dfa0
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107314004"
---
# <a name="tutorial-create-your-project"></a>Tutorial: Creación de un proyecto

Enhorabuena por decidir certificar el dispositivo mediante el programa Azure Certified Device. Ya ha seleccionado el programa de certificación adecuado para el dispositivo y está listo para empezar a trabajar en el portal.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Iniciar sesión en el [portal de Azure Certified Device](https://certify.azure.com/)
> * Crear un nuevo proyecto de certificación para su dispositivo
> * Especificar los detalles básicos del dispositivo asociado al proyecto

## <a name="prerequisites"></a>Requisitos previos

- Necesitará una [cuenta de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis) profesional o educativa que sea válida.
- Necesitará una cuenta comprobada de Microsoft Partner Network (MPN). Si no tiene una cuenta de MPN, [únase a la red de asociados](https://partner.microsoft.com/) antes de empezar.

## <a name="signing-into-the-azure-certified-device-portal"></a>Inicio de sesión en el portal de Azure Certified Device

Para empezar, inicie sesión en el portal, en el que debe proporcionar la información del dispositivo, completar las pruebas de certificación y administrar las publicaciones del dispositivo en el catálogo de Azure Certified Device.

1. Vaya al [portal de Azure Certified Device](https://certify.azure.com).
1. Seleccione `Company profile` en el lado izquierdo y actualice la información del fabricante.
   ![Sección del perfil de empresa](./media/images/company-profile.png)
1. Acepte el contrato del programa para comenzar el proyecto.

## <a name="creating-your-project-on-the-portal"></a>Creación del proyecto en el portal

Ahora que está todo configurado en el portal, puede comenzar el proceso de certificación. Primero, debe crear un proyecto para el dispositivo.

1. En la pantalla de Inicio, seleccione `Create new project`. Se abrirá una ventana para agregar información básica sobre el dispositivo en la sección siguiente.

 ![Imagen del botón para crear un nuevo proyecto](./media/images/create-new-project.png)

## <a name="identifying-basic-device-information"></a>Identificación de la información básica del dispositivo

A continuación, proporcione la información básica del dispositivo. Puede editar esta información más adelante.

1. Complete los campos obligatorios en la sección `Basics`. Consulte la tabla siguiente para identificar los campos **obligatorios**:

    | Campos                  | Descripción                                                                                                                         |
    |------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
    | Nombre de proyecto           | Nombre interno que no será visible en el catálogo de Azure Certified Device                                                        |
    | Nombre de dispositivo            | Nombre público del dispositivo                                                                                                |
    | Tipo de dispositivo            | Especificación del producto terminado o del kit de desarrolladores preparado para la solución.     Para obtener más información sobre la terminología, consulte el [glosario de certificación](./resources-glossary.md).                                                                     |
    | Clase de dispositivo           | Puerta de enlace, sensor u otro dispositivo.  Para obtener más información sobre la terminología, consulte el [glosario de certificación](./resources-glossary.md).                                                                    |
    | URL del código fuente del dispositivo | Obligatorio si va a certificar un kit de desarrollo preparado para la solución. En caso contrario, es opcional. La URL debe dirigir a una ubicación de GitHub para el código del dispositivo. |
1. Seleccione el botón `Next` para continuar en la pestaña `Certifications`.

    ![Imagen del formulario para crear un nuevo proyecto en la pestaña de certificaciones](./media/images/create-new-project-certificationswindow.png)

1. Especifique qué certificaciones desea obtener para el dispositivo.
1. Seleccione `Create` y el nuevo proyecto se guardará y mostrará en la página principal del portal.

    ![Imagen de la tabla de proyectos](./media/images/project-table.png)

1. Seleccione en el nombre del proyecto en la tabla. De este modo, se abrirá la página de resumen del proyecto, en donde podrá agregar y ver otros detalles del dispositivo.

    ![Imagen de la página de detalles del proyecto](./media/images/device-details-section.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora puede agregar detalles del dispositivo y probar el dispositivo con nuestro servicio de certificación. Vaya al siguiente artículo para obtener información sobre cómo editar los detalles del dispositivo.
> [!div class="nextstepaction"]
> [Tutorial: Incorporación de detalles del dispositivo](tutorial-02-adding-device-details.md)
