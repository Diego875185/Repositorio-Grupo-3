# Entregable 8

## Prototipo de Dispositivo en el Tobillo

### Ficha resumen

- **Nombre del proyecto:**  
  *Análisis y propuesta de una ortesis inteligente tobillo-pie.*

- **Problema que resuelve:**  
  Mi prototipo resuelve el control de la dorsiflexión mediante un sensor flex que detecta cuándo ocurre este movimiento. Esto permite monitoreo fácil y visual del progreso o necesidad de corrección.

- **Usuario final objetivo:**  
  Personas con debilidad o parálisis en el músculo tibial anterior, como pacientes con pie caído por lesión neurológica, que requieren asistencia o monitoreo en la dorsiflexión durante la marcha.

- **Materiales usados:**  
  El prototipo fue elaborado con cartón para simular la estructura flexible del TPE (elastómero termoplástico), dos tapas de botella para representar las zonas de anclaje y un hilo o pili que simula el funcionamiento del sistema BOA, permitiendo el ajuste del dispositivo al pie del usuario.

- **Observaciones o limitaciones del modelo:**  
  Actualmente, el prototipo no puede ser utilizado con zapatillas, ya que su diseño está pensado para usarse directamente sobre el pie, lo cual restringe su funcionalidad en entornos reales de uso diario.

---

## Maqueta física

### Vista 1: Vista oblicua anterolateral

En esta vista se puede observar la estructura general del dispositivo, que comprende una superficie tipo férula que envuelve el tobillo-pie y se extiende hacia la pantorrilla. Se ha simulado el uso de materiales ligeros y resistentes como TPE o TPU, anticipando una futura fabricación mediante impresión 3D.

> **Figura 1.** Boceto físico de órtesis en vista oblicua anterolateral.

---

### Vista 2: Vista Frontal con Etiqueta "Sensor Flex"

Aquí se destaca la posible ubicación del sensor de flexión (Flex Sensor) a nivel anterior del tobillo, que permitirá medir los ángulos de dorsiflexión durante la fase de marcha.  
La colocación estratégica busca registrar el movimiento sin interferir con la comodidad ni con el calzado. Se ha dejado un espacio en la férula que facilite su integración mediante una pequeña ranura o encapsulado.

> **Figura 2.** Integración propuesta de sensor de flexión en el empeine para monitoreo del ángulo de dorsiflexión durante la marcha.

---

### Vista 3: Vista Lateral con Etiqueta "Arduino Nano"

Esta imagen muestra el costado lateral donde se plantea ubicar un microcontrolador Arduino Nano, que permitirá recolectar datos desde los sensores y eventualmente controlar actuadores o ESP32.  
Su ubicación fuera del rango de movimiento articular busca evitar interferencias biomecánicas, y se reserva una cavidad que facilitará la protección del circuito con un encapsulado resistente al sudor y humedad.

> **Figura 3.** Propuesta de ubicación lateral del microcontrolador (Arduino Nano).

---

## Reflexión

- **¿Qué representaron en la maqueta y por qué?**  
  En la maqueta representamos el sistema BOA utilizando cartón, dos tapas y un hilo o pili. Este diseño buscó simular el mecanismo de ajuste que permite adaptar la presión sobre el pie de manera precisa. Elegimos representarlo así porque queríamos mostrar cómo funcionaría el sistema antes de desarrollar el prototipo funcional con componentes electrónicos.

- **¿Qué fue difícil de construir?**  
  Lo más difícil fue tomar correctamente las medidas del pie. Tuvimos que asegurarnos de que el modelo fuera cómodo, se adaptara bien a la forma del pie y que los materiales no causaran molestias. Esto implicó varios ajustes para lograr una buena sujeción sin incomodar al usuario.

- **¿Qué harían distinto en el paso siguiente (prototipo funcional)?**  
  En el siguiente paso, al construir el prototipo funcional, usaremos un Arduino Nano, un sensor flex y una pulsera. Mejoraríamos el diseño para que sea más ergonómico y resistente, usando materiales más duraderos. También optimizaríamos la colocación de los componentes electrónicos para asegurar tanto su funcionamiento correcto como la comodidad del usuario.
