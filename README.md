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

For multiple wands, put all wand spell sensors into the same trigger and branch from `trigger.to_state.state`. Add your real wand entity IDs to the list.

```yaml
alias: Cast Wand And Draw Spells
description: Magic Caster Wand FX multi-wand spell test
mode: restart
trigger:
  - platform: state
    entity_id:
      - sensor.magic_caster_wand_fx_1_spell
      - sensor.magic_caster_wand_fx_2_spell
      - sensor.magic_caster_wand_fx_3_spell
      - sensor.magic_caster_wand_fx_4_spell
    to:
      - lumos
      - nox
    id: wand_spell
  - platform: state
    entity_id:
      - sensor.magic_caster_wand_fx_draw_spell
    to:
      - draw_lumos
      - draw_nox
    id: draw_spell
action:
  - variables:
      spell: "{{ trigger.to_state.state }}"
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

Tip: each connected wand has its own `Spell` sensor. The draw/finger sensor is separate so you can automate `draw_lumos` differently from wand `lumos`.

## Spell Book Gestures

These are the spells currently listed in the iframe Spell Book. Cards marked with a fluid path preview can draw into the WebGL canvas when clicked.

<table>
  <tr>
    <td align="center"><b>Colovaria</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/colovaria.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Expelliarmus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/expelliarmus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Finite</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/finite.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Lumos</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/lumos.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Lumos Maxima</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/lumos_maxima.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Nox</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/nox.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Petrificus Totalus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/petrificus_totalus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Stupefy</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/stupefy.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Meteolojinx</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/meteolojinx.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Expecto Patronum</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/expecto_patronum.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Immobulus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/immobulus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Incendio</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/incendio.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Aguamenti</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/aguamenti.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Glacius</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/glacius.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Ascendio</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/ascendio.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Protego</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/protego.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Wingardium Leviosa</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/wingardium_leviosa.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Brachiabindo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/brachiabindo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Finestra</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/finestra.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Incarcerous</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/incarcerous.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Reparo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/reparo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Bombarda</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/bombarda.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Arania Exumai</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/arania_exumai.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Reducto</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/reducto.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Ventus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/ventus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Fulgari</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/fulgari.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Orchideous</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/orchideous.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Expulso</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/expulso.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Alohomora</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/alohomora.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Herbivicus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/herbivicus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Cantis</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/cantis.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Flagrate</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/flagrate.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Salvio Hexia</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/salvio_hexia.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Verdimillious</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/verdimillious.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Vermillious</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/vermillious.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Impedimenta</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/impedimenta.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Confundo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/confundo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Confringo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/confringo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Appare Vestigium</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/appare_vestigium.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Accio</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/accio.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Riddikulus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/riddikulus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>The Force Spell</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_force_spell.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Piertotum Locomotor</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/piertotum_locomotor.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Scourgify</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/scourgify.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Colloshoo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/colloshoo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Pestis Incendium</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/pestis_incendium.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Flipendo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/flipendo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Locomotor</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/locomotor.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Sonorus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/sonorus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Depulso</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/depulso.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Everte Statum</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/everte_statum.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Descendo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/descendo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>The Pepper-Breath Hex</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_pepper_breath_hex.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Langlock</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/langlock.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Spongify</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/spongify.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Aberto</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/aberto.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Anteoculatia</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/anteoculatia.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Calvorio</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/calvorio.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Colloportus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/colloportus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Densaugeo</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/densaugeo.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Entomorphis</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/entomorphis.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Evanesco</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/evanesco.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Melefors</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/melefors.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Mucus Ad Nauseum</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/mucus_ad_nauseum.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>Quietus</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/quietus.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Revelio</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/revelio.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Rictusempra</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/rictusempra.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Silencio</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/silencio.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>The Cheering Charm</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_cheering_charm.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>The Hair Thickening Growing Charm</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_hair_thickening_growing_charm.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>The Hour Reversal Charm</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_hour_reversal_charm.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>The Sleeping Charm</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_sleeping_charm.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
  </tr>
  <tr>
    <td align="center"><b>The Stretching Jinx</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/the_stretching_jinx.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td align="center"><b>Avada Kedavra</b><br/><img src="custom_components/magic_caster_wand_fluid/frontend/fluid/gestures/avada_kedavra.png" width="100"/><br/><sub>Fluid path preview included</sub></td>
    <td></td>
    <td></td>
  </tr>
</table>

## Spell Recognition

Install the `hass-tflite` add-on and upload your compatible `model.tflite` through its web UI. Configure the TFLite server URL from the integration options if needed.

> The `model.tflite` file is not included publicly in this repository.

## References

- [Magic-Caster-Wand-Open-app-ai](https://github.com/whymaxwhy/Magic-Caster-Wand-Open-app-ai.git)
- [OpenCaster](https://github.com/Blues-Hailfire/OpenCaster.git)
- [Original hass-magic-caster-wand](https://github.com/eigger/hass-magic-caster-wand)
