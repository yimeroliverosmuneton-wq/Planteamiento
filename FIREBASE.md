# Guardar las cuentas en la nube (Firebase)

La página ya tiene inicio de sesión y creación de cuenta. Sin configuración, cada cuenta y sus datos se guardan en el navegador donde se crearon. Con Firebase (gratis, de Google), las cuentas y los datos quedan en la nube y se puede entrar desde cualquier celular o computador.

## Pasos (unos 10 minutos)

1. Entre a https://console.firebase.google.com con una cuenta de Google y toque **Crear un proyecto**. Nombre: `bella-cristal`. Puede desactivar Google Analytics.
2. **Authentication** → **Comenzar** → pestaña **Método de acceso** → active **Correo electrónico/contraseña** → Guardar.
3. **Authentication** → **Configuración** → **Dominios autorizados** → **Agregar dominio** → `yimeroliverosmuneton-wq.github.io`.
4. **Firestore Database** → **Crear base de datos** → ubicación `southamerica-east1` (o la que sugiera) → **Modo de producción**.
5. En Firestore → pestaña **Reglas**, reemplace todo por esto y toque **Publicar**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {

       function duenoTienda() {
         return request.auth != null &&
           get(/databases/$(database)/documents/tienda/catalogo).data.ownerUid == request.auth.uid;
       }

       // Datos privados de cada cuenta (costos, catálogo, facturas)
       match /usuarios/{uid}/{document=**} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }

       // Catálogo público de la tienda: lo lee cualquiera; lo publica solo la dueña
       // (la primera cuenta que publica queda como dueña).
       match /tienda/catalogo {
         allow read: if true;
         allow create: if request.auth != null && request.resource.data.ownerUid == request.auth.uid;
         allow update: if request.auth != null && resource.data.ownerUid == request.auth.uid
                       && request.resource.data.ownerUid == request.auth.uid;
         allow delete: if request.auth != null && resource.data.ownerUid == request.auth.uid;
       }
       match /tienda_fotos/{id} {
         allow read: if true;
         allow write: if duenoTienda();
       }

       // Carritos y pedidos: las clientas los crean sin cuenta; solo la dueña los lee.
       match /pedidos/{id} {
         allow read, delete: if duenoTienda();
         allow update: if duenoTienda() && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['estado']);
         allow create, update: if
           request.resource.data.keys().hasOnly(['nombre','telefono','ciudad','items','total','estado','creado','actualizado','acepta','nota'])
           && request.resource.data.nombre is string && request.resource.data.nombre.size() <= 80
           && request.resource.data.telefono is string && request.resource.data.telefono.size() >= 7
           && request.resource.data.telefono.size() <= 20
           && request.resource.data.items is list && request.resource.data.items.size() <= 40
           && request.resource.data.acepta == true
           && request.resource.data.estado in ['carrito', 'enviado']
           && (resource == null || resource.data.estado != 'atendido');
       }
     }
   }
   ```

   Así cada persona solo ve sus propios datos, el catálogo de la tienda es público, y los pedidos (con los teléfonos de las clientas) solo los ve la dueña de la tienda.
6. Engranaje ⚙ → **Configuración del proyecto** → en «Tus apps» toque el ícono **</>** (Web) → nombre `bella-cristal-web` → **Registrar app**. Copie el bloque `firebaseConfig` que aparece, algo así:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "bella-cristal.firebaseapp.com",
     projectId: "bella-cristal",
     storageBucket: "bella-cristal.appspot.com",
     messagingSenderId: "123...",
     appId: "1:123...:web:abc..."
   };
   ```
7. En `index.html`, la línea `var FIREBASE_CONFIG = ...` ya tiene la configuración del proyecto `bella-cristal-35f57` (la usan `index.html` y `tienda.html`). Estos datos no son secretos: la protección la dan las reglas del paso 5.

Al abrir la página, debajo del formulario dirá «Sus datos se guardan en la nube». Si alguien ya tenía datos en ese navegador sin cuenta, pasan automáticamente a la primera cuenta que se cree ahí.

## Tienda para clientas

La página `tienda.html` (https://yimeroliverosmuneton-wq.github.io/Planteamiento/tienda.html) es pública y no pide cuenta. Muestra las prendas que se publican desde la pestaña **Tienda** de la administración. La primera cuenta que toque **Publicar catálogo en la tienda** queda como dueña: solo esa cuenta puede volver a publicar y ver los pedidos.
