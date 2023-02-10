# Xorg Configurations for DevTerm

## Mapping GamePad to Keys

This Xorg config will,

- Map D-Pad to up/down/left/right
- Map YBXA to hjk
- Disable the joystick as a mouse

Install `xserver-xorg-input-joystick` package on Debian/Ubuntu, create the following configuration in /etc/X11/xorg.conf.d/51-joystick.conf, and restart Xorg.

```
Section "InputClass"
  Identifier "DevTerm Gamepad"
  Option "StartKeysEnabled" "True"
  Option "MatchDevicePath" "/dev/input/event*"
  Option "MatchIsJoystick" "on"

  #Do not use as a mouse
  Option "StartMouseEnabled" "False"

  #XABY buttons
  #Keycodes: 43=h, 44=j, 45=k, 46=l
  Option "MapButton1" "key=45" #X
  Option "MapButton2" "key=46" #A
  Option "MapButton3" "key=44" #B
  Option "MapButton4" "key=43" #Y

  #D-Pad
  #Keycodes:  111=up, 116=down, 113=left, 114=right
  Option "MapAxis1" "mode=accelerated keylow=113 keyhigh=114" #left/right
  Option "MapAxis2" "mode=accelerated keylow=111 keyhigh=116" #up/down
  Option "MapAxis3" "mode=accelerated keylow=113 keyhigh=114" #left/right
  Option "MapAxis4" "mode=accelerated keylow=111 keyhigh=116" #up/down
EndSection
```
