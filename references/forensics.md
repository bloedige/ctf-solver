# Playbook — Forensics

Objetivo: extraer la flag de artefactos (capturas de red, imágenes de disco, volcados
de memoria, archivos con datos incrustados o metadatos). Ordenado de barato a costoso.

## 0. Identificar el artefacto

```bash
file *
binwalk <archivo>          # ¿algo incrustado? (muy común en forensics)
exiftool <archivo>         # metadatos (autor, GPS, comentarios, software)
strings -n 6 <archivo> | grep -aiE 'flag|ctf|key|pass|\{'
```

`binwalk` casi siempre paga: si detecta archivos anidados, extráelos:

```bash
binwalk -e <archivo>       # extrae a _<archivo>.extracted/
foremost -i <archivo> -o out_foremost   # alternativa de carving
```

## 1. Capturas de red (.pcap / .pcapng)

```bash
capinfos captura.pcap                 # resumen
tshark -r captura.pcap -q -z io,phs   # jerarquía de protocolos: ¿qué hay?
tshark -r captura.pcap -q -z conv,tcp # conversaciones
```

Pistas frecuentes y filtros (Wireshark o `tshark -Y '<filtro>'`):

- **HTTP:** `http.request` / `http.response`; exporta objetos:
  `File > Export Objects > HTTP` (GUI) para sacar archivos transferidos.
- **Credenciales/texto plano:** `http`, `ftp`, `telnet`, `imap`, `smtp`.
- **DNS exfil:** `dns` — subdominios largos/raros suelen esconder datos (a veces base32/hex).
- **Seguir un flujo:** `tshark -r captura.pcap -z follow,tcp,ascii,<stream>`.
- **Extraer un campo:** `tshark -r captura.pcap -Y 'dns' -T fields -e dns.qry.name`.

Recompón datos exfiltrados concatenando el campo y decodifica (ver `crypto.md`).
Si hay TLS y te dan una `SSLKEYLOGFILE`/clave, cárgala en Wireshark para descifrar.

USB HID en pcap (teclados): extrae `usb.capdata` y traduce los keycodes a texto.

## 2. Imágenes de disco / sistemas de archivos

```bash
file imagen.dd
fdisk -l imagen.dd                 # particiones
binwalk -e imagen.dd
# Montar (solo lectura) una partición en su offset:
mount -o ro,loop,offset=$((SECTOR*512)) imagen.dd /mnt/x
# Recuperar borrados:
foremost -i imagen.dd -o out
# Grande / NTFS-EXT: usa Autopsy o The Sleuth Kit (fls, icat)
fls -r -o <offset> imagen.dd
icat -o <offset> imagen.dd <inode> > archivo_recuperado
```

Busca en el árbol: papelera, historiales, `.bash_history`, archivos ocultos (`.` inicial),
ADS en NTFS, timestamps anómalos.

## 3. Volcados de memoria (RAM)

Usa **Volatility** (3 preferente):

```bash
python3 vol.py -f dump.raw windows.info          # perfil/SO
python3 vol.py -f dump.raw windows.pslist        # procesos
python3 vol.py -f dump.raw windows.cmdline       # líneas de comando
python3 vol.py -f dump.raw windows.filescan | grep -i flag
python3 vol.py -f dump.raw windows.dumpfiles --viraddr <addr>
python3 vol.py -f dump.raw windows.netscan
```

En Linux: plugins `linux.*`. Busca procesos raros, comandos, archivos en memoria,
y `strings dump.raw | grep -i flag` como red de seguridad.

## 4. Documentos y archivos ofimáticos

- Office/`.docx`/`.xlsx`/`.pptx` son ZIP: `unzip -l doc.docx` y revisa XML + `word/media/`.
- PDF: `pdf-parser.py`, `pdfdetach`, `strings`, `binwalk`; JavaScript embebido y objetos ocultos.
- Comentarios y metadatos con `exiftool`.

## 5. Cuando encuentres datos codificados

Es lo normal en forensics: cadenas en base64/base32/hex/URL-encode. Anótalas en el
cuaderno de hallazgos y decodifícalas siguiendo `crypto.md` (o CyberChef con "Magic").
No abras un frente nuevo hasta agotar la pista que ya tienes.
