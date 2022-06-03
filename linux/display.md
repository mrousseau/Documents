```
# étape 1 
xrand #récupérer le nom de l'ecran ex eDP-1 ou HDMI-1 ou Virtual1

# étape 2 
cvt <resolution><rafraichissement> # cvt 1920 1080 75


xrandr --newmode "1920x1080_75.00"  220.75  1920 2064 2264 2608  1080 1083 1088 1130 -hsync +vsync
xrandr --addmode Virtual1 "1920x1080_75.00"
xrandr --output Virtual1 --mode "1920x1080_75.00"
```

#### Documents origin
https://dev.to/ksckaan1/setting-up-custom-screen-resolution-on-linux-permanently-1abg
