---
title: Introducción a las directivas personalizadas de Azure Active Directory B2C | Microsoft Docs
description: Un tema sobre las directivas personalizadas de Azure Active Directory B2C y el marco de experiencia de identidad.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/08/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2a8aabac6960909f2a3d90fcee01cebb0ad7a832
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256946"
---
# <a name="azure-ad-b2c-custom-policy-overview"></a>Información general sobre las directivas personalizadas de Azure AD B2C

Las directivas personalizadas son archivos de configuración que definen el comportamiento del inquilino de Azure Active Directory B2C (Azure AD B2C). Mientras que en el portal de Azure AD B2C se predefinen [flujos de usuario](user-flow-overview.md) para las tareas de identidad más comunes, un desarrollador de identidad puede editar las directivas personalizadas en su totalidad a fin de realizar muchas tareas diferentes.

Una directiva personalizada es totalmente configurable y se controla mediante directivas. Una directiva personalizada orquesta la confianza entre entidades en protocolos estándar. Por ejemplo, OpenID Connect, OAuth, SAML y otros no estándar, como intercambios de notificaciones de sistema a sistema basados en la API REST. El marco crea experiencias propias fáciles de usar.

Una directiva personalizada se representa como uno o más archivos con formato XML que se hacen referencia entre sí en una cadena jerárquica. Los elementos XML definen los bloques de creación, la interacción con el usuario y otras entidades, y la lógica de negocios. 

## <a name="custom-policy-starter-pack"></a>Paquete de inicio de directivas personalizadas

