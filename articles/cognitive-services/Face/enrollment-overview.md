---
title: Procedimientos recomendados para agregar usuarios a un servicio Face
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre el proceso de inscripción de Face para registrar usuarios en un servicio de reconocimiento de caras.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: overview
ms.date: 04/19/2021
ms.author: pafarley
ms.openlocfilehash: 183983779c09658d8d6f609319a127b1f406452d
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107728435"
---
# <a name="best-practices-for-adding-users-to-a-face-service"></a>Procedimientos recomendados para agregar usuarios a un servicio Face

Para usar la API Face de Cognitive Services para la comprobación o identificación facial, debe inscribir las caras en **LargePersonGroup** o una estructura de datos parecida. En este análisis en profundidad se muestran los procedimientos recomendados para recopilar un consentimiento significativo de los usuarios y la lógica de ejemplo para crear inscripciones de gran calidad que optimicen la precisión del reconocimiento. 

## <a name="meaningful-consent"></a>Consentimiento significativo 

Uno de los objetivos clave de una aplicación de inscripción para el reconocimiento facial es brindar a los usuarios la oportunidad de dar su consentimiento para el uso de imágenes de su rostro para fines concretos, como el acceso a un sitio de trabajo. Como se podría pensar que las tecnologías de reconocimiento facial recopilan datos personales confidenciales, es especialmente importante pedir el consentimiento de forma transparente y respetuosa. El consentimiento es significativo para los usuarios cuando les permite tomar la decisión que cree que es la mejor se para ellos.   

