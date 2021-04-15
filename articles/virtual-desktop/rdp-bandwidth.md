---
title: Requisitos de ancho de banda del protocolo de escritorio remoto de Windows Virtual Desktop (Azure)
titleSuffix: Azure
description: Descripción de los requisitos de ancho de banda de RDP para Windows Virtual Desktop.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: 6fe12c985f5d9a519380d1d9b5a7d6c2820630c6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105932342"
---
# <a name="remote-desktop-protocol-rdp-bandwidth-requirements"></a>Requisitos de ancho de banda del protocolo de escritorio remoto (RDP)

Protocolo de escritorio remoto (RDP) es una tecnología sofisticada que usa varias técnicas para perfeccionar la entrega de gráficos remotos del servidor al dispositivo cliente. En función del caso de uso, la disponibilidad de los recursos informáticos y el ancho de banda de red, el RDP ajusta dinámicamente varios parámetros para ofrecer la mejor experiencia de usuario.

El protocolo de escritorio remoto multiplexa varios canales virtuales dinámicos (DVC) en un solo canal de datos enviado a través de diferentes transportes de red. Existen canales virtuales dinámicos independientes para gráficos remotos, entrada, redireccionamiento de dispositivos, impresión, etc. Los asociados de Windows Virtual Desktop también pueden usar sus extensiones que emplean interfaces de DVC.

La cantidad de datos enviados a través del RDP depende de la actividad del usuario. Por ejemplo, un usuario puede trabajar con contenido textual básico durante la mayor parte de la sesión y consumir un ancho de banda mínimo, pero generar una copia impresa de un documento de 200 páginas en la impresora local. Este trabajo de impresión usará una significativa cantidad de ancho de banda de red.

Al usar una sesión remota, el ancho de banda disponible de la red afecta drásticamente a la calidad de la experiencia. Las distintas aplicaciones y resoluciones de pantalla requieren configuraciones de red diferentes, por lo que es esencial asegurarse de que la configuración de la red satisfaga sus necesidades.

## <a name="estimating-bandwidth-utilization"></a>Cálculo de utilización del ancho de banda

RDP usa varios algoritmos de compresión para los diferentes tipos de datos. La tabla siguiente sirve como guía para calcular las transferencias de datos:

