El Estilo de Paso de Destino (Destination Passing Style) es una técnica de programación, utilizada más comúnmente (aunque no exclusivamente) en lenguajes con manejo explícito de memoria.

Esta técnica se utiliza para garantizar un único alojamiento de memoria de algún objeto de interés, o evitar alojamientos y copias subsecuentes.

La técnica consiste en realizar un alojamiento del espacio en memoria del objeto, y posteriormente pasar el apuntador a la memoria asignada entre funciones que alterarán el objeto en memoria.

Esto se puede ejemplificar en el siguiente extracto de código:

```rust
fn fun_a(dest: &Array) -> () {...}
fn fun_b(dest: &Array) -> () {...}
fn fun_c(dest: &Array) -> () {...}

fn init_array(size: int) -> &Array {
  // Initial allocation
  let array_ptr: &Array = alloc(size);

  // Construction
  fun_a(array_ptr);
  fun_b(array_ptr);
  fun_c(array_ptr);

  // Return pointer to only allocation
  return array_ptr;
}
```

Podemos asumir que las funciones `fun_a`, `fun_b`, `fun_c` únicamente alteran el arreglo a través del apuntador `array_ptr`, por lo que la única asignación de memoria se realizó mediante la función `alloc`.