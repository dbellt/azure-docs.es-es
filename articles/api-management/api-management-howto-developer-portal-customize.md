---
title: 'Tutorial: acceso y personalización del portal para desarrolladores: Azure API Management | Microsoft Docs'
description: Siga este tutorial para aprender a personalizar el portal para desarrolladores de API Management, un sitio web totalmente personalizable generado automáticamente con la documentación de las API.
services: api-management
author: mikebudzynski
ms.service: api-management
ms.topic: tutorial
ms.date: 11/16/2020
ms.author: apimpm
ms.openlocfilehash: 7c341dee3106530715248355da4412b97ed30980
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739627"
---
# <a name="tutorial-access-and-customize-the-developer-portal"></a>Tutorial: Acceso y personalización del portal para desarrolladores

El *portal para desarrolladores* es un sitio web totalmente personalizable que se genera automáticamente con la documentación de las API. Aquí, los consumidores de API pueden detectar las API, aprender a usarlas y solicitar acceso.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * acceder a la versión administrada del portal para desarrolladores;
> * navegar por su interfaz administrativa;
> * personalizar el contenido;
> * publicar los cambios;
> * ver el portal publicado.

Puede encontrar más detalles del portal para desarrolladores en [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md).

:::image type="content" source="media/api-management-howto-developer-portal-customize/cover.png" alt-text="Modo de administrador del portal para desarrolladores de API Management" border="false":::

## <a name="prerequisites"></a>Requisitos previos

- Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)
- Importe y publique una instancia de Azure API Management. Para obtener más información, consulte [Importación y publicación](import-and-publish.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="access-the-portal-as-an-administrator"></a>Acceso al portal como administrador

Siga los pasos siguientes para acceder a la versión administrada del portal.

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. Seleccione el botón **Portal para desarrolladores** en la barra de navegación superior. Se abrirá una nueva pestaña del explorador con una versión administrativa del portal.


## <a name="developer-portal-architectural-concepts"></a>Conceptos arquitectónicos del portal para desarrolladores

Los componentes del portal se pueden dividir lógicamente en dos categorías: *código* y *contenido*.

### <a name="code"></a>Código

El código se mantiene en el [repositorio GitHub](https://github.com/Azure/api-management-developer-portal) del portal para desarrolladores de API Management e incluye:

- **Widgets**: representan elementos visuales y combinan HTML, JavaScript, capacidad de estilo, configuración y asignación de contenido. Por ejemplo, una imagen, un párrafo de texto, un formulario o una lista de API.
- **Definiciones de estilos**: especifican cómo se pueden aplicar estilos a los widgets.
- **Motor**: que genera páginas web estáticas a partir del contenido del portal y está escrito en JavaScript.
- **Editor de objetos visuales**: permite la personalización y la experiencia de creación en el explorador.

### <a name="content"></a>Contenido

El contenido se divide en dos subcategorías: *contenido del portal* y *contenido de API Management*.

El *contenido del portal* es específico del portal e incluye lo siguiente:

- **Páginas**: por ejemplo, página de aterrizaje, tutoriales de API y entradas de blog.
- **Elementos multimedia**: imágenes, animaciones y otro contenido basado en archivos.
- **Diseños**: plantillas, que se comparan con una dirección URL y definen cómo se muestran las páginas.
- **Estilos**: valores para las definiciones de los estilos; por ejemplo, fuentes, colores o bordes.
- **Valores**: configuración, como iconos de favoritos (favicon) y metadatos de sitios web.

    El contenido del portal, a excepción de los elementos multimedia, se expresa como documentos JSON.

El *contenido de API Management* incluye entidades como API, operaciones, productos y suscripciones.
## <a name="understand-the-portals-administrative-interface"></a>Descripción de la interfaz administrativa del portal

### <a name="default-content"></a>Contenido predeterminado 

Si accede al portal por primera vez, se aprovisionará automáticamente y en segundo plano el contenido predeterminado. El contenido predeterminado se ha diseñado para presentar las funcionalidades del portal y minimizar las personalizaciones necesarias para personalizarlo. Puede obtener más información sobre lo que se incluye en el contenido del portal en [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md).

### <a name="visual-editor"></a>Editor visual

Puede personalizar el contenido del portal con el editor visual. 
* Las secciones de menú de la izquierda permiten crear o modificar las páginas, los elementos multimedia, los diseños, los menús, los estilos o la configuración de sitios web. 
* Los elementos de menú de la parte inferior permiten cambiar entre las ventanillas (por ejemplo, móvil o escritorio), ver los elementos del portal visibles para los usuarios autenticados o anónimos, o guardar o deshacer acciones.
* Para agregar filas a una página, haga clic en el icono azul con un signo más. 
* Los widgets (por ejemplo, texto, imágenes o lista de API) se pueden agregar presionando un icono de color gris con un signo más.
* Reorganice los elementos de una página con la interacción de arrastrar y colocar. 

### <a name="layouts-and-pages"></a>Diseños y páginas

:::image type="content" source="media/api-management-howto-developer-portal-customize/pages-layouts.png" alt-text="Páginas y diseños" border="false":::

Los diseños definen cómo se muestran las páginas. Por ejemplo, en el contenido predeterminado, hay dos diseños: uno se aplica a la página principal y el otro a todas las demás páginas.

Para aplicar un diseño a una página, se hace coincidir la plantilla de dirección URL con la dirección URL de la página. Por ejemplo, un diseño con una plantilla de dirección URL con el valor `/wiki/*` se aplicará a todas las páginas con el segmento `/wiki/` en la dirección URL: `/wiki/getting-started`, `/wiki/styles`, etc.

En la imagen anterior, el contenido que pertenece al diseño está marcado en azul, mientras que la página se marca en rojo. Las secciones de menú se marcan respectivamente.

### <a name="styling-guide"></a>Guía de estilos

:::image type="content" source="media/api-management-howto-developer-portal-customize/styling-guide.png" alt-text="Guía de estilos" border="false":::

La guía de estilos es un panel creado teniendo en cuenta a los diseñadores. Permite supervisar todos los elementos visuales del portal y aplicar estilos en ellos. Los estilos son jerárquicos: muchos elementos heredan propiedades de otros elementos. Por ejemplo, los elementos de botón usan colores para el texto y el fondo. Para cambiar el color de un botón, debe cambiar la variante de color original.

Para editar una variante, selecciónela y seleccione el icono del lápiz que aparece encima. Una vez realizados los cambios en la ventana emergente, ciérrela.

### <a name="save-button"></a>Botón Guardar

:::image type="content" source="media/api-management-howto-developer-portal-customize/save-button.png" alt-text="Botón Guardar" border="false":::

Siempre que realice un cambio en el portal, deberá guardarlo manualmente. Para ello, seleccione el botón **Guardar** del menú de la parte inferior o pulse [Ctrl]+[S]. Al guardar los cambios, el contenido modificado se carga automáticamente en el servicio de API Management.

## <a name="customize-the-portals-content"></a>Personalización del contenido del portal

Antes de que el portal esté disponible para los visitantes, debe personalizar el contenido generado automáticamente. Entre los cambios recomendados se incluyen los diseños, los estilos y el contenido de la página principal.

> [!NOTE]
> Debido a las consideraciones sobre integración, las páginas siguientes no pueden quitarse ni moverse a una dirección URL diferente: `/404`, `/500`, `/captcha`, `/change-password`, `/config.json`, `/confirm/invitation`, `/confirm-v2/identities/basic/signup`, `/confirm-v2/password`, `/internal-status-0123456789abcdef`, `/publish`, `/signin`, `/signin-sso` y `/signup`.

### <a name="home-page"></a>Página de inicio

La página **Inicio** predeterminada se rellena con el contenido del marcador de posición. Puede eliminar todas las secciones con este contenido o mantener la estructura y ajustar los elementos uno a uno. Reemplace las imágenes y el texto generados por los suyos y asegúrese de que los vínculos señalan a las ubicaciones deseadas.

### <a name="layouts"></a>Diseños

Reemplace el logotipo generado automáticamente en la barra de navegación por el suyo.

### <a name="styling"></a>Estilos

Aunque no es necesario ajustar ningún estilo, puede ajustar elementos determinados. Por ejemplo, cambie el color principal para que coincida con el de la marca.

### <a name="customization-example"></a>Ejemplo de personalización

En el siguiente vídeo se muestra cómo editar el contenido del portal, personalizar el aspecto del sitio web y publicar los cambios.

> [!VIDEO https://www.youtube.com/embed/5mMtUSmfUlw]

## <a name="publish-the-portal"></a><a name="publish"></a> Publicación del portal

Para que el portal y los últimos cambios estén disponibles para los visitantes, debe *publicarlos*. Puede publicar el portal dentro de la interfaz administrativa del portal o desde Azure Portal.

### <a name="publish-from-the-administrative-interface"></a>Publicación desde la interfaz administrativa

1. Asegúrese de guardar los cambios; para ello, seleccione el icono **Guardar**.
1. Seleccione **Publicar sitio web** en la sección **Operaciones** del menú. Esta operación puede tardar algunos minutos.  

    :::image type="content" source="media/api-management-howto-developer-portal-customize/publish-portal.png" alt-text="Publicación del portal" border="false":::

### <a name="publish-from-the-azure-portal"></a>Publicación desde Azure Portal

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En el menú de la izquierda, en **Portal para desarrolladores**, seleccione **Información general del portal**.
1. En la ventana **Información general del portal**, seleccione **Publicar**.

    :::image type="content" source="media/api-management-howto-developer-portal-customize/pubish-portal-azure-portal.png" alt-text="Publicación del portal desde Azure Portal":::

> [!NOTE]
> Es necesario volver a publicar el portal después de los cambios en la configuración del servicio API Management. Por ejemplo, vuelva a publicar el portal después de asignar un dominio personalizado, actualizar los proveedores de identidades, configurar la delegación o especificar los términos del producto y el inicio de sesión.


## <a name="visit-the-published-portal"></a>Visita del portal publicado

Después de publicar el portal, puede acceder a él en la misma dirección URL que el panel administrativo, por ejemplo, `https://contoso-api.developer.azure-api.net`. Véalo en una sesión de explorador independiente (modo de exploración de incógnito o privada) como visitante externo.

## <a name="apply-the-cors-policy-on-apis"></a>Aplicación de la directiva CORS en las API

Debe habilitar CORS (uso compartido de recursos entre orígenes) en las API para que los visitantes del portal puedan probar las API con la consola interactiva integrada. Para más información, consulte las [preguntas frecuentes acerca del portal para desarrolladores de Azure API Management](developer-portal-faq.md#cors).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el portal para desarrolladores:

- [Introducción al portal para desarrolladores de Azure API Management](api-management-howto-developer-portal.md)
- [Migración al nuevo portal para desarrolladores](developer-portal-deprecated-migration.md) desde el portal heredado obsoleto.