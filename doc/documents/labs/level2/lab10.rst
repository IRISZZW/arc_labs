.. _lab10:

A WiFi temperature monitor
===========================

ESPB266 WIFI module
----------------------

Purpose
^^^^^^^^

* To learn how to build a wireless sensor terminal based on the |embarc| package
* To know how to use ESP8266 module and AT commands
* To learn more about the usage of FreeRTOS operating system

Equipment
^^^^^^^^^^
The following hardware and tools are required:

* PC host
* |arcgnu| / |mwdt|
* ARC board (|emsk| / |iotdk|)
* |embarc| package
* ``embarc_osp/arc_labs/labs/lab10_esp8266_wifi``

Content
^^^^^^^^

Through this lab, you get a preliminary understanding of ESP8266 WIFI module and the AT command.

The lab is based on the |embarc| package and on the supports of the popular WIFI module, ESP8266.
During the lab, you will first use the AT command to set the ESP8266 to the server mode.
Then you can use your labtop or mobile phone to access ESP8266 by IP address.
You will get a static webpage transmitted via TCP protocol.


Principles
^^^^^^^^^^^^

**ESP8266**

The ESP8266 is an ultra-low-power WIFI chip with industry-leading package size and ultra-low power technology.
It is designed for mobile devices and IoT applications, facilitating the connection between user devices to IoT environments.

The ESP8266 is available with various encapsulations. Onboard PCB antenna, IPEX interface, and stamp hole interface are supported.

ESP8266 can be widely used in smart grid, intelligent transportation, smart furniture, handhold devices, industrial control, and other IoT fields.

Ai-Thinker company has developed several WIFI modules based on ESP8266, including ESP01 and ESP01S which will be used in this lab.

.. note::  See `embARC doc <http://embarc.org/embarc_osp/doc/build/html/getting_started/peripheral_preparation.html#other-pmod-or-compatible-modules>`_ to learn how to connect it with your board.

|figure1|

**Program structure** is shown below

|figure2|

**Code** is shown below

.. code-block:: c

    #include "embARC.h"
    #include "embARC_debug.h"

    #include "board.h"
    //#include "dev_uart.h"
    #include "esp8266.h"

    #include <stdio.h>
    #include <string.h>

    #define WIFI_SSID	"\"embARC\""
    #define WIFI_PWD	"\"12345678\""

    static char http_get[] = "GET /";
    static char http_IDP[] = "+IPD,";
    static char http_html_header[] = "HTTP/1.x 200 OK\r\nContent-type: text/html\r\n\r\n";
    static char http_html_body_1[] = "<html><head><title>ESP8266_AT_HttpServer</title></head><body><h1>Welcome to this Website</h1>";
    static char http_html_body_2[] = "<p>This Website is used to test the AT command about HttpServer of ESP8266.</p></body></html>";

    int main(void)
    {
            char *conn_buf;
            char scan_result[1024];

            //ESP8266 Init
            EMBARC_PRINTF("============================ Init ============================\n");

            ESP8266_DEFINE(esp8266);
            esp8266_init(esp8266, UART_BAUDRATE_115200);
            at_test(esp8266->p_at);
            board_delay_ms(100, 1);

            //Set Mode
            EMBARC_PRINTF("============================ Set Mode ============================\n");

            esp8266_wifi_mode_get(esp8266, false);
            board_delay_ms(100, 1);
            esp8266_wifi_mode_set(esp8266, 3, false);
            board_delay_ms(100, 1);

            //Connect WiFi
            EMBARC_PRINTF("============================ Connect WiFi ============================\n");

            do
            {
                    esp8266_wifi_scan(esp8266, scan_result);
                    EMBARC_PRINTF("Searching for WIFI %s ......\n", WIFI_SSID);
                    board_delay_ms(100, 1);
            }
            while (strstr(scan_result, WIFI_SSID) == NULL);

            EMBARC_PRINTF("WIFI %s found! Try to connect\n", WIFI_SSID);

            while(esp8266_wifi_connect(esp8266, WIFI_SSID, WIFI_PWD, false)!=AT_OK)
            {
                    EMBARC_PRINTF("WIFI %s connect failed\n", WIFI_SSID);
                    board_delay_ms(100, 1);
            }

            EMBARC_PRINTF("WIFI %s connect succeed\n", WIFI_SSID);

            //Creat Server
            EMBARC_PRINTF("============================ Connect Server ============================\n");

            esp8266_tcp_server_open(esp8266, 80);

            //Show IP
            EMBARC_PRINTF("============================ Show IP ============================\n");

            esp8266_address_get(esp8266);
            board_delay_ms(1000, 1);

            EMBARC_PRINTF("============================ while ============================\n");

            while (1)
            {
                    memset(scan_result, 0, sizeof(scan_result));
                    at_read(esp8266->p_at ,scan_result ,1000);
                    board_delay_ms(200, 1);
                    //EMBARC_PRINTF("Alive\n");

                    if(strstr(scan_result, http_get) != NULL)
                    {

                            EMBARC_PRINTF("============================ send ============================\n");

                            EMBARC_PRINTF("\nThe message is:\n%s\n", scan_result);

                            conn_buf = strstr(scan_result, http_IDP) + 5;
                            *(conn_buf+1) = 0;

                            EMBARC_PRINTF("Send Start\n");
                            board_delay_ms(10, 1);

                            esp8266_connect_write(esp8266, http_html_header, conn_buf, (sizeof(http_html_header)-1));
                            board_delay_ms(100, 1);

                            esp8266_connect_write(esp8266, http_html_body_1, conn_buf, (sizeof(http_html_body_1)-1));
                            board_delay_ms(300, 1);

                            esp8266_connect_write(esp8266, http_html_body_2, conn_buf, (sizeof(http_html_body_2)-1));
                            board_delay_ms(300, 1);

                            esp8266_CIPCLOSE(esp8266, conn_buf);

                            EMBARC_PRINTF("Send Finish\n");
                    }
            }

            return E_OK;
    }


