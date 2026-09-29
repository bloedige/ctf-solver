# Playbook — Misc

Cajón de sastre: programación, formatos raros, "jails", cosas que no encajan en otra
categoría. Estrategia: identificar el patrón concreto y atacarlo.

## 0. Identifica el subtipo

```bash
file *
strings -n 6 <archivo> | head -100
```

## 1. Retos de programación / algoritmia

- El server manda muchos problemas rápidos (mates, secuencias, decodificaciones) y hay
  que resolverlos **automáticamente** dentro de un límite de tiempo.
- Conéctate y automatiza con **pwntools** (`remote()`), parsea la pregunta, calcula,
  responde en bucle hasta que suelte la flag.
- Recon de patrones: base64/hex por línea, aritmética, "invierte esto", etc.

## 2. Jails (pyjail / bashjail)

- Te dan un intérprete restringido; el objetivo es ejecutar/leer la flag saltándote el filtro.
- **pyjail:** evade blacklists con `getattr`, `__builtins__`, `__import__`, `().__class__.
  __bases__`, f-strings, `breakpoint()`; recupera `os.system`/`open` por vías indirectas.
- **bashjail:** variables, `${IFS}`, comodines, `$(...)`, rutas alternativas a binarios.

## 3. Formatos y datos raros

- **Archivos comprimidos:** `unzip -l`, contraseñas con `fcrackzip`/`john` (zip2john),
  bombas anidadas (zip dentro de zip → automatiza el desanidado).
- **QR / códigos de barras:** `zbarimg imagen.png`.
- **Esolangs** (Brainfuck, Whitespace, Ook, Piet): identifícalo e interpreta (intérpretes online).
- **Serialización** (pickle, protobuf, ASN.1): deserializa y examina.
- **Datos binarios estructurados:** `xxd`, deduce el layout, parsea con Python `struct`.

## 4. Timing / juegos / interacción

- Retos que exigen ganar un "juego", predecir un RNG (semilla débil → replica el PRNG),
  o responder más rápido que un humano → automatiza con pwntools.

## 5. Cuando aparezca algo codificado o de otra categoría

Es normal que un Misc se transforme en Crypto/Stego a mitad. Cambia al playbook que
corresponda (`crypto.md`, `stego.md`, …) y sigue. Anota la pista en el cuaderno.

## 6. Al terminar

Guarda en `solucion.md` el script o los pasos exactos que produjeron la flag.
