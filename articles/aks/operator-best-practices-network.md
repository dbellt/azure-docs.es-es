---
title: Procedimientos recomendados de seguridad de la red
titleSuffix: Azure Kubernetes Service
description: Procedimientos recomendados con los recursos de red virtual y la conectividad para el operador del clúster en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: conceptual
ms.date: 03/10/2021
ms.openlocfilehash: 1e0212766e7d5443664d57a97cfa9ea9d0035da3
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107104956"
---
# <a name="best-practices-for-network-connectivity-and-security-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados con la conectividad de red y la seguridad en Azure Kubernetes Service (AKS)

Al crear y administrar clústeres en Azure Kubernetes Service (AKS), se proporciona conectividad de red para los nodos y las aplicaciones. Estos recursos de red incluyen los intervalos de direcciones IP, los equilibradores de carga y los controladores de entrada. Para mantener un servicio de alta calidad para las aplicaciones, debe elaborar estrategias y configurar estos recursos.

Este artículo de procedimientos recomendados se centra en la conectividad de red y la seguridad para los operadores del clúster. En este artículo aprenderá a:

> [!div class="checklist"]
> * Comparar los modos de red kubenet y Azure Container Networking Interface (CNI) en AKS.
> * Planear la conectividad y el direccionamiento IP necesarios.
> * Distribuir el tráfico mediante equilibradores de carga, controladores de entrada o un firewall de aplicaciones web (WAF).
> * Conectarse de forma segura a los nodos del clúster.

## <a name="choose-the-appropriate-network-model"></a>Elección del modelo de red adecuado

> **Guía de procedimientos recomendados** 
> 
> Use las redes de Azure CNI en AKS para la integración con redes virtuales existentes o redes locales. Este modelo de red permite mayor separación de los recursos y los controles en un entorno empresarial.

Las redes virtuales proporcionan conectividad básica para que los nodos de AKS y los clientes accedan a las aplicaciones. Hay dos maneras diferentes de implementar los clústeres de AKS en redes virtuales:

* **Redes de CNI de Azure**

    Se implementan en una red virtual y usan el complemento [Azure CNI][cni-networking] de Kubernetes. Los pods reciben direcciones IP individuales que se pueden enrutar a otros servicios de red o recursos locales.
* **Redes de Kubenet**

    Azure administra los recursos de red virtual cuando el clúster se implementa y utiliza el complemento [kubenet][kubenet] de Kubernetes.


En las implementaciones de producción, tanto kubenet como Azure CNI son opciones válidas.

### <a name="cni-networking"></a>Redes de CNI

Azure CNI es un protocolo independiente del proveedor que permite que el entorno de ejecución del contenedor realice solicitudes a un proveedor de red. Asigna direcciones IP a los nodos y los pods, y proporciona características de administración de direcciones IP (IPAM) cuando se conecta a redes virtuales de Azure existentes. Cada recurso de nodo y pod recibe una dirección IP en la red virtual de Azure y no se necesita más enrutamiento para la comunicación con otros servicios o recursos.

![Diagrama que muestra dos nodos con puentes que conectan cada uno a una única red virtual de Azure](media/operator-best-practices-network/advanced-networking-diagram.png)

En particular, las redes de Azure CNI para producción permiten separar el control y la administración de los recursos. Desde una perspectiva de seguridad, se suele preferir que distintos equipos administren y protejan los recursos. Con las redes de Azure CNI, puede conectarse a recursos de Azure existentes, recursos locales u otros servicios mediante direcciones IP asignadas a cada pod.

