# SafeShare

Sistema de transferencia cifrada de archivos con autorización por políticas y control aproximado de presencia en zonas autorizadas.

## Descripción

SafeShare es un prototipo que estamos desarrollando para controlar la transferencia de archivos dentro de aulas, laboratorios u oficinas.

El sistema estará compuesto por dos aplicaciones de escritorio para Windows, un servidor local de políticas y un agente de zona implementado con una placa ESP32. Antes de autorizar una transferencia, el servidor verificará:

* La identidad del usuario.
* El dispositivo registrado.
* El rol del emisor y del receptor.
* El BSSID de la red Wi-Fi.
* El código temporal anunciado por el ESP32 mediante BLE.
* El tipo y tamaño del archivo.
* La política aplicable a la solicitud.

Si todas las condiciones se cumplen, el receptor deberá aceptar expresamente la solicitud. Después de la aceptación, el archivo se enviará directamente entre las dos computadoras mediante Wi-Fi y cifrado AES-256-GCM.

El servidor se encargará de validar la operación y registrar metadatos mínimos de auditoría, pero no recibirá ni almacenará el contenido del archivo.

La versión 0.1 estará limitada a una zona de prueba, dos computadoras Windows, un agente ESP32 y archivos individuales de hasta 50 MB.

## Estado actual

* Versión del prototipo: **0.1 MVP**
* Estado: **En desarrollo**
* Nivel de madurez actual: **TRL 2**
* Nivel de madurez objetivo: **TRL 4**
* Entorno previsto de validación: **Laboratorio**

Actualmente se encuentran definidos el alcance, la arquitectura, los componentes y los casos de prueba. El código fuente será incorporado progresivamente en este repositorio.

## Integrantes

* Melanie Jared Barrios Lopez
* Shanella Rosita Begazo Sullca
* Alexia Linsay Ccosi Mamani
* Angie Estefania Chullo Mamani
* Marco Albert Cori Herrera

## Tecnologías utilizadas

| Tecnología      | Uso dentro del proyecto                                                 |
| --------------- | ----------------------------------------------------------------------- |
| Python 3.12     | Lenguaje principal del cliente y del servidor                           |
| PySide6         | Desarrollo de la interfaz gráfica                                       |
| FastAPI         | Implementación de la API del servidor local                             |
| Uvicorn         | Ejecución del servidor ASGI                                             |
| SQLite          | Almacenamiento de usuarios, dispositivos, zonas, políticas y auditorías |
| Pydantic        | Validación de solicitudes, respuestas y políticas                       |
| Bleak           | Detección de anuncios BLE desde Windows                                 |
| ESP32           | Emisión del código temporal de la zona                                  |
| python-zeroconf | Descubrimiento del receptor mediante mDNS/DNS-SD                        |
| cryptography    | X25519, HKDF-SHA-256, AES-256-GCM y SHA-256                             |
| pytest          | Pruebas unitarias y de integración                                      |
| Git y GitHub    | Control de versiones y trabajo colaborativo                             |

## Requisitos

### Software

* Windows de 64 bits.
* Python 3.12.
* Git.
* Visual Studio Code u otro editor compatible.
* Controladores de Wi-Fi y Bluetooth actualizados.
* Arduino IDE o ESP-IDF para programar la placa ESP32.

### Hardware

* Dos computadoras Windows con Wi-Fi y Bluetooth Low Energy.
* Una placa ESP32.
* Un cable USB compatible con la placa.
* Un punto de acceso o router Wi-Fi.
* Un equipo conectado a la misma red para ejecutar el servidor local.

## Estructura del repositorio

```text
11_SafeShare/
├── client/                 Aplicación de escritorio
├── server/                 API, políticas y auditoría
├── esp32/                  Firmware del agente de zona
├── database/
│   └── migrations/         Migraciones y scripts de SQLite
├── scripts/                Scripts de configuración y ejecución
├── tests/                  Pruebas del sistema
├── docs/
│   └── diagramas/          Diagramas y documentación técnica
├── .env.example            Variables de entorno de ejemplo
├── .gitignore              Archivos excluidos del repositorio
├── requirements.txt        Dependencias de Python
└── README.md               Documentación principal
```

## Instalación

### 1. Clonar el repositorio

Abrir PowerShell o la terminal de Visual Studio Code y ejecutar:

```powershell
git clone https://github.com/nombre-usuario/11_SafeShare.git
```

### 2. Ingresar a la carpeta del proyecto

```powershell
cd 11_SafeShare
```

### 3. Crear el entorno virtual

```powershell
py -3.12 -m venv .venv
```

### 4. Activar el entorno virtual

```powershell
.\.venv\Scripts\Activate.ps1
```

Si PowerShell bloquea la activación, ejecutar:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Después, volver a activar el entorno virtual:

```powershell
.\.venv\Scripts\Activate.ps1
```

### 5. Instalar las dependencias

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 6. Crear el archivo de configuración local

```powershell
Copy-Item .env.example .env
```

El archivo `.env` deberá contener la dirección del servidor, el puerto, la ruta de la base de datos y la configuración de la zona de prueba.

Este archivo no deberá subirse al repositorio porque puede contener datos privados, credenciales o claves de servicios externos.

## Ejecución

### Iniciar el servidor local

Desde la carpeta principal del proyecto:

```powershell
uvicorn server.main:app --host 0.0.0.0 --port 8000
```

El servidor deberá ejecutarse en un equipo conectado a la misma red Wi-Fi que los clientes.

### Iniciar el cliente de escritorio

En cada computadora cliente se deberá abrir una terminal, ingresar al proyecto, activar el entorno virtual y ejecutar:

```powershell
python -m client.main
```

Uno de los clientes actuará como emisor y el otro como receptor.

### Programar el agente ESP32

1. Conectar la placa ESP32 mediante USB.
2. Abrir el proyecto ubicado en la carpeta `esp32`.
3. Seleccionar la placa y el puerto correspondientes.
4. Configurar el identificador de la zona.
5. Compilar y cargar el firmware.
6. Verificar que el ESP32 anuncie el código temporal mediante BLE.

### Ejecutar las pruebas

```powershell
pytest
```

## Funcionamiento básico

1. El ESP32 anuncia un código temporal de zona mediante BLE.
2. Los clientes detectan el código y obtienen el BSSID de la WLAN.
3. El cliente emisor descubre al receptor mediante mDNS/DNS-SD.
4. El emisor selecciona un archivo y solicita autorización.
5. El servidor evalúa el usuario, dispositivo, zona, red, tipo y tamaño del archivo.
6. El receptor acepta o rechaza la solicitud.
7. Si la solicitud es autorizada, los clientes generan claves efímeras.
8. El archivo se cifra y se transfiere directamente al receptor.
9. El receptor verifica la autenticidad de los bloques y el hash final.
10. El servidor registra el resultado sin almacenar el archivo.