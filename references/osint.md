# Playbook — OSINT

Objetivo: encontrar información pública (persona, empresa, lugar, evento) que revele la
flag. Aquí **la IA no delega el buscar en herramientas**: hay que buscar de verdad, con
método. Contexto: reto autorizado; no acoses ni contactes a personas reales.

## 0. Ordena las pistas del enunciado

Extrae todo dato duro: nombres, apodos/usernames, correos, dominios, fechas, coordenadas,
nombres de archivo, hashtags. Cada uno es un hilo. Anótalos en el cuaderno de hallazgos.

## 1. Metadatos primero (si dan un archivo)

```bash
exiftool archivo      # GPS, fecha, cámara/dispositivo, autor, software
```

GPS en EXIF resuelve muchos retos de geolocalización al instante.

## 2. Búsqueda de identidades

- **Username:** el mismo alias suele repetirse en GitHub, X, Reddit, foros (sherlock,
  whatsmyname). Revisa perfiles, repos, gists, historial.
- **Email:** búscalo entre comillas; comprueba brechas conocidas; deduce el patrón de la org.
- **Nombre real:** buscador + comillas, redes profesionales, notas de prensa.
- **Dominio:** `whois dominio`, registros DNS, certificados (crt.sh), Wayback Machine
  (versiones antiguas del sitio con datos ya borrados).

## 3. Geolocalización de imágenes

- Sin EXIF: geolocaliza por pistas visuales (idioma de carteles, matrículas, arquitectura,
  señales, vegetación, sol/sombras).
- **Reverse image search** (varios motores) para hallar el origen.
- Coteja con mapas y street view; usa hitos (torres, montañas, líneas de costa).

## 4. Redes y contenido

- **Búsqueda inversa** de fotos de perfil para enlazar cuentas.
- Publicaciones antiguas, imágenes borradas (caché/Wayback), comentarios.
- Metadatos de documentos publicados (autor, rutas internas).

## 5. Disciplina

- Trabaja un hilo hasta agotarlo antes de saltar a otro.
- Guarda URLs exactas de dónde salió cada dato: son tu prueba y tus pasos.
- Si un hilo no lleva a nada en ~2 intentos, pasa al siguiente dato duro.

## 6. Al obtener la flag

En `solucion.md`: qué dato de partida usaste, en qué recurso/URL lo encontraste y cómo
derivaste la flag. Enumera las URLs concretas.
