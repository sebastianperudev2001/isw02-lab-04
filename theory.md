# isw02-lab-04

Lab 04: Patrones de comportamiento

Elaborado por: Sebastián Chávarry

Temas a tratar:

- Strategy

# Command

- Propósito: Convertir una solicitud (payload) en un objeto independiente para facilitar la ejecución de métodos.
  > En sencillo: Command es la capa intermedia entre tu UI y tu lógica de negocio

## ¿Cómo lo pueden implementar en su proyecto?

1. Job Queue: Para procesar tareas en segundo plano (enviar mails, generar PDFs, hacer cálculos pesados)
2. Auditoría y trazabilidad: Registrar qué acciones se ejecutaron en el sistema, cuándo y cómo.
3. Reintentos: Reintentar tareas que fallaron como enviar SMS, confirmar pagos, llamar a APIs externas.
4. Testing: Poder testear acciones de negocio desacopladas de sus invocadores.
5. Implementar UNDO/REDO: Interfaces que permiten revertir acciones, como en editores o apps de diseño.
6. Orquestar workflows/pipelines: Ejecutar una secuencia ordenada de pasos (por ejemplo, un flujo de onboard de cliente).
7. Control de permisos por tipo de acción: Aplicaciones con reglas de negocio complejas.
8. Intercambio de acciones entre microservicios: Comunicar servicios desacoplados mediante eventos o comandos serializados.

# ¿Cómo implementarlo?

## 🛒 Caso de uso (texto):

> En un ecommerce, los clientes pueden realizar acciones como **agregar productos al carrito**, **remover productos** o **vaciar el carrito**. Cada acción debe encapsularse como un comando para que pueda registrarse, deshacerse o incluso ejecutarse de manera asincrónica en el futuro (por ejemplo, en un flujo de checkout).

---

### ✅ Paso 1: Interfaz de comando con un único método de ejecución

```java
public interface Comando {
    // Solo tiene un método
    void ejecutar();
}
```

---

### ✅ Paso 2: Receptor y comandos concretos

Empieza extrayendo solicitudes y poniéndolas dentro de clases concretas de comando que implementen la interfaz de comando. Cada clase debe contar con un grupo de campos para almacenar los argumentos de las solicitudes junto con referencias al objeto receptor. Todos estos valores deben inicializarse a través del constructor del comando.

```java
// Receptor
public class Carrito {
    public void agregarProducto(String producto) {
        System.out.println("Producto agregado: " + producto);
    }

    public void removerProducto(String producto) {
        System.out.println(" Producto removido: " + producto);
    }

    public void vaciar() {
        System.out.println("Carrito vaciado.");
    }
}

// Comando concreto para agregar
public class ComandoAgregarProducto implements Comando {
    private Carrito carrito;
    private String producto;

    public ComandoAgregarProducto(Carrito carrito, String producto) {
        this.carrito = carrito;
        this.producto = producto;
    }

    @Override
    public void ejecutar() {
        carrito.agregarProducto(producto);
    }
}

// Comando concreto para remover
public class ComandoRemoverProducto implements Comando {
    private Carrito carrito;
    private String producto;

    public ComandoRemoverProducto(Carrito carrito, String producto) {
        this.carrito = carrito;
        this.producto = producto;
    }

    @Override
    public void ejecutar() {
        carrito.removerProducto(producto);
    }
}

// Comando concreto para vaciar
public class ComandoVaciarCarrito implements Comando {
    private Carrito carrito;

    public ComandoVaciarCarrito(Carrito carrito) {
        this.carrito = carrito;
    }

    @Override
    public void ejecutar() {
        carrito.vaciar();
    }
}
```

---

### ✅ Paso 3: Emisora (el sistema que dispara el comando)

Identifica clases que actúen como emisoras. Añade los campos para almacenar comandos dentro de estas clases. Las emisoras deberán comunicarse con sus comandos tan solo a través de la interfaz de comando. Normalmente las emisoras no crean objetos de comando por su cuenta, sino que los obtienen del código cliente.

```java
public class BotonAccionCarrito {
    private Comando comando;

    public BotonAccionCarrito(Comando comando) {
        this.comando = comando;
    }

    public void presionar() {
        comando.ejecutar();
    }
}
```

---

### ✅ Paso 4: Cliente (inicializa y ejecuta)

Cambia las emisoras de forma que ejecuten el comando en lugar de enviar directamente una solicitud al receptor.

```java
public class Main {
    public static void main(String[] args) {
        // Receptor
        Carrito carrito = new Carrito();

        // Comandos
        Comando agregarLaptop = new ComandoAgregarProducto(carrito, "Laptop Gamer");
        Comando removerLaptop = new ComandoRemoverProducto(carrito, "Laptop Gamer");
        Comando vaciarCarrito = new ComandoVaciarCarrito(carrito);

        // Emisores
        BotonAccionCarrito botonAgregar = new BotonAccionCarrito(agregarLaptop);
        BotonAccionCarrito botonRemover = new BotonAccionCarrito(removerLaptop);
        BotonAccionCarrito botonVaciar = new BotonAccionCarrito(vaciarCarrito);

        // Ejecutar comandos
        botonAgregar.presionar();
        botonRemover.presionar();
        botonVaciar.presionar();
    }
}
```

---

### ✅ Salida esperada:

```
 Producto agregado: Laptop Gamer
 Producto removido: Laptop Gamer
 Carrito vaciado.
```

# Iterator

- Propósito: Permitirte recorrer una colección (como una lista o conjunto) paso a paso, sin tener que saber cómo está construida internamente.

