# ProyectoTyR
GRUPO "B"

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

El diagrama presentado representa un receptor para la demodulación y decodificación de una señal BPSK (Binary Phase Shift Keying) utilizando GNU Radio y LimeSDR.


Explicación del Receptor 

LimeSDR Source (RX) El flujo comienza con el bloque LimeSDR Source (RX), que captura la señal RF (Radiofrecuencia) recibida por la tarjeta LimeSDR. La frecuencia de RF se establece en 435 MHz y la tasa de muestreo en 1.024 MSps. Este bloque convierte la señal RF en una señal de banda base para su procesamiento posterior. 

La señal de banda base pasa a través de un Low Pass Filter, que filtra las frecuencias no deseadas y permite el paso de las frecuencias de interés. Este filtro tiene una frecuencia de corte de 10 kHz y un ancho de transición de 1.6 kHz, utilizando una ventana Hamming con un parámetro beta de 6.76. El propósito de este filtro es limpiar la señal y eliminar el ruido fuera de la banda de interés. 

A continuación, la señal pasa por un Rational Resampler, que ajusta la tasa de muestreo de la señal. En este caso, la señal se interpola por 1 y se decima por 16, ajustando la tasa de muestreo para que coincida con la tasa de procesamiento requerida por los bloques posteriores. 

El bloque Throttle controla la tasa de procesamiento de los datos, limitando la velocidad a 48 kSps. Esto evita que el flujo de datos sea demasiado rápido para ser manejado por el hardware o el software. 

El AGC (Automatic Gain Control) ajusta automáticamente la ganancia de la señal para mantener un nivel de amplitud constante. Se configura con una referencia de 1, una ganancia inicial de 1 y una ganancia máxima de 2. Este bloque es esencial para manejar variaciones en la amplitud de la señal recibida. 

La señal sincronizada se pasa a través del bloque Symbol Sync, que sincroniza los símbolos de la señal con el reloj del sistema. Este bloque utiliza el detector de error de temporización Mueller y Müller, con muestras por símbolo configuradas en 4 y un ancho de banda del lazo de 62.8 m. El bloque también cuenta con un resamplador interpolante MMSE de 8 tap FIR, lo que garantiza una correcta recuperación de los datos. 

El siguiente bloque es el Costas Loop, que recupera la portadora y corrige cualquier desplazamiento de fase en la señal recibida. Este lazo tiene un ancho de banda de 62.8 m y un orden de 2, lo que permite una recuperación precisa de la portadora y la corrección de la fase. 

La señal corregida en fase se decodifica usando el bloque Constellation Decoder, que convierte la señal modulada en BPSK de nuevo a datos binarios. Este bloque es crucial para recuperar los bits originales de la señal modulada. 

A continuación, los datos pasan por el bloque Differential Decoder, que decodifica los datos codificados diferencialmente. Esto asegura que la secuencia original de bits se recupere correctamente, incluso si hubo errores de fase durante la transmisión. 

El bloque Unpack K Bits desempaqueta los bits de la señal recibida, convirtiéndolos de una representación compacta a una más adecuada para el procesamiento posterior. Este bloque toma bits por byte de entrada y los convierte en 8 bits por byte de salida. 

Los datos desempaquetados pasan por el bloque Correlate Access Code - Tag Stream, que busca un código de acceso específico en la secuencia de bits y etiqueta los paquetes de datos. Este bloque utiliza un código de acceso definido por el usuario y asegura que cada paquete esté correctamente identificado y etiquetado para su posterior procesamiento. 

El siguiente paso es la verificación de la integridad de los paquetes de datos utilizando el bloque Stream CRC32. Este bloque verifica el código CRC32 añadido durante la transmisión para asegurarse de que los datos no han sido corrompidos. Si el CRC coincide, los datos son considerados válidos. 

Finalmente, los datos decodificados y verificados se almacenan en un archivo utilizando el bloque File Sink. Este bloque guarda los datos en un archivo especificado por el usuario, permitiendo su posterior análisis y utilización. 

Adicionalmente, hay varios bloques QT GUI Sink utilizados para la visualización de la señal en diferentes etapas del procesamiento. Estos bloques permiten observar la señal en el dominio del tiempo y la frecuencia, facilitando la depuración y el análisis del desempeño del receptor. 



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


CONCLUSIONES
1. Se logró implementar un sistema funcional de recepción de datos utilizando la modulación BPSK. La configuración de los bloques en GNU Radio permitió la correcta modulación, transmisión, recepción y decodificación de los datos.

2. El proyecto facilitó una comprensión más profunda de cómo utilizar GNU Radio y las tarjetas LimeSDR para la creación de sistemas de comunicación digital. Aprendimos a configurar y ajustar diversos bloques para asegurar un flujo de datos eficiente y preciso.

3. A lo largo del proyecto, se enfrentaron y resolvieron varios desafíos técnicos, como errores de sintaxis en la definición de variables y la correcta configuración de los bloques de control de tasa y sincronización. Estos desafíos proporcionaron una oportunidad para desarrollar habilidades de resolución de problemas y depuración.

4. Se implementaron métodos para asegurar la integridad de los datos, como el uso de CRC32 para la verificación de la integridad de los paquetes. Esto garantizó que los datos transmitidos y recibidos no estuvieran corruptos, lo cual es crucial en cualquier sistema de comunicación digital.

 

