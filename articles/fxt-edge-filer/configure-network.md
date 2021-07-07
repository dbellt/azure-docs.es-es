---
title: 'Tutorial: Configuración de la red en un clúster de Azure FXT Edge Filer'
description: Cómo personalizar la configuración de red después de crear el clúster de Azure FXT Edge Filer
author: ekpgh
ms.author: v-erkel
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.openlocfilehash: fbbb9837a54d323b568e42c90132c799c3927c30
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111414670"
---
# <a name="tutorial-configure-the-clusters-network-settings"></a>Tutorial: Configuración de red del clúster

Antes de usar un clúster de Azure FXT Edge Filer recién creado, debe comprobar y personalizar varias configuraciones de red del flujo de trabajo.

En este tutorial se explica la configuración de red que podría tener que ajustar para un nuevo clúster.

Aprenderá a:

> [!div class="checklist"]
>
> * Qué configuración de red podría tener que actualizar después de crear un clúster
> * Qué casos de uso de Azure FXT Edge Filer requieren un servidor AD o un servidor DNS
> * Cómo configurar DNS (RRDNS) round robin para equilibrar automáticamente la carga de las solicitudes cliente en el clúster FXT

El tiempo necesario para completar estos pasos depende de la cantidad de cambios de configuración que sean necesarios en el sistema:

* Si solo necesita leer el tutorial y comprobar algunos valores, tardará de 10 a 15 minutos.
* Si necesita configurar DNS round robin, esa tarea puede tardar una hora o más.

## <a name="adjust-network-settings"></a>Ajuste de la configuración de red

En la configuración de un nuevo clúster de Azure FXT Edge Filer intervienen varias tareas relacionadas con la red. Compruebe esta lista y decida cuáles se aplican a su sistema.

Para más información sobre la configuración de red de un clúster, lea [Configuring network services](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/network_overview.html) (Configuración de los servicios de red) en la guía de configuración del clúster.

