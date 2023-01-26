# GD32VF103_led_light
/*!
    \file    readme.txt

    \version 2023-01-25, V1.0.0, firmware for GD32VF103

*/


  This demo is based on the GD32VF103V-EVAL-V1.0 board, it shows how to use EXMC 
peripheral to drive LCD. This board uses EXMC_NE0 to support LCD. You can see
GigaDevice logo and website on the LCD screen.

  JP13 and JP4 must be fitted to EXMC. 


注意： 遇到的几个问题
1、riscv编译器的设定，可以使用在文件夹中指定的方法，也可以install之后使用
2、gcc库的匹配性，需要正确设置编译器参数
3、使用cmake以便于移植