> En sencillo: Quizás tengas conjuntos de datas en listas, binary trees, etc. Entonces, creamos una clase que permita extraer esa lógica de comportamiento de recorrido y lo colocamos en un objeto **_iterator_**.

## 🧠 Caso de Estudio

> Somos una empresa educativa que ofrece recursos: **libros** y **podcasts**.
> Queremos que el cliente pueda recorrer ambos **sin saber cómo se almacenan internamente**, usando un iterador uniforme.

---

### ✅ 1. **Declara la interfaz iteradora.**

Como mínimo, debe tener un método para extraer el siguiente elemento de una colección. Por conveniencia, puedes añadir un par de métodos distintos, como para extraer el elemento previo, localizar la posición actual o comprobar el final de la iteración.

```java
public interface Iterador<T> {
    boolean tieneSiguiente();
    T siguiente();
}
```

---

### ✅ 2. **Declara la interfaz de colección**

Debe incluir un método para crear un iterador. El tipo de retorno debe ser igual al de la interfaz iteradora. Puedes declarar métodos similares si planeas tener varios grupos distintos de iteradores.

```java
public interface Coleccion<T> {
    Iterador<T> crearIterador();
}
```

---

### ✅ 3. **Implementa clases iteradoras concretas.**

Cada iterador se asocia a una colección específica y la recorre. Un objeto iterador debe estar vinculado a una única instancia de la colección. Normalmente, este vínculo se establece a través del constructor del iterador.

#### 📚 `IteradorBiblioteca` para libros:

```java
public class IteradorBiblioteca implements Iterador<Libro> {
    private Biblioteca biblioteca;
    private int posicion = 0;

    public IteradorBiblioteca(Biblioteca biblioteca) {
        this.biblioteca = biblioteca;
    }

    @Override
    public boolean tieneSiguiente() {
        return posicion < biblioteca.getLibros().size();
    }

    @Override
    public Libro siguiente() {
        return biblioteca.getLibros().get(posicion++);
    }
}
```

#### 🎧 `IteradorPodcast` para episodios:

```java
public class IteradorPodcast implements Iterador<Podcast> {
    private PlataformaPodcasts plataforma;
    private int posicion = 0;

    public IteradorPodcast(PlataformaPodcasts plataforma) {
        this.plataforma = plataforma;
    }

    @Override
    public boolean tieneSiguiente() {
        return posicion < plataforma.getPodcasts().size();
    }

    @Override
    public Podcast siguiente() {
        return plataforma.getPodcasts().get(posicion++);
    }
}
```

---

### ✅ 4. **Implementa la interfaz de colección.**

Las colecciones (Biblioteca y PlataformaPodcasts) implementan `Coleccion<T>` y devuelven su iterador correspondiente.

#### 📚 Clase `Biblioteca`

```java
import java.util.ArrayList;
import java.util.List;

public class Biblioteca implements Coleccion<Libro> {
    private List<Libro> libros = new ArrayList<>();

    public void agregarLibro(Libro libro) {
        libros.add(libro);
    }

    public List<Libro> getLibros() {
        return libros;
    }

    @Override
    public Iterador<Libro> crearIterador() {
        return new IteradorBiblioteca(this);
    }
}
```

#### 🎧 Clase `PlataformaPodcasts`

```java
import java.util.ArrayList;
import java.util.List;

public class PlataformaPodcasts implements Coleccion<Podcast> {
    private List<Podcast> podcasts = new ArrayList<>();

    public void agregarPodcast(Podcast p) {
        podcasts.add(p);
    }

    public List<Podcast> getPodcasts() {
        return podcasts;
    }

    @Override
    public Iterador<Podcast> crearIterador() {
        return new IteradorPodcast(this);
    }
}
```

---

### ✅ 5. **Repasa el código cliente.**

El cliente no se preocupa por cómo está implementada cada colección. Solo pide un iterador.

```java
public class Main {
    public static void main(String[] args) {
        // Crear colecciones
        Biblioteca biblioteca = new Biblioteca();
        biblioteca.agregarLibro(new Libro("Clean Code"));
        biblioteca.agregarLibro(new Libro("Design Patterns"));

        PlataformaPodcasts plataforma = new PlataformaPodcasts();
        plataforma.agregarPodcast(new Podcast("Java Talks"));
        plataforma.agregarPodcast(new Podcast("Arquitectura Moderna"));

        // Recorrer libros
        System.out.println("📚 Libros:");
        Iterador<Libro> itLibros = biblioteca.crearIterador();
        while (itLibros.tieneSiguiente()) {
            System.out.println("- " + itLibros.siguiente().getTitulo());
        }

        // Recorrer podcasts
        System.out.println("\n🎧 Podcasts:");
        Iterador<Podcast> itPods = plataforma.crearIterador();
        while (itPods.tieneSiguiente()) {
            System.out.println("- " + itPods.siguiente().getNombre());
        }
    }
}
```

---

### 📦 Clases de modelo (Libro y Podcast)

```java
public class Libro {
    private String titulo;

    public Libro(String titulo) {
        this.titulo = titulo;
    }

    public String getTitulo() {
        return titulo;
    }
}
```

```java
public class Podcast {
    private String nombre;

    public Podcast(String nombre) {
        this.nombre = nombre;
    }

    public String getNombre() {
        return nombre;
    }
}
```

---

### ✅ Salida esperada

```
📚 Libros:
- Clean Code
- Design Patterns

🎧 Podcasts:
- Java Talks
- Arquitectura Moderna
```
