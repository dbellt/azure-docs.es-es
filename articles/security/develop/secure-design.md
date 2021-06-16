---
title: Diseño de aplicaciones seguras en Microsoft Azure
description: En este artículo se describen los procedimientos recomendados que hay que tener en cuenta durante las fases de diseño y requisitos del proyecto de aplicación web.
author: TerryLanfear
manager: barbkess
ms.author: terrylan
ms.date: 06/11/2019
ms.topic: article
ms.service: security
ms.subservice: security-develop
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.openlocfilehash: ee4e8f3abad2a4da924e42f81004c86691a1bc8a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110792517"
---
# <a name="design-secure-applications-on-azure"></a>Diseño de aplicaciones seguras en Azure
En esta serie de artículos se presentan las actividades y controles de seguridad que hay que tener en cuenta al diseñar aplicaciones para la nube. Se abarcan los recursos de entrenamiento junto con los conceptos y preguntas de seguridad que se han de tener en cuenta durante las fases de diseño y requisitos del [ciclo de vida de desarrollo de seguridad (SDL)](/previous-versions/windows/desktop/cc307891(v=msdn.10)) de Microsoft. El objetivo es ayudarle a definir actividades y servicios de Azure que pueden usarse para diseñar una aplicación más segura.

En este artículo se tratan las siguientes fases del SDL:

- Cursos
- Requisitos
- Diseño

## <a name="training"></a>Cursos
Antes de comenzar a desarrollar la aplicación en la nube, dedique tiempo a comprender la seguridad y privacidad en Azure. Al realizar este paso, puede reducir la cantidad y gravedad de las vulnerabilidades infringibles en la aplicación. Estará mejor preparado para reaccionar adecuadamente ante el cambiante panorama de amenazas.

