# 🌀 Visualizador del Conjunto de Julia

Prototipo de **programación creativa** (< 1024 caracteres) que genera visualizaciones interactivas de [fractales del conjunto de Julia](https://en.wikipedia.org/wiki/Julia_set) con coloración aleatoria en tiempo real. Desarrollado en [Processing](https://processing.org/) para la asignatura CIU — *Creando Interfaces de Usuario*.

> **Autor:** Leopoldo López Reverón  
> Escuela de Ingeniería Informática — Universidad de Las Palmas de Gran Canaria

---

## Resultado

![Animación del conjunto de Julia](./animation.gif)

Cada fotograma genera un fractal distinto: parámetros `c`, radio `R`, grado polinomial y paleta de color se eligen aleatoriamente en cada llamada a `draw()`.

---

## ¿Qué es el Conjunto de Julia?

El [conjunto de Julia](https://en.wikipedia.org/wiki/Julia_set) es una figura [fractal](https://en.wikipedia.org/wiki/Fractal) definida en el [plano complejo](https://en.wikipedia.org/wiki/Complex_plane). Para una función `f(z)` y una constante compleja `c`, el conjunto de Julia `J(f)` es la frontera entre los puntos cuya órbita bajo iteración repetida de `f` permanece acotada y los que divergen al infinito.

La forma más conocida usa `f(z) = z² + c`, pero este visualizador generaliza a potencias arbitrarias mediante la forma polar:

```
z^n = r^n · (cos(n·θ) + i·sin(n·θ))     [Fórmula de De Moivre]
```

donde `r = |z|` y `θ = atan2(Im(z), Re(z))`.

Esto produce una familia infinita de fractales con simetrías de orden `n`, controlada en el código por `grade`.

---

## Cómo funciona el código

### Algoritmo de escape (*escape time algorithm*)

El [algoritmo de tiempo de escape](https://en.wikipedia.org/wiki/Plotting_algorithms_for_the_Mandelbrot_set#Escape_time_algorithm) es el método estándar para visualizar fractales de plano complejo:

1. Cada píxel `(x, y)` se mapea a un número complejo `z = zx + i·zy` dentro de la ventana `[-R/2, R/2]²`.
2. Se itera `z ← z^grade + c` hasta que `|z| > K` (el punto "escapa") o se alcanza `MAXITER` iteraciones.
3. El número de iteraciones `n` hasta el escape determina el color del píxel.

```processing
while(zx*zx + zy*zy < K*K && n < MAXITER) {
    float r_n  = pow(zx*zx + zy*zy, grade / 2.0);
    float theta = atan2(zy, zx);
    float aux = r_n * cos(grade * theta) + cx;   // Re(z^n + c)
    zy        = r_n * sin(grade * theta) + cy;   // Im(z^n + c)
    zx = aux;
    n++;
}
```

Los puntos que nunca escapan (pertenecen al conjunto) se pintan de negro (`stroke(0)`). El resto recibe un color proporcional a `n`.

### Espacio de color y mapeo

En lugar de HSB clásico, el color se genera multiplicando `n` por tres factores aleatorios independientes `fn1, fn2, fn3` sobre los canales RGB, con módulo 255:

```processing
stroke((n * fn1) % 255, (n * fn2) % 255, (n * fn3) % 255);
```

Esto produce paletas distintas en cada fotograma sin necesidad de un selector de color explícito.

### Optimizaciones de rendimiento

| Decisión | Efecto |
|----------|--------|
| Malla de píxeles 2×2 (`x=x+2`, `y=y+2`) | Divide el tiempo de render a la mitad |
| `MAXITER = 40` | Equilibrio entre detalle y latencia |
| Grado polinomial máximo = 9 | Evita iteraciones excesivamente costosas |
| `noSmooth()` | Elimina antialiasing innecesario en píxeles discretos |

### Parámetros aleatorios por fotograma

| Variable | Rango | Efecto visual |
|----------|-------|---------------|
| `cx, cy` | `[-1, 1]` | Constante compleja `c` → cambia la forma del fractal |
| `R` | `[1, 9]` | Zoom / escala del plano complejo visualizado |
| `grade` | `[0, 9]` | Orden del polinómio → simetría rotacional del fractal |
| `fn1/fn2/fn3` | `[0, 255]` | Paleta de color del fotograma |

---

## Estructura del proyecto

```
visualizacion_julia_set/
├── CIU_Julia_Set.pde   # Sketch de Processing (< 1024 caracteres)
├── animation.gif       # Captura de varios fotogramas
└── README.md
```

---

## Ejecución

1. Instalar [Processing 3+](https://processing.org/download).
2. Abrir `CIU_Julia_Set.pde`.
3. Pulsar ▶ — cada fotograma genera un fractal distinto automáticamente.

No se requieren librerías externas: el sketch usa únicamente la API estándar de Processing y `java.lang.Math`.

---

## Conceptos clave — Referencias

| Concepto | Wikipedia |
|----------|-----------|
| Conjunto de Julia | [Julia set](https://en.wikipedia.org/wiki/Julia_set) |
| Fractal | [Fractal — Wikipedia](https://en.wikipedia.org/wiki/Fractal) |
| Plano complejo | [Complex plane](https://en.wikipedia.org/wiki/Complex_plane) |
| Algoritmo de tiempo de escape | [Escape time algorithm](https://en.wikipedia.org/wiki/Plotting_algorithms_for_the_Mandelbrot_set#Escape_time_algorithm) |
| Fórmula de De Moivre | [De Moivre's formula](https://en.wikipedia.org/wiki/De_Moivre%27s_formula) |
| Conjunto de Mandelbrot (contexto) | [Mandelbrot set](https://en.wikipedia.org/wiki/Mandelbrot_set) |
| Modelo de color HSB/HSV | [HSL and HSV](https://en.wikipedia.org/wiki/HSL_and_HSV) |