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

## Installation

1. Download `template_fritzbox_7590_tr064.yaml`.
2. Import the template into Zabbix.
3. Create a host for your FRITZ!Box.
4. Set the host interface address to the IP address of your FRITZ!Box.
5. Link the template `FRITZBox 7590 TR-064` to the host.
6. Wait for the first polling interval.

No Zabbix agent is required on the FRITZ!Box.

The template uses:

`{HOST.CONN}`

for the FRITZ!Box address, so no IP address is hard-coded in the template.

It therefore works with custom network configurations such as:

- `192.168.178.1`
- `192.168.1.1`
- `10.0.0.1`
- other custom FRITZ!Box addresses

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
