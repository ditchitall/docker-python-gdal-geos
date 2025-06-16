# docker-python-gdal-geos

Docker images bundling **Python (3.12 / 3.13)** with **GDAL 3.10.0**, **GEOS 3.11.5**, and **PROJ 9.5.1** — built from source for robust geospatial workloads.

---

## ✅ Features

- Currently only Python 3.12 and 3.13 are supported
- GDAL 3.10.0, GEOS 3.11.5, PROJ 9.5.1
- Tagged variants for Lambda and Slim Debian
- Lightweight builds with clean dependencies

---

## 📦 Tags

| Tag                          | Base Image                          |
|-----------------------------|--------------------------------------|
| `py3.12-lambda`             | `public.ecr.aws/lambda/python:3.12` |
| `py3.13-lambda`             | `public.ecr.aws/lambda/python:3.13` |
| `py3.12-slim-bookworm`     | `python:3.12-slim-bookworm`         |
| `py3.13-slim-bookworm`     | `python:3.13-slim-bookworm`         |

---

## 🚀 Usage

### Run interactively

```bash
docker run --rm -it ditchitall/python-gdal-geos:py3.13-slim-bookworm bash