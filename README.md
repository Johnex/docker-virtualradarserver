# docker-virtualradarserver

Dockerized version of [VirtualRadarServer](https://www.virtualradarserver.co.uk/) for Linux.

Current and latest release is on the `:dev` branch. Latest stable releases are at `v3pX` branches - according to the availabe pre-releases of VRS.

## Requirements
This container expects data input via network from a running instance of e.g. [readsb-protobuf](https://github.com/sdr-enthusiasts/docker-readsb-protobuf), [readsb bare-metal](https://github.com/wiedehopf/readsb) or [tar1090](https://github.com/sdr-enthusiasts/docker-tar1090) (as long it's attached to your SDR). Any other data sources _should_ work, as long tehy offer one of the [supported formats](https://www.virtualradarserver.co.uk/Documentation/WebServer/ReceiversOptions.aspx)

It will run on a Raspberry Pi, recommended is a Raspberry Pi 4 with at least 2GB RAM. As VRS is a bit of ressource hog, your results will depend very much on the amount of planes your receiver picks up (or you get from some other sources).

## example docker-compose.yml

```
version: '3.8'

services:

  vrs:
    image: ghcr.io/sdr-enthusiasts/vrs:dev
    tty: true
    container_name: vrs
    hostname: vrs
    restart: always
    ports:
        - 8085:8080
    environment:
        - VRS_ADMIN_USERNAME=your_webadmin_user
        - VRS_ADMIN_PASSWORD=your_webadmin_pass
        - VRS_CULTURE=de-DE
        #see http://msdn.microsoft.com/en-us/goglobal/bb896001.aspx for a list of supported culture names. Not all translations may be available
        - VRS_DB_UPDATE_POLICY_FULLAUTO=yes #default unset / no
        - VRS_DB_UPDATE_WITH_VACUUM=yes #default unset / no
        - VRS_DB_UPDATE_BACKUP_UNCOMPRESSED=yes #default unset / compressed
        - VRS_ENHANCED_MARKERS=normal #default unset
    tmpfs:
        - /tmp:rw,nosuid,nodev,noexec,relatime,size=128M
    volumes:
        - /opt/adsb/vrs:/root/.local/share/VirtualRadar
        - "/etc/localtime:/etc/localtime:ro"
        - "/etc/timezone:/etc/timezone:ro"
```

The final configuration is done via the VRS Web Admin Interface, which can be found at `http://<HOST_IP:8085/VirtualRadar/WebAdmin/Index.html`
After startup, VRS will point to `readsb` as it's first and only receiver - which will work out of the box if the container is in the same stack as `readsb`, which is the case if you followed [this guide](https://sdr-enthusiasts.gitbook.io/ads-b/).

On first startup the container downloads [some additional files](https://github.com/rikgale/VRSOperatorFlags), namely a pre-populated BaseStation.sqb, a [LocalAircraft.txt](https://github.com/rikgale/LocalAircraft) which corrects some incorrectly tagged (civil a/c flagged as military a/c or visa versa only) aircrafts and some nice custom Operator Flags and Silhouettes made by [rikgale](https://github.com/rikgale) and the group effort of [the community](https://discord.com/channels/734090820684349521/797799467880677377)
Everything is constantly updated and help is much appreciated. Any requests for operator flags / silhouettes can be filed through the [issues](https://github.com/rikgale/VRSOperatorFlags/issues).

Subsequent updates of the files occur on a container-restart. Additionally, the originating repository is checked every 6 hours for updates. New files will be downloaded automatically.

Requests for additions to the LocalAircraft.txt go [Here](https://github.com/rikgale/LocalAircraft/issues)

## Scope of supply

The container image comes with the following preinstalled VRS V3 [plugins](https://www.virtualradarserver.co.uk/Download.aspx#panel-web-admin):

* Language Pack
* Custom Content
* Database Editor
* Database Writer
* Feed Filter
* SQL Server
* Tile Server Cache
* Web Admin

(batteries are not included)

## Environment variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VRS_DB_UPDATE_POLICY_FULLAUTO` | Activates the initial download of an extended Basestation.sqb DB and keeps it up to date. Warning! This may overwrite user entries | `unset` |
| `VRS_DB_UPDATE_WITH_VACUUM` | Works only in conjunction with `VRS_DB_UPDATE_POLICY_FULLAUTO`. DB is compressed after update. The update takes longer but the sqb will be smaller.| `unset` |
| `VRS_DB_UPDATE_BACKUP_UNCOMPRESSED` | Works only in conjunction with `VRS_DB_UPDATE_POLICY_FULLAUTO`. Prior the update a backup of the database is created. If this is set, the backup will remain uncompressed | `unset` |
| `VRS_CULTURE` | see [here](http://msdn.microsoft.com/en-us/goglobal/bb896001.aspx) for a list of supported culture names. Not all translations may be available | `unset` means `en-GB` |
| `VRS_ENHANCED_MARKERS` | Installs and activates VRS custom markers. `normal` is for the ADS-B set, `hfdl` is for an extra set in case you have a HFDL input and `disable` deactivates the markers. [here](https://github.com/rikgale/VRSCustomMarkers) are more details about the markers. | `unset` |

If not stated otherwise, the envvars should be set to "yes" or "true" (or actually anything, as long they are set.)

## Something does not work, is broken or missing

Well, that happens. Drop by our [Discord](https://discord.com/channels/734090820684349521/797799467880677377) and we'll see what we can do.

## Disclaimer and acknowledgements

**IMPORTANT NOTICE:** Additional map overlays used in this container may contain aviation airscape, weather, navigational aids, airfield information, etc. All of these are purely for entertainment / hobby use and MUST NEVER be relied upon to any extent for real world aviation / flying. You should always use the latest legal airspace charts and weather information as published by official sources. This is not an official source. By using this code you agree to make users of your webpage aware of this, and agree that the publisher is not responsible for any loss or damage resulting from the use of this code. The airspace charts, navigational aids,aerodrome information, weather, etc will not be 100% complete or accurate and will contain errors.

**THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.**

SDR Enthusiasts are very grateful for the following resources which have been used in the creation of this container:

* Virtual Radar Server (Andrew Whewell) [VRS on GitHub](https://github.com/vradarserver/vrs) / [VRS Website](http://www.virtualradarserver.co.uk/)
* Database snapshot original data provide from PlaneBase, [mictronics aircraft database](https://www.mictronics.de/aircraft-database/export.php), VRS Standing Data and various user additions.
* Docker container base image, original container and other docker related fun-times ([SDR Enthusiasts](https://github.com/sdr-enthusiasts))
* Container maintainer (Rhodan76)
* [Sideviews](https://github.com/rikgale/VRSOperatorFlags) and [Silhoeuttes](https://github.com/rikgale/VRSOperatorFlags) (rikgale)
* [Additonal Aircraft Markers](https://github.com/rikgale/VRSCustomMarkers) (rikgale)
* [Map overlays](https://github.com/rikgale/VRSCustomLayers) (rikgale)
* [LocalAircraft.txt](https://github.com/rikgale/LocalAircraft) - (rikgale / Jonboy1081)
