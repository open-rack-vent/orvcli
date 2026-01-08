# Open Rack Vent CLI (`orvcli`)

![](./art.jpeg)

Designed to run on embedded linux boards like the Beaglebone Black, the `orvcli` is a python project
that surfaces control of intake/exhaust fans, temperature sensors and other IO to manage the
environment inside a server rack. These IO can then be controlled by a standalone web API or by 
homeassistant via MQTT.

Please check out the [blog post](https://esologic.com/open-rack-vent) for more about this project
including an open source PCB, 3D printable parts and a ton of background and build photos.

## Usage

The file `orvcli.py` contains the command line interfaces that make up this application:

```
(.venv) devon@ESO-3-DEV-VM:~/Documents/projects/open_rack_vent$ python orvcli.py --help
Usage: orvcli.py [OPTIONS] COMMAND [ARGS]...

  Programs to manage the airflow inside a server rack.

Options:
  --help  Show this message and exit.

Commands:
  render-systemd  Creates a systemd unit that will start the run at boot.
  run             Main program to actually drive the fans
```

The main one `run` configures the output on the single board computer (bbb) to be able to drive the
attached PCB. The `render-systemd` command creates a systemd unit that can be installed to run
a given `run` config at boot. Here are the help sections for those two commands:

```
(.venv) devon@ESO-3-DEV-VM:~/Documents/projects/open_rack_vent$ python orvcli.py run --help
Usage: orvcli.py run [OPTIONS]

  Main air management program. Controls fans, reads sensors.

Options:
  --platform TEXT             The type of hardware running this application.
                              Options below. Either provide index or value:
                                 0: BeagleBoneBlack  [env var: ORV_PLATFORM; default: BeagleBoneBlack]
  --pcb-revision TEXT         The revision of the board driving the fans etc.
                              Options below. Either provide index or value:
                                 0: v1.0.0  [env var: ORV_PCB_REVISION; default: v1.0.0]
  --wire-mapping-json TEXT    JSON payload string with keys:
                                 • version: Enum[1]
                                 • fans: Dict[Enum[INTAKE_LOWER, INTAKE_UPPER, EXHAUST_LOWER, EXHAUST_UPPER], List[Enum[ONBOARD, PN1, PN2, PN3, PN4, PN5]]]
                                 • thermistors: Dict[Enum[INTAKE_LOWER, INTAKE_UPPER, EXHAUST_LOWER, EXHAUST_UPPER], List[Enum[TMP0, TMP1, TMP2, TMP3, TMP4, TMP5, TMP6]]]  [env var: ORV_WIRE_MAPPING_JSON; required]
  --web-api                   Providing this enables the web control api.
                              [env var: ORV_WEB_API_ENABLED; default: True;
                              required]
  --mqtt-api                  Providing this enables the MQTT api.  [env var:
                              ORV_MQTT_API_ENABLED; default: True; required]
  --web-api-host TEXT         Host address the web API binds to.  [env var:
                              ORV_WEB_API_HOST; default: 0.0.0.0]
  --web-api-port INTEGER      Port the web API listens on.  [env var:
                              ORV_WEB_API_PORT; default: 8000]
  --mqtt-broker-host TEXT     Hostname or IP of the MQTT broker.  [env var:
                              ORV_MQTT_BROKER_HOST; default:
                              homeassistant.local]
  --mqtt-broker-port INTEGER  Port of the MQTT broker.  [env var:
                              ORV_MQTT_BROKER_PORT; default: 1883]
  --mqtt-device-id TEXT       Device ID used for MQTT discovery/state topics.
                              [env var: ORV_MQTT_DEVICE_ID; default: orv-1]
  --mqtt-username TEXT        MQTT Broker username.  [env var:
                              ORV_MQTT_USERNAME; default: orv_user]
  --mqtt-password TEXT        MQTT Broker password.  [env var:
                              ORV_MQTT_PASSWORD; default: password]
  --help                      Show this message and exit.

  
(.venv) devon@ESO-3-DEV-VM:~/Documents/projects/open_rack_vent$ python orvcli.py render-systemd --help
Usage: orvcli.py render-systemd [OPTIONS]

  Creates a systemd unit that will start the run at boot with the given
  parameters. The current executable is used as the systemd executable and all
  arguments are pre-validated and passed as environment variables in the unit.

Options:
  --platform TEXT             The type of hardware running this application.
                              Options below. Either provide index or value:
                                 0: BeagleBoneBlack  [env var: ORV_PLATFORM; default: BeagleBoneBlack]
  --pcb-revision TEXT         The revision of the board driving the fans etc.
                              Options below. Either provide index or value:
                                 0: v1.0.0  [env var: ORV_PCB_REVISION; default: v1.0.0]
  --wire-mapping-json TEXT    JSON payload string with keys:
                                 • version: Enum[1]
                                 • fans: Dict[Enum[INTAKE_LOWER, INTAKE_UPPER, EXHAUST_LOWER, EXHAUST_UPPER], List[Enum[ONBOARD, PN1, PN2, PN3, PN4, PN5]]]
                                 • thermistors: Dict[Enum[INTAKE_LOWER, INTAKE_UPPER, EXHAUST_LOWER, EXHAUST_UPPER], List[Enum[TMP0, TMP1, TMP2, TMP3, TMP4, TMP5, TMP6]]]  [env var: ORV_WIRE_MAPPING_JSON; required]
  --web-api                   Providing this enables the web control api.
                              [env var: ORV_WEB_API_ENABLED; default: True;
                              required]
  --mqtt-api                  Providing this enables the MQTT api.  [env var:
                              ORV_MQTT_API_ENABLED; default: True; required]
  --web-api-host TEXT         Host address the web API binds to.  [env var:
                              ORV_WEB_API_HOST; default: 0.0.0.0]
  --web-api-port INTEGER      Port the web API listens on.  [env var:
                              ORV_WEB_API_PORT; default: 8000]
  --mqtt-broker-host TEXT     Hostname or IP of the MQTT broker.  [env var:
                              ORV_MQTT_BROKER_HOST; default:
                              homeassistant.local]
  --mqtt-broker-port INTEGER  Port of the MQTT broker.  [env var:
                              ORV_MQTT_BROKER_PORT; default: 1883]
  --mqtt-device-id TEXT       Device ID used for MQTT discovery/state topics.
                              [env var: ORV_MQTT_DEVICE_ID; default: orv-1]
  --mqtt-username TEXT        MQTT Broker username.  [env var:
                              ORV_MQTT_USERNAME; default: orv_user]
  --mqtt-password TEXT        MQTT Broker password.  [env var:
                              ORV_MQTT_PASSWORD; default: password]
  --output-path FILE          The resulting systemd service def will be
                              written to this path.  [default: /home/devon/Doc
                              uments/projects/open_rack_vent/open_rack_vent.se
                              rvice]
  --help                      Show this message and exit.
```

## Getting Started

### Python Dependencies

Poetry is required to manage Python dependencies. You can install it easily by following the
operating system specific instructions [here](https://python-poetry.org/docs/#installation).

`pyproject.toml` contains dependencies for required Python modules for building, testing, and 
developing. They can all be installed in a [virtual environment](https://docs.python.org/3/library/venv.html) 
using the follow commands:

```
python3.10 -m venv .venv
source ./.venv/bin/activate
poetry install
```

There's also a bin script to do this, and will install poetry if you don't already have it:

```
./tools/create_venv.sh
```

## Developer Guide

The following is documentation for developers that would like to contribute
to OpenRackVent.

### Pycharm Note

Make sure you mark `open_rack_vent` and `./test` as source roots!

### Testing

This project uses pytest to manage and run unit tests. Unit tests located in the `test` directory 
are automatically run during the CI build. You can run them manually with:

```
./tools/run_tests.sh
```

### Local Linting

There are a few linters/code checks included with this project to speed up the development process:

* Black - An automatic code formatter, never think about python style again.
* Isort - Automatically organizes imports in your modules.
* Pylint - Check your code against many of the python style guide rules.
* Mypy - Check your code to make sure it is properly typed.

You can run these tools automatically in check mode, meaning you will get an error if any of them
would not pass with:

```
./tools/run_checks.sh
```

Or actually automatically apply the fixes with:

```
./tools/apply_linters.sh
```

There are also scripts in `./tools/` that include run/check for each individual tool.


### Using pre-commit

Upon cloning the repo, to use pre-commit, you'll need to install the hooks with:

```
pre-commit install --hook-type pre-commit --hook-type pre-push
```

By default:

* black
* pylint
* isort
* mypy

Are all run in apply-mode and must pass in order to actually make the commit.

Also by default, pytest needs to pass before you can push.

If you'd like skip these checks you can commit with:

```
git commit --no-verify
```

If you'd like to quickly run these pre-commit checks on all files (not just the staged ones) you
can run:

```
pre-commit run --all-files
```


