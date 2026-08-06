# AGENTS.md

Guía de mantenimiento para agentes y colaboradores de este repositorio.

## Proyecto

`BCompiler` produce una imagen Docker Alpine (`alpine:3.24.1`) que compila
**solo Boost 1.92.0_b1** (beta) y la publica en GHCR (`ghcr.io/zen0x7/bcompiler`).

- `Dockerfile` — base Alpine; acepta `BUILD_VARIANT`, `LINK_TYPE`, `BOOST_VERSION`, `BOOST_LIBS`. Incluye `LICENSE` en la imagen en `/usr/share/licenses/boost-license/`.
- `scripts/build-boost.sh` — descarga el tarball de Boost desde `archives.boost.io`, ejecuta `bootstrap.sh` y `b2 install`.
- `.github/workflows/ci.yml` — build + push a GHCR.
- `LICENSE` — Boost Software License 1.0 (toda cabecera de archivo debe llevarla).

## Variantes

Cross product de `BUILD_VARIANT` (Debug/Release) × `LINK_TYPE` (static/shared):

| Variante | BUILD_VARIANT | LINK_TYPE |
|----------|---------------|-----------|
| `debug`  | Debug         | static    |
| `release`| Release       | shared    |
| `static` | Release       | static    |
| `shared` | Debug         | shared    |

## CI / CD

- Runners nativos **sin QEMU**: `ubuntu-latest` (amd64) y `ubuntu-24.04-arm` (arm64). Cada runner construye su propia arquitectura.
- No usar QEMU ni `--provenance`/attestation (falla con el driver docker de los runners).
- Tags:
  - master: `latest-{variant}-{arch}`
  - tag `v*`: `v{version}-{variant}-{arch}`
- Disparadores: push a `master`, tags `v*`, `workflow_dispatch`.

## Comandos útiles

```bash
# Build local (variante estática release)
docker build --build-arg BUILD_VARIANT=Release --build-arg LINK_TYPE=static -t bcompiler:test .

# Ejecutar el script fuera de Docker
BOOST_VERSION=1.92.0_b1 BUILD_VARIANT=Release LINK_TYPE=static ./scripts/build-boost.sh
```

## Cómo comitear

El agente **no** debe commitear salvo que el usuario lo pida explícitamente.
Cuando se pida:

```bash
git status                 # revisar cambios
git diff                   # revisar el contenido
git add <archivos>         # stagear solo lo intencional
git commit -m "mensaje"    # mensaje descriptivo, estilo del repo (ver git log)
```

Reglas:
- No commitear secretos ni archivos de entorno.
- Mensajes claros, imperativo, primera línea corta.
- No amendear commits fallidos: crear commit nuevo.
