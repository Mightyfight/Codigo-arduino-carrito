# Codigo-arduino-carrito
const int Trigger = 2;   // Pin digital 2 para el Trigger del sensor
const int Echo = 3;      // Pin digital 3 para el Echo del sensor
const int numMuestras = 10;  // Número de muestras en 2 segundos
long distancias[numMuestras]; // Array para almacenar las distancias medidas
int contador = 0;             // Contador para llevar el seguimiento de las muestras recibidas
const int Bomba = 11;

int distancia_deseada = 0;


void setup() {
  Serial.begin(9600); // Iniciamos la comunicación
  pinMode(Trigger, OUTPUT); // Pin como salida
  pinMode(Echo, INPUT);  // Pin como entrada
  digitalWrite(Trigger, LOW); // Inicializamos el pin con 0
  pinMode(Bomba, OUTPUT);
  Serial.println("Ingresa la distancia deseada en centímetros: ");
  while (!Serial.available()) {
    // Espera a que el usuario ingrese la distancia deseada
  }
  delay(100); // Pequeño retardo para asegurar la transmisión completa de datos
  readSerialInput();
}

void loop() {
  long t;  // Tiempo que demora en llegar el eco
  long d;  // Distancia en centímetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);   // Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);

  t = pulseIn(Echo, HIGH); // Obtenemos el ancho del pulso
  d = 31 - (t / 58.822352);              // Escalamos el tiempo a una distancia en cm

  distancias[contador] = d;
  contador++;

  // Si han pasado 2 segundos, calcular y mostrar la mediana
  if (contador == numMuestras) {
    long distanciaMediana = calcularMediana();
    Serial.print(distanciaMediana);
    Serial.print(", ");
    Serial.println(distanciaMediana < distancia_deseada ? HIGH : LOW);
    if (distanciaMediana <= distancia_deseada-1) {
      digitalWrite(Bomba, HIGH); // Enciende la BOMBA si la distancia es menor que la deseada
       
    }
    if (distanciaMediana >= distancia_deseada+1) {
      digitalWrite(Bomba, LOW); // Apaga la BOMBA si la distancia es mayor que la deseada
       
    }

    // Reiniciar el contador y el array
    contador = 0;
    delay(500);  // Esperar 0.5 segundos antes de la próxima medición
  }

  delay(100);  // Esperar 0.1 segundos antes de la próxima medición
  readSerialInput();
}

long calcularMediana() {
  // Ordenar el array de distancias
  for (int i = 0; i < numMuestras - 1; i++) {
    for (int j = 0; j < numMuestras - i - 1; j++) {
      if (distancias[j] > distancias[j + 1]) {
        // Intercambiar elementos si no están en orden
        long temp = distancias[j];
        distancias[j] = distancias[j + 1];
        distancias[j + 1] = temp;
      }
    }
  }

  // Calcular la mediana
  if (numMuestras % 2 == 0) {
    // Si hay un número par de elementos, promediar los dos valores centrales
    return (distancias[numMuestras / 2 - 1] + distancias[numMuestras / 2]) / 2;
  } else {
    // Si hay un número impar de elementos, devolver el valor central
    return distancias[numMuestras / 2];
  }
}

void readSerialInput() {
  while (Serial.available()) {
    delay(10); // Pequeño retardo para asegurar la transmisión completa de datos
    String input = Serial.readStringUntil('\n');
    if (input.length() > 0) {
      int newDistance = input.toInt();
      if (newDistance != 0) {
        distancia_deseada = newDistance;
        Serial.print("Nuevo valor de distancia deseada: ");
        Serial.println(distancia_deseada);
      }
    }
  }
}