Al usar redes de Azure CNI, el recurso de red virtual se encuentra en un grupo de recursos independiente del clúster de AKS. Delegue permisos en la identidad de clúster de AKS para acceder y administrar estos recursos. La identidad de clúster que usa el clúster de AKS debe tener como mínimo permisos de [Colaborador de la red](../role-based-access-control/built-in-roles.md#network-contributor) en la subred de la red virtual. 

Si quiere definir un [rol personalizado](../role-based-access-control/custom-roles.md) en lugar de usar el rol integrado de colaborador de red, se requieren los permisos siguientes:
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

De manera predeterminada, AKS utiliza una identidad administrada para su identidad de clúster. Sin embargo, puede usar una entidad de servicio en su lugar. Para más información acerca de:
* la delegación en entidad de servicio de AKS, consulte [Delegación del acceso a otros recursos de Azure][sp-delegation]. 
* las identidades administradas, consulte [Uso de identidades administradas.](use-managed-identity.md)

Como cada pod y cada nodo recibe su propia dirección IP, planee los intervalos de direcciones para las subredes de AKS. Tenga en cuenta lo siguiente:
* La subred debe ser lo suficientemente grande como para proporcionar direcciones IP para cada nodo, pod y recurso de red que implemente. 
    * Con las redes de kubenet y de Azure CNI, cada nodo que se ejecuta tiene límites predeterminados del número de pods.
* Cada clúster de AKS se debe colocar en su propia subred. 
* Evite el uso de intervalos de direcciones IP que se superponen con los recursos de red existentes. 
    * Es necesario para permitir la conectividad a redes locales o emparejadas en Azure.
* Para controlar los eventos de escalado horizontal o las actualizaciones de clúster, es necesario que haya otras direcciones IP disponibles en la subred asignada. 
    * Este espacio de direcciones adicional es especialmente importante si usa contenedores de Windows Server, como aquellos grupos de nodos que requieren una actualización para aplicar las revisiones de seguridad más recientes. Para obtener más información sobre los nodos de Windows Server, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

Para calcular la dirección IP necesaria, consulte [Configuración de redes de Azure CNI en AKS][advanced-networking].

Cuando se crea un clúster con redes de Azure CNI, se especifican otros intervalos de direcciones para el clúster, como la dirección del puente de Docker, la IP del servicio DNS y el intervalo de direcciones de servicio. En general, asegúrese de que estos intervalos de direcciones:
* No se superponen entre sí.
* No se superponen con las redes asociadas al clúster, incluidas las redes virtuales, las subredes, las redes locales y las redes emparejadas. 

Para obtener detalles específicos sobre los límites y el tamaño de estos intervalos de direcciones, consulte [Configuración de redes de Azure CNI en AKS][advanced-networking].

### <a name="kubenet-networking"></a>Redes de kubenet

Aunque las redes de kubenet no requieren configuración de las redes virtuales antes de la implementación del clúster, existen algunos inconvenientes por la espera:

* Puesto que los nodos y pods se colocan en subredes IP diferentes, el enrutamiento definido por el usuario (UDR) y el reenvío IP enruta el tráfico entre pods y nodos. Este enrutamiento adicional puede reducir el rendimiento de las redes.
* Las conexiones a redes locales existentes o el emparejamiento con otras redes virtuales de Azure pueden ser complejos.

Dado que no crea la red virtual y las subredes por separado del clúster de AKS, Kubenet es ideal para:
* Cargas de trabajo pequeñas de desarrollo o pruebas. 
* Sitios web sencillos con poco tráfico.
* Elevación y desplazamiento de cargas de trabajo a contenedores.

Para la mayoría de las implementaciones de producción, debe planificar y usar redes de Azure CNI.

También puede [configurar sus propios intervalos de direcciones IP y redes virtuales con kubenet][aks-configure-kubenet-networking]. De forma similar a las redes de Azure CNI, estos intervalos de direcciones no deben superponerse entre sí y no deben superponerse con las redes asociadas al clúster (las redes virtuales, las subredes, las redes locales y las redes emparejadas). 

Para obtener detalles específicos sobre los límites y el tamaño de estos intervalos de direcciones, consulte [Uso de redes kubenet con sus propios intervalos de direcciones IP en AKS][aks-configure-kubenet-networking].

## <a name="distribute-ingress-traffic"></a>Distribución del tráfico de entrada

> **Guía de procedimientos recomendados** 
> 
> Para distribuir el tráfico HTTP o HTTPS a las aplicaciones, use los recursos y los controladores de entrada. En comparación con el equilibrador de carga de Azure, los controladores de entrada proporcionan características adicionales y se pueden administrar como recursos nativos de Kubernetes.

Aunque un equilibrador de carga de Azure puede distribuir el tráfico de cliente a las aplicaciones en el clúster de AKS, su conocimiento sobre ese tráfico es limitado. Un recurso de equilibrador de carga funciona en la capa 4 y distribuye el tráfico en función de protocolos o puertos. 

La mayoría de las aplicaciones web que usan HTTP o HTTPS deben usar recursos y controladores de entrada de Kubernetes, que funcionan en la capa 7. En la entrada se puede distribuir el tráfico en función de la dirección URL de la aplicación y administrar la terminación TLS/SSL. La entrada también reduce el número de direcciones IP que se exponen y se asignan. 

Con un equilibrador de carga, cada aplicación normalmente necesita una dirección IP pública asignada y a ella y al servicio del clúster de AKS. Con un recurso de entrada, una única dirección IP puede distribuir tráfico a varias aplicaciones.

![Diagrama que muestra el flujo de tráfico de entrada en un clúster de AKS](media/operator-best-practices-network/aks-ingress.png)

 En la entrada existen dos componentes:

 * un *recurso* de entrada
 * un *controlador* de entrada

### <a name="ingress-resource"></a>Recurso de entrada

El *recurso de entrada* es un manifiesto YAML de `kind: Ingress`. Define el host, los certificados y las reglas para enrutar el tráfico a los servicios que se ejecutan en el clúster de AKS. 

El siguiente ejemplo de manifiesto de YAML distribuye el tráfico de *myapp.com* a uno de los dos servicios, *blogservice* o *storeservice*. El cliente se dirige a un servicio o al otro en función de la dirección URL a la que acceden.

```yaml
kind: Ingress
metadata:
 name: myapp-ingress
   annotations: kubernetes.io/ingress.class: "PublicIngress"
spec:
 tls:
 - hosts:
   - myapp.com
   secretName: myapp-secret
 rules:
   - host: myapp.com
     http:
      paths:
      - path: /blog
        backend:
         serviceName: blogservice
         servicePort: 80
      - path: /store
        backend:
         serviceName: storeservice
         servicePort: 80
```

### <a name="ingress-controller"></a>Controlador de entrada

Un *controlador de entrada* es un demonio que se ejecuta en un nodo de AKS y supervisa las solicitudes entrantes. A continuación, el tráfico se distribuye según las reglas definidas en el recurso de entrada. Aunque el controlador de entrada más común se basa en [NGINX], AKS no le restringe a un controlador específico. Puede usar [Contour][contour], [HAProxy][haproxy], [Traefik][traefik], etc.

El controlador de entrada también debe programarse en un nodo de Linux. Indique que el recurso debe ejecutarse en un nodo basado en Linux mediante un selector de nodo en el manifiesto YAML o una implementación de gráfico de Helm. Para obtener más información, consulte [Uso de selectores de nodo para controlar dónde se programan los pods en AKS][concepts-node-selectors].

> [!NOTE]
> Los nodos de Windows Server no deben ejecutar el controlador de entrada.

Hay muchos escenarios de entrada, incluidas las siguientes guías paso a paso:

* [Creación de un controlador de entrada básico con conectividad de red externa][aks-ingress-basic]
* [Creación de un controlador de entrada que use una red privada interna y una dirección IP][aks-ingress-internal]
* [Crear un controlador de entrada que usa sus propios certificados TLS][aks-ingress-own-tls]
* Creación de un controlador de entrada que use Let's Encrypt para generar certificados TLS de forma automática [con una dirección IP pública dinámica][aks-ingress-tls] o [con una dirección IP pública estática][aks-ingress-static-tls]

## <a name="secure-traffic-with-a-web-application-firewall-waf"></a>Protección del tráfico con un firewall de aplicaciones web (WAF)

> **Guía de procedimientos recomendados**
> 
> Para analizar el tráfico entrante en busca de posibles ataques, use un firewall de aplicaciones web (WAF) como [Barracuda WAF para Azure][barracuda-waf] o Azure Application Gateway. Estos recursos de red más avanzados también pueden enrutar el tráfico más allá de las conexiones HTTP y HTTPS sencillas o la terminación TLS básica.

Normalmente, un controlador de entrada es un recurso de Kubernetes en el clúster de AKS que distribuye el tráfico a servicios y aplicaciones. El controlador se ejecuta como demonio en un nodo de AKS y consume algunos de los recursos del nodo, como la CPU, la memoria y el ancho de banda de red. En entornos mayores, se prefiere:
* Descargar algo de tráfico mediante el enrutamiento o la terminación TLS a un recurso de red externo al clúster de AKS.
* Analizar el tráfico entrante en busca de posibles ataques.

![Un firewall de aplicaciones web (WAF), como Azure Application Gateway, puede proteger y distribuir el tráfico del clúster de AKS.](media/operator-best-practices-network/web-application-firewall-app-gateway.png)

Para esa capa adicional de seguridad, un firewall de aplicaciones web (WAF) filtra el tráfico entrante. Con un conjunto de reglas, Open Web Application Security Project (OWASP) detecta ataques, como el scripting entre sitios o el envenenamiento de cookies. [Azure Application Gateway][app-gateway] (actualmente en versión preliminar en AKS) es una solución WAF que se integra con los clústeres de AKS, lo que bloquea estas características de seguridad antes de que el tráfico llegue al clúster y a las aplicaciones de AKS. 

Dado que otras soluciones de terceros también realizan estas funciones, puede seguir usando las inversiones existentes o la experiencia en su producto preferido.

Los recursos de entrada o el equilibrador de carga continúan ejecutándose en el clúster de AKS y mejoran la distribución del tráfico. Application Gateway se puede administrar de manera centralizada como controlador de entrada con definición de recurso. Para empezar, [cree un controlador de entrada de Application Gateway][app-gateway-ingress].

## <a name="control-traffic-flow-with-network-policies"></a>Control del flujo de tráfico con directivas de red

> **Guía de procedimientos recomendados** 
>
> Use directivas de red para permitir o denegar el tráfico a los pods. De forma predeterminada, se permite todo el tráfico entre los pods dentro de un clúster. Para mejorar la seguridad, defina reglas que limiten la comunicación del pod.

La directiva de red es una característica de Kubernetes disponible en AKS que permite controlar el flujo de tráfico entre pods. Según la configuración se permite o deniega el tráfico al pod, como, por ejemplo, etiquetas asignadas, espacio de nombres o puerto de tráfico. Las directivas de red son una manera nativa de nube para controlar el flujo de tráfico de los pods. Como los pods se crean dinámicamente en un clúster de AKS, se pueden aplicar automáticamente las directivas de red necesarias.

Para usar la directiva de red, habilite la característica al crear un clúster de AKS. No se puede habilitar la directiva de red en un clúster de AKS existente. Planee con antelación la habilitación de la directiva de red en los clústeres necesarios. 

>[!NOTE]
>La directiva de red solo se debe usar para los pods y los nodos basados en Linux de AKS.

Se crea una directiva de red como un recurso de Kubernetes mediante un manifiesto YAML. Las directivas se aplican a pods definidos, con reglas de entrada o salida que definen el flujo de tráfico. 

El ejemplo siguiente aplica una directiva de red a los pods que tienen la etiqueta *app: backend*. La regla de entrada solo permite el tráfico desde los pods con la etiqueta *app: frontend*:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

Para empezar a usar directivas, consulte [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)][use-network-policies].