| Tipo de datos | Dirección | Cómo se realiza el cálculo |
|---|---|---|
| Gráficos remotos | Host de sesión a cliente | [Consulte las instrucciones detalladas](#estimating-bandwidth-used-by-remote-graphics) |
| Latidos | Ambas direcciones | Aproximadamente 20 bytes cada 5 segundos  |
| Entrada | Cliente a host de sesión | La cantidad de datos se basa en la actividad del usuario, menos de 100 bytes en la mayoría de las operaciones.  |
| Transferencias de archivos | Ambas direcciones | Las transferencias de archivos usan la compresión en masa. Use la compresión en formato .zip para la aproximación. |
| Impresión | Host de sesión a cliente | La transferencia de trabajos de impresión depende del controlador y del uso de la compresión en masa, use la compresión en formato .zip para la aproximación. |

Otros escenarios pueden cambiar sus requisitos de ancho de banda en función de cómo los uses, por ejemplo:

* Conferencias de audio o vídeo
* Comunicación en tiempo real
* Streaming de vídeo 4K

## <a name="estimating-bandwidth-used-by-remote-graphics"></a>Cálculo del ancho de banda que usan los gráficos remotos

Es difícil predecir el uso del ancho de banda que realiza el escritorio remoto. Las actividades del usuario generan la mayor parte del tráfico del escritorio remoto. Cada usuario es único y las diferencias en sus patrones de trabajo pueden cambiar significativamente el uso de la red.

La mejor manera de conocer los requisitos de ancho de banda es supervisar las conexiones de usuario reales. La supervisión pueden realizarla tanto los contadores de rendimiento integrados como el equipo de red.

Sin embargo, en muchos casos, para calcular el uso de la red es preciso conocer el funcionamiento del protocolo de escritorio remoto y analizar los patrones de trabajo de los usuarios.

El protocolo remoto entrega los gráficos generados por el servidor remoto para que se muestren en un monitor local. Más concretamente, proporciona el mapa de bits del escritorio completamente compuesto en el servidor.
Aunque el envío de un mapa de bits del escritorio parece inicialmente una tarea sencilla, requiere una cantidad significativa de recursos. Por ejemplo, una imagen de escritorio de 1080 puntos en su forma sin comprimir tiene un tamaño aproximado de 8 MB. La visualización de esta imagen en el monitor conectado localmente con una modesta frecuencia de actualización de pantalla de 30 Hz requiere un ancho de banda de aproximadamente 237 MB/s.

Para reducir la cantidad de datos que se transfieren a través de la red, RDP usa una combinación de varias técnicas, incluidas, entre otras:

* Optimizaciones de la velocidad de fotogramas
* Clasificación del contenido de la pantalla
* Códecs específicos del contenido
* Codificación de imagen progresiva
* Almacenamiento en caché del lado cliente

Para conocer mejor los gráficos remotos, tenga en cuenta lo siguiente:

* Cuanto mejores sean los gráficos, más ancho de banda utilizará
  * El texto, los elementos de la interfaz de usuario de la ventana y las áreas de color sólido son los elementos que menos ancho de banda consumen.
  * Las imágenes naturales son los elementos que más ancho de banda utilizan. Pero el almacenamiento en la caché en el lado del cliente ayuda a reducirlo.
* Solo se transmiten las partes de la pantalla que han cambiado. Si no hay actualizaciones visibles en la pantalla, no se envían actualizaciones.
* La reproducción de vídeo y de otro contenido con una velocidad de fotogramas alta es esencialmente una presentación de imágenes. RDP usa de forma dinámica los códecs de vídeo adecuados para entregarlos con una velocidad de fotogramas cercana a la original. Sin embargo, no dejan de ser gráficos y siguen siendo el elemento que más ancho de banda usa.
* El tiempo de inactividad en el escritorio remoto significa que no hay actualizaciones de la pantalla, o que las que hay son mínimas; por lo tanto, el uso de la red es mínimo durante los tiempos de inactividad.
* Cuando se minimiza la ventana del cliente de escritorio remoto, no se envía ninguna actualización gráfica desde el host de sesión.

Ten en cuenta que el estrés que se aplica a la red depende de la resolución de pantalla y la velocidad de fotogramas de salida de la carga de trabajo de la aplicación. Si la velocidad de fotogramas o la resolución de pantalla aumentan, el requisito de ancho de banda también lo hará. Por ejemplo, una carga de trabajo ligera con una pantalla de alta resolución requiere más ancho de banda disponible que una carga de trabajo ligera con una resolución normal o baja. Diferentes resoluciones de pantalla requieren diferentes anchos de banda disponibles.

La siguiente tabla sirve de guía para calcular los datos que usan los distintos escenarios de gráficos. Estos números se aplican a la configuración con un único monitor con una resolución de 1920x1080 y con el modo de gráficos predeterminado y el modo de gráficos H.264/AVC 444.

| Escenario | Modo predeterminado | Modo H.264/AVC 444 | Miniatura | Descripción del escenario |
|:---|---:|---:|---|---|
| Inactivo | 0,3 Kbps | 0,3 Kbps |:::image type="content" source="media/idle.png" alt-text="Captura de pantalla de la conexión inactiva":::| El usuario ha detenido su trabajo y no hay actualizaciones de pantalla activas |
| Microsoft Word | 100-150 Kbps | 200-300 Kbps |:::image type="content" source="media/word.gif" alt-text="Animación de Microsoft Word":::| El usuario trabaja activamente con Microsoft Word, escribe, pega gráficos y cambia de un documento a otro |
| Microsoft Excel | 150-200 Kbps | 400-500 Kbps |:::image type="content" source="media/excel.gif" alt-text="Animación de Microsoft Excel":::| El usuario trabaja activamente con Microsoft Excel, se actualizan al mismo tiempo varias celdas con fórmulas y gráficos |
| Microsoft PowerPoint | 4-4,5 MBps | 1,6-1,8 MBps |:::image type="content" source="media/powerpoint.gif" alt-text="Animación de Microsoft PowerPoint":::| El usuario trabaja activamente con Microsoft PowerPoint, escribe y pega. El usuario también modifica gráficos enriquecidos y usa efectos de transición de diapositivas |
| Exploración web | 6-6,5 MBps | 0,9-1 MBps |:::image type="content" source="media/web.gif" alt-text="Animación de exploración web":::| El usuario trabaja activamente con un sitio web con muchos gráficos que contiene varias imágenes, tanto estáticas como animadas. El usuario se desplaza por las páginas horizontal y verticalmente |
| Galería de imágenes | 3,3-3,6 MBps | 0,7-0,8 MBps |:::image type="content" source="media/image-gallery.gif" alt-text="Animación de la galería de imágenes":::| El usuario trabaja activamente con la aplicación de la galería de imágenes. exploración, zoom, cambio de tamaño y rotación de imágenes |
| Reproducción de vídeo | 8,5-9,5 MBps | 2,5-2,8 MBps |:::image type="content" source="media/video.gif" alt-text="Animación de reproducción de vídeo":::| El usuario ve un vídeo de 30 FPS que consume la mitad de la pantalla |
| Reproducción de vídeo a pantalla completa | 7,5-8,5 MBps | 2,5-3,1 MBps |:::image type="content" source="media/fullscreen-video.gif" alt-text="Animación de reproducción de vídeo a pantalla completa":::| El usuario ve un vídeo de 30 FPS que ha maximizado a pantalla completa |

## <a name="dynamic-bandwidth-allocation"></a>Asignación dinámica de ancho de banda

El protocolo de escritorio remoto es un protocolo moderno diseñado para ajustarse de forma dinámica a las cambiantes condiciones de la red.
En lugar de usar límites estrictos en el uso de ancho de banda, RDP usa la detección de red continua que supervisa activamente el ancho de banda de red disponible y el tiempo de ida y vuelta de los paquetes. En función de las conclusiones, RDP selecciona dinámicamente las opciones de codificación de gráficos y asigna ancho de banda para la redirección de dispositivos y otros canales virtuales.  
Esta tecnología permite que RDP use toda la canalización de red cuando esté disponible y retroceda rápidamente si la red se necesita para otra cosa.
RDP lo detecta y ajusta la calidad de la imagen, la velocidad de fotogramas o los algoritmos de compresión si otras aplicaciones solicitan la red.

## <a name="limit-network-bandwidth-use-with-throttle-rate"></a>Limitación del uso del ancho de banda de la red con la velocidad límite

En la mayoría de los escenarios, no es necesario limitar el uso del ancho de banda, ya que ello puede afectar a la experiencia del usuario. Sin embargo, en las redes restringidas, puede que desee limitar el uso de la red. Otro ejemplo son las redes alquiladas cuyo costo depende de la cantidad de tráfico que se usa.

En tales casos, para limitar el tráfico de red de salida del RDP, se especifica una velocidad limite en la directiva de QoS.

  >[!NOTE]
  > [Asegúrese de que RDP Shortpath está habilitado](./shortpath.md) (la limitación de velocidad no se admite para el transporte de conexión inversa).

### <a name="implement-throttle-rate-limiting-on-session-host-using-group-policy"></a>Implementación de una limitación de velocidad en un host de sesión mediante una directiva de grupo

Puede usar la calidad de servicio (QoS) basada en directivas en directiva de grupo para establecer la velocidad límite predefinida.

Para crear una directiva de QoS para hosts de sesión unidos a un dominio, lo primero que debe hacer es iniciar sesión en un equipo en el que se haya instalado la Administración de directivas de grupo. Abra la Administración de directivas de grupo (seleccione Inicio, Herramientas administrativas y seleccione Administración de directivas de grupo) y, después, siga estos pasos:

1. En Administración de directivas de grupo, busque el contenedor en el que se debe crear la nueva directiva. Por ejemplo, si todos los equipos host de sesión se encuentran en una unidad organizativa denominada **Hosts de sesión**, la nueva directiva se debe crear en la unidad organizativa Hosts de sesión.

2. Haga clic con el botón derecho en el contenedor adecuado y, después, seleccione **Crear un GPO en este dominio y vincularlo aquí**.

3. En el cuadro de diálogo **Nuevo GPO**, escriba el nombre del nuevo objeto de directiva del grupo en el cuadro **Nombre** y seleccione **Aceptar**.

4. Haga clic con el botón derecho en la directiva recién creada y seleccione **Editar**.

5. En el Editor de administración de directivas de grupo, expanda **Configuración del equipo** y **Configuración de Windows**, haga clic con el botón derecho en **QoS basada en directiva** y, después, seleccione **Crear nueva directiva**.

6. En el cuadro de diálogo **QoS basada en directiva**, en la página de apertura, escriba el nombre de la nueva directiva en el cuadro **Nombre**. Seleccione **Specify Outbound Throttle Rate** (Especificar velocidad de regulador de carga de trabajo de salida), establezca el valor requerido y seleccione **Next** (Siguiente).

7. En la página siguiente, seleccione **Only applications with this executable name** (Solo aplicaciones con su nombre ejecutable), escriba el nombre **svchost.exe** y, después, seleccione **Next** (Siguiente). Esta configuración indica a la directiva que solo dé prioridad al tráfico coincidente del servicio de Escritorio remoto.

8. En la tercera página, asegúrese de que tanto **Any source IP address** (Cualquier dirección IP de origen) como **Any destination IP address** (Cualquier dirección IP de destino) estén seleccionadas. Seleccione **Siguiente**. Estos dos valores garantizan que los paquetes se administrarán independientemente de cuál sea el equipo (dirección IP) que envíe los paquetes y cuál (dirección IP) el que los va a recibir.

9. En la página cuatro, seleccione **UDP** en la lista desplegable **Select the protocol this QoS policy applies to** (Seleccionar el protocolo al que se aplica esta directiva de QoS).

10. En el título **Specify the source port number** (Especificar el número de puerto de origen), seleccione **From this source port or range** (Desde este intervalo o número de puerto de origen). En el cuadro de texto que lo acompaña, escriba **3390**. Seleccione **Finalizar**.

Las nuevas directivas que ha creado no surtirán efecto hasta que la directiva de grupo se haya actualizado en los equipos host de sesión. Aunque la directiva de grupo se actualiza sola periódicamente, siga estos pasos para forzar una actualización inmediata:

1. En cada host de sesión para el que desee actualizar directiva de grupo, abra un símbolo del sistema como administrador (*Ejecutar como administrador*).

2. En el símbolo del sistema, escriba:

   ```console
   gpupdate /force
   ```

### <a name="implement-throttle-rate-limiting-on-session-host-using-powershell"></a>Implementación de una limitación de velocidad en un host de sesión mediante PowerShell

Puede establecer la velocidad límite para RDP Shortpath mediante el siguiente cmdlet de PowerShell:

```powershell
New-NetQosPolicy -Name "RDP Shortpath" -AppPathNameMatchCondition "svchost.exe" -IPProtocolMatchCondition UDP -IPSrcPortStartMatchCondition 3390 -IPSrcPortEndMatchCondition 3390 -DSCPAction 46 -NetworkProfile All
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los requisitos de ancho de banda de Windows Virtual Desktop, consulte la [descripción de los requisitos del protocolo de escritorio remoto (RDP) para Windows Virtual Desktop](rdp-bandwidth.md).
* Para más información sobre la conectividad de red de Windows Virtual Desktop, consulte la [descripción de la conectividad de red de Windows Virtual Desktop](network-connectivity.md).
* Para empezar a trabajar con calidad de servicio (QoS) para Windows Virtual Desktop, consulte [Implementación de calidad de servicio (QoS) para Windows Virtual Desktop](rdp-quality-of-service-qos.md).
