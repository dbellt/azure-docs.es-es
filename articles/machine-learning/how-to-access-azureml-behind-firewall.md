---
title: Uso de un firewall
titleSuffix: Azure Machine Learning
description: Controle el acceso a las áreas de trabajo de Azure Machine Learning con instancias de Azure Firewall. Averigüe qué hosts debe permitir a través del firewall.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 05/11/2021
ms.custom: devx-track-python
ms.openlocfilehash: fc04655db898902a93c4e404f51d15393db3d92e
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109785266"
---
# <a name="use-workspace-behind-a-firewall-for-azure-machine-learning"></a>Uso de áreas de trabajo detrás de un firewall en Azure Machine Learning

En este artículo, aprenderá a configurar Azure Firewall para controlar el acceso al área de trabajo de Azure Machine Learning y a la red pública de Internet. Para obtener más información sobre la protección de Azure Machine Learning, consulte [Seguridad de empresa para Azure Machine Learning](concept-enterprise-security.md).

> [!WARNING]
> El acceso al almacenamiento de datos detrás de un firewall solo se admite en las experiencias de Code First. No se admite el uso de [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md) para acceder a los datos que se encuentran detrás de un firewall. Para trabajar con el almacenamiento de datos en una red privada con Studio, primero debe [configurar una red virtual](../virtual-network/quick-create-portal.md) y [proporcionar al estudio acceso a los datos almacenados dentro de una red virtual](how-to-enable-studio-virtual-network.md).

## <a name="azure-firewall"></a>Azure Firewall

