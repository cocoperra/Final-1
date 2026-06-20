# Flujo de autenticación

## Intro
    En este documento se explicará el flujo que sigue la autenticación de la aplicación de gestión de jugadores. Se ha adoptado un enfoque fullstack, esto es, tendré en cuenta no solo el flujo, la lógica de negocio y las interacciones con la BBDD, sino que tendré en cuenta los eventos del frontend que dan como origen esa petición.

### Paso 1.- login
        Cuando entramos en el React de Login.jsx, nos encontramos con un formulario. Al enviar el usuario y la contraseña, el controlador de evento de envío (handleSubmit) hace una llamada al endpoint POST /token.
        En caso de tener credenciales correctas, genera el JWT. En caso contrario, envía un 401.

        Con un login correcto, recibimos el token. Con el susodicho, el frontend realiza una segunda llamada a /me. Ojo, este endpoint construye el perfil de la sesión (me extenderé en este detalle más abajo)

### Paso 2.- Endpoint: token
    Proviene de la biblioteca rest_framework_simplejwt. No me extenderé demasiado.

### Paso 3.- Endpoint: me

    
    El endpoint /api/me es gestionado por la vista me_view:

        Definiendo los permisos personalizados y de vista (más en detalle cuando lleguemos a ellos), construye un objeto de sesión, tal como sigue:
            
    Response({
        "username": user.username,
        "is_staff": user.is_staff,
        "is_superuser": user.is_superuser,
        "groups": [group.name for group in user.groups.all()],
        "permisos": list(permisos.values('categoria', 'subcategoria', 'equipo')),
        "vistas": list(vistas),
    })

### Paso 4.- Endpoint: token/refresh

        Proviene de la misma librería que el endpoint token. {{UN DETALLE IMPORTANTE: NO LO ESTOY USANDO}}

### Paso 5.- Endpoint: /logout


La aplicación no dispone de un endpoint de logout.

Actualmente el cierre de sesión se realiza exclusivamente en el frontend:

```javascript
const handleLogout = () => {
    sessionStorage.clear();
    navigate("/");
};
```