# Reminder

Defina lembretes periódicos e personalizáveis, com mídia, áudio e texto, e com mecanismos específicos para desativação.

## O que faz

- Agenda lembretes (intervalo fixo, aleatório ou por expressão)
- Exibe imagem/vídeo em fullscreen com texto configurável
- Reproduz áudio de acompanhamento
- Modo "dismissal": tempo ou frase obrigatória

## Requisitos

GTK4 >= 4.10, GLib >= 2.76, json-glib >= 1.6, GStreamer >= 1.20.

## Instalação

meson setup build && meson compile -C build && sudo meson install -C build

## Configuração

Defina dinâmicamente através da interface de configuração fornecida pela aplicação.

Arquivo: `~/.config/memento-mori/config.json`
Schema: `data/schemas/config.schema.json`

## Limitações conhecidas

- Não suporta comentaŕios em arquivos em config.json

## Desenvolvimento

- Build: `meson setup build -Dbuild_tests=true`
- Testes: `meson test -C build`
- Debug: `G_MESSAGES_DEBUG=all ./build/memento-mori`

## Licença

[LICENSE](/LICENSE)
