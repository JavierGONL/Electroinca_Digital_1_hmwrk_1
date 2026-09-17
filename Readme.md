# QuadGB

## Diagrama de Flujo Principal

El sistema se divide en tres estados principales: Verificación, Menu principal y Juego.

```mermaid
flowchart TD
    A[Inicio] --> B
    B[Inicializar display y entradas] --> C
    C[Intro y verificacion de sistemas]
    C --> D[Entrada PS2 del usuario ]
    D --> E{Menu Start}
    E --> I[Juego]
    E -->|Opciones| F[Menu de configuracion]
    I --> |Termino juego| E
    F --> H[Modificar brillo o controles] -->E
```

## Diagrama de flujo de la consola

Este flujo representa la secuencia completa del sistema desde el encendido. Las
pruebas del POST pueden ejecutarse en paralelo, pero el sistema central espera
la respuesta de todos los módulos antes de habilitar el menú.

```mermaid
flowchart TD
    A[Encender consola]
    A --> D
    D[Mostrar intro e Iniciar Verificaciones]
    D --> E[Probar memorias]
    D --> F[Probar perifericos PS2]
    D --> G[Probar salidas A/V]
    E --> H{Todas las pruebas responden}
    H -->|No| I[Registrar error]
    I --> J[Pantallazo azul]
    J --> K{Reiniciar consola}
    K -->|Si| A
    K -->|No| L[Esperar]
    H -->|Si| M[Input de Validacion Control]
    M --> N{Menu Principal}
    N -->|Jugar| O[Solicitar recursos del juego]
    N -->|Opciones| P[Configurar consola por I2C]
    O --> Q[Cargar datos de Flash a RAM]
    Q --> R[Inicializar estado del juego]
    R --> S[Ejecutar game loop]
    S --> T{Terminar juego}
    F --> H
    G --> H
    P --> M
    T -->|No| S
    T -->|Si| M
```

## Verificaciones de los sistemas al encender

Al encender el sistema, se ejecuta un diagnostico al hardware mientras se carga la secuencia una intro tipo ps3.

[Ejemplo intro](https://www.youtube.com/watch?v=Ywh-aIfEcew)

La idea es que mientras se este ejecutando esta intro, se ejecuten test de todos los modulos:

- Diagnóstico de Memoria: Verificación rápida de lectura/escritura en BRAM 

- Test de Periféricos: Detección de entradas conectadas mediante los controladores PS/2

- Test de salidas de video/audio.

cada test lo hace el grupo correspondiente y en el software al iniciarse simplemente se llaman.

## Menu Principal


### Menu
Interfaz de usuario en estado de espera para seleccionar juegos o ajustar parámetros de la consola.

<p align="center">
    <img src="Imagenes/Menu_Principal.png" alt="Menú principal">
</p>



## Juego (Game Loop)

```mermaid
flowchart TD
    A[Seleccionar juego] --> B[Solicitar recursos a memoria Flash]
    B --> C[Transferir recursos de Flash a RAM]
    C --> D[Inicializar estado y variables del juego]
    D --> E[Esperar sincronizacion de video]
    E --> F[Leer eventos de PS2 y otros controles]
    F --> G[Actualizar estado del jugador y del mundo]
    G --> H[Procesar fisica y colisiones]
    H --> I[Construir datos del siguiente cuadro]
    I --> J[Enviar cuadro al display driver]
    J --> K[Enviar eventos al modulo de audio I2S]
    K --> L{¿Terminar?}
    L -->|No| E
    L -->|Si| M[Regresar al menu]
```

- Carga de datos: Cargar los recursos necesarios desde la memoria.
- Logica principal: Procesar el estado del juego, físicas y colisiones.
- Entrada: Leer las acciones del jugador.
- Video: Actualizar los elementos graficos.
- Audio: Reproducir los sonidos del juego.

## Manejo de errores

Cada modulo debe informar si una operación termino correctamente o si ocurrio un error. En caso de fallo, se identifica el módulo, el código y la operación afectada.

Los errores pueden ser recuperables o críticos. Los recuperables permiten reintentar o volver al menú, los errores críticos detienen la operación y muestran una pantalla azul con la información del error. Si un módulo no responde a tiempo, se considera un error de comunicacion.

```mermaid
flowchart TD
    A[Iniciar operacion] --> B{Respuesta del modulo}
    B -->|Valida| C[Continuar operacion]
    B -->|Error| D[Recibir codigo y origen del error]
    B -->|Sin respuesta| E[Generar error de comunicacion]
    D --> F{Tipo de error}
    E --> F
    F -->|Recuperable| G[Reintentar o volver al menu]
    F -->|Critico| H[Detener operacion]
    H --> J[Pantallazo azul]
    J --> K[Mostrar el Error y de donde]
    K --> L{Reiniciar consola}
    L -->|Si| M[Reiniciar]
    L -->|No| N[Esperar]
```