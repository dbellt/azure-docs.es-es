---
title: Implementación de la plataforma de Azure Industrial IoT
description: En este tutorial, aprenderá a implementar la plataforma de IIoT.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: 995ee04e913ad4e363045e0aacc295aaec25f3e6
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677933"
---
# <a name="tutorial-deploy-the-azure-industrial-iot-platform"></a>Tutorial: Implementación de la plataforma de Azure Industrial IoT

En este tutorial, aprenderá:

> [!div class="checklist"]
> * Acerca de los componentes principales de la plataforma de IIoT
> * Acerca de los diferentes tipos de instalación
> * Cómo implementar la plataforma de Azure Industrial IoT

## <a name="prerequisites"></a>Prerrequisitos

- Se debe crear una suscripción de Azure
- Descargar [Git](https://git-scm.com/downloads)
- Los registros de aplicaciones de Azure Active Directory (AAD) usados para la autenticación requieren derechos de administrador global, administrador de aplicaciones o administrador de aplicaciones en la nube para proporcionar el consentimiento del administrador a todo el inquilino (consulte a continuación más opciones).
- Los sistemas operativos compatibles para la implementación son Windows, Linux y Mac.
- IoT Edge admite Windows 10 IoT Enterprise LTSC y Ubuntu Linux 16.08 o 18.04 LTS

## <a name="main-components"></a>Componentes principales

- Dependencias mínimas: IoT Hub, Cosmos DB, Service Bus, Event Hub, Key Vault, Storage
- Dependencias estándar: dependencias mínimas + SignalR Service, registros de aplicaciones de AAD, servicio de aprovisionamiento de dispositivos, Time Series Insights, libros, Log Analytics, Application Insights
- Microservicios: plan de App Service, App Service
- Interfaz de usuario (aplicación web): plan de App Service (compartido con microservicios), App Service
- Simulación: máquina virtual, red virtual, IoT Edge
- Azure Kubernetes Service

## <a name="installation-types"></a>Tipos de instalación

- Mínima: dependencias mínimas
- Local: mínima y las dependencias estándar
- Servicios: local y microservicios
- Simulación: dependencias mínimas y componentes de simulación
- Aplicación: servicios y la interfaz de usuario
- Todo (valor predeterminado): aplicación y la simulación

## <a name="deployment"></a>Implementación

1. Para empezar a trabajar con la implementación de la plataforma de IIoT, clone el repositorio desde el símbolo del sistema o el terminal.

    Git Clone https://github.com/Azure/Industrial-IoT  cd Industrial-IoT

2. Inicie la implementación guiada, el script recopilará la información necesaria, como la cuenta de Azure, la suscripción, el recurso de destino y el nombre del grupo y de la aplicación.

    En Windows:
        ```
        .\deploy -version <version>
        ```

    En Linux o Mac:
        ```
        ./deploy.sh -version <version>
        ```

    Reemplace \<version> por la versión que desea implementar.

3. Los microservicios y la interfaz de usuario son aplicaciones web que requieren autenticación lo cual requiere tres registros de aplicaciones en AAD. Si faltan los derechos necesarios, hay dos soluciones posibles:

    - Pídale al administrador de AAD que conceda consentimiento del administrador para la aplicación en todo el inquilino
    - Un administrador de AAD puede crear las aplicaciones de AAD. La carpeta deploy/scripts contiene el script aad-register.ps1 para realizar el registro de AAD de forma independiente de la implementación. La salida del script es un archivo que contiene la información pertinente que se va a usar como parte de la implementación y que debe pasarse al script deploy.ps1 en la misma carpeta mediante el argumento - aadConfig.
        ```bash
        cd deploy/scripts
        ./aad-register.ps1 -Name <application-name> -Output aad.json
        ./deploy.ps1 -aadConfig aad.json
        ```

En el caso de las implementaciones de producción que requieren almacenamiento provisional, reversión, escalado y resistencia, la plataforma se puede implementar en [Azure Kubernetes Service (AKS)](https://github.com/Azure/Industrial-IoT/blob/master/docs/deploy/howto-deploy-aks.md).

Referencias:
- [Implementación de la plataforma de Azure Industrial IoT](https://github.com/Azure/Industrial-IoT/tree/master/docs/deploy)
- [Implementación todo en uno](https://github.com/Azure/Industrial-IoT/blob/master/docs/deploy/howto-deploy-all-in-one.md)
- [Implementación de la plataforma en AKS](https://github.com/Azure/Industrial-IoT/blob/master/docs/deploy/howto-deploy-aks.md)


## <a name="next-steps"></a>Pasos siguientes
Ahora que ha implementado la plataforma de IIoT, puede aprender a personalizar la configuración de los componentes:

> [!div class="nextstepaction"]
> [Personalización de la configuración de los componentes](tutorial-configure-industrial-iot-components.md)
