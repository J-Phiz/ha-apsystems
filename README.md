# Home Assistant Linky Add-on

[![Build Status](https://flat.badgen.net/github/checks/J-Phiz/ha-apsystems?label=build)](https://github.com/J-Phiz/ha-apsystems/actions/workflows/run.yml?query=branch%3Amaster)
[![Last Tag](https://flat.badgen.net/github/tag/J-Phiz/ha-apsystems?style=flat)](https://github.com/J-Phiz/ha-apsystems/tags)
[![Version](https://flat.badgen.net/github/release/J-Phiz/ha-apsystems?gradient=b65cff,11cbfa&style=flat&label=version)](https://github.com/J-Phiz/ha-apsystems/releases)

> A **Home Assistant** add-on to sync Energy dashboards with your **APSystems** solar panels.

---

**HA APSystems** is a Home Assistant add-on that synchronizes data from your APSystems solar panels with Home Assistant Energy dashboards.

It uses the OpenAPI service provided by APSystems [apsystems-faq](https://emea.apsystems.com/resources/installer-faq/#toggle-id-91).

<p align="center">
  <img src="https://raw.githubusercontent.com/J-Phiz/ha-apsystems/refs/heads/master/assets/ha-energy-result.png">
</p>

## Prerequisites

To use this add-on, you need:

- An APSystems solar system
- One or more ECU devices
- An APSystems user account
- The EMA App smartphone app

## Installation

- Click [here](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FJ-Phiz%2Fha-apsystems) to add the repository to Home Assistant.
  If the link does not work:
  - In Home Assistant, open the _Settings_ menu, then _Add-ons_
  - Click the _Store_ button in the bottom-right corner
  - Click the three dots in the top-right corner, then _Repositories_
  - Add `https://github.com/J-Phiz/ha-apsystems`
- Click _Add_ and close the dialog
- Search for _APSystems_ in the add-ons list (you can use the search bar)
- Install the add-on by clicking the install button

## Configuration

Once the add-on is installed, open the _Configuration_ tab.

The YAML configuration is divided into two parts:

- The first, `openapi`: defines the OpenAPI configuration
- The second, `meters`: defines each ECU configuration

### OpenAPI Section

You must fill in both required fields: `appId` and `secretId`.

You can find this information in the EMA App:

- Open the app and go to the _Settings_ menu
- Tap _OpenAPI Service_, then _Developer Authorization_
- Retrieve the values of _APP ID_ and _APP Secret_ and map them as follows:
  - `appId` = _APP ID_
  - `secretId` = _APP Secret_

### Meters Section

The base YAML configuration includes two ECUs:

- The first for your main residence
- The second for a secondary residence

You can remove the lines related to the secondary residence if you only have one ECU, or add more ECUs if needed.

For each ECU, fill in the following fields:

- `systemId`: Your APSystems system identifier.
  - Find it in the EMA App:
    - Go to _Settings_ → _Account Details_ → copy the value of _sid_.
- `ecuId`: Your ECU identifier.
  - It is printed on the label under your ECU device.
- `name`: The name that will appear in Home Assistant Energy dashboards (you can change it later).
- `action`: Keep the default value: `sync`.

> Example configuration for one ECU meter:

```yaml
- systemId: 'D18435095771264'
  ecuId: '482375945623'
  name: my home solar panels
  action: sync
```

## Usage / Operation

Once the add-on is started, open the _Log_ tab to monitor the synchronization progress.

At first launch, **HA APSystems** will try to retrieve up to **2 months** of historical data.

Then, it will synchronize data twice per day while running:

- Once between 6 AM and 7 AM to fetch data from the previous day
- Once between 9 AM and 10 AM in case the first sync failed

You can verify that the add-on is working properly by checking the _Log_ tab — all relevant information will be displayed there.

### Dashboards

To display **HA APSystems** data in your Home Assistant Energy dashboard:

- Click [here](https://my.home-assistant.io/redirect/config_energy/), or go to _Settings_ → _Dashboards_ → _Energy_
- Under the _Electricity grid_ section, click _Add solar production_
- Choose the statistic that matches the `name` you configured earlier
- Click _Save_

### Useful information

As you may have noticed when retrieving your OpenAPI IDs, the number of API requests per month is limited to 1000 (free version). Therefore:

- Daily data is **not available in real time** — it can only be retrieved the next day, between 6 AM and 10 AM.
- For recent dates (less than 7 days old), **HA APSystems** will try to get **hourly** data first.
- For older dates, it will only fetch **daily** data — in this case, you’ll see a single large bar around midnight when viewing that day’s detailed chart.

### Resetting data

If you encounter issues, you can reset all energy data created by **HA APSystems**.

#### First method (via the add-on)

Go back to the add-on’s _Configuration_ tab and change the `action` value to `reset` for the meter you want to reset.
Save and restart the add-on.

Then, check the _Log_ tab to confirm that the reset was successful.

At the next startup, if `action` is set back to `sync`, **HA APSystems** will re-import your historical data.

#### Second method (via HA Developer Tools)

- Go to _Developer Tools_ → _Actions_
- Choose the _recorder.purge_entities_ action
- Check _Entities to suppress_ and select the APSystems entity
- Click _Execute_

## Standalone installation

If your Home Assistant installation doesn’t support add-ons, you can also run HA APSystems using Docker.

> [!NOTE]
> This method is **not recommended** and should be considered a **fallback solution** for advanced users who prefer not to use HAOS and its add-on system.
> If you chose to install Home Assistant manually, you should already be comfortable with your setup.
> **No support** is provided for Docker, Kubernetes, or any system other than HAOS.

### Setup

Build a Docker image `ha-apsystems` suitable for your system using the following command:

```sh
docker build https://github.com/bokub/ha-apsystems.git -f standalone.Dockerfile -t ha-apsystems
```

Then create a file named `options.json` as shown below, and follow the “Configuration” section above to fill it in.

```json
{
  "openapi": {
    "appId": "",
    "appSecret": ""
  },
  "meters": [
    {
      "name": "APSystems main residence",
      "systemId": "",
      "ecuId": "",
      "action": "sync"
    },
    {
      "name": "APSystems second residence",
      "systemId": "",
      "ecuId": "",
      "action": "sync"
    }
  ]
}
```

Create a **long-lived access token** from your Home Assistant profile page (click your initials at the bottom of the sidebar).

### Running

You can now run the HA APSystems Docker image using either `docker run` or Docker Compose.

In both cases, replace:

- `<options-folder>` → the **folder** containing your `options.json`
- `<token>` → the long-lived access token you just created in Home Assistant
- `<ha-ip>` → the **IP address** of your Home Assistant instance (include port if necessary)
- `<timezone>` → your system timezone (e.g. `Europe/Paris`) if different from your system configuration

```sh
# docker run
docker run -e SUPERVISOR_TOKEN='<token>' -e WS_URL='ws://<ha-ip>/api/websocket' -e TZ='<timezone>' -v <options-folder>:/data ha-apsystems
```

```yml
# docker-compose.yml
services:
  ha-apsystems:
    image: ha-apsystems
    environment:
      - SUPERVISOR_TOKEN=<token>
      - WS_URL=ws://<ha-ip>/api/websocket
      - TZ=<timezone>
    volumes:
      - <options-folder>:/data
```