Utilice los recursos siguientes durante la fase de entrenamiento para familiarizarse con los servicios de Azure disponibles para los desarrolladores y con los procedimientos recomendados en Azure:

  - La [Guía de Azure para desarrolladores](https://azure.microsoft.com/campaigns/developer-guide/) le muestra cómo empezar a trabajar con Azure. La guía le muestra los servicios que puede usar para ejecutar las aplicaciones, almacenar los datos, incorporar inteligencia, compilar aplicaciones de IoT e implementar sus soluciones de forma más eficaz y segura.

  - [Guía de introducción para desarrolladores de Azure](../../guides/developer/azure-developer-guide.md) ofrece información esencial para los desarrolladores que quieran comenzar a usar la plataforma Microsoft Azure para sus necesidades de desarrollo.

  - [SDK y herramientas](../../index.yml?pivot=sdkstools) describe las herramientas que están disponibles en Azure.

  - [Azure DevOps Services](/azure/devops/) proporciona herramientas de colaboración de desarrollo. Las herramientas incluyen canalizaciones de alto rendimiento, repositorios GIT gratuitos, paneles Kanban configurables y amplias pruebas de carga automatizadas y basadas en la nube.
    El [centro de recursos de DevOps](/azure/devops/learn/) combina nuestros recursos para el aprendizaje de prácticas de DevOps, control de versiones de GIT, métodos ágiles, cómo se trabaja con DevOps en Microsoft y cómo puede evaluar su progresión de DevOps.

  - [5 elementos de seguridad principales que deben tenerse en cuenta antes de pasar a producción](/learn/modules/top-5-security-items-to-consider/index?WT.mc_id=Learn-Blog-tajanca) le muestra cómo contribuir a proteger las aplicaciones web en Azure y a proteger sus aplicaciones frente a los ataques de aplicación web más comunes y peligrosos.

  - [Secure DevOps Kit para Azure](https://azsk.azurewebsites.net/index.html) es una colección de scripts, herramientas, extensiones y automatizaciones que satisface las necesidades integrales de seguridad de recursos y suscripción a Azure de los equipos de DevOps que utilizan automatización extensiva. Secure DevOps Kit para Azure puede mostrarle cómo integrar sin problemas la seguridad en los flujos de trabajo de DevOps nativos. El kit aborda herramientas tales como pruebas de comprobación de la seguridad, que pueden ayudar a los desarrolladores escribir código seguro y probar la configuración segura de sus aplicaciones en la nube en las fases de desarrollo temprano y codificación.

  - [Security best practices for Azure solutions](https://azure.microsoft.com/resources/security-best-practices-for-azure-solutions) (Procedimientos recomendados de seguridad para soluciones de Azure) ofrece una colección de procedimientos recomendados de seguridad que se pueden usar al diseñar, implementar y administrar soluciones en la nube mediante Azure.

## <a name="requirements"></a>Requisitos
La fase de la definición de requisitos es un paso fundamental en la definición de lo que es la aplicación y lo que hará cuando se publique. La fase de requisitos también es el momento de pensar en los controles de seguridad que se compilarán en la aplicación. Durante esta fase, también se inician los pasos que realizará en todo el SDL para asegurarse de que lanza e implementa una aplicación segura.

### <a name="consider-security-and-privacy-issues"></a>Consideración de cuestiones de seguridad y privacidad
Esta fase es el mejor momento para plantearse cuestiones fundamentales de seguridad y privacidad. La definición de los niveles aceptables de seguridad y privacidad al principio de un proyecto ayuda a un equipo a:

- Comprender los riesgos asociados a problemas de seguridad.
- Identificar y corregir errores de seguridad durante el desarrollo.
- Aplicar los niveles establecidos de seguridad y privacidad a lo largo de todo el proyecto.

Al escribir los requisitos para la aplicación, ha de tener en cuenta los controles de seguridad que pueden ayudar a proteger sus datos y aplicaciones.

### <a name="ask-security-questions"></a>Formulación de preguntas de seguridad
Formule preguntas de seguridad tales como:

  - ¿Mi aplicación contiene información confidencial?

  - ¿Mi aplicación recopila o almacena datos que exigen que se cumplan estándares del sector y programas de cumplimiento como el [Consejo Federal de Inspección de Instituciones Financieras (FFIEC)](/previous-versions/azure/security/blueprints/ffiec-analytics-overview) o las [Normas de seguridad de datos del sector de las tarjetas de pago (PCI DSS)](/previous-versions/azure/security/blueprints/pcidss-analytics-overview)?

  - ¿Mi aplicación recopila o contiene datos confidenciales personales o de clientes que pueden utilizarse, solos o con otra información, para identificar, ponerse en contacto o buscar a una persona?

  - ¿Mi aplicación recopila o contiene datos que se pueden usar para acceder a información médica, educativa, financiera o laboral de una persona? La identificación de la confidencialidad de los datos durante la fase de requisitos le ayuda a clasificar los datos e identificar el método de protección de datos que va a utilizar para la aplicación.

  - ¿Dónde y cómo están los datos almacenados? Tenga en cuenta cómo va a supervisar los servicios de almacenamiento que usa la aplicación para detectar cualquier cambio inesperado de comportamiento (por ejemplo, tiempos de respuesta más lentos). ¿Podrá influir en el registro para recopilar datos más detallados y analizar un problema en profundidad?

  - ¿Mi aplicación estará disponible para el público (en Internet) o solo internamente? Si la aplicación está disponible al público, ¿cómo protege los datos que puedan recopilarse para que no se utilicen de forma incorrecta? Si la aplicación solo está disponible internamente, tenga en cuenta qué usuarios de la organización han de tener acceso a la aplicación y durante cuánto tiempo.

  - ¿Entiende el modelo de identidad antes de empezar a diseñar la aplicación? ¿Cómo va a determinar que los usuarios son quienes dicen ser y lo que un usuario está autorizado a hacer?

  - ¿Mi aplicación lleva a cabo las tareas importantes o confidenciales (por ejemplo, transferencias de dinero, desbloqueo de puertas o administración de medicamentos)?
    Tenga en cuenta cómo va a validar que el usuario que realiza una tarea confidencial está autorizado a realizarla y cómo se va a autenticar que la persona es quien dice ser. Autorización (AuthZ) es el acto de conceder permiso para hacer algo a una entidad de seguridad autenticada. Autenticación es el acto de (AuthN) de exigir credenciales legítimas a una entidad.

  - ¿Mi aplicación realiza actividades de riesgo de software, como permitir que los usuarios carguen o descarguen archivos u otros datos? Si la aplicación lleva a cabo actividades de riesgo, tenga en cuenta cómo va a proteger a los usuarios de la manipulación de datos o archivos malintencionados.

### <a name="review-owasp-top-10"></a>Revisión de OWASP Top 10
Considere la posibilidad de consultar el documento [<span class="underline">OWASP Top 10 Application Security Risks</span>](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project) (10 principales riesgos de seguridad según OWASP).
OWASP Top 10 aborda los riesgos de seguridad críticos en aplicaciones web.
El conocimiento de estos riesgos de seguridad puede ayudarle a tomar decisiones de diseño y de requisitos que minimicen estos riesgos en la aplicación.

Es importante pensar en los controles de seguridad para evitar infracciones de seguridad.
Aunque también es de esperar que se produzca [alguna infracción de seguridad](/azure/devops/learn/devops-at-microsoft/security-in-devops). Suponer que se produce una infracción de seguridad ayuda a responder de antemano a algunas preguntas importantes sobre la seguridad, para que no haya que responderlas en una emergencia:

  - ¿Cómo se detecta un ataque?

  - ¿Qué hacer si se produce un ataque o una infracción de seguridad?

  - ¿Cómo voy a recuperarme de ataques tales como filtraciones de datos o manipulaciones de datos?

## <a name="design"></a>Diseño

La fase de diseño es fundamental para el establecimiento de procedimientos recomendados para el diseño y las especificaciones funcionales. También es decisiva para realizar un análisis de riesgos que ayude a mitigar los problemas de seguridad y privacidad a lo largo de un proyecto.

Cuando se establecen requisitos de seguridad y se usan los conceptos de diseño seguro, puede evitar o minimizar las posibilidades de brecha de seguridad. Una brecha de seguridad es una omisión en el diseño de la aplicación que puede permitir a un usuario realizar acciones maliciosas o inesperadas después del lanzamiento de la aplicación.

Durante la fase de diseño, piense también en cómo puede aplicar la seguridad en capas; un nivel de defensa no es necesariamente suficiente. ¿Qué ocurre si un atacante traspasa el firewall de aplicaciones web (WAF)? Quiere establecer otro control de seguridad para defenderse contra este ataque.

Con esta intención, se analizan los siguientes conceptos de diseño seguro y los controles de seguridad que hay que abordar al diseñar aplicaciones seguras:

- Uso de una biblioteca de codificación segura y un marco de software.
- Búsqueda de componentes vulnerables.
- Uso del modelado de amenazas durante el diseño de la aplicación.
- Reducción de la superficie expuesta a ataques.
- Adopción de una directiva de identidad como perímetro de seguridad principal.
- Exigencia de reautenticación con transacciones importantes.
- Uso de una solución de administración de claves para proteger las claves, las credenciales y otros secretos.
- Protección de datos confidenciales.
- Implementación de medidas para notificaciones de error.
- Aprovechamiento del control de errores y excepciones.
- Uso del registro y las alertas.

### <a name="use-a-secure-coding-library-and-a-software-framework"></a>Uso de una biblioteca de codificación segura y un marco de software

Durante el desarrollo, utilice una biblioteca de codificación segura y un marco de software con seguridad incorporada. Los desarrolladores pueden usar características existentes y probadas (cifrado, sanitización de entrada, codificación de salida, claves o cadenas de conexión y cualquier otra cosa que se considere un control de seguridad) en lugar de desarrollar controles de seguridad desde cero. Esto ayuda a protegerse frente a errores de implementación y de diseño relacionados con la seguridad.

Asegúrese de que utiliza la versión más reciente del marco y todas las características de seguridad disponibles en él. Azure ofrece un completo [conjunto de herramientas de desarrollo](https://azure.microsoft.com/product-categories/developer-tools/) (que funcionan en cualquier plataforma o lenguaje) para que todos los desarrolladores entreguen aplicaciones en la nube. Puede programar con el lenguaje que prefiera mediante la elección de diversos [SDK](https://azure.microsoft.com/downloads/).
Puede aprovechar entornos de desarrollo integrados (IDE) completos y editores con funcionalidades de depuración avanzadas y Soporte técnico de Azure integrado.

Microsoft ofrece diversos [lenguajes, marcos y herramientas](../../index.yml?panel=sdkstools-all&pivot=sdkstools) que puede usar para desarrollar aplicaciones en Azure. Un ejemplo de ello es [Azure para desarrolladores de .NET y .NET Core](/dotnet/azure/). Para cada lenguaje y marco que ofrecemos, encontrará inicios rápidos, tutoriales y referencias de API que le ayudarán a empezar a trabajar rápidamente.

Azure ofrece diversos servicios que puede usar para hospedar aplicaciones web y sitios web. Estos servicios le permiten desarrollar en su lenguaje preferido, ya sea .NET, .NET Core, Java, Ruby, Node.js, PHP o Python.
[Azure App Service Web Apps](../../app-service/overview.md) (Web Apps) es uno de estos servicios.

Web Apps agrega el potencial de Microsoft Azure a la aplicación. Incluye seguridad, equilibrio de carga, escalado automático y administración automatizada. También puede sacar partido de las funcionalidades de DevOps en Web Apps, como, por ejemplo, administración de paquetes, entornos de ensayo, dominios personalizados, certificados SSL/TLS e implementación continua desde Azure DevOps, GitHub, Docker Hub y otros orígenes.

Azure ofrece otros servicios que puede usar para hospedar aplicaciones web y sitios web. En la mayoría de los casos, Web Apps es la mejor opción. Como arquitectura de microservicios, considere la posibilidad de usar [Azure Service Fabric](https://azure.microsoft.com/documentation/services/service-fabric).
Si necesita más control sobre las máquinas virtuales en las que se ejecuta el código, considere [Azure Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/).
Para más información sobre qué servicio de Azure elegir, consulte una [comparación de Azure App Service, Virtual Machines, Service Fabric y Cloud Services](/azure/architecture/guide/technology-choices/compute-decision-tree).

### <a name="apply-updates-to-components"></a>Aplicación de actualizaciones a componentes

Para evitar vulnerabilidades, debe hacer un inventario continuo de los componentes del lado cliente y del lado servidor (por ejemplo, marcos y bibliotecas) y sus dependencias a efectos de las actualizaciones. Continuamente se publican nuevas vulnerabilidades y versiones de software actualizadas. Asegúrese de tener un plan continuo de supervisión, evaluación de prioridades, aplicación de actualizaciones o cambios de configuración para las bibliotecas y los componentes que utiliza.

Consulte la página [Open Web Application Security Project (OWASP)](https://www.owasp.org/index.php/Main_Page) sobre el [uso de componentes con vulnerabilidades conocidas](https://www.owasp.org/index.php/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities) para obtener sugerencias de herramientas. También puede suscribirse a alertas de correo electrónico sobre vulnerabilidades de seguridad que están relacionadas con los componentes que usa.

### <a name="use-threat-modeling-during-application-design"></a>Usar el modelado de amenazas durante el diseño de la aplicación

El proceso de modelado de amenazas consiste en identificar posibles amenazas de seguridad para la empresa y la aplicación y luego asegurarse de establecer las mitigaciones adecuadas. El SDL especifica que los equipos deben participar en el modelado de amenazas durante la fase de diseño, cuando la resolución de problemas potenciales es relativamente fácil y rentable. El uso del modelado de amenazas en la fase de diseño puede reducir considerablemente el costo total de desarrollo.

Para facilitar el proceso de modelado de amenazas, diseñamos [Microsoft Threat Modeling Tool](threat-modeling-tool.md) pensando en expertos no especializados en seguridad. Esta herramienta facilita el modelado de amenazas para todos los programadores al ofrecer instrucciones claras sobre cómo crear y analizar los modelos de amenazas.

El modelado del diseño de aplicaciones y la enumeración de amenazas [STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy) —Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service y Elevation of Privilege (suplantación de identidad, manipulación, rechazo, divulgación de información, denegación de servicio y elevación de privilegios)— a través de todos los límites de la confianza ha demostrado ser una forma eficaz de detectar errores de diseño desde el principio. En esta tabla se enumeran las amenazas STRIDE y se incluyen algunas mitigaciones de ejemplo donde se usan características que Azure ofrece. Estas mitigaciones no funcionarán en todas las situaciones.

| Threat | Propiedad de seguridad | Posible mitigación de la plataforma Azure |
| ---------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Suplantación de identidad               | Authentication        | [Requiere conexiones HTTPS](/aspnet/core/security/enforcing-ssl?tabs=visual-studio). |
| Alteración de datos              | Integridad             | Valida certificados SSL/TLS. Las aplicaciones que utilizan SSL/TLS deben comprobar completamente los certificados X.509 de las entidades a las que se conectan. Usa certificados de Azure Key Vault para [administrar sus certificados x509](../../key-vault/general/about-keys-secrets-certificates.md). |
| Rechazo            | No rechazo       | Habilita opciones de [supervisión y diagnóstico](/azure/architecture/best-practices/monitoring) de Azure.|
| Divulgación de información | Confidencialidad       | Cifra datos confidenciales [en reposo](../fundamentals/encryption-atrest.md) y [en tránsito](../fundamentals/data-encryption-best-practices.md#protect-data-in-transit). |
| Denegación de servicio      | Disponibilidad          | Supervisa las métricas de rendimiento de las posibles condiciones de denegación de servicio. Implementa filtros de conexión. [Azure DDoS Protection](../../ddos-protection/ddos-protection-overview.md#next-steps), junto con los procedimientos recomendados de diseño de aplicaciones, constituyen una defensa frente a los ataques DDoS.|
| Elevación de privilegios | Authorization         | Use <span class="underline"> </span> [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md) de Azure Active Directory.|

### <a name="reduce-your-attack-surface"></a>Reducción de la superficie expuesta a ataques

Una superficie expuesta a ataques es la superficie total de los lugares en los que pueden producirse vulnerabilidades potenciales. En este artículo, nos centramos en la superficie expuesta a ataques de la aplicación.
El objetivo es proteger una aplicación contra ataques. Una manera sencilla y rápida de minimizar la superficie expuesta a ataques es quitar el código y los recursos que no se usan en la aplicación. Cuanto menor sea la aplicación, menor será la superficie expuesta a ataques. Por ejemplo, quite lo siguiente:

- Código de las características que aún no ha lanzado.
- Depuración del código de soporte.
- Interfaces de red y protocolos que no se utilizan o que han quedado en desuso.
- Máquinas virtuales y otros recursos que no se usen.

Realizar una limpieza regular de los recursos y asegurarse de quitar código que no se usa son excelentes formas de asegurarse de que hay menos posibilidades de que ataquen actores malintencionados.

Una forma más detallada y exhaustiva de reducir la superficie expuesta a ataques es completar un análisis de dicha superficie. Un análisis de la superficie expuesta a ataques le ayuda a asignar los elementos de un sistema que se han de revisar y probar para detectar vulnerabilidades de seguridad.

La finalidad de un análisis de la superficie expuesta a ataques es entender las áreas de riesgo de una aplicación para que los desarrolladores y especialistas en seguridad sepan qué componentes de la aplicación están expuestos a ataques. A continuación, puede encontrar formas de minimizar este potencial, realizar un seguimiento de cuándo y cómo cambia la superficie expuesta a ataques y lo que esto significa desde una perspectiva de riesgo.

Un análisis de la superficie expuesta a ataques ayuda a identificar:

- Funciones y componentes del sistema que hay que revisar y probar para detectar vulnerabilidades de seguridad.
- Áreas de alto riesgo del código que requieren protección de defensa en profundidad (componentes del sistema que necesita defender).
- Cuando se modifica la superficie expuesta a ataques y hay que actualizar la valoración de amenazas.

La reducción de las posibilidades de que los atacantes aprovechen un punto débil o una vulnerabilidad potencial requiere que analice a fondo la superficie global expuesta a ataques de la aplicación. También incluye la deshabilitación o restricción del acceso a los servicios del sistema, la aplicación del principio de privilegios mínimos y el empleo defensas por capas siempre que sea posible.

Veremos [cómo llevar a cabo una revisión de la superficie expuesta a ataques](secure-develop.md#conduct-attack-surface-review) durante la fase de comprobación del SDL.

> [!NOTE]
> **¿Cuál es la diferencia entre el modelado de amenazas y análisis de la superficie expuesta a ataques?**
El proceso de modelado de amenazas consiste en identificar posibles amenazas de seguridad para la aplicación y luego asegurarse de establecer las mitigaciones adecuadas frente a las amenazas. El análisis de la superficie expuesta a ataques identifica áreas de alto riesgo del código que están expuestas a ataques. Implica la búsqueda de formas de defender áreas de alto riesgo de la aplicación y la revisión y pruebas de esas áreas de código antes de implementar la aplicación.

### <a name="adopt-a-policy-of-identity-as-the-primary-security-perimeter"></a>Adoptar una directiva de identidad como perímetro de seguridad principal

Al diseñar aplicaciones en la nube, es importante ampliar el enfoque de perímetro de seguridad del centrado en redes a uno centrado en identidades. Históricamente, el perímetro de seguridad local principal era la red de la organización. La mayoría de los diseños de seguridad local utilizan la red como eje de seguridad principal. Para las aplicaciones en la nube, se recomienda considerar la identidad como perímetro de seguridad principal.

Lo que puede hacer para desarrollar un enfoque centrado en identidades para el desarrollo de aplicaciones web:

- Activación de la autenticación multifactor para usuarios.
- use plataformas sólidas de autenticación y autorización.
- Aplicación del principio de privilegios mínimos.
- Implementación de acceso Just-In-Time.

#### <a name="enforce-multi-factor-authentication-for-users"></a>Exigencia de la autenticación multifactor para usuarios

Use la autenticación en dos fases. La autenticación en dos fases es el estándar actual para la autenticación y autorización porque evita las vulnerabilidades de seguridad inherentes a tipos de nombres de usuario y contraseñas de autenticación. El acceso a las interfaces de administración de Azure (Azure Portal/PowerShell remoto) y a los servicios usados por el cliente se debe diseñar y configurar para usar [Azure Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md).

#### <a name="use-strong-authentication-and-authorization-platforms"></a>Uso de plataformas sólidas de autenticación y autorización

Use los mecanismos de autenticación y autorización proporcionados en la plataforma en lugar del código personalizado. Esto se debe a que el desarrollo de código de autenticación personalizado puede resultar propenso a errores. El código comercial (por ejemplo, de Microsoft) suele revisarse de manera amplia a efectos de seguridad. [Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) es la solución de Azure para la administración de identidades y de acceso. Estas herramientas y servicios de Azure AD contribuyen a un desarrollo seguro:

- La [plataforma de identidad de Microsoft](../../active-directory/develop/index.yml) es un conjunto de componentes que los desarrolladores usan para compilar aplicaciones que permiten el inicio de sesión seguro de los usuarios. La plataforma ayuda tanto a desarrolladores que compilan aplicaciones de línea de negocio (LOB) de inquilino único como a desarrolladores que quieren desarrollar aplicaciones multiinquilino. Además del inicio de sesión básico, las aplicaciones compiladas con la plataforma de identidad de Microsoft pueden llamar a las API de Microsoft y a API personalizadas. La plataforma de identidad de Microsoft admite protocolos estándar del sector tales como OAuth 2.0 y OpenID Connect.

- [Azure Active Directory B2C (Azure AD B2C)](../../active-directory-b2c/index.yml) es un servicio de administración de identidades que puede usar para personalizar y controlar la manera en que los clientes se registran, inician sesión y administran sus perfiles al usar las aplicaciones. Esto incluye las aplicaciones que se desarrollan para iOS, Android y .NET, entre otros. Azure AD B2C permite estas acciones al tiempo que protege las identidades de los clientes.

#### <a name="apply-the-principle-of-least-privilege"></a>Aplicación del principio de privilegios mínimos

El concepto de [privilegios mínimos ](https://en.wikipedia.org/wiki/Principle_of_least_privilege) implica conceder a los usuarios el nivel preciso de acceso y control que necesitan para hacer su trabajo y nada más.

¿Necesitará un desarrollador de software derechos de administrador de dominio? ¿Necesitará un auxiliar administrativo acceso a controles administrativos en su equipo personal? La evaluación del acceso al software de evaluación no es diferente. Si usa el [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md) para proporcionar a los usuarios distintas posibilidades y autoridad en la aplicación, no concederá a todos el acceso a todo. Al limitar el acceso a lo que se requiere para cada rol, se limita el riesgo de que se produzca un problema de seguridad.

Asegúrese de que la aplicación aplica [privilegios mínimos](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models#in-applications) a lo largo de sus patrones de acceso.

> [!NOTE]
> Las reglas de privilegios mínimos han de aplicarse al software y a las personas que lo crean. Los desarrolladores de software pueden suponer un gran riesgo para la seguridad de TI si se les da demasiado acceso. Las consecuencias pueden ser graves si un desarrollador tiene intenciones maliciosas o si se le concede demasiado acceso. Se recomienda aplicar las reglas de privilegios mínimos a los desarrolladores a lo largo del ciclo de vida de desarrollo.

#### <a name="implement-just-in-time-access"></a>Implementación de acceso Just-In-Time

Implemente acceso *Just-In-Time* (JIT) para reducir aún más el tiempo de exposición de los privilegios. Utilice [Azure AD Privileged Identity Management](../../active-directory/roles/security-planning.md#stage-3-take-control-of-administrator-activity) para:

- Conceder a los usuarios los permisos que requieren solo cuando sean necesarios.
- Asignar roles para una duración reducida con confianza de que los privilegios se revocan automáticamente.

### <a name="require-re-authentication-for-important-transactions"></a>Exigencia de reautenticación con transacciones importantes

La [falsificación de solicitud entre sitios](/aspnet/core/security/anti-request-forgery) (también conocida como *XSRF* o *CSRF*) es un ataque contra aplicaciones hospedadas en web, en el que una aplicación web malintencionada influye en la interacción entre un explorador cliente y una aplicación web que confía en ese explorador. Los ataques de falsificación de solicitud entre sitios son posibles porque los exploradores web envían automáticamente algunos tipos de token de autenticación con cada solicitud en un sitio web.
Esta forma de aprovechamiento también se conoce como *ataque con un clic* o *montaje en la sesión* porque el ataque aprovecha la sesión del usuario autenticada anteriormente.

La mejor manera de defenderse contra este tipo de ataque es pedirle al usuario algo que solo él puede proporcionar antes de cada transacción importante, como una compra, la desactivación de la cuenta o un cambio de contraseña. Puede pedirle al usuario que vuelva a escribir su contraseña, rellene un captcha o envíe un token secreto que solo él pueda tener. El enfoque más común es el token secreto.

### <a name="use-a-key-management-solution-to-secure-keys-credentials-and-other-secrets"></a>Uso de una solución de administración de claves para proteger las claves, las credenciales y otros secretos

La pérdida de claves y credenciales es un problema común. mucho peor que perder las claves y las credenciales es que otra persona no autorizada acceda a ellas. Los atacantes pueden aprovechar técnicas automatizadas y manuales para encontrar claves y secretos almacenados en repositorios de código como GitHub. No incluya claves ni secretos en estos repositorios de código públicos ni en ningún otro servidor.

Coloque siempre sus claves, certificados, secretos y cadenas de conexión en una solución de administración de claves. Puede usar una solución centralizada en la que las claves y los secretos se almacenen en módulos de seguridad de hardware (HSM). Azure ofrece un HSM en la nube con [Azure Key Vault](../../key-vault/general/overview.md).

Key Vault es un *almacén de secretos*: se trata de un servicio centralizado en la nube para el almacenamiento de secretos de aplicación. Key Vault mantiene los secretos de aplicación en una única ubicación centralizada y proporciona acceso seguro, control de permisos y registro de acceso para proteger sus datos confidenciales.

Los secretos se almacenan en *almacenes* individuales. Cada almacén tiene sus propias directivas de configuración y seguridad para controlar el acceso. Los datos se obtienen a través de una API REST o un SDK que está disponible para la mayoría de lenguajes de programación de cliente.

> [!IMPORTANT]
> Azure Key Vault se ha diseñado para almacenar secretos de configuración de las aplicaciones de servidor. No tiene por objeto almacenar datos que pertenecen a los usuarios de la aplicación. Esto se refleja en sus características de rendimiento, la API y el modelo de costos.
>
> Los datos de usuario deben almacenarse en otro lugar, como en una instancia de Azure SQL Database con cifrado de datos transparente (TDE) o en una cuenta de almacenamiento que usa Azure Storage Service Encryption. En Azure Key Vault se pueden conservar los secretos que usa la aplicación para acceder a estos almacenes de datos.

### <a name="protect-sensitive-data"></a>Protección de datos confidenciales

La protección de datos es una parte esencial de su estrategia de seguridad.
La clasificación de los datos y la identificación de las necesidades de protección de datos le ayudan a diseñar la aplicación teniendo en cuenta la seguridad de datos. La clasificación de datos almacenados por confidencialidad e impacto de negocio permite a los desarrolladores determinar los riesgos asociados a los datos.

Etiquete todos los datos aplicables como confidenciales al diseñar sus formatos de datos. Asegúrese de que la aplicación trata los datos aplicables como confidenciales. Estas prácticas pueden ayudarle a proteger sus datos confidenciales:

- Uso de cifrado.
- Evitación de secretos codificados de forma rígida como claves y contraseñas.
- Comprobación de que se han establecido controles de acceso y auditoría.

#### <a name="use-encryption"></a>Uso de cifrado

La protección de datos debe ser una parte esencial de su estrategia de seguridad.
Si los datos se almacenan en una base de datos o se mueven entre ubicaciones, use el cifrado de [datos en reposo](../fundamentals/encryption-atrest.md) (mientras se encuentran en la base de datos) y el cifrado de [datos en tránsito](../fundamentals/data-encryption-best-practices.md#protect-data-in-transit) (en su camino hacia y desde el usuario, la base de datos, una API o un punto de conexión de servicio). Se recomienda usar siempre protocolos SSL/TLS para intercambiar datos. Asegúrese de usar la versión más reciente de TLS para el cifrado (actualmente, es la versión 1.2).

#### <a name="avoid-hard-coding"></a>Evitación del codificado de forma rígida

Algunos elementos no deben codificarse nunca de forma rígida en el software. Por ejemplo, los nombres de host o las direcciones IP, las direcciones URL, las direcciones de correo electrónico, los nombres de usuario, las contraseñas, las claves de cuenta de almacenamiento y otras claves criptográficas. Considere la posibilidad de implementar requisitos en torno a lo que puede o no estar codificado de forma rígida en su código, lo que incluye las secciones de comentarios del código.

Al insertar comentarios en el código, asegúrese de no guardar ninguna información confidencial. Esto incluye su dirección de correo electrónico, contraseñas, cadenas de conexión, información sobre su aplicación que solo pueda conocer alguien de su organización y cualquier otra cosa que pudiera dar ventaja a un atacante al atacar su aplicación u organización.

Básicamente, presuponga que todo el contenido de su proyecto de desarrollo será de conocimiento público cuando se implemente. Evite incluir datos confidenciales de ningún tipo en el proyecto.

Anteriormente, analizamos [Azure Key Vault](../../key-vault/general/overview.md). Puede usar Key Vault para almacenar secretos tales como las claves y contraseñas en lugar de codificarlas de forma rígida. Si usa Key Vault en combinación con identidades administradas para recursos de Azure, la aplicación web de Azure puede acceder a valores de configuración secretos de forma fácil y segura sin almacenar ningún secreto en la configuración o el control de código fuente. Para obtener más información, consulte [Administración de secretos en aplicaciones de servidor con Azure Key Vault](/learn/modules/manage-secrets-with-azure-key-vault/).

### <a name="implement-fail-safe-measures"></a>Implementación de medidas para notificaciones de error

La aplicación debe poder controlar [errores](/dotnet/standard/exceptions/) que se produzcan durante la ejecución de forma coherente. La aplicación debe detectar todos los errores y protegerse o cerrarse.

También debe asegurarse de que los errores se registran con contexto de usuario suficiente para identificar cualquier actividad sospechosa o malintencionada. Los registros deben conservarse durante un tiempo suficiente para permitir el análisis forense diferido. Los registros deben estar en un formato que una solución de administración de registro pueda consumir fácilmente. Asegúrese de que se desencadenan alertas en errores relacionados con la seguridad. El registro y la supervisión insuficientes permiten a los atacantes seguir atacando los sistemas y mantener la persistencia.

### <a name="take-advantage-of-error-and-exception-handling"></a>Aprovechamiento del control de errores y excepciones

La implementación de un correcto [control de errores y excepciones](/dotnet/standard/exceptions/best-practices-for-exceptions) es una parte importante de la codificación defensiva. El control de errores y excepciones es fundamental para que el sistema sea confiable y seguro. Los errores en el control de errores pueden traducirse en distintos tipos de vulnerabilidades de seguridad, como filtrar información a los atacantes y permitir que los atacantes comprendan mejor su plataforma y su diseño.

Asegúrese de que:

- Controla las excepciones de forma centralizada para evitar la duplicación de [bloques try/catch](/dotnet/standard/exceptions/how-to-use-the-try-catch-block-to-catch-exceptions) en el código.

- Todos los comportamientos inesperados se controlan dentro de la aplicación.

- Los mensajes que se muestran a los usuarios no filtran datos críticos, pero ofrecen suficiente información para explicar el problema.

- Las excepciones se registran y proporcionan suficiente información para que los equipos de respuesta a incidentes o de análisis forense puedan investigarlas.

[Azure Logic Apps](../../logic-apps/logic-apps-overview.md) ofrece una experiencia de primera clase para el [control de errores y excepciones](../../logic-apps/logic-apps-exception-handling.md) producidos por sistemas dependientes. Puede usar Azure Logic Apps para crear flujos de trabajo que automaticen las tareas y los procesos de integración de aplicaciones, datos, sistemas y servicios a través de empresas y organizaciones.

### <a name="use-logging-and-alerting"></a>Uso del registro y las alertas

[Registre](/aspnet/core/fundamentals/logging/) sus problemas de seguridad a efectos de investigaciones de seguridad y desencadene alertas sobre los problemas para asegurarse de que la gente conozca los problemas de manera oportuna. Habilite la auditoría y el registro en todos los componentes. Los registros de auditoría deben capturar el contexto de usuario e identificar todos los eventos importantes.

Compruebe que no se registra ningún dato confidencial que un usuario envíe a su sitio. Ejemplos de datos confidenciales incluyen:

- Credenciales de usuario
- Número del seguro social u otra información de identificación
- Números de tarjeta de crédito u otra información financiera
- Información relacionada con la salud
- Claves privadas u otros datos que se puedan usar para descifrar información cifrada
- Información del sistema o de la aplicación que se puede usar para atacar la aplicación de forma más eficaz

Compruebe que la aplicación supervisa los eventos de administración de usuarios como los inicios de sesión de usuario correctos e incorrectos, el restablecimiento de contraseñas, los cambios de contraseña, el bloqueo de cuentas y el registro de usuarios. El registro de estos eventos le ayuda a detectar comportamientos potencialmente sospechosos y reaccionar ante ellos. También le permite recopilar datos de operaciones, como, por ejemplo, quién accede a la aplicación.

## <a name="next-steps"></a>Pasos siguientes
En los siguientes artículos, se recomiendan controles de seguridad y actividades que pueden ayudarle a desarrollar e implementar aplicaciones seguras.

- [Desarrollo de aplicaciones seguras](secure-develop.md)
- [Implementación de aplicaciones seguras](secure-deploy.md)