ShopNow - Tienda Virtual Android 🛍️
![Android](https://developer.android.com)
![Java](https://www.java.com)
![Firebase](https://firebase.google.com)
![License](#)
Aplicación móvil nativa para Android desarrollada como proyecto académico en la
Fundación Universitaria Compensar — Programa de Ingeniería de Sistemas.
---
📱 Descripción
ShopNow es un marketplace de moda y accesorios que conecta vendedores
independientes con compradores, bajo una arquitectura MVVM + Repository Pattern.
Roles del sistema
Rol	Funcionalidades principales
Administrador	Dashboard KPIs, CRUD usuarios, gestión productos, reportes de ventas
Vendedor	Panel tienda, CRUD productos + cámara, historial pedidos
Comprador	Catálogo, detalle producto, carrito, pasarela Wompi, mapa geolocalización
---
🏗️ Arquitectura
```
com.shopnow.app/
├── model/          # Entidades de datos (User, Product, CartItem, Order)
├── repository/     # Fuentes de verdad: Firestore, Room, Firebase Storage
├── viewmodel/      # ViewModels con LiveData (patrón MVVM)
├── ui/
│   ├── auth/       # Login, Registro, Recuperar contraseña
│   ├── admin/      # Dashboard, Usuarios, Productos, Reportes
│   ├── seller/     # Mi Tienda, CRUD Productos, Pedidos
│   ├── buyer/      # Inicio, Catálogo, Detalle, Carrito, Pago
│   └── shared/     # Perfil, Mapa (compartidos entre roles)
└── utils/          # AuditLogger, LocationHelper, WompiPaymentHelper, FCMService
```
Patrón MVVM
```
Fragment (UI)  →  ViewModel  →  Repository  →  Firebase / Room
     ↑                ↓
  LiveData    observa cambios
```
---
🛠️ Stack Tecnológico
Capa	Tecnología
Lenguaje	Java 17
Min SDK	Android 10 (API 29)
Target SDK	Android 14 (API 34)
Base de datos	Firebase Firestore + Room Database
Almacenamiento	Firebase Storage
Autenticación	Firebase Auth + Google OAuth + BiometricPrompt
Notificaciones	Firebase Cloud Messaging (FCM)
Pagos	Wompi API
Geolocalización	Google Maps SDK + FusedLocationProvider
Cámara	CameraX API
Carga de imágenes	Glide 4.16
HTTP	OkHttp + Retrofit 2
Navegación	Android Navigation Component
Arquitectura UI	Material Design 3
---
🚀 Configuración del proyecto
1. Clonar el repositorio
```bash
git clone https://github.com/tu-usuario/ShopNow.git
cd ShopNow
```
2. Configurar Firebase
Ir a console.firebase.google.com
Crear proyecto → Agregar app Android con package `com.shopnow.app`
Descargar `google-services.json` y colocarlo en `app/`
Activar en Firebase Console:
Authentication → Email/Password y Google
Firestore Database → modo prueba
Storage → modo prueba
Cloud Messaging
3. Configurar Google Maps
Crear o editar `local.properties`:
```properties
MAPS\\\_API\\\_KEY=tu\\\_api\\\_key\\\_de\\\_google\\\_maps
```
En `app/build.gradle` dentro de `defaultConfig`:
```groovy
manifestPlaceholders = \\\[MAPS\\\_API\\\_KEY: project.findProperty("MAPS\\\_API\\\_KEY") ?: ""]
```
4. Configurar Wompi (pagos)
En `WompiPaymentHelper.java` reemplazar:
```java
private static final String PUBLIC\\\_KEY\\\_SANDBOX = "pub\\\_test\\\_TU\\\_LLAVE\\\_PUBLICA";
```
> ⚠️ La llave \\\*\\\*privada\\\*\\\* de Wompi NUNCA debe incluirse en el código Android.
5. Sincronizar y compilar
```
File → Sync Project with Gradle Files
Build → Make Project
```
---
🔒 Firestore Security Rules
```javascript
rules\\\_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Usuarios: solo el propio o Admin
    match /users/{userId} {
      allow read, update: if request.auth != null \\\&\\\&
        (request.auth.uid == userId ||
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.rol == 'ADMIN');
      allow create: if request.auth != null;
    }

    // Productos: todos leen, solo propietario o Admin escribe
    match /products/{productId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null \\\&\\\&
        (resource.data.vendedorId == request.auth.uid ||
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.rol == 'ADMIN');
    }

    // Órdenes: el comprador crea y ve las suyas, Admin ve todas
    match /orders/{orderId} {
      allow create: if request.auth != null;
      allow read: if request.auth != null \\\&\\\&
        (resource.data.compradorId == request.auth.uid ||
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.rol == 'ADMIN');
      allow update: if request.auth != null \\\&\\\&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.rol == 'ADMIN';
    }

    // Logs de auditoría: solo Admin lee
    match /auditLogs/{logId} {
      allow create: if request.auth != null;
      allow read: if request.auth != null \\\&\\\&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.rol == 'ADMIN';
    }
  }
}
```
---
📂 Estructura de datos Firestore
Colección `users`
```json
{
  "uid": "string",
  "nombre": "string",
  "apellido": "string",
  "correo": "string",
  "celular": "string",
  "rol": "ADMIN | SELLER | BUYER",
  "estado": "active | inactive",
  "fotoUrl": "string (Firebase Storage URL)",
  "direccion": "string",
  "fechaCreacion": "timestamp (ms)",
  "intentosFallidos": "number",
  "bloqueadoHasta": "timestamp (ms)"
}
```
Colección `products`
```json
{
  "productoId": "string",
  "vendedorId": "string (UID)",
  "nombre": "string",
  "descripcion": "string",
  "precio": "number (COP)",
  "categoria": "string",
  "stock": "number",
  "imagenesUrl": \\\["string"],
  "activo": "boolean",
  "eliminado": "boolean",
  "fechaCreacion": "timestamp (ms)",
  "vistas": "number"
}
```
Colección `orders`
```json
{
  "orderId": "string",
  "compradorId": "string (UID)",
  "items": \\\[{ "productoId": "string", "nombre": "string", "cantidad": "number", "precioUnitario": "number" }],
  "subtotal": "number",
  "ivaValor": "number",
  "total": "number",
  "direccionEntrega": "string",
  "estado": "pendiente | pagado | enviado | entregado | cancelado",
  "metodoPago": "TARJETA | PSE | EFECTY",
  "wompiTransaccionId": "string",
  "fechaCreacion": "timestamp (ms)",
  "fechaPago": "timestamp (ms)"
}
```
---
👥 Integrantes
Nombre	Rol en el proyecto
Laura Tatiana rincon acosta	UI/UX + Frontend
Laura Tatiana rincon acosta	Backend + Firebase
Laura Tatiana rincon acosta	Integración + Testing
---
📚 Referencias
Android Developers Documentation
Firebase Documentation
Wompi API Docs
Stack Overflow - Android Studio
Android Arsenal
---
Proyecto académico — Fundación Universitaria Compensar 2025
