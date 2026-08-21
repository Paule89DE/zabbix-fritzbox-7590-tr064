# Zabbix FRITZ!Box 7590 TR-064 Template

A Zabbix template for monitoring an AVM FRITZ!Box 7590 using the local
TR-064 / UPnP interfaces.

No FRITZ!Box user account, password or API token is required.

## Features

The template currently monitors:

- Current DSL synchronization downstream
- Current DSL synchronization upstream
- Maximum attainable DSL rate downstream
- Maximum attainable DSL rate upstream
- DSL SNR margin downstream
- DSL SNR margin upstream
- DSL CRC errors
- DSL FEC errors
- Current WAN download traffic
- Current WAN upload traffic

## Tested configuration

Successfully tested with:

- **Router:** AVM FRITZ!Box 7590
- **FRITZ!OS:** 8.25
- **DSL provider:** Deutsche Telekom
- **DSL type:** VDSL2 35b G.Vector (ITU G.993.5)
- **Template export format:** Zabbix 7.0
- **Polling interval:** 60 seconds
- **Zabbix Server:** Docker
- **FRITZ!Box interface:** local HTTP/TR-064 on TCP port 49000

Example DSL values during testing:

- Current downstream sync: ~292 Mbit/s
- Current upstream sync: ~46.7 Mbit/s
- Maximum attainable downstream: ~327 Mbit/s
- Maximum attainable upstream: ~48.5 Mbit/s
- Downstream SNR margin: ~10 dB
- Upstream SNR margin: ~7 dB

No Zabbix agent is required on the FRITZ!Box.

The template uses:

`{HOST.CONN}`

for the FRITZ!Box address, so no IP address is hard-coded in the template.

It therefore works with custom network configurations such as:

- `192.168.178.1`
- `192.168.1.1`
- `10.0.0.1`
- other custom FRITZ!Box addresses
  
## Installation

1. Download `template_fritzbox_7590_tr064.yaml`.
2. Import the template into Zabbix.
3. Create a host for your FRITZ!Box.
4. Under **Interfaces**, add an **Agent** interface.
5. Enter the IP address of your FRITZ!Box, for example `192.168.178.1`.
6. Keep **Connect to: IP** selected.
7. The Agent port `10050` can remain unchanged. No Zabbix agent actually needs to run on the FRITZ!Box.
8. Link the template `FRITZBox 7590 TR-064` to the host.
9. Wait for the first polling interval.

The Agent interface is only used to provide the FRITZ!Box address for the `{HOST.CONN}` macro.

The template itself communicates directly with the FRITZ!Box via HTTP on TCP port `49000`.

No Zabbix agent needs to be installed or running on the FRITZ!Box.

## Troubleshooting

### `Could not connect to server`

During development and testing, Zabbix occasionally reported errors similar to:

`Cannot perform request: Failed to connect to <FRITZ!Box-IP>:49000: Could not connect to server`

although the FRITZ!Box was reachable and the HTTP item test worked correctly.

In the tested Docker installation, restarting the **Zabbix Server container** resolved the issue.

Example:

```bash
docker restart zabbix-server
```

After the restart, the HTTP items started collecting data normally again.

Before restarting Zabbix, it is recommended to verify that the Zabbix Server itself can reach the FRITZ!Box on TCP port `49000`.

For Docker installations, this can be tested from inside the Zabbix Server container, for example:

```bash
docker exec -it zabbix-server sh
wget -O- http://192.168.178.1:49000/igddesc.xml
```

If XML data is returned, network connectivity between the Zabbix Server container and the FRITZ!Box is working.

This behavior was observed during testing and may be related to Zabbix configuration/cache state rather than the FRITZ!Box itself.


## Interfaces used

### WAN traffic

Endpoint:

`http://{HOST.CONN}:49000/igdupnp/control/WANCommonIFC1`

SOAP action:

`urn:schemas-upnp-org:service:WANCommonInterfaceConfig:1#GetAddonInfos`

Used for:

- Current WAN download traffic
- Current WAN upload traffic

### DSL information

Endpoint:

`http://{HOST.CONN}:49000/upnp/control/wandslifconfig1`

SOAP action:

`urn:dslforum-org:service:WANDSLInterfaceConfig:1#X_AVM-DE_GetDSLInfo`

Used for:

- Current DSL sync rate
- Maximum attainable DSL rate
- SNR margin
- FEC errors
- CRC errors

Despite the historical `dslforum-org` namespace, this is the service
type advertised by the FRITZ!Box itself via `tr64desc.xml`.

## Authentication

The SOAP actions currently used by this template work without
authentication on the tested FRITZ!Box 7590 with FRITZ!OS 8.25.

Therefore:

- No FRITZ!Box user account is required
- No password is stored in Zabbix
- No credentials are included in this repository

## Known limitation: GetStatisticsTotal

The tested FRITZ!Box advertises the action:

`GetStatisticsTotal`

in:

`/wandslifconfigSCPD.xml`

This action could potentially provide additional values such as:

- Link retrains
- Errored seconds
- Severely errored seconds

However, on the tested FRITZ!Box 7590 with FRITZ!OS 8.25 the following
behaviour was observed:

- Without authentication: HTTP `401 Unauthorized`
- With Digest authentication: SOAP error `502 XML error`

For this reason these metrics are currently not included.

## Compatibility

Currently confirmed:

| Model | FRITZ!OS | Status |
|---|---|---|
| FRITZ!Box 7590 | 8.25 | ✅ Tested |

Other FRITZ!Box models may work if they expose the same TR-064 / UPnP
services.

If you successfully test another model or FRITZ!OS version, please
open an issue or pull request.

## Security

This template only communicates with the FRITZ!Box inside the local
network using HTTP on TCP port `49000`.

No credentials or secrets are required.

## Author

Created and tested by **Paule89DE**.

Feedback, testing on other FRITZ!Box models and contributions are welcome.

## License

MIT License.
