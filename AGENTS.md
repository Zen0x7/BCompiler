# AGENTS.md

Guía de mantenimiento para agentes y colaboradores de este repositorio.

## Proyecto

`BCompiler` produce imágenes Docker multi-distro que compilan **solo Boost
1.92.0_b1** (beta) + GoogleTest + Google Benchmark y las publica en GHCR
(`ghcr.io/zen0x7/bcompiler`).

- `alpine/Dockerfile` — Alpine (musl); ASAN/UBSAN únicamente (TSAN/MSAN no son viables en musl).
- `ubuntu/Dockerfile` — Ubuntu 24.04 (glibc); ASAN/TSAN/UBSAN/MSAN.
- `debian/Dockerfile` — Debian bookworm (glibc); ASAN/TSAN/UBSAN/MSAN.
- `Dockerfile` — alias retro-compatible de Alpine.
- `scripts/build-boost.sh` — descarga el tarball de Boost desde `archives.boost.io`, ejecuta `bootstrap.sh` y `b2 install`. Acepta `SANITIZER`.
- `scripts/build-gtest.sh`, `scripts/build-benchmark.sh` — GoogleTest v1.16.0 y Google Benchmark v1.9.1 con el mismo build-args.
- `.github/workflows/ci.yml` — build + push a GHCR.
- `LICENSE` — Boost Software License 1.0 (toda cabecera de archivo debe llevarla).

## Variantes

Cross product de `BUILD_VARIANT` (Debug/Release) × `LINK_TYPE` (static/shared) más sanitizers:

| Variante | BUILD_VARIANT | LINK_TYPE | SANITIZER | Alpine | Ubuntu/Debian |
|----------|---------------|-----------|-----------|--------|---------------|
| `debug`  | Debug         | static    | off       | ✅     | ✅ |
| `release`| Release       | shared    | off       | ✅     | ✅ |
| `static` | Release       | static    | off       | ✅     | ✅ |
| `shared` | Debug         | shared    | off       | ✅     | ✅ |
| `asan`   | Release       | shared    | asan      | ✅     | ✅ |
| `tsan`   | Release       | shared    | tsan      | ❌     | ✅ |
| `ubsan`  | Release       | shared    | ubsan     | ✅     | ✅ |
| `msan`   | Release       | shared    | msan      | ❌     | ✅ (requiere clang) |

> Sanitizer builds fuerzan `link=shared runtime-link=shared` y `debug-symbols=on`.
> MSAN usa clang (`toolset=clang` en boost, `clang++` en cmake).

## CI / CD

- Runners nativos **sin QEMU**: `ubuntu-latest` (amd64) y `ubuntu-24.04-arm` (arm64). Cada runner construye su propia arquitectura.
- No usar QEMU ni `--provenance`/attestation (falla con el driver docker de los runners).
- Tags:
  - master: `latest-{variant}-{distro}-{arch}`
  - tag `v*`: `v{version}-{variant}-{distro}-{arch}`
- Disparadores: push a `master`, tags `v*`, `workflow_dispatch`.

## Comandos útiles

```bash
# Build local (variante estática release, Alpine)
docker build -f alpine/Dockerfile --build-arg BUILD_VARIANT=Release --build-arg LINK_TYPE=static -t bcompiler:test .

# Build con sanitizer (ASAN) en Ubuntu
docker build -f ubuntu/Dockerfile --build-arg BUILD_VARIANT=Release --build-arg LINK_TYPE=shared --build-arg SANITIZER=asan -t bcompiler:asan .

# Ejecutar el script fuera de Docker
BOOST_VERSION=1.92.0_b1 BUILD_VARIANT=Release LINK_TYPE=static SANITIZER=off ./scripts/build-boost.sh
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
