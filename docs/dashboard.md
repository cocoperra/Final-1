## Intro

    El Dashboard es el eje central de la aplicación. Desde aquí se despliegan todos los elementos desde los que el usuario final puede llegar a las distintas operaciones, desde crear jugadores, asignar fechas de partidos, crear entrenamientos, y registros de jornada.

    Renderiza distintos módulos en función de los roles y de los permisos.

### 1.- Guardar

    Recordemos que el endpoint /me devolvía un objeto de sesión. El componente de Dashboard lee de dicho objeto para obtener los permisos y los roles del usuario.
    En el código :
    
    ```javascript
    const [permisos, setPermisos] = useState([]);
    const [vistas, setVistas] = useState([]);
        useEffect(() => {
    const storedPerms = JSON.parse(
      sessionStorage.getItem("userPermisos") || "[]"
    );
    const storedVistas = JSON.parse(sessionStorage.getItem("userVistas") || "[]");
        setPermisos(storedPerms);
        setVistas(storedVistas);
    }, []);

  const tienePermisoSen = permisos.some((p) => p.categoria === "SEN" && p.subcategoria === "A");
  const tienePermisoAcademia = permisos.some((p) => p.categoria !== "SEN" || (p.categoria === "SEN" && p.subcategoria !== "A"));

     Esto lee del sessionStorage para discernir si tiene permisos del 1er equipo (Sénior) y/o de la Academia.
     También hay que tener en cuenta que los sénior solo tienen subcategoría A. Es indiferente sobre el sexo (M o F)