Al usar Azure Firewall, use la __traducción de direcciones de red de destino (DNAT)__ para crear reglas NAT para el tráfico entrante. En el caso del tráfico saliente, cree reglas de __red__ o de __aplicación__. Estas colecciones de reglas se describen con más detalle en [¿Cuáles son algunos de los conceptos de Azure Firewall?](../firewall/firewall-faq.yml#what-are-some-azure-firewall-concepts)

### <a name="inbound-configuration"></a>Configuración de entrada

Si usa una __instancia de proceso__ o __clúster de proceso__ de Azure Machine Learning, agregue [rutas definidas por el usuario (UDR)](../virtual-network/virtual-networks-udr-overview.md) para la subred que contiene los recursos de Azure Machine Learning. Esta ruta fuerza el tráfico __de__ las direcciones IP de los recursos `BatchNodeManagement` y `AzureMachineLearning` a la dirección IP pública de la instancia de proceso y del clúster de proceso.

Estas UDR permiten que el servicio Batch se comunique con los nodos de proceso para programar tareas. Agregue también la dirección IP de Azure Machine Learning Service, ya que esto es necesario para acceder a las instancias de proceso. Cuando agregue la dirección IP de Azure Machine Learning Service, debe agregar la dirección IP para las regiones __primaria y secundaria__ de Azure. La región primaria es aquella en la que se encuentra el área de trabajo.

Para ubicar la región secundaria, consulte [Garantía de continuidad empresarial y recuperación ante desastres con regiones emparejadas de Azure](../best-practices-availability-paired-regions.md#azure-regional-pairs). Por ejemplo, si Azure Machine Learning Service está en la región Este de EE. UU. 2, la región secundaria es Centro de EE. UU. 

Para obtener una lista de direcciones IP del servicio Batch y de Azure Machine Learning Service, utilice uno de los métodos siguientes:

* Descargue los [intervalos de direcciones IP y las etiquetas de servicio de Azure](https://www.microsoft.com/download/details.aspx?id=56519) y busque `BatchNodeManagement.<region>` y `AzureMachineLearning.<region>` en el archivo, donde `<region>` es su región de Azure.

* Use la [CLI de Azure](/cli/azure/install-azure-cli) para descargar la información. En el ejemplo siguiente se descarga la información de la dirección IP y se filtra para la región Este de EE. UU. 2 (primaria) y Centro de EE. UU. (secundaria):

    ```azurecli-interactive
    az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'Batch')] | [?properties.region=='eastus2']"
    # Get primary region IPs
    az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='eastus2']"
    # Get secondary region IPs
    az network list-service-tags -l "Central US" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='centralus']"
    ```

    > [!TIP]
    > Si usa las regiones US-Virginia o US-Arizona, o las regiones China-East-2, estos comandos no devuelven direcciones IP. Use mejor uno de los siguientes vínculos para descargar una lista de direcciones IP:
    >
    > * [Intervalos de direcciones IP y etiquetas de servicio de Azure para Azure Government](https://www.microsoft.com/download/details.aspx?id=57063)
    > * [Intervalos de direcciones IP y etiquetas de servicio de Azure para Azure China](https://www.microsoft.com//download/details.aspx?id=57062)

Cuando agregue las rutas definidas por el usuario, defina la ruta del prefijo de cada dirección IP de Batch relacionada y en __Tipo del próximo salto__, seleccione __Internet__. En la imagen siguiente se muestra un ejemplo de esta UDR en Azure Portal:

![Ejemplo de una ruta definida por el usuario para un prefijo de dirección](./media/how-to-enable-virtual-network/user-defined-route.png)

> [!IMPORTANT]
> Las direcciones IP pueden cambiar con el tiempo.

Para más información, consulte [Creación de un grupo de Azure Batch en una red virtual](../batch/batch-virtual-network.md#user-defined-routes-for-forced-tunneling).

### <a name="outbound-configuration"></a>Configuración de salida

1. Agregue __reglas de red__ para permitir el tráfico __hacia__ y __desde__ las siguientes etiquetas de servicio:

    * AzureActiveDirectory
    * AzureMachineLearning
    * AzureResourceManager
    * Storage.region
    * KeyVault.region
    * ContainerRegistry.region

    Si planea usar las imágenes de Docker predeterminadas proporcionadas por Microsoft y habilitar las dependencias administradas por el usuario, también debe agregar las siguientes etiquetas de servicio:

    * MicrosoftContainerRegistry.region
    * AzureFrontDoor.FirstParty

    En el caso de las entradas que contienen `region`, reemplace por la región de Azure que esté usando. Por ejemplo, `keyvault.westus`.

    Para el __protocolo__, seleccione `TCP`. Para los __puertos__ de origen y de destino, seleccione `*`.

1. Agregue __reglas de aplicación__ para los siguientes hosts:

    > [!NOTE]
    > Esta no es una lista completa de los hosts necesarios para todos los recursos de Python en Internet, solo el que se usa con más frecuencia. Por ejemplo, si necesita acceder a un repositorio de GitHub o a otro host, debe identificar y agregar los hosts necesarios para ese escenario.

    | **Nombre de host** | **Propósito** |
    | ---- | ---- |
    | **graph.windows.net** | Se usa en instancias de proceso/clústeres de Azure Machine Learning. |
    | **anaconda.com**</br>**\*.anaconda.com** | Se usa para instalar paquetes predeterminados. |
    | **\*.anaconda.org** | Se usa para obtener datos del repositorio. |
    | **pypi.org** | Se usa para enumerar las dependencias del índice predeterminado, si hay alguna, y el índice no se sobrescribe con la configuración del usuario. Si el índice se sobrescribe, también debe permitir **\*.pythonhosted.org**. |
    | **cloud.r-project.org** | Se usar al instalar paquetes CRAN para el desarrollo en R. |
    | **\*pytorch.org** | Se usa en algunos ejemplos basados en PyTorch. |
    | **\*.tensorflow.org** | Se usa en algunos ejemplos basados en TensorFlow. |

    En __Protocolo:Puerto__, seleccione usar __http, https__.

    Para obtener más información sobre la configuración de reglas de aplicación, consulte [Implementación y configuración de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md#configure-an-application-rule).

1. Para restringir el acceso a los modelos implementados en Azure Kubernetes Service (AKS), consulte [Control del tráfico de salida en Azure Kubernetes Service](../aks/limit-egress-traffic.md).

## <a name="other-firewalls"></a>Otros firewalls

La guía de esta sección es genérica, ya que cada firewall tiene su propia terminología y configuraciones específicas. Si tiene alguna pregunta sobre cómo permitir la comunicación a través del firewall, consulte la documentación del firewall que usa.

Si no está configurado correctamente, el firewall puede provocar problemas al usar el área de trabajo. Hay una serie de nombres de host que se usan en el área de trabajo de Azure Machine Learning. En las secciones siguientes se enumeran los hosts necesarios para Azure Machine Learning.

### <a name="microsoft-hosts"></a>Hosts de Microsoft

Los hosts de esta sección son propiedad de Microsoft y proporcionan servicios necesarios para que el área de trabajo funcione correctamente. En las tablas siguientes se enumeran los nombres de host para las regiones de Azure público, Azure Government y Azure China 21Vianet.

**Hosts generales de Azure**

| **Requerido para** | **Azure público** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Azure Active Directory | login.microsoftonline.com | login.microsoftonline.us | login.chinacloudapi.cn |
| Azure portal | management.azure.com | management.azure.us | management.azure.cn |
| Azure Resource Manager | management.azure.com | management.usgovcloudapi.net | management.chinacloudapi.cn |

**Hosts de Azure Machine Learning**

| **Requerido para** | **Azure público** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Azure Machine Learning Studio | ml.azure.com | ml.azure.us | studio.ml.azure.cn |
| API |\*.azureml.ms | \*.ml.azure.us | \*.ml.azure.cn |
| Cuaderno integrado | \*.notebooks.azure.net | \*.notebooks.usgovcloudapi.net |\*.notebooks.chinacloudapi.cn |
| Cuaderno integrado | \*.file.core.windows.net | \*.file.core.usgovcloudapi.net | \*.file.core.chinacloudapi.cn |
| Cuaderno integrado | \*.dfs.core.windows.net | \*.dfs.core.usgovcloudapi.net | \*.dfs.core.chinacloudapi.cn |
| Cuaderno integrado | \*.blob.core.windows.net | \*.blob.core.usgovcloudapi.net | \*.blob.core.chinacloudapi.cn |
| Cuaderno integrado | graph.microsoft.com | graph.microsoft.us | graph.chinacloudapi.cn |
| Cuaderno integrado | \*.aznbcontent.net |  | |

**Hosts de instancia de proceso y clúster de proceso de Azure Machine Learning**

| **Requerido para** | **Azure público** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Instancia/clúster de proceso | \*.batchai.core.windows.net | \*.batchai.core.usgovcloudapi.net |\*.batchai.ml.azure.cn |
| Instancia/clúster de proceso | graph.windows.net | graph.windows.net | graph.chinacloudapi.cn |
| Instancia de proceso | \*.instances.azureml.net | \*.instances.azureml.us | \*.instances.azureml.cn |
| Instancia de proceso | \*.instances.azureml.ms |  |  |

> [!IMPORTANT]
> El firewall debe permitir la comunicación con \*.instances.azureml.ms a través del puerto __TCP__ __18881__.

**Recursos asociados usados por Azure Machine Learning**

| **Requerido para** | **Azure público** | **Azure Government** | **Azure China 21Vianet** |
| ----- | ----- | ----- | ----- |
| Cuenta de Azure Storage | core.windows.net | core.usgovcloudapi.net | core.chinacloudapi.cn |
| Azure Key Vault | vault.azure.net | vault.usgovcloudapi.net | vault.azure.cn |
| Azure Container Registry | azurecr.io | azurecr.us | azurecr.cn |
| Registro de contenedor de Microsoft | mcr.microsoft.com | mcr.microsoft.com | mcr.microsoft.com |


> [!TIP]
> Si tiene previsto usar la identidad federada, siga el artículo [Procedimientos recomendados para proteger los Servicios de federación de Active Directory (AD FS)](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs).

Además, use la información de [tunelización forzada](how-to-secure-training-vnet.md#forced-tunneling) para agregar direcciones IP para `BatchNodeManagement` y `AzureMachineLearning`.

Para obtener información sobre la restricción del acceso a los modelos implementados en Azure Kubernetes Service (AKS), consulte [Control del tráfico de salida en Azure Kubernetes Service](../aks/limit-egress-traffic.md).

### <a name="python-hosts"></a>Hosts de Python

Los hosts de esta sección se usan para instalar paquetes de Python. Son necesarios durante el desarrollo, el entrenamiento y la implementación. 

> [!NOTE]
> Esta no es una lista completa de los hosts necesarios para todos los recursos de Python en Internet, solo el que se usa con más frecuencia. Por ejemplo, si necesita acceder a un repositorio de GitHub o a otro host, debe identificar y agregar los hosts necesarios para ese escenario.

| **Nombre de host** | **Propósito** |
| ---- | ---- |
| **anaconda.com**</br>**\*.anaconda.com** | Se usa para instalar paquetes predeterminados. |
| **\*.anaconda.org** | Se usa para obtener datos del repositorio. |
| **pypi.org** | Se usa para enumerar las dependencias del índice predeterminado, si hay alguna, y el índice no se sobrescribe con la configuración del usuario. Si el índice se sobrescribe, también debe permitir **\*.pythonhosted.org**. |
| **\*pytorch.org** | Se usa en algunos ejemplos basados en PyTorch. |
| **\*.tensorflow.org** | Se usa en algunos ejemplos basados en TensorFlow. |

### <a name="r-hosts"></a>Hosts de R

Los hosts de esta sección se usan para instalar paquetes de R. Son necesarios durante el desarrollo, el entrenamiento y la implementación.

> [!NOTE]
> Esta no es una lista completa de los hosts necesarios para todos los recursos de R en Internet, solo el que se usa con más frecuencia. Por ejemplo, si necesita acceder a un repositorio de GitHub o a otro host, debe identificar y agregar los hosts necesarios para ese escenario.

| **Nombre de host** | **Propósito** |
| ---- | ---- |
| **cloud.r-project.org** | Usado al instalar paquetes CRAN |

> [!IMPORTANT]
> Internamente, el SDK de R para Azure Machine Learning usa paquetes de Python. Por lo tanto, también debe permitir los hosts de Python a través del firewall.
## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](../firewall/tutorial-firewall-deploy-portal.md)
* [Protección de los trabajos de experimentación e inferencia de ML en una instancia de Azure Virtual Network](how-to-network-security-overview.md)
