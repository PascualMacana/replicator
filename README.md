# replicante

Un programa chico en Rust que se copia. El binario lleva su propia fuente y puede escribir un proyecto Cargo hijo que, una vez compilado, hace lo mismo.

No es un modelo de lenguaje y no se copia solo. Le señalás una carpeta; sólo escribe ahí.

```
generación 0  ──spawn──►  generación 1  ──spawn──►  generación 2
linaje     0               linaje     0.1            linaje     0.1.1
```

El hijo es el mismo programa, una generación después. El linaje cuenta hijas, no el número de generación: la primera hija de `0` es `0.1`, la primera hija de esa es `0.1.1`, y una segunda hija de `0` es `0.2`. No se vuelve más inteligente; hereda el genoma y sube el contador.

![Una célula padre que brota una hija](cell.svg)

Míralo en la terminal. El padre se queda; una hija se desprende. Eso es `spawn`, dibujado como una célula.

```bash
cargo run -- dish
```

## Cómo correrlo

Hace falta [Rust](https://rustup.rs/).

```bash
cargo build --release
./target/release/replicante identity
./target/release/replicante spawn ./hijo --build
./hijo/target/debug/replicante identity
./hijo/target/debug/replicante spawn ./nieto --build
./nieto/target/debug/replicante identity
```

`identity` imprime generación, linaje y los archivos embebidos.  
`spawn ./hijo --build` escribe un crate hijo en `./hijo` y lo compila.

## Comandos

```
replicante              ayuda
replicante identity     generación, linaje, archivos del genoma
replicante genome       imprime las fuentes embebidas
replicante dish         anima una célula que brota hijas
                 --gens N  cuántos brotes (default 8)
                 --delay MS  ms por cuadro (default 80)
replicante spawn <dir>  escribe un proyecto Cargo hijo
                 --build   compila a ese hijo
                 --force   pisa un hijo anterior
```

## Cómo funciona

En tiempo de compilación, `include_str!` embebe cada archivo del proyecto dentro del binario:

```rust
const GENOME: &[(&str, &str)] = &[
    ("Cargo.toml", include_str!("../Cargo.toml")),
    ("src/main.rs", include_str!("main.rs")),
    // ...
];
```

`spawn` escribe esos archivos a disco y actualiza dos constantes en `src/main.rs`:

```rust
const GENERATION: u32 = 0;
const LINEAGE: &str = "0";
```

El hijo nace con `GENERATION = 1` y `LINEAGE = "0.1"`. Cuando lo compilás, su propio `include_str!` captura esa fuente nueva. Lo que se copia es el genoma, no el ejecutable.

El hijo sigue necesitando `rustc` / `cargo` para poder correr.

## Seguridad

- Un hijo por corrida. No hay bucles en segundo plano ni red.
- No escribe sobre el directorio home, `/`, `/usr`, `/etc`, ni el directorio en el que estás parado.
- `--force` sólo borra una carpeta que ya parece un proyecto `replicante`.

## Relacionados

[mejorante](https://github.com/PascualMacana/mejorante) es un hermano que se copia y además intenta mejorar.  
[demostrante](https://github.com/PascualMacana/demostrante) es un hermano que sólo escribe una mejora afirmada si hay una prueba verificable.  
[reinante](https://github.com/PascualMacana/reinante) es un hermano que se sigue reescribiendo porque el objetivo mismo se mueve.  
[cruzante](https://github.com/PascualMacana/cruzante) es un hermano que se queda con los cruces del río que todavía eran legales.
