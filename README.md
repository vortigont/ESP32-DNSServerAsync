## ESP32 Async DNSServer

This is a forked version of ESP32's [bundled](https://github.com/espressif/arduino-esp32/tree/master/libraries/DNSServer) DNSServer.
Unlike bundled version based on [WiFiUDP](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFi) lib this fork was rewritten to use [AsyncUDP](https://github.com/espressif/arduino-esp32/tree/master/libraries/AsyncUDP) lib and is more efficient on using CPU/mem resources.

A PR espressif/arduino-esp32#7482 has been raised for the upstream.
Untill(unless) it accepted this repo will be a test-lab for developing. Feel free to use this lib to replace bundled ESP32's DNSServer.


====

Bundled implementation of [DNSServer](https://github.com/espressif/arduino-esp32/tree/master/libraries/DNSServer) lib for ESP32 uses WiFiUDP which is not very efficient considering cpu/mem resources. It hooks to loop() for packet processing doing useless malloc's/free *each* run cycle, even when no requests are received.

https://github.com/espressif/arduino-esp32/blob/master/libraries/WiFi/src/WiFiUdp.cpp#L205-L221

```C++
int WiFiUDP::parsePacket(){
// skipped
  char * buf = new char[1460];
  if ((len = recvfrom(udp_server, buf, 1460, MSG_DONTWAIT, (struct sockaddr *) &si_other, (socklen_t *)&slen)) == -1){
    delete[] buf;
    return 0;
  }
```

This PR offers refactored DNSServer code based on bundled AsyncUDP lib along with other optimizations and enhansments.
- got rid of intermediate mem buffers and extra data copies in DNSServer code
- added sanity checks for mem bounds
- optimize label/packet length calculations
- removed some dynamically allocated members
- refactored "Captive Portal" example, now it will display portal detection pop-up on Android, Windows, etc...
- other code cleanup


Other known issues:

 - non A req's ain't handled properly, i.e. always reply with A type records
 - pkts with edns opt ain't handled properly, packets ignored due to additional RR's>1
