## ESP32 Async DNSServer

This is a forked version of ESP32's [bundled](https://github.com/espressif/arduino-esp32/tree/master/libraries/DNSServer) DNSServer.
Unlike bundled version based on [WiFiUDP](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFi) lib this fork was rewritten to use [AsyncUDP](https://github.com/espressif/arduino-esp32/tree/master/libraries/AsyncUDP) lib and is more efficient on using CPU/mem resources.

A PR espressif/arduino-esp32#7482 has been raised for upstream.
Untill(unless) it accepted this repo will be a test-lab for developing.


====
Following bugfix #7475. Current implementation of DNSServer lib uses WiFiUDP which is not very efficient considering cpu/mem resources. It hooks to loop() for packet processing doing useless malloc's/free each run cycle.

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

This PR offers refactored DNSServer code based on bundled AsyncUDP lib and some other optimizations.
- got rid of intermediate mem buffers and extra data copies in DNSServer code
- added sanity checks for mem bounds
- optimize label/packet length calculations
- removed some dynamically allocated members
- other code cleanup

Other known issues:

    non A req's ain't handled properly, i.e. always reply with A type records
    pkts with edns opt ain't handled properly, packets ignored due to additional RR's>1

