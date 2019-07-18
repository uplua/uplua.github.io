---
layout: post
title: "Sixth Week Report"
date: 2019-07-18 12:49:56 +0530
categories: jekyll update
---

### NodeMCU as HTTP Server using Wi-Fi STA mode

NodeMCU has Station (STA) mode using which it can connect to existing wi-fi network and can act as HTTP server with IP address assigned by that network.

![NodeMCU STA HTTP Server](images/NodeMCU_WiFi_STA_Mode.png)

NodeMCU gets an IP from Wi-Fi Router to which it is connected. With this IP address, it can act as an HTTP server to which any wi-fi device can connect.

#### Example

Let's write a Lua Script to enable NodeMCU as HTTP server with Wi-Fi STA/AP mode and control an LED connected at server side from client side.

Here we have connected LED to the pin no. 2 i.e. D2 pin on NodeMCU board as shown in  the figure.

![NodeMCU_Circuit](images/HTTP_server_with_nodeMCU.png)

#### HTML page for client

As we are making HTTP server for LED on/off functionality, we are going to make simple HTML page which will be visible at client side and able to take user input for LED on/off. It is user friendly representation of button input which takes input from user on click.

We need to write two HTML pages for LED ON and LED OFF state i.e. when client clicks the LED ON button, then in next action we need to provide option for LED OFF. Below are the two HTML code snippets for LED ON and LED OFF state presentation.

#### HTML Code Snippet for LED ON

```
<!DOCTYPE html>
<html>
<head><title>LED Control</title></head>
<body>
<h1>LED</h1>
<p>Click to switch LED on and off.</p>
<form method="get">
<input type="button" value="LED ON" onclick="window.location.href='/ledon'">
</form>
</body>
</html>

```

#### HTML Code Snippet for LED OFF

```
<!DOCTYPE html>
<html>
<head><title>LED Control</title></head>
<body>
<h1>LED</h1>
<p>Click to switch LED on and off.</p>
<form method="get">
<input type="button" value="LED OFF" onclick="window.location.href='/ledoff'">
</form>
</body>
</html>

```

From above two HTML snippets we can see that only forms are different for LED ON and LED OFF state.

Let's look at HTML lines:

<strong><em> !Doctype html : </em></strong> This declaration defines that document as an HTML and helps browsers to display web pages correctly. It must appear once, at the top of the page.

<strong><em> html : </em></strong> This is the root element of HTML page.

<strong><em> head : </em></strong> This element contains meta information about the document.

<strong><em> title : </em></strong> This element specifies a title for the document.

<strong><em> body : </em></strong> This element contains the visible page content i.e. body of document.

<strong><em> h1 : </em></strong> This element defines the largest font size for heading. similarly, we can use h2, h3 and so on for smaller font sizes of the header.

<strong><em> p : </em></strong> This element defines a paragraph.

<strong><em> form : </em></strong> This element defines a form that is used to collect user input.

<strong><em> window.location.href : </em></strong> This is a property that will tell us the current URL location. Changing the value of the property will redirect the page.

e.g. ``` window.location.href='/ledon' ``` will redirect the current page to the URL ``` current_url/ledon ``` page.If current location is ``` http://192.168.0.1 ``` then it will redirect to ``` http://192.168.0.1/ledon ```. Page redirect action is taken on click event ( e.g. click on button ).

Here, we are using above mentioned concept (page redirect) to redirect the client from LED ON page to LED OFF page and vice versa.

Now, we can send above HTML snippets when client connects to the server and also when client clicks on button.

#### Program

In Wi-Fi Access Point mode, NodeMCU creates server hence we can set its IP address, IP subnetmask and IP gateway.

Let's take below SSID, Password to join network and addresses for AP mode.
* SSID = "NodeMCU"
* Password = "123456789"
* IP = "192.168.2.1"
* Subnet mask = "255.255.255.0"
* Gateway = "192.168.2.1"

#### Lua Script for HTTP server with wi-fi AP mode

