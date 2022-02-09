# commaai_notes
All notes related to turning my corolla to self driving car using comma AI


# Fingerprinting Issue:

 - Had no idea what the issue was as after the setup I was getting the fall back message that device not supported and working as a dashcam mode.
 - Researched on the discord and quickly was able to identify the issue.
 - Biggest struggle was getting the ssh to work as the comma 2  > comma 3 the username changed from root > comma. This wasn't well documented
 - Learnt some neet techniques when struggling with setting up ssh via github. The way comma appraches this is amazing as well. ( using github keys)
 - my ssh config was setup to use a different key for local ip ( this is also learnt from unix stackexchange)

### SSH Config
 ```
Host github.com
  AddKeysToAgent yes
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/<key name#1>
Host comma
  Hostname 192.168.0.*
  AddKeysToAgent yes
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/<keyname # 2 >
Host *
  UseKeychain yes
 ```


Once I got logged in the file can be found here: `/data/openpilot/selfdrive/car/toyota` 
The cars can be found here:https://engage.toyota.com/static/images/toyota_safety_sense/TSS_Applicability_Chart.pdf
Hybrid versions of cars are distictly identified similarly TSS 2.0. So check with your cars exact version. Toyota Hatchback being unique has a different value.

Follow the editing instructions as per here: https://github.com/commaai/openpilot/wiki/Fingerprinting

basically once you login to the comma admin https://useradmin.comma.ai/ > go to logs > routes for your device.
Select carParams

```
{'carParams': {'carFingerprint': 'mock',
               'carFw': [{'address': 1968, 'ecu': 'esp', 'fwVersion': b'\x01F152612B62\x00\x00\x00\x00\x00\x00', 'subAddress': 0},
                         {'address': 1792, 'ecu': 'engine', 'fwVersion': b'\x018966312S5000\x00\x00\x00\x00', 'subAddress': 0},
                         {'address': 1953, 'ecu': 'eps', 'fwVersion': b'\x018965B12530\x00\x00\x00\x00\x00\x00', 'subAddress': 0},
                         {'address': 1872, 'ecu': 'fwdRadar', 'fwVersion': b'\x018821F3301400\x00\x00\x00\x00', 'subAddress': 15},
                         {'address': 1872,
                          'ecu': 'fwdCamera',
                          'fwVersion': b'\x028646F1202200\x00\x00\x00\x008646G2601500\x00\x00\x00\x00',
                          'subAddress': 109}],
               'carName': 'mock',
```

This needs to be converted as per docs. Its a little inundating but once you figure what is what its super simple. The following was the change (lots of the values already there and had to edit just one value here)


 ```

0x700 / None
{'address': 1792, 'ecu': 'engine', 'fwVersion': b'\x018966312S5000\x00\x00\x00\x00', 'subAddress': 0} 


Ox7E0 / None
#  NOT THERE

0x7A1 / None
{'address': 1953, 'ecu': 'eps', 'fwVersion': b'\x018965B12530\x00\x00\x00\x00\x00\x00', 'subAddress': 0},
# Already There

0x7b0 / None
{'address': 1968, 'ecu': 'esp', 'fwVersion': b'\x01F152612B62\x00\x00\x00\x00\x00\x00', 'subAddress': 0},
# Need to be added

0x750 / 0xf
{'address': 1872, 'ecu': 'fwdRadar', 'fwVersion': b'\x018821F3301400\x00\x00\x00\x00', 'subAddress': 15},
#Already there

0x750 / 0x6d
{'address': 1872, 'ecu': 'fwdCamera', 'fwVersion': b'\x028646F1202200\x00\x00\x00\x008646G2601500\x00\x00\x00\x00','subAddress': 109}]
#Already there

 ```