* Configure DNS round robin para la red orientada al cliente (opcional).

  Equilibre la carga del tráfico del clúster mediante la configuración del sistema DNS como se describe en [Configuración de DNS para el clúster de Azure FXT Edge Filer](#configure-dns-for-load-balancing).

* Compruebe la configuración de NTP.

* Configure las descargas de nombre de grupo y nombre de usuario de Active Directory (si es necesario).

  Si los hosts de red usan Active Directory u otro tipo de servicio de directorio externo, debe modificar la configuración de los servicios de directorio del clúster para configurar cómo el clúster descarga la información de nombre de usuario y grupo. Lea **Cluster** > **Directory Services** (Clúster > Servicios de directorio) en la guía de configuración del clúster para más información.

  Un servidor de AD es necesario si desea compatibilidad con SMB. Configure AD antes de comenzar a configurar SMB.

* Defina las redes VLAN (opcional).
  
  Configure cualquier red VLAN adicional necesaria antes de definir el espacio de nombres global y las instancias de servidor virtual del clúster. Lea [Working with VLANs](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/network_overview.html#vlan-overview) (Uso de redes VLAN) en la guía de configuración del clúster para más información.

* Configure los servidores proxy (si es necesario).

  Si el clúster usa un servidor proxy para tener acceso a direcciones externas, siga estos pasos para configurarlo:

  1. Defina el servidor proxy en la página **Proxy Configuration** (Configuración de proxy).
  1. Aplique la configuración del servidor proxy con la página **Cluster** > **General Setup** (Clúster > Configuración general) o la página **Core Filer Details** (Detalles principales de Filer).
  
  Para más información, lea [Using web proxies](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/proxy_overview.html) (Uso de servidores proxy) en la guía de configuración del clúster.

* Cargue los [certificados de cifrado](#encryption-certificates) que usará el clúster (opcional)

### <a name="encryption-certificates"></a>Certificados de cifrado

El clúster de FXT Edge Filer usa certificados X.509 para estas funciones:

* Para cifrar el tráfico de administración del clúster

* Para autenticarse en nombre de un cliente en servidores de terceros KMIP

* Para comprobar los certificados de servidor de los proveedores de nube

Si tiene que cargar los certificados en el clúster, use la página de configuración **Cluster** > **Certificates** (Clúster > Certificados). Los detalles se encuentran en la página [Cluster > Certificates](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_certificates.html) (Clúster > Certificados) de la guía de configuración del clúster.

Para cifrar la comunicación de administración del clúster, use la página de configuración **Cluster** > **General Setup** (Clúster > Configuración general) para seleccionar qué certificado usar para TLS administrativo.

Asegúrese de que las máquinas de administración cumplen los [estándares de cifrado](supported-ciphers.md) del clúster.

> [!Note]
> Las claves de acceso del servicio en la nube se almacenan mediante la página de configuración **Cloud Credentials** (Credenciales de nube). En la sección anterior [Add a core filer](add-storage.md#add-a-core-filer) (Adición de un archivador principal) se muestra un ejemplo; lea la sección [Cloud Credentials](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cloud_credentials.html) (Credenciales de nube) de la guía de configuración del clúster para más información.

## <a name="configure-dns-for-load-balancing"></a>Configuración de DNS para el equilibrio de carga

En esta sección se explican los conceptos básicos de la configuración de un sistema de DNS (RRDNS) round-robin para distribuir la carga del cliente entre todas las direcciones IP orientadas al cliente en el clúster de FXT Edge Filer.

### <a name="decide-whether-or-not-to-use-dns"></a>Decidir si usar o no DNS

Aunque siempre se recomienda equilibrar la carga, no es necesario usar siempre DNS. Por ejemplo, con algunos tipos de flujos de trabajo de cliente podría ser más adecuado usar un script para asignar direcciones IP del clúster uniformemente entre los clientes cuando montan el clúster. Algunos métodos se describen en [Mount the cluster](mount-clients.md) (Montaje del clúster).

Tenga esto en mente la hora de decidir si quiere usar un servidor DNS:

* Si solo los clientes NFS acceden al sistema, no es necesario DNS. Es posible especificar todas las direcciones de red mediante direcciones IP numéricas.

* Si el sistema admite el acceso de SMB (CIFS), DNS es necesario, ya que debe especificar un dominio DNS para el servidor de Active Directory.

* DNS es necesario si quiere utilizar la autenticación Kerberos.

### <a name="round-robin-dns-configuration-details"></a>Detalles de configuración de DNS round robin

Cuando los clientes acceden al clúster, RRDNS equilibra automáticamente sus solicitudes entre todas las interfaces disponibles.

Para obtener un rendimiento óptimo, configure el servidor DNS para controlar las direcciones de clúster orientadas al cliente, como se muestra en el diagrama siguiente.

Un vserver de clúster se muestra a la izquierda y las direcciones IP aparecen en el centro y a la derecha. Configure todos los puntos de acceso de cliente con registros A y punteros como se muestra en la ilustración.

![Diagrama de DNS round-robin: tras la imagen de un vínculo a texto detallado alternativo](media/fxt-cluster-config/fxt-rrdns-diagram.png)
[descripción de texto detallado](https://azure.github.io/Avere/legacy/Azure-FXT-EdgeFilerDNSconfiguration-alt-text.html)

Cada dirección IP orientada al cliente debe tener un nombre único para que el clúster la use internamente. (En este diagrama, las direcciones IP del cliente se denominan vs1-client-IP-* por motivos de claridad, pero en producción debería usar probablemente algo más conciso, como client*.)

Los clientes montan el clúster con el nombre de vserver como argumento del servidor.

Modifique el archivo ``named.conf`` del servidor DNS para establecer un orden cíclico para las consultas en su vserver. Esta opción garantiza que todos los valores disponibles se recorran de forma cíclica. Agregue una instrucción como la siguiente:

```
options {
    rrset-order {
        class IN A name "vserver1.example.com" order cyclic;
    };
};
```

Los siguientes comandos ``nsupdate`` proporcionan un ejemplo de configuración correcta de DNS:

```
update add vserver1.example.com. 86400 A 10.0.0.10
update add vserver1.example.com. 86400 A 10.0.0.11
update add vserver1.example.com. 86400 A 10.0.0.12
update add vs1-client-IP-10.example.com. 86400 A 10.0.0.10
update add vs1-client-IP-11.example.com. 86400 A 10.0.0.11
update add vs1-client-IP-12.example.com. 86400 A 10.0.0.12
update add 10.0.0.10.in-addr.arpa. 86400 PTR vs1-client-IP-10.example.com
update add 11.0.0.10.in-addr.arpa. 86400 PTR vs1-client-IP-11.example.com
update add 12.0.0.10.in-addr.arpa. 86400 PTR vs1-client-IP-12.example.com
```

### <a name="enable-dns-in-the-cluster"></a>Habilitación de DNS en el clúster

Especifique el servidor DNS que usa el clúster en la página de configuración **Cluster** > **Administrative Network** (Clúster > Red administrativa). La configuración de esta página incluye:

* Dirección del servidor DNS
* Nombre de dominio DNS
* Dominios de búsqueda DNS

Para más información, lea [DNS Settings](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_admin_network.html#gui-dns>) (Configuración de DNS) en la guía de configuración de clúster.

## <a name="next-steps"></a>Pasos siguientes

Este es el último paso de la configuración básica del clúster de Azure FXT Edge Filer.

* Aprenda sobre los LED del sistema y otros indicadores en [Monitor hardware status](monitor.md) (Supervisión del estado del hardware).
* Conozca más sobre cómo los clientes deben montar el clúster de FXT Edge Filer en [Mount the cluster](mount-clients.md) (Montaje del clúster).
* Para más información sobre el funcionamiento y la administración de un clúster de FXT Edge Filer, consulte la [guía de configuración del clúster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/ops_conf_index.html).
