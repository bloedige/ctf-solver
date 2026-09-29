# Playbook — Crypto & codificaciones

Cubre dos cosas que se confunden: **codificación** (reversible sin clave: base64, hex…)
y **cifrado** (necesita clave/ataque). En forensics/stego casi siempre acabas aquí para
decodificar una pista. Orden: identificar → decodificar/atacar.

## 0. Identificar qué es

```bash
# ¿parece base64/hex/…? Deja que CyberChef "Magic" lo detecte, o a ojo:
echo -n "<texto>" | wc -c        # longitud (base64 múltiplo de 4, acaba en '='; hex par)
```

Pistas visuales:
- Solo `0-9a-fA-F` y longitud par → **hex**.
- `A-Za-z0-9+/` con `=` de relleno → **base64**; solo mayúsculas + `=` → **base32**.
- `%41%42` → **URL-encode**; `\x41` → hex escapado; `&#65;` → HTML entities.
- Palabras desplazadas legibles → **César/ROT**; texto con frecuencia normal pero letras
  cambiadas → **sustitución/Vigenère**.

## 1. Decodificar (sin clave)

```bash
echo -n "<b64>" | base64 -d
echo -n "<b32>" | base32 -d
echo -n "<hex>" | xxd -r -p
python3 -c "import urllib.parse,sys;print(urllib.parse.unquote(sys.argv[1]))" "<txt>"
# ROT13:
echo "<txt>" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

A menudo hay **capas** (base64 de hex de base64…): decodifica en bucle hasta obtener texto
legible / la flag. CyberChef con "Magic" + recetas encadenadas es lo más rápido para esto.

## 2. Cifrados clásicos

- **César/ROT-n:** prueba los 25 desplazamientos (`for i in $(seq 1 25)`), o ROT13.
- **Sustitución monoalfabética:** análisis de frecuencia (quipqiup online, o script).
- **Vigenève:** si sabes/adivinas la clave, descífralo; si no, Kasiski/índice de coincidencia
  para la longitud de clave (herramientas: dcode, cryptool).
- **XOR clave corta:** si conoces parte del texto claro (p. ej. `CIDSI{`), recupera la clave
  por XOR con el cifrado; luego descifra todo. Prueba clave de 1 byte por fuerza bruta.
- **Otros a reconocer:** Atbash, Bacon, Baudot, Morse, Base58 (wallets), Railfence, Playfair.

## 3. Hashes

```bash
hashid "<hash>"           # identifica el tipo
hashcat -m <modo> hash.txt rockyou.txt      # o john
john --format=<fmt> hash.txt --wordlist=rockyou.txt
```

No "descifras" un hash: lo rompes por diccionario/fuerza bruta o lo buscas en bases online.

## 4. RSA

Reúne `n, e, c` (y `p, q` si aparecen). Casos típicos de CTF:

- **n factorizable:** prueba `factordb` (online) o `yafu`; con `p,q` calculas `d`.
- **e pequeño (3) sin padding:** raíz cúbica de `c` si `m^e < n`.
- **primos cercanos:** factorización de Fermat.
- **módulo/clave repetidos, Wiener (d pequeño), Håstad, common modulus.**
- Herramienta que automatiza casi todo: **RsaCtfTool** (`python3 RsaCtfTool.py --publickey key.pub --uncipher c`).

Script base:

```python
from Crypto.Util.number import inverse, long_to_bytes
d = inverse(e, (p-1)*(q-1))
m = pow(c, d, n)
print(long_to_bytes(m))
```

## 5. Simétrico moderno (AES/DES)

- Si tienes clave/IV, descifra con `openssl enc -d` o PyCryptodome.
- **ECB** filtra patrones (bloques iguales → texto igual): útil en oráculos de padding/ECB.
- **Padding oracle / bit flipping (CBC):** si hay un oráculo, explótalo (padbuster o script).

## 6. Al terminar

La flag suele quedar en claro tras la última decodificación/descifrado. Guarda en
`solucion.md` la secuencia exacta de transformaciones (comandos o receta de CyberChef).
