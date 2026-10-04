# Reminder

Nome da aplicação: **Reminder**.

Aplicação escrita em **C** usando **GTK4**.

---

## Funcionamento do serviço

- Um timer define quando o conteúdo aparece.
- O usuário decide se o timer se repete em loop ou dispara uma única vez.
- O conteúdo pode ser exibido em fullscreen.
- Arquivo de configuração em JSON: `~/.config/reminder/config.json`.
- O bloqueio de tela do modo timed é uma janela GTK em fullscreen capturando foco, e irá haver a tentativa de utilização do portal de sessão (org.freedesktop.impl.portal.LockScreen ou logind) para um bloqueio real do ambiente.

---

## Funcionalidades do usuário

- Definir vários lembretes, cada um independente.
- O usuário poderá definir o idioma da aplicação a qualquer momento.

---

## UI de configuração

### Conteúdo exibido em cada lembrete

- Selecionar imagem **ou** vídeo.
- Definir texto exibido sob (ou sobre) a imagem/vídeo.
- Definir áudio reproduzido junto.

### Controles gerais

- Botão para alternar fullscreen.
- Botão para tornar o tempo do timer aleatório, com inputs para `min` e `max`.
- Botão para restringir a janela de horários em que o scheduler pode disparar.
- Botões para:
  - Salvar configuração.
  - Importar configuração.
  - Exportar configuração.
  - Executar serviço.
- Botão para habilitar inicialização automática com o sistema.
- Opção de dismissal:
  - Padrão: bloqueio temporário da tela até o timeout terminar.
  - Alternativa: exigir que o usuário digite uma frase específica.

---

## Estrutura do projeto

reminder/meson.build
reminder/meson.options
reminder/README.md
reminder/LICENSE
reminder/.gitignore
reminder/.editorconfig
reminder/include/reminder/model/config.h
reminder/include/reminder/model/reminder.h
reminder/include/reminder/model/schedule.h
reminder/include/reminder/core/app.h
reminder/include/reminder/core/scheduler.h
reminder/include/reminder/core/service.h
reminder/include/reminder/ui/window.h
reminder/include/reminder/ui/reminder_editor.h
reminder/include/reminder/ui/settings_dialog.h
reminder/include/reminder/media/player.h
reminder/include/reminder/platform/autostart.h
reminder/include/reminder/platform/desktop.h
reminder/include/reminder/reminder.h
reminder/src/main.c
reminder/src/core/app.c
reminder/src/core/scheduler.c
reminder/src/core/service.c
reminder/src/model/config.c
reminder/src/model/config_schema.c
reminder/src/model/config_migrate.c
reminder/src/model/reminder.c
reminder/src/model/schedule.c
reminder/src/ui/window.c
reminder/src/ui/reminder_editor.c
reminder/src/ui/settings_dialog.c
reminder/src/ui/text_layout.c
reminder/src/media/player.c
reminder/src/platform/autostart.c
reminder/src/platform/desktop.c
reminder/data/icons/org.example.Reminder.svg
reminder/data/ui/window.ui
reminder/data/ui/reminder_editor.ui
reminder/data/ui/settings_dialog.ui
reminder/data/schemas/config.schema.json
reminder/data/org.example.Reminder.desktop.in
reminder/data/org.example.Reminder.metainfo.xml.in
reminder/data/org.example.Reminder.gschema.xml.in
reminder/po/LINGUAS
reminder/po/POTFILES.in
reminder/po/pt_BR.po
reminder/po/es.po
reminder/docs/README.md
reminder/docs/architecture.md
reminder/docs/config-versioning.md
reminder/docs/text-layout.md
reminder/docs/security.md
reminder/tests/meson.build
reminder/tests/test_config.c
reminder/tests/test_reminder.c
reminder/tests/test_scheduler.c
reminder/tests/test_text_layout.c
reminder/packaging/systemd/reminder.service.in
reminder/packaging/runit/run
reminder/packaging/flatpak/org.example.Reminder.yml

---

## Responsabilidade de cada módulo

| Módulo | Responsabilidade |
|---|---|
| `main.c` | Inicialização do processo e tratamento dos argumentos |
| `app.c` | Ciclo de vida e coordenação da aplicação |
| `config.c` | Leitura, escrita e validação do JSON |
| `config_schema.c` | Validação do JSON contra `config.schema.json` |
| `config_migrate.c` | Migração de configurações entre versões |
| `reminder.c` | Modelo de dados e regras de cada lembrete |
| `schedule.c` | Estrutura e cálculo de agendamento de cada lembrete |
| `scheduler.c` | Temporizadores, recorrência e sorteio de intervalos |
| `player.c` | Carregamento e reprodução de mídia |
| `window.c` | Janela de exibição, fullscreen e apresentação do conteúdo |
| `text_layout.c` | Posicionamento e estilo do texto sobre a mídia |
| `reminder_editor.c` | Formulário GTK para criar e editar lembretes |
| `settings_dialog.c` | Diálogo de configurações globais do serviço |
| `service.c` | Inicialização e controle do serviço |
| `platform/` | Integração com recursos específicos do ambiente Linux |

