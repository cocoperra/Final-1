# Migración del sistema RBAC actual a un modelo de capabilities

## 1.- Estado actual del sistema.

    Actualmente la aplicación está usando un sistema híbrido de control de acceso compuesto por:

### 1.1- Roles (Django groups):

    -admin
    -staff
    -(otros grupos implícitos)
### 1.2- Permisos personalizados (PermisoPersonalizado)

    Estructura actual:

    ```json
    {
        "categoria": "SEN",
        "subcategoria": "A",
        "equipo": "..."
    }
    ```

### 1.3- Vistas (PermisoVista)

``` JSON
    ["calendario", "dashboard", "gestion_x"]
```

## 2.- Problemas del modelo actual.

### 2.1- Mezcla de conceptos

    Estamos mezclando roles (quien es el usuario), permisos (que puede hacer) y vistas (qué ves en la UI
    )

    Ejemplo:
    
    ``` javascript:
        const tienePermisoSen = 
            permisos.some(p=>
            p.categoria=="SEN" && p.subcategoria === "A"
        );
    ```
### 2.3- Feature flags no estructurados

    ``` javascript:
        vistas.includes("calendario")
    ```
## 3.- Nuevo modelo propuesto: Capabilities System

### 3.1.-Concepto
    Introducimos un único sistema de control basado en :
        Roles -> identidad del usuario
        Capabilities -> acciones permitidas
        Scope -> ámbito de aplicación

## 4.- Definición del nuevo modelo

### 4.1- Roles
    
    ´´´ JSON

        {
            "roles": ["admin", "coach", "player"]
        }
### 4.2- Capabilities

    ```JSON
    {
        "capabilities" : [
            "players.read",
            "players.write",
            "training.create",
            "training.manage",
            "stats.read",
            "calendar.manage",
            "attendance.confirm"
        ]
    }
    ```
### 4.3- Scope (opcional, pero recomendado)

    ```
    {
        "scope": {
            "players.read": "team",
            "stats.read": "self",
            "training.create": "team"
        }
    } 
    ```
## 5.- Mapping del sistema actual ---> capabilities

### 5.1.- SEN (primer equipo)

Antes 

``` javascript
    
    p.categoria === "SEN" && p.subcategoria === "A"
    ```
Después
    ```JSON
    {
    "capabilities": [
    "team.first.read",
    "team.first.manage"
    ]
}
```
## 6.- Migración del backend

### 6.1.- Nuevo endpoint /me:

Antes 

```Python
return Response({
    "groups": [...],
    "permisos": [...],
    "vistas": [...]
})
```
Después:

```Python
return Response({
    "user": {
        "username": user.username,
        "roles": list(user.groups.values_list("name," flat=True))
    },
    "capabilities" :[
        "players.read",
        "training.create",
        "calendar.read"
    ]
})
```
## 7. Migración del frontend

### 7.1 Antes (dashboard actual)

```javascript

const tienePermisoSen =....
const tienePermisoAcademia = ...
if(vistas.includes("calendario"))
```
### 7.2.- Después

``` javascript: 
const { capabilities } = useAuth();

const canSeeCalendar = capabilities.includes("calendar.read"):
const canSeeAcademy = capabilities.includes("team.academy.read");
```
### 7.3.- Generación de paneles

Antes:
```JavaScript

if(tienePermisoSen) paneles.push(...)
if(vistas.includes("calendario")) paneles.push
```
Ahora 
```javascript
const paneles = [
    capabilities.includes("team.first.read") && {
        key:"first-team",
        title: "Primer Equipo",
    },
    capabilities.includes("team.academy.read") && {
        key: "academy",
        title: "Academia",
    },
    capabilities.includes("calendar.read") && {
        key: "calendar",
        title: "Calendario"
    }
    
].filter(Boolean);
```

## 8.- Migración progresiva (importante)

### Fase 1

    Mantener permisos actuales

    Introducir capabilities paralelas en /me

### Fase 2

    Frontend empieza a usar capabilities

    Permisos legacy siguen existiendo

### Fase 3

    Eliminar "vistas"

    Deprecate PermisoPersonalizado si ya no aporta valor

## 9.- Resultado final del sistema

Antes:

    · Roles
    · Permisos
    · Vistas
    · Lógica en frontend
    · Lógica en backend mezclada

Después:

    · Roles -> identidad
    · Capabilities -> acceso
    · Scope -> contexto
    · Frontend -> render puro
    · Backend -> autoridad única

## 10.- Conclusión

El objetivo no es simplificar por simplificar, sino

Tener un único lenguaje de autorización coherente en todo el sistema

Esto reduce:

    · lógica duplicada
    · condiciones en frontend
    · inconsistencias entre módulos
    · complejidad al escalar features nuevas