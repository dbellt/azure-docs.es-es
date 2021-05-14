---
title: Protección de servicios web con TLS
titleSuffix: Azure Machine Learning
description: Más información sobre cómo habilitar HTTPS con la versión 1.2 de TLS para proteger un servicio web que se implementó a través de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: aashishb
author: aashishb
ms.date: 03/11/2021
ms.topic: how-to
ms.openlocfilehash: 9531862ffb62a92a3b9be33b38e4ecef97bf974e
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884667"
---
# <a name="use-tls-to-secure-a-web-service-through-azure-machine-learning"></a>Uso de TLS para proteger un servicio web con Azure Machine Learning


En este artículo se muestra cómo proteger un servicio web implementado con Azure Machine Learning.

Se usa [HTTPS](https://en.wikipedia.org/wiki/HTTPS) para restringir el acceso a los servicios web y proteger los datos que los clientes envían. HTTPS le ayuda a proteger las comunicaciones entre un cliente y un servicio web mediante el cifrado de las comunicaciones entre los dos. El cifrado usa [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS se conoce a veces todavía como *Capa de sockets seguros* (SSL), que fue su predecesor.

> [!TIP]
> El SDK de Azure Machine Learning usa el término "SSL" para las propiedades relacionadas con las comunicaciones seguras. Esto no significa que su servicio web no use *TLS*. SSL es simplemente un término más comúnmente reconocido.
>
> En concreto, los servicios web implementados a través de Azure Machine Learning admiten la versión 1.2 de TLS para AKS y ACI. En el caso de las implementaciones ACI, si está usando una versión anterior de TLS, se recomienda volver a realizar la implementación para obtener la versión más reciente de TLS.
>
> La versión 1.3 de TLS para la inferencia entre Azure Machine Learning y AKS no es compatible.

TLS y SSL dependen ambos de los *certificados digitales*, que ayudan con la comprobación de la identidad y el cifrado. Para obtener más información sobre cómo funcionan los certificados digitales, vea el tema de Wikipedia [Public key infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) (Infraestructura de clave pública).

> [!WARNING]
> Si no usa HTTPS para el servicio web, los datos que se envían hacia y desde el servicio podrían ser visibles para otros usuarios en Internet.
>
> HTTPS también permite al cliente comprobar la autenticidad del servidor al que se está conectando. Esta característica protege a los clientes contra ataques de tipo [Man in the middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

Este es el proceso general para proteger un servicio web:

1. Obtenga un nombre de dominio.

2. Obtenga un certificado digital.

3. Implemente o actualice el servicio web con el protocolo TLS habilitado.

4. Actualice el DNS para que apunte al servicio web.

> [!IMPORTANT]
> Si va a implementar en Azure Kubernetes Service (AKS), puede adquirir su propio certificado o usar un certificado proporcionado por Microsoft. Si usa un certificado de Microsoft, no es necesario obtener un nombre de dominio ni un certificado TLS/SSL. Para obtener más información, consulte la sección [Habilitación de TLS e implementación](#enable) de este artículo.

Hay ligeras diferencias al proteger los servicios web a través de los [destinos de implementación](how-to-deploy-and-where.md).

## <a name="get-a-domain-name"></a>Obtención de un nombre de dominio

Si todavía no tiene un nombre de dominio, compre uno en un *registrador de nombres de dominio*. El proceso y los precios difieren entre los registradores. El registrador proporciona herramientas para administrar el nombre de dominio. Estas herramientas se usan para asignar un nombre de dominio completo (como www\.contoso.com ) a la dirección IP que hospeda el servicio web.

## <a name="get-a-tlsssl-certificate"></a>Obtención de un certificado TLS/SSL

Hay muchas maneras de obtener un certificado TLS/SSL (certificado digital). La más común es comprar uno en una *entidad de certificación* (CA). Independientemente de dónde obtenga el certificado, se necesitan estos archivos:

* Un **certificado**. El certificado debe contener la cadena de certificados completa y debe estar codificado en PEM.
* Una **clave**. La clave debe estar codificada en PEM.

Cuando solicite un certificado, debe proporcionar el nombre de dominio completo (FQDN) de la dirección que pretende usar para el servicio web (por ejemplo, www\.contoso.com). La dirección que aparece en el certificado y la que usan los clientes se comparan para comprobar la identidad del servicio web. Si esas direcciones no coinciden, el cliente obtiene un mensaje de error.

> [!TIP]
> Si la entidad de certificación no puede proporcionar el certificado y la clave como archivos codificados en PEM, puede usar una utilidad como [OpenSSL](https://www.openssl.org/) para cambiar el formato.

> [!WARNING]
> Use certificados *autofirmados* solo para desarrollo. No deben usarse en entornos de producción. Los certificados autofirmados pueden provocar problemas en las aplicaciones cliente. Para más información, consulte la documentación de las bibliotecas de red utilizadas en la aplicación cliente.

## <a name="enable-tls-and-deploy"></a><a id="enable"></a> Habilitación de TLS e implementación

**En el caso de la implementación de Azure Container Service**, puede habilitar la terminación de Seguridad de la capa de transporte al [crear o adjuntar un clúster de Azure Container Service](how-to-create-attach-kubernetes.md) en el área de trabajo de Azure Machine Learning. En el momento de implementar el modelo de Azure Container Service, se puede deshabilitar la terminación de Seguridad de la capa de transporte con el objeto de configuración de implementación; de lo contrario, todas las implementaciones de modelos de Azure Container Service de forma predeterminada tendrán la terminación de Seguridad de la capa de transporte habilitada en el momento de conexión o creación de Azure Kubernetes Service.

Para la implementación de ACI, puede habilitar la terminación TLS en el momento de implementación del modelo con el objeto de configuración de implementación.


### <a name="deploy-on-azure-kubernetes-service"></a>Implementación en Azure Kubernetes Service

  > [!NOTE]
  > La información de esta sección también se aplica al implementar un servicio web seguro para el diseñador. Si no está familiarizado con el uso del SDK para Python, consulte [What is the Azure Machine Learning SDK for Python?](/python/api/overview/azure/ml/intro) (¿Qué es el SDK de Azure Machine Learning para Python?).

Cuando se [crea o conecta un clúster de Azure Container Service](how-to-create-attach-kubernetes.md) en el área de trabajo de Azure Machine Learning, se puede habilitar la terminación de Seguridad de la capa de transporte con los objetos de configuración **[AksCompute.provisioning_configuration ()](/python/api/azureml-core/azureml.core.compute.akscompute#provisioning-configuration-agent-count-none--vm-size-none--ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--location-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--service-cidr-none--dns-service-ip-none--docker-bridge-cidr-none--cluster-purpose-none--load-balancer-type-none--load-balancer-subnet-none-)** y **[AksCompute.attach_configuration ()](/python/api/azureml-core/azureml.core.compute.akscompute#attach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)** . Ambos métodos devuelven un objeto de configuración que tiene un método **enable_ssl** y se puede usar el método **enable_ssl** para habilitar TLS.

TLS se puede habilitar con el certificado de Microsoft o con un certificado personalizado adquirido en una CA. 

* **Cuando use un certificado de Microsoft**, debe usar el parámetro *leaf_domain_label*. Este parámetro genera el nombre DNS para el servicio. Por ejemplo, el valor "contoso" crea el nombre de dominio "contoso\<six-random-characters>.\<azureregion>.cloudapp.azure.com", donde \<azureregion> es la región que contiene el servicio. Si lo desea, puede usar el parámetro *overwrite_existing_domain* para sobrescribir el *leaf_domain_label* existente. En el siguiente ejemplo se muestra cómo se crea una configuración que habilita una TLS con certificado de Microsoft:

    ```python
    from azureml.core.compute import AksCompute

    # Config used to create a new AKS cluster and enable TLS
    provisioning_config = AksCompute.provisioning_configuration()

    # Leaf domain label generates a name using the formula
    #  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.com"
    #  where "######" is a random series of characters
    provisioning_config.enable_ssl(leaf_domain_label = "contoso")


    # Config used to attach an existing AKS cluster to your workspace and enable TLS
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                          cluster_name = cluster_name)

    # Leaf domain label generates a name using the formula
    #  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.com"
    #  where "######" is a random series of characters
    attach_config.enable_ssl(leaf_domain_label = "contoso")
    ```
    > [!IMPORTANT]
    > Si usa un certificado de Microsoft, no es necesario adquirir su propio nombre de dominio ni certificado.

    > [!WARNING]
    > Si el clúster de AKS se configura con un equilibrador de carga interno, __no se admite__ el uso de un certificado proporcionado por Microsoft, se debe usar un certificado personalizado para habilitar TLS.

* **Cuando use un certificado que haya comprado**, emplee los parámetros *ssl_cert_pem_file*, *ssl_key_pem_file* y *ssl_cname*. El ejemplo siguiente muestra cómo usar archivos .pem para crear una configuración que use un certificado TLS/SSL adquirido:
 
    ```python
    from azureml.core.compute import AksCompute

    # Config used to create a new AKS cluster and enable TLS
    provisioning_config = AksCompute.provisioning_configuration()
    provisioning_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")

    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
    attach_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

Para más información acerca de *enable_ssl*, consulte [AksProvisioningConfiguration.enable_ssl()](/python/api/azureml-core/azureml.core.compute.aks.aksprovisioningconfiguration#enable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-) y [AksAttachConfiguration.enable_ssl()](/python/api/azureml-core/azureml.core.compute.aks.aksattachconfiguration#enable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-).

### <a name="deploy-on-azure-container-instances"></a>Implementación en Azure Container Instances

Al implementar en Azure Container Instances, proporcione valores para los parámetros relacionados con TLS, como el siguiente fragmento de código:

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(
    ssl_enabled=True, ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
```

Para más información, consulte [AciWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aciwebservice#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-).

## <a name="update-your-dns"></a>Actualización del DNS

Para realizar una implementación de Azure Container Service con un certificado o una implementación de ACI, debe actualizar el registro de DNS para que apunte a la dirección IP del punto de conexión de puntuación.

  > [!IMPORTANT]
  > Cuando se usa un certificado de Microsoft para la implementación de Azure Container Service, no es preciso actualizar manualmente el valor de DNS para el clúster. El valor debe establecerse automáticamente.

Para actualizar el registro DNS para el nombre de dominio personalizado puede seguir estos pasos:
* Obtenga la dirección IP del punto de conexión de puntuación del identificador URI del punto de conexión de puntuación, que normalmente tiene el formato *http://104.214.29.152:80/api/v1/service/<service-name>/score* . 
* Use las herramientas del registrador de nombres de dominio para actualizar el registro de DNS de su nombre de dominio. El registro debe apuntar a la dirección IP del punto de conexión de puntuación.
* Después de la actualización del registro de DNS, puede validar la resolución de DNS mediante el comando *nslookup custom-domain-name*. Si el registro de DNS se actualiza correctamente, el nombre de dominio personalizado apuntará a la dirección IP del punto de conexión de puntuación.
* Puede haber una demora de varios minutos o de horas antes de que los clientes puedan resolver el nombre de dominio, en función del registrador y del período de vida (TTL) configurado para el nombre de dominio.


## <a name="update-the-tlsssl-certificate"></a>Actualización del certificado TLS/SSL

Los certificados TLS/SSL expiran y se deben renovar. Normalmente esto sucede cada año. Utilice la información de las secciones siguientes para actualizar y renovar el certificado para los modelos implementados en Azure Kubernetes Service:

### <a name="update-a-microsoft-generated-certificate"></a>Actualización de un certificado generado por Microsoft

Si Microsoft ha generado el certificado originalmente (mediante *leaf_domain_label* para crear el servicio), **se renueva automáticamente** cuando resulta necesario. Si quiere renovarlo manualmente, use alguno de los ejemplos siguientes para actualizarlo:

> [!IMPORTANT]
> * Si el certificado existente sigue siendo válido, use `renew=True` (SDK) o `--ssl-renew` (CLI) para forzar que la configuración lo renueve. Por ejemplo, si el certificado existente sigue siendo válido durante 10 días y no usa `renew=True`, es posible que el certificado no se renueve.
> * Cuando el servicio se implementó originalmente, `leaf_domain_label` se usa para crear un nombre DNS con el patrón `<leaf-domain-label>######.<azure-region>.cloudapp.azure.com`. Para conservar el nombre existente (incluidos los 6 dígitos generados originalmente), use el valor `leaf_domain_label` original. No incluya los 6 dígitos que se generaron.

**Uso del SDK**

```python
from azureml.core.compute import AksCompute
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute.aks import SslConfiguration

# Get the existing cluster
aks_target = AksCompute(ws, clustername)

# Update the existing certificate by referencing the leaf domain label
ssl_configuration = SslConfiguration(leaf_domain_label="myaks", overwrite_existing_domain=True, renew=True)
update_config = AksUpdateConfiguration(ssl_configuration)
aks_target.update(update_config)
```

**Uso de la CLI**

```azurecli
az ml computetarget update aks -g "myresourcegroup" -w "myresourceworkspace" -n "myaks" --ssl-leaf-domain-label "myaks" --ssl-overwrite-domain True --ssl-renew
```

Para más información, consulte los siguientes documentos de referencia:

* [SslConfiguration](/python/api/azureml-core/azureml.core.compute.aks.sslconfiguration)
* [AksUpdateConfiguration](/python/api/azureml-core/azureml.core.compute.aks.aksupdateconfiguration)

### <a name="update-custom-certificate"></a>Actualización del certificado personalizado

Si una entidad de certificación generó originalmente el certificado, siga estos pasos:

1. Use la documentación proporcionada por la entidad de certificación para renovar el certificado. Este proceso crea nuevos archivos de certificado.

1. Use el SDK o la CLI para actualizar el servicio con el nuevo certificado:

    **Uso del SDK**

    ```python
    from azureml.core.compute import AksCompute
    from azureml.core.compute.aks import AksUpdateConfiguration
    from azureml.core.compute.aks import SslConfiguration
    
    # Read the certificate file
    def get_content(file_name):
        with open(file_name, 'r') as f:
            return f.read()

    # Get the existing cluster
    aks_target = AksCompute(ws, clustername)
    
    # Update cluster with custom certificate
    ssl_configuration = SslConfiguration(cname="myaks", cert=get_content('cert.pem'), key=get_content('key.pem'))
    update_config = AksUpdateConfiguration(ssl_configuration)
    aks_target.update(update_config)
    ```

    **Uso de la CLI**

    ```azurecli
    az ml computetarget update aks -g "myresourcegroup" -w "myresourceworkspace" -n "myaks" --ssl-cname "myaks"--ssl-cert-file "cert.pem" --ssl-key-file "key.pem"
    ```

Para más información, consulte los siguientes documentos de referencia:

* [SslConfiguration](/python/api/azureml-core/azureml.core.compute.aks.sslconfiguration)
* [AksUpdateConfiguration](/python/api/azureml-core/azureml.core.compute.aks.aksupdateconfiguration)

## <a name="disable-tls"></a>Deshabilitación de TLS

Para deshabilitar TLS para un modelo implementado en Azure Kubernetes Service, cree un `SslConfiguration` con `status="Disabled"` y, a continuación, realice una actualización:

```python
from azureml.core.compute import AksCompute
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute.aks import SslConfiguration

# Get the existing cluster
aks_target = AksCompute(ws, clustername)

# Disable TLS
ssl_configuration = SslConfiguration(status="Disabled")
update_config = AksUpdateConfiguration(ssl_configuration)
aks_target.update(update_config)
```

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo:
+ [Consumir un modelo de Machine Learning implementado como un servicio web](how-to-consume-web-service.md)
+ [Información general sobre la privacidad y el aislamiento de la red virtual](how-to-network-security-overview.md)