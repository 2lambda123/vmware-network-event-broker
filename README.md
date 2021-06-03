### network-event-broker
----
A Daemon configures network on ```systemd-networkd's``` [DBus](https://www.freedesktop.org/wiki/Software/dbus/) events. Use cases like a way to run a command when get a new ip via dhcp. As of now there is no native ```systemd-networkd``` solution for this problem. Afterburn listens to dbus signals and reacts according to the events. It's written in golang.

```network-event-broker``` creates link state directories ```carrier.d```,  ```configured.d```,  ```degraded.d```  ```no-carrier.d```  ```routable.d``` and manager state dir ```manager.d``` in ```/etc/network-event-broker```. Executable scripts can be placed into directories that reflect systemd-networkd operational states, and are executed when the daemon receives the relevant event from `systemd-networkd`. See [networkctl](https://www.freedesktop.org/software/systemd/man/networkctl.html).

```bash
❯ networkctl list

IDX LINK    TYPE     OPERATIONAL SETUP
  1 lo      loopback carrier     unmanaged
  2 ens33   ether    routable    configured
  3 ens37   ether    routable    unmanaged
  4 virbr0  bridge   no-carrier  unmanaged

4 links listed.
```

Enviroment variables ```OperationalState=``` , ```LINK=```, ```LINKIFINDEX=``` and DHCP lease information ```DHCP_LEASE= ``` passed to the scripts via enviroment viariables.

```bash
May 14 17:08:13 Zeus cat[273185]: OperationalState="routable"
May 14 17:08:13 Zeus cat[273185]: LINK=ens33
```


#### Building from source
----

```bash

❯ make build
❯ sudo make install

```

### Configuration
----

Configuration file `network-broker.toml` located in ```/etc/network-broker/``` directory to manage the configuration.
The `[Network]` section takes following Keys:

```bash
Links=
```
A whitespace-separated list of links which which events should be monitored.


The `[System]` section takes following Keys:
``` bash
LogLevel=
```
Specifies the log level. Takes one of `info`, `debug` ... Defaults to `info`.


```bash

❯ sudo cat /etc/network-broker/network-broker.toml
[System]
LogLevel="debug"
LogFormat="text"

[Network]
Links="ens33 ens37"                    
```

```bash
❯ sudo systemctl status network-broker.service

● network-broker.service - A Daemon configures network on systemd-networkd's dbus event
     Loaded: loaded (/usr/lib/systemd/system/network-broker.service; disabled; vendor preset: disabled)
     Active: active (running) since Sat 2021-05-15 02:53:10 CEST; 18s ago
       Docs: man:network-brokerconf(5)
   Main PID: 29886 (network-broker)
      Tasks: 6 (limit: 8989)
     Memory: 1.8M
        CPU: 16ms
     CGroup: /system.slice/network-broker.service
             └─29886 /usr/bin/network-broker

May 15 02:53:10 Zeus systemd[1]: Started A Daemon configures network on systemd-networkd's dbus event.
May 15 02:53:10 Zeus network-broker[29886]: time="2021-05-15T02:53:10+02:00" level=info msg="Parsed links 'ens33 ens37' from configuration"
May 15 02:53:10 Zeus network-broker[29886]: time="2021-05-15T02:53:10+02:00" level=info msg="Acquired netlink message link='ens33' ifindex='2'"
May 15 02:53:10 Zeus network-broker[29886]: time="2021-05-15T02:53:10+02:00" level=info msg="Acquired netlink message link='ens37' ifindex='3'"
May 15 02:53:10 Zeus network-broker[29886]: time="2021-05-15T02:53:10+02:00" level=info msg="Acquired netlink message link='virbr0' ifindex='4'"


```
DBus signals generated by ```systemd-networkd```
```bash

&{:1.683 /org/freedesktop/network1/link/_32 org.freedesktop.DBus.Properties.PropertiesChanged [org.freedesktop.network1.Link map[AdministrativeState:"configured"] []] 10}
```

```
‣ Type=signal  Endian=l  Flags=1  Version=1 Cookie=24  Timestamp="Sun 2021-05-16 08:06:05.905781 UTC"
  Sender=:1.292  Path=/org/freedesktop/network1  Interface=org.freedesktop.DBus.Properties  Member=PropertiesChanged
  UniqueName=:1.292
  MESSAGE "sa{sv}as" {
          STRING "org.freedesktop.network1.Manager";
          ARRAY "{sv}" {
                  DICT_ENTRY "sv" {
                          STRING "OperationalState";
                          VARIANT "s" {
                                  STRING "degraded";
                          };
                  };
          };
          ARRAY "s" {
          };
  };

```


#### Contributing
----

The **Network Event Broker** project team welcomes contributions from the community. If you wish to contribute code and you have not signed our contributor license agreement (CLA), our bot will update the issue when you open a Pull Request. For any questions about the CLA process, please refer to our [FAQ](https://cla.vmware.com/faq).

slack channel [#photon](https://code.vmware.com/web/code/join).

#### License
----

[Apache-2.0](https://spdx.org/licenses/Apache-2.0.html)
