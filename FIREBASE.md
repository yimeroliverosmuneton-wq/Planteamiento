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
       match /usuarios/{uid}/{document=**} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

   Así cada persona solo puede leer y escribir sus propios datos.
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
7. En `index.html`, busque la línea `var FIREBASE_CONFIG = null;` y reemplace `null` por el objeto copiado (desde `{` hasta `}`). Guarde y suba el cambio a GitHub. Estos datos no son secretos: la protección la dan las reglas del paso 5.

Al abrir la página, debajo del formulario dirá «Sus datos se guardan en la nube». Si alguien ya tenía datos en ese navegador sin cuenta, pasan automáticamente a la primera cuenta que se cree ahí.
