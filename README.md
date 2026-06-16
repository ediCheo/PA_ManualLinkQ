# PA_ManualLinkQ
Manual de Todo lo visto en PA
# Manual de Usuario — LINQ en Programación Avanzada
### Sistema de Gestión de Pacientes — Arquitectura de 4 Capas con LINQ to SQL

> **Asignatura:** Programación Avanzada  
> **Base de datos:** SQL Server (ProgramacionAvanzada)  
> **Tecnología principal:** LINQ to SQL (.NET Framework 4.8 — C#)  
> **IDE:** Visual Studio 2022+

---

## Tabla de Contenidos

1. [¿Qué es LINQ?](#1-qué-es-linq)
2. [Arquitectura de 4 Capas](#2-arquitectura-de-4-capas)
3. [Estructura del Proyecto](#3-estructura-del-proyecto)
4. [Base de Datos](#4-base-de-datos)
5. [Capa de Entidades](#5-capa-de-entidades)
6. [Capa de Datos con LINQ](#6-capa-de-datos-con-linq)
7. [Capa de Lógica de Negocio](#7-capa-de-lógica-de-negocio)
8. [Capa de Presentación](#8-capa-de-presentación)
9. [LINQ vs ADO.NET — Comparativa](#9-linq-vs-adonet--comparativa)
10. [Configuración y Requisitos](#10-configuración-y-requisitos)
11. [Operaciones CRUD con LINQ](#11-operaciones-crud-con-linq)
12. [Errores Comunes y Soluciones](#12-errores-comunes-y-soluciones)

---

## 1. ¿Qué es LINQ?

**LINQ** (Language Integrated Query) es una tecnología de Microsoft que permite escribir consultas directamente en C# para interactuar con fuentes de datos como bases de datos SQL, colecciones en memoria, XML y más.

**LINQ to SQL** es la variante usada en este proyecto. Genera automáticamente clases C# a partir de las tablas de SQL Server mediante un archivo `.dbml`, eliminando la necesidad de escribir SQL manualmente.

### Ventajas de LINQ sobre ADO.NET tradicional

| Característica | ADO.NET | LINQ to SQL |
|---|---|---|
| Escritura de SQL | Manual, en strings | Automático / integrado en C# |
| Verificación de errores | En tiempo de ejecución | En tiempo de compilación |
| Código necesario | Mucho (conexión, comando, reader) | Mínimo |
| Legibilidad | Baja | Alta |
| Mantenimiento | Complejo | Simple |

---

## 2. Arquitectura de 4 Capas

Este proyecto implementa una **arquitectura de 4 capas** que separa responsabilidades y facilita el mantenimiento:

```
┌─────────────────────────────────────────┐
│        CAPA DE PRESENTACIÓN             │
│    GestionUsuariosPresentacion          │
│   (Windows Forms — Form_Paciente.cs)    │
└──────────────────┬──────────────────────┘
                   │ usa
┌──────────────────▼──────────────────────┐
│       CAPA DE LÓGICA DE NEGOCIO         │
│    GestionUsuariosLogicaNegocio         │
│  (PacienteNegocio.cs / GeneroNegocio.cs)│
└──────────────────┬──────────────────────┘
                   │ usa
┌──────────────────▼──────────────────────┐
│          CAPA DE DATOS (LINQ)           │
│      GestionUsuariosDatosLinQ           │
│  (PacienteDatos.cs / GeneroDatos.cs)    │
│  (ModelProgramacionAvanzada.dbml)       │
└──────────────────┬──────────────────────┘
                   │ comparten
┌──────────────────▼──────────────────────┐
│         CAPA DE ENTIDADES               │
│      GestionUsuariosEntidades           │
│ (PacienteEntidades.cs / GeneroEntidades)│
└─────────────────────────────────────────┘
                   │
                   ▼
              SQL SERVER
         (BD: ProgramacionAvanzada)
```

> **Principio clave:** Cada capa solo conoce a la capa inmediatamente inferior. La Presentación nunca accede directamente a la base de datos.

---

## 3. Estructura del Proyecto

```
GestionUsuariosv2/
└── GestionUsuarios/
    ├── GestionUsuarios.slnx                   ← Solución principal
    │
    ├── GestionUsuariosEntidades/              ← Capa 1: Entidades
    │   ├── PacienteEntidades.cs
    │   └── GeneroEntidades.cs
    │
    ├── GestionUsuariosDatosLinQ/              ← Capa 2: Datos con LINQ 
    │   ├── ModelProgramacionAvanzada.dbml     ← Modelo LINQ to SQL
    │   ├── ModelProgramacionAvanzada.designer.cs
    │   ├── PacienteDatos.cs
    │   └── GeneroDatos.cs
    │
    ├── GestionUsuariosDatos/                  ← Capa 2 alternativa: ADO.NET
    │   ├── PacienteDatos.cs
    │   └── GeneroDatos.cs
    │
    ├── GestionUsuariosLogicaNegocio/          ← Capa 3: Negocio
    │   ├── PacienteNegocio.cs
    │   └── GeneroNegocio.cs
    │
    └── GestionUsuariosPresentacion/           ← Capa 4: Presentación
        ├── Form_Paciente.cs
        ├── Form_Paciente.Designer.cs
        └── Program.cs
```

---

## 4. Base de Datos

### Nombre de la base de datos
```
ProgramacionAvanzada
```

### Tablas

#### Tabla `Paciente`
| Campo | Tipo SQL | Tipo C# | Descripción |
|---|---|---|---|
| id | INT IDENTITY (PK) | int | Identificador único |
| id_Genero | INT (FK) | int? | Referencia a Genero |
| nombre | NVARCHAR(50) | string | Nombre del paciente |
| apellido | NVARCHAR(50) | string | Apellido |
| cedula | NCHAR(10) | string | Número de cédula |
| fechaNacimient | DATE | DateTime | Fecha de nacimiento |
| telefono | NVARCHAR(10) | string | Teléfono |
| direccion | NCHAR(50) | string | Dirección |
| afiliado | BIT | bool? | Si está afiliado al IEES |
| codigoIEES | NVARCHAR(20) | string | Código del IEES |

#### Tabla `Genero`
| Campo | Tipo SQL | Tipo C# | Descripción |
|---|---|---|---|
| id | INT IDENTITY (PK) | int | Identificador único |
| nombre | NVARCHAR(50) | string | Nombre del género |

### Script SQL para crear las tablas

```sql
-- Crear base de datos
CREATE DATABASE ProgramacionAvanzada;
GO

USE ProgramacionAvanzada;
GO

-- Tabla Genero
CREATE TABLE Genero (
    id     INT IDENTITY(1,1) PRIMARY KEY,
    nombre NVARCHAR(50)
);

-- Tabla Paciente
CREATE TABLE Paciente (
    id             INT IDENTITY(1,1) PRIMARY KEY,
    id_Genero      INT FOREIGN KEY REFERENCES Genero(id),
    nombre         NVARCHAR(50) NOT NULL,
    apellido       NVARCHAR(50) NOT NULL,
    cedula         NCHAR(10)    NOT NULL,
    fechaNacimient DATE         NOT NULL,
    telefono       NVARCHAR(10) NOT NULL,
    direccion      NCHAR(50)    NOT NULL,
    afiliado       BIT,
    codigoIEES     NVARCHAR(20)
);

-- Datos de ejemplo para Genero
INSERT INTO Genero (nombre) VALUES ('Masculino'), ('Femenino'), ('Otro');
```

---

## 5. Capa de Entidades

Las entidades son **clases POCO** (Plain Old C# Object) que representan los datos sin lógica adicional. Son compartidas por todas las capas del sistema.

### `PacienteEntidades.cs`

```csharp
namespace GestionUsuariosEntidades
{
    public class PacienteEntidades
    {
        public int      Id              { get; set; }
        public int      IdGenero        { get; set; }
        public string   Genero          { get; set; }
        public string   Nombre          { get; set; }
        public string   Apellido        { get; set; }
        public string   Cedula          { get; set; }
        public DateTime FechaNacimiento { get; set; }
        public string   Telefono        { get; set; }
        public string   Direccion       { get; set; }
        public bool     Afiliado        { get; set; }
        public string   CodigoIEES      { get; set; }

        // Constructor con parámetros
        public PacienteEntidades(int id, int idGenero, string genero, string nombre,
            string apellido, string cedula, DateTime fechaNacimiento,
            string telefono, string direccion, bool afiliado, string codigoIEES)
        {
            Id = id; IdGenero = idGenero; Genero = genero;
            Nombre = nombre; Apellido = apellido; Cedula = cedula;
            FechaNacimiento = fechaNacimiento; Telefono = telefono;
            Direccion = direccion; Afiliado = afiliado; CodigoIEES = codigoIEES;
        }

        // Constructor vacío (necesario para crear objetos sin datos iniciales)
        public PacienteEntidades() { }
    }
}
```

> **¿Por qué dos constructores?** El constructor vacío se usa cuando se crea un nuevo paciente desde el formulario (`new PacienteEntidades()`). El constructor con parámetros se usa al recuperar datos de la base de datos.

### `GeneroEntidades.cs`

```csharp
namespace GestionUsuariosEntidades
{
    public class GeneroEntidades
    {
        public int    Id     { get; set; }
        public string Nombre { get; set; }

        public GeneroEntidades() { }

        public GeneroEntidades(int id, string nombre)
        {
            Id = id;
            Nombre = nombre;
        }
    }
}
```

---

## 6. Capa de Datos con LINQ

Esta es la capa más importante del proyecto desde el punto de vista de LINQ to SQL.

### 6.1 El Modelo LINQ to SQL (`.dbml`)

El archivo `ModelProgramacionAvanzada.dbml` es el corazón de LINQ to SQL. Fue creado en Visual Studio mediante **Diseñador de LINQ to SQL** y genera automáticamente el `DataContext` y las clases de tabla.

**Cadena de conexión configurada:**
```
Data Source=.\sqlexpress;
Initial Catalog=ProgramacionAvanzada;
Integrated Security=True;
Trust Server Certificate=True
```

**¿Qué genera el `.dbml`?**
- La clase `ModelProgramacionAvanzadaDataContext` — el puente con la base de datos
- La clase `Paciente` — mapea la tabla `dbo.Paciente`
- La clase `Genero` — mapea la tabla `dbo.Genero`
- La relación `Genero_Paciente` (FK entre Paciente y Genero)

### 6.2 `PacienteDatos.cs` — Operaciones LINQ

#### Insertar (Nuevo)

```csharp
public static PacienteEntidades Nuevo(PacienteEntidades paciente)
{
    try
    {
        // 1. Crear objeto LINQ que mapea la tabla
        Paciente pacienteLinQ = new Paciente();
        pacienteLinQ.id_Genero     = (int)paciente.IdGenero;
        pacienteLinQ.nombre        = paciente.Nombre;
        pacienteLinQ.apellido      = paciente.Apellido;
        pacienteLinQ.cedula        = paciente.Cedula;
        pacienteLinQ.fechaNacimient = paciente.FechaNacimiento;
        pacienteLinQ.telefono      = paciente.Telefono;
        pacienteLinQ.direccion     = paciente.Direccion;
        pacienteLinQ.afiliado      = (bool)paciente.Afiliado;
        pacienteLinQ.codigoIEES    = paciente.CodigoIEES;

        // 2. Usar el DataContext para insertar
        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            contexto.Paciente.InsertOnSubmit(pacienteLinQ); // Marca para insertar
            contexto.SubmitChanges();                        // Ejecuta el INSERT
        }

        // 3. Devolver el paciente con el Id generado por la BD
        paciente.Id = pacienteLinQ.id;
        return paciente;
    }
    catch (Exception) { throw; }
}
```

> **Conceptos clave:**
> - `InsertOnSubmit()` — marca el objeto para ser insertado
> - `SubmitChanges()` — ejecuta todos los cambios pendientes en la BD
> - `using (...)` — cierra automáticamente la conexión al terminar el bloque
> - El `id` se actualiza automáticamente con el valor IDENTITY generado por SQL Server

#### Actualizar

```csharp
public static PacienteEntidades Actualizar(PacienteEntidades pacientes)
{
    try
    {
        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            // 1. Buscar el registro en la BD con FirstOrDefault y expresión lambda
            Paciente pacienteLinQ = contexto.Paciente
                                    .FirstOrDefault(p => p.id == pacientes.Id);

            // 2. Modificar las propiedades del objeto recuperado
            pacienteLinQ.id_Genero      = pacientes.IdGenero;
            pacienteLinQ.nombre         = pacientes.Nombre;
            pacienteLinQ.apellido       = pacientes.Apellido;
            pacienteLinQ.cedula         = pacientes.Cedula;
            pacienteLinQ.fechaNacimient = pacientes.FechaNacimiento;
            pacienteLinQ.telefono       = pacientes.Telefono;
            pacienteLinQ.direccion      = pacientes.Direccion;
            pacienteLinQ.afiliado       = pacientes.Afiliado;
            pacienteLinQ.codigoIEES     = pacientes.CodigoIEES;

            // 3. Confirmar los cambios (genera UPDATE automáticamente)
            contexto.SubmitChanges();
            return pacientes;
        }
    }
    catch (Exception) { throw; }
}
```

>  **Concepto clave:** LINQ to SQL rastrea los cambios en los objetos. Al modificar las propiedades del objeto recuperado del contexto y llamar `SubmitChanges()`, genera el `UPDATE` automáticamente.

####  Listar Todos los Pacientes

```csharp
public static List<PacienteEntidades> DevolverListarPacientes()
{
    try
    {
        List<PacienteEntidades> listaPacientesEntidades = new List<PacienteEntidades>();
        List<Paciente> listaPacientesLinQ = new List<Paciente>();

        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            // Consulta LINQ — sintaxis de consulta (similar a SQL)
            var resultado = from p in contexto.Paciente
                            select p;

            listaPacientesLinQ = resultado.ToList();
        }

        // Convertir objetos LINQ a objetos Entidades
        foreach (var item in listaPacientesLinQ)
        {
            listaPacientesEntidades.Add(new PacienteEntidades(
                item.id,
                (int)item.id_Genero,
                GeneroDatos.DevolverNombreGeneroPorId((int)item.id_Genero), // JOIN manual
                item.nombre,
                item.apellido,
                item.cedula,
                item.fechaNacimient,
                item.telefono,
                item.direccion,
                (bool)item.afiliado,
                item.codigoIEES
            ));
        }
        return listaPacientesEntidades;
    }
    catch (Exception) { throw; }
}
```

>  **Sintaxis LINQ:** `from p in contexto.Paciente select p` es la forma de escribir `SELECT * FROM Paciente` en LINQ. Se puede extender con `where`, `orderby`, `join`, etc.

####  Buscar por ID

```csharp
public static PacienteEntidades CargarPacientePorId(int id)
{
    try
    {
        PacienteEntidades pacienteEntidades = new PacienteEntidades();

        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            // FirstOrDefault con expresión lambda — devuelve null si no encuentra
            Paciente pacienteLinq = contexto.Paciente
                                    .FirstOrDefault(p => p.id == id);

            // Mapear propiedades al objeto Entidades
            pacienteEntidades.Id             = pacienteLinq.id;
            pacienteEntidades.IdGenero       = (int)pacienteLinq.id_Genero;
            pacienteEntidades.Genero         = GeneroDatos
                                              .DevolverNombreGeneroPorId((int)pacienteLinq.id_Genero);
            pacienteEntidades.Nombre         = pacienteLinq.nombre;
            pacienteEntidades.Apellido       = pacienteLinq.apellido;
            pacienteEntidades.Cedula         = pacienteLinq.cedula;
            pacienteEntidades.FechaNacimiento = pacienteLinq.fechaNacimient;
            pacienteEntidades.Telefono       = pacienteLinq.telefono;
            pacienteEntidades.Direccion      = pacienteLinq.direccion;
            pacienteEntidades.Afiliado       = (bool)pacienteLinq.afiliado;
            pacienteEntidades.CodigoIEES     = pacienteLinq.codigoIEES;

            return pacienteEntidades;
        }
    }
    catch (Exception) { throw; }
}
```

####  Eliminar por ID

```csharp
public static bool EliminarPacientePorId(int id)
{
    try
    {
        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            // Buscar el objeto a eliminar
            Paciente pacienteLinq = contexto.Paciente
                                    .FirstOrDefault(p => p.id == id);

            // Marcar para eliminar y confirmar
            contexto.Paciente.DeleteOnSubmit(pacienteLinq);
            contexto.SubmitChanges(); // Genera DELETE automáticamente
            return true;
        }
    }
    catch (Exception)
    {
        return false;
    }
}
```

### 6.3 `GeneroDatos.cs` — Consultas LINQ de Género

#### Listar todos los géneros

```csharp
public static List<GeneroEntidades> DevolverListaGeneros()
{
    try
    {
        List<GeneroEntidades> listaGeneroEntidades = new List<GeneroEntidades>();
        List<Genero> listaGenero = new List<Genero>();

        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            var resultado = from g in contexto.Genero
                            select g;

            listaGenero = resultado.ToList();

            foreach (var item in listaGenero)
            {
                listaGeneroEntidades.Add(new GeneroEntidades(item.id, item.nombre));
            }
            return listaGeneroEntidades;
        }
    }
    catch (Exception) { throw; }
}
```

#### Obtener nombre de género por ID

```csharp
public static string DevolverNombreGeneroPorId(int idGenero)
{
    try
    {
        using (ModelProgramacionAvanzadaDataContext contexto =
               new ModelProgramacionAvanzadaDataContext())
        {
            // Sintaxis de método (alternativa a sintaxis de consulta)
            var resultado = contexto.Genero.FirstOrDefault(g => g.id == idGenero);
            return resultado.nombre;
        }
    }
    catch (Exception) { throw; }
}
```

---

## 7. Capa de Lógica de Negocio

Esta capa actúa como intermediaria entre la Presentación y los Datos. Contiene la lógica de decisión del sistema.

### `PacienteNegocio.cs`

```csharp
using GestionUsuariosDatosLinQ; // Referencia a la capa de Datos LinQ
using GestionUsuariosEntidades;

namespace GestionUsuariosLogicaNegocio
{
    public static class PacienteNegocio
    {
        // Lógica: si Id == 0 es nuevo, si Id != 0 actualiza
        public static PacienteEntidades GuardarPaciente(PacienteEntidades paciente)
        {
            if (paciente.Id == 0)
                return PacienteDatos.Nuevo(paciente);     // INSERT
            else
                return PacienteDatos.Actualizar(paciente); // UPDATE
        }

        public static List<PacienteEntidades> DevolverListaPaciente()
        {
            return PacienteDatos.DevolverListarPacientes();
        }

        public static PacienteEntidades CargarPacientePorId(int id)
        {
            return PacienteDatos.CargarPacientePorId(id);
        }

        public static bool EliminarPacientePorId(int id)
        {
            return PacienteDatos.EliminarPacientePorId(id);
        }
    }
}
```

>  **Decisión inteligente:** El método `GuardarPaciente` determina si debe insertar o actualizar según el valor del `Id`. Si `Id == 0` es un nuevo registro; si `Id != 0` ya existe en la base de datos.

### `GeneroNegocio.cs`

```csharp
public static class GeneroNegocio
{
    public static List<GeneroEntidades> DevolverListaGeneros()
    {
        return GeneroDatos.DevolverListaGeneros();
    }
}
```

---

## 8. Capa de Presentación

La capa de presentación implementa el formulario Windows Forms que el usuario ve y con el que interactúa. Solo se comunica con la Lógica de Negocio, nunca con la capa de Datos directamente.

### Funcionalidades del `Form_Paciente`

#### Carga inicial del formulario

```csharp
private void Form_Paciente_Load(object sender, EventArgs e)
{
    CargarListadoPacientesEnDataGridView(); // Llenar la grilla
    CargarComboGenero();                   // Llenar el ComboBox de géneros
}
```

#### Cargar ComboBox de géneros (usando LINQ indirectamente)

```csharp
private void CargarComboGenero()
{
    cb_Genero.DataSource    = GeneroNegocio.DevolverListaGeneros();
    cb_Genero.DisplayMember = "nombre"; // Qué mostrar al usuario
    cb_Genero.ValueMember   = "id";     // Valor interno que se guarda
}
```

#### Guardar paciente (Nuevo o Actualizar)

```csharp
private void GuardarPaciente()
{
    paciente.Nombre         = txtb_Nombres.Text.ToUpper();
    paciente.Apellido       = txtb_Apellidos.Text.ToUpper();
    paciente.Cedula         = txtb_Cedula.Text;
    paciente.FechaNacimiento = dtp_FechaNacimiento.Value;
    paciente.Telefono       = txtb_Telefono.Text;
    paciente.Direccion      = txtb_Direccion.Text.ToUpper();
    paciente.Afiliado       = cb_Afiliado.Checked;
    paciente.CodigoIEES     = txtb_CodigoIees.Text;
    paciente.IdGenero       = Convert.ToInt32(cb_Genero.SelectedValue);

    // Llama a la capa de Negocio
    paciente = PacienteNegocio.GuardarPaciente(paciente);

    if (paciente != null)
    {
        txtb_Id.Text = paciente.Id.ToString();
        MessageBox.Show("Los datos se almacenaron correctamente");
        CargarListadoPacientesEnDataGridView();
    }
}
```

#### Seleccionar fila del DataGridView

```csharp
private void dataGridView1_CellClick(object sender, DataGridViewCellEventArgs e)
{
    try
    {
        var id = Convert.ToInt32(
            dataGridView1.Rows[e.RowIndex].Cells["id"].Value.ToString());
        CargarValoresPacientePorId(id);
    }
    catch (Exception ex)
    {
        MessageBox.Show("Error en seleccion de elemento", ex.Message);
    }
}
```

#### Eliminar con confirmación

```csharp
private void EliminarPaciente()
{
    if (MessageBox.Show("¿Está seguro de eliminar permanentemente este registro?",
                        "Eliminar registro",
                        MessageBoxButtons.OKCancel,
                        MessageBoxIcon.Question) == DialogResult.OK)
    {
        if (PacienteNegocio.EliminarPacientePorId(paciente.Id))
        {
            MessageBox.Show("El registro se eliminó correctamente",
                            "Eliminar paciente",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Information);
            CargarListadoPacientesEnDataGridView();
        }
    }
}
```

---

## 9. LINQ vs ADO.NET — Comparativa

El proyecto implementa **las dos versiones** de la capa de datos para comparar ambas tecnologías:

### Insertar un registro — INSERT

**ADO.NET (más verboso):**
```csharp
SqlConnection conexion = new SqlConnection(connectionString);
conexion.Open();
SqlCommand cmd = conexion.CreateCommand();
cmd.CommandType = CommandType.Text;
cmd.CommandText = @"INSERT INTO Paciente (id_Genero, nombre, apellido, cedula,
                    fechaNacimient, telefono, direccion, afiliado, codigoIEES)
                    VALUES (@id_Genero, @nombre, @apellido, @cedula,
                    @fechaNacimient, @telefono, @direccion, @afiliado, @codigoIEES);
                    SELECT SCOPE_IDENTITY()";
cmd.Parameters.AddWithValue("@id_Genero", paciente.IdGenero);
cmd.Parameters.AddWithValue("@nombre", paciente.Nombre);
// ... más parámetros ...
int id = Convert.ToInt32(cmd.ExecuteScalar());
conexion.Close();
```

**LINQ to SQL (simple y limpio):**
```csharp
Paciente pacienteLinQ = new Paciente();
pacienteLinQ.id_Genero = (int)paciente.IdGenero;
pacienteLinQ.nombre    = paciente.Nombre;
// ... más propiedades ...

using (var contexto = new ModelProgramacionAvanzadaDataContext())
{
    contexto.Paciente.InsertOnSubmit(pacienteLinQ);
    contexto.SubmitChanges();
}
```

### Consultar un registro — SELECT WHERE

**ADO.NET:**
```csharp
cmd.CommandText = "SELECT * FROM Paciente WHERE id = @id";
cmd.Parameters.AddWithValue("@id", id);
using (var dr = cmd.ExecuteReader())
{
    while (dr.Read())
    {
        paciente.Nombre = dr["nombre"].ToString();
        // ... más campos ...
    }
}
```

**LINQ to SQL:**
```csharp
using (var contexto = new ModelProgramacionAvanzadaDataContext())
{
    Paciente p = contexto.Paciente.FirstOrDefault(x => x.id == id);
    paciente.Nombre = p.nombre;
}
```

---

## 10. Configuración y Requisitos

### Requisitos del sistema

- Visual Studio 2019 o superior
- .NET Framework 4.8
- SQL Server Express (instancia `.\sqlexpress`) o SQL Server
- Windows 10/11

### Pasos para configurar el proyecto

**1. Clonar o descargar el repositorio**
```bash
git clone https://github.com/tu-usuario/GestionUsuariosv2.git
```

**2. Crear la base de datos**

Ejecutar el script SQL de la [sección 4](#4-base-de-datos) en SQL Server Management Studio.

**3. Verificar la cadena de conexión**

En `GestionUsuariosDatosLinQ/Properties/Settings.settings`, revisar:
```
Data Source=.\sqlexpress;
Initial Catalog=ProgramacionAvanzada;
Integrated Security=True;
Trust Server Certificate=True
```

Cambiar `.\sqlexpress` por el nombre de tu instancia de SQL Server si es diferente.

**4. Compilar y ejecutar**

Abrir `GestionUsuarios.slnx` en Visual Studio → Establecer `GestionUsuariosPresentacion` como proyecto de inicio → `F5` para ejecutar.

---

## 11. Operaciones CRUD con LINQ

Resumen de los métodos LINQ implementados:

| Operación | SQL equivalente | Método LINQ | Clase |
|---|---|---|---|
| Crear | `INSERT INTO` | `InsertOnSubmit()` + `SubmitChanges()` | `PacienteDatos.Nuevo()` |
| Leer todos | `SELECT *` | `from p in contexto.Paciente select p` | `PacienteDatos.DevolverListarPacientes()` |
| Leer uno | `SELECT WHERE` | `FirstOrDefault(p => p.id == id)` | `PacienteDatos.CargarPacientePorId()` |
| Actualizar | `UPDATE SET WHERE` | Modificar objeto + `SubmitChanges()` | `PacienteDatos.Actualizar()` |
| Eliminar | `DELETE WHERE` | `DeleteOnSubmit()` + `SubmitChanges()` | `PacienteDatos.EliminarPacientePorId()` |

### Sintaxis LINQ — Referencia Rápida

```csharp
// Sintaxis de consulta (Query Syntax) — similar a SQL
var resultado = from p in contexto.Paciente
                where p.nombre == "JUAN"
                orderby p.apellido
                select p;

// Sintaxis de método (Method Syntax) — encadenamiento de métodos
var resultado = contexto.Paciente
                .Where(p => p.nombre == "JUAN")
                .OrderBy(p => p.apellido)
                .ToList();

// Ambas producen el mismo resultado — elige la que prefieras
```

---

## 12. Errores Comunes y Soluciones

###  Error de conexión a la base de datos
```
A network-related or instance-specific error occurred while establishing a connection to SQL Server
```
**Solución:** Verificar que SQL Server Express esté corriendo. Revisar la cadena de conexión en `Settings.settings`.

###  `NullReferenceException` en `FirstOrDefault`
```
Object reference not set to an instance of an object
```
**Solución:** `FirstOrDefault` devuelve `null` si no encuentra el registro. Siempre verificar antes de usar el resultado:
```csharp
var paciente = contexto.Paciente.FirstOrDefault(p => p.id == id);
if (paciente != null)
{
    // usar paciente
}
```

###  Error al regenerar el modelo `.dbml`
Si se modifican las tablas en SQL Server, hay que actualizar el modelo:
1. Abrir `ModelProgramacionAvanzada.dbml` en el diseñador
2. Borrar las tablas del diseñador
3. Arrastrar nuevamente las tablas desde el **Explorador de servidores**

###  El `DataGridView` no muestra datos
**Solución:** Verificar que `DataSource` esté asignado correctamente y que la consulta LINQ devuelva datos (no una lista vacía).

---

##  Resumen Conceptual

```
LINQ to SQL = Clases C# que mapean tablas SQL
DataContext  = La conexión inteligente que traduce C# a SQL
.dbml        = El archivo que genera automáticamente las clases
SubmitChanges() = Ejecuta todos los INSERT/UPDATE/DELETE pendientes
FirstOrDefault() = SELECT TOP 1 WHERE ... (devuelve null si no hay resultado)
InsertOnSubmit() = Marca un objeto para INSERT
DeleteOnSubmit() = Marca un objeto para DELETE
```

---

*Manual elaborado para la materia de Programación Avanzada — Sistema de Gestión de Pacientes con LINQ to SQL y Arquitectura de 4 Capas.*
