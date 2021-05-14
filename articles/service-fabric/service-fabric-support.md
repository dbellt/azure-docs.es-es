---
title: Más información sobre las opciones de soporte técnico de Azure Service Fabric
description: Versiones de clúster de Azure Service Fabric compatibles y vínculos para presentar incidencias de soporte técnico
author: pkcsf
ms.topic: troubleshooting
ms.date: 8/24/2018
ms.author: pkc
ms.openlocfilehash: 9ba2f25665620524cd55a6f81c796aa024362d9d
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108123040"
---
# <a name="azure-service-fabric-support-options"></a>Opciones de soporte técnico de Azure Service Fabric

Hemos creado una serie de opciones de solicitud de soporte técnico para satisfacer las necesidades de administración de los clústeres y las cargas de trabajo de aplicación de Service Fabric, en función de la urgencia de la ayuda necesaria y de la gravedad del problema.

## <a name="create-an-azure-support-request"></a>Creación de una solicitud de soporte técnico de Azure

Para notificar los problemas relacionados con el clúster de Service Fabric que se ejecuta en Azure, abra una incidencia de soporte técnico [en Azure Portal](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) o en el [portal de soporte técnico de Microsoft](https://support.microsoft.com/oas/default.aspx?prid=16146).

Más información sobre:

- [Opciones de soporte técnico de Azure](https://azure.microsoft.com/support/plans/?b=16.44).
- [Soporte técnico Premier de Microsoft](https://support.microsoft.com/premier).

> [!Note]
> Los clústeres que se ejecutan en un nivel de confiabilidad Bronze o clúster de un solo nodo le permitirán ejecutar solo cargas de trabajo de prueba. Si experimenta problemas con un clúster que se ejecuta en una confiabilidad de nivel Bronze o clúster de un solo nodo, el equipo de soporte técnico de Microsoft le ayudará a mitigar el problema, pero no llevará a cabo un análisis de causa principal. Para obtener más información, consulte las [características de confiabilidad del clúster](./service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster).
>
> Para obtener más detalles sobre lo que es necesario para un clúster listo para producción, consulte la [lista de comprobación sobre la preparación para la producción](./service-fabric-production-readiness-checklist.md).

<a id="getlivesitesupportonprem"></a>

## <a name="create-a-support-request-for-standalone-service-fabric-clusters"></a>Creación de una solicitud de soporte técnico para clústeres de Service Fabric independientes

Si desea informar sobre problemas relacionados con los clústeres de Service Fabric que se ejecutan de forma local o en otras nubes, puede abrir una incidencia de soporte técnico en el [Portal de soporte técnico de Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

Más información sobre:

- [Soporte técnico profesional para Microsoft local](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0).
- [Soporte técnico Premier de Microsoft](https://support.microsoft.com/en-us/premier).

## <a name="post-a-question-to-microsoft-qa"></a>Publicación de una pregunta en Microsoft Q&A

Obtenga respuestas a las preguntas sobre Service Fabric directamente de los ingenieros de Microsoft, los profesionales más valiosos (MVP) de Azure y los miembros de nuestra comunidad de expertos.

[Microsoft Q&A](https://docs.microsoft.com/answers/products/) es el origen recomendado de Azure del soporte técnico de la comunidad.

Si no encuentra una respuesta a su problema en Microsoft Q&A, envíe una pregunta nueva. Asegúrese de publicar su pregunta con la etiqueta **azure-service-fabric**. Estas son algunas sugerencias de Microsoft Q&A para escribir [preguntas de alta calidad](https://docs.microsoft.com/answers/articles/24951/how-to-write-a-quality-question.html).

## <a name="open-a-github-issue"></a>Apertura de una incidencia en GitHub

Informe de los problemas de Azure Service Fabric en el [GitHub de Service Fabric](https://github.com/microsoft/service-fabric/issues). Este repositorio está diseñado para informar y rastrear problemas con Azure Service Fabric y para generar solicitudes de funciones pequeñas. **No use este medio para informar problemas con el sitio activo**.

## <a name="check-the-stackoverflow-forum"></a>Consulta del foro StackOverflow

La etiqueta `azure-service-fabric` de [StackOverflow][stackoverflow] se usa para formular preguntas generales sobre cómo funciona la plataforma y cómo usarla para llevar a cabo determinadas tareas.

## <a name="submit-feedback-on-azure-feedback"></a>Envío de comentarios sobre Azure

El [foro de comentarios de Azure para Service Fabric][uservoice-forum] es el mejor lugar para enviar ideas significativas sobre las características del producto. Revisamos las solicitudes más populares y las factorizamos para nuestra planeación de medio a largo plazo. Se recomienda buscar apoyo para sus sugerencias dentro de la comunidad.


> [!Note]
> Las **versiones preliminares de Service Fabric no se admiten para su uso en producción.** En ocasiones, creamos versiones preliminares especiales que contienen cambios significativos en las características para los que nos gustaría sondear los comentarios iniciales. Solo debe usar versiones preliminares en entornos de prueba aislados que no atiendan las cargas de trabajo de producción. El clúster de producción debe estar ejecutando siempre una versión de Service Fabric compatible y estable. No ofrecemos ninguna opción de soporte técnico de pago para estas versiones preliminares.
>
> Una versión preliminar siempre comienza con un número de versión principal y secundaria de 255. Por ejemplo, si ve una versión 255.255.5703.949 de Service Fabric, esa versión es preliminar y solo está prevista para su uso en clústeres de prueba. Estas versiones preliminares también se anuncian en el [blog del equipo de Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) y se detallan en las características incluidas. Use una de las opciones enumeradas anteriormente para proporcionar comentarios.

## <a name="next-steps"></a>Pasos siguientes

[Versiones admitidas de Service Fabric](service-fabric-versions.md)

<!--references-->
[Microsoft Q&A question page]: /answers/topics/azure-service-fabric.html
[stackoverflow]: https://stackoverflow.com/questions/tagged/azure-service-fabric
[uservoice-forum]: https://feedback.azure.com/forums/293901-service-fabric
[acom-docs]: ./index.yml
[sample-repos]: /samples/browse/?products=azure