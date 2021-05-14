---
title: Solución de problemas de Azure Static Web Apps
description: Primeros pasos para realizar el seguimiento de problemas al usar Azure Static Web Apps.
services: static-web-apps
author: geektrainer
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 04/07/2021
ms.author: chrhar
ms.openlocfilehash: 082c3af694932f4e864b99aa8c2cdc7d1f5fca30
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108210193"
---
# <a name="troubleshooting-deployment-and-runtime-errors"></a>Solución de problemas de errores de implementación y tiempo de ejecución

En este artículo se incluyen guías paso a paso para solucionar problemas de implementación y de otro tipo de la aplicación web estática.

## <a name="retrieve-deployment-error-messages"></a>Recuperación de mensajes de error de implementación

El flujo de trabajo de Azure Static Web Apps usa el [proceso de compilación de Oryx](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/nodejs.md#build) de Node.js que ejecuta automáticamente los siguientes comandos:

```bash
npm install
npm run build # if specified in package.json
npm run build:azure # if specified in package.json
```

Los errores producidos por este proceso se registran en la ejecución del flujo de trabajo de GitHub.

1. Vaya al repositorio de GitHub de la aplicación web estática.
1. Seleccione **Actions** (Acciones).

    > [!NOTE]
    > Las ejecuciones de flujo de trabajo con errores se mostrarán con una *X* roja en lugar de una marca de verificación verde.

1. Seleccione el vínculo para la ejecución del flujo de trabajo que quiere validar.
1. Seleccione **Build and Deploy Job** (Compilar e implementar trabajo) para abrir los detalles de la implementación.
1. Seleccione **Compilar e implementar** para mostrar el registro.

    ![Captura de pantalla del registro de implementación de una aplicación web estática](./media/troubleshooting/build-deploy-log.png)

1. Revise los registros y los mensajes de error.

    > [!NOTE]
    > Algunos mensajes de error de advertencia pueden mostrarse en rojo, como notas sobre *.oryx_prod_node_modules* y el *área de trabajo*. Estos mensajes forman parte del proceso de implementación normal.

Si algún paquete no se puede instalar u se produce otro problema, aquí verá los mensajes de error originales, como si ejecutara los comandos `npm` localmente.

### <a name="confirm-folder-configuration"></a>Confirmación de la configuración de carpetas

Azure Static Web Apps debe saber qué carpetas usar para hospedar la aplicación. El proceso de compilación confirma esta configuración al final del flujo de trabajo. Los errores se registran durante los pasos de validación.

Si ve uno de los siguientes mensajes de error en el registro de errores, es una indicación de que la configuración de carpetas en el flujo de trabajo es incorrecta.

| Mensaje de error | Descripción |
| --- | --- |
|Ubicación del directorio de la aplicación: "/*folder*" no es válida. No se pudo detectar este directorio. | Compruebe que el flujo de trabajo refleje la estructura del repositorio. |
| La compilación de la aplicación no pudo generar la carpeta de artefactos: "*folder*". | Asegúrese de que la propiedad `folder` esté configurada correctamente en el archivo del flujo de trabajo. |
| No se especificó ningún directorio de API o no se encontró el directorio especificado. | Azure Functions no se creará porque el flujo de trabajo no define un valor para la carpeta `api`. |

Hay tres ubicaciones de carpeta especificadas en el flujo de trabajo. Asegúrese de que esta configuración coincide con el proyecto y con las herramientas que transforman el código fuente antes de la implementación.

| Opción de configuración | Descripción |
| --- | --- |
| `app_location` | Ubicación raíz del código fuente que se va a implementar. Esta configuración suele ser */* o la ubicación de JavaScript y HTML del proyecto. |
| `output_location` | Nombre de la carpeta creada por cualquier proceso de compilación a partir de un empaquetador como webpack. El proceso de compilación debe crear esta carpeta y un subdirectorio en `app_location`. |
| `api_location` |Ubicación raíz de la aplicación Azure Functions hospedada por Azure Static Web Apps. Apunta a la carpeta raíz de todas las instancias de Azure Functions del proyecto, normalmente *api*. |

> [!NOTE]
> Los mensajes de error generados por una configuración de `api_location` incorrecta pueden compilarse correctamente, ya que Azure Static Web Apps no requiere código sin servidor.

## <a name="review-server-errors"></a>Revisión de errores del servidor

Use [Application Insights](../azure-monitor/app/app-insights-overview.md) para buscar mensajes de error en tiempo de ejecución. Si aún no tiene una instancia creada, consulte [Supervisión de Azure Static Web Apps](monitor.md). Application Insights registra el mensaje de error completo y el seguimiento de la pila generados por cada error.

> [!NOTE]
> Solo puede ver los mensajes de error que se generan después de la instalación de Application Insights.

1. Dentro de Azure Portal, abra el **grupo de recursos** asociado a la aplicación web estática.
1. Seleccione la instancia de Application Insights, que tiene el mismo nombre que la aplicación web estática (si se creó mediante los pasos anteriores).
1. En *Investigar*, seleccione **Errores**.
1. Desplácese hacia abajo hasta *Aumentar detalle* a la derecha.
1. En la esquina inferior derecha, en *Aumentar detalle*, un botón mostrará el número de operaciones con errores recientes.

    ![Captura de pantalla de los errores](./media/troubleshooting/app-insights-errors.png)

1. Seleccione el botón que dice **X operaciones** para abrir un panel que muestra las operaciones con errores recientes.

    ![Captura de pantalla de las operaciones](./media/troubleshooting/app-insights-operations.png)

1. Para explorar un error, seleccione uno de la lista.

    ![Captura de pantalla de detalles del error](./media/troubleshooting/app-insights-details.png)

## <a name="environment-variables"></a>Variables de entorno

Muchas aplicaciones web usan variables de entorno para almacenar configuraciones confidenciales o específicas del entorno. Azure Static Web Apps admite variables de entorno gracias a la configuración de aplicaciones.

La configuración de aplicaciones son pares clave-valor que establecen variables de entorno para la aplicación. Las variables están disponibles para la aplicación usando la misma sintaxis habitual para acceder a las variables de entorno.

En la implementación, compruebe que todas las variables de entorno se establecen como configuración de la aplicación.

1. Abra la aplicación web estática en Azure Portal.
1. Seleccione **Configuración**. La pantalla *Configuración* muestra la lista de todas las opciones de configuración de la aplicación.

    ![Captura de pantalla de Configuración en una aplicación web estática](media/troubleshooting/app-settings.png)

Siga estos pasos para agregar una nueva variable.

1. Seleccione **Agregar**.
1. Establezca el **nombre**.
1. Establezca el **valor**.
1. Seleccione **Aceptar**.
1. Seleccione **Guardar**.