El [paquete de inicio](tutorial-create-user-flows.md?pivots=b2c-custom-policy#get-the-starter-pack) de directivas predeterminadas de Azure AD B2C viene con varias directivas predefinidas para que pueda empezar a trabajar rápidamente. Cada uno de estos paquetes de inicio contiene el menor número de perfiles técnicos y recorridos del usuario necesarios para lograr los escenarios descritos:

- **LocalAccounts**: habilita el uso solo de cuentas locales.
- **SocialAccounts**: habilita el uso solo de cuentas sociales (o federadas).
- **SocialAndLocalAccounts**: habilita el uso de cuentas locales y sociales. La mayoría de los ejemplos hacen referencia a esta directiva.
- **SocialAndLocalAccountsWithMFA**: habilita opciones sociales, locales y de autenticación multifactor.

## <a name="understanding-the-basics"></a>Descripción de los conceptos básicos 

### <a name="claims"></a>Notificaciones

Una notificación proporciona un almacenamiento temporal de datos durante la ejecución de una directiva de Azure AD B2C. Puede almacenar información sobre el usuario, como el nombre, el apellido o cualquier otra notificación obtenida del usuario u otros sistemas (intercambios de notificaciones). El [esquema de notificaciones](claimsschema.md) es el lugar en el que se declaran las notificaciones. 

Al ejecutarse la directiva, Azure AD B2C envía y recibe notificaciones de entidades internas y externas y, a continuación, envía un subconjunto de estas notificaciones a su aplicación de usuario de confianza como parte del token. Las notificaciones se utilizan de las siguientes maneras: 

- Una notificación se guarda o se actualiza en el objeto de usuario de directorio.
- Se recibe una notificación de un proveedor de identidades externo.
- Las notificaciones se envían o reciben mediante un servicio de la API de REST personalizado.
- Los datos se recopilan como notificaciones del usuario durante los flujos de perfil de edición o registro.

### <a name="manipulating-your-claims"></a>Manipulación de las notificaciones

Las [transformaciones de notificaciones](claimstransformations.md) son funciones predefinidas que se pueden usar para convertir una notificación especificada en otra, evaluar una notificación o establecer un valor de notificación. Por ejemplo, agregar un elemento a una colección de cadenas, cambiar las mayúsculas y minúsculas de una cadena o evaluar una notificación de fecha y hora. Una transformación de notificaciones especifica un método de transformación. 

### <a name="customize-and-localize-your-ui"></a>Personalización y localización de la interfaz de usuario

Si desea recopilar información de los usuarios presentando una página en su explorador web, use el [perfil técnico autoafirmado](self-asserted-technical-profile.md). Puede editar el perfil técnico autoafirmado para [agregar notificaciones y personalizar la entrada de usuario](./configure-user-input.md).

Para [personalizar la interfaz de usuario](customize-ui-with-html.md) para su perfil técnico autoafirmado, especifique una dirección URL en el elemento [definición de contenido](contentdefinitions.md) con contenido HTML personalizado. En el perfil técnico autoafirmado, apunte a este identificador de definición de contenido.

Para personalizar cadenas específicas del idioma, use el elemento [Localization](localization.md). Una definición de contenido puede contener una referencia [Localization](localization.md) que especifique una lista de los recursos localizados que se van a cargar. Azure AD B2C combina elementos de la interfaz de usuario con el contenido HTML cargado desde la URL y, después, muestra la página al usuario. 

## <a name="relying-party-policy-overview"></a>Información general de la directiva del usuario de confianza

Una aplicación de usuario de confianza, que en el protocolo SAML se conoce como un proveedor de servicios, llama a la [directiva del usuario de confianza](relyingparty.md) para ejecutar un recorrido específico. La directiva del usuario de confianza especifica el recorrido del usuario que se va a ejecutar, así como la lista de notificaciones que incluye el token. 

![Diagrama que muestra el flujo de ejecución de la directiva](./media/custom-policy-overview/custom-policy-execution.png)

Todas las aplicaciones de usuario de confianza que usan la misma directiva reciben las mismas notificaciones de token, y el usuario realiza el mismo recorrido.

### <a name="user-journeys"></a>Recorridos de usuario

[Recorridos del usuario](userjourneys.md) le permite definir la lógica de negocios con la ruta de acceso que seguirá el usuario para obtener acceso a la aplicación. Se conduce al usuario por el recorrido del usuario para recuperar las notificaciones que se van a presentar a la aplicación. Un recorrido del usuario se genera a partir de una secuencia de [pasos de orquestación](userjourneys.md#orchestrationsteps). Un usuario debe llegar al último paso para adquirir un token. 

En las siguientes instrucciones se explica cómo agregar pasos de orquestación a la directiva del [paquete de inicio de la cuenta local y social](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccounts). Este es un ejemplo de una llamada API REST que se ha agregado.

![recorrido del usuario personalizado](media/custom-policy-overview/user-journey-flow.png)


### <a name="orchestration-steps"></a>Estados de orquestación

El paso de orquestación hace referencia a un método que implementa su propósito o funcionalidad. Este método se denomina [perfil técnico](technicalprofiles.md). Cuando el recorrido del usuario necesita una bifurcación para representar mejor la lógica de negocios, el paso de orquestación hace referencia a un [subrecorrido](subjourneys.md). Un subrecorrido contiene su propio conjunto de pasos de orquestación.

Un usuario debe alcanzar el último paso de orquestación en el recorrido del usuario para adquirir un token. Sin embargo, es posible que los usuarios no necesiten seguir todos los pasos de orquestación. Los pasos de orquestación se pueden ejecutar de forma condicional en función de las [condiciones previas](userjourneys.md#preconditions) definidas en el paso de orquestación. 

Una vez que se completa un paso de orquestación, Azure AD B2C almacena las notificaciones generadas en el **contenedor de notificaciones**. Cualquier otro paso de orquestación puede usar las notificaciones del contenedor de notificaciones en el recorrido del usuario.

En el diagrama siguiente se muestra cómo los pasos de orquestación del recorrido del usuario pueden tener acceso al contenedor de notificaciones.

![Recorrido del usuario de Azure AD B2C](media/custom-policy-overview/user-journey-diagram.png)

### <a name="technical-profile"></a>Perfil técnico

Un perfil técnico proporciona una interfaz para comunicarse con distintos tipos de entidades. Un recorrido del usuario combina la llamada a perfiles técnicos a través de los pasos de orquestación para definir la lógica de negocios.

Todos los tipos de perfiles técnicos comparten el mismo concepto. Puede enviar notificaciones de entrada, ejecutar la transformación de notificaciones y comunicarse con la entidad configurada. Una vez finalizado el proceso, el perfil técnico devuelve las notificaciones de salida al contenedor de notificaciones. Para obtener más información, vea la [introducción a los perfiles técnicos](technicalprofiles.md).

### <a name="validation-technical-profile"></a>Perfil técnico de validación

Cuando un usuario interactúa con la interfaz de usuario, es posible que desee validar los datos que se recopilan. Para interactuar con el usuario, debe usarse un [perfil técnico autoafirmado](self-asserted-technical-profile.md).

Para validar la entrada de usuario, se llama a un [perfil técnico de validación](validation-technical-profile.md) desde el perfil técnico autoafirmado. Un perfil técnico de validación es un método para llamar a cualquier perfil técnico no interactivo. En este caso, el perfil técnico puede devolver notificaciones de salida o un mensaje de error. El mensaje de error se representa para el usuario en la pantalla, lo que le permite volver a intentarlo.

En el siguiente diagrama se muestra cómo Azure AD B2C usa un perfil técnico de validación para validar las credenciales de usuario.

![Diagrama del perfil técnico de validación](media/custom-policy-overview/validation-technical-profile.png)

## <a name="inheritance-model"></a>Modelo de herencia

Cada paquete de inicio incluye los siguientes archivos:

- Un archivo de **base** que contiene la mayoría de las definiciones. Para facilitar la solución de problemas y el mantenimiento a largo plazo de las directivas, intente minimizar el número de cambios realizados en este archivo.
- Un archivo de **extensiones** que contiene los cambios de configuración únicos del inquilino. Este archivo de directiva se basa en el archivo de base. Use este archivo para agregar nuevas funciones o reemplazar funciones existentes. Por ejemplo, puede usar este archivo para federar con nuevos proveedores de identidades.
- Un archivo de **usuario de confianza** que es el único archivo centrado en tareas que invoca directamente la aplicación de usuario de confianza, como aplicaciones de escritorio, móviles o web. Cada tarea única (como un registro, un inicio de sesión, un restablecimiento de contraseña o una edición de perfil) necesita su propio archivo de directiva de usuario de confianza. Este archivo de directiva se basa en el archivo de extensiones.

Este es el modelo de herencia:

- La directiva secundaria de cualquier nivel puede heredar de la directiva principal y extenderse mediante la adición de nuevos elementos.
- En escenarios más complejos, puede agregar más niveles de herencia (hasta 10 en total).
- Puede agregar más directivas del usuario de confianza. Por ejemplo, elimine mi cuenta, cambie un número de teléfono, una directiva del usuario de confianza de SAML y mucho más.

En el diagrama siguiente, se muestra la relación entre los archivos de directiva y las aplicaciones de usuario de confianza.

![Diagrama que muestra el modelo de herencia de directiva del marco de confianza](media/custom-policy-overview/policies.png)


## <a name="guidance-and-best-practices"></a>Guía y procedimientos recomendados

### <a name="best-practices"></a>Procedimientos recomendados

En una directiva personalizada de Azure AD B2C puede integrar su propia lógica de negocios a fin de compilar las experiencias de usuario que requiere y extender la funcionalidad del servicio. Tenemos un conjunto de procedimientos recomendados y recomendaciones para comenzar.

- Cree la lógica en la **directiva de extensión** o la **directiva del usuario de confianza**. Puede agregar nuevos elementos, lo que invalidará la directiva base haciendo referencia al mismo identificador. Esto le permitirá escalar horizontalmente el proyecto mientras se facilita la actualización de la directiva base posteriormente si Microsoft lanza nuevos paquetes de inicio.
- En la **directiva base**, recomendamos encarecidamente evitar realizar cualquier tipo de cambio. En caso necesario, realice comentarios donde se hayan hecho los cambios.
- Al invalidar un elemento, como los metadatos del perfil técnico, evite copiar todo el perfil técnico de la directiva base. En su lugar, copie solo la sección necesaria del elemento. Consulte [Deshabilitación de la comprobación de correo electrónico](./disable-email-verification.md) para ver un ejemplo de cómo hacer el cambio.
- Para reducir la duplicación de perfiles técnicos, donde se comparte la funcionalidad básica, use la [inclusión del perfil técnico](technicalprofiles.md#include-technical-profile).
- Evite escribir en el directorio de Azure AD durante el inicio de sesión, lo que puede provocar problemas de limitación.
- Si la directiva tiene dependencias externas, como API REST, asegúrese de que tengan alta disponibilidad.
- Para lograr una mejor experiencia de usuario, asegúrese de que las plantillas HTML personalizadas se implementan globalmente con la [entrega de contenido en línea](../cdn/index.yml). Azure Content Delivery Network (CDN) permite reducir los tiempos de carga, ahorrar ancho de banda y mejorar la velocidad de respuesta.
- Si quiere realizar un cambio en un recorrido, copie todo el recorrido del usuario de la directiva base en la directiva de extensión. Proporcione un identificador de recorrido único al recorrido del usuario que ha copiado. A continuación, en la [directiva del usuario de confianza](relyingparty.md), cambie el elemento del [recorrido del usuario predeterminado](relyingparty.md#defaultuserjourney) para apuntar al nuevo recorrido del usuario.

## <a name="troubleshooting"></a>Solución de problemas

Al llevar a cabo el desarrollo con las directivas de Azure AD B2C, es posible que se produzcan errores o excepciones al ejecutar el recorrido del usuario. Se pueden investigar mediante Application Insights.

- Integre Application Insights con Azure AD B2C para [diagnosticar excepciones](troubleshoot-with-application-insights.md).
- La [extensión Azure AD B2C para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c) puede ayudar a acceder y [visualizar los registros](https://github.com/azure-ad-b2c/vscode-extension/blob/master/src/help/app-insights.md) en función de una hora y un nombre de directiva.
- El error más común al configurar directivas personalizadas es el formato incorrecto del código XML. Use la [validación del esquema XML](troubleshoot-custom-policies.md) para identificar los errores antes de cargar el archivo XML.

## <a name="continuous-integration"></a>Integración continua

Mediante el uso de una canalización de integración y entrega continuas (CI/CD) configurada en Azure Pipelines, puede [incluir las directivas personalizadas de Azure AD B2C en la entrega de software](deploy-custom-policies-devops.md) y la automatización del control de código. A medida que se implementan en diferentes entornos de Azure AD B2C, por ejemplo, en desarrollo, prueba y producción, se recomienda quitar los procesos manuales y realizar pruebas automatizadas con Azure Pipelines.

## <a name="prepare-your-environment"></a>Preparación del entorno

Empiece a trabajar con la directiva personalizada de Azure AD B2C:

1. [Creación de un inquilino de Azure AD B2C](tutorial-create-tenant.md)
1. [Registro de una aplicación web](tutorial-register-applications.md) mediante Azure Portal para poder probar la directiva.
1. Incorporación de las [claves de directiva](tutorial-create-user-flows.md?pivots=b2c-custom-policy#add-signing-and-encryption-keys) y [Registro de las aplicaciones de Identity Experience Framework](tutorial-create-user-flows.md?pivots=b2c-custom-policy#register-identity-experience-framework-applications).
1. [Obtenga el paquete de inicio de directivas de Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy#get-the-starter-pack) y cárguelo en el inquilino. 
1. Después de cargar el paquete de inicio, realice una [prueba de la directiva de registro o de inicio de sesión](tutorial-create-user-flows.md?pivots=b2c-custom-policy#test-the-custom-policy).
1. Le recomendamos que descargue e instale [Visual Studio Code](https://code.visualstudio.com/) (VS Code). Visual Studio Code es un editor de código fuente ligero pero eficaz que se ejecuta en el escritorio y está disponible para Windows, macOS y Linux. Con VS Code puede desplazarse rápidamente por los archivos XML de directivas personalizadas de Azure AD B2C y editarlos si instala la [extensión Azure AD B2C para VS Code](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c)
 
## <a name="next-steps"></a>Pasos siguientes

Después de configurar y probar la directiva de Azure AD B2C, puede empezar a personalizar la directiva. Consulte los artículos siguientes para obtener información sobre cómo:

- [Agregar notificaciones y personalizar la entrada de usuario](./configure-user-input.md) mediante directivas personalizadas. Aprenda a definir una notificación y agregue una a la interfaz de usuario mediante la personalización de algunos de los perfiles técnicos del paquete de inicio.
- [Personalizar la interfaz de usuario](customize-ui-with-html.md) de la aplicación mediante una directiva personalizada. Aprenda a crear su propio contenido HTML y personalice la definición de contenido.
- [Encontrar la interfaz de usuario](./language-customization.md) de la aplicación mediante una directiva personalizada. Aprenda a configurar la lista de idiomas admitidos y proporcione etiquetas específicas del idioma agregando el elemento de recursos localizado.
- Durante el desarrollo y las pruebas de la directiva puede [deshabilitar la comprobación del correo electrónico](./disable-email-verification.md). Aprenda a sobrescribir los metadatos de un perfil técnico.
- [Configurar el inicio de sesión con una cuenta de Google](./identity-provider-google.md) mediante directivas personalizadas. Aprenda a crear un nuevo proveedor de notificaciones con el perfil técnico de OAuth2. A continuación, personalice el recorrido del usuario para incluir la opción de inicio de sesión de Google.
- Para diagnosticar problemas con las directivas personalizadas, puede [recopilar registros de Azure Active Directory B2C con Application Insights](troubleshoot-with-application-insights.md). Aprenda a agregar nuevos perfiles técnicos y configure la directiva del usuario de confianza.