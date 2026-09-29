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

## 1. Imágenes PNG

```bash
pngcheck -v imagen.png           # chunks corruptos/extra, dimensiones falseadas
zsteg imagen.png                 # LSB y planos de bits (PNG/BMP) — muy efectivo
zsteg -a imagen.png              # todos los métodos
```

- Dimensiones alteradas ocultan zonas: corrige altura/anchura en la cabecera IHDR.
- Planos de bits y canales: prueba con StegSolve (GUI) — "bit planes", XOR de canales.

## 2. Imágenes JPG

```bash
steghide info imagen.jpg
steghide extract -sf imagen.jpg            # pide passphrase (prueba vacía y del enunciado)
stegseek imagen.jpg wordlist.txt           # fuerza bruta rápida de steghide
outguess -r imagen.jpg salida.txt          # otra herramienta común
```

Si `steghide` pide clave, busca la passphrase en el enunciado, en metadatos, o en otro
archivo del reto. `stegseek` con rockyou suele romper claves débiles.

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
