# Maloja

[![](https://img.shields.io/github/v/tag/krateng/maloja?label=GitHub&style=for-the-badge)](https://github.com/krateng/maloja)
[![](https://img.shields.io/pypi/v/malojaserver?label=PyPI&style=for-the-badge)](https://pypi.org/project/malojaserver/)
[![](https://img.shields.io/docker/v/krateng/maloja?label=Docker&style=for-the-badge)](https://hub.docker.com/r/krateng/maloja)

[![](https://img.shields.io/pypi/l/malojaserver?style=for-the-badge)](https://github.com/krateng/maloja/blob/master/LICENSE)
[![](https://img.shields.io/codeclimate/maintainability/krateng/maloja?style=for-the-badge)](https://codeclimate.com/github/krateng/maloja)

Simple self-hosted music scrobble database to create personal listening statistics. No recommendations, no social network, no nonsense.

You can check [my own Maloja page](https://maloja.krateng.ch) to see what it looks like (it's down fairly often because I use it as staging environment, that doesn't reflect the stability of the Maloja software!).


> **IMPORTANT**: With the update 2.7, the backend has been reworked to use a password. With a normal installation, you are asked to provide a password on setup. If you use docker or skip the setup for other reasons, you need to provide the environment variable `MALOJA_FORCE_PASSWORD` on first startup.

> **IMPORTANT**: With the update 2.9, the API endpoints have changed. All old endpoints should be redirected properly, but I recommend updating your clients to use the new ones.

## Table of Contents
* [Features](#features)
* [How to install](#how-to-install)
	* [Requirements](#requirements)
	* [PyPI](#pypi)
	* [From Source](#from-source)
	* [Docker](#docker)
	* [Extras](#extras)
* [How to use](#how-to-use)
	* [Basic control](#basic-control)
	* [Data](#data)
	* [Customization](#customization)
* [How to scrobble](#how-to-scrobble)
	* [Native support](#native-support)
	* [Native API](#native-api)
	* [Standard-compliant API](#standard-compliant-api)
	* [Manual](#manual)
* [How to extend](#how-to-extend)

## Features

* **Self-hosted**: You will always be able to access your data in an easily-parseable format. Your library is not synced with any public or official music database, so you can follow your own tagging schema.
* **Associated Artists**: Compare different artists' popularity in your listening habits including subunits, collaboration projects or solo performances by their members. Change these associations at any time without losing any information.
* **Multi-Artist Tracks**: Some artists often collaborate with others or are listed under "featuring" in the track title. Instead of tracking each combination of artists, each individual artist competes in your charts.
* **Custom Images**: Don't rely on the community to select the best pictures for your favorite artists. Upload your own so that your start page looks like you want it to look.
* **Proxy Scrobble**: No need to fully commit or set up every client twice - you can configure your Maloja server to forward your scrobbles to other services.
* **Standard-compliant API**: Use existing, mature apps or extensions to scrobble to your Maloja server.
* **Manual Scrobbling**: Listening to vinyl or elevator background music? Simply submit a scrobble with the web interface.
* **Keep it Simple**: Unlike Last.fm and similar alternatives, Maloja doesn't have social networking, radios, recommendations or any other gimmicks. It's a tool to keep track of your listening habits over time - and nothing more.


## How to install

### Requirements

Maloja should run on any x86 or ARM machine that runs Python.

I can support you with issues best if you use **Alpine Linux**.

Your CPU should have a single core passmark score of at the very least 1500. 500 MB RAM should give you a decent experience, but performance will benefit greatly from up to 2 GB.

### PyPI

You can install Maloja with

```console
	pip install malojaserver
```

To make sure all dependencies are installed, you can also use one of the included scripts in the `install` folder.

### From Source

Clone this repository and enter the directory with

```console
	git clone https://github.com/krateng/maloja
	cd maloja
```

Then install all the requirements and build the package, e.g.:

```console
	sh ./install/install_dependencies_alpine.sh
	pip install -r requirements.txt
	pip install .
```

### Docker

Pull the [latest image](https://hub.docker.com/r/krateng/maloja) or check out the repository and use the included Dockerfile.

Of note are these settings which should be passed as environmental variables to the container:

* `MALOJA_DATA_DIRECTORY` -- Set the directory in the container where configuration folders/files should be located
  * Mount a [volume](https://docs.docker.com/engine/reference/builder/#volume) to the specified directory to access these files outside the container (and to make them persistent)
* `MALOJA_FORCE_PASSWORD` -- Set an admin password for maloja

You must publish a port on your host machine to bind to the container's web port (default 42010). The Docker version uses IPv4 per default.

An example of a minimum run configuration to access maloja via `localhost:42010`:

```console
	docker run -p 42010:42010 -v $PWD/malojadata:/mljdata -e MALOJA_DATA_DIRECTORY=/mljdata maloja
```

### Extras

* If you'd like to display images, you will need API keys for [Last.fm](https://www.last.fm/api/account/create) and [Spotify](https://developer.spotify.com/dashboard/applications). These are free of charge!

* Put your server behind a reverse proxy for SSL encryption. Make sure that you're proxying to the IPv6 or IPv4 address according to your settings.

* You can set up a cronjob to start your server on system boot, and potentially restart it on a regular basis:

```
@reboot sleep 15 && maloja start
42 0 7 * * maloja restart
```


## How to use

### Basic control

Start and stop the server in the background with

```console
	maloja start
	maloja stop
	maloja restart
```

If you need to run the server in the foreground, use

```console
	maloja run
```


### Data

If you would like to import your previous scrobbles, use the command `maloja import *filename*`. This works on:

* a Last.fm export generated by [benfoxall's website](https://benjaminbenben.com/lastfm-to-csv/) ([GitHub page](https://github.com/benfoxall/lastfm-to-csv))
* an official [Spotify data export file](https://www.spotify.com/us/account/privacy/)

To backup your data, run `maloja backup`, optional with `--include_images`.

### Customization

* Have a look at the [available settings](settings.md) and specifiy your choices in `/etc/maloja/settings.ini`. You can also set each of these settings as an environment variable with the prefix `MALOJA_` (e.g. `MALOJA_SKIP_SETUP`).

* If you have activated admin mode in your web interface, you can upload custom images for artists or tracks by simply dragging them onto the existing image on the artist or track page. You can also manage custom images directly in the file system - consult `images.info` in the `/var/lib/maloja/images` folder.

* To specify custom rules, consult the `rules.info` file in `/etc/maloja/rules`. You can also apply some predefined rules on the `/admin_setup` page of your server.


## How to scrobble

You can set up any amount of API keys in the file `authenticated_machines.tsv` in the `/etc/maloja/clients` folder. It is recommended to define a different API key for every scrobbler you use.

### Native support

These solutions allow you to directly setup scrobbling to your Maloja server:
* [Tauon](https://tauonmusicbox.rocks) Desktop Player
* [Web Scrobbler](https://github.com/web-scrobbler/web-scrobbler) Browser Extension
* [Multi Scrobbler](https://github.com/FoxxMD/multi-scrobbler) Desktop Application
* [Cmus-maloja-scrobbler](https://git.sr.ht/~xyank/cmus-maloja-scrobbler) Script
* [OngakuKiroku](https://github.com/Atelier-Shiori/OngakuKiroku) Desktop Application (Mac)
* [Maloja Scrobbler](https://chrome.google.com/webstore/detail/maloja-scrobbler/cfnbifdmgbnaalphodcbandoopgbfeeh) Chromium Extension (also included in the repository) for Plex Web, Spotify, Bandcamp, Soundcloud or Youtube Music

### Native API

If you want to implement your own method of scrobbling, it's very simple: You only need one POST request to `/apis/mlj_1/newscrobble` with the keys `artist`, `title` and `key` (and optionally `album`,`duration` (in seconds) and `time`(for cached scrobbles)) - either as form-data or json.

If you're the maintainer of a music player or server and would like to implement native Maloja scrobbling, feel free to reach out - I'll try my best to help. For Python applications, you can simply use the [`malojalib` package](https://pypi.org/project/maloja-lib/) for a consistent interface even with future updates.

### Standard-compliant API

You can use any third-party scrobbler that supports the audioscrobbler (GNUFM) or the ListenBrainz protocol. This is still somewhat experimental, but give it a try with these settings:

GNU FM | &nbsp;
------ | ---------
Gnukebox URL | Your Maloja URL followed by `/apis/audioscrobbler`
Username | Doesn't matter
Password | Any of your API keys

ListenBrainz | &nbsp;
------ | ---------
API URL | Your Maloja URL followed by `/apis/listenbrainz`
Username | Doesn't matter
Auth Token | Any of your API keys

Audioscrobbler v1.2 | &nbsp;
------ | ---------
Server URL | Your Maloja URL followed by `/apis/audioscrobbler_legacy`
Username | Doesn't matter
Password | Any of your API keys

Known working scrobblers:
* [Pano Scrobbler](https://github.com/kawaiiDango/pScrobbler) for Android
* [Simple Scrobbler](https://simple-last-fm-scrobbler.github.io) for Android
* [Airsonic Advanced](https://github.com/airsonic-advanced/airsonic-advanced) (requires you to supply the full endpoint (`yoururl.tld/apis/listenbrainz/1/submit-listens`))
* [Funkwhale](https://dev.funkwhale.audio/funkwhale/funkwhale) (use the legacy API `yoururl.tld/apis/audioscrobbler_legacy`)
* [mpdscribble](https://github.com/MusicPlayerDaemon/mpdscribble) (use the legacy API `yoururl.tld/apis/audioscrobbler_legacy`)

I'm thankful for any feedback whether other scrobblers work!



### Manual

If you can't automatically scrobble your music, you can always do it manually on the `/admin_manual` page of your Maloja server.


## How to extend

If you'd like to implement anything on top of Maloja, visit `/api_explorer`.
