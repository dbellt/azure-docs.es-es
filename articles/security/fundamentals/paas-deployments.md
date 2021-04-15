---
title: Procedimientos recomendados para implementaciones seguras de PaaS - Microsoft Azure
description: Conozca los procedimientos recomendados para diseñar, compilar y administrar aplicaciones seguras en la nube en Azure y comprender las ventajas de seguridad de PaaS en comparación con otros modelos de servicio en la nube.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: techlake
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/29/2021
ms.author: terrylan
ms.openlocfilehash: d28d55c8dff16df987f6535b0f1452b840b35c43
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105960510"
---
# <a name="securing-paas-deployments"></a>Protección de implementaciones de PaaS

En este artículo se proporciona información que ayuda a:

- Comprender las ventajas de seguridad del hospedaje de aplicaciones en la nube
- Evaluar las ventajas de seguridad de plataforma como servicio (PaaS) en comparación con otros modelos de servicio en la nube
- Cambiar el enfoque de seguridad de un perímetro basado en la red por un perímetro basado en identidades
- Implementar recomendaciones generales de procedimientos recomendados de PaaS

[Desarrollo de aplicaciones seguras en Azure](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/) es una guía general que incluye los controles y las preguntas de seguridad que se deben considerar en cada fase del ciclo de vida de desarrollo de software al desarrollar aplicaciones para la nube.

## <a name="cloud-security-advantages"></a>Ventajas de seguridad en la nube
Es importante comprender la [división de responsabilidad](shared-responsibility.md) entre usted y Microsoft. De forma local, es propietario de la pila completa, pero a medida que se pasa a la nube, algunas responsabilidades se transfieren a Microsoft.

