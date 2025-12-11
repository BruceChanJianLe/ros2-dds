# ROS2 DDS

This repository demonstrates the use of ROS2 DDS.  

As of jazzy, there is two major dds that is available, one `CycloneDDS` and
the other is `Zenoh`. Although, `Zenoh` is not DDS per se, but it's a modern
communication protocol designed to complement and sometimes surpass DDS,
especially in challenging wireless/IoT environments.  


## CycloneDDS

Below are some recommended settings by [Autoware](https://autowarefoundation.github.io/autoware-documentation/main/installation/additional-settings-for-developers/network-configuration/dds-settings/).
For quick reference, you may follow the steps below:  

### 1. Enable multicast for lo

<details>
  <summary>Quick but temporary solution:</summary>

  ```bash
  sudo ip link set lo multicast on
  ```

</details>

Proper solution:  
```bash
sudoedit /etc/systemd/system/multicast-lo.service
```

Paste the following:  
```bash
[Unit]
Description=Enable Multicast on Loopback

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set lo multicast on

[Install]
WantedBy=multi-user.target
```

Start and enable service:  
```bash
# Make it recognized
sudo systemctl daemon-reload

# Make it run on startup
sudo systemctl enable multicast-lo.service

# Start it now
sudo systemctl start multicast-lo.service
```

Validation:  
```bash
sudo systemctl status multicast-lo.service
```

### 2. Enlarge Linux Kernel Max Buffer Size

<details>
  <summary>Quick but temporary solution:</summary>

  ```bash
  # Increase the maximum receive buffer size for network packets
  sudo sysctl -w net.core.rmem_max=2147483647  # 2 GiB, default is 208 KiB

  # IP fragmentation settings
  sudo sysctl -w net.ipv4.ipfrag_time=3  # in seconds, default is 30 s
  sudo sysctl -w net.ipv4.ipfrag_high_thresh=134217728  # 128 MiB, default is 256 KiB
  ```

</details>

Proper solution:  
```bash
sudoedit /etc/sysctl.d/10-cyclone-max.conf
```

Paste the following:  
```bash
# Increase the maximum receive buffer size for network packets
net.core.rmem_max=2147483647  # 2 GiB, default is 208 KiB

# IP fragmentation settings
net.ipv4.ipfrag_time=3  # in seconds, default is 30 s
net.ipv4.ipfrag_high_thresh=134217728  # 128 MiB, default is 256 KiB
```

Validate:  
```bash
sysctl net.core.rmem_max net.ipv4.ipfrag_time net.ipv4.ipfrag_high_thresh
```

Do note that if the values differs from our settings, just perform a reboot or
run the temporary solution as the config files will kick into action only on
the next boot.  

### 3. Configure CycloneDDS Profile  

Note to select your network interface based on your need, if you only do
local testing, just uncomment the Interfaces tag:  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
  <Domain Id="any">
    <General>
      <!-- <Interfaces> -->
      <!--   <NetworkInterface autodetermine="false" name="lo" priority="default" multicast="default" /> -->
      <!-- </Interfaces> -->
      <AllowMulticast>default</AllowMulticast>
      <MaxMessageSize>65500B</MaxMessageSize>
    </General>
    <Internal>
      <SocketReceiveBufferSize min="10MB"/>
      <Watermarks>
        <WhcHigh>500kB</WhcHigh>
      </Watermarks>
    </Internal>
  </Domain>
</CycloneDDS>
```
