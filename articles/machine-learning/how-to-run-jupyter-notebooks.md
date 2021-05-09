---
title: Ejecución de cuadernos de Jupyter en el área de trabajo
titleSuffix: Azure Machine Learning
description: Aprenda a ejecutar un cuaderno de Jupyter sin dejar el área de trabajo en Azure Machine Learning Studio.
services: machine-learning
author: abeomor
ms.author: osomorog
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 01/19/2021
ms.openlocfilehash: 0a95d95842d0b361a1a276566b01b7ea735c4670
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107952087"
---
# <a name="run-jupyter-notebooks-in-your-workspace"></a>Ejecución de cuadernos de Jupyter Notebook en el área de trabajo

Aprenda a ejecutar sus cuadernos de Jupyter directamente en el área de trabajo en Azure Machine Learning Studio. Aunque puede iniciar [Jupyter](https://jupyter.org/) o [JupyterLab](https://jupyterlab.readthedocs.io), también puede editar y ejecutar los cuadernos sin tener que salir del área de trabajo.

Para obtener información sobre cómo crear y administrar archivos, incluidos los cuadernos, consulte [Creación y administración de archivos en el área de trabajo](how-to-manage-files.md).

> [!IMPORTANT]
> Las características marcadas como (versión preliminar) se ofrecen sin Acuerdo de nivel de servicio y no se recomiendan para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
* Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="edit-a-notebook"></a>Edición de un cuaderno

Para editar un cuaderno, abra cualquiera que esté situado en la sección **Archivos de usuario** de su área de trabajo. Haga clic en la celda que desee editar.  Si no tiene ningún cuaderno en esta sección, consulte [Creación y administración de archivos en el área de trabajo](how-to-manage-files.md).

Puede editar el cuaderno sin necesidad de conectarse a una instancia de proceso.  Cuando desee ejecutar las celdas en el cuaderno, seleccione o cree una instancia de proceso.  Si selecciona una instancia de proceso detenida, se iniciará automáticamente al ejecutar la primera celda.

Cuando se ejecuta una instancia de proceso, también puede usar la finalización de código, con la tecnología de [IntelliSense](https://code.visualstudio.com/docs/editor/intellisense), en cualquier cuaderno de Python.

También puede iniciar Jupyter o JupyterLab desde la barra de herramientas del cuaderno.  Azure Machine Learning no proporciona actualizaciones ni corrige errores de Jupyter o JupyterLab, ya que son productos de código abierto fuera de los límites del servicio de soporte técnico de Microsoft.

## <a name="focus-mode"></a>Modo de enfoque

Use el modo de enfoque para expandir la vista actual de forma que pueda centrarse en las pestañas activas. El modo de enfoque oculta el explorador de archivos de Notebooks.

1. En la barra de herramientas de la ventana del terminal, seleccione **Modo de enfoque** para activar el modo de enfoque. En función del ancho de la ventana, la herramienta puede encontrarse en el elemento de menú **...** de la barra de herramientas.
1. En el modo de enfoque, seleccione **Vista estándar** para volver a la vista estándar.

    :::image type="content" source="media/how-to-run-jupyter-notebooks/focusmode.gif" alt-text="Cambio entre el modo de enfoque y la vista estándar":::

## <a name="code-completion-intellisense"></a>Finalización de código (IntelliSense)

[IntelliSense](https://code.visualstudio.com/docs/editor/intellisense) es una ayuda de finalización de código que incluye una serie de características: enumerar miembros, información de parámetros, información rápida y completar palabra. Con solo unas cuantas pulsaciones de tecla, puede:
* Obtener información sobre el código que usa
* Realizar un seguimiento de los parámetros que escribe
* Agregar llamadas a propiedades y métodos 

### <a name="insert-code-snippets-preview"></a>Insertar fragmentos de código (versión preliminar)

Use **Ctrl+Space** para desencadenar los fragmentos de código de IntelliSense.  Desplácese por las sugerencias o comience a escribir el código que quiere insertar.  Una vez insertado el código, recorra los argumentos para personalizar el código para su propio uso.

:::image type="content" source="media/how-to-run-jupyter-notebooks/insert-snippet.gif" alt-text="Insertar un fragmento de código":::

Estos mismos fragmentos de código están disponibles al abrir el cuaderno en VS Code. Para obtener una lista completa de los fragmentos de código disponibles, consulte [Fragmentos de código de VS Code para Azure Machine Learning](https://github.com/Azure/azureml-snippets/blob/main/snippets/snippets.md).

Puede examinar y buscar en la lista de fragmentos de código mediante la barra de herramientas del cuaderno para abrir el panel de fragmentos de código.

:::image type="content" source="media/how-to-run-jupyter-notebooks/open-snippet-panel.png" alt-text="Abrir la herramienta del panel de fragmentos de código en la barra de herramientas del cuaderno":::

En el panel fragmentos de código, también puede enviar una solicitud para agregar nuevos fragmentos de código.

:::image type="content" source="media/how-to-run-jupyter-notebooks/propose-new-snippet.png" alt-text="El panel de fragmentos de código permite proponer un nuevo fragmento de código.":::

## <a name="clean-your-notebook-preview"></a>Limpieza del cuaderno (versión preliminar)

Durante la creación de un cuaderno, normalmente acaba con las celdas usadas para la exploración o la depuración de datos. La característica de *recopilación* le ayudará a crear un cuaderno limpio sin estas celdas extrañas.

1. Ejecute todas las celdas del cuaderno.
1. Seleccione la celda que contiene el código que desea que ejecute el nuevo cuaderno. Por ejemplo, el código que envía un experimento o quizás el código que registra un modelo.
1. Seleccione el icono de **recopilación** que aparece en la barra de herramientas de celda.
    :::image type="content" source="media/how-to-run-jupyter-notebooks/gather.png" alt-text="Captura de pantalla: seleccione el icono de recopilación":::
1. Escriba el nombre del nuevo cuaderno "recopilado".  

El nuevo cuaderno solo contiene celdas de código, con todas las celdas necesarias para generar los mismos resultados que la celda seleccionada para la recopilación.

## <a name="save-and-checkpoint-a-notebook"></a>Guardar y revisar un cuaderno

Azure Machine Learning crea un archivo de punto de comprobación cuando se crea un archivo *ipynb*.

En la barra de herramientas del cuaderno, seleccione el menú y luego, **Archivo&gt;Guardar y punto de control** para guardar manualmente el cuaderno y agregará un archivo de punto de control asociado con el cuaderno.

:::image type="content" source="media/how-to-run-jupyter-notebooks/file-save.png" alt-text="Captura de pantalla de la herramienta guardar en la barra de herramientas del cuaderno":::

Cada cuaderno se guarda automáticamente cada 30 segundos. El guardado automático actualiza solo el archivo *ipynb* inicial, no el archivo de puntos de control.
 
Seleccione **Puntos de control** en el menú del cuaderno para crear un punto de control con nombre y revertir el cuaderno a un punto de control guardado.

## <a name="export-a-notebook"></a>Exportación de un cuaderno

En la barra de herramientas del cuaderno, seleccione el menú y, a continuación, **Exportar como** para exportar el cuaderno en cualquiera de los tipos admitidos:

* Notebook
* Python
* HTML
* LaTeX

:::image type="content" source="media/how-to-run-jupyter-notebooks/export-notebook.png" alt-text="Exportación de un cuaderno al equipo":::

El archivo exportado se guarda en el equipo.

## <a name="run-a-notebook-or-python-script"></a>Ejecución de un cuaderno o un script de Python

Para ejecutar un cuaderno o un script de Python, primero tiene que conectarse a una [instancia de proceso](concept-compute-instance.md) en ejecución.

* Si no tiene una instancia de proceso, siga estos pasos para crear una:

    1. En la barra de herramientas del cuaderno o script, a la derecha de la lista desplegable Proceso, seleccione **+ Nuevo proceso**. En función del tamaño de la pantalla, es posible que se encuentre en el menú **...**
        :::image type="content" source="media/how-to-run-jupyter-notebooks/new-compute.png" alt-text="Creación de un nuevo proceso":::
    1. Asigne un nombre al proceso y elija el **Tamaño de la máquina virtual**. 
    1. Seleccione **Crear**.
    1. La instancia de proceso se conecta automáticamente al archivo.  Ahora puede ejecutar las celdas del cuaderno o el script de Python mediante la herramienta que se encuentra a la izquierda de la instancia de proceso.

* Si tiene una instancia de proceso detenida, seleccione **Start compute** (iniciar proceso) a la derecha de la lista desplegable Proceso. En función del tamaño de la pantalla, es posible que se encuentre en el menú **...**

    :::image type="content" source="media/how-to-run-jupyter-notebooks/start-compute.png" alt-text="Inicio de la instancia de proceso":::

Solo puede ver y usar las instancias de proceso que usted cree.  Los **Archivos de usuario** se almacenan de forma independiente de la máquina virtual y se comparten entre todas las instancias de proceso en el área de trabajo.

### <a name="view-logs-and-output"></a>Ver registros y resultados

Use los [widgets del cuaderno](/python/api/azureml-widgets/azureml.widgets) para ver el progreso de la ejecución y los registros. Un widget es asincrónico y proporciona actualizaciones hasta que finaliza el entrenamiento. Los widgets de Azure Machine Learning también se admiten en Jupyter y JupterLab.

:::image type="content" source="media/how-to-run-jupyter-notebooks/jupyter-widget.png" alt-text="Captura de pantalla: Widget "::: del cuaderno de Jupyter

## <a name="explore-variables-in-the-notebook"></a>Exploración de las variables del cuaderno

En la barra de herramientas del cuaderno, use la herramienta **Explorador de variables** para mostrar el nombre, el tipo, la longitud y los valores de ejemplo de todas las variables que se han creado en el cuaderno.

:::image type="content" source="media/how-to-run-jupyter-notebooks/variable-explorer.png" alt-text="Captura de pantalla: herramienta Explorador de variables":::

Seleccione la herramienta para mostrar la ventana del explorador de variables.

:::image type="content" source="media/how-to-run-jupyter-notebooks/variable-explorer-window.png" alt-text="Captura de pantalla: ventana del explorador de variables":::

## <a name="navigate-with-a-toc"></a>Navegación con una tabla de contenido

En la barra de herramientas del cuaderno, use la herramienta **Tabla de contenido** para mostrar u ocultar la misma.  Inicie una celda de Markdown con un encabezado para agregarla a la tabla de contenido. Haga clic en una entrada de la tabla para desplazarse a esa celda en el cuaderno.  

:::image type="content" source="media/how-to-run-jupyter-notebooks/table-of-contents.png" alt-text="Captura de pantalla: tabla de contenido del cuaderno":::

## <a name="change-the-notebook-environment"></a>Cambio del entorno del cuaderno.

La barra de herramientas del cuaderno le permite cambiar el entorno en el que se ejecuta el mismo.  

Estas acciones no cambiarán el estado del cuaderno o los valores de las variables del mismo:

|Acción  |Resultado  |
|---------|---------| --------|
|Detención del kernel     |  Detiene cualquier celda en ejecución. Al ejecutar una celda, se reiniciará automáticamente el kernel. |
|Desplazamiento a otra sección del área de trabajo     |     Se detienen las celdas en ejecución. |

Estas acciones restablecerán el estado del cuaderno y todas las variables del mismo.

|Acción  |Resultado  |
|---------|---------| --------|
| Cambio del kernel | Notebook usa un nuevo kernel |
| Cambio de proceso    |     Notebook utiliza automáticamente el nuevo proceso. |
| Restablecimiento del proceso | Se inicia de nuevo cuando se intenta ejecutar una celda. |
| Detención del proceso     |    No se ejecutarán las celdas.  |
| Apertura del cuaderno en Jupyter o JupyterLab     |    Notebook se abre en una nueva pestaña.  |

## <a name="add-new-kernels"></a>Incorporación de nuevos kernels

[Use el terminal](how-to-access-terminal.md#add-new-kernels) para crear y agregar kernels nuevos a la instancia de proceso. El cuaderno buscará automáticamente todos los kernels de Jupyter instalados en la instancia de proceso conectada.

Use la lista desplegable Kernel de la derecha para cambiar a cualquiera de los kernels instalados.  


### <a name="status-indicators"></a>Indicadores de estado

Un indicador situado junto al elemento de la lista desplegable **Proceso** muestra su estado.  El estado también se muestra en el mismo elemento de la lista desplegable.  

|Color |Estado de proceso |
|---------|---------| 
| Verde | Proceso en ejecución |
| Rojo |Error de proceso | 
| Negro | Proceso detenido |
|  Azul claro |Proceso creando, iniciando, reiniciando, configurando |
|  Gris |Proceso eliminando, deteniendo |

Un indicador situado junto al elemento de lista desplegable **Kernel** muestra su estado.

|Color |Estado del kernel |
|---------|---------|
|  Verde |Kernel conectado, inactivo, ocupado|
|  Gris |Kernel no conectado |

## <a name="find-compute-details"></a>Detalles de proceso

Busque los detalles sobre las instancias de proceso en la página **Proceso** de [Studio](https://ml.azure.com).

## <a name="useful-keyboard-shortcuts"></a>Métodos abreviados de teclado útiles
De forma similar a los cuadernos de Jupyter Notebook, los cuadernos de Azure Machine Learning Studio tienen una interfaz de usuario modal. El teclado realiza diferentes acciones en función del modo en que se encuentre la celda del cuaderno. Los cuadernos de Azure Machine Learning Studio admiten los siguientes dos modos para una celda de código determinada: el modo de comando y el de edición.

### <a name="command-mode-shortcuts"></a>Métodos abreviados del modo de comando

Una celda se encuentra en modo de comando cuando no hay ningún cursor de texto que le pida que escriba. Cuando una celda está en modo de comando, puede editar el cuaderno en su conjunto, pero no escribir en celdas individuales. Para ingresar al modo de comando, presione `ESC` o use el mouse para seleccionar fuera del área del editor de una celda.  El borde izquierdo de la celda activa es azul y sólido, y el botón **Ejecutar** es azul.

   :::image type="content" source="media/how-to-run-jupyter-notebooks/command-mode.png" alt-text="Celda de cuaderno en el modo de comando ":::

| Acceso directo                      | Descripción                          |
| ----------------------------- | ------------------------------------|
| Entrar                         | Acceso al modo de edición             |        
| Mayús + Entrar                 | Ejecución de la celda, selección del siguiente contenido         |     
| Control/Comando + Entrar       | Ejecución de una celda                            |
| Alt + Entrar                   | Ejecución de una celda, inserción de la celda de código siguiente    |
| Control/Comando + Alt + Entrar | Ejecución de una celda, inserción de la celda de Markdown siguiente|
| Alt + R                       | Ejecutar todo      |                       
| Y                             | Convertir la celda en código    |                         
| M                             | Convertir la celda en Markdown  |                       
| Arriba/K                          | Seleccionar la celda anterior    |               
| Abajo/J                        | Seleccionar la celda siguiente    |               
| Un                             | Insertar la celda de código más arriba  |            
| B                             | Insertar la celda de código abajo   |           
| Control/Comando + Mayús + A   | Insertar la celda de Markdown arriba    |      
| Control/Comando + Mayús + B   | Insertar la celda de Markdown abajo   |       
| X                             | Cortar la celda seleccionada    |               
| C                             | Copiar la celda seleccionada   |               
| Mayús + V                     | Pegar la celda seleccionada arriba           |
| V                             | Pegar la celda seleccionada abajo    |       
| D D                           | Eliminar la celda seleccionada|                
| O                             | Toggle Output         |              
| Mayús + O                     | Alternar el desplazamiento de salida   |          
| I I                           | Interrumpir kernel |                   
| 0 0                           | Reiniciar kernel |                     
| Mayús + barra espaciadora                 | Desplazarse hacia arriba  |                         
| Space                         | Desplazarse hacia abajo|
| Pestaña                           | Cambiar el foco al siguiente elemento enfocable (cuando se deshabilita la captura de pestañas)|
| Control/comando + S           | Guardar cuaderno |                      
| 1                             | Cambiar a H1|                       
| 2                             | Cambiar a H2|                        
| 3                             | Cambiar a H3|                        
| 4                             | Cambiar a H4 |                       
| 5                             | Cambiar a H5 |                       
| 6                             | Cambiar a H6 |                       

### <a name="edit-mode-shortcuts"></a>Métodos abreviados del modo de edición

El modo de edición se indica mediante un cursor de texto que le pide que escriba en el área del editor. Cuando una celda se encuentra en modo de edición, puede escribir en la celda. Para ingresar al modo de edición, presione `Enter` o use el mouse para seleccionar en el área del editor de una celda. El borde izquierdo de la celda activa es verde y está sombreado, y el botón **Ejecutar** es verde. En el modo de edición también verá la solicitud del cursor en la celda.

   :::image type="content" source="media/how-to-run-jupyter-notebooks/edit-mode.png" alt-text="Celda del cuaderno en el modo de edición":::

Con los siguientes métodos abreviados de teclado, puede navegar y ejecutar código más fácilmente en cuadernos de Azure Machine Learning en el modo de edición.

| Acceso directo                      | Descripción|                                     
| ----------------------------- | ----------------------------------------------- |
| Escape                        | Especificar el modo de comandos|  
| Control/Comando + Espacio       | Activar IntelliSense |
| Mayús + Entrar                 | Ejecución de la celda, selección del siguiente contenido |                         
| Control/Comando + Entrar       | Ejecución de una celda  |                                      
| Alt + Entrar                   | Ejecución de una celda, inserción de la celda de código siguiente  |              
| Control/Comando + Alt + Entrar | Ejecución de una celda, inserción de la celda de Markdown siguiente  |          
| Alt + R                       | Ejecución de todas las celdas     |                              
| Subir                            | Subir el cursor o la celda anterior    |             
| Bajar                          | Bajar el cursor o la celda siguiente |                  
| Control/comando + S           | Guardar cuaderno   |                                
| Control/Comando + arriba          | Ir al inicio de la celda   |                             
| Control/Comando + abajo        | Ir al final de la celda |                                 
| Pestaña                           | Finalizar el código o aplicarle sangría (si se habilita la captura de pestañas) |
| Control/Comando + M           | Habilitar o deshabilitar la captura de pestañas  |                       
| Control/Cmando + ]           | Aplicar sangría |                                         
| Control/Comando + [           | Desaplicar sangría  |                                        
| Control/Comando + A           | Seleccionar todo|                                      
| Control/Comando + Z           | Deshacer |                                           
| Control/Comando + Mayús + Z   | Rehacer |                                           
| Control/Comando + Y           | Rehacer |                                           
| Control/Comando + Inicio        | Ir al inicio de la celda|                                
| Control/Comando + Fin         | Ir al final de la celda   |                               
| Control/Comando + Izquierda        | Ir una palabra a la izquierda |                               
| Control/Comando + Derecha       | Ir una palabra a la derecha |                              
| Control/Comando + Retroceso   | Eliminar palabra anterior |                             
| Control/Comando + Eliminar      | Eliminar palabra posterior |                              
| Control/Comando + /           | Alternar comentario en celda

## <a name="troubleshooting"></a>Solucionar problemas

* Si no puede conectarse a un cuaderno, asegúrese de que la comunicación de socket web **no** está deshabilitada. Para que la funcionalidad de Jupyter de instancia de proceso haga su trabajo, debe habilitarse la comunicación de socket web. Asegúrese de que la red permite conexiones de websocket a *.instances.azureml.net y *.instances.azureml.ms. 

* Cuando la instancia de proceso se implementa en un área de trabajo de Private Link, solo se puede [acceder a ella desde la red virtual](./how-to-secure-training-vnet.md#compute-instance). Si usa un archivo de hosts o DNS personalizado, agregue una entrada para <nombre-de-instancia>.<región>.instances.azureml.ms con la dirección IP privada del punto de conexión privado del área de trabajo. Para obtener más información, consulte el artículo [DNS personalizado](./how-to-custom-dns.md?tabs=azure-cli).

* Si el kernel se bloquea y se reinicia, puede ejecutar el siguiente comando para ver el registro de Jupyter y obtener más detalles. `sudo journalctl -u jupyter`. Si los problemas del kernel persisten, considere la posibilidad de usar una instancia de proceso con más memoria.
    
## <a name="next-steps"></a>Pasos siguientes

* [Ejecución de su primer experimento](tutorial-1st-experiment-sdk-train.md)
* [Copia de seguridad del almacenamiento de archivos con instantáneas](../storage/files/storage-snapshots-files.md)
* [Trabajo en entornos seguros](./how-to-secure-training-vnet.md#compute-instance)
