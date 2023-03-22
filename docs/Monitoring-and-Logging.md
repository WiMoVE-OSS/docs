## Monitoring

For monitoring purposes, WiMoVE exposes a prometheus endpoint. Via the endpoint, information about how many stations are associated, events have been handled and how long the event handling was can be gathered. 
This is activated by default. We plan on making monitoring optional via a config option.

## Logging

WiMove provides logs via syslog. A syslog aggregator like syslogng can be used to aggregate those logs on a different host.