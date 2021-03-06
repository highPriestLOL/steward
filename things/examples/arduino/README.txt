README.txt

The TSRP examples

EggShield/
GroveAQ/
PressureMat/
WaterSensor/
WeatherStation/

all make use of multicast UDP. This isn't supported by the standard Ethernet library, you'll need to patch the Ethernet library as below. Add the following to EthernetUdp.h directly after line 55,

virtual uint8_t beginMulti(IPAddress, uint16_t);	// initialize, start listening on specified port. Returns 1 if successful, 0 if there are no sockets available to use

then add the following code to the end of EthernetUdp.cpp,

/* Start EthernetUDP socket, listening at local port PORT */
uint8_t EthernetUDP::beginMulti(IPAddress ip, uint16_t port) {
    Serial.println("beginMulti()");
    if (_sock != MAX_SOCK_NUM)
        return 0;
    
    for (int i = 0; i < MAX_SOCK_NUM; i++) {
        uint8_t s = W5100.readSnSR(i);
        if (s == SnSR::CLOSED || s == SnSR::FIN_WAIT) {
            _sock = i;
            break;
        }
    }
    
    if (_sock == MAX_SOCK_NUM)
        return 0;
    
    
    // Calculate MAC address from Multicast IP Address
    byte mac[] = {  0x01, 0x00, 0x5E, 0x00, 0x00, 0x00 };
    
    mac[3] = ip[1] & 0x7F;
    mac[4] = ip[2];
    mac[5] = ip[3];
    
    W5100.writeSnDIPR(_sock, rawIPAddress(ip));   //239.255.0.1
    W5100.writeSnDPORT(_sock, port);
    W5100.writeSnDHAR(_sock,mac);
    
    _remaining = 0;
    
    socket(_sock, SnMR::UDP, port, SnMR::MULTI);
    
    return 1;
}

you shouldn't need to restart the Arduino IDE, the changes will be included next time you compile a sketch.