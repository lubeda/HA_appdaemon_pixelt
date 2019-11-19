# appdaemon_pixelt
There is a marvelous project [PixelIT](https://www.bastelbunker.de/pixel-it/) which is a 8x32bit RGB LED matrix to display text and graphics. The screen is more or less dumb, you have to feed it with data by a server.
The original developer provides a node-red flow to do so.

## Homeassistant
I did not manage to feed data via node-red to the display so i started to develop an appdaemon class for this task. The result is in an early stage but is working

### concept

The appdaemon show a "screen" after another. A screen is assembled from an json [template](https://wiki.dietru.de/books/pixel-it/page/apiscreen) to set static information like colors and a "dynamic" part the message. E.g. you can design a template for temperature with a special icon, text color and display duration and feed the actual value via home-assistant. Screennames should be unique if you update data dynamicaly.

### setup

Install the class to  your appdaemon an configure it:

```yaml
pixelit:
  module: pixelit
  class: pixelIT
  ip: 192.168.178.24
  path: "/config/appdaemon/apps/pixelit/"
  entitiy_id: sensor.pixelit
```
parameter | meaning
----------|----------
ip|ip of the pixel controller
path|path to the json templates
entity_id| entity in HA to update with information about the display

configuration.yaml
```
sensor:
- platform: rest
  resource: http://192.168.178.47:5050/api/appdaemon/pixelit_sensor
  method: POST
  headers:
    Content-Type: "application/json"
  name: Pixelit
  value_template: > 
      {{ value_json.state }}
  scan_interval: 300

notify:
  - name: pixelit
    platform: rest
    method: POST_JSON
    headers:
      Content-Type: "application/json"
    resource: http://192.168.178.47:5050/api/appdaemon/pixelit_add
    title_param_name: "title"

rest_command:
  pixelit_delete:
    url: http://192.168.178.47:5050/api/appdaemon/pixelit_delete
    method: POST
    payload: '{"title": "{{ title }}"}'
    content_type:  'application/json; charset=utf-8'

  pixelit_update:
    url: http://192.168.178.47:5050/api/appdaemon/pixelit_update
    method: POST
    payload: '{"title": "{{ title }}","message": "{{ message }}"}'
    content_type:  'application/json; charset=utf-8'
```
**user your own ip instead of 192...47!!!**

### usage

create the templates according to the wiki of pixelit and add these two values:
```json
    "repeat": 10,
    "seconds": 15,
 ```
parameter | meaning
----------|----------
repeat|how often will this screen be shown
seconds|how long is this screen visible
 
name the json-files according to your screen an put it in the defined path.

#### To add a screen use the service:

*service:* notify.pixelit
with e.g.
```
title: "alarm"
message: "Door is open!"
```
parameter | meaning
----------|----------
title|selects the "*title*.json" template and equals the screen name
message|the text to display

#### To delete a screen from the playlist

*service:* rest_command.pixelit_delete

parameter | meaning
----------|----------
title|selects screen to delete first match is deleted, so unique screen names are useful

#### To update a screen message text

*service:* rest_command.pixelit_update

parameter | meaning
----------|----------
title|selects screen to update first match is used, so unique screen names are useful
message: new display text


#### To update the color of a screen message text

*service:* rest_command.pixelit_color

parameter | meaning
----------|----------
title|selects screen to update first match is used, so unique screen names are useful
r: red value 0..255
g: green value 0..255
b: blue value 0..255

#### special screens

The screen **alert** is special, it is display everysecond screen e.g. *screen1, alert, screen2, alert, screen3*
you have to delete ist to disable this mode.

The second special screen is "clock" here is a clock displayed without a message-text.

