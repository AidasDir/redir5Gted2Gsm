# redir5Gted2Gsm using LimeSDR

To allow this location to be customised, set an environment variable:
```bash
export SRSRAN_INSTALL=${HOME}/srsran
```

#Install SoapySDR
```bash
sudo apt-get install cmake g++ libpython3-dev python3-numpy swig
git clone --branch soapy-sdr-0.8.1 https://github.com/pothosware/SoapySDR.git
cd SoapySDR
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=${SRSRAN_INSTALL} -DCMAKE_INSTALL_PREFIX=${SRSRAN_INSTALL} ..
make -j`nproc` && make install
```

Having built and installed the SoapySDR framework, a basic sanity check can be performed:
````bash
export LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib
export PATH=${SRSRAN_INSTALL}/bin:$PATH
SoapySDRUtil --info
````
#Install LimeSuite

```bash
sudo apt-get install git g++ cmake libsqlite3-dev
sudo apt-get install libi2c-dev libusb-1.0-0-dev
sudo apt-get install libwxgtk3.0-gtk3-dev freeglut3-dev
git clone --branch v22.09.1 https://github.com/myriadrf/LimeSuite.git
cd LimeSuite
mkdir builddir && cd builddir
cmake -DCMAKE_PREFIX_PATH=${SRSRAN_INSTALL} -DCMAKE_INSTALL_PREFIX=${SRSRAN_INSTALL} ..
make -j`nproc` && make install
cd ../udev-rules
sudo bash install.sh
```
The above should build and install LimeSuite, along with its udev rules configuring security such that regular users have permission to access the hardware.
<br>
LimeSuiteGUI may be launched if desired for testing as follows:

```bash
export LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib
export PATH=${SRSRAN_INSTALL}/bin:$PATH
LimeSuiteGUI
```
As a side note, it was about this point in my setup that I wanted to ensure my SDR was running the latest firmware. Which can be achieved using LimeUtil as follows:

```bash
export LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib
export PATH=${SRSRAN_INSTALL}/bin:$PATH
LimeUtil --update
```

#srsGUI is a graphics library providing generating plots specifically for SDR applications.
```bash
sudo apt-get install libboost-system-dev libboost-test-dev libboost-thread-dev libqwt-qt5-dev qtbase5-dev
git clone --branch release_2_0_qt5 https://github.com/srsran/srsGUI.git
cd srsGUI
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=${SRSRAN_INSTALL} -DCMAKE_INSTALL_PREFIX=${SRSRAN_INSTALL} ..
make -j`nproc` && make install
```

#Install srsRAN

```bash
git clone https://github.com/srsran/srsran
cd srsran
git checkout 254cc719a9a31f64ce0262f4ca6ab72b1803477d
wget https://raw.githubusercontent.com/bbaranoff/redir5Gted2Gsm/main/redir5Gted2Gsm.patch
patch -p0 < redir5Gted2Gsm

mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=${SRSRAN_INSTALL} -DCMAKE_INSTALL_PREFIX=${SRSRAN_INSTALL} -DUSE_LTE_RATES=ON ..
make -j`nproc`
make test
make install
```
build now srsran as usual
```
git clone https://github.com/bbaranoff/redir5Gted2Gsm/
cd redir5Gted2Gsm
cp *.conf /root/.config/srsran
```

#edit enb.conf
<br>
In the “[enb]” section we need to update the MCC and MNC
```bash
[enb]
enb_id = 0x19B
mcc = 208
mnc = 15
...
```
In the “[rf]” section we need to inform srsRAN that we’re using a LimeSDR, while selecting which antenna ports are in use. We use the device_name and device_args parameters for this.
```bash
[rf]
dl_earfcn = 9460
tx_gain = 80
rx_gain = 40

device_name = soapy
device_args = driver=lime,rxant=LNAH,txant=BAND2
```
#rr.conf

Finally we need to select the radio frequency to transmit and receive on, which can conveniently be expressed

Find the cell_list section and update the dl_earfcn parameter. The section of my updated file is as follows:
```bash
...
cell_list =
(
  {
    // rf_port = 0;
    cell_id = 0x01;
    tac = 0x19cb;
    pci = 2;
    root_seq_idx = 0;
    dl_earfcn = 3350;
    //ul_earfcn = 21400;
    ho_active = false;
...
```
With that the eNodeB configuration is complete.

#epc.conf


In the “[mme]” section we need to update the MCC and MNC as above. My “[mme]” section therefore appears as follows:

```bash
[mme]
mme_code = 0x1a
mme_group = 0x0001
tac = 0x19cb
mcc = 208
mnc = 15
```

#Startup

1) In one terminal launch the EPC:
```bash
sudo LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib sh -c "cd ${HOME}/.config/srsran; ${SRSRAN_INSTALL}/bin/srsepc epc.conf"
```

2) In another terminal launch the eNodeB:
```bash
echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
sudo LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib sh -c "cd ${HOME}/.config/srsran; ${SRSRAN_INSTALL}/bin/srsenb enb.conf"
```


