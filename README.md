# Esp32-Wifi-Beacon
<h1 text-align:center>A raw code for normal esp32 devkit v1</h1>


<p>Should be using this kind of esp32 wroom</p>
<img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTx4GqH1i-pfCjOJqTg6NuWEV5myG7zQKYqGGWMHKEcvg&s=10" width="400">

<h1 text-align:center>Raw code for arduino IDE:</h1>

<p>

#include <WiFi.h>
#include <esp_wifi.h>

const char *BASE_SSID = "Joey was here";
#define NUM_SSIDS 500
#define CHANNEL 1

uint8_t packet[128];
char ssidList[NUM_SSIDS][33];
uint8_t macList[NUM_SSIDS][6];

void setup() {
  Serial.begin(115200);
  delay(500);
  randomSeed(esp_random());

  for (int i = 0; i < NUM_SSIDS; i++) {
    snprintf(ssidList[i], 33, "%s %d", BASE_SSID, i);
    for (int m = 0; m < 6; m++) macList[i][m] = random(256);
    macList[i][0] = (macList[i][0] & 0xFC) | 0x02;
    macList[i][5] = i & 0xFF;
    macList[i][4] = (i >> 8) & 0xFF;
  }

  WiFi.mode(WIFI_MODE_STA);
  esp_wifi_set_promiscuous(true);
  esp_wifi_set_channel(CHANNEL, WIFI_SECOND_CHAN_NONE);
  esp_wifi_set_max_tx_power(78);
  esp_wifi_start();
  esp_wifi_set_promiscuous(true);

  Serial.println("Persistent beacon spam started...");
}

void sendBeacon(int idx) {
  const char *ssid = ssidList[idx];
  uint8_t *mac = macList[idx];

  int ssidLen = strlen(ssid);
  if (ssidLen > 32) ssidLen = 32;

  int i = 0;

  // 802.11 beacon frame — no radiotap header
  packet[i++] = 0x80; packet[i++] = 0x00;              // frame control: beacon
  packet[i++] = 0x00; packet[i++] = 0x00;              // duration
  for (int m = 0; m < 6; m++) packet[i++] = 0xFF;      // dest: broadcast
  for (int m = 0; m < 6; m++) packet[i++] = mac[m];    // src
  for (int m = 0; m < 6; m++) packet[i++] = mac[m];    // BSSID
  packet[i++] = 0x00; packet[i++] = 0x00;              // seq/frag

  // Timestamp
  for (int t = 0; t < 8; t++) packet[i++] = 0x00;

  // Beacon interval + capability
  packet[i++] = 0x64; packet[i++] = 0x00;
  packet[i++] = 0x01; packet[i++] = 0x04;

  // SSID tag
  packet[i++] = 0x00; packet[i++] = ssidLen;
  for (int s = 0; s < ssidLen; s++) packet[i++] = (uint8_t)ssid[s];

  // Supported rates
  packet[i++] = 0x01; packet[i++] = 0x01; packet[i++] = 0x82;

  // DS parameter set (channel)
  packet[i++] = 0x03; packet[i++] = 0x01; packet[i++] = CHANNEL;

  esp_wifi_80211_tx(WIFI_IF_STA, packet, i, false);
}

void loop() {
  for (int i = 0; i < NUM_SSIDS; i++) {
    sendBeacon(i);
  }
  delay(1);
}
  
</p>





