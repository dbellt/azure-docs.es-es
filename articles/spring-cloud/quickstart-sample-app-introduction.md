---
title: 'Inicio rápido: Introducción a la aplicación de ejemplo (Azure Spring Cloud)'
description: Describe la aplicación de ejemplo que se usa en esta serie de guías de inicio rápido para la implementación en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 6c1837b6992090f1b02d89720db298fe5714d4c3
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108286514"
---
# <a name="introduction-to-the-sample-app"></a>Introducción a la aplicación de ejemplo

::: zone pivot="programming-language-csharp"
En esta serie de guías de inicio rápido se usa una aplicación de ejemplo compuesta por dos microservicios para mostrar cómo implementar una aplicación .NET Core de Steeltoe en el servicio Azure Spring Cloud. Usará las funcionalidades de Azure Spring Cloud, como detección de servicios, servidor de configuración, registros, métricas y seguimiento distribuido.

## <a name="functional-services"></a>Servicios funcionales

La aplicación de ejemplo se compone de dos microservicios:

* El servicio `planet-weather-provider` devuelve el texto meteorológico en respuesta a una solicitud HTTP que especifica el nombre del planeta. Por ejemplo, puede devolver "muy cálido" para el planeta Mercurio. Obtiene los datos meteorológicos del servidor de configuración. El servidor de configuración obtiene los datos meteorológicos de un archivo YAML en un repositorio Git, por ejemplo:

  ```yaml
  MercuryWeather: very warm
  VenusWeather: quite unpleasant
  MarsWeather: very cool
  SaturnWeather: a little bit sandy
  ```

* El servicio `solar-system-weather` devuelve datos de cuatro planetas en respuesta a una solicitud HTTP. Obtiene los datos de realizar cuatro solicitudes HTTP a `planet-weather-provider`. Usa el servicio de detección de servidores Eureka para llamar a `planet-weather-provider`. Devuelve un archivo JSON, por ejemplo:

  ```json
  [{
      "Key": "Mercury",
      "Value": "very warm"
  }, {
      "Key": "Venus",
      "Value": "quite unpleasant"
  }, {
      "Key": "Mars",
      "Value": "very cool"
  }, {
      "Key": "Saturn",
      "Value": "a little bit sandy"
  }]
  ```

En el diagrama siguiente se muestra la arquitectura de la aplicación de ejemplo:

:::image type="content" source="media/spring-cloud-quickstart-sample-app-introduction/sample-app-diagram.png" alt-text="Diagrama de la aplicación de ejemplo":::

## <a name="code-repository"></a>Repositorio de código

La aplicación de ejemplo se encuentra en la carpeta [steeltoe-sample](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample) del repositorio Azure-Samples/Azure-Spring-Cloud-Samples en GitHub.

Las instrucciones de las siguientes guías de inicio rápido hacen referencia al código fuente según sea necesario.

::: zone-end

::: zone pivot="programming-language-java"
En este inicio rápido, vamos a usar la versión de microservicios de la ya conocida aplicación de ejemplo [PetClinic](https://github.com/spring-petclinic/spring-petclinic-microservices), que le mostrará cómo implementar aplicaciones en el servicio Azure Spring Cloud. **PetClinic** muestra el patrón de arquitectura de microservicios y resalta el desglose de los servicios. Verá cómo se implementan los servicios en Azure con las funcionalidades de Azure Spring Cloud como la detección de servicios, el servidor de configuración, los registros, las métricas, el seguimiento distribuido y el soporte técnico para herramientas sencillo de usar para los desarrolladores. 

Para seguir los ejemplos de implementación de Azure Spring Cloud, solo necesita la ubicación del código fuente, que se proporciona si es necesario.

![Arquitectura de PetClinic](media/build-and-deploy/microservices-architecture-diagram.jpg)

## <a name="functional-services-to-be-deployed"></a>Servicios funcionales que se implementarán

PetClinic se descompone en 4 microservicios principales. y todos ellos son aplicaciones que se pueden implementar de forma independiente y organizadas por dominios empresariales.

* **Customers service:** contiene la lógica de entrada del usuario general y la validación, incluida la información sobre mascotas y sus propietarios (nombre, dirección, ciudad, teléfono).
* **Visits service:** almacena y muestra la información sobre las visitas en la información de cada mascota.
* **Vets service:** almacena y muestra la información sobre los veterinarios, incluidos sus nombres y especialidades.
* **API Gateway**: API Gateway es un punto de entrada único en el sistema que se usa para controlar las solicitudes y enrutarlas al servicio adecuado, o para invocar varios servicios y agregar los resultados.  los tres servicios principales exponen una API externa al cliente. En los sistemas reales, el número de funciones puede aumentar rápidamente con la complejidad del sistema. Cuando se representa una página web compleja puede haber cientos de servicios implicados. 

## <a name="infrastructure-services-hosted-by-azure-spring-cloud"></a>Servicios de infraestructura que hospeda Azure Spring Cloud

En los sistemas distribuidos hay varios patrones comunes que facilitan el funcionamiento de los principales servicios. Azure Spring Cloud proporciona herramientas que permiten mejorar las aplicaciones de Spring Boot para implementar los siguientes patrones: 

* **Servicio de configuración**: Azure Spring Cloud Config es un servicio de configuración centralizado escalable horizontalmente para sistemas distribuidos. Usa un repositorio conectable que actualmente admite el almacenamiento local, Git y Subversion.
* **Detección de servicios**: permite la detección automática de ubicaciones de red para las instancias de servicio que podrían tener direcciones asignadas dinámicamente debido al escalado automático, los errores y las actualizaciones.

## <a name="database-configuration"></a>Configuración de la base de datos
En su configuración predeterminada, **PetClinic** usa una base de datos en memoria (HSQLDB) que se rellena en el inicio con datos. Se proporciona una configuración similar para MySql si se necesita una configuración de base de datos persistente. La dependencia de Connector/J, el controlador JDBC de MySQL, ya está incluida en los archivos pom.xml.

## <a name="sample-usage-of-petclinic"></a>Ejemplo de uso de PetClinic

Para obtener detalles completos de la implementación, consulte nuestra bifurcación de [PetClinic](https://github.com/Azure-Samples/spring-petclinic-microservices). Los ejemplos hacen referencia al código fuente cuando se necesita.

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aprovisionamiento de una instancia de Azure Spring Cloud](./quickstart-provision-service-instance.md)