```
wifi.setmode(wifi.SOFTAP)   -- set AP parameter
config = {}
config.ssid = "NodeMCU"
config.pwd = "12345678"
wifi.ap.config(config)

config_ip = {}  -- set IP,netmask, gateway
config_ip.ip = "192.168.2.1"
config_ip.netmask = "255.255.255.0"
config_ip.gateway = "192.168.2.1"
wifi.ap.setip(config_ip)

LEDpin = 2                  -- declare LED pin
gpio.mode(LEDpin, gpio.OUTPUT)
server = net.createServer(net.TCP)-- create TCP server

function SendHTML(sck, led) -- Send LED on/off HTML page
   htmlstring = "<!DOCTYPE html>\r\n"
   htmlstring = htmlstring.."<html>\r\n"
   htmlstring = htmlstring.."<head>\r\n"
   htmlstring = htmlstring.."<title>LED Control</title>\r\n"
   htmlstring = htmlstring.."</head>\r\n"
   htmlstring = htmlstring.."<body>\r\n"
   htmlstring = htmlstring.."<h1>LED</h1>\r\n"
   htmlstring = htmlstring.."<p>Click to switch LED on and off.</p>\r\n"
   htmlstring = htmlstring.."<form method=\"get\">\r\n"
  if (led)  then
   htmlstring = htmlstring.."<input type=\"button\" value=\"LED OFF\" onclick=\"window.location.href='/ledoff'\">\r\n"
  else
   htmlstring = htmlstring.."<input type=\"button\" value=\"LED ON\" onclick=\"window.location.href='/ledon'\">\r\n"
  end
   htmlstring = htmlstring.."</form>\r\n"
   htmlstring = htmlstring.."</body>\r\n"
   htmlstring = htmlstring.."</html>\r\n"
   sck:send(htmlstring)
end

function receiver(sck, data)-- process callback on recive data from client
  if string.find(data, "GET /ledon")  then
   SendHTML(sck, true)
   gpio.write(LEDpin, gpio.HIGH)
  elseif string.find(data, "GET / ") or string.find(data, "GET /ledoff") then
   SendHTML(sck, false)
   gpio.write(LEDpin, gpio.LOW)
  else
   sck:send("<h2>Not found...!!</h2>")
   sck:on("sent", function(conn) conn:close() end)
  end
end

if server then
  server:listen(80, function(conn)-- listen to the port 80
  conn:on("receive", receiver)
  end)
end

```
<strong>Note :</strong> After successful uploading of above script, client needs to connect to the network created by NodeMCU first.
After connecting to NodeMCU network from wifi, enter the server address in browser i.e. ``` http://server_ip_address ``` e.g. in our case it is ``` http://192.168.2.1 ```. After pressing Enter Key, we can see the HTML page response from server as shown in below image:

![LED_State](images/led_state.png)

Now, let's do the HTTP server to NodeMCU using Wi-Fi station mode.

IN Wi-Fi Station mode (STA), NodeMCU gets IP addresses from Wi-Fi router (access point). If we are also in same network then we can directly connect to NodeMCU HTTP server using IP address only.

#### Lua Script for HTTP server with wi-fi STA mode

```
station_cfg={}
station_cfg.ssid="ssid"  -- Enter SSID here
station_cfg.pwd="password"  -- Enter password here


wifi.setmode(wifi.STATION)  -- set wi-fi mode to station
wifi.sta.config(station_cfg)-- set ssid&pwd to config
wifi.sta.connect()          -- connect to router

LEDpin = 2                  -- declare LED pin
gpio.mode(LEDpin, gpio.OUTPUT)
server = net.createServer(net.TCP)-- create TCP server

function SendHTML(sck, led) -- Send LED on/off HTML page
   htmlstring = "<!DOCTYPE html>\r\n"
   htmlstring = htmlstring.."<html>\r\n"
   htmlstring = htmlstring.."<head>\r\n"
   htmlstring = htmlstring.."<title>LED Control</title>\r\n"
   htmlstring = htmlstring.."</head>\r\n"
   htmlstring = htmlstring.."<body>\r\n"
   htmlstring = htmlstring.."<h1>LED</h1>\r\n"
   htmlstring = htmlstring.."<p>Click to switch LED on and off.</p>\r\n"
   htmlstring = htmlstring.."<form method=\"get\">\r\n"
  if (led)  then
   htmlstring = htmlstring.."<input type=\"button\" value=\"LED OFF\" onclick=\"window.location.href='/ledoff'\">\r\n"
  else
   htmlstring = htmlstring.."<input type=\"button\" value=\"LED ON\" onclick=\"window.location.href='/ledon'\">\r\n"
  end
   htmlstring = htmlstring.."</form>\r\n"
   htmlstring = htmlstring.."</body>\r\n"
   htmlstring = htmlstring.."</html>\r\n"
   sck:send(htmlstring)
end

function receiver(sck, data)-- process callback on recive data from client
  if string.find(data, "GET /ledon")  then
   SendHTML(sck, true)
   gpio.write(LEDpin, gpio.HIGH)
  elseif string.find(data, "GET / ") or string.find(data, "GET /ledoff") then
   SendHTML(sck, false)
   gpio.write(LEDpin, gpio.LOW)
  else
   sck:send("<h2>Not found...!!</h2>")
   sck:on("sent", function(conn) conn:close() end)
  end
end

if server then
  server:listen(80, function(conn)-- listen to the port 80
  conn:on("receive", receiver)
  end)
end

```
<strong>Note : </strong> In wi-fi station mode we need to enter the ssid and password of existing network. After connecting to WiFi network enter the server address in browser i.e. ``` http://assigned_ip_address ```. After pressing Enter Key we can see the HTML page response from server in the browser as shown above for AP mode.