---
title: Implementación de la primera aplicación en Cloud Foundry en Microsoft Azure
description: Implemente una aplicación en Cloud Foundry en Azure
author: seanmck
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 06/14/2017
ms.author: seanmck
ms.openlocfilehash: 1cffc36cbd4f24bbcbb5996a323ffa963e311693
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530953"
---
# <a name="deploy-your-first-app-to-cloud-foundry-on-microsoft-azure"></a>Implementación de la primera aplicación en Cloud Foundry en Microsoft Azure

[Cloud Foundry](https://cloudfoundry.org) es una popular plataforma de aplicaciones de código abierto disponible en Microsoft Azure. En este artículo, se muestra cómo implementar y administrar una aplicación en Cloud Foundry en un entorno de Azure.

## <a name="create-a-cloud-foundry-environment"></a>Creación de un entorno de Cloud Foundry

Hay varias opciones para crear un entorno de Cloud Foundry en Azure:

- Use la [oferta de Pivotal Cloud Foundry][pcf-azuremarketplace] que encontrará en Azure Marketplace para crear un entorno estándar que incluya PCF Ops Manager y Azure Service Broker. En la documentación de Pivotal puede encontrar [instrucciones completas][pcf-azuremarketplace-pivotaldocs] para implementar la oferta de Marketplace.
- Cree un entorno personalizado [implementando Pivotal Cloud Foundry manualmente][pcf-custom].
- [Implemente directamente los paquetes de Cloud Foundry de código abierto][oss-cf-bosh]. Para ello, configure un director de [BOSH](https://bosh.io), una máquina virtual que coordina la implementación del entorno de Cloud Foundry.

> [!IMPORTANT] 
> Si va a implementar PCF desde Azure Marketplace, anote tanto SYSTEMDOMAINURL como las credenciales de administrador requeridas para acceder a Pivotal Apps Manager, ambos se describen en la guía de implementación de Marketplace. Ambos son necesarios para completar este tutorial. En el caso de las implementaciones de Marketplace, SYSTEMDOMAINURL tiene el formato `https://system.*ip-address*.cf.pcfazure.com`.

## <a name="connect-to-the-cloud-controller"></a>Conexión a Cloud Controller

Cloud Controller es el punto de entrada principal a un entorno de Cloud Foundry para implementar y administrar aplicaciones. La API de Cloud Controller (CCAPI) principal es una API de REST, pero se puede acceder a ella a través de varias herramientas. En este caso, se interactúa con ella mediante la [CLI de Cloud Foundry][cf-cli]. La CLI se puede instalar en Linux, macOS o Windows, pero si prefiere no instalarla, está disponible preinstalada en [Azure Cloud Shell][cloudshell-docs].

Para iniciar sesión, anteponga `api` a la dirección de SYSTEMDOMAINURL que ha obtenido en la implementación de Marketplace. Dado que la implementación predeterminada usa un certificado autofirmado, también debe incluir el modificador `skip-ssl-validation`.

```bash
cf login -a https://api.SYSTEMDOMAINURL --skip-ssl-validation
```

Se le solicita que inicie sesión en Cloud Controller. Utilice las credenciales de la cuenta de administrador que adquirió en los pasos de implementación de Marketplace.

Cloud Foundry proporciona *organizaciones* y *espacios* como espacios de nombres para aislar los equipos y entornos en una implementación compartida. La implementación de PCF de Marketplace incluye la organización predeterminada del *sistema* y un conjunto de espacios que se crean para contener los componentes base, como el servicio de escalado automático y Azure Service Broker. Por ahora, elija el espacio del *sistema*.


## <a name="create-an-org-and-space"></a>Creación de una organización y un espacio

Si escribe `cf apps`, verá un conjunto de aplicaciones del sistema que se han implementado en el espacio del sistema dentro de la organización del sistema. 

Debe mantener la organización del *sistema* reservada para las aplicaciones del sistema, así que cree una organización y un espacio que alojen nuestra aplicación de ejemplo.

```bash
cf create-org myorg
cf create-space dev -o myorg
```

Use el comando de destino para cambiar a la organización y espacio nuevos:

```bash
cf target -o testorg -s dev
```

Ahora, cuando se implementa una aplicación, se crea automáticamente en la organización y el espacio nuevos. Para confirmar que actualmente no existen aplicaciones en la organización y espacios nuevos, vuelva a escribir `cf apps`.

> [!NOTE] 
> Para más información sobre las organizaciones y los espacios y cómo se pueden usar con el control de acceso basado en roles de Cloud Foundry (RBAC de Cloud Foundry), consulte la [documentación de Cloud Foundry][cf-orgs-spaces-docs].

## <a name="deploy-an-application"></a>Implementar una aplicación

Vamos a usar una aplicación de Cloud Foundry de ejemplo denominada Hello Spring Cloud, que está escrita en Java y se basa en [Spring Framework](https://spring.io) y [Spring Boot](https://projects.spring.io/spring-boot/).

### <a name="clone-the-hello-spring-cloud-repository"></a>Clonación del repositorio de Hello Spring Cloud

La aplicación de ejemplo Hello Spring Cloud está disponible en GitHub. Clónela en su entorno y cambie al directorio nuevo:

```bash
git clone https://github.com/cloudfoundry-samples/hello-spring-cloud
cd hello-spring-cloud
```

### <a name="build-the-application"></a>Compilar la aplicación

Compile la aplicación con [Apache Maven](https://maven.apache.org).

```bash
mvn clean package
```

### <a name="deploy-the-application-with-cf-push"></a>Implementación de la aplicación con cf push

Para implementar la mayoría de las aplicaciones en Cloud Foundry, se puede usar el comando `push`:

```bash
cf push
```

Cuando se *inserta* una aplicación, Cloud Foundry detecta el tipo de aplicación (en este caso, una aplicación de Java) e identifica sus dependencias (en este caso, el marco de Spring). A continuación, empaqueta todo lo necesario para ejecutar el código en una imagen de contenedor independiente, que se conoce como *droplet*. Por último, Cloud Foundry programa la aplicación en una de las máquinas disponibles en el entorno y crea una dirección URL para acceder a él que está disponible en la salida del comando.

![Salida de comando cf push][cf-push-output]

Para ver la aplicación hello-spring-cloud, abra la dirección URL del explorador:

![Interfaz de usuario predeterminada de Hello Spring Cloud][hello-spring-cloud-basic]

> [!NOTE] 
> Para más información acerca de lo que sucede durante `cf push`, consulte [How Applications Are Staged][cf-push-docs] (Cómo se almacenan provisionalmente las aplicaciones) en la documentación de Cloud Foundry.

## <a name="view-application-logs"></a>Visualización de registros de aplicaciones

La CLI de Cloud Foundry se puede usar para ver los registros de una aplicación por su nombre:

```bash
cf logs hello-spring-cloud
```

De forma predeterminada, el comando logs usa *tail*, que muestra los nuevos registros cuando se escriben. Para ver aparecer los nuevos registros, actualice la aplicación hello-spring-cloud en el explorador.

Para ver los registros que ya se han escrito, agregue el modificador `recent`:

```bash
cf logs --recent hello-spring-cloud
```

## <a name="scale-the-application"></a>Escalado de una aplicación

De forma predeterminada, `cf push` solo crea una instancia individual de la aplicación. Para garantizar una alta disponibilidad y habilitar el escalado horizontal para que aumente el rendimiento, por lo general habrá que ejecutar más de una instancia de las aplicaciones. Las aplicaciones ya implementadas se pueden escalar horizontalmente con facilidad mediante el comando `scale`:

```bash
cf scale -i 2 hello-spring-cloud
```

La ejecución del comando `cf app` en la aplicación muestra que Cloud Foundry crea otra instancia de la aplicación. Una vez que se ha iniciado la aplicación, Cloud Foundry inicia automáticamente en ella el tráfico con equilibrio de carga.


## <a name="next-steps"></a>Pasos siguientes

- [Consulte la documentación de Cloud Foundry][cloudfoundry-docs]
- [Configure el complemento Azure DevOps Services para Cloud Foundry][vsts-plugin]
- [Configure Microsoft Log Analytics Nozzle para Cloud Foundry][loganalytics-nozzle]

<!-- LINKS -->

[pcf-azuremarketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry
[pcf-custom]: https://docs.pivotal.io/pivotalcf/1-10/customizing/azure.html
[oss-cf-bosh]: https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs
[pcf-azuremarketplace-pivotaldocs]: https://docs.pivotal.io/ops-manager/2-10/install/pcf_azure.html
[cf-cli]: https://github.com/cloudfoundry/cli
[cloudshell-docs]: ../cloud-shell/overview.md
[cf-orgs-spaces-docs]: https://docs.cloudfoundry.org/concepts/roles.html
[spring-boot]: https://projects.spring.io/spring-boot/
[spring-framework]: https://spring.io
[cf-push-docs]: https://docs.cloudfoundry.org/concepts/how-applications-are-staged.html
[cloudfoundry-docs]: https://docs.cloudfoundry.org
[vsts-plugin]: https://github.com/Microsoft/vsts-cloudfoundry
[loganalytics-nozzle]: https://github.com/Azure/oms-log-analytics-firehose-nozzle

<!-- IMAGES -->
[cf-push-output]: ./media/cloudfoundry-deploy-your-first-app/cf-push-output.png
[hello-spring-cloud-basic]: ./media/cloudfoundry-deploy-your-first-app/hello-spring-cloud-basic.png