---

## Estrutura do `config.json`

```json
{
  "version": 1,
  "service": {
    "autostart": false
  },
  "reminders": [
    {
      "id": "reminder-001",
      "enabled": true,
      "schedule": {
        "mode": "random",
        "min_seconds": 300,
        "max_seconds": 900,
        "repeat": true,
        "active_window": ["08:00", "22:00"]
      },
      "presentation": {
        "fullscreen": true,
        "media_type": "image",
        "media_path": "/home/user/Pictures/example.png",
        "text": "Memento mori.",
        "text_layout": {
          "placement": "overlay",
          "anchor": "bottom",
          "align": "center",
          "margin_px": 24,
          "background": "rgba(0,0,0,0.5)",
          "font": "Sans 32",
          "color": "#ffffff"
        },
        "audio_path": null
      },
      "dismissal": {
        "mode": "timed",
        "duration_seconds": 15,
        "required_phrase": null
      }
    }
  ]
}
```

---

## `text_layout`

Define como o texto é posicionado sobre/sob a mídia. Três modos suportados:

- **`placement: "below"`** — o texto fica em um `GtkBox` vertical logo abaixo da mídia.
- **`placement: "overlay"`** — o texto fica sobreposto à mídia via `GtkOverlay`.
- **`placement: "free"`** — o texto é posicionado por `x_percent` / `y_percent` (0–100).

Campos comuns:

| Campo | Tipo | Descrição |
|---|---|---|
| `placement` | `below \| overlay \| free` | Como o texto se relaciona com a mídia |
| `anchor` | `top \| center \| bottom` | Alinhamento vertical (usado em `overlay`) |
| `align` | `start \| center \| end` | Alinhamento horizontal |
| `margin_px` | inteiro | Margem em pixels |
| `background` | string RGBA | Fundo do texto; `null` para transparente |
| `font` | string Pango | Ex.: `"Sans 32"` |
| `color` | string RGB | Ex.: `"#ffffff"` |
| `x_percent` / `y_percent` | 0–100 | Usado apenas quando `placement: "free"` |

---

## `meson.build` (raiz do projeto)

```meson
project(
  'reminder',
  'c',
  version: '0.1.0',
  default_options: ['c_std=c17', 'warning_level=3', 'werror=false'],
  meson_version: '>= 0.60.0',
)

i18n  = import('i18n')
gnome = import('gnome')

# --- Gera config.h com GETTEXT_PACKAGE e LOCALEDIR ---
conf = configuration_data()
conf.set_quoted('GETTEXT_PACKAGE', 'reminder')
conf.set_quoted('LOCALEDIR', get_option('prefix') / get_option('localedir'))
conf.set_quoted('VERSION', meson.project_version())
configure_file(output: 'config.h', configuration: conf)
add_project_arguments('-DHAVE_CONFIG_H', language: 'c')

# --- Dependências ---
gtk_dep        = dependency('gtk4')
glib_dep       = dependency('glib-2.0')
gio_dep        = dependency('gio-2.0')
json_dep       = dependency('json-glib-1.0')
gst_dep        = dependency('gstreamer-1.0')
gst_video_dep  = dependency('gstreamer-video-1.0')

deps = [
  gtk_dep,
  glib_dep,
  gio_dep,
  json_dep,
  gst_dep,
  gst_video_dep,
]

subdir('src')
subdir('data')
subdir('po')
subdir('tests')
subdir('docs')
```

---

## i18n (gettext)

### `po/meson.build`

```meson
i18n.gettext(
  'reminder',
  preset: 'glib',
  args: [
    '--from-code=UTF-8',
    '--add-comments=TRANSLATORS:',
    '--keyword=_',
    '--keyword=N_',
    '--keyword=C_:1c,2',
    '--keyword=ngettext:1,2',
  ],
)
```

### `po/LINGUAS`

```text
pt_BR
es
fr
de
```

### `po/POTFILES.in`

```text
src/main.c
src/core/app.c
src/core/scheduler.c
src/core/service.c
src/model/config.c
src/model/reminder.c
src/model/schedule.c
src/ui/window.c
src/ui/reminder_editor.c
src/ui/settings_dialog.c
src/ui/text_layout.c
data/org.example.Reminder.desktop.in
data/org.example.Reminder.metainfo.xml.in
data/org.example.Reminder.gschema.xml.in
```

# Prompt

Desenvolva o README.md explicando o funcionamento básico do projeto, como:
Propósito da aplicação.
Diretório de configurações.

# Mais detalhes

Inspirar interface em: https://memento-mori.matthewsimo.com/

Permitir o usuário definir a data de nascimento, para ver quanto dias faltam para ele ir de arrasta.
Fornecer representação de quantas semanas se passaram e quantas falta, em várias bolinhas, cada bolinha representa uma semana.
As semanas que passaram ficam em bolinhas vermelhas.
A semana atual, a bolinha fica amarela.
A semanas futuras/que faltam até a idade comum de morte do sujeito, irão ficar cinzas.

Opção para o usuário escolher frases aleatórias pré-definidas para serem exibidas sob a mídia.
