Código Fuente
A.1. Firmware del Arduino UNO (C++)
Firmware encargado de muestrear la entrada analógica A0 a una frecuencia fija de 4 Hz (21 muestras en aproximadamente 5 segundos) y enviarlas por el puerto serie a 115200 baudios, delimitando cada bloque de datos con las marcas "INICIO" y "FIN" para su sincronización con MATLAB.
int pinEntrada = A0;      // Pin donde conectas la salida del circuito
int muestrasTotales = 21; // Cantidad de muestras a tomar (aprox 5 segundos)
int retardo = 250;        // Tiempo entre muestras en milisegundos (4 muestras por segundo)
 
void setup() {
  // Iniciar la comunicación serial a la velocidad que pide MATLAB
  Serial.begin(115200);
}
 
void loop() {
  // 1. Avisar a MATLAB que vamos a empezar a enviar datos
  Serial.println("INICIO");
 
  // 2. Tomar las muestras y enviarlas
  for (int i = 0; i < muestrasTotales; i++) {
    int valorADC = analogRead(pinEntrada); // Leer el pin A0 (da un valor de 0 a 1023)
 
    // Convertir el número del Arduino a Voltaje real (0 a 5V)
    float voltaje = valorADC * (5.0 / 1023.0);
 
    // Enviar el voltaje por el cable USB a la computadora
    Serial.println(voltaje, 4); // El '4' es para enviar 4 decimales de precisión
 
    // Esperar un momento antes de tomar la siguiente muestra
    delay(retardo);
  }
 
  // 3. Avisar a MATLAB que ya terminamos de enviar el bloque de datos
  Serial.println("FIN");
 
  // Esperar 10 segundos antes de volver a repetir todo el proceso
  // (Te da tiempo de ver la gráfica en MATLAB)
  delay(10000);
}



