# ProyectoTyR

INTRODUCCIÓN 

El proyecto se basa en la implementación de un sistema de comunicación digital utilizando modulación por desplazamiento de fase binaria (BPSK). El objetivo principal es transmitir un archivo de texto a través de una señal BPSK usando GNU Radio y dos tarjetas LimeSDR, una para la transmisión y otra para la recepción. Este documento describe la configuración del transmisor y receptor, los bloques utilizados y sus funciones, así como los desafíos encontrados y las conclusiones obtenidas.  

Para la ejecución de este proyecto se utilizó lo siguiente:  


    Hardware: 

        2 tarjetas LimeSDR 

        2 antenas compatibles 

        2 computadoras con puertos USB compatibles 

 
    Software: 

        Sistema operativo Linux (Ubuntu recomendado) 

        Librería LimeSuite 

        GNU Radio 

        Python 3 (para corrección de errores) 

         

Receptor 

El receptor mostrado en el diagrama de GNU Radio Companion está diseñado para procesar y demodular una señal digital recibida a través de una LimeSDR.  

El bloque Options configura el entorno del flujo de trabajo en GNU Radio, estableciendo el lenguaje de salida como Python y especificando la generación de opciones para la interfaz gráfica QT. Seguido a esto, se definen varias variables: `samp_rate` (32k), `freq` (102.4k), `if_freq` (10.24k), y `rx_gain` (62.8m), que son esenciales para configurar parámetros clave del sistema como la tasa de muestreo, frecuencia central, frecuencia intermedia y ganancia de recepción respectivamente. 

  

El bloque LimeSDR Source (RX) recibe la señal RF a 433 MHz y la muestrea a una tasa de 102.4 kHz. La señal recibida es luego filtrada por un Low Pass Filter, que elimina componentes de alta frecuencia no deseadas utilizando un filtro con una frecuencia de corte de 16 kHz y un ancho de transición de 16 kHz. 

  

Para ajustar la tasa de muestreo de la señal, se utiliza un Rational Resampler que interpola y decima la señal. El bloque Throttle controla la velocidad de procesamiento para que coincida con la tasa de muestreo de 48 kHz, asegurando que los datos se procesen a la velocidad correcta. 

  

El Automatic Gain Control (AGC) ajusta automáticamente la ganancia de la señal para mantener un nivel constante. Este es seguido por el bloque FLL Band-Edge, que realiza la estimación de frecuencia y sincronización de la señal, utilizando parámetros como muestras por símbolo (4), factor de rolloff del filtro (350m), y ancho de banda del bucle (62.8m). 

  

La salida de este procesamiento inicial se guarda temporalmente en un Virtual Sink con el ID de flujo `RX_op2` para su uso posterior. La QT GUI Frequency Sink proporciona una representación visual de la señal en el dominio de frecuencia, mostrando información importante sobre la frecuencia central y el ancho de banda. 

  

El siguiente paso en el procesamiento de la señal es la sincronización de símbolos, realizada por el bloque Symbol Sync. Este bloque utiliza un detector de error de sincronización (Mueller and Mueller), ajusta los parámetros de sincronización, y asegura que los símbolos en la señal recibida estén alineados correctamente. El Costas Loop corrige cualquier desvío de fase en la señal. 

  

Para decodificar la señal, se emplean dos bloques: Constellation Decoder, que decodifica la constelación de la señal digital, y Differential Decoder, que realiza la decodificación diferencial con un módulo de 2. Los datos decodificados se almacenan nuevamente en un Virtual Sink con el ID de flujo `RXop2`. 

  

En la última etapa del procesamiento, los datos son mapeados utilizando un bloque Map que convierte los datos recibidos a un formato adecuado para el procesamiento posterior. El bloque Correlate Access Code - Tag Stream detecta y etiqueta los paquetes en el flujo de datos basados en un código de acceso específico (101010).  

  

Los datos etiquetados se reempaquetan en bytes mediante el bloque Repack Bits y su integridad es verificada utilizando el bloque Stream CRC32. Los datos procesados y verificados se guardan finalmente en un archivo especificado por el bloque File Sink. 

  

Adicionalmente, para fines de visualización, los datos se convierten de caracteres sin signo a flotantes usando el bloque UChar to Float, y se muestran en el dominio del tiempo mediante dos bloques QT GUI Time Sink, proporcionando una representación visual detallada de la señal en el dominio del tiempo. 

  

En resumen, este flujo de bloques en GNU Radio implementa un completo receptor de señal digital que incluye recepción, filtrado, resampling, sincronización de símbolos, corrección de fase, decodificación de constelación y diferencial, etiquetado y verificación de integridad de datos, y almacenamiento en un archivo. La interfaz gráfica QT permite visualizar tanto la señal en frecuencia como en el dominio del tiempo, facilitando el análisis y monitoreo del proceso de recepción y demodulación. 