En función de la investigación de usuarios de Microsoft, de los principios de inteligencia artificial responsable de Microsoft y de la [investigación externa](ftp://ftp.cs.washington.edu/tr/2000/12/UW-CSE-00-12-02.pdf), hemos descubierto que el consentimiento es significativo cuando ofrece lo siguiente a los usuarios que se inscriben en la tecnología:

* Conocimiento: los usuarios no deberían tener la más mínima duda cuando se les pida que proporcionen sus fotos de inscripción o su plantilla de caras. 
* Descripción: los usuarios deberían poder describir con precisión con sus propias palabras lo que se les pide, quién se lo ha pedido, con qué fin y con qué garantías. 
* Libertad de elección: los usuarios no deberían sentirse coaccionados o manipulados al decidir si dan su consentimiento y se inscriben en el reconocimiento facial. 
* Control: los usuarios deberían poder revocar su consentimiento y eliminar sus datos en cualquier momento. 

En esta sección se ofrece una guía para desarrollar una aplicación de inscripción para el reconocimiento facial. Esta guía se ha desarrollado basándose en la investigación de usuarios de Microsoft en el contexto de la inscripción de personas en el reconocimiento facial para el acceso a edificios. Por lo tanto, es posible que estas recomendaciones no se apliquen a todas las soluciones de reconocimiento faciales. El uso responsable de Face API depende en gran medida del contexto específico en el que está integrado, por lo que la tanto priorización como la aplicación de estas recomendaciones deben adaptarse a su escenario. 

> [!NOTE]
> Es responsabilidad suya adecuar su aplicación de inscripción a los requisitos legales aplicables en su jurisdicción y reflejar con precisión todas las prácticas de recopilación y procesamiento de datos.

## <a name="application-development"></a>Desarrollo de aplicaciones 

Antes de diseñar un flujo de inscripción, piense en la forma en que la aplicación que está compilando puede mantener las promesas que realiza a los usuarios sobre cómo se protegen sus datos. Las siguientes recomendaciones pueden ayudarle a crear una experiencia de inscripción que incluye métodos responsables de protección de datos personales, la administración de la privacidad de los usuarios y la garantía de que todos los usuarios pueden acceder a la aplicación.  

|Category | Recomendaciones |
|---|---|
|Hardware | Considere la calidad de la cámara del dispositivo de inscripción. |
|Características de inscripción recomendadas | Incluya un paso de inicio de sesión con autenticación multifactor. </br></br>Vincule la información del usuario, como un alias o un número de identificación, con su identificador de plantilla de caras del Face API (conocido como identificador de persona). Esta asignación es necesaria para recuperar y administrar la inscripción de un usuario. Nota: el identificador de persona debe tratarse como un secreto en la aplicación.</br></br>Configure un proceso automatizado para eliminar todos los datos de inscripción, lo que incluye las plantillas de caras y las fotos de inscripción de aquellas personas que han dejado de ser usuarios de la tecnología de reconocimiento facial, como los empleados anteriores. </br></br>Evite la inscripción automática, ya que no proporciona al usuario el conocimiento, la descripción, la libertad de elección o el control que se recomiendan para obtener el consentimiento. </br></br>Pida a los usuarios permiso para guardar las imágenes que se usan para la inscripción. Esto resulta útil cuando hay una actualización del modelo, ya que se requerirán nuevas fotos de inscripción para volver a inscribirse en el nuevo modelo cada diez meses. Si no se guardan las imágenes originales, los usuarios deberán realizar el proceso de inscripción desde el principio.</br></br>Permita a los usuarios no participar en el almacenamiento de fotografías en el sistema. Para que la elección sea más clara, puede agregar una segunda pantalla de solicitud de consentimiento para guardar las fotos de inscripción. </br></br>Si se guardan fotos, cree un proceso automatizado para volver a inscribir a todos los usuarios cuando haya una actualización del modelo. Los usuarios que hayan guardado sus fotos de inscripción no tendrán que volver a inscribirse. </br></br>Cree una característica de aplicación que permita a los administradores designados reemplazar determinados filtros de calidad si un usuario tiene problemas para inscribirse. |
|Seguridad | Cognitive Services sigue los [procedimientos recomendados](../cognitive-services-virtual-networks.md?tabs=portal) para cifrar los datos de usuario tanto en reposo como en tránsito. A continuación se muestran varios procedimientos adicionales que pueden ayudar a mantener los compromisos de seguridad que se realizan a los usuarios durante la experiencia de inscripción. </br></br>Tome las medidas de seguridad necesarias para asegurarse de que nadie tenga acceso al identificador de la persona en ningún momento durante la inscripción. Nota: el identificador de persona debe tratarse como un secreto en el sistema de inscripción. </br></br>Use el [control de acceso basado en rol](../../role-based-access-control/overview.md) en Cognitive Services. </br></br>Use la autenticación basada en tokens o las firmas de acceso compartido (SAS) sobre las claves y los secretos para acceder a ciertos recursos, como las bases de datos. Mediante el uso una solicitud o de tokens SAS, puede conceder acceso limitado a los datos sin poner en peligro las claves de su cuenta, y también puede especificar una hora de expiración en el token. </br></br>No almacene secretos, claves o contraseñas en la aplicación. |
|Privacidad del usuario |Proporcione varias opciones de inscripción para abordar los problemas de los distintos niveles privacidad. No exija que se usen los dispositivos personales para inscribirse en un sistema de reconocimiento facial. </br></br>Deje que los usuarios se vuelvan a inscribir, revoquen el consentimiento y eliminen datos de la aplicación de inscripción en cualquier momento y por cualquier motivo. |
|Accesibilidad |Siga los estándares de accesibilidad (por ejemplo, [ADA](https://www.ada.gov/regs2010/2010ADAStandards/2010ADAstandards.htm) o [W3C](https://www.w3.org/TR/WCAG21/)) para asegurarse de que la aplicación pueda ser utilizada por personas con discapacidades visuales o motoras. |

## <a name="next-steps"></a>Pasos siguientes  

Siga la guía para la [creación de aplicaciones de inscripción](build-enrollment-app.md) para empezar a trabajar con una aplicación de inscripción de ejemplo. Luego, personalícela o cree su propia aplicación para ajustarse a las necesidades de su producto.