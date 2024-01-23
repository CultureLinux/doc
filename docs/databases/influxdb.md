# Influxdb

## Retention
    influx
    > SHOW RETENTION POLICIES on icinga2
    > CREATE RETENTION POLICY "2years" ON icinga2 DURATION 100w REPLICATION 1 DEFAULT
    > DROP RETENTION POLICY "autogen"  ON icinga2