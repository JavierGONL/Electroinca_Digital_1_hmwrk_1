# Proyecto Consola Digital - Arquitectura y Flujo

Este repositorio contiene la arquitectura de hardware para el desarrollo de una consola interactiva, basada en la integración de módulos periféricos y de procesamiento.

## Diagrama de Flujo Principal

El sistema se divide en tres estados principales: Verificación, Menu principal y Juego.

```mermaid
flowchart TD
    A(Inicio) --> B[Intro/verificacion de sistemas]
    B --> C{Menu Start}
    C <--> I[Juegos]
    C --> E{Opciones} -->D
    E --> G
    D[Salir] --> C
    G[Controles] --> F[Volver] --> E
    E --> H[Brillo] --> F
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
    D --> E[Probar memoria BRAM]
    D --> F[Probar perifericos PS2]
    D --> G[Probar salidas A/V]
    E --> H{Todas las pruebas responden}
    H -->|No| I[Registrar error]
    I --> J[Pantallazo azul]
    J --> K{Reiniciar consola}
    K -->|Si| A
    K -->|No| L[Esperar]
    H -->|Si| M[Menu principal]
    M --> N{Seleccion del usuario}
    N -->|Jugar| O[Cargar juego]
    N -->|Opciones| P[Configurar consola]
    O --> Q[Ejecutar game loop]
    Q --> R{Terminar juego}
    F --> H
    G --> H
    P --> M
    R -->|No| Q
    R -->|Si| M
```

## Verificaciones de los sistemas al encender

Al encender el sistema, se ejecuta un diagnóstico de hardware mientras se carga la secuencia una intro tipo ps3.

[Ejemplo](https://www.youtube.com/watch?v=Ywh-aIfEcew)

La idea es que mientras se este ejecutando esta intro, se ejecuten test de todos los modulos:

- Diagnóstico de Memoria: Verificación rápida de lectura/escritura en BRAM 

- Test de Periféricos: Detección de entradas conectadas mediante los controladores PS/2

- Test de salidas de video/audio.

cada test lo hace el grupo correspondiente y en el software al iniciarse simplemente se llaman.

### Manejo de errores

Cada función debe tener manejo de errores. El sistema central espera un output
de cada módulo; si no lo recibe, considera que el módulo falló, detiene la
operación actual y muestra una pantalla de error.

```mermaid
flowchart TD
    A[Inicio de función] --> B{El modulo devuelve output}
    B -->|Sí| C[Continuar operacion]
    B -->|No| D[Registrar tipo y código del error]
    D --> E[Detener operación actual]
    E --> F[Pantalla azul: Ocurrio un error]
    F --> G[Mostrar descripción del error]
    G --> H{Reiniciar}
    H -->|Si| I[Reiniciar consola]
    H -->|No| J[esperar que el usuario haga algo]
```

## Menu Principal y Configuracion


### Menu
Interfaz de usuario en estado de espera para seleccionar juegos o ajustar parámetros de la consola.

<p align="center">
    <img src="Imagenes/Menu_Principal.png" alt="Menú principal">
</p>

### Configuracion

<p align="center">
    <img src="Imagenes/Menu_Opciones.png" alt="Menú principal">
</p>

Interfaz Gráfica: Renderizado de menú estático a cargo del display driver (Grupo J).

Navegación: El controlador PS/2 (Grupo E) decodifica las pulsaciones del usuario para mover el cursor y seleccionar opciones.

Ajustes: Modificación de parámetros físicos (ej. brillo, volumen) transmitidos a la placa mediante I2C (Grupo H).

## Juego (Game Loop)

```mermaid
flowchart TD
    A[Seleccionar juego] --> B[Cargar recursos]
    B --> C[Inicializar juego]
    C --> D[Loop de juego ]

    D --> E[Leer entradas]
    E --> F[Actualizar estado]
    F --> G[Procesar física]
    G --> H[Detectar colisiones]
    H --> I[Actualizar gráficos]
    I --> J[Actualizar audio]

    J --> K{¿Terminar?}

    K -->|No| E
    K -->|Sí| L[Regresar al menú]
```

- Carga de datos: Cargar los recursos necesarios desde la memoria.
- Logica principal: Procesar el estado del juego, físicas y colisiones.
- Entrada: Leer las acciones del jugador.
- Video: Actualizar los elementos graficos.
- Audio: Reproducir los sonidos del juego.

## Referencias

- https://docs.espressif.com/projects/esp-idf/en/v4.4/esp32/api-guides/error-handling.html
- https://learn.microsoft.com/en-us/windows-hardware/drivers/whea/components-of-the-windows-hardware-error-architecture