Steps
^^^^^^^

**Hardware connection**
(as shown below)

|figure3|

**Compile and download**

Compile and download the program, after downloading successfully, the relevant download information is displayed in the command window(as shown in the following example).

.. code-block:: console

    0x00000004 in ?? ()
    Loading section .init, size 0x1b0 lma 0x10000000
    Loading section .vector, size 0x400 lma 0x10000400
    Loading section .text, size 0x1446c lma 0x10000800
    Loading section .rodata, size 0x1cb4 lma 0x10014c6c
    Loading section .data, size 0xc2c lma 0x10016920
    Start address 0x10000004, load size 94972
    Transfer rate: 602 KB/sec, 9497 bytes/write.
    Continuing.

At this point, feedback information will be shown on your serial port console, representing the process of the board establishing connection with http server with AT command (showing below).

.. code-block:: console

    embARC Build Time: Mar 21 2018, 17:53:27
    Compiler Version: ARC GNU, 7.1.1 20170710
    ============================ Init ============================
    [at_parser_init]56: obj->psio 0x1006ba30 -> 0x10057948

    .............

    OK" (9)
    ============================ Set Mode ============================
    [at_send_cmd]117: at_out: "AT+CWMODE_CUR?
    " (16)

    .................

    OK" (24)
    ============================ Connect WiFi ============================
    [at_send_cmd]117: at_out: "AT+CWLAP
    " (10)

    ..................

    OK" (24)
    [at_send_cmd]117: at_out: "AT+CWJAP_CUR="embARC_test","123456789"

    ..........

    WIFI "embARC_test" connect succeed
    ============================ Connect Server ============================
    [at_send_cmd]117: at_out: "AT+CIPMUX=1
    " (13)

    ........

    OK" (26)
    ============================ Show IP ============================
    [at_send_cmd]117: at_out: "AT+CIFSR
    " (10)
    [at_get_reply]137: "
    AT+CIFSR
    +CIFSR:STAIP,"192.168.137.81"
    +CIFSR:STAMAC,"5c:cf:7f:0b:5c:d1"

    OK" (83)
    ============================ while ============================
    .............
    ============================ send  ============================
    ..............

    Send Start
    Send Finish

**Access server**

The serial port feedback information above shows that the board has successfully connected to the target WIFI through ESP8266. It is set to the server mode by using the AT command, and the IP address of the server is also given.

At this point, use a PC or mobile phone to connect to the same WIFI, open a browser, and enter the IP address 192.168.137.81 to see the static HTTP page. Notice the IP address that you enter should be the same IP address shown in *Show IP* section at your serial port console.

Exercises
^^^^^^^^^^

Referring to the embARC documents, using ESP8266 and TCN75 temperature sensor to build http server to make the page display the sensor temperature in real time.

.. |figure1| image:: /img/lab10.2_figure1.png
.. |figure2| image:: /img/lab10.2_figure2.png
.. |figure3| image:: /img/lab10.2_figure3.png
