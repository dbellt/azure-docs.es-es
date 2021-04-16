---
title: Uso de Azure API Management con microservicios implementados en Azure Kubernetes Service | Microsoft Docs
description: En este artículo se describen las opciones de implementación de API Management con AKS
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 12/14/2019
ms.author: apimpm
ms.openlocfilehash: 36dfc8c906c52c6822e583db3a08c891306f7e78
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105047939"
---
# <a name="use-azure-api-management-with-microservices-deployed-in-azure-kubernetes-service"></a>Uso de Azure API Management con microservicios implementados en Azure Kubernetes Service

Los microservicios son idóneos para crear interfaces de programación de aplicaciones. Con [Azure Kubernetes Service](https://azure.microsoft.com/services/kubernetes-service/) (AKS), puede implementar y operar rápidamente una [arquitectura basada en microservicios](/azure/architecture/guide/architecture-styles/microservices) en la nube. Después, puede aprovechar [Azure API Management](https://aka.ms/apimrocks) (API Management) para publicar sus microservicios como interfaces de programación de aplicaciones para consumo interno y externo. En este artículo se describen las opciones de implementación de API Management con AKS. Se presupone conocimientos básicos de las redes de Kubernetes, API Management y Azure. 

## <a name="background"></a>Información previa

Al publicar microservicios como interfaces de programación de aplicaciones para su consumo, puede ser difícil administrar la comunicación entre los microservicios y los clientes que los consumen. Hay multitud de cuestiones transversales tales como la autenticación, la autorización, la limitación, el almacenamiento en caché, la transformación y la supervisión. Estos problemas son válidos independientemente de si los microservicios se exponen a clientes internos o externos. 

El patrón de [puerta de enlace de API](/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern) aborda estos problemas. Una puerta de enlace de API actúa como puerta de entrada de los microservicios, desacopla los clientes de los microservicios, agrega una capa de seguridad adicional y reduce la complejidad de los microservicios al eliminar la carga que supone el control de las cuestiones transversales. 

[Azure API Management](https://aka.ms/apimrocks) es una solución llave en mano que permite resolver las necesidades de la puerta de enlace de API. Puede crear rápidamente una puerta de enlace coherente y moderna para los microservicios y publicarlos como interfaces de programación de aplicaciones. Como solución de administración de API de ciclo de vida completo, también proporciona funcionalidades adicionales como, por ejemplo, un portal para desarrolladores de autoservicio para la detección de API, la administración del ciclo de vida de API y el análisis de API.

Cuando se usan conjuntamente, AKS y API Management ofrecen una plataforma para implementar, publicar, proteger, supervisar y administrar las interfaces de programación de aplicaciones basadas en microservicios. En este artículo, repasaremos algunas opciones de implementación de AKS junto con API Management. 

## <a name="kubernetes-services-and-apis"></a>Kubernetes Services e interfaces de programación de aplicaciones

En un clúster de Kubernetes, los contenedores se implementan en [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), que son efímeros y tienen un ciclo de vida. Cuando se elimina un nodo de trabajo, se pierden los pods que se ejecutan en el nodo. Por lo tanto, la dirección IP de un pod puede cambiar en cualquier momento. No podemos confiar en ella para comunicarse con el pod. 

Para solucionar este problema, Kubernetes presentó el concepto de [Services](https://kubernetes.io/docs/concepts/services-networking/service/). Una instancia de Kubernetes Service es una capa de abstracción que define un grupo lógico de pods y permite la exposición del tráfico externo, el equilibrio de carga y la detección de servicios para esos pods. 

Cuando estamos preparados para publicar nuestros microservicios como interfaces de programación de aplicaciones a través de API Management, debemos pensar en cómo asignar nuestros servicios en Kubernetes a las API de API Management. No hay reglas establecidas. Depende de cómo haya diseñado y particionado sus capacidades o dominios empresariales en microservicios al principio. Por ejemplo, si los pods detrás de un servicio son responsables de todas las operaciones en un recurso determinado (por ejemplo, el cliente), el servicio se puede asignar a una API. Si las operaciones de un recurso se particionan en varios microservicios (por ejemplo, GetOrder, PlaceOrder), es posible que varios servicios se agreguen lógicamente en una única API de API Management (vea la figura 1). 

También se pueden desarrollar las asignaciones. Como API Management crea una fachada delante de los microservicios, nos permite refactorizar y ajustar el tamaño de nuestros microservicios a lo largo del tiempo. 

![Asignación de servicios a interfaces de programación de aplicaciones](./media/api-management-aks/service-api-mapping.png)

## <a name="deploy-api-management-in-front-of-aks"></a>Implementación de API Management delante de AKS

Existen algunas opciones de implementación de API Management delante de un clúster de AKS. 

Aunque un clúster de AKS se implementa siempre en una red virtual, no hay que implementar una instancia de API Management en una red virtual. Cuando API Management no reside en la red virtual del clúster, el clúster de AKS tiene que publicar puntos de conexión públicos a los que API Management se conecte. En ese caso, es necesario proteger la conexión entre API Management y AKS. En otras palabras, debemos asegurarnos de que al clúster solo se pueda acceder exclusivamente a través de API Management. Vamos a analizar estas opciones. 

### <a name="option-1-expose-services-publicly"></a>Opción 1: Exposición pública de los servicios

Los servicios de un clúster de AKS se pueden exponer públicamente con [tipos de servicio](../aks/concepts-network.md) de NodePort, LoadBalancer o ExternalName. En este caso, los servicios son accesibles directamente desde Internet público. Después de implementar API Management delante del clúster, hay que asegurarse de que todo el tráfico de entrada pase por API Management aplicando la autenticación en los microservicios. Por ejemplo, API Management puede incluir un token de acceso en cada solicitud realizada al clúster. Cada microservicio es responsable de validar el token antes de procesar la solicitud. 


Esta podría ser la opción más sencilla de implementar API Management delante de AKS, especialmente si ya tiene lógica de autenticación implementada en sus microservicios. 

![Publicación directa de los servicios](./media/api-management-aks/direct.png)

Ventajas:
* Fácil configuración en el lado de API Management porque no hay que insertarla en la red virtual del clúster
* No se produce ningún cambio en el lado de AKS si los servicios ya se exponen públicamente y la lógica de autenticación ya existe en microservicios

Inconvenientes:
* Posible riesgo de seguridad debido a la visibilidad pública de los puntos de conexión de servicio
* No hay un punto de entrada único para el tráfico de entrada del clúster
* Complica los microservicios con lógica de autenticación duplicada

### <a name="option-2-install-an-ingress-controller"></a>Opción 2: Instalación de un controlador de entrada

Aunque la opción 1 podría ser más fácil, tiene notables inconvenientes como se ha mencionado anteriormente. Si una instancia de API Management no reside en la red virtual del clúster, la autenticación Mutual TLS (mTLS) es una manera robusta de garantizar que el tráfico sea seguro y de confianza en ambas direcciones entre una instancia de API Management y un clúster de AKS. 

La autenticación Mutual TLS es [compatible de forma nativa](./api-management-howto-mutual-certificates.md) con API Management y se puede habilitar en Kubernetes mediante la [instalación de un controlador de entrada](../aks/ingress-own-tls.md) (figura 3). Como resultado, la autenticación se realizará en el controlador de entrada, lo que simplifica los microservicios. Además, puede agregar las direcciones IP de API Management a la lista de permitidos por entrada para asegurarse de que solo API Management tiene acceso al clúster.  

 
![Publicación a través de un controlador de entrada](./media/api-management-aks/ingress-controller.png)


Ventajas:
* Fácil configuración en el lado de API Management porque no hay que insertarla en la red virtual del clúster y mTLS es compatible de forma nativa
* Centraliza la protección del tráfico de clústeres de entrada en la capa de controlador de entrada
* Reduce el riesgo de seguridad al minimizar los puntos de conexión de clúster visibles públicamente

Inconvenientes:
* Aumenta la complejidad de la configuración del clúster debido al trabajo adicional de instalar, configurar y mantener el controlador de entrada y administrar los certificados usados con mTLS
* Riesgo de seguridad debido a la visibilidad pública de los puntos de conexión del controlador de entrada


Al publicar las API mediante API Management, es sencillo y frecuente proteger el acceso a ellas mediante claves de suscripción. Los desarrolladores que necesiten usar las API publicadas deben incluir una clave de suscripción válida en las solicitudes HTTP al realizar llamadas a esas API. De lo contrario, la puerta de enlace de API Management rechaza las llamadas inmediatamente. No se reenvían a los servicios back-end.

Para obtener una clave de suscripción para acceder a las API se necesita una suscripción. Una suscripción es básicamente un contenedor con nombre para un par de claves de suscripción. Los desarrolladores que necesiten usar las API publicadas pueden obtener suscripciones. Y no necesitan aprobación de los publicadores de API. Los publicadores de API también pueden crear suscripciones directamente para los consumidores de API.

### <a name="option-3-deploy-apim-inside-the-cluster-vnet"></a>Opción 3: Implementación de APIM dentro de la red virtual del clúster

En algunos casos, es posible que a los clientes con restricciones normativas o estrictos requisitos de seguridad les parezca que las opciones 1 y 2 no son soluciones viables debido a los puntos finales expuestos públicamente. En otros, el clúster de AKS y las aplicaciones que consumen los microservicios pueden residir en la misma red virtual; por lo tanto, no hay ninguna razón para exponer el clúster de forma pública, ya que todo el tráfico de la API permanecerá dentro de la red virtual. En estos casos, puede implementar API Management en la red virtual del clúster. Los [niveles de desarrollador y prémium de API Management](https://aka.ms/apimpricing) admiten la implementación de red virtual. 

Hay dos modos de [implementación de API Management en una red virtual](./api-management-using-with-vnet.md): externo e interno. 

Si los consumidores de la API no residen en la red virtual del clúster, ha de usarse el modo externo (figura 4). En este modo, la puerta de enlace de API Management se inserta en la red virtual del clúster, pero es accesible desde Internet público a través de un equilibrador de carga externo. Ayuda a ocultar por completo el clúster mientras sigue permitiendo a los clientes externos consumir los microservicios. Además, puede usar funcionalidades de red de Azure tales como los grupos de seguridad de red (NSG), para restringir el tráfico.

![Modo de red virtual externa](./media/api-management-aks/vnet-external.png)

Si todos los consumidores de la API residen dentro de la red virtual del clúster, se podría usar el modo interno (figura 5). En este modo, la puerta de enlace de API Management se inserta en la red virtual del clúster y solo es accesible desde el interior de esta red virtual a través de un equilibrador de carga interno. No hay forma de llegar a la puerta de enlace de API Management o al clúster de AKS desde Internet público. 

![Modo de red virtual interna](./media/api-management-aks/vnet-internal.png)

 En ambos casos, el clúster de AKS no es visible públicamente. A diferencia de la opción 2, es posible que el controlador de entrada no sea necesario. Según el escenario y la configuración, es posible que se requiera autenticación entre API Management y los microservicios. Por ejemplo, si se adopta una malla de servicio, siempre se requiere la autenticación Mutual TLS. 

Ventajas:
* Se trata de la opción más segura porque el clúster de AKS no tiene ningún punto de conexión público
* Simplifica la configuración del clúster porque no tiene ningún punto de conexión público
* Posibilidad de ocultar API Management y AKS dentro de la red virtual mediante el modo interno
* Posibilidad de controlar el tráfico de red con funcionalidades de red de Azure tales como los grupos de seguridad de red (NSG)

Inconvenientes:
* Complica la implementación y configuración de API Management para trabajar dentro de la red virtual

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [Conceptos de red para aplicaciones en AKS](../aks/concepts-network.md)
* Más información sobre el [uso de API Management con redes virtuales](./api-management-using-with-vnet.md)
