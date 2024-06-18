# LocalMoments

Enable local control of your Princess Moments Wi-Fi enabled hardware.  

The process works for sure with the `Princess 236060 Moments Jug Kettle Wi-Fi`  
From the reverse engineering the app, seeing how the hardware is built I'm willing to bet it would be also applicable to other hardware in the Princess Moments brand.

---

The device is based on an ESP32, and it connects to `kitchen.cloud.homewizard.com` MQTT server, and it uses certificate authentication. That makes it difficult to MITM the connection, but if we control the DNS and MQTT server we can completely bypass the cloud and control it locally

# Requirements

 - A way to override DNS entry - here I'll refer to adguard
 - MQTT server with customizable websocket path, or reverse proxy supporting rewrites

# Steps

 1. Set up the device with the app, to connect it to the Wi-Fi (see TBD).
 2. (Optional) Update firmware, we won't have access to it later.
 3. Set up MQTT server
    1. The device connects over port `443` via websockets with TLS and path `/mqtt`, but the server certificate is not actually verified.
    2. My solution: using `EMQX`, bind port 443 with type wss, and specify max connections for 1 device. Then also disable authentication for this one endpoint.
    3. Other option: Reverse proxy that rewrites path `/mqtt` to `/ws`, then it can be sent to `Mosquitto` instance. In my case, `Caddy` seemed to have a websocket activity timeout of 5 minutes, and the kettle was constantly disconnecting. I also tried `YARP` and specifying it's activity timeouts, but with similiar results.
 4. Set up entry on adguard rewrite rules for `kitchen.cloud.homewizard.com` and point it to your `EMQX` instance/reverse proxy. 
    1. Note: I had issues when the record pointed at a `CNAME` for my `EMQX` cluster, and similarly when I had multiple `A` entries, so it seems it might handle DNS responses with multiple destinations weirdly?
 5. Disconnect power to the device for a few seconds, so it fully turns off, then start it again.
 6. Verify the device connected correctly to your MQTT server

# TBD

- [ ] Script to automate connection of device to Wi-Fi, to skip the need to install an app  
- [ ] Home Assistant component that would discover the MQTT device and all it's sensors  
- [ ] The device listens on `appliance/<device_type>/<device ID>/$update/set`, which seems to be a way to update firmware, but would require dumping the flash memory of the device to see the format it expects (or extracting authentication private key and connecting to the original server).
