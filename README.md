# Transport controls

A strip of media keys on one row: skip back, stop, play, pause, play/pause, skip forward and eject. Each key performs
the Home Assistant action you give it, so the same tile drives a media player, a Blu-ray player behind an IR remote, or
anything a script can reach.

Give a key an action and it appears; leave it empty and it is left out. The keys always keep this order and sit together
in the middle of the tile. The tile is two cells wide at least and grows as wide as the page, but never taller than one
row.

## Set up

1. In Tessera, open **Plugins**, choose **Transport controls** and tick your screens. Each screen builds once with the
   plugin.
2. In Layout, place the **Transport controls** tile from the library's Plugins group, two cells wide or more.
3. In the inspector, choose the entity the keys act on: a media player, a remote, a script and so on.
4. Type an action for each key you want, for example:

   | Key | Media player | IR remote (Broadlink and the like) |
   |---|---|---|
   | Skip back | `media_player.media_previous_track` | `remote.send_command command=PREVIOUS` |
   | Stop | `media_player.media_stop` | `remote.send_command command=STOP` |
   | Play | `media_player.media_play` | `remote.send_command command=PLAY` |
   | Pause | `media_player.media_pause` | `remote.send_command command=PAUSE` |
   | Play/pause | `media_player.media_play_pause` | `remote.send_command command=PLAY_PAUSE` |
   | Skip forward | `media_player.media_next_track` | `remote.send_command command=NEXT` |
   | Eject | (a media player has no eject) | `remote.send_command command=EJECT` |

5. Make sure the screen may perform Home Assistant actions. Tessera turns this on for a screen it added to Home
   Assistant; for a screen added by hand, tick **Allow the device to perform Home Assistant actions** in the ESPHome
   integration's options for that device.

## What a key's action can say

A key's text is the action, then optionally an entity of its own, then optionally one field, separated by spaces:

- `media_player.media_play`: the action, on the tile's entity.
- `remote.send_command command=EJECT`: the action with one field, on the tile's entity. The value runs to the end of
  the text, so it may contain spaces.
- `script.turn_on script.eject_disc`: the action on another entity than the tile's.
- `remote.send_command remote.hall command=up`: both.

A key whose text is not an action shows its icon in red, and a tap sends nothing. At most 64 characters per key, and
one field: for an action that needs more, write a script in Home Assistant and give the key `script.turn_on
script.<its name>`.

## On the screen

- A tap on a key lights it blue while the action goes out, or red when it could not be sent (no entity, no action, or
  the screen may not perform actions).
- With a media player as the tile's entity, the key of what it does is lit: play and play/pause while it plays, pause
  while it is paused. The icons go grey while the entity is unavailable.
- Works on every board, the CYD included. On a small screen with all seven keys in two cells, the icons get smaller;
  make the tile wider to give them more room.

## Good to know

- A key runs whatever action you type for it, on whatever entity you name. The manifest lists the actions the tile is
  made for (media player, remote, script, button, scene and switch actions), but the screen does not hold a key to that
  list: anyone who can edit the layout can make a key perform any action the screen is allowed to.
- Nothing leaves your home: the tile uses no internet service and no Home Assistant entity but the ones you choose.

## How it works

- `tessera-plugin.yaml`: one tile, `keys`, 2x1 to 9x1, that belongs to an entity of the domains it acts on, with one
  text option per key.
- `components/transport_controls/transport_command.h`: reads a key's text into an action, a target and a field.
- `components/transport_controls/transport_controls.cpp`: lays the keys out in the middle of the tile, finds the key
  under the finger (`tessera::tap_point`) and sends its action (`tessera::action`). The eject symbol is not in
  Tessera's icon set, so its key draws Material Design's eject itself.
