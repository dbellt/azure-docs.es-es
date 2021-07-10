---
title: 'Audio Content Creation: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Audio Content Creation es una herramienta en línea que le permite personalizar y ajustar la salida de texto a voz de Microsoft para sus aplicaciones y productos.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: trbye
ms.openlocfilehash: 94e060176b921529a42e28e8c735b0f1cdda5293
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110075085"
---
# <a name="improve-synthesis-with-the-audio-content-creation-tool"></a>Mejora de la síntesis con la herramienta Audio Content Creation

[Creación de contenido de audio](https://aka.ms/audiocontentcreation) es una herramienta eficaz y fácil de usar que permite crear contenido de audio muy natural para diversos escenarios como audiolibros, retransmisión de noticias, narraciones en vídeo y bots de chat. Con Creación de contenido de audio, puede ajustar las voces de texto a voz y diseñar experiencias de audio personalizadas de manera eficaz y rentable.

La herramienta se basa en el [lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md). Le permite ajustar los atributos de salida de texto a voz en tiempo real o la síntesis de lotes como, por ejemplo, caracteres de voz, estilos de voz, velocidad de habla, pronunciación y prosodia.

Puede acceder fácilmente a más de 150 voces pregeneradas en más de 60 idiomas diferentes, incluidas las voces TTS neuronales de última generación o su voz personalizada si ha creado una.

Consulte el [tutorial de vídeo](https://youtu.be/ygApYuOOG6w) para Audio Content Creation.

## <a name="how-to-get-started"></a>Primeros pasos

Creación de contenido de audio es una herramienta gratuita, pero pagará por el servicio Voz de Azure que consuma. Para trabajar con la herramienta, debe iniciar sesión con una cuenta de Azure y crear un recurso de voz. Para cada cuenta de Azure, tiene cuotas de voz gratis mensuales que incluyen 500 000 caracteres para voces TTS neuronales (al mes), 5 millones de caracteres para voces estándar y personalizadas (al mes) y un servicio de hospedaje de puntos de conexión de voz personalizado (al mes). La cantidad asignada mensualmente suele ser suficiente para un pequeño equipo de creación de contenido de alrededor de 3-5 personas. Estos son los pasos para crear una cuenta de Azure y obtener un recurso de voz.

### <a name="step-1---create-an-azure-account"></a>Paso 1: Creación de una cuenta de Azure

Para trabajar con Creación de contenido de audio, debe tener una [cuenta de Microsoft](https://account.microsoft.com/account) y una [cuenta de Azure](https://azure.microsoft.com/free/ai/). Siga estas instrucciones para [configurar la cuenta](./overview.md#try-the-speech-service-for-free).

[Azure Portal](https://portal.azure.com/) es el lugar centralizado para que administre su cuenta de Azure. Puede crear el recurso de voz, administrar el acceso al producto y supervisar todo, desde aplicaciones web sencillas hasta implementaciones complejas en la nube.

### <a name="step-2---create-a-speech-resource"></a>Paso 2: Creación de un recurso de voz

Después de registrarse para obtener la cuenta de Azure, debe crear un recurso de voz en su cuenta de Azure para acceder a los servicios de voz. Consulte las instrucciones para [crear un recurso de voz](./overview.md#create-the-azure-resource).

La implementación del recurso de voz nuevo puede tardar unos instantes. Una vez completada la implementación, puede iniciar el recorrido de creación de contenido de audio.

 > [!NOTE]
   > Si tiene previsto usar voces neuronales, asegúrese de crear el recurso en [una región que admita este tipo de voces](regions.md#neural-and-standard-voices).

### <a name="step-3---log-into-the-audio-content-creation-with-your-azure-account-and-speech-resource"></a>Paso 3: Inicio de sesión en Creación de contenido de audio con la cuenta de Azure y el recurso de voz

1. Después de obtener la cuenta de Azure y el recurso de voz, puede iniciar sesión en [Creación de contenido de audio](https://aka.ms/audiocontentcreation) haciendo clic en **Empezar**.
2. En la página principal se enumeran todos los productos de Speech Studio. Haga clic en **Creación de contenido de audio** para iniciar.
3. Aparecerá la página **Bienvenido a Speech Studio** para configurar el servicio de Voz. Seleccione la suscripción de Azure y el recurso de Voz en el que desea trabajar. Haga clic en **Usar recurso** para completar la configuración. Cuando inicie sesión en la herramienta Creación de contenido de audio la próxima vez, le vincularemos directamente a los archivos de trabajo de audio del recurso de voz actual. Puede comprobar los detalles y el estado de las suscripciones de Azure en [Azure Portal](https://portal.azure.com/). Si no tiene un recurso de Voz disponible y es el propietario o administrador de una suscripción de Azure, también puede crear un nuevo recurso de Voz en Speech Studio; para ello, haga clic en **Crear un nuevo recurso**. Si tiene un rol de usuario para una determinada suscripción de Azure, es posible que no tenga permisos para crear un nuevo recurso de Voz. Póngase en contacto con el administrador para obtener acceso al recurso de Voz. 
4. Puede modificar su recurso de voz en cualquier momento con la opción **Configuración**, que se encuentra en la navegación superior.
5. Si desea cambiar de directorio, vaya a **Configuración** o su perfil para hacerlo. 

## <a name="how-to-use-the-tool"></a>Uso de la herramienta

En este diagrama se muestran los pasos necesarios para ajustar las salidas de texto a voz. Use los siguientes vínculos para obtener más información sobre cada paso.

:::image type="content" source="media/audio-content-creation/audio-content-creation-diagram.jpg" alt-text="Un diagrama de los pasos necesarios para ajustar las salidas de texto a voz":::

1. Elija el recurso de voz en el que desea trabajar.
2. [Creación de un archivo de ajuste de audio](#create-an-audio-tuning-file) con texto sin formato o scripts SSML. Escriba o cargue el contenido en Creación de contenido de audio.
3. Elija la voz y el lenguaje del contenido del script. Audio Content Creation incluye todas las [voces de texto a voz de Microsoft](language-support.md#text-to-speech). Puede usar voces estándar, neuronales o una propia personalizada.
   > [!NOTE]
   > El acceso controlado está disponible para las voces neuronales personalizadas, que le permiten crear voces de alta definición similares a una voz natural. Para obtener más información, consulte [Proceso de acceso controlado](./text-to-speech.md).

4. Seleccione el contenido que desea obtener en vista previa y haga clic en el icono de **reproducción** (un triángulo) para obtener una vista previa de la salida de síntesis predeterminada. Tenga en cuenta que si realiza algún cambio en el texto, debe hacer clic en el icono **Detener** y, a continuación, volver a hacer clic en el icono de **reproducción** para volver a generar el audio con los scripts modificados. 
5. Mejore la salida mediante el ajuste de la pronunciación, las interrupciones, el tono, la velocidad, la entonación, el estilo de voz, etc. Para obtener una lista completa de opciones, consulte [Lenguaje de marcado de síntesis de voz](speech-synthesis-markup.md). En este [vídeo](https://youtu.be/ygApYuOOG6w) se muestra cómo ajustar la salida de voz con Audio Content Creation.
6. Guarde y [exporte el audio optimizado](#export-tuned-audio). Cuando guarde la pista de ajuste en el sistema, podrá seguir trabajando e iterar en la salida. Cuando esté satisfecho con el resultado, puede crear una tarea de creación de audio con la característica de exportación. Puede observar el estado de la tarea de exportación y descargar la salida para usarla con sus aplicaciones y productos.

## <a name="create-an-audio-tuning-file"></a>Creación de un archivo de ajuste de audio

Hay dos maneras de obtener el contenido de la herramienta Audio Content Creation.

**Opción 1:**

1. Haga clic en **Nuevo** > **Archivo** para crear un nuevo archivo de ajuste de audio.
2. Escriba o pegue el contenido en la ventana de edición. En número máximo de caracteres de cada archivo es 20 000. Si el script tiene más de 20 000 caracteres, puede usar la opción 2 para dividir automáticamente el contenido en varios archivos.
3. No olvide guardar.

**Opción 2:**

1. Haga clic en **Cargar** para importar uno o varios archivos de texto. Se admiten el texto sin formato y SSML. Si el archivo de script tiene más de 20 000 caracteres, divida el archivo por párrafos, por caracteres o por expresiones regulares.
3. Al cargar los archivos de texto, asegúrese de que el archivo cumple estos requisitos.

   | Propiedad | Valor/notas |
   |----------|---------------|
   | Formato de archivo | Texto sin formato (.txt)<br/> Texto en SSML (.txt)<br/> No se admiten los archivos ZIP. |
   | Formato de codificación | UTF-8 |
   | Nombre de archivo | Cada archivo debe tener un nombre único. No se admiten los duplicados. |
   | Longitud del texto | La limitación de caracteres del archivo de texto es de 20 000. Si los archivos superan esa limitación, divida los archivos con las instrucciones de la herramienta. |
   | Restricciones de SSML | Cada archivo SSML solo puede contener un único fragmento de SSML. |

**Ejemplo de texto sin formato**

```txt
Welcome to use Audio Content Creation to customize audio output for your products.
```

**Ejemplo de texto en SSML**

```xml
<speak xmlns="http://www.w3.org/2001/10/synthesis" xmlns:mstts="http://www.w3.org/2001/mstts" version="1.0" xml:lang="en-US">
    <voice name="Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)">
    Welcome to use Audio Content Creation <break time="10ms" />to customize audio output for your products.
    </voice>
</speak>
```

## <a name="export-tuned-audio"></a>Exportación de audio ajustado

Una vez que haya revisado la salida de audio y esté satisfecho con la optimización y el ajuste, puede exportar el audio.

1. Haga clic en **Exportar** para crear una tarea de creación de audio. Se recomienda **Export to Audio Library (Exportar a biblioteca de audio)** , ya que admite la salida de audio larga y la experiencia de salida de audio completa. También puede descargar el audio directamente en el disco local, pero solo estarán disponibles los primeros 10 minutos.
2. Elija el formato de salida para el audio ajustado. A continuación se ofrece una lista de formatos admitidos y frecuencias de muestreo.
3. Puede ver el estado de la tarea en la pestaña **Export task** (tarea de exportación). Si se produce un error en la tarea, consulte la página de información detallada para obtener un informe completo.
4. Una vez completada la tarea, el audio está disponible para su descarga en la pestaña **Audio Library** (biblioteca de audio).
5. Haga clic en **Descargar**. Ahora está listo para usar el audio ajustado personalizado en sus aplicaciones o productos.

**Formatos de audio compatibles**

| Formato | Frecuencia de muestreo de 16 kHz | Frecuencia de muestreo de 24 kHz |
|--------|--------------------|--------------------|
| wav | riff-16khz-16bit-mono-pcm | riff-24khz-16bit-mono-pcm |
| mp3 | audio-16khz-128kbitrate-mono-mp3 | audio-24khz-160kbitrate-mono-mp3 |

## <a name="how-to-addremove-audio-content-creation-users"></a>Incorporación o eliminación de usuarios de Creación de contenido de audio

Si más de un usuario desea usar Creación de contenido de audio, puede conceder acceso de usuario a la suscripción de Azure y al recurso de voz. Si agrega un usuario a una suscripción de Azure, este puede acceder a todos los recursos de esa suscripción. Pero si agrega solo un usuario a un recurso de voz, este solo tendrá acceso a dicho recurso y no podrá acceder a otros recursos de esta suscripción de Azure. Un usuario con acceso al recurso de voz puede utilizar Creación de contenido de audio.

El usuario debe preparar una [cuenta Microsoft](https://account.microsoft.com/account). Si el usuario no tiene una cuenta Microsoft, puede crear una en solo unos minutos. El usuario puede utilizar el correo electrónico y el vínculo existentes como una cuenta Microsoft, o crear un nuevo correo electrónico de Outlook como cuenta Microsoft.


### <a name="add-users-to-a-speech-resource"></a>Incorporación de usuarios a un recurso de voz

Siga estos pasos para agregar un usuario a un recurso de voz para que pueda usar Creación de contenido de audio.

1. Busque **Cognitive Services** en [Azure Portal](https://portal.azure.com/) y seleccione el recurso de voz al que desea agregar usuarios.
2. Haga clic en **Control de acceso (IAM).** Haga clic en la pestaña **Asignaciones de roles** para ver todas las asignaciones de roles para esta suscripción.
    :::image type="content" source="media/audio-content-creation/access-control-roles.png" alt-text="Pestaña Asignación de roles":::
3. Haga clic en **Agregar** > **Agregar asignación de roles** para abrir el panel Agregar asignación de roles. En la lista desplegable Rol, seleccione el rol **Usuario de Cognitive Services**. Si desea conceder al usuario la propiedad de este recurso de voz, puede seleccionar el rol **Propietario**.
4. Seleccione un usuario de la lista. Si no ve el usuario en la lista, puede escribir en el cuadro Seleccionar para buscar nombres para mostrar y direcciones de correo electrónico en el directorio. Si el usuario no está en este directorio, puede especificar la [cuenta de Microsoft](https://account.microsoft.com/account) (que es de confianza para Azure Active Directory).
5. Haga clic en **Guardar** para asignar el rol. El usuario recibirá una invitación por correo electrónico. Para aceptar la invitación, haga clic en **Aceptar invitación** > **Aceptar para unirse a Azure** en el correo electrónico. A continuación, se redirigirá al usuario a Azure Portal. El usuario no tiene que realizar ninguna otra acción en Azure Portal.
6. Transcurridos unos instantes, al usuario se le asigna el rol Usuario de Cognitive Services en el ámbito del recurso de voz. El usuario puede visitar o actualizar la página [Creación de contenido de audio](https://aka.ms/audiocontentcreation) y elegir el recurso de voz para empezar. 

    :::image type="content" source="media/audio-content-creation/add-role-first.png" alt-text="Cuadro de diálogo Agregar rol":::


Los usuarios que están en el mismo recurso de voz verán el trabajo de los demás en el estudio de Creación de contenido de audio. Si desea que cada usuario individual tenga un área de trabajo única y privada en Creación de contenido de audio, [cree un nuevo recurso de voz](#step-2---create-a-speech-resource) para cada usuario y conceda a cada usuario el acceso exclusivo al recurso de voz.

### <a name="remove-users-from-a-speech-resource"></a>Eliminación de usuarios de un recurso de voz

1. Busque **Cognitive Services** en Azure Portal y seleccione el recurso de voz del que desea eliminar usuarios.
2. Haga clic en **Control de acceso (IAM).** Haga clic en la pestaña **Asignaciones de roles** para ver todas las asignaciones de roles de este recurso de voz.
3. Seleccione los usuarios que desea eliminar y haga clic en **Quitar** > **Aceptar**.
    :::image type="content" source="media/audio-content-creation/remove-user.png" alt-text="Botón Quitar":::

### <a name="enable-users-to-grant-access"></a>Permitir a los usuarios conceder acceso

Si desea que uno de los usuarios proporcione acceso a otros usuarios, debe conceder al usuario el rol de propietario del recurso de voz y establecer el usuario como lector de directorios de Azure.
1. Agregue al usuario como propietario del recurso de voz. Consulte [Incorporación de usuarios a un recurso de voz](#add-users-to-a-speech-resource).
    :::image type="content" source="media/audio-content-creation/add-role.png" alt-text="Campo Propietario de rol":::
1. En [Azure Portal](https://portal.azure.com/), seleccione el menú contraído de la parte superior izquierda. Haga clic en **Azure Active Directory** y, luego, en **Usuarios**.
1. Busque en el cuenta de Microsoft del usuario y vaya a la página de detalles del usuario. Haga clic en **Roles asignados**.
1. Haga clic en **Agregar asignaciones** -> **Lectores de directorios**.

## <a name="see-also"></a>Consulte también

* [Long Audio API](./long-audio-api.md)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Speech Studio](https://speech.microsoft.com)
