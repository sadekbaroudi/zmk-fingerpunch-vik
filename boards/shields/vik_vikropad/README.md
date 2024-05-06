# vikropad

Standalone macropad connected as a [VIK module](https://github.com/sadekbaroudi/vik/) only!

See the [vikropad pcb github page](https://github.com/sadekbaroudi/vik/tree/master/pcb/vikropad/)

Sample local build command:
`west build --pristine -b "xivik@0.2.0" -- -DZMK_CONFIG=/home/sadek/zmk-fingerpunch-vik/config -DSHIELD="vik_vikropad" -DZMK_EXTRA_MODULES='/home/sadek/zmk-fingerpunch-controllers;/home/sadek/zmk-fingerpunch-vik'`