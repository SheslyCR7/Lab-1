# Instrumentación Biomédica y Biosensores 
 LABORATORIO - 1 Monitoreo del patrón y frecuencia respiratoria 

hola
## Requisitos
 Hardware:
 - Arduino IDE
 - Matlab
   
  Software:
 - Sensor FSR 402
 - ESP32


----
## Introducción

En este laboratorio se analizó el patrón y la frecuencia respiratoria mediante la adquisición de una señal biológica usando sensores y Arduino. Posteriormente, la señal se procesó en MATLAB para observar su comportamiento en el tiempo y la frecuencia, comparando condiciones de reposo y habla.
ó
En este laboratorio se estudió el patrón y la frecuencia respiratoria mediante la adquisición y análisis de una señal biológica con sensores, Arduino y MATLAB. Esta práctica brinda una base teórica y aplicada sobre el monitoreo respiratorio, permitiendo comprender su relevancia dentro de la instrumentación biomédica y su utilidad en contextos reales de evaluación fisiológica.

-----
# OBJETIVOS

## Objetivo General

Evaluar la influencia del habla o verbalización sobre el
patrón respiratorio

## Objetivos Específicos
• Reconocer las variables físicas principalmente involucradas en el proceso
respiratorio.

• Desarrollar un sistema que extraiga el patrón respiratorio y la frecuencia
respiratoria.

• Identificar tareas de verbalización a partir del patrón y/o la frecuencia
respiratoria.


-----
## Proceso Respiratorio
La respiración es un proceso vital para el funcionamiento normal en todos los niveles de organización, desde la célula hasta el organismo. El oxígeno, suministrado por la circulación local a nivel tisular, funciona en la membrana interna mitocondrial como mediador esencial para la liberación de energía. En las mitocondrias, los nutrientes digeridos experimentan reacciones metabólicas, llegan a la cadena de transporte de electrones y liberan compuestos de alta energía (p. ej., trifosfato de adenosina). El principal subproducto de este proceso, el dióxido de carbono, se libera en la sangre venosa y regresa a los pulmones. El dióxido de carbono se difunde a través de las paredes alveolares y se disuelve en el aire exhalado. La frecuencia respiratoria (es decir, el número de respiraciones por minuto) está altamente regulada para que las células produzcan la energía óptima en cualquier momento. Un complejo sistema nervioso de tejidos nerviosos regula la tasa de entrada de oxígeno y la tasa de salida de dióxido de carbono, ajustándola en consecuencia en condiciones que alteran las presiones parciales de los gases en la sangre. La respiración involucra el cerebro, el tronco encefálico, los músculos respiratorios, los pulmones, las vías respiratorias y los vasos sanguíneos. Todas estas estructuras tienen una participación estructural, funcional y reguladora en la respiración. [1]

<p align="center">
  <img width="400" height="300" alt="ChatGPT Image 10 feb 2026, 11_51_06 p m" src="https://github.com/user-attachments/assets/af834e30-64be-4e7f-9ad0-447e488c4079" />
</p>
<p align="center">
  Fig 1. Proceso respiratorio
</p>


# 4. Parte B (Captura de la señal)

Este proyecto implementa un sistema de adquisición de señal respiratoria utilizando una ESP32 programada en Arduino IDE, la cual captura la variación de voltaje generada por un sensor respiratorio y envía los datos a MATLAB mediante comunicación serial.

El objetivo del script en MATLAB es:

Capturar la señal durante un tiempo definido.

- Visualizarla en tiempo real.

- Guardarla en un archivo .mat.

- Permitir análisis posterior (filtrado, FFT y cálculo de frecuencia respiratoria)

 ## Descripción General del Funcionamiento

El flujo del sistema es:

- Sensor → ESP32 (ADC 12 bits) → Arduino IDE → Serial USB → MATLAB → Archivo .mat

### La ESP32 digitaliza la señal analógica del sensor respiratorio y la envía como valores numéricos por puerto serial. MATLAB recibe esos valores, los grafica en tiempo real y los almacena.

## Explicación del Código (adquisición)


###  1️. Inicialización y entrada de datos

```python
clc; 
clear;
close all;

nombre = input('Ingrese su nombre: ', 's');
Trec = input('Ingrese la duración de la muestra en segundos: ');
archivo = nombre + ".mat";

```
Este bloque limpia el entorno de MATLAB y solicita al usuario el nombre y el tiempo total de grabación.
El archivo final se guardará con el nombre ingresado, permitiendo identificar fácilmente cada adquisición.

