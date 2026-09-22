# kittyconfig

Configuración de [Kitty](https://sw.kovidgoyal.net/kitty/) para este equipo. Este README es una guía de referencia rápida de qué se puede configurar y cómo, no documentación oficial completa (esa vive en https://sw.kovidgoyal.net/kitty/conf/).

## Archivos

- `kitty.conf` — configuración principal. Recargar con `ctrl+shift+f5`; algunos ajustes (`env`, barra de tabs) solo se aplican al reiniciar kitty (`cmd+q`).
- `custom.conf` — **session file** de la sesión "Ops" (ver abajo). No es config: define tabs y ventanas a abrir.
- `current-theme.conf` — paleta de colores del tema activo, generada por el kitten de temas (ver abajo). No editar a mano; se regenera cada vez que se cambia de tema.
- `current-font.conf` — tamaño de fuente (`font_size 15.0`).

## Estado actual de este `kitty.conf`

| Línea | Qué hace |
|---|---|
| `map kitty_mod+t new_tab_with_cwd` | La tab nueva (`ctrl+shift+t`) abre en el directorio de la pestaña activa, en vez del default de Kitty (siempre home). |
| `env PATH=${HOME}/.local/bin:/opt/homebrew/bin:/opt/homebrew/sbin:${PATH}` | Abierto desde el Dock, kitty no lee `.zshrc` y hereda un PATH mínimo. Sin esto, los programas que lanza la sesión no encuentran Homebrew (`k8s-htop` falla con `kubectl not found in PATH`). |
| `map kitty_mod+o goto_session ~/.config/kitty/custom.conf` | `ctrl+shift+o` abre la sesión "Ops"; si ya está abierta, salta a ella. Reemplaza el default `pass_selection_to_program`. |
| `# background_opacity` / `# dynamic_background_opacity` | Transparencia, hoy comentada (fondo opaco). |
| Bloque `BEGIN_KITTY_THEME` / `END_KITTY_THEME` | Tema de colores activo, aplicado con el kitten `themes` (actualmente "Selenized Dark"). No tocar a mano — se reescribe solo al cambiar de tema. |
| `enabled_layouts grid,tall:bias=50;full_size=2, *` | Layout por defecto `grid`; `ctrl+shift+l` rota entre los demás. |
| `tab_bar_edge bottom` / `tab_bar_style powerline` | Barra de tabs abajo, con estilo de flechas. Se ve solo con 2+ tabs (`tab_bar_min_tabs` default). |

## Sesión "Ops" (`custom.conf`)

Dashboard de monitoreo en un solo tab, layout `splits`:

```
┌──────────────┬──────────────┐
│              │ finops (55%) │
│              ├──────────────┤
│ shell        │ htop   (23%) │
│              ├──────────────┤
│              │ k8s-htop(22%)│
└──────────────┴──────────────┘
```

Abrir:

```bash
ctrl+shift+o                                   # desde cualquier ventana de kitty
kitty --session ~/.config/kitty/custom.conf    # desde cualquier terminal
```

| Panel | Programa | Notas |
|---|---|---|
| shell | zsh | Queda con el foco al abrir. |
| finops | `~/.local/bin/finops` → `~/Proyectos/finops-console` | Dashboard de costos Azure. Mínimo **76x20**; usa datos cacheados (agregar `--live` para bajar de Azure, requiere `az`). |
| htop | `/opt/homebrew/bin/htop` | Sin tamaño mínimo. |
| k8s-htop | `~/Proyectos/k8s-htop/k8s-htop` | Contexto actual de kubectl. Mínimo 50x8. Requiere `kubectl` en el PATH. |

Detalles del archivo:

- **Rutas absolutas** en los `launch`, para no depender del PATH (`k8s-htop` no está en `~/.local/bin`).
- **`--hold`** en los monitores: al salir con `q` queda un prompt de shell en vez de cerrarse el panel.
- **Alturas dispares con `--bias`** (el % del panel partido que se lleva el nuevo), porque finops necesita 20 filas y en tercios le tocaban ~14. Si la ventana de kitty es más baja, finops puede mostrar `Terminal muy chica`: agrandar la ventana o arrastrar el borde.
- **`--var pane=...` + `--next-to`** para elegir qué panel se parte, sin depender de cuál está activo.
- Si se edita `custom.conf` con la sesión abierta, cerrarla antes de volver a `ctrl+shift+o`, si no salta a la vieja.

Validar el archivo sin abrir la sesión (usa el parser de kitty):

```bash
cd ~/.config/kitty && kitty +runpy "
from kitty.session import parse_session
from kitty.config import load_config
for s in parse_session(open('custom.conf').read(), load_config('kitty.conf')):
    for w in s.tabs[0].windows:
        o = vars(w.launch_spec.opts); print(o['window_title'], o['location'], o['bias'], o['next_to'])
"
```

## Categorías de configuración disponibles

### 1. Comportamiento de tabs y ventanas (splits)

```conf
map kitty_mod+t new_tab_with_cwd          # tab nueva hereda el cwd (ya aplicado)
map kitty_mod+enter new_window_with_cwd   # split hereda el cwd
map ctrl+shift+right next_tab
map ctrl+shift+left  previous_tab
map ctrl+shift+z      toggle_layout stack # maximiza el panel activo temporalmente
enabled_layouts tall,stack,grid           # layouts disponibles para splits
```

### 2. Apariencia — colores y temas

Kitty trae un selector de temas con catálogo curado (~250 temas: Dracula, Nord, Gruvbox, Catppuccin, Material, Solarized, etc.):

```bash
kitty +kitten themes            # interactivo: buscar, previsualizar, aplicar
kitty +kitten themes "Nord"     # aplica directo por nombre
kitty +kitten themes --dump-theme "Nord"   # solo mostrar la paleta, sin aplicar
```

También se pueden fijar colores a mano (sin usar un tema del catálogo):

```conf
background            #1e1e2e
foreground            #cdd6f4
cursor                #f5e0dc
selection_background  #f5e0dc
color0  #45475a
color1  #f38ba8
# ... hasta color15
background_opacity 0.90
background_blur 20          # difumina lo que se ve detrás (requiere compositor)
```

Para volver a los colores por defecto de Kitty: borrar el bloque `BEGIN_KITTY_THEME`/`END_KITTY_THEME` de `kitty.conf`.

### 3. Fuentes

```conf
font_family      JetBrains Mono
bold_font        auto
italic_font      auto
bold_italic_font auto
font_size        12.0
disable_ligatures never     # never | cursor | always
```

Ver fuentes instaladas con soporte para Kitty:
```bash
kitty +list-fonts | grep -i mono
```

### 4. Atajos de teclado (`map`)

```conf
kitty_mod ctrl+shift            # tecla base para los atajos (se puede cambiar)

map ctrl+shift+equal change_font_size all +1
map ctrl+shift+minus change_font_size all -1
map kitty_mod+f2      edit_config_file      # abre kitty.conf en un editor
map ctrl+shift+f5     load_config_file      # recarga config a mano
map ctrl+shift+a>m    change_background_opacity -0.1   # más transparente
map ctrl+shift+a>l    change_background_opacity +0.1   # más opaco
map ctrl+shift+a>1    change_background_opacity 1       # reset a opaco
```

Para quitar un atajo por defecto: `unmap <combinación>`. Para borrar todos y empezar de cero: `clear_all_shortcuts yes`.

### 5. Otros ajustes comunes

```conf
scrollback_lines 10000
copy_on_select yes                 # copia al portapapeles al seleccionar texto
mouse_hide_wait 3.0
confirm_os_window_close 0          # no preguntar al cerrar con procesos corriendo
shell_integration enabled          # marcadores de prompt, cwd tracking, etc.
allow_remote_control yes           # habilita `kitten @ ...` para controlar la sesión activa
listen_on unix:/tmp/kitty          # socket para remote control
```

## Comandos útiles

```bash
kitty --debug-config          # muestra qué config está efectivamente cargada y desde dónde
kitty +list-fonts             # lista fuentes disponibles
kitty +kitten themes          # selector de temas interactivo
```

## Notas

- Kitty no recarga `kitty.conf` al guardar: usar `ctrl+shift+f5`, y para `env` o la barra de tabs, reiniciar con `cmd+q`.
- Este directorio está versionado con git — antes de probar algo experimental, conviene commitear el estado que funciona.