## <a name="securely-connect-to-nodes-through-a-bastion-host"></a>Conexión segura a los nodos mediante una pasarela de aplicaciones

> **Guía de procedimientos recomendados** 
>
> No exponga la conectividad remota a los nodos de AKS. Cree una pasarela de aplicaciones, o host jump, en una red virtual de administración. Utilice la pasarela de aplicaciones para enrutar el tráfico de forma segura en el clúster de AKS para las tareas de administración remota.

La mayoría de las operaciones en AKS puede realizarse con las herramientas de administración de Azure o mediante el servidor de API de Kubernetes. Los nodos de AKS solo están disponibles en una red privada y no están conectados a la red pública de Internet. Para conectarse a los nodos y proporcionar mantenimiento y soporte técnico, enrute las conexiones a través de un host bastión o jumpbox. Compruebe que este host se encuentre en una red virtual de administración independiente emparejada de forma segura a la red virtual del clúster de AKS.

![Conexión a los nodos de AKS mediante una pasarela de aplicaciones o host de salto](media/operator-best-practices-network/connect-using-bastion-host-simplified.png)

La red de administración para la pasarela de aplicaciones también debe protegerse. Use [Azure ExpressRoute][expressroute] o [VPN Gateway][vpn-gateway] para conectarse a una red local y controlar el acceso con grupos de seguridad de red.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en la conectividad de red y la seguridad. Para obtener más información sobre los conceptos básicos de red en Kubernetes, consulte [Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)][aks-concepts-network].

<!-- LINKS - External -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet
[app-gateway-ingress]: https://github.com/Azure/application-gateway-kubernetes-ingress
[nginx]: https://www.nginx.com/products/nginx/kubernetes-ingress-controller
[contour]: https://github.com/heptio/contour
[haproxy]: https://www.haproxy.org
[traefik]: https://github.com/containous/traefik
[barracuda-waf]: https://www.barracuda.com/products/webapplicationfirewall/models/5

<!-- INTERNAL LINKS -->
[aks-concepts-network]: concepts-network.md
[sp-delegation]: kubernetes-service-principal.md#delegate-access-to-other-azure-resources
[expressroute]: ../expressroute/expressroute-introduction.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-own-tls]: ingress-own-tls.md
[app-gateway]: ../application-gateway/overview.md
[use-network-policies]: use-network-policies.md
[advanced-networking]: configure-azure-cni.md
[aks-configure-kubenet-networking]: configure-kubenet.md
[concepts-node-selectors]: concepts-clusters-workloads.md#node-selectors
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool