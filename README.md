# Tracker map for [Home Assistant](https://home-assistant.io)
Tracking and mapping a device using Home Assistant, Google Sheets, Google Maps Api, and IFTTT
```diff
+ Displays the history of a Home Assistant device tracker on a Google map. Uses Google Sheets as data storage.<br> Similar to [Google Maps Timeline feature](https://www.google.com/maps/timeline).
```

<img align="left" src="https://i.imgur.com/E6yZfuf.png" height="350">

***

## Features
* Date filters: day, period;
* Markers: on/off;
* Polylines: on/off;
* Heatmap: +/- radius, +/- intensity, opacity, gradient color;
* Code can be easily customized to add/remove features (e.g. add filter for multiple devices);
* Can be used as a [*panel iframe*](https://home-assistant.io/components/panel_iframe) or as a [*custom state card iframe*](https://github.com/covrig/homeassistant-iframe-card);
* Disabled more-info card.
***
KNOWN PROBLEMS: the code is a bit of a mess - could use a cleanup. <br>I can't show all features in a picture (confidentiality issues).

LIMITATIONS: Can display more devices at the same time, but will use the same color.
***
## Installation - a bit complicated :)
* Download `/www/trackermap.html` to `<your-hass-configuration-dir>/www/` 
<br>(create the folder structure if you don't have it - mind the permissions)
* Get an API key for Google Maps. Edit the `/www/trackermap.html` file: replace `*YOURAPIKEY*` with your API key.
<br> [*Get an API key from Google here*](https://developers.google.com/maps/documentation/javascript/get-api-key)
* Download `/www/js/d3.v4.min.js` to `<your-hass-configuration-dir>/www/js` if you plan to host the script yourself. For this use `<script src="js/d3.v4.min.js"></script>`. By default I am using the hosted version from [*D3.js*](https://d3js.org))
<br>I am using this to read and filter the CSV file from Google Drive.
* Download `/www/pin.png` to `<your-hass-configuration-dir>/www/` 
<br>This will be your marker. [Flaticon](https://www.flaticon.com/) is a great place to get new markers. 
<br>You can also use an URL for this. In the html replace `pin.png` with a URL pointing to your (preferably png) marker in your code.

* Add it to your HASS configuration. There are two posibilities: as a [*panel iframe*](https://home-assistant.io/components/panel_iframe) or as a [*custom state card iframe*](https://github.com/covrig/homeassistant-iframe-card). 
<br>You will find detailed instructions for the iframe state card in the link.<br>
The URL you need to use should be similar to: **http://yourhostorIP:8123/local/trackermap.html**.
* If you don't have it already, enable IFTTT in your configuration. All the instructions [**here**](https://home-assistant.io/components/ifttt/).
* Connect to your IFTTT account the *[Webhooks](https://ifttt.com/maker_webhooks)* and *[Google Sheets](https://ifttt.com/google_sheets)*
<br>Home Assistant will use Webhooks to send data to a Google sheet via IFTTT. An automation will trigger this.
* Create an applet in IFTTT to transfer the data from HASS to Google Sheets.
<br>**this** (trigger) should be Webhooks with the event name **LatLong**
<br> *that** (action service) should be Google Sheets, with the action "Add row to spreadsheet"; You can choose your own spreadsheet name and drive folder path (remember both), however the **formatted row** should be `{{OccurredAt}} ||| {{Value1}} |||{{Value2}} ||| {{Value3}}`
* Create a new automation in HASS (when the state of your tracked device changes send data to Google sheets via IFTTT). You can enrich the automation as you wish.
```yaml
 - alias: Store Location GoogleDrive
   trigger:
     platform: state
     entity_id: device_tracker.88888888888
   action:
    - service: ifttt.trigger
      data_template: {"event": "LatLong", "value1": "{{now().year}}-{{now().strftime('%m')}}-{{now().strftime('%d')}}", "value2": "{{ states.device_tracker['88888888888'].attributes.latitude }}", "value3": "{{ states.device_tracker['88888888888'].attributes.longitude }}"}
```
* At this point you can restart HASS. If the automation wasn't triggered already, trigger it manually.
* Check you Google Drive. After triggering the automation you should find your new Google Sheets file in the folder you specified. You can also search of it (the name you specified in IFTTT).
* Replace the first row of the file with the headers below. You can write over the existing data.
             <img src="https://i.imgur.com/qFc3lw5.jpg" width="450">
* Publish the file as CSV: `File/Publish to thw web` -> `Sheet1` -> `CSV`. In the `Published content and & settings` (drop down, same screen) you should have the `Automatically republish when changes are made` checked.
<br> This will allow Home Assistant to read your file. *Copy the link.*
* Paste the link you copied above in your `/www/trackermap.html` file. Replace `MyGoogleSheetLink` with your link.<br>(`var URL = 'MyGoogleSheetLink';`)
* All done! It might take a few minutes for HASS to register the points.<br> You might need to clear your browser cache or restart HASS.


## Changelog
```
Version 20180211:
Start.
```