### 2️. Configuración del sistema

```python
puerto = "COM13";
baudrate = 115200;
Fs = 50; % Hz
ventana = 10;

Ntotal = Fs * Trec;
Nvent = Fs * ventana;


```

Aquí se definen los parámetros principales del sistema:

- puerto: corresponde al puerto asignado a la ESP32.

- baudrate: debe coincidir con Serial.begin(115200) en Arduino IDE para que la comunicación sea correcta.

- Fs (frecuencia de muestreo): determina cuántas muestras por segundo se adquieren.

- ventana: cantidad de segundos visibles en tiempo real.
 
¿Por qué se eligió Fs = 50 Hz?

La respiración humana en reposo tiene una frecuencia aproximada entre 0.2 y 0.33 Hz (12–20 respiraciones por minuto), y durante el habla puede aumentar aprox de 0.5 Hz.


### 3️. Conexión con la ESP32

```python
s = serialport(puerto, baudrate);
configureTerminator(s,"LF");
flush(s);
pause(2);

```

Se configura el terminador de línea para que MATLAB pueda leer correctamente cada dato enviado desde Arduino IDE.
La pausa de 2 segundos permite que la ESP32 se estabilice después de abrir el puerto.



### 4️. Preparación de vectores y gráfica

```python
buffer = zeros(Nvent,1);
voltaje = zeros(Ntotal,1);
tiempo_total = (0:Ntotal-1)/Fs;
tiempo_vent = linspace(-ventana, 0, Nvent);

figure;
h = plot(tiempo_vent, buffer, 'LineWidth', 1.5);
ylim([0 3.3]);


```
- voltaje almacena toda la señal adquirida.

- buffer permite mostrar solo los últimos 10 segundos.

- tiempo_total genera el eje temporal completo.

- ylim([0 3.3]) corresponde al rango típico de operación del ADC de la ESP32 (0–3.3 V).


### 5️. Adquisición en tiempo real

```python
for k = 1:Ntotal
 valor = str2double(readline(s));
 if ~isnan(valor)
 voltaje(k) = valor;
 buffer = [buffer(2:end); valor];
 set(h, 'YData', buffer);
 drawnow limitrate;
 end
end


```

- Lee cada valor enviado por la ESP32.

- Lo convierte a número.

- Lo almacena en el vector completo.

- Actualiza la gráfica en tiempo real.

### 6️. Cierre y almacenamiento

```python
clear s;
save(archivo, "voltaje", "tiempo_total", "Fs");

```
Se libera el puerto serial y se guarda la señal junto con el eje temporal y la frecuencia de muestreo para análisis posterior.


### Codigo completo de captura

```python
clc;
clear; 
close all;

% ================= SOLICITUD DE DATOS =================
nombre = input('Ingrese su nombre: ', 's');
Trec = input('Ingrese la duración de la muestra en segundos: ');
archivo = nombre + ".mat";

% ================= CONFIGURACIÓN TÉCNICA =================
puerto = "COM13";
baudrate = 115200;
Fs = 50; % Hz
ventana = 10; 

Ntotal = Fs * Trec; 
Nvent = Fs * ventana; 

% ================= CONEXIÓN SERIAL =================
s = serialport(puerto, baudrate);
configureTerminator(s,"LF");
flush(s);
pause(2);

% Buffers
buffer = zeros(Nvent,1); 
voltaje = zeros(Ntotal,1); 
tiempo_total = (0:Ntotal-1)/Fs;
tiempo_vent = linspace(-ventana, 0, Nvent);

% Figura
figure;
h = plot(tiempo_vent, buffer, 'LineWidth', 1.5);
xlabel('Tiempo (s)');
ylabel('Voltaje (V)');
title(['Señal respiratoria de: ', nombre]);
grid on;
ylim([0 3.3]);

% ======== ADQUISICIÓN ========
disp("Iniciando grabación...");

for k = 1:Ntotal
 valor = str2double(readline(s));
 if ~isnan(valor)
 voltaje(k) = valor;
 buffer = [buffer(2:end); valor]; 
 set(h, 'YData', buffer);
 drawnow limitrate;
 end
end

% Cerrar puerto
clear s;


% Guardar con el nombre ingresado
save(archivo, "voltaje", "tiempo_total", "Fs");

disp("Grabación finalizada y guardada como: " + archivo);
```




## Parte B (Parte Filtrado)

