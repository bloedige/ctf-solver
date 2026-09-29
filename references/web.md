# Playbook — Web

Objetivo: encontrar la vulnerabilidad de la app y sacar la flag (en respuesta, BD,
archivo del server o panel). Contexto: reto CTF con URL/servicio autorizado.

## 0. Recon

```bash
# fuentes y cabeceras
curl -sI <url>                        # cabeceras, tecnología, cookies
curl -s <url> | less                  # HTML: comentarios, rutas, JS
# archivos y rutas típicas
curl -s <url>/robots.txt
curl -s <url>/.git/HEAD               # ¿repo expuesto? -> git-dumper
```

Mira **siempre**: comentarios HTML, JS del cliente (lógica y endpoints ocultos),
`robots.txt`, `sitemap.xml`, `/.git/`, `/backup`, `.bak`, cookies y su contenido.

Fuzzing de rutas/params cuando haga falta:

```bash
ffuf -u <url>/FUZZ -w wordlist.txt -mc 200,301,302,403
gobuster dir -u <url> -w wordlist.txt
```

## 1. Inyección SQL (SQLi)

- Detección: `'`, `"`, `1 or 1=1`, `admin'--` en parámetros/login.
- Manual: `UNION SELECT` para volcar (`ORDER BY` para nº de columnas), o blind
  (booleana/tiempo con `SLEEP`).
- Rápido: `sqlmap -u "<url>?id=1" --batch --dump` (con `--data` para POST, `--cookie`).
- Objetivo: leer tabla de usuarios/flags, o `load_file`/`INTO OUTFILE` según motor.

## 2. Inclusión de archivos / path traversal (LFI/RFI)

- `?page=../../../../etc/passwd`; wrappers PHP: `php://filter/convert.base64-encode/resource=index.php`
  para leer el fuente (y encontrar dónde está la flag).
- LFI → RCE vía log poisoning, `/proc/self/environ`, o wrappers `data://`/`expect://`.

## 3. Autenticación / autorización

- **IDOR:** cambia `id`/UUID en la URL o API para acceder a datos de otros.
- **JWT:** decodifica (`base64` de las 3 partes). Ataques: `alg:none`, clave débil
  (`hashcat -m 16500`), confusión RS256→HS256.
- **Cookies:** valores predecibles, firmas débiles, `admin=false` manipulable.

## 4. Inyección de plantillas / comandos

- **SSTI:** prueba `{{7*7}}`, `${7*7}`, `#{7*7}`; si evalúa, escala a RCE según motor
  (Jinja2, Twig, Freemarker…).
- **Command injection:** `; id`, `| id`, `$(id)`, backticks en parámetros que ejecutan.
- **XXE:** en endpoints que parsean XML, define una entidad externa para leer archivos.
- **SSRF:** haz que el server pida URLs internas (`http://localhost/…`, metadata cloud).

## 5. Cliente

- **XSS:** más común en retos con "admin/bot" que visita tu payload → roba cookie/flag.
- Revisa CSP, sinks (`innerHTML`, `eval`), y si hay un bot que hay que engañar.

## 6. Herramientas

- **Burp Suite** (proxy) para interceptar/repetir/manipular peticiones — el caballo de batalla.
- `curl`/`httpie` para scripts; `ffuf`/`gobuster` para fuzz; `sqlmap` para SQLi.

## 7. Al obtener la flag

Guarda en `solucion.md` la petición exacta (curl con parámetros/headers/cookies) o los
pasos en Burp que devolvieron la flag.