La [nube ofrece ventajas de seguridad](shared-responsibility.md#cloud-security-advantages). En un entorno local, las organizaciones probablemente tengan responsabilidades inadecuadas y recursos limitados disponibles para invertir en seguridad, de tal manera que se crea un entorno donde los atacantes pueden aprovechar vulnerabilidades a todos los niveles.

Las organizaciones pueden mejorar los tiempos de detección de amenazas y respuesta mediante la utilización de inteligencia de la nube y funciones de seguridad basada en la nube del proveedor.  Con la transferencia de responsabilidades al proveedor de nube, las organizaciones pueden obtener más cobertura de seguridad, lo que les permite reasignar recursos de seguridad y presupuestos a otras prioridades empresariales.

## <a name="security-advantages-of-a-paas-cloud-service-model"></a>Ventajas de seguridad de un modelo de servicio en la nube de PaaS
Echemos un vistazo a las ventajas de seguridad de una implementación de PaaS de Azure frente a las instalaciones locales.

![Ventajas de seguridad de PaaS](./media/paas-deployments/advantages-of-paas.png)

A partir de la parte inferior de la pila, la infraestructura física, Microsoft mitiga las responsabilidades y los riesgos comunes. Dado que Microsoft supervisa continuamente Microsoft Cloud, es difícil atacarla. No tiene sentido que un atacante se fije Microsoft Cloud como un objetivo. A menos que el atacante tenga una gran cantidad de dinero y recursos, lo más probable es que el atacante se fije otro objetivo.  

En el medio de la pila, no hay ninguna diferencia entre una implementación de PaaS y una implementación local. A nivel de aplicación y de cuenta y administración del acceso, tiene riesgos similares. En la siguiente sección de pasos de este artículo, presentamos los procedimientos recomendados para eliminar o minimizar estos riesgos.

En la parte superior de la pila, gobernanza de datos y administración de derechos, corre un riesgo que puede mitigarse con la administración de claves. (La administración de claves se cubre en los procedimientos recomendados). Si bien la administración de claves es una responsabilidad adicional, tiene áreas en una implementación de PaaS que ya no tiene que administrar, por lo que puede mover recursos a la administración de claves.

La plataforma Azure también proporciona protección segura contra ataques DDoS mediante el uso de varias tecnologías basadas en red. Sin embargo, todos los tipos de métodos de protección contra ataques DDoS basados en red tienen sus límites por vínculo y por centro de datos. Para ayudar a evitar el impacto de los ataques DDoS grandes, puede beneficiarse de la funcionalidad de la nube principal de Azure que permite escalar horizontalmente de manera rápida y automática para defenderse frente a ataques DDoS. En los artículos de procedimientos recomendados se proporcionan más detalles sobre cómo se puede hacer esto.

## <a name="modernizing-the-defenders-mindset"></a>Modernización de la mentalidad del defensor
Las implementaciones de PaaS conllevan un cambio del enfoque general en relación con la seguridad. Se pasa de la necesidad de controlar todo por uno mismo a compartir la responsabilidad con Microsoft.

Otra importante diferencia entre las implementaciones de PaaS y las implementaciones locales tradicionales es una nueva visión de lo que define el perímetro de seguridad principal. Históricamente, el perímetro de seguridad local principal era la red, y la mayoría de los diseños de seguridad locales utilizan la red como la dinámica de seguridad principal. Para las implementaciones de PaaS, se recomienda considerar la identidad como el perímetro de seguridad principal.

## <a name="adopt-a-policy-of-identity-as-the-primary-security-perimeter"></a>Adoptar una directiva de identidad como perímetro de seguridad principal
Una de las cinco características fundamentales de la informática en la nube es el acceso amplio a la red, lo que hace que el concepto basado en la red resulte menos relevante. El objetivo de gran parte de la informática en la nube es permitir a los usuarios acceder a los recursos con independencia de su ubicación. Para la mayoría de los usuarios, su ubicación estará en alguna parte de Internet.

En la figura siguiente se muestra cómo el perímetro de seguridad ha evolucionado de un perímetro de red a un perímetro de identidad. La seguridad se centra menos en proteger la red y más en proteger los datos, así como en administrar la seguridad de las aplicaciones y los usuarios. La diferencia principal es que la intención es basar más la seguridad en lo que resulta importante para su empresa.

![La identidad como el nuevo perímetro de seguridad](./media/paas-deployments/identity-perimeter.png)

Inicialmente, los servicios PaaS en Azure (por ejemplo, los roles web y Azure SQL) ofrecían poca o ninguna defensa para el perímetro de red tradicional. Se entiende que el propósito del elemento era exponerse a Internet (rol web) y que la autenticación ofrece el nuevo perímetro (por ejemplo, BLOB o Azure SQL).

Las prácticas de seguridad modernas asumen que el adversario ha infringido el perímetro de red. Por lo tanto, las prácticas de defensa modernas se han pasado al perímetro de identidad. Las organizaciones deben establecer un perímetro de seguridad basado en identidades con autenticación sólida e higiene de autorización (procedimientos recomendados).

Durante décadas, se han encontrado disponibles principios y patrones para el perímetro de red. En cambio, el sector tiene relativamente poca experiencia con el uso de la identidad como el perímetro de seguridad principal. Dicho esto, se ha adquirido suficiente experiencia para ofrecer algunas recomendaciones generales que se han probado sobre el terreno y que se han aplicado a casi todos los servicios PaaS.

Aquí tiene unas recomendaciones para administrar el perímetro de identidad.

**Procedimiento recomendado**: proteger las claves y las credenciales para proteger la implementación de PaaS.   
**Detalles**: La pérdida de claves y credenciales es un problema común. Puede usar una solución centralizada que permita almacenar claves y secretos en módulos de seguridad de hardware (HSM). [Azure Key Vault](../../key-vault/general/overview.md) guarda claves y secretos mediante el cifrado de claves de autenticación, claves de cuenta de almacenamiento, claves de cifrado de datos, archivos .pfx y contraseñas a través del uso de claves protegidas por HSM.

**Procedimiento recomendado**: no incluya credenciales ni otros secretos en el código fuente ni en GitHub.   
**Detalles**: mucho peor que perder las claves y las credenciales es que otra persona no autorizada acceda a ellas. Los atacantes pueden aprovechar las tecnologías de bots para encontrar claves y secretos almacenados en repositorios de código, como GitHub. No guarde claves ni secretos en estos repositorios públicos de código.

**Procedimiento recomendado**: proteja las interfaces de administración de las máquinas virtuales en servicios híbridos PaaS e IaaS. Para ello, use una interfaz de administración que permita administrar de manera remota estas máquinas virtuales directamente.   
**Detalles**: se pueden usar protocolos de administración remota como [SSH](https://en.wikipedia.org/wiki/Secure_Shell), [RDP](https://support.microsoft.com/kb/186607) y de [comunicación remota de PowerShell](/powershell/module/microsoft.powershell.core/enable-psremoting). En general, se recomienda no habilitar el acceso remoto directo a máquinas virtuales desde Internet.

Si es posible, siga otros enfoques, como usar redes privadas virtuales en una red virtual de Azure. Si no hay otros métodos disponibles, asegúrese de usar frases de contraseña complejas y la autenticación en dos fases (como [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)).

**Procedimiento recomendado**: use plataformas sólidas de autenticación y autorización.   
**Detalles**: Use identidades federadas en Azure AD en lugar de tiendas de usuarios personalizadas. Cuando se usan identidades federadas, se puede beneficiar de un enfoque basado en plataformas y delegar la administración de identidades autorizadas en sus asociados. Un enfoque federado resulta especialmente importante cuando los empleados terminan los contratos y dicha información necesita reflejarse a través de varios sistemas de identidad y autorización.

Use los mecanismos de autenticación y autorización proporcionados en la plataforma en lugar del código personalizado. La razón es que desarrollar código de autenticación personalizado puede resultar un método propenso a errores. La mayoría de los desarrolladores no son expertos en seguridad y es poco probable que conozcan los detalles y los últimos desarrollos en términos de autenticación y autorización. El código comercial (por ejemplo, de Microsoft) suele revisarse de manera amplia a efectos de seguridad.

Use la autenticación en dos fases. La autenticación en dos fases es el estándar actual para la autenticación y autorización porque evita las vulnerabilidades de seguridad inherentes a tipos de nombres de usuario y contraseñas de autenticación. El acceso a las interfaces de administración de Azure (portal/PowerShell remoto) y a los servicios usados por el cliente debe diseñarse y configurarse para que use [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md).

Use los protocolos de autenticación estándar, como OAuth2 y Kerberos. Estos protocolos se han sometido a revisiones exhaustivas del mismo nivel y posiblemente se implementen como parte de las bibliotecas de la plataforma a efectos de autenticación y autorización.

## <a name="use-threat-modeling-during-application-design"></a>Usar el modelado de amenazas durante el diseño de la aplicación
El [ciclo de vida de desarrollo de seguridad](https://www.microsoft.com/en-us/sdl) de Microsoft especifica que los equipos deben llevar a cabo un proceso denominado modelado de amenazas durante la fase de diseño. Para que este proceso sea más sencillo, Microsoft ha creado [SDL Threat Modeling Tool](../develop/threat-modeling-tool.md). Al modelar el diseño de la aplicación y enumerar las amenazas de [STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy) en todos los límites de confianza, pueden detectarse errores de diseño desde el principio.

En esta tabla se enumeran las amenazas STRIDE y se incluyen algunos ejemplos de mitigación donde se usan características de Azure. Estas mitigaciones no funcionarán en todas las situaciones.

| Threat | Propiedad de seguridad | Posibles mitigaciones de la plataforma Azure |
| --- | --- | --- |
| Suplantación de identidad | Authentication | Necesita conexiones HTTPS. |
| Alteración de datos | Integridad | Valida certificados TLS/SSL. |
| Rechazo | No rechazo | Habilita opciones de [supervisión y diagnóstico](/azure/architecture/best-practices/monitoring) de Azure. |
| Divulgación de información | Confidencialidad | Cifra datos confidenciales en reposo mediante [certificados de servicio](/rest/api/appservice/certificates). |
| Denegación de servicio | Disponibilidad | Supervisa las métricas de rendimiento de las posibles condiciones de denegación de servicio. Implementa filtros de conexión. |
| Elevación de privilegios | Authorization | Usa [Privileged Identity Management](../../active-directory/privileged-identity-management/subscription-requirements.md). |

## <a name="develop-on-azure-app-service"></a>Desarrollar en Azure App Service
[Azure App Service](../../app-service/overview.md) es una oferta PaaS que permite crear aplicaciones web y móviles para cualquier plataforma o dispositivo y conectarse a datos en cualquier lugar, en la nube o en un entorno local. App Service incluye las funcionalidades web y móviles que anteriormente se ofrecían por separado como Azure Websites y Azure Mobile Services. También incluye nuevas funcionalidades para automatizar procesos empresariales y hospedar las API en la nube. Como único servicio integrado, App Service ofrece un amplio conjunto de funcionalidades para escenarios web, móviles y de integración.

Estas son algunos procedimientos recomendados para usar App Service.

**Procedimiento recomendado**: [autenticación mediante Azure Active Directory](../../app-service/overview-authentication-authorization.md).   
**Detalles**: App Service proporciona un servicio OAuth 2.0 para el proveedor de identidades. OAuth 2.0 se centra en la sencillez del desarrollador del cliente y ofrece flujos de autorización específicos de aplicaciones web, aplicaciones de escritorio y teléfonos móviles. Azure AD usa OAuth 2.0 para permitir la autorización del acceso a los dispositivos móviles y a las aplicaciones web.

**Procedimiento recomendado**: Restricción del acceso siguiendo los principios de seguridad de limitar el acceso a lo que se necesita saber y a los principios de seguridad con privilegios mínimos.   
**Detalles**: La restricción del acceso es fundamental para las organizaciones que deseen aplicar directivas de seguridad para el acceso a los datos. Puede usar Azure RBAC para asignar permisos a los usuarios, los grupos y las aplicaciones en un ámbito determinado. Vea [Introducción a la administración de acceso](../../role-based-access-control/overview.md) para aprender más sobre cómo conceder acceso a los usuarios a las aplicaciones.

**Procedimiento recomendado**: Protección de las claves.   
**Detalles**: Azure Key Vault ayuda a proteger las claves criptográficas y los secretos que usan los servicios y aplicaciones en la nube. Con Key Vault, puede cifrar claves y secretos (por ejemplo claves de autenticación, claves de cuenta de almacenamiento, claves de cifrado de datos, archivos .PFX y contraseñas) a través del uso de claves que están protegidas por módulos de seguridad de hardware (HSM). Para tener mayor seguridad, puede importar o generar las claves en HSM. Vea [Azure Key Vault](../../key-vault/general/overview.md) para más información. También puede utilizar Key Vault para administrar los certificados TLS con renovación automática.

**Procedimiento recomendado**: Restricción de las direcciones IP de origen entrantes.   
**Detalles**: [App Service Environment](../../app-service/environment/intro.md) tiene una característica de integración de la red virtual que ayuda a restringir las direcciones IP de origen entrantes mediante grupos de seguridad de red. Las redes virtuales permiten colocar recursos de Azure en una red que se pueda enrutar distinta de Internet y a la que se controla el acceso. Vea [Integración de su aplicación con una instancia de Azure Virtual Network](../../app-service/web-sites-integrate-with-vnet.md) para más información.

**Procedimiento recomendado**: supervise el estado de seguridad de los entornos de App Service.   
**Detalles**: use Azure Security Center para supervisar los entornos de App Service. Cuando Security Center identifica posibles vulnerabilidades de seguridad, crea [recomendaciones](../../security-center/asset-inventory.md) que lo guiarán por el proceso de configuración de los controles necesarios.

## <a name="azure-cloud-services"></a>Azure Cloud Services
[Azure Cloud Services](../../cloud-services/cloud-services-choose-me.md) es un ejemplo de PaaS. Al igual que Azure App Service, esta tecnología está diseñada para ser compatible con aplicaciones escalables, confiables y de funcionamiento asequible. En la misma manera que App Service se hospeda en máquinas virtuales (VM), así ocurre también con Azure Cloud Services. Sin embargo, tiene más control sobre las máquinas virtuales. Puede instalar su propio software en las máquinas virtuales que usan Azure Cloud Services y puede tener acceso a ellas remotamente.

## <a name="install-a-web-application-firewall"></a>Instalar un firewall de aplicaciones web
Las aplicaciones web son cada vez más los objetivos de ataques malintencionados que aprovechan vulnerabilidades comunes conocidas, como ataques por inyección de código SQL o ataques de scripts de sitios, por nombrar unos pocos. Impedir tales ataques en el código de aplicación puede ser un verdadero desafío y requerir tareas rigurosas de mantenimiento, aplicación de revisiones y supervisión en varias capas de la topología de aplicación. Un firewall de aplicaciones web centralizado facilita enormemente la administración de la seguridad y proporciona mayor protección a los administradores de la aplicación frente a amenazas o intrusiones. Las soluciones de WAF también pueden reaccionar más rápido ante una amenaza de la seguridad aplicando revisiones que aborden una vulnerabilidad conocida en una ubicación central en lugar de proteger cada una de las aplicaciones web por separado. Las puertas de enlace de aplicaciones existentes pueden transformarse rápidamente en puertas de enlace con un firewall de aplicaciones web habilitado.

[Firewall de aplicaciones web (WAF)](../../web-application-firewall/afds/afds-overview.md) es una característica de Application Gateway que proporciona a las aplicaciones una protección centralizada contra vulnerabilidades de seguridad comunes. Firewall de aplicaciones web se basa en las reglas contenidas en el [conjunto de reglas básicas de OWASP (Open Web Application Security Project)](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 o 2.2.9.

## <a name="monitor-the-performance-of-your-applications"></a>Supervisar el rendimiento de las aplicaciones
La supervisión es el acto de recopilar y analizar datos para determinar el rendimiento, el mantenimiento y la disponibilidad de su aplicación. Una estrategia de supervisión eficaz le ayuda a comprender el funcionamiento detallado de los componentes de la aplicación. También sirve para aumentar el tiempo de actividad, ya que le notifica de cuestiones críticas para que pueda resolverlas antes de que se conviertan en problemas. También permite detectar anomalías que podrían estar relacionados con la seguridad.

Use [Azure Application Insights](https://azure.microsoft.com/documentation/services/application-insights) para supervisar la disponibilidad, el rendimiento y el uso de la aplicación, tanto si se hospeda en la nube como en un entorno local. Con Application Insights, podrá identificar y diagnosticar rápidamente errores en la aplicación sin tener que esperar a que un usuario informe de ellos. Con la información que recopile, puede tomar decisiones informadas sobre el mantenimiento y las mejoras de la aplicación.

Application Insights tiene numerosas herramientas para interactuar con los datos que recopila. Application Insights almacena sus datos en un repositorio común. Puede sacar partido a las funciones compartidas, como alertas, paneles y análisis detallados con el lenguaje de consulta de Kusto.

## <a name="perform-security-penetration-testing"></a>Realizar pruebas de penetración de seguridad
Validar defensas de seguridad es tan importante como probar cualquier otra característica. Convierta las [pruebas de penetración](pen-testing.md) en una parte estándar del proceso de compilación e implementación. Programe pruebas de seguridad y exámenes de vulnerabilidades periódicos en las aplicaciones, y supervise si hay puertos abiertos, ataques y puntos de conexión.

Las pruebas de vulnerabilidad ante datos aleatorios o inesperados son un método para buscar errores de programa (errores de código), proporcionando los datos de entrada con formato incorrecto para interfaces de programación (puntos de entrada) que analizan y consumen estos datos. [Detección de riesgos de seguridad de Microsoft](https://www.microsoft.com/en-us/security-risk-detection/) es una herramienta en la nube que puede usar para buscar errores y otras vulnerabilidades de seguridad en el software antes de implementarlo en Azure. La herramienta está diseñada para detectar vulnerabilidades antes de implementar software para que no tenga que aplicar revisiones a un error, controlar bloqueos o responder a un ataque después del lanzamiento del software.

## <a name="next-steps"></a>Pasos siguientes
Este artículo se centra en las ventajas de seguridad que ofrece una implementación de PaaS de Azure y los procedimientos de seguridad recomendados para aplicaciones en la nube. Después, conocerá los procedimientos recomendados para proteger las soluciones móviles y web de PaaS mediante determinados servicios de Azure. Empezaremos con Azure App Service, Azure SQL Database, Azure Synapse Analytics, Azure Storage y Azure Cloud Services. A medida que estén disponibles los artículos sobre procedimientos recomendados para otros servicios de Azure, se proporcionarán los vínculos correspondientes en la lista siguiente:

- [Azure App Service](paas-applications-using-app-services.md)
- [Azure SQL Database y Azure Synapse Analytics](paas-applications-using-sql.md)
- [Almacenamiento de Azure](paas-applications-using-storage.md)
- [Azure Cloud Services](../../cloud-services/security-baseline.md)
- Azure Cache for Redis
- Azure Service Bus
- Firewall de aplicaciones web

Vea [Desarrollo de aplicaciones seguras en Azure](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/) para profundizar en los controles y las preguntas de seguridad que se deben considerar en cada fase del ciclo de vida de desarrollo de software al desarrollar aplicaciones para la nube.

Vea [Patrones y procedimientos recomendados de seguridad en Azure](best-practices-and-patterns.md) para obtener más procedimientos recomendados de seguridad que pueda aplicar cuando diseñe, implemente y administre las soluciones en la nube mediante Azure.

En los siguientes recursos se ofrece más información general sobre la seguridad de Azure y los servicios de Microsoft relacionados:

- [Blog del equipo de seguridad de Azure](/archive/blogs/azuresecurity/): ofrece información actualizada sobre lo último en seguridad de Azure
- [Microsoft Security Response Center](https://technet.microsoft.com/library/dn440717.aspx): aquí podrá notificar vulnerabilidades de seguridad de Microsoft, incluidos problemas con Azure, o también mediante correo electrónico a secure@microsoft.com.