### Después de adquirir y almacenar la señal respiratoria desde la ESP32, es necesario procesarla para obtener información fisiológicamente relevante. La señal original puede contener ruido, artefactos de movimiento y componentes no deseadas, por lo que se aplica un filtrado digital.

### Posteriormente, se utiliza la Transformada Rápida de Fourier (FFT) para transformar la señal del dominio del tiempo al dominio de la frecuencia, permitiendo identificar la frecuencia respiratoria dominante de manera automática y objetiva.


## 1. Carga de datos

```python
load("HABLANDO.mat"); 
% variables: voltaje, tiempo_total, Fs

x = voltaje; 
t = tiempo_total;
```

carga la señal previamente adquirida desde la ESP32.

- voltaje → señal respiratoria en el dominio del tiempo.

- tiempo_total → eje temporal.

- Fs → frecuencia de muestreo (50 Hz).

Aquí comienza el procesamiento de la señal almacenada


### 2. Filtrado digital (Pasa-banda 0.1–0.5 Hz)

```python
% Respiración: 0.1 a 0.5 Hz (6–30 resp/min)

f_low = 0.1; % Hz
f_high = 0.5; % Hz

[b,a] = butter(2,[f_low f_high]/(Fs/2),"bandpass");
x_filt = filtfilt(b,a,x);
```

### ¿Por qué se aplicó un filtro?

La señal original puede contener:

- Ruido eléctrico.

- Movimiento corporal, etc

Por ello se utiliza un filtro pasa-banda para conservar únicamente el rango fisiológico de interés.

### ¿Por qué 0.1–0.5 Hz?

La frecuencia respiratoria humana normal se encuentra aproximadamente entre:

- 6 respiraciones/min → 0.1 Hz

- 30 respiraciones/min → 0.5 Hz

 Por tanto, este rango:

- Elimina componentes demasiado lentas (por ejemplo movimiento).

- Elimina componentes (ruido, vibraciones, artefactos).

###  ¿Por qué Butterworth de orden 2?


El filtro Butterworth se eligió porque:

- Tiene respuesta suave y sin ondulaciones.

- No distorsiona significativamente la amplitud.

- Es adecuado para señales biomédicas.

### El orden 2 es suficiente porque:

- La respiración es una señal simple y lenta.

- No se requiere una pendiente  brusca.


## 3️. Visualización en el dominio del tiempo

```python
plot(t,x,'Color',[0.6 0.6 0.6]); hold on;
plot(t,x_filt,'b','LineWidth',1.5);
```
Aquí se comparan:

- Señal original 

- Señal filtrada 

Esto permite observar cómo el filtrado mejora la claridad del patrón respiratorio y facilita la identificación de ciclos.


## 4. Transformada Rápida de Fourier (FFT)


```python
N = length(x_filt);
X = fft(x_filt);
f = (0:N-1)*(Fs/N);

mag = abs(X)/N;
```
La señal respiratoria inicialmente está en el dominio del tiempo.
La FFT permite convertirla al dominio de la frecuencia para identificar qué frecuencias la componen.

El objetivo principal es encontrar la frecuencia dominante, que corresponde a la frecuencia respiratoria.

- La respiración ocurre por debajo de 1 Hz.

- Frecuencias mayores no son fisiológicamente relevantes en este contexto.


## 5. Cálculo de la frecuencia respiratoria dominante


```python
[~,idx] = max(mag);
f_resp = f(idx);
resp_min = f_resp * 60;
```

- Identifica el pico máximo del espectro.

- Se obtiene la frecuencia correspondiente.

- Se convierte a respiraciones por minuto multiplicando por 60.


### Justificación

El filtrado pasa-banda 0.1–0.5 Hz permite aislar la componente fisiológica respiratoria eliminando ruido y artefactos. Posteriormente, la Transformada Rápida de Fourier permite identificar la frecuencia dominante de la señal, facilitando el cálculo automático de la frecuencia respiratoria.

Este procedimiento mejora la precisión del análisis y permite evaluar objetivamente las diferencias entre condiciones como reposo y verbalización.








### 3. Grafico de la convolución
```python
fig = plt.figure(figsize=(10, 5)) 
plt.plot(y,color='g')
plt.title("Señal Resultante (santiago)")  
plt.xlabel("(n)") 
plt.ylabel("y [n]") 
plt.grid() 
plt.stem(range(len(y)), y)
```

