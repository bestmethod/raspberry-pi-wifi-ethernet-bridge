# Raspberry Pi OS - enable SSH and connect to WiFi completely headless

Raspberry Pi OS allows you to enable SSH and connect to WiFi by setting it up from the machine on which you have imaged the OS onto the SD card. This allows you to image thr SD card, setup ssh and WiFi and start up in a Raspberry Pi without having to connect the Pi to HDMI and keyboard. Truly headless.

Of course, you will need to work out what the IP address of your Pi is to SSH to it, but that's outside of the scope of this article.

## Enable ssh

On the machine on which you have created the SD card, it should have a partition called `boot` of type fat32 that you can access (if you just imaged the OS, you may need to remove and reinsert the card).

Go to that partition and create an empty file called `ssh`. That's it. On next boot, the OS will enable openssh server.

## Configure WiFi

On the machine on which you have created the SD card, it should have a partition called `boot` of type fat32 that you can access (if you just imaged the OS, you may need to remove and reinsert the card).

Go to that partition and create a file called `wpa_supplicant.conf` with the following contents:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=«your_ISO-3166-1_two-letter_country_code»
 
network={
ssid="«your_SSID»"
psk="«your_PSK»"
key_mgmt=WPA-PSK
}
```

Replace:

item | value
--- | ---
«your_ISO-3166-1_two-letter_country_code» | [country codes](https://www.wikiwand.com/en/ISO_3166-1_alpha-2#/Officially_assigned_code_elements)
«your_SSID» | The SSID of your WiFi network
«your_PSK» | The password of your WiFi network

That's it. When you boot the Pi, it will replace it's `/etc/wpa_supplicant/wpa_supplicant.conf` with this file and connect to that WiFi.

## Non-persistent one-off WiFi name change

If you would like to connect one-off to a network, without updating the OS config file, follow the above, but set `update_config=0`.
