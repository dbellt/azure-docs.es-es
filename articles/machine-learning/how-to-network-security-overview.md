---
title: Información general sobre el aislamiento y la seguridad de la red virtual
titleSuffix: Azure Machine Learning
description: Use una instancia de Azure Virtual Network aislada con Azure Machine Learning para proteger los recursos del área de trabajo y los entornos de proceso.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 03/02/2021
ms.topic: how-to
ms.custom: devx-track-python, references_regions, contperf-fy21q1
ms.openlocfilehash: e6b8a4bbbe596ec06f7f9b445dbaa439e1207e46
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107888717"
---
# <a name="virtual-network-isolation-and-privacy-overview"></a>Información general sobre la privacidad y el aislamiento de la red virtual

En este artículo, aprenderá a usar redes virtuales (VNET) para proteger la comunicación de red en Azure Machine Learning. En este artículo se usa un escenario de ejemplo para mostrar cómo configurar una red virtual completa.

Este artículo forma parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Se recomienda encarecidamente leer este artículo de información general para comprender los conceptos en primer lugar. 

Los demás artículos de esta serie son:

**1. Introducción a las redes virtuales** > [2. Protección del área de trabajo](how-to-secure-workspace-vnet.md) > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md) > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por hecho que está familiarizado con los siguientes temas:
+ [Azure Virtual Network](../virtual-network/virtual-networks-overview.md)
+ [Redes de IP](../virtual-network/public-ip-addresses.md)
+ [Azure Private Link](how-to-configure-private-link.md)
+ [Grupos de seguridad de red (NSG)](../virtual-network/network-security-groups-overview.md)
+ [Firewalls de red](../firewall/overview.md)
## <a name="example-scenario"></a>Escenario de ejemplo

En esta sección, aprenderá cómo se configura un escenario de red común para proteger la comunicación de Azure Machine Learning con las direcciones IP privadas.

En la tabla siguiente se compara el modo en que los servicios acceden a diferentes partes de una red de Azure Machine Learning con y sin una red virtual.

| Escenario | Área de trabajo |  Recursos asociados | Entrenamiento del entorno de proceso | Inferencia del entorno de proceso |
|-|-|-|-|-|-|
|**Sin red virtual**| Dirección IP pública | Dirección IP pública | Dirección IP pública | Dirección IP pública |
|**Protección de recursos en una red virtual**| IP privada (punto de conexión privado) | IP pública (punto de conexión de servicio) <br> **- o -** <br> IP privada (punto de conexión privado) | Dirección IP privada | Dirección IP privada  | 

* **Área de trabajo**: cree un punto de conexión privado desde la red virtual para conectarse a Private Link en el área de trabajo. El punto de conexión privado conecta el área de trabajo a la red virtual a través de varias direcciones IP privadas.
* **Recurso asociado**: use puntos de conexión de servicio o puntos de conexión privados para conectarse a recursos del área de trabajo como Azure Storage, Azure Key Vault y Azure Container Services.
    * Los **puntos de conexión de servicio** proporcionan la identidad de la red virtual al servicio de Azure. Una vez que habilita puntos de conexión de servicio en su red virtual, puede agregar una regla de red virtual para proteger los recursos de los servicios de Azure en la red virtual. Los puntos de conexión de servicio usan direcciones IP públicas.
    * Los **puntos de conexión privados** son interfaces de red que le permiten conectarse de forma segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual, y coloca el servicio de manera eficaz en su red virtual.
* **Acceso al proceso de entrenamiento**: acceda a destinos de proceso de entrenamiento como la instancia de proceso y los clústeres de proceso de Azure Machine Learning de forma segura con direcciones IP privadas. 
* **Acceso al proceso de inferencia**: acceda a los clústeres de proceso de Azure Kubernetes Services (AKS) con direcciones IP privadas.


En las cinco secciones siguientes se muestra cómo proteger el escenario de red descrito anteriormente. Para proteger la red, debe hacer lo siguiente:

1. Proteger el [**área de trabajo y los recursos asociados**](#secure-the-workspace-and-associated-resources).
1. Proteger el [**entorno de entrenamiento**](#secure-the-training-environment).
1. Proteger el [**entorno de inferencia**](#secure-the-inferencing-environment).
1. Opcional: [**habilitar la funcionalidad de Studio**](#optional-enable-studio-functionality).
1. Configurar el [**firewall**](#configure-firewall-settings).
1. Configurar la [resolución de nombres DNS](#custom-dns).
## <a name="secure-the-workspace-and-associated-resources"></a>Proteger el área de trabajo y los recursos asociados

Realice los pasos siguientes para proteger el área de trabajo y los recursos asociados. Estos pasos permiten que los servicios se comuniquen en la red virtual.

1. Cree una [área de trabajo habilitada para Private Link](how-to-secure-workspace-vnet.md#secure-the-workspace-with-private-endpoint) para habilitar la comunicación entre la red virtual y el área de trabajo.
1. Agregue los siguientes servicios a la red virtual _ya sea_ mediante un __punto de conexión de servicio__ o mediante un __punto de conexión privado__. También debe permitir que los servicios de confianza de Microsoft accedan a estos servicios:
    
    | Servicio | Información de punto de conexión | Información sobre los servicios de confianza permitidos |
    | ----- | ----- | ----- |
    | __Azure Key Vault__| [Punto de conexión de servicio](../key-vault/general/overview-vnet-service-endpoints.md)</br>[Punto de conexión privado](../key-vault/general/private-link-service.md) | [Permite que los servicios de Microsoft de confianza omitan este firewall](how-to-secure-workspace-vnet.md#secure-azure-key-vault) |
    | __Cuenta de Azure Storage__ | [Punto de conexión de servicio](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints)</br>[Punto de conexión privado](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-private-endpoints) | [Concesión de acceso a servicios de Azure de confianza](../storage/common/storage-network-security.md#grant-access-to-trusted-azure-services) |
    | __Azure Container Registry__ | [Punto de conexión de servicio](how-to-secure-workspace-vnet.md#enable-azure-container-registry-acr)</br>[Punto de conexión privado](../container-registry/container-registry-private-link.md) | [Permitir servicios de confianza](../container-registry/allow-access-trusted-services.md) |


![Diagrama de arquitectura que muestra cómo el área de trabajo y los recursos asociados se comunican entre sí a través de puntos de conexión de servicio o puntos de conexión privados en una red virtual](./media/how-to-network-security-overview/secure-workspace-resources.png)

Para obtener instrucciones detalladas sobre cómo completar estos pasos, consulte [Protección de un área de trabajo de Azure Machine Learning](how-to-secure-workspace-vnet.md). 

### <a name="limitations"></a>Limitaciones

La protección del área de trabajo y los recursos asociados en una red virtual presenta las siguientes limitaciones:
- El uso de un área de trabajo Azure Machine Learning con un vínculo privado no está disponible en las regiones de Azure Government ni Azure China 21Vianet.
- Todos los recursos deben estar detrás de la misma red virtual. Sin embargo, se permiten subredes en la misma red virtual.

## <a name="secure-the-training-environment"></a>Protección del entorno de entrenamiento

En esta sección, aprenderá a proteger el entorno de entrenamiento en Azure Machine Learning. También aprenderá cómo Azure Machine Learning completa un trabajo de entrenamiento para comprender el funcionamiento conjunto de las configuraciones de red.

Para proteger el entorno de entrenamiento, siga estos pasos:

1. Cree una [instancia de proceso y un clúster de proceso](how-to-secure-training-vnet.md#compute-instance) de Azure Machine Learning en la red virtual para ejecutar el trabajo de entrenamiento.
1. [Permita la comunicación entrante desde el servicio de Azure Batch](how-to-secure-training-vnet.md#mlcports) para que el servicio de Batch pueda enviar trabajos a los recursos de proceso. 

![Diagrama de arquitectura que muestra cómo proteger las instancias y los clústeres de proceso administrados](./media/how-to-network-security-overview/secure-training-environment.png)

Para obtener instrucciones detalladas sobre cómo completar estos pasos, consulte [Protección de un entorno de entrenamiento](how-to-secure-training-vnet.md). 

### <a name="example-training-job-submission"></a>Ejemplo de envío de trabajo de entrenamiento 

En esta sección, aprenderá cómo Azure Machine Learning se comunica de forma segura entre los servicios para enviar un trabajo de entrenamiento. Muestra cómo funcionan todas las configuraciones de forma conjunta para proteger la comunicación.

1. El cliente carga scripts y datos de entrenamiento en las cuentas de almacenamiento protegidas con un punto de conexión de servicio o privado.

1. El cliente envía un trabajo de entrenamiento al área de trabajo de Azure Machine Learning a través del punto de conexión privado.

1. El servicio de Azure Batch recibe el trabajo del área de trabajo y envía el trabajo de entrenamiento al entorno de proceso a través del equilibrador de carga público que se aprovisiona con el recurso de proceso. 

1. El recurso de proceso recibe el trabajo e inicia el entrenamiento. El recurso de proceso accede a cuentas de almacenamiento seguras para descargar archivos de entrenamiento y cargar la salida.

### <a name="limitations"></a>Limitaciones

- La instancia de proceso de Azure y los clústeres de proceso de Azure deben estar en la misma red virtual, región y suscripción que el área de trabajo y sus recursos asociados. 

## <a name="secure-the-inferencing-environment"></a>Protección del entorno de inferencia

En esta sección, aprenderá las opciones disponibles para proteger un entorno de inferencia. Se recomienda usar los clústeres de Azure Kubernetes Services (AKS) para las implementaciones de producción a gran escala.

Tiene dos opciones para los clústeres de AKS en una red virtual:

- Implementar o conectar un clúster de AKS predeterminado a la red virtual.
- Conectar un clúster de AKS privado a la red virtual.

Los **clústeres de AKS predeterminados** tienen un plano de control con las direcciones IP públicas. Puede agregar un clúster de AKS predeterminado a la red virtual durante la implementación o conectar un clúster después de su creación.

Los **clústeres de AKS privados** tienen un plano de control, al que solo se puede acceder a través de direcciones IP privadas. Los clústeres de AKS privados se deben conectar una vez creado el clúster.

Para obtener instrucciones detalladas sobre cómo agregar clústeres predeterminados y privados, consulte [Protección de un entorno de inferencia](how-to-secure-inferencing-vnet.md). 

En el diagrama de red siguiente se muestra un área de trabajo de Azure Machine Learning protegida con un clúster de AKS privado conectado a la red virtual.

![Diagrama de arquitectura que muestra cómo conectar un clúster de AKS privado a la red virtual. El plano de control de AKS se coloca fuera de la red virtual del cliente.](./media/how-to-network-security-overview/secure-inferencing-environment.png)

### <a name="limitations"></a>Limitaciones
- Los clústeres de AKS deben pertenecer a la misma red virtual que el área de trabajo y sus recursos asociados. 

## <a name="optional-enable-public-access"></a>Opcional: Habilitación del acceso público

Puede proteger el área de trabajo detrás de una red virtual mediante un punto de conexión privado y seguir permitiendo el acceso a través de la red pública de Internet. La configuración inicial equivale a [proteger el área de trabajo y los recursos asociados](#secure-the-workspace-and-associated-resources). 

Después de proteger el área de trabajo con un vínculo privado, [habilitar el acceso público](how-to-configure-private-link.md#enable-public-access). Después de esto, puede tener acceso al área de trabajo desde la red pública de Internet y la red virtual.

### <a name="limitations"></a>Limitaciones

- Si usa Estudio de Azure Machine Learning a través de la red pública de Internet, es posible que algunas características como el diseñador no tengan acceso a sus datos. Este problema se produce cuando los datos se almacenan en un servicio protegido detrás de la red virtual. Por ejemplo, una cuenta de Azure Storage.
## <a name="optional-enable-studio-functionality"></a>Opcional: habilitación de la funcionalidad de Studio

[Proteger el área de trabajo](#secure-the-workspace-and-associated-resources) > [Proteger el entorno de entrenamiento](#secure-the-training-environment) > [Proteger el entorno de inferencia](#secure-the-inferencing-environment) > **Habilitar la funcionalidad de Studio** > [Configurar el firewall](#configure-firewall-settings)

Si el almacenamiento se encuentra en una red virtual, primero debe realizar pasos de configuración adicionales para habilitar la funcionalidad completa en [Studio](overview-what-is-machine-learning-studio.md). De forma predeterminada, la característica siguiente está deshabilitada:

* Vista previa de los datos en Studio.
* Visualización de los datos en el diseñador.
* Implementación de un modelo en el diseñador.
* Envío de un experimento de AutoML.
* Inicio de un proyecto de etiquetado.

Para habilitar la funcionalidad de Studio completa desde una red virtual, consulte [Uso de Azure Machine Learning Studio en una red virtual](how-to-enable-studio-virtual-network.md#configure-data-access-in-the-studio). Studio admite cuentas de almacenamiento que usan puntos de conexión de servicio o puntos de conexión privados.

### <a name="limitations"></a>Limitaciones

[El etiquetado de datos asistido por ML](how-to-create-labeling-projects.md#use-ml-assisted-data-labeling) no es compatible con las cuentas de almacenamiento predeterminadas que están protegidas en una red virtual. Debe usar una cuenta de almacenamiento no predeterminada para el etiquetado de datos asistidos por ML. No obstante, la cuenta de almacenamiento no predeterminada se puede proteger en la red virtual. 

## <a name="configure-firewall-settings"></a>Configuración del firewall

Configure el firewall para controlar el acceso a los recursos del área de trabajo de Azure Machine Learning y a la red pública de Internet. Aunque se recomienda Azure Firewall, debería poder usar otros productos de firewall para proteger la red. Si tiene alguna pregunta sobre cómo permitir la comunicación a través del firewall, consulte la documentación del firewall que usa.

Para obtener más información sobre la configuración del firewall, consulte [Uso de áreas de trabajo detrás de un firewall](how-to-access-azureml-behind-firewall.md).

## <a name="custom-dns"></a>DNS personalizado

Si necesita usar una solución DNS personalizada para la red virtual, debe agregar registros de host para el área de trabajo.

Para obtener más información sobre los nombres de dominio y las direcciones IP que se requieren, consulte [Uso de un área de trabajo con un servidor DNS personalizado](how-to-custom-dns.md).

## <a name="next-steps"></a>Pasos siguientes

Este artículo es la primera parte de una serie de cinco capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 2: Introducción a las redes virtuales](how-to-secure-workspace-vnet.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)
* [Parte 5: Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

Consulte también el artículo sobre el uso de [DNS personalizado](how-to-custom-dns.md) para la resolución de nombres.