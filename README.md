# 🧪 Reto Técnico -- Microservicios

## **Postulación: Desarrollador Fullstack Senior**

Este examen práctico evalúa tus habilidades en **arquitectura de
microservicios**, **.NET 8/9**, **mensajería con RabbitMQ**, **cliente
web React**, **procesamiento asíncrono**, **implementación de
colas**, **trazabilidad**, **persistencia en PostgreSQL**,
**notificaciones por correo** y buenas prácticas de desarrollo.

El reto simula un flujo real de **carga masiva de datos**, completamente
distribuido, siguiendo la arquitectura mostrada en el diagrama
entregado.

<img width="1812" height="861" alt="image" src="https://github.com/user-attachments/assets/c4491384-ba6d-41c4-9347-7d7255ee38a6" />

------------------------------------------------------------------------

# 🚀 **1. Objetivo del Reto**

Implementar un sistema de microservicios donde un cliente web permita
subir un archivo Excel con información masiva, este archivo se procese
de manera asíncrona mediante colas RabbitMQ y finalmente se envíe una
notificación por correo al usuario una vez que la carga haya finalizado.

El reto debe ser **100% funcional**, **dockerizado** y siguiendo buenas
prácticas **senior**.

------------------------------------------------------------------------

# 🏗️ **2. Arquitectura General a Implementar**

La solución completa consiste en:

### ✔️ **1. Cliente Web React**

Permite: 
    - Subir un archivo Excel (.xlsx). 
    - Consultar el historial de cargas. 
    - Ver el estado de cada procesamiento: 
        1. **Pendiente** 
        2. **En proceso** 
        3. **Cargado** 
        4. **Finalizado** 
        5. **Notificado**

------------------------------------------------------------------------

### ✔️ **2. API Gateway (NET 8/9)**

Punto de entrada centralizado que: - Recibe todas las solicitudes del
cliente web. - Valida JWT. - Reenvía peticiones al microservicio
correspondiente.

------------------------------------------------------------------------

### ✔️ **3. Microservicio 1 -- Control / Publicador (NET 8/9)**

Funciones: - Recibe desde el Gateway la solicitud para cargar el
archivo. - Guarda trazabilidad del archivo (estado inicial:
**Pendiente**). - Publica un mensaje en RabbitMQ para que el archivo sea
procesado. - Envía el archivo al servicio de almacenamiento seaweedFS.

------------------------------------------------------------------------

### ✔️ **4. Microservicio 2 -- Carga Masiva (Consumidor / Publicador) (NET 8/9)**

Responsabilidades: - Escucha la cola RabbitMQ. - Descarga el archivo
desde SeaweedFS. - Procesa registro por registro. - Inserta la
información en PostgreSQL. - Marca la trazabilidad en estados: - **En
proceso** - **Cargado** - **Finalizado** - Publica una notificación en
una segunda cola RabbitMQ indicando que el proceso ha terminado.

------------------------------------------------------------------------

### ✔️ **5. Microservicio 3 -- Notificaciones (Consumidor) (NET 8/9)**

-   Escucha la cola de notificaciones.
-   Envía un correo al usuario indicando que la carga finalizó.
-   Usa MailKit.
-   Actualiza el estado final a **Notificado**.

------------------------------------------------------------------------

### ✔️ **6. RabbitMQ**

-   Cola 1: `carga_masiva`
-   Cola 2: `notificaciones`

------------------------------------------------------------------------

### ✔️ **7. Base de Datos -- PostgreSQL**

Tablas sugeridas: - `CargaArchivo` (trazabilidad) - `DetalleCarga` (si
fuera necesario) - `DataProcesada` (registros extraídos del Excel)

------------------------------------------------------------------------

### ✔️ **8. SeaweedFS**

Servicio distribuido para almacenar los archivos Excel subidos.

------------------------------------------------------------------------

------------------------------------------------------------------------

# 📌 **3. Flujo Completo del Caso de Uso**

------------------------------------------------------------------------

### **1️⃣ El usuario sube un Excel desde el cliente web**

-   El archivo se envía al **API Gateway**.
-   El Gateway reenvía la solicitud al **Microservicio de Control**.

------------------------------------------------------------------------

### **2️⃣ Microservicio de Control**

-   Guarda un registro en PostgreSQL como:
    -   `Pendiente`
-   Sube el archivo a **SeaweedFS**.
-   Publica en RabbitMQ:

``` json
{
  "idCarga": 123,
  "rutaArchivo": "seaweed://.../archivo.xlsx",
  "usuario": "user@example.com"
}
```

------------------------------------------------------------------------

### **3️⃣ Microservicio de Carga Masiva**

-   Consume el mensaje.
-   Actualiza el estado → **En proceso**.
-   Descarga el archivo.
-   Procesa todas las filas del Excel.
-   Inserta datos en PostgreSQL.
-   Estado → **Finalizado**.
-   Publica mensaje de notificación:

``` json
{
  "idCarga": 123,
  "usuario": "user@example.com",
  "fechaFin": "2025-02-10T10:20:00"
}
```

------------------------------------------------------------------------

### **4️⃣ Microservicio de Notificaciones**

-   Lee la notificación.
-   Envía correo con MailKit.
-   Actualiza el estado final → **Notificado**.

------------------------------------------------------------------------

### **5️⃣ Cliente Web**

-   Muestra el historial de cargas.
-   Permite ver estados en tiempo real (mediante pooling).

------------------------------------------------------------------------

------------------------------------------------------------------------

# 📦 **4. Requerimientos Técnicos Obligatorios**

## **Backend -- Todos los microservicios**

-   Lenguaje: **NET 8 o NET 9**
-   Arquitectura limpia
-   CQRS o Inversión de dependencias
-   SOLID
-   JWT (Refresh token opcional pero valorado)
-   Manejo de excepciones global
-   Logging estructurado
-   Dockerfile propio para cada microservicio
-   docker-compose general orquestando:
    -   todos los microservicios
    -   rabbitmq
    -   seaweedfs
    -   postgres
    -   gateway

------------------------------------------------------------------------

## **Frontend**

-   React 16+
-   Uso de componentes
-   Pantallas requeridas:
    \### 1. Subida de Excel
    \### 2. Historial de cargas (tabla)
    \### 3. Detalle del estado de una carga

------------------------------------------------------------------------

## **Base de datos**

Debe incluir migraciones automáticas.

------------------------------------------------------------------------

## **Mensajería**

RabbitMQ: - Intercambio directo o topic - Mínimo 2 colas

------------------------------------------------------------------------

## **Almacenamiento**

SeaweedFS: - Servicio dockerizado - Endpoint para subir archivos

------------------------------------------------------------------------

## **Correo**

-   Usar MailKit
-   Configurable por variables de entorno

------------------------------------------------------------------------

------------------------------------------------------------------------

# 📊 5. Estructura sugerida de la base de datos
`Script referencial`
``` sql
CREATE TABLE CargaArchivo (
    Id SERIAL PRIMARY KEY,
    NombreArchivo VARCHAR(200),
    Usuario VARCHAR(150),
    FechaRegistro TIMESTAMP,
    Estado VARCHAR(50),
    FechaFin TIMESTAMP NULL
);
```

------------------------------------------------------------------------

# 🔥 **6. Criterios de Evaluación**

### **1. Arquitectura (25%)**

-   Microservicios independientes
-   Limpieza del código
-   Manejo de colas y estados

### **2. Funcionalidad (35%)**

-   Flujo completo operativo
-   Procesamiento real del Excel
-   Persistencia correcta

### **3. Docker / DevOps (20%)**

-   docker-compose funcional
-   Servicios se levantan sin errores (opcional)

### **4. Frontend (20%)**

-   Interfaz limpia y funcional
-   Manejo correcto de estados
-   UX básica pero consistente

------------------------------------------------------------------------

# 📬 **7. Entrega final**

El postulante debe entregar un repositorio con:

### ✔️ Código fuente completo

### ✔️ Documentación en README

### ✔️ Instrucciones de despliegue (opcional)

### ✔️ Scripts de base de datos

### ✔️ Postman collection (opcional)

### ✔️ Video corto (máximo 5 minutos) mostrando flujo completo funcionando

------------------------------------------------------------------------

# 🎯 **8. Resultado esperado**

Una solución funcional, modular, distribuida, escalable y construida con
estándares SENIOR.

Este reto está diseñado para verificar tu dominio práctico sobre: -
microservicios - colas - .NET - React - procesamiento masivo -
asincronía - docker - arquitectura limpia

------------------------------------------------------------------------

# ✅ MUCHA SUERTE
