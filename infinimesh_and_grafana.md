# Sending and visualizing data via infinimesh and Grafana

## Creating Device

1. Generate new certificate

    ```sh
    openssl genrsa -out my_device.key 4096
    openssl req -new -x509 -sha256 -key my_device.key -out my_device.crt -days 365 -subj "/C=/ST=/L=/O=/CN=/emailAddress=/"
    ```

    This will produce certificate(`my_device.crt`) and key(`my_device.key`). Save them, dont't lose them.

2. Go to infinimesh Console
3. Create new Namespace if needed

    > You can't create devices in your default namespace

4. Create new Device with `my_device.crt` you generated at step 1.

    > Each device must have unique certificate, so don't reuse them if you're creating more devices

5. Make sure device is Enabled
6. Copy its ID (let's say its `0x777`)

    > It has hex form, so looks like **0x***123*

## Sending Data from your computer

1. Install one of the available MQTT CLIs (mosquitto / mqtt.js)
2. Decide on your data, here is an example with all types(it's just common JSON):

    ```json
        {
            "str_key": "string value",
            "num_key": 123.456,
            "bool_key": true,
        }
    ```

3. Run one of this

    ```sh
    # Mosquitto example
    mosquitto_pub \
    --cert my_device.crt \
    --key my_device.key \
    -m '{"test": 0}' \
    -t "devices/0x777/state/reported/delta" \
    -h mqtt.api.infinimesh.app -d -p 8883

    # mqtt.js example
    mqtt pub \
    --cert my_device.crt \
    --key my_device.key \
    -m '{"test": 0}' \
    -t 'devices/0x777/state/reported/delta' \
    -h mqtt.api.infinimesh.app -d -p 8883
    ```

If transfer was successful, you'll see that state(`{"test": 0}`) on device page in console.

> Note, in the current way of things, the values would be saved according to the following schema:

  ```
  Integers and Floats - directly as Timeseries value
  Boolean - 0 for False, 1 for True
  Strings - Not saved directly, only added as labels in RedisTimeseries
  ```

So if you send data like:

```json
{
    "test": 123,
    "flag": true,
    "note": "locationX"
}
```

Two records in two keys going to be made like:

```redis
TS.ADD test timeStamp 123 note=locationX
TS.ADD flag timeStamp 1   note=locationX
```

## Visualising Data in Grafana

1. Go to grafana
2. Connect Redis connector to your Redis-TS or use the one given if it's present
3. Create Dashboard and Panel
4. Click Add Query
5. Pick `RedisTimeSeries` as Query Type
6. Pick `TS.RANGE` as Command
7. Enter your device id and data key in form like `id:key`

    > For example we have device `0x777` from previous steps
    We also sent the data like `{"test": 0}`
    Your key should be `0x777:test`

8. Repeat steps starting from step 4 if you want to see more keys