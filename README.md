## Other Project
For Proxmox VE(PVE) Anti Detection, see https://github.com/zhaodice/proxmox-ve-anti-detection

# QEMU Anti Detection
A patch for various QEMU versions that aims to prevent VM detection methods based on data reported by the emulator. The "QEMU keyboard" for example is then renamed to "ASUS keyboard". Serial numbers, the VM bit in the guest's UEFI and the Boot Graphics Record Table are also modified. 
However, because of timing based attacks like RDTSC, [which is reported incorrectly in a VM](https://github.com/WCharacter/RDTSC-KVM-Handler), this is not a silver bullet. 
But changing this information of the virtual devices is still an integral part of creating an undetected virutal machine. 

 | Type       | Engine | Bypass |
 |------------|--------|--------|
 | AntiCheat  | Anti Cheat Expert (ACE) | ☑️ |
 | AntiCheat  | Easy Anti Cheat (EAC) | ☑️ | 
 | AntiCheat  | Gepard Shield | ☑️ (Needs patched kernel on host: https://github.com/WCharacter/RDTSC-KVM-Handler ) |
 | AntiCheat  | Mhyprot | ☑️ |
 | AntiCheat  | nProtect GameGuard (NP) | ☑️ | 
 | AntiCheat  | Roblox | ☑️ May work with Hyper-V in the guest: https://github.com/zhaodice/qemu-anti-detection/issues/56 | 
 | AntiCheat  | Vanguard | ‼️(1: Incorrect function) | 
 | Encrypt    | Enigma Protector | ☑️ | 
 | Encrypt    | Safegine Shielden | ☑️ |
 | Encrypt    | Themida | ☑️ |
 | Encrypt    | VMProtect | ☑️ | 
 | Encrypt    | VProtect | ☑️ |       

‼️ There are games that cannot run under this environment but I am not sure whether QEMU has been detected, because the game doesn't report "Virtual machine detected" specifically. 
If you have any clue, feel free to tell me :)

### Flaws this patch does not fix in QEMU's source:
These commands exit with "No instance(s) available" and could therefore EXPOSE THE VM. We do not yet know how to simulate this data.
```
wmic path Win32_Fan get *

wmic path Win32_CacheMemory get *

wmic path Win32_VoltageProbe get *

wmic path Win32_PerfFormattedData_Counters_ThermalZoneInformation get *

wmic path CIM_Memory get *

wmic path CIM_Sensor get *

wmic path CIM_NumericSensor get *

wmic path CIM_TemperatureSensor get *

wmic path CIM_VoltageSensor get *
```

## Build Dependencies
⚠️ _Always maintain an installation of QEMU managed by your package manager, because it may delete necessary runtime dependencies otherwise! The binaries you compile are saved in **/usr/local/bin**, so they will take precedence._

**Arch**:
`sudo pacman -S git wget base-devel glib2 ninja python`

**Ubuntu**:
`sudo apt install git build-essential ninja-build python-venv libglib2.0-0 flex bison`

## Patching and building QEMU
```
git clone https://github.com/kunihir0/better-qemu-anti-detection.git
wget https://download.qemu.org/qemu-8.2.2.tar.xz
tar xvJf qemu-8.2.2.tar.xz
cd qemu-8.2.2
git apply ../qemu-8.2.0.patch
git apply ../qemu-8.2.0-libnfs6.patch
git apply ../qemu-8.2.0-sysc.patch
./configure
sudo make install -j$(nproc)
```

# QEMU XML Config

Insert YOUR virtual machine's uuid.
```
<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">
  <name>Entertainment</name>
  <uuid>REPLACE YOUR UUID HERE!</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://microsoft.com/win/10"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory unit="KiB">1548288</memory>
  <currentMemory unit="KiB">1548288</currentMemory>
  <memoryBacking>
    <source type="memfd"/>
    <access mode="shared"/>
  </memoryBacking>
  <vcpu placement="static">12</vcpu>
  <os firmware="efi">
    <type arch="x86_64" machine="pc-q35-7.0">hvm</type>
    <loader/>
    <smbios mode="host"/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vendor_id state="on" value="GenuineIntel"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <vmport state="off"/>
    <smm state="on"/>
    <ioapic driver="kvm"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on">
    <feature policy="disable" name="hypervisor"/>
  </cpu>
  <clock offset="localtime">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
    <timer name="hypervclock" present="yes"/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <qemu:commandline>
    <qemu:arg value="-smbios"/>
    <qemu:arg value="type=0,version=UX305UA.201"/>
    <qemu:arg value="-smbios"/>
    <qemu:arg value="type=1,manufacturer=ASUS,product=UX305UA,version=2021.1"/>
    <qemu:arg value="-smbios"/>
    <qemu:arg value="type=2,manufacturer=Intel,version=2021.5,product=Intel i9-12900K"/>
    <qemu:arg value="-smbios"/>
    <qemu:arg value="type=3,manufacturer=XBZJ"/>
    <qemu:arg value="-smbios"/>
    <qemu:arg value="type=17,manufacturer=KINGSTON,loc_pfx=DDR5,speed=4800,serial=000000,part=0000"/>
    <qemu:arg value="-smbios"/>
    <qemu:arg value="type=4,manufacturer=Intel,max-speed=4800,current-speed=4800"/>
    <qemu:arg value="-cpu"/>
    <qemu:arg value="host,family=6,model=158,stepping=2,model_id=Intel(R) Core(TM) i9-12900K CPU @ 2.60GHz,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true,hypervisor=off"/>
    <qemu:arg value="-machine"/>
    <qemu:arg value="q35,kernel_irqchip=on"/>
  </qemu:commandline>
</domain>
```
![Screenshot_20220819_230305](https://user-images.githubusercontent.com/63996691/185649897-b7609626-ee6d-42b1-bc5e-4465cb41a19a.png)
