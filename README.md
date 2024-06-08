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

         

RECEPTOR
![image](https://github.com/luisferhz99/ProyectoTyR/assets/31906680/65019a17-5361-4f68-aded-8313e93a76a0)

El receptor está diseñado para capturar, filtrar, resamplear, sincronizar y decodificar una señal BPSK, recuperando los datos originales transmitidos. Cada bloque en el flujo tiene una función específica, asegurando que los datos se procesen y recuperen de manera eficiente y precisa. Los bloques de sincronización y corrección de fase, como el Symbol Sync y el Costas Loop, son esenciales para la correcta recuperación de los datos, mientras que los bloques de verificación de integridad, como el Stream CRC32, aseguran que los datos no se hayan corrompido durante la transmisión. 

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



TRANSMISOR 
![image](https://github.com/luisferhz99/ProyectoTyR/assets/31906680/002b30e2-b797-4a28-9c7a-cb0b8a7595be)

El transmisor está diseñado para leer datos de un archivo, procesarlos y modulados utilizando BPSK, y finalmente transmitirlos a través de una red utilizando ZeroMQ. Cada bloque en el flujo del transmisor tiene una función específica, desde la preparación de los datos hasta su modulación y transmisión, asegurando que los datos se transmiten de manera eficiente y con la integridad preservada. 

El diagrama mostrado representa un transmisor que utiliza modulación BPSK (Binary Phase Shift Keying) para transmitir datos digitales. 

*Explicación del Transmisor*

EPB: File Source to Tagged Stream El flujo comienza con un bloque de fuente de archivo etiquetado (EPB: File Source to Tagged Stream). Este bloque lee los datos de un archivo de entrada, especificado por el usuario, y convierte estos datos en un flujo etiquetado. La longitud del paquete se define en este bloque, lo que permite que los datos sean segmentados y procesados correctamente a lo largo del flujo de bloques. 

El siguiente bloque en el flujo es Stream CRC32, que añade un código de redundancia cíclica (CRC) de 32 bits a cada paquete de datos. Este código CRC se utiliza para verificar la integridad de los datos durante la transmisión y la recepción, asegurando que los datos no hayan sido corrompidos. 

El bloque Protocol Formatter formatea los paquetes de datos de acuerdo con un formato específico definido por el usuario. Este formato puede incluir la adición de encabezados y otros datos de control necesarios para la transmisión correcta de los paquetes. 

El Tagged Stream Mux combina múltiples flujos etiquetados en uno solo. Este bloque es crucial para manejar múltiples tipos de datos o varios flujos de datos simultáneamente, asegurando que cada paquete esté correctamente etiquetado para su posterior procesamiento. 

El bloque Constellation Modulator modula los datos utilizando una constelación BPSK. La modulación BPSK asigna bits de datos a fases específicas de la señal portadora, permitiendo la transmisión de datos digitales sobre una portadora analógica. 

El FFT Filter aplica un filtrado en el dominio de la frecuencia a los datos modulados. Este bloque es utilizado para eliminar componentes de frecuencia no deseados y mejorar la calidad de la señal transmitida. 

El Fractional Resampler ajusta la tasa de muestreo de la señal. En este caso, la tasa de muestreo se ajusta mediante un cambio fraccionario para asegurar que la señal se transmite a la tasa de muestreo correcta requerida por el hardware de transmisión. 

Throttle El bloque Throttle controla la tasa de procesamiento de los datos, asegurando que el flujo de datos no sea demasiado rápido para ser manejado por el hardware de transmisión o el software. 

ZMQ PUB Sink Finalmente, la señal modulada y procesada se envía a través del bloque ZMQ PUB Sink, que transmite los datos utilizando el protocolo ZeroMQ (ZMQ). Este bloque permite la transmisión de datos a través de una red, facilitando la comunicación entre diferentes sistemas o componentes del sistema de comunicación digital. 

 