<p align="center">
    <img src="https://github.com/user-attachments/assets/030a2690-8be0-4fd6-bf8f-c76ff7ca80f9" alt="imagen" width="450">
</p>



Este fragmento de código genera un gráfico de la señal resultante y[n], que es el resultado de la convolución entre x[n] y h[n]. Se traza la señal con una línea verde usando plt.plot(y, color='g'). Luego, se superpone un gráfico de tipo stem con plt.stem(range(len(y)), y), resaltando los valores discretos de la señal.

---


## Correlación
La correlación en señales mide estadísticamente cómo dos señales varían de manera conjunta, evaluando su similitud o relación lineal. Es clave en el procesamiento de señales, ya que permite analizar sincronización, patrones y dependencias entre flujos de datos o formas de onda. Prácticamente, se usa para detectar similitudes, identificar patrones, filtrar ruido y extraer información relevante.[2]
### Fórmula de la correlación cruzada:

$$
R_{xy}[n] = \sum_{k=-\infty}^{\infty} x[k] y[k+n]
$$

Donde:
- \(R_{xy}[n]\) es la correlación cruzada entre \(x\) y \(y\).
- \(x[k]\) y \(y[k+n]\) representan las señales en diferentes desplazamientos temporales.


### 1. Señal Cosenoidal
```python
Ts = 1.25e-3
n = np.arange(0, 9) #valores enteros
x1 = np.cos(2*np.pi*100*n*Ts)
fig = plt.figure(figsize=(10, 5)) 
plt.plot(n, x1, label="", color='black')
plt.title("Señal Cosenoidal")  
plt.xlabel("(n)") 
plt.ylabel("x1 [nTs]") 
plt.grid()
plt.stem(range(len(x1)), x1)
```

<p align="center">
    <img src="https://github.com/user-attachments/assets/a6b5c5b2-e536-418f-9f35-36c75dd033bd" alt="imagen" width="450">
</p>

Se genera y grafica una señal cosenoidal muestreada. Primero, se define un periodo de muestreo Ts = 1.25e-3, y luego se crea un arreglo n con valores enteros de 0 a 8 usando np.arange(0, 9). La función np.arange(inicio, fin) genera una secuencia de números desde inicio hasta fin-1 con un paso de 1 por defecto. En este caso, n representa los instantes de muestreo en el dominio discreto.

A partir de n, se calcula la señal x1 como un coseno de 100 Hz evaluado en los instantes n * Ts. Para la visualización, se crea una figura de tamaño 10x5, donde plt.plot(n, x1, color='black') traza la señal con una línea negra, y plt.stem(range(len(x1)), x1) resalta los valores discretos.


---

### 2. Señal Senoidal
```python
x2 = np.sin(2*np.pi*100*n*Ts)
fig = plt.figure(figsize=(10, 5)) 
plt.plot(n, x2, label="", color='black')
plt.title("Señal Senoidal")  
plt.xlabel("(n)") 
plt.ylabel("x2 [nTs]") 
plt.grid()
plt.stem(range(len(x2)), x2)
```
<p align="center">
    <img src="https://github.com/user-attachments/assets/0926d754-2336-4313-8858-af1f28e19ed2" alt="imagen" width="450">
</p>

Al igual que en la gráfica anterior, este código genera y visualiza una señal, pero en este caso es una señal senoidal en lugar de una cosenoidal. Se usa el mismo conjunto de valores n = np.arange(0, 9), generado con np.arange(), y se calcula x2 como un seno de 100 Hz evaluado en los instantes n * Ts.

---

### 3. Correlación de las Señales y Representación Grafica
```python
correlacion = np.correlate(x1,x2,mode='full')
print('Correlación =',correlacion)
fig = plt.figure(figsize=(10, 5)) 
plt.plot(correlacion, color='black')
plt.stem(range(len(correlacion)), correlacion)
plt.title("Correlación")  
plt.xlabel("(n)") 
plt.ylabel("R[n]") 
plt.grid()
```
Se calcula y grafica la correlación cruzada entre las señales x1 y x2. La correlación mide la similitud entre dos señales a diferentes desplazamientos en el tiempo, lo que permite identificar patrones compartidos o desfases entre ellas.

Primero, np.correlate(x1, x2, mode='full') computa la correlación cruzada, generando una nueva señal correlacion, cuya longitud es len(x1) + len(x2) - 1. Luego, el resultado se imprime en la consola.

