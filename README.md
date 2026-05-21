# hass-magic-caster-wand-fx
[![HACS](https://img.shields.io/badge/HACS-Custom-41BDF5.svg?logo=home-assistant)](https://hacs.xyz/)
[![GitHub Release](https://img.shields.io/github/release/L-Milko/hass-magic-caster-wand-fx.svg)](https://github.com/L-Milko/hass-magic-caster-wand-fx/releases)
[![License](https://img.shields.io/github/license/L-Milko/hass-magic-caster-wand-fx)](https://github.com/L-Milko/hass-magic-caster-wand-fx/blob/main/LICENSE)
![integration usage](https://img.shields.io/badge/dynamic/json?color=41BDF5&logo=home-assistant&label=integration%20usage&suffix=%20installs&cacheSeconds=15600&url=https://analytics.home-assistant.io/custom_integrations.json&query=%24.magic_caster_wand_fluid.total)

Magic Caster Wand FX is a Home Assistant custom integration for Magic Caster Wand Bluetooth control, WebGL fluid spell visuals, finger or mouse spell casting, and multi-wand play.

This fork uses the separate Home Assistant domain `magic_caster_wand_fluid`, so it can be installed without overlapping the original `magic_caster_wand` integration.

## Supported Models

- Loyal
- Honourable
- Defiant

## Installation

1. Install with HACS as a custom repository: `https://github.com/L-Milko/hass-magic-caster-wand-fx`.
2. Select category `Integration`.
3. Restart Home Assistant after installing or updating.
4. Add the integration from **Settings -> Devices & services -> Add integration -> Magic Caster Wand FX**.

If installing manually, copy this repository's `custom_components/magic_caster_wand_fluid` folder into Home Assistant's `/config/custom_components/magic_caster_wand_fluid` folder.

## Important Bluetooth Notes

- A Bluetooth proxy is strongly recommended for stable range and multiple wands.
- `bluetooth_proxy` should use `active: true`.
- Keep scan interval at the default unless you are troubleshooting.

Recommended ESPHome shape:

```yaml
esp32_ble_tracker:
  scan_parameters:
    active: true

bluetooth_proxy:
  active: true
```

## Fluid Canvas

Open the WebGL fluid page from a Home Assistant dashboard iframe:

```yaml
type: iframe
url: /magic_caster_wand_fluid/fluid
aspect_ratio: 75%
```

The canvas supports wand motion, multiple connected wands, mouse or finger casting, learn mode, a Spell Book, and clickable spell path previews where a path file is bundled.

## Draw Spell Sensor

The draw-only spell sensor uses the FX name on new installs:

```text
sensor.magic_caster_wand_fx_draw_spell
```

Existing Home Assistant entity registry entries may keep an older entity ID until Home Assistant reloads the integration. If an old draw spell entity remains, rename it in **Settings -> Devices & services -> Entities**, or remove the stale entity registry entry and reload the integration.

## Multi-Wand Automation Pattern

For multiple wands, put every wand's `Spell` sensor into the first trigger. Home Assistant will run the same automation no matter which wand casts the spell.

Notes for the example below:

- `sensor.magic_caster_wand_fx_1_spell`, `sensor.magic_caster_wand_fx_2_spell`, and the other wand entries are placeholders. Replace them with the real `Spell` entity IDs from your wand devices.
- `sensor.magic_caster_wand_fx_draw_spell` is the shared finger, mouse, and Spell Book draw sensor. It is separate so drawn spells can automate differently from physical wand spells.
- The trigger watches the `last_updated` attribute so casting the same spell twice in a row still fires the automation.
- Add more spell names to the template condition when you want the same automation to react to more spells.

```yaml
alias: Cast Wand And Draw Spells
description: Magic Caster Wand FX multi-wand spell test
mode: restart
triggers:
  - trigger: state
    entity_id:
      - sensor.magic_caster_wand_fx_1_spell
      - sensor.magic_caster_wand_fx_2_spell
      - sensor.magic_caster_wand_fx_3_spell
      - sensor.magic_caster_wand_fx_4_spell
    attribute: last_updated
    id: wand_spell

  - trigger: state
    entity_id:
      - sensor.magic_caster_wand_fx_draw_spell
    attribute: last_updated
    id: draw_spell

conditions:
  - condition: template
    value_template: >
      {{ trigger.to_state.state | lower in
      ['lumos', 'nox', 'draw_lumos', 'draw_nox'] }}

actions:
  - variables:
      spell: "{{ trigger.to_state.state | lower }}"
      source_entity: "{{ trigger.entity_id }}"

  - choose:
      - conditions: "{{ spell == 'lumos' }}"
        sequence:
          - action: light.turn_on
            target:
              entity_id: light.theater
            data:
              brightness_pct: 60

      - conditions: "{{ spell == 'draw_lumos' }}"
        sequence:
          - action: light.turn_on
            target:
              entity_id: light.theater
            data:
              brightness_pct: 100

      - conditions: "{{ spell in ['nox', 'draw_nox'] }}"
        sequence:
          - action: light.turn_off
            target:
              entity_id: light.theater
            data: {}
```

## Spell Book Gestures

These are the spells currently listed in the iframe Spell Book. Each entry has a bundled fluid path preview, so clicking a Spell Book card can draw that spell into the WebGL canvas.

<table>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Colovaria</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/colovaria.png" width="108" height="108" alt="Colovaria spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Expelliarmus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/expelliarmus.png" width="108" height="108" alt="Expelliarmus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Finite</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/finite.png" width="108" height="108" alt="Finite spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Lumos</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/lumos.png" width="108" height="108" alt="Lumos spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Lumos Maxima</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/lumos_maxima.png" width="108" height="108" alt="Lumos Maxima spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Nox</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/nox.png" width="108" height="108" alt="Nox spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Petrificus Totalus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/petrificus_totalus.png" width="108" height="108" alt="Petrificus Totalus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Stupefy</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/stupefy.png" width="108" height="108" alt="Stupefy spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Meteolojinx</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/meteolojinx.png" width="108" height="108" alt="Meteolojinx spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Expecto Patronum</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/expecto_patronum.png" width="108" height="108" alt="Expecto Patronum spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Immobulus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/immobulus.png" width="108" height="108" alt="Immobulus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Incendio</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/incendio.png" width="108" height="108" alt="Incendio spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Aguamenti</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/aguamenti.png" width="108" height="108" alt="Aguamenti spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Glacius</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/glacius.png" width="108" height="108" alt="Glacius spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Ascendio</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/ascendio.png" width="108" height="108" alt="Ascendio spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Protego</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/protego.png" width="108" height="108" alt="Protego spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Wingardium Leviosa</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/wingardium_leviosa.png" width="108" height="108" alt="Wingardium Leviosa spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Brachiabindo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/brachiabindo.png" width="108" height="108" alt="Brachiabindo spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Finestra</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/finestra.png" width="108" height="108" alt="Finestra spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Incarcerous</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/incarcerous.png" width="108" height="108" alt="Incarcerous spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Reparo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/reparo.png" width="108" height="108" alt="Reparo spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Bombarda</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/bombarda.png" width="108" height="108" alt="Bombarda spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Arania Exumai</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/arania_exumai.png" width="108" height="108" alt="Arania Exumai spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Reducto</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/reducto.png" width="108" height="108" alt="Reducto spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Ventus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/ventus.png" width="108" height="108" alt="Ventus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Fulgari</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/fulgari.png" width="108" height="108" alt="Fulgari spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Orchideous</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/orchideous.png" width="108" height="108" alt="Orchideous spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Expulso</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/expulso.png" width="108" height="108" alt="Expulso spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Alohomora</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/alohomora.png" width="108" height="108" alt="Alohomora spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Herbivicus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/herbivicus.png" width="108" height="108" alt="Herbivicus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Cantis</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/cantis.png" width="108" height="108" alt="Cantis spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Flagrate</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/flagrate.png" width="108" height="108" alt="Flagrate spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Salvio Hexia</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/salvio_hexia.png" width="108" height="108" alt="Salvio Hexia spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Verdimillious</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/verdimillious.png" width="108" height="108" alt="Verdimillious spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Vermillious</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/vermillious.png" width="108" height="108" alt="Vermillious spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Impedimenta</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/impedimenta.png" width="108" height="108" alt="Impedimenta spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Confundo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/confundo.png" width="108" height="108" alt="Confundo spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Confringo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/confringo.png" width="108" height="108" alt="Confringo spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Appare Vestigium</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/appare_vestigium.png" width="108" height="108" alt="Appare Vestigium spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Accio</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/accio.png" width="108" height="108" alt="Accio spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Riddikulus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/riddikulus.png" width="108" height="108" alt="Riddikulus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>The Force Spell</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_force_spell.png" width="108" height="108" alt="The Force Spell spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Piertotum Locomotor</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/piertotum_locomotor.png" width="108" height="108" alt="Piertotum Locomotor spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Scourgify</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/scourgify.png" width="108" height="108" alt="Scourgify spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Colloshoo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/colloshoo.png" width="108" height="108" alt="Colloshoo spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Pestis Incendium</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/pestis_incendium.png" width="108" height="108" alt="Pestis Incendium spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Flipendo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/flipendo.png" width="108" height="108" alt="Flipendo spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Locomotor</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/locomotor.png" width="108" height="108" alt="Locomotor spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Sonorus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/sonorus.png" width="108" height="108" alt="Sonorus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Depulso</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/depulso.png" width="108" height="108" alt="Depulso spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Everte Statum</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/everte_statum.png" width="108" height="108" alt="Everte Statum spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Descendo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/descendo.png" width="108" height="108" alt="Descendo spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>The Pepper-Breath Hex</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_pepper_breath_hex.png" width="108" height="108" alt="The Pepper-Breath Hex spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Langlock</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/langlock.png" width="108" height="108" alt="Langlock spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Spongify</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/spongify.png" width="108" height="108" alt="Spongify spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Aberto</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/aberto.png" width="108" height="108" alt="Aberto spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Anteoculatia</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/anteoculatia.png" width="108" height="108" alt="Anteoculatia spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Calvorio</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/calvorio.png" width="108" height="108" alt="Calvorio spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Colloportus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/colloportus.png" width="108" height="108" alt="Colloportus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Densaugeo</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/densaugeo.png" width="108" height="108" alt="Densaugeo spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Entomorphis</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/entomorphis.png" width="108" height="108" alt="Entomorphis spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Evanesco</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/evanesco.png" width="108" height="108" alt="Evanesco spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Melefors</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/melefors.png" width="108" height="108" alt="Melefors spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Mucus Ad Nauseum</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/mucus_ad_nauseum.png" width="108" height="108" alt="Mucus Ad Nauseum spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>Quietus</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/quietus.png" width="108" height="108" alt="Quietus spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Revelio</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/revelio.png" width="108" height="108" alt="Revelio spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Rictusempra</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/rictusempra.png" width="108" height="108" alt="Rictusempra spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Silencio</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/silencio.png" width="108" height="108" alt="Silencio spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>The Cheering Charm</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_cheering_charm.png" width="108" height="108" alt="The Cheering Charm spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>The Hair Thickening Growing Charm</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_hair_thickening_growing_charm.png" width="108" height="108" alt="The Hair Thickening Growing Charm spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>The Hour Reversal Charm</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_hour_reversal_charm.png" width="108" height="108" alt="The Hour Reversal Charm spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>The Sleeping Charm</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_sleeping_charm.png" width="108" height="108" alt="The Sleeping Charm spell gesture"/></td>
  </tr>
  <tr>
    <td align="center" valign="top" width="150" height="156"><b>The Stretching Jinx</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/the_stretching_jinx.png" width="108" height="108" alt="The Stretching Jinx spell gesture"/></td>
    <td align="center" valign="top" width="150" height="156"><b>Avada Kedavra</b><br/><img src="https://raw.githubusercontent.com/L-Milko/hass-magic-caster-wand-fx/main/docs/gesture_thumbnails/avada_kedavra.png" width="108" height="108" alt="Avada Kedavra spell gesture"/></td>
    <td width="150" height="156"></td>
    <td width="150" height="156"></td>
  </tr>
</table>

### Spell Changes

- Casting `lumos` once reports `lumos`. Casting `lumos` again as the next Lumos-family cast reports `lumos_maxima`, so automations can treat Lumos Maxima as its own spell.
- `avada_kedavra` is not a normal stock Magic Caster Wand spell. In this fork, the movement that originally came through as `sonorus` is mapped to `avada_kedavra` because the gesture looked much closer to the Killing Curse.
- The old model/developer label `the_hour_reversal_reversal_charm` is treated as `sonorus`, keeping Sonorus available after the Avada Kedavra swap.
- Drawn spells use the same naming idea with the `draw_` prefix, for example `draw_lumos`, `draw_lumos_maxima`, and `draw_avada_kedavra`.

## Spell Recognition

Wand spell recognition uses the [hass-tflite](https://github.com/ificator/hass-tflite) add-on/server.

1. Install `hass-tflite` from `https://github.com/ificator/hass-tflite`.
2. Open the hass-tflite web UI.
3. Upload your compatible `model.tflite` file.
4. In the Magic Caster Wand FX integration options, keep the default TFLite Server URL unless you installed the server somewhere custom.

Default local add-on URL:

```text
http://b5e3f765-tflite-server:8000
```

> The `model.tflite` file is not included publicly in this repository.

## References

- [Magic-Caster-Wand-Open-app-ai](https://github.com/whymaxwhy/Magic-Caster-Wand-Open-app-ai.git)
- [OpenCaster](https://github.com/Blues-Hailfire/OpenCaster.git)
- [Original hass-magic-caster-wand](https://github.com/eigger/hass-magic-caster-wand)
