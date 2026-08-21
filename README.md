# kittyconfig

Configuración de [Kitty](https://sw.kovidgoyal.net/kitty/) para este equipo. Este README es una guía de referencia rápida de qué se puede configurar y cómo, no documentación oficial completa (esa vive en https://sw.kovidgoyal.net/kitty/conf/).

## Archivos

- `kitty.conf` — configuración principal. Kitty la recarga sola al guardar (no hace falta reiniciar la terminal).
- `current-theme.conf` — paleta de colores del tema activo, generada por el kitten de temas (ver abajo). No editar a mano; se regenera cada vez que se cambia de tema.

## Estado actual de este `kitty.conf`

| Línea | Qué hace |
|---|---|
| `map kitty_mod+t new_tab_with_cwd` | La tab nueva (`ctrl+shift+t`) abre en el directorio de la pestaña activa, en vez del default de Kitty (siempre home). |
| `background_opacity 0.90` | Fondo levemente transparente (90% opaco). |
| `dynamic_background_opacity yes` | Permite subir/bajar la opacidad en caliente sin editar el archivo (ver atajos abajo). |
| Bloque `BEGIN_KITTY_THEME` / `END_KITTY_THEME` | Tema de colores activo, aplicado con el kitten `themes` (actualmente "Oceanic Material"). No tocar a mano — se reescribe solo al cambiar de tema. |

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

- Los cambios en `kitty.conf` se recargan solos al guardar (no hace falta `ctrl+shift+f5` salvo casos raros).
- Este directorio está versionado con git — antes de probar algo experimental, conviene commitear el estado que funciona.