$$
\text{Correlación} = \begin{bmatrix}
-2.44929360 \times 10^{-16} & -7.07106781 \times 10^{-1} & -1.50000000 & -1.41421356 \\
-1.93438661 \times 10^{-16} & 2.12132034 \times 10^{0} & 3.50000000 & 2.82842712 \\
8.81375476 \times 10^{-17} & -2.82842712 \times 10^{0} & -3.50000000 & -2.12132034 \\
3.82856870 \times 10^{-16} & 1.41421356 \times 10^{0} & 1.50000000 & 7.07106781 \times 10^{-1} \\
0.00000000 \times 10^{0}
\end{bmatrix}
$$

<p align="center">
    <img src="https://github.com/user-attachments/assets/c0028249-0f51-430a-bebc-44794b47bfc0" alt="imagen" width="450">
</p>

Para visualizar la correlación, se crea una figura de 10x5 donde plt.plot(correlacion, color='black') dibuja la señal con una línea negra, mientras que plt.stem(range(len(correlacion)), correlacion) resalta sus valores discretos. 

---
## Transformación (Señal Electromiografica)
### 1. Caracterizacion en Función del Tiempo 
```python
datos = wfdb.rdrecord('session1_participant1_gesture10_trial1') 
t = 1500
señal = datos.p_signal[:t, 0] 
fs = datos.fs
```
Se carga una señal de electromiografía (EMG) y se extraen los primeros 1500 puntos.

#### 1.1. Estadisticos Descriptivos y frecuencia de muestreo
Se calculan los siguientes estadísticos:
- *Media (μ):* Valor promedio de la señal.
- *Desviación Estándar (σ):* Medida de la dispersión de los datos respecto a la media.
- *Coeficiente de Variación (CV):* Relación entre desviación estándar y media, expresada en porcentaje.
```python
def caracterizacion():
    print()
    print()
    media = np.mean(señal)
    desvesta = np.std(señal)
    print('Media de la señal:',np.round(media,6))
    print('Desviación estándar:',np.round(desvesta,6))
    print("Coeficiente de variación:",np.round((media/desvesta),6))
    print('Frecuencia de muestreo:',fs,'Hz')
    
    fig = plt.figure(figsize=(8, 4))
    sns.histplot(señal, kde=True, bins=30, color='black')
    plt.hist(señal, bins=30, edgecolor='blue')
    plt.title('Histograma de Datos')
    plt.xlabel('datos')
    plt.ylabel('Frecuencia')

caracterizacion()
```
- Media de la señal: 0.000131
- Desviación estándar: 0.071519
- Coeficiente de variación: 0.001834
- Frecuencia de muestreo: 2048 Hz
<p align="center">
    <img src="https://github.com/user-attachments/assets/d64b9102-821a-4754-a102-2a5977baab0c" alt="imagen" width="450">
</p>

- Histograma:El histograma resultante tiene una distribución que se asemeja a una campana de Gauss, lo cual es un fuerte indicativo de una distribución normal en los datos. Esto significa que la mayoría de los valores están concentrados alrededor del promedio, mientras que las frecuencias disminuyen gradualmente hacia ambos extremos.
  
#### 1.2. Grafica de Electromiografía
```python
fig = plt.figure(figsize=(10, 5)) 
plt.plot(señal, color='m')
plt.title("Electromiografía [EMG]")  
plt.xlabel("muestras[n]") 
plt.ylabel("voltaje [mv]") 
plt.grid()
```
<p align="center">
    <img src="https://github.com/user-attachments/assets/9ea890d5-b5b2-46e3-aaf5-dd1f556bec0b" alt="imagen" width="450">
</p>

### 2. Descripción la señal en cuanto a su clasificación 

La señal electromiográfica (EMG) es el registro de la actividad eléctrica generada por los músculos esqueléticos. Se clasifica como una señal biomédica no estacionaria y altamente variable debido a factores como la activación muscular, la fatiga, la calidad de los electrodos y el ruido ambiental. Su análisis en los dominios temporal y espectral permite extraer información relevante para aplicaciones como el control de prótesis, el diagnóstico de trastornos neuromusculares y el estudio del rendimiento deportivo. En el análisis temporal, se evalúan parámetros como la amplitud y la duración de los potenciales de acción de las unidades motoras. En el análisis espectral, técnicas como la transformada de Fourier y el análisis wavelet permiten descomponer la señal y caracterizar su dinámica.[3][4]

### 3. Tranformada de Fourier
La transformada de Fourier permite convertir una señal del dominio del tiempo al dominio de la frecuencia.

### Fórmula de la Transformada de Fourier Discreta (DFT):

$$
X[k] = \sum_{n=0}^{N-1} x[n] e^{-j 2 \pi k n / N}
$$

Donde:

- \(X[k]\) es la representación en frecuencia de la señal.
- \(x[n]\) es la señal original en el dominio del tiempo.
- \(N\) es el número total de muestras.
- \(e^{-j 2 \pi k n / N}\) representa la base exponencial compleja.

La DFT utiliza una suma ponderada de las muestras de la señal con bases exponenciales complejas para transformar la señal desde el tiempo hacia el dominio de la frecuencia.

#### 3.1. Grafica de la transformada de fourier
El siguiente código muestra cómo calcular y graficar la transformada de Fourier de una señal:
```python
N = len(señal)
frecuencias = np.fft.fftfreq(N, 1/fs)
transformada = np.fft.fft(señal) / N
magnitud = (2 * np.abs(transformada[:N//2]))**2

plt.figure(figsize=(10, 5))
plt.plot(frecuencias[:N//2], np.abs(transformada[:N//2]), color='black')
plt.title("Transformada de Fourier de la Señal")
plt.xlabel("Frecuencia (Hz)")
plt.ylabel("Magnitud")
plt.grid()
```
- np.fft.fft: Calcula la transformada de Fourier de la señal.
- np.fft.fftfreq: Devuelve las frecuencias correspondientes a cada componente de la transformada.
- N//2: Se utiliza para considerar únicamente las frecuencias positivas.
- plt.plot: Genera una gráfica de la magnitud de la transformada.

Esta gráfica muestra las frecuencias presentes en la señal y su magnitud asociada.

<p align="center">
    <img src="https://github.com/user-attachments/assets/bbd94695-07fa-475c-8b62-6d6f98bfd948" alt="imagen" width="450">
</p>

#### 3.2. Grafica de la densidad espectral
En la práctica, para señales discretas y de duración finita, la DEP se estima utilizando la transformada de Fourier discreta (DFT). Al calcular la DFT de una señal y normalizar adecuadamente, se obtiene una estimación de su densidad espectral de potencia. Esta estimación permite identificar las frecuencias predominantes y analizar cómo se distribuye la energía de la señal en el dominio de la frecuencia.
```python
plt.figure(figsize=(10, 5))
plt.plot(frecuencias[:N//2], magnitud, color='black')
plt.xlabel('Frecuencia (Hz)')
plt.ylabel('Potencia')
plt.title('Densidad espectral de la señal')
plt.grid()

plt.show()
```

<p align="center">
    <img src="https://github.com/user-attachments/assets/d5f9a88b-1baf-47e8-86c6-3f8b68610271" alt="imagen" width="450">
</p>

- La Densidad Espectral de Potencia (DSP) mide la potencia de una señal en función de la frecuencia.[5]
- magnitud: Representa la potencia de cada frecuencia, calculada como el cuadrado de la magnitud de la transformada de Fourier.
Ambas gráficas son fundamentales para comprender el comportamiento de la señal en el dominio de la frecuencia. La primera da información sobre las frecuencias presentes, mientras que la segunda muestra cómo se distribuye la energía de la señal en esas frecuencias.

----
## Conclusión

- La comparación entre correlación y convolución resaltó que la correlación mide la similitud sin invertir la señal, mostrando la independencia entre señales senoidales y cosenoidales.
- El análisis de señales EMG en los dominios de tiempo y frecuencia nos permite caracterizar su comportamiento, y la DFT es crucial para identificar las distribuciones de frecuencia y potencia dominantes.

----
## Bibliografias
- [1] Chourpiliadis C, Bhardwaj A. Physiology, Respiratory Rate. [Updated 2022 Sep 12]. In: StatPearls [Internet]. Treasure Island (FL): StatPearls Publishing; 2025 Jan-. Available from: https://www.ncbi.nlm.nih.gov/books/NBK537306/
- [2] 
- [3] 
- [4] https://scielo.isciii.es/scielo.php?script=sci_arttext&pid=S1137-66272009000600003
- [5] https://prezi.com/p/cwcmwut1n1fx/densidad-espectral-de-potencia/#:~:text=La%20Densidad%20Espectral%20de%20Potencia%20(DSP)%20mide%20la%20potencia%20de,frecuencias%20en%20una%20se%C3%B1al%20analizada
----
## Autores 
- 
- Daniel Herrera
- Santiago Mora
