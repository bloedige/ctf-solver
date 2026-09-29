---
name: ctf-solver
description: >-
  Resuelve retos CTF de tipo Jeopardy de forma autónoma por línea de comandos:
  hace reconocimiento, identifica la categoría (Forensics, Reversing, Exploiting,
  Stego, Crypto, Web, OSINT, Misc), aplica el playbook correcto ejecutando las
  herramientas él mismo, encuentra la flag y guarda los pasos exactos en
  solucion.md. Úsalo SIEMPRE que el usuario mencione un reto CTF, un "challenge",
  buscar/encontrar una flag, un archivo sospechoso de un CTF, un binario/pcap/
  imagen a analizar, o diga cosas como "resuelve este reto", "saca la flag",
  "analiza este archivo", "qué esconde esto". Solo para CTFs, laboratorios y
  sistemas con autorización explícita.
---

# CTF Solver — hallar la flag rápido

Tu único objetivo es **encontrar la flag**. Ejecutas tú las herramientas (no dictas
comandos al usuario), vas directo al grano y, al terminar, dejas los pasos exactos
guardados para reproducirlo.

> **Alcance / ética:** esto es para CTFs, laboratorios y máquinas con permiso
> explícito. Es contexto autorizado de competición/educación. No lo uses contra
> sistemas de terceros sin autorización.

---

## Principio rector: enfoque, no dispersión

El error nº1 al resolver CTFs con una IA es **dispersarse**: encuentra una pista
buena y en vez de explotarla se pone a probar diez cosas más. Evítalo:

- **Una hipótesis a la vez.** Formúlala, pruébala, confírmala o descártala. Luego la siguiente.
- **Cuando encuentres una pista concreta** (una cadena en base64, una wallet, un archivo
  incrustado, una clave), **anótala y trabájala hasta agotarla** antes de abrir otro frente.
- **Regla de los 2 intentos:** si un enfoque no avanza en ~2 intentos reales, cámbialo
  por otro distinto; no repitas variaciones del mismo comando esperando suerte.
- **Registra hallazgos según aparecen** (ver "Cuaderno de hallazgos"). Si el contexto se
  ensucia, ese cuaderno es lo que te permite retomar sin perderlo todo.

---

## Flujo de trabajo

```
Recon  →  Clasificar  →  Playbook de la categoría  →  Extraer flag  →  Verificar  →  Guardar solucion.md
                              └────── iterar con foco ──────┘
```

### 1. Recon (siempre primero, 30 segundos)

Antes de teorizar, mira qué tienes. Ejecuta:

```bash
ls -la
file *
```

Y para cada archivo relevante, según su tipo:

```bash
strings -n 6 <archivo> | head -100     # ¿flag o pistas a simple vista?
strings -n 6 <archivo> | grep -aiE 'flag|ctf|key|pass|token|\{'   # patrón de flag
xxd <archivo> | head -20               # cabecera / magic bytes reales
binwalk <archivo>                      # ¿hay algo incrustado dentro?
```

Muchos retos fáciles se caen aquí mismo. Si ves la flag, salta a "Verificar".

### 2. Clasificar

Decide la categoría a partir del enunciado + el resultado del recon. Guía rápida
(magic bytes / señales → categoría → playbook):

| Señal                                                        | Categoría   | Lee                          |
|--------------------------------------------------------------|-------------|------------------------------|
| `.pcap`, imagen de disco, logs, `file` dice "capture"        | Forensics   | `references/forensics.md`    |
| Imagen/audio (PNG/JPG/WAV) sin dato obvio, enunciado "oculto"| Stego       | `references/stego.md`        |
| `ELF`/`PE`, "crackme", pide contraseña/serial                | Reversing   | `references/reversing.md`    |
| `ELF` + servicio remoto (`nc host port`), overflow, "pwn"    | Exploiting  | `references/exploiting.md`   |
| Texto cifrado/codificado, claves, hashes, RSA, "cipher"      | Crypto      | `references/crypto.md`       |
| URL, "login", cookies, "inyección", app web                  | Web         | `references/web.md`          |
| Nombre de persona/empresa, "busca", perfiles, geolocaliza    | OSINT       | `references/osint.md`        |
| No encaja / programación / mezcla                            | Misc        | `references/misc.md`         |

Si dudas entre dos, empieza por la que el **enunciado** sugiere; el enunciado casi
siempre trae la pista intencionada.

### 3. Aplicar el playbook

Lee **solo** el archivo de `references/` de la categoría elegida y sigue su checklist.
Cada playbook está ordenado de lo más probable/barato a lo más costoso. No leas todos
los playbooks a la vez: cargas contexto inútil y te dispersas.

### 4. Verificar la flag

Antes de cantar victoria:

- La flag debe salir de **ejecutar algo real**, nunca inventada ni "deducida".
- Si el CTF define formato (p. ej. `CIDSI{...}`, `flag{...}`), compruébalo. Si no hay
  formato, la flag puede ser una cadena suelta: valídala por contexto (es lo que el reto pedía).
- Si el reto tiene submit/servicio, la confirmación real es que lo acepte.

### 5. Guardar `solucion.md` (siempre al final)

En la carpeta del reto, crea `solucion.md` con **todos** los pasos concretos que
llevaron a la flag, en orden y copiables. Nada de explicar el porqué; esto es una
receta para reproducir, no un writeup. No omitas ningún paso (ni el recon inicial ni
la decodificación final).

```markdown
# <reto>  (<categoría>)
Flag: <la flag exacta>

Pasos:
1. <comando o acción exacta>
2. <comando o acción exacta>
3. <comando o acción exacta>
```

---

## Cuaderno de hallazgos (para no perder el hilo)

Mantén una nota viva mientras trabajas — en memoria o en un `notas.md` temporal en la
carpeta del reto. Apunta cada dato duro en cuanto aparezca:

- Pistas encontradas (cadenas, claves, wallets, offsets, credenciales, endpoints).
- Qué ya probaste y qué descartaste (para no repetir).
- La hipótesis actual.

Esto es tu seguro contra la pérdida de contexto: si hay que empezar una sesión limpia,
con este cuaderno retomas al instante diciendo "ya tengo estas pistas, trabajo sobre ellas".

---

## Disciplina operativa

- **Ejecuta tú.** Si el entorno pide permiso para correr una herramienta, es esperable
  en un CTF con autorización; procede.
- **No inventes rutas, offsets ni direcciones.** Verifica cada dirección/offset contra
  *este* binario o *este* archivo, nunca de memoria.
- **Herramienta que no existe:** si un comando o flag falla por no existir, no insistas;
  usa `--help`/`man` o cambia de herramienta.
- **Un reto por sesión/carpeta.** No mezcles archivos de retos distintos.
- **Sé económico con el contexto:** lee un playbook, no seis; lee la parte del binario
  que importa, no volcados enormes completos.

---

## Referencias

Playbooks por categoría en `references/` (léelos bajo demanda):

- `references/forensics.md` — pcaps, discos, memoria, metadatos, archivos incrustados.
- `references/stego.md` — datos ocultos en imágenes y audio.
- `references/reversing.md` — entender/derrotar la lógica de un binario.
- `references/exploiting.md` — pwn: overflows, formato, ret2*, pwntools.
- `references/crypto.md` — clásicos, XOR, bases, RSA, hashes.
- `references/web.md` — SQLi, LFI, IDOR, SSTI, cookies/JWT.
- `references/osint.md` — búsqueda dirigida, metadatos, geolocalización.
- `references/misc.md` — programación, formatos raros, cosas que no encajan.
