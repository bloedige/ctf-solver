# Playbook — Reversing

Objetivo: entender qué comprueba el binario y obtener la entrada/valor que suelta la
flag (o extraerla directamente). Orden barato → costoso.

## 0. Recon del binario

```bash
file bin
strings -n 6 bin | grep -aiE 'flag|ctf|key|pass|correct|wrong|\{'
strings -n 6 bin | less        # mensajes, rutas, nombres de función
nm bin ; nm -D bin             # símbolos (si no está stripped)
checksec --file=bin            # protecciones (útil también si acaba en pwn)
./bin                          # ejecútalo: mira su comportamiento e I/O
```

Muchos crackmes fáciles tienen la flag o la contraseña en `strings`. Pruébalo siempre.

## 1. Ejecución dinámica rápida

```bash
ltrace ./bin      # llamadas a librería: ¿strcmp de tu input contra la clave?
strace ./bin      # syscalls: ficheros que abre, lecturas
```

`ltrace` revela oro: un `strcmp("tu_input", "PASSWORD_REAL")` te da la clave gratis.

## 2. Análisis estático (decompilador)

> ⚠️ **No abras Ghidra/IDA/Cutter (GUI) desde la terminal**: el agente se queda **colgado**
> esperando a que la ventana se cierre. Al empezar el análisis estático, **ofrece elegir**:
> > *"Para leer el binario puedo: (A) hacerlo por línea de comandos con radare2/objdump aquí
> > mismo; o (B) tú abres Ghidra y me pegas el `main` (el pseudo-código de Ghidra suele
> > leerse mejor). ¿Cómo prefieres?"*
> Con **(A)** usas los comandos de abajo. Con **(B)** trabajas sobre el pseudo-C que te
> peguen. Si hace falta Ghidra sin intervención humana, úsalo en modo **headless**
> (`analyzeHeadless`), nunca la GUI.

Decompilación/desensamblado por CLI (esto es lo que ejecuta el agente):

```bash
objdump -d -M intel ./bin | less        # desensamblado completo
r2 -A ./bin                             # radare2 con auto-análisis
#   dentro de r2:
#     afl            listar funciones
#     s main         ir a main
#     pdf            desensamblar la función actual
#     pdc            pseudo-código (o pdg si tienes el plugin r2ghidra)
```

Qué buscar en el decompilado/desensamblado:

- Comparaciones contra constantes (contraseña/serial embebido).
- Transformaciones sobre tu input (XOR, sumas, `atoi`, índices) que hay que **invertir**.
- Una flag construida byte a byte en el stack (júntala en orden).
- Ramas `if (check(input)) puts(flag)`.

**Si el humano pega pseudo-C de Ghidra:** la ventana **Decompile** usa nombres inventados
tipo `local_c`; la ventana **Listing** es el ensamblador con registros/offsets reales. No
confundas el número del nombre `local_XX` con el offset real.

## 3. Depuración

```bash
gdb ./bin        # con pwndbg/gef instalados va mucho mejor
# comandos útiles:
#   b main / b *0x...    poner breakpoint
#   run / r arg          ejecutar
#   info functions       listar funciones
#   x/20i $pc            desensamblar
#   x/s <dir>            leer cadena
#   set {char[8]}$rbp-0x.. = ...   parchear memoria
```

Salta el `check` parcheando el salto condicional, o lee el valor correcto en el momento
de la comparación. Para tomar decisiones de ruta rápido, considera **angr** (ejecución
simbólica) cuando la lógica es enrevesada pero la entrada es corta.

## 4. Patrones concretos

- **Serial/keygen:** entiende la validación y escribe el generador inverso en Python.
- **XOR con clave fija:** localiza la clave y `bytes([c ^ k for ...])`.
- **Flag ofuscada en .data/.rodata:** vuélcala y aplica la transformación que hace el binario.
- **Anti-debug (`ptrace`):** parchea la llamada o usa `LD_PRELOAD` para stubbearla.
- **VM/bytecode custom:** identifica el intérprete y reconstruye las instrucciones.
- **.NET / Java:** usa dnSpy / jd-gui / procyon (decompilan casi al fuente).
- **Empaquetado (UPX):** `upx -d bin` para desempacar antes de nada.

## 5. Al obtener la flag

Verifícala ejecutando el binario con la entrada hallada (debe imprimirla / aceptarla) y
guarda en `solucion.md` la cadena/serial exacto y cómo lo derivaste (comando o script).
