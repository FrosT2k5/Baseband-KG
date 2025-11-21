references:
https://docs.srsran.com/projects/4g/en/latest/general/source/1_installation.html

1. Install UHD from source (can use prebuilt packages from apt as well, but building from source is prefferable)
Install Dependencies :
```sudo apt-get install autoconf automake build-essential ccache cmake cpufrequtils doxygen ethtool \
g++ git inetutils-tools libboost-all-dev libncurses5 libncurses5-dev libusb-1.0-0 libusb-1.0-0-dev \
libusb-dev python3-dev python3-mako python3-numpy python3-requests python3-scipy python3-setuptools \
python3-ruamel.yaml
```
Build commands
```
git clone https://github.com/EttusResearch/uhd.git
cd uhd/host
mkdir build
cd build
cmake ../
make
make test # This step is optional
sudo make install
sudo ldconfig
```
Confirm Installation:
```
uhd_usrp_probe --version
```

2. Install SrsRAN build dependencies:
```
sudo apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev
# SRSGUI dependencies:
sudo apt-get install libboost-system-dev libboost-test-dev libboost-thread-dev libqwt-qt5-dev qtbase5-dev
```
3. Install SrsGUI (Optional):
```
git clone https://github.com/srsLTE/srsGUI.git
cd srsgui
mkdir build
cd build
cmake ../
make 
sudo make install
sudo ldconfig
```
4.  Build SrsRAN from source:
```
git clone https://github.com/srsRAN/srsRAN_4G.git
cd srsRAN_4G
mkdir build
cd build
cmake ../
make
make test
# Installation commands:
sudo make install
srsran_install_configs.sh user
```
5. Confirm Installation:
```
srsepc --version
srsenb --versionsms
```
6. Setup configuration files
```
sudo srsran_4g_install_configs.sh user
```
Initialises default configuration in folder `/root/.config/srsran`
Important configuration Files:
`enb.conf` - configuration related to srsENB
`epc.conf` - configuration related to srsEPC
`rr.conf` - configuration related to radio resouces, LTE network cells (nodes), frequencies, etc

Additional Resouces:
Full SRSRan documentation: https://docs.srsran.com/projects/4g/en/latest/getting_started.html
FirmWire Devs LTE network setup: https://hernan.de/blog/creating-a-cellular-testbed-with-yatebts-and-srslte/
SRSRan with COTS UE guide: https://docs.srsran.com/projects/4g/en/latest/app_notes/source/cots_ue/source/index.html