Ambient-To-CWOP
==========================

Heroku deployment-ready Python script for sending data packets to the [Citizen Observer Weather Program](https://www.weather.gov/media/epz/mesonet/CWOP-OfficialGuide.pdf) (CWOP) from an Ambient Weather personal weather station. This script utilizes Ambient Weather Network's public API. Once configured, this script will send a properly formatted CWOP packet automatically every 10 minutes (or longer). While this script can run on any device that supports Python, these instructions are for hosting it on a free Heroku dyno.

### 20 FEB 2023 Update

Ambient-To-CWOP project has reached an end-of-life cycle and is being archives. Should you need a simple replacement with more functions and no hosting requirements, take a look at https://ambientcwop.com. 

## Setup, Install, and Deployment

### Requirements

- An Ambient Weather personal weather station (PWS) configured to report to the [Ambient Weather Network](https://ambientweather.net)
- A [Heroku](https://heroku.com) account
- A Github account (unless running Git locally or using Heroku Git/CLI)
- A [CWOP](http://www.wxqa.com) weather station registration/callsign

### Fork or Clone

Fork or clone ambient-to-cwop repository so that you can edit the variables for your PWS and CWOP callsign.

### Update PWS Variables In ambient-to-cwop.py

Go to your Ambient Weather Network account and generate a new API key and a new application key ([Follow these instructions](https://ambientweather.docs.apiary.io/#introduction/authentication)).

Add your keys, along with CWOP site info to ambient-to-cwop.py:

```bash
os.environ["AMBIENT_API_KEY"] = '###' #Enter your Ambient API Key
os.environ["AMBIENT_APPLICATION_KEY"] = '###' #Enter your Ambient App Keu

callsign = '######' #Enter your CWOP Callsign

latitude = ##.#### #Enter Latitude - Lat must be listed in DECIMAL DEGREES (DD.DDDD). Number of digits doesn't really matter. Use positive values for N/E, negative for S/W. The program then converts to degrees decimal minutes (DD MM.MMMM), which is the format APRS requires.

longitude = ##.#### #Enter Longitude - Long must be listed in DECIMAL DEGREES (DD.DDDD). Number of digits doesn't really matter. Use positive values for N/E, negative for S/W. The program then converts to degrees decimal minutes (DD MM.MMMM), which is the format APRS requires.

devicename = '###' #This identifies your equipment/software. You can put anything you want. Example: 'WS2902A'.
```

### Create New Heroku Dyno/App

Create a new dyno/app in your Heroku account. The free tier works fine. Next, go to your dyno's settings and add the PYTHON buildpack. Also in settings, add a CONFIG VARS for your PWS timezone (a list of timezones can be found [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)):

```bash
-- CONFIG VARS --

KEY   VALUE
TZ    America\New_York
```

Comment: While Procfile / runtime.txt / requirements.txt should setup everything when you first deploy, if configuring manually from the Heroku command line, the dyno is configured as a worker with the following command:

```bash
worker: python ambient-to-cwop.py
```

### Link Github to Heroku App

In your Heroku dyno, go to Deploy and link your Github account. Then connect to your forked repository.

### Deploy

In your Heroku dyno, select Deploy Branch under Manual deploy.

Comment: I use manual instead of automatic so that after you confirm the script is working, you can remove your site info and keys from the Github source and commit without it overwriting the Heroko dyno. I'm sure this can be done cleaner by pushing code out locally, but for now, this is the method I used with the shortest learning curve for an end user to get up and running.

### Setup Heroku Scheduler

In your Heroku dyno, go to Resources and search for the "Heroku Scheduler" Add-on. Add the Scheduler to your dyno and configure with the following settings:

```bash
-- Interval --
Every 10 minutes (or longer if desired)

-- Run Command --
python ambient-to-cwop.py
```

Be sure to save your job!

### Test findu.com Response

Your script should now start to run automatically. After your initial setup, it may take up to 10 minutes for the first packet to be reported. Go to your FindU.com station page and check to see when it was last updated. If more than 10 minutes has passed, you should check your keys and station settings, deploy again, and see if updates.

## Additional Comments & Resources

Most of the issues I personally had getting started were related to improperly formatted values (mainly my Lat/Long). Check those first, along with your keys, before digging too deep. Also, be sure to commit your changes before deploying!

While this setup intends to avoid using the Heroku Command Line Interface (CLI) and can be configured completely via a web browser, if you run into repeated issues, I advise you to move to the CLI for troubleshooting. Most useful is that you can manually trigger the script from the CLI and check for failure without having to wait until the next Scheduler run.

Lastly, this is my first script repository on Github. It was forked from the original Ambient_API repo and hacked together from there with spaghetti code to run on a free Heroku dyno. Yes there is still work to do, primarily getting the hourly rain report working, but for now, it should hopefully help the novice begin to report from their PWS to CWOP.

Sources:

* [Ambient Weather Network API](https://ambientweather.docs.apiary.io/#) - Ambient Weather Network's API documentation.
* [Ambient_API Library](https://github.com/avryhof/ambient_api) - Original repository for the interface library.
* [EWQCB's CWOP Script](http://www.wxforum.net/index.php?topic=36181.0) - Original script for posting formatted packets to CWOP using Ambient_API.
* [Ambient APRS](https://github.com/avryhof/ambient_aprs) - An Ambient_API wrapper from the library's developer.
