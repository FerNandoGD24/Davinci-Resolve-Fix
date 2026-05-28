# DaVinci Resolve 20.2{21.3b} — Fix en Arch Linux (RDNA2)

## Problemas
- unico lugar disponible para hacer tareas es el gestor de medios
- todo lo demas provoca que se cierre

---
 
## Sistema donde se resolvió
 
| Componente | Versión |
|---|---|
| OS | Arch Linux x86_64 |
| Kernel | 7.0.10-arch1-1 |
| GPU | AMD Radeon RX 6600M (RDNA2 / gfx1032) |
| DE/WM | Hyprland 0.55.2 (Wayland) |
| DaVinci Resolve | Studio 20.2.2.0010 {21.3b} |
| OpenCL | opencl-amd 1:7.2.1-1 (AUR) |
| libc++ | 20.1.6-2 (downgrade desde 22.x) |
 
---
 
## Fix 1 — Generar el locale en_US.UTF-8
Resolve necesita este locale aunque tu sistema esté en otro idioma.
 
```bash
# Agregar en_US.UTF-8 al archivo de locales
echo "en_US.UTF-8 UTF-8" | sudo tee -a /etc/locale.gen
 
# Regenerar locales
sudo locale-gen
 
# Verificar que quedó generado
locale -a | grep en_US
# Debe mostrar: en_US.utf8
```
 
---
 
## Fix 2 — Forzar X11 (XCB) en lugar de Wayland
 
```bash
# Prueba temporal desde terminal
LANG=en_US.UTF-8 QT_QPA_PLATFORM=xcb /opt/resolve/bin/resolve
```
 
Para hacerlo permanente, editar el lanzador de escritorio:
 
```bash
sudo nano /usr/share/applications/com.blackmagicdesign.resolve.desktop
```
 
Cambiar la línea `Exec=` por:
 
```
Exec=env LANG=en_US.UTF-8 QT_QPA_PLATFORM=xcb /opt/resolve/bin/resolve
```
 
---
 
## Fix 3 — Reemplazar ROCm por opencl-amd (AUR)
 
Los paquetes `rocm-*` del repositorio oficial pueden causar crashes adicionales con GPUs RDNA2 en Resolve. Se recomienda usar `opencl-amd` del AUR que empaqueta los drivers directamente desde AMD.
 
```bash
# Instalar opencl-amd (reemplaza rocm-opencl-runtime y otros)
yay -S opencl-amd
# Aceptar la eliminación de los paquetes rocm en conflicto
```
 
---
 
## Fix 4 — Downgrade de libc++ (opcional pero recomendado)
 
Arch actualiza `libc++` más rápido de lo que Blackmagic actualiza Resolve. Si después de una actualización del sistema Resolve deja de arrancar con un error de símbolo, hacer downgrade:
 
```bash
# Descargar versión compatible desde el Arch Archive
wget "https://archive.archlinux.org/packages/l/libc%2B%2B/libc%2B%2B-20.1.6-2-x86_64.pkg.tar.zst"
wget "https://archive.archlinux.org/packages/l/libc%2B%2Babi/libc%2B%2Babi-20.1.6-2-x86_64.pkg.tar.zst"
 
# Instalar
sudo pacman -U libc++-20.1.6-2-x86_64.pkg.tar.zst libc++abi-20.1.6-2-x86_64.pkg.tar.zst
```
 
Pinear para que no se actualicen solos. Editar `/etc/pacman.conf`:
 
```
IgnorePkg = libc++ libc++abi
```
 
---
