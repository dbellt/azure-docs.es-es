---
title: Escalado de aplicaciones de HPC (Azure Virtual Machines) | Microsoft Docs
description: Aprenda a escalar aplicaciones de informática de alto rendimiento en máquinas virtuales de Azure.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 04/16/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 8012702e9004361c8b19797bf0ca66553d307dce
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138200"
---
# <a name="scaling-hpc-applications"></a>Escalado de aplicaciones de HPC

Para que el rendimiento tanto de la escalabilidad vertical como de la horizontal de las aplicaciones de HPC sea óptimo en Azure es preciso realizar experimentos para lograr ajustar el rendimiento y optimizarlas para la carga de trabajo específica. Tanto esta sección como las páginas específicas de la serie de las máquinas virtuales ofrecen una guía general para escalar las aplicaciones.

## <a name="application-setup"></a>Configuración de la aplicación
El [repositorio azurehpc](https://github.com/Azure/azurehpc) contiene muchos ejemplos de:
- Configuración y ejecución óptimas de [aplicaciones](https://github.com/Azure/azurehpc/tree/master/apps).
- Configuración de [sistemas de archivos y clústeres](https://github.com/Azure/azurehpc/tree/master/examples).
- [Tutoriales](https://github.com/Azure/azurehpc/tree/master/tutorials) sobre cómo empezar a trabajar fácilmente con algunos flujos de trabajo de aplicaciones comunes.

## <a name="optimally-scaling-mpi"></a>Escalado óptimo de MPI 

Mediante la aplicación de las siguientes sugerencias se logra una coherencia, rendimiento y eficiencia óptimos en el escalado de aplicaciones:

- En el caso de trabajos a pequeña escala (p.ej., menos de 256 000 conexiones) use la opción:
   ```bash
   UCX_TLS=rc,sm
   ```

- En el caso de trabajos a gran escala (p.ej., más de 256 000 conexiones) use la opción:
   ```bash
   UCX_TLS=dc,sm
   ```

- En el anterior, para calcular el número de conexiones para el trabajo de MPI, use:
   ```bash
   Max Connections = (processes per node) x (number of nodes per job) x (number of nodes per job) 
   ```

## <a name="adaptive-routing"></a>Enrutamiento adaptable
El enrutamiento adaptable (AR) permite a las máquinas virtuales (VM) de Azure que ejecutan EDR y HDR InfiniBand detectar automáticamente y evitar la congestión de la red al seleccionar dinámicamente rutas de acceso de red más óptimas. Como resultado, AR ofrece una mayor latencia y ancho de banda en la red InfiniBand, que a su vez impulsa un mayor rendimiento y escalabilidad. Para obtener más detalles, consulte el [artículo TechCommunity](https://techcommunity.microsoft.com/t5/azure-compute/adaptive-routing-on-azure-hpc/ba-p/1205217).

## <a name="process-pinning"></a>Anclaje de procesos

- Ancle procesos a núcleos mediante un enfoque de anclaje secuencial (en lugar de un enfoque de equilibrio automático). 
- El enlace por Numa/Core/HwThread es mejor que el enlace predeterminado.
- En el caso de aplicaciones híbridas paralelas (OpenMP+MPI), utilice cuatro subprocesos y una clasificación MPI por CCX en tamaños de máquina virtual HB y HBv2.
- En el caso de aplicaciones de MPI puras, pruebe con 1-4 clasificaciones MPI por CCX para lograr un rendimiento óptimo en tamaños de máquina virtual HB y HBv2.
- Algunas aplicaciones con una sensibilidad extrema al ancho de banda de memoria pueden beneficiarse del uso de un menor número de núcleos por CCX. En el caso de estas aplicaciones, el uso de 2 o 3 núcleos por CCX puede reducir la contención del ancho de banda de memoria y producir mayor rendimiento real o una escalabilidad más coherente. En concreto, MPI Allreduce se puede beneficiar de este enfoque.
- En el caso de ejecuciones a mucho mayor escala, se recomienda usar transportes UD o RC + UD híbridos. Muchas bibliotecas o bibliotecas del runtime de MPI lo hacen internamente (como UCX o MVAPICH2). Compruebe las configuraciones de transporte en las ejecuciones a gran escala.

## <a name="compiling-applications"></a>Compilación de aplicaciones
<br>
<details>
<summary>Hacer clic para expandir</summary>

Aunque no es necesario, la compilación de aplicaciones con las marcas de optimización adecuadas proporciona el mejor rendimiento de escalado vertical en máquinas virtuales de las series HC y HB.

### <a name="amd-optimizing-cc-compiler"></a>Compilador AMD Optimizing C/C++ Compiler

El compilador AMD Optimizing C/C++ Compiler (AOCC) ofrece un alto nivel de optimizaciones avanzadas, subprocesamiento múltiple y compatibilidad de procesador que incluye la optimización global, la vectorización, el análisis entre procedimientos, las transformaciones de bucle y la generación de código. Los archivos binarios del compilador AOCC son adecuados para sistemas Linux que tengan la versión 2.17 de la biblioteca GNU C Library (glibc), y cualquier versión superior. El conjunto de compiladores consta de un compilador de C/C++ (clang), un compilador de Fortran (FLANG) y un front-end de Fortran para Clang (Dragon Egg).

### <a name="clang"></a>Clang

Clang es un compilador para C, C++ y Objective-C que se usa para controlar el preprocesamiento, el análisis, la optimización, la generación de código, el ensamblado y la vinculación. Clang admite la marca `-march=znver1`, que permite mejorar y ajustar la generación de código para la arquitectura X86 basada en Zen de AMD.

### <a name="flang"></a>FLANG

El compilador FLANG es una incorporación reciente al conjunto AOCC (se agregó en abril de 2018) y la versión que está a disposición de los programadores para que la descarguen y hagan pruebas es la versión preliminar. Se basa en Fortran 2008 y AMD amplía la versión de GitHub de FLANG (https://github.com/flang-compiler/flang). El compilador FLANG admite todas las opciones del compilador Clang y, además, tiene varias opciones adicionales específicas del compilador FLANG.

### <a name="dragonegg"></a>DragonEgg

DragonEgg es un complemento de gcc que reemplaza los optimizadores y los generadores de código de GCC por los del proyecto LLVM. DragonEgg, que se incluye con AOCC, funciona con gcc-4.8.x, se ha probado para destinos x86-32/x86-64 y se ha utilizado correctamente en diversas plataformas de Linux.

GFortran es el front-end real para los programas de Fortran responsables del preprocesamiento, la redistribución y el análisis semántico que genera la representación intermedia de GCC GIMPLE (IR). DragonEgg es un complemento de GNU que conecta con el flujo de compilación de GFortran. Implementa la API del complemento de GNU. Con la arquitectura del complemento, DragonEgg se convierte en el controlador del compilador e impulsa las distintas fases de compilación.  Después de seguir las instrucciones de descarga e instalación, DragonEgg se puede invocar mediante: 

```bash
$ gfortran [gFortran flags] 
   -fplugin=/path/AOCC-1.2-Compiler/AOCC-1.2-     
   FortranPlugin/dragonegg.so [plugin optimization flags]     
   -c xyz.f90 $ clang -O3 -lgfortran -o xyz xyz.o $./xyz
```
   
### <a name="pgi-compiler"></a>Compilador PGI
Está confirmado que PGI Community Edition, versión 17 funciona con AMD EPYC. Una versión compilada por PGI de STREAM entrega todo el ancho de banda de la memoria de la plataforma. La más reciente Community Edition 18.10 (noviembre de 2018) también debería funcionar correctamente. A continuación encontrará una CLI de ejemplo para realizar una compilación óptima con el compilador Intel:

```bash
pgcc $(OPTIMIZATIONS_PGI) $(STACK) -DSTREAM_ARRAY_SIZE=800000000 stream.c -o stream.pgi
```

### <a name="intel-compiler"></a>Compilador Intel
Está confirmado que la versión 18 del compilador Intel funciona con AMD EPYC. A continuación encontrará una CLI de ejemplo para realizar una compilación óptima con el compilador Intel.

```bash
icc -o stream.intel stream.c -DSTATIC -DSTREAM_ARRAY_SIZE=800000000 -mcmodel=large -shared-intel -Ofast –qopenmp
```

### <a name="gcc-compiler"></a>Compilador GCC 
En el caso de HPC, AMD la versión 7.3 del compilador GCC, o cualquier versión posterior. Las versiones anteriores, como la 4.8.5, que se incluía en RHEL/CentOS 7.4, no se recomiendan. GCC 7.3, y las versiones posteriores, ofrecerán un rendimiento considerablemente mejor en las pruebas de HPL, HPCG y DGEMM.

```bash
gcc $(OPTIMIZATIONS) $(OMP) $(STACK) $(STREAM_PARAMETERS) stream.c -o stream.gcc
```
</details>

## <a name="next-steps"></a>Pasos siguientes

- Pruebe sus conocimientos con un [módulo de aprendizaje sobre la optimización de aplicaciones HPC en Azure](/learn/modules/optimize-tightly-coupled-hpc-apps/).
- Revise la [información general de la serie HBv3](hbv3-series-overview.md) y la [información general de la serie HC](hc-series-overview.md).
- En los [blogs de Azure Compute Community Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes, ejemplos de la carga de trabajo HPC y resultados de HPC.
- Obtenga más información acerca de [HPC](/azure/architecture/topics/high-performance-computing/) en Azure.