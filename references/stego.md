# Playbook — Esteganografía

Objetivo: sacar datos ocultos dentro de imágenes o audio. Orden barato → costoso.
Regla de oro: **primero lo mecánico** (strings, binwalk, metadatos) antes de LSB o cosas finas.

## 0. Siempre primero (cualquier archivo)

```bash
file archivo
exiftool archivo                 # comentarios, autor, campos raros
strings -n 6 archivo | grep -aiE 'flag|ctf|key|pass|\{'
binwalk archivo && binwalk -e archivo   # ¿archivo/zip incrustado?
xxd archivo | head; xxd archivo | tail   # magic bytes y cola (datos tras EOF)
```

Truco clásico: datos **después del fin de imagen**. Un ZIP pegado tras un PNG/JPG se
extrae con `binwalk -e` o directamente `unzip archivo.png`.

## 0.5 ¿Hay algo escondido? (cuando file/exiftool/binwalk no muestran nada)

No hay un detector 100% fiable —el stego bien hecho es invisible a propósito— pero estos
comandos delatan la mayoría de los payloads que el recon normal pasa por alto:

```bash
binwalk -E archivo          # entropía: una zona alta y plana = blob comprimido/cifrado oculto
ent archivo                 # entropía global (~8.0 bits/byte sugiere datos cifrados/comprimidos)
xxd archivo | tail          # ¿datos DESPUÉS del fin del archivo? (lo más común)
foremost -i archivo -o out_fore   # carving alternativo cuando binwalk no marca nada
binwalk -Me archivo         # extracción recursiva profunda
stegoveritas archivo        # batería automática de chequeos de stego (LSB, canales, metadatos…)
```

Marcas de fin de archivo (lo que venga **después** es payload incrustado):
`PNG` → `49 45 4E 44 AE 42 60 82` (IEND) · `JPG` → `FF D9` · `GIF` → `3B`.

Señal indirecta: una imagen "sencilla" que **pesa de más** suele esconder algo; compárala
con otra similar. Regla rápida: `binwalk -E` + `xxd | tail` + `stegoveritas` cubren casi todo.

## 1. Imágenes PNG

```bash
pngcheck -v imagen.png           # chunks corruptos/extra, dimensiones falseadas
zsteg imagen.png                 # LSB y planos de bits (PNG/BMP) — muy efectivo
zsteg -a imagen.png              # todos los métodos
```

- Dimensiones alteradas ocultan zonas: corrige altura/anchura en la cabecera IHDR.
- Planos de bits y canales: prueba con StegSolve (GUI) — "bit planes", XOR de canales.

**Canal alfa y transparencia (RGBA) — escondite muy típico:**

Un PNG con transparencia tiene un 4º canal (alfa). Los píxeles **totalmente transparentes
(alpha = 0) son invisibles pero conservan su valor RGB**, así que se puede esconder una
imagen/texto "debajo" de lo transparente, o modular datos en el propio canal alfa.

```bash
# Separar el canal alfa como escala de grises (a menudo "dibuja" la flag):
convert imagen.png -alpha extract alpha.png
# Apagar la transparencia para revelar el RGB oculto bajo los píxeles invisibles:
convert imagen.png -alpha off revelado.png
# zsteg ya cubre el canal alfa y sus planos de bit:
zsteg -a imagen.png          # busca 'a' (alpha) en la salida; prueba bit 0 del alfa
```

Si `-alpha extract` o `-alpha off` muestran algo distinto a la imagen visible, ahí está el dato.

**Otros escondites en PNG:**

- **Chunks de texto:** `tEXt`, `zTXt`, `iTXt` guardan cadenas (a veces la flag o pistas):
  `exiftool imagen.png` o `pngcheck -v` los listan; `zTXt` va comprimido.
- **Chunks privados/no estándar:** `pngcheck -v` marca chunks raros o fuera de orden; vuélcalos.
- **Paleta (PLTE) en PNG indexado:** colores casi idénticos o índices reordenados ocultan datos.

## 2. Imágenes JPG

```bash
steghide info imagen.jpg
steghide extract -sf imagen.jpg            # pide passphrase (prueba vacía y del enunciado)
stegseek imagen.jpg wordlist.txt           # fuerza bruta rápida de steghide
outguess -r imagen.jpg salida.txt          # otra herramienta común
```

Si `steghide` pide clave, busca la passphrase en el enunciado, en metadatos, o en otro
archivo del reto. `stegseek` con rockyou suele romper claves débiles.

**Passphrase escondida a plena vista (heurística muy común):** si el reto lleva algo
incrustado con steg y **pide contraseña**, y ni los metadatos (`exiftool`) ni `strings`
dan una pista clara, fíjate en el **texto que aparece dibujado dentro de la propia imagen**
(un cartel, una frase, un nombre, una marca de agua). Esa palabra suele ser la passphrase.
El texto renderizado como píxeles no sale en `strings`; hay que leerlo con OCR:

```bash
tesseract imagen.png stdout        # OCR: saca el texto visible en la imagen
# Arma un diccionario de candidatos con esas palabras + las del enunciado,
# nombre de archivo, autor de exiftool y strings; guárdalas en candidatos.txt y prueba:
stegseek imagen.jpg candidatos.txt         # rápido, prueba toda la lista
steghide extract -sf imagen.jpg -p "PALABRA"   # o una a una a mano
```

Prueba también variantes: minúsculas/mayúsculas, sin espacios, con/sin signos. La clave
casi siempre es una de las palabras que el reto te está mostrando.

## 3. Audio (WAV / MP3)

- **Espectrograma:** abre en Audacity/Sonic Visualiser → vista de espectro; muchas flags
  se "dibujan" ahí. Con sox: `sox audio.wav -n spectrogram -o spec.png`.
- **LSB en audio:** herramientas tipo `WavSteg` (stegolsb).
- **DTMF / Morse:** tonos telefónicos o pitidos → decodifica DTMF/Morse.
- `steghide` también soporta WAV.

## 4. Otros trucos

- **Zeros/whitespace stego** en texto: espacios/tabs al final de línea → decodifica.
- **QR/códigos** troceados o invertidos dentro de la imagen.
- **Múltiples capas:** un stego dentro de otro; repite el proceso sobre lo extraído.
- **Paletas y transparencia** (GIF/PNG): índices de color ocultan datos.

## 5. Si sacas datos codificados

Base64/hex/etc. → ve a `crypto.md` (o CyberChef "Magic"). Agota esa pista antes de
probar otra herramienta de stego; anótala en el cuaderno de hallazgos.
