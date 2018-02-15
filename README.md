# Tracker map for [Home Assistant](https://home-assistant.io)
Tracking and mapping a device using Home Assistant, Google Sheets, Google Maps Api, and IFTTT. <br>Similar to [Google Maps Timeline feature](https://www.google.com/maps/timeline).
```diff
+ Displays the history of a Home Assistant device tracker on a Google map. Uses Google Sheets as data storage.
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
* Disabled more-info card, auto resize/recenter map...
***
KNOWN PROBLEMS: <br>Browser compatibility issues: the date picker could default to a text picker. <br>The code is a bit of a mess - could use a cleanup. <br>I can't show all features in a picture (confidentiality issues). <br> I am aware, that all this could be done a lot easier with a small python script and a local file. I prepared this as a small training project that goes through a lot of elements.

LIMITATIONS: Can display more devices at the same time, but will use the same color.
***
## Installation - a bit complicated :)
* Download `/www/trackermap.html` to `<your-hass-configuration-dir>/www/` 
<br>(create the folder structure if you don't have it - mind the permissions)
* Get an API key for Google Maps. Edit the `/www/trackermap.html` file: replace `*YOURAPIKEY*` with your API key.
<br> [*Get an API key from Google here*](https://developers.google.com/maps/documentation/javascript/get-api-key)
* Add it to your HASS configuration. There are two posibilities: as a [*panel iframe*](https://home-assistant.io/components/panel_iframe) or as a [*custom state card iframe*](https://github.com/covrig/homeassistant-iframe-card). 
<br>You will find detailed instructions for the iframe state card in the link.<br>
The URL you need to use should be similar to: **http://yourhostorIP:8123/local/trackermap.html**.
* If you don't have it already, enable IFTTT in your configuration. All the instructions [**here**](https://home-assistant.io/components/ifttt/).
* Connect to your IFTTT account the *[Webhooks](https://ifttt.com/maker_webhooks)* and *[Google Sheets](https://ifttt.com/google_sheets)*
<br>Home Assistant will use Webhooks to send data to a Google sheet via IFTTT. An automation will trigger this.
* Create an applet in IFTTT to transfer the data from HASS to Google Sheets.
<br>**this** (trigger) should be Webhooks with the event name **LatLong**
<br> **that** (action service) should be Google Sheets, with the action "Add row to spreadsheet"; you can choose your own spreadsheet name and drive folder path (remember both), however the **formatted row** should be `{{OccurredAt}} ||| {{Value1}} |||{{Value2}} ||| {{Value3}}`
* Create a new automation in HASS (when the state of your tracked device changes send data to Google sheets via IFTTT). You can enrich the automation as you wish (e.g. don't send data over the night).
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
* Check you Google Drive. After triggering the automation you should find your new Google Sheets file in the folder you specified. You can also search it by name (the name you specified in IFTTT). Do not add anything to it. Let's say the name of this sheet is "Source" (proceed to next bullet).
* For some reason there is a 2000 row/per sheet IFTTT limit (the 2001 data point creates a new sheet). <br>To solve this, create a new Google Sheet file in the same folder (better keep both in the same folder). Let's say the name of this sheet is "Archive".
* Open the "Source" file and go to `Tools\Script Editor`. Replace everything with the following function that moves the data from "Source" to "Archive" (each sheet Id can be found in the URL e.g. `1ytJkkDIs7alHLcAbDwhnRoul-ltFiof6JvhORxgEdWc`).
```javascript
function onChange(event) {

  var source = SpreadsheetApp.openById('IdOfSource');
  var archive = SpreadsheetApp.openById('IdOfArchive');
  
  var sourceSheet = source.getSheetByName('Sheet1');
  var destSheet = archive.getSheetByName('Sheet1');    
  var sourceData = sourceSheet.getRange('A1:D1').getValues();  
  
  destSheet.getRange(destSheet.getLastRow()+1,1,sourceData.length,sourceData[0].length).setValues(sourceData);   
  sourceSheet.getRange(1, 1, sourceSheet.getLastRow(), sourceSheet.getLastColumn()).clear({contentsOnly: true}); 
}

```
* Create a trigger for the function above. In the script editor go to `Edit\Current project's trigger`, give the script a name, then click on the link pointing you to create a new trigger. Select the `onChange` function and change from time based (you can also use time) to `From spreadsheet + On Change`.
* Save evertyhing. You will be asked to authorize your scropt with your account. Proceed to do that.
* Replace the first row of the Archive file with the headers below. You can write over the existing data.
  <img src="https://i.imgur.com/qFc3lw5.jpg" width="450">
* Something to consider. The *Date* rows should be in the format *2018-02-11* (the order might be different depending on your locale). If you see just a number in the column change its format to match (*[see here how](https://i.imgur.com/d8SpBFf.png)*). To check your date format open the developer console (F12) on the tab the map is running and press the "Today" button on the map.
* Publish the "Archive" file as CSV: `File/Publish to thw web` -> `Sheet1` -> `CSV`. In the `Published content and & settings` (drop down, same screen) you should have the `Automatically republish when changes are made` checked.
<br> This will allow Home Assistant to read your file. *Copy the Archive file link.*
* Paste the link you copied above in your `/www/trackermap.html` file. Replace `MyGoogleSheetLink` with your link.<br>(`var URL = 'MyGoogleSheetLink';`)
* **All done!** It might take a few minutes for HASS to register the points.<br> You might need to clear your browser cache or restart HASS.
* If the map does not show in the frontend try to paste the html file contents to [JSBIN](http://jsbin.com/?html,output). If it works there your problem is with the HASS configuration.

## Additional information + self hosting icon and d3.js script
* Download `/www/js/d3.v4.min.js` to `<your-hass-configuration-dir>/www/js` if you plan to host the script yourself. For this use `<script src="js/d3.v4.min.js"></script>`. <br>By default I am using the hosted version from [*D3.js*](https://d3js.org).
<br>I am using this to read and filter the CSV file from Google Drive.
* Download `/www/pin.png` to `<your-hass-configuration-dir>/www/`  if you plan to host the marker picture yourself.
<br>This will be your marker. [Flaticon](https://www.flaticon.com/) is a great place to get new markers. 
<br>In the html replace `https://maps.gstatic.com/mapfiles/ridefinder-images/mm_20_red.png` (2 locations) with `pin.png` (or any other name you choose).

## Changelog
```diff
Version 20180215:
+Added extra information on how to bypass the 2000 rows per Google Sheet IFTTT limit;
+Fixed time zone problem;
```
