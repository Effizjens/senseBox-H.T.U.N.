<img src="https://github.com/sensebox/OER/blob/master/senseBox_edu/images/sensebox_logo_neu.png" width="200"/>

# sensebox H.T.U.N.
Unsere senseBox bauen wir so auf, dass sie verschiedene Wetterdaten Bei openSenseMap hoch lädt.

## Ziel
Wir messen neben Feuchtigkeit (humidity), Temperatur (temperatur) und UV-Strahlung in Standard-Skalen auch die Lautstärke (noise) mangels genauer Umrechnungsvorgeben, mit unserer eigenen Funktion die an die Decibelskala angelenht ist und der Einfachheit halber von uns auch genauso benannt wurde. Wir interessieren uns für sämtliche Daten die unsere Sensoren aufnehmen, insbesondere die Lautstärke welche in Nähe von Studentenwohnungen interessante Daten zu deren Verhalten ergeben könnten.

## Materialien
#### Aus der senseBox:edu
* Genuino UNO
* Temperatur und Feuchtigkeitssensor Watterott HDC1008
* UV-Intesitätssensor Watterott VEML6070
* Mic-Breakout

#### Zusätzliche Hardware
* Breadbord
* Kabel in verschiedenen Farben
* Soundsensor Sparkfire ADMP 401
* W5500 Ethernetshield V1.0
* WetterBox der Firma FIBOX
* Panzertape und Heißkleber

## Setup Beschreibung
#### Hardwarekonfiguration

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/IMG-20160417-WA0005.jpg" width="400"/>

Die Verkabelung entspricht dem Fritzingschema, da alle Chips 3.3V Eingangspannung haben bietet es sich an alle über diese Leiste mit Energie zu versorgen, die Masse wird genauso eingesetzt. Zu beachten ist ebenfalls das Sowohl UV-Werte Als auch Temperatur und Feuchtigkeit über den selben Pin ausgeben.

Die Fibox Schutzhülle bereiten wir auf den Einsatz der Sensoren vor, indem wir zunächst Öffnungen für Kabel, einmal Netzwerk und Stromversorgung zum anderen für alle Sensoren bis auf den UV-Chip.

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_110146.jpg" width="400"/>  <img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_110135.jpg" width="400"/>  <img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_111319.jpg" width="400"/>

Der Temperatur und Feuchtigkeitssensor wird zusammen mit den beiden Microphonen in ein externes Fach, dass mit Heißkleber an die "Fibox" angebracht wird, verpackt. Das Fach wird stärker der Witterung ausgesetzt sein als das Innere der "Fibox" um genaure Daten zu gewinnen.

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_112654.jpg" width="400"/>

Den UV-Sensor befestigen wir mit einem Adapterstück und etwas Heißkleber einfach direkt unter der transparenten Fibox Scheibe.

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_110129.jpg" width="400"/>

Wichtig ist, dass wir die Wetterbedingungen beim Aufbau der senseBox beachten, die Sensoren sollten sowohl vor Regen als auch Kondenzwasser geschützt sein, zeitgleich aber so offen liegen, dass Messwerte nicht verfälscht werden.

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_112634.jpg" width="400"/>  <img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_112643.jpg" width="400"/>  <img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_120958.jpg" width="400"/>

#### Softwaresketch


##### Links zu zusätzlichen Bibliotheken:
Die Bibliotheken zum Temperatur- und Feuchtigkeitssensor findet ihr hier: 
https://github.com/ftruzzi/HDC1000-Arduino.

die wir in einer Samlung entweder schon im Ordner (...)\Arduino\libraries findet oder von der github Website https://github.com/watterott/HDC100X-Breakout. Ansonsten lassen sich die Librarys auch prima über https://google.com finden!

Den Code für das Ethernetshield bekommt ihr personalisiert von openSenseMap per E-mail zugeschickt.


#### Der Sketch
Hier erklären wir euch die eigentliche Software, die im nachhinein dafür sorgt, dass die Impulse auch tatsächlich zu Messwerten werden.

Der Start von Librarys, über die Schaltflächen "Sketch" -> "Bibliothek einbinden"
``` c
#include <SPI.h>
#include <Ethernet.h>
#include <Wire.h>
#include <HDC100X.h>
```
Wir Deklarieren unsere Sensorwerte
```c
float humi = 0;   //Luftfeuchtigkeit
float temp = 0;   //Temperatur
float uv = 0;     //UV-Wert
float db1 = 0;    //Mikro 1
float db2 = 0;    //Mikro 2
```
Deklaration der Ein- und Ausgänge beziehungsweise der Kabelverbindungen auf dem Genuino Uno mit den Sensoren.
```c
HDC100X HDC(0x43);		// 

#define I2C_ADDR 0x38
//Integration Time
#define IT_1_2 0x0 //1/2T
#define IT_1   0x1 //1T
#define IT_2   0x2 //2T
#define IT_4   0x3 //4T
#include<SHT1x.h>
#define dataPin 6
#define clockPin 7
SHT1x sht1x(dataPin, clockPin);
```

Einige der Zeilen die ihr per E-mail von openSenseMap bekommt.
wir erkennen das die Sensorwerte an die ID´s gebunden werden. Ausserdem wird das Ethernetshield konfiguriert.
```c
//SenseBox ID
#define SENSEBOX_ID "57065ef945fd40c81974c696"

//Sensor IDs
#define TEMPSENSOR_ID "57065ef945fd40c81974c69c"  //Temperatur
#define SENSOR1_ID "57065ef945fd40c81974c69b" // Luftfeuchtigkeit 
#define UVSENSOR_ID "57065ef945fd40c81974c69a"// UV
#define SENSOR2_ID "57065ef945fd40c81974c699" // Schall 
#define SENSOR3_ID "57065ef945fd40c81974c698" // Schall 

//Ethernet-Parameter
char server[] = "www.opensensemap.org";
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
// Diese IP Adresse nutzen falls DHCP nicht möglich
IPAddress myIP(192, 168, 0, 42);
EthernetClient client;

//Messparameter
int postInterval = 10000; //Uploadintervall in Millisekunden
long oldTime = 0;

```
Im Setup starten wir Routinen um uns mit openSenseMap zu verbinden, der Code ist Inhalt der openSenseMap-Mail. 
```c
void setup()
{
  Serial.begin(9600);
  Serial.print("Starting network...");
  //Ethernet Verbindung mit DHCP ausführen..
  if (Ethernet.begin(mac) == 0)
  {
    Serial.println("DHCP failed!");
    //Falls DHCP fehltschlägt, mit manueller IP versuchen
    Ethernet.begin(mac, myIP);
  }
  
  HDC.begin(HDC100X_TEMP_HUMI, HDC100X_14BIT, HDC100X_14BIT, DISABLE);
  
  Wire.begin();
  Wire.beginTransmission(I2C_ADDR);
  Wire.write((IT_1 << 2) | 0x02);
  Wire.endTransmission();
  
  delay(500);
  Serial.println("done!");
  delay(1000);
  Serial.println("Starting loop.");
}
```
Im Loop laufen die Messroutinen ab, die Frequenz haben wir oben am Ehternetshield definiert (postInterval). Ausserdem Laden wir die Daten gesammelt in die "upload()" Funktion um sie später in die openSenseMap hoch zu laden.
```c
void loop()
{
  //Upload der Daten mit konstanter Frequenz
  if (millis() - oldTime >= postInterval)
  {
    oldTime = millis();
    uv = uvf(); 
    //uvf Funktion misst Werte, UV-Sensors, und korrigiert sie.
    humi = HDC.getHumi(); //HDC.getHumi() gibt Luftfeuchtigkeit zurück
    temp = HDC.getTemp(); //HDC.getTemp() gibt Temperatur zurück
    db1 = miccalc(analogRead(A3)); //dB vom Sensor 1
    db2 = miccalc(analogRead(A1)); //dB vom Sensor 2
    upload(); //Lädt gesammelte Daten hoch
  }
}
```
Die Funktion für den UV-Sensor, der nicht direkt UV-Werte sondern Stromwiederstände abhängig von der UV-Strahlung ausgibt. Ausserdem laufen Temperatur und Feuchtigkeitssensor und UV-Sensor über den selben Input, die "Wire.aviable()" funktion sowie das "delay(1)" sorgen dafür, dass die Werte nicht vertauscht werden 
```c
float uvf() {
  float uvf = 0;
  byte msb = 0, lsb = 0;
  Wire.requestFrom(I2C_ADDR + 1, 1); //MSB
  delay(1);
  if (Wire.available())
    msb = Wire.read();
  Wire.requestFrom(I2C_ADDR + 0, 1); //LSB
  delay(1);
  if (Wire.available())
    lsb = Wire.read();
  uvf = (msb << 8) | lsb;
  uvf = uvf * 5.625;
  return uvf;
}
```
###### Unsere improvisierte Funktion zur Umrechnung der Mikrofonsignale.

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_093116.jpg" width="400"/>  <img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_093106.jpg" width="400"/>  <img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/20160408_093058.jpg" width="400"/>

Nach einigen Testläufen mit einem Geeichten Decibelmesser haben wir eine Regelmäßigkeit herausgefunden nämlich, dass Geräusche ab 60db sowohl nach oben wie unten entlang von 355 ausschlagen, dementsprechend haben wir unsere Funktion gebaut. Genau sind diese Werte aber nicht!

```c
float miccalc(int x) {
  int j;
  if (x < 335) {
    x = 335 + (335 - x);    //Spiegeln an 335
  }
  x = x - 335;              //335 wird auf 0 gesetzt, entspricht ~60 dB
  if (x <= 10) {            //Sonderfall für 60-70 dB
    x = x + 60;
  } else {
    for (int i = 0; x > (pow(2, i) * 10); i++) {  
    //Durchläuft, bis x in einem Bereich liegt,
      x = x - pow(2, i) * 10;                     
      //der einen 10er Bereich im dB bereich liegt
      j++;
    }
    x = map(x, 0, pow(2, j) * 10, ((j + 6) * 10), ((j + 7) * 10));  
    //Überträgt diese beiden Bereiche aufeinadner
  }
  return x;
}
```
Noch mehr Code zum Upload in die openSenseMap, beinhaltet unter anderem Abfragen zum Verbindungsstatus, und Befehle zum Ausgeben der Daten.
```c
void postFloatValue(float measurement, int digits, String sensorId)
{
  //Float zu String konvertieren
  char obs[10];
  dtostrf(measurement, 5, digits, obs);
  //Json erstellen
  String jsonValue = "{\"value\":";
  jsonValue += obs;
  jsonValue += "}";
  //Mit OSeM Server verbinden und POST Operation durchführen
  Serial.println("-------------------------------------");
  Serial.print("Connectingto OSeM Server...");
  if (client.connect(server, 8000))
  {
    Serial.println("connected!");
    Serial.println("-------------------------------------");
    //HTTP Header aufbauen
    client.print("POST /boxes/"); client.print(SENSEBOX_ID); client.print("/"); client.print(sensorId); client.println(" HTTP/1.1");
    client.println("Host: www.opensensemap.org");
    client.println("Content-Type: application/json");
    client.println("Connection: close");
    client.print("Content-Length: "); client.println(jsonValue.length());
    client.println();
    //Daten senden
    client.println(jsonValue);
  } else
  {
    Serial.println("failed!");
    Serial.println("-------------------------------------");
  }
  //Antwort von Server im seriellen Monitor anzeigen
  waitForServerResponse();
}

void waitForServerResponse()
{
  //Ankommende Bytes ausgeben
  boolean repeat = true;
  do {
    if (client.available())
    {
      char c = client.read();
      Serial.print(c);
    }
    //Verbindung beenden
    if (!client.connected())
    {
      Serial.println();
      Serial.println("--------------");
      Serial.println("Disconnecting.");
      Serial.println("--------------");
      client.stop();
      repeat = false;
    }
  } while (repeat);
}
```
Setzt unsere Werte in die hochzuladenden Funktionen vom Anfang ein.
```c
void upload() {
  postFloatValue(humi, 2, SENSOR1_ID);
  postFloatValue(temp, 2, TEMPSENSOR_ID);
  postFloatValue(uv, 2, UVSENSOR_ID);
  postFloatValue(db1, 2, SENSOR2_ID);
  postFloatValue(db2, 2, SENSOR3_ID);
}
```



## openSenseMap Registrierung

Für die Registstrierung bei openSenseMap.org wurden die Bildschirmanweisungen auf openSenseMap.org befolgt. Nachdem man seinen Namen, Mail-Adresse, Name und Aufstellungsort, sowie das Hardware Setup, inklusive Einheiten angegeben hat, erhält man eine eMail. In dieser ist das senseBox Skript enthalten, dass schon das eingene Hardware Setup enthält, sowie den Code zum hochladen auf openSenseMap.org. Wenn man, wie im Beispiel gezeigt, für jede Messung eine eigene Funktion anlegt und auch eine Upload Funktion erstellt, sollte es kein Problem sein, das senseBox Skript mit dem eingenen zu Verbinden, damit die gemessenen Daten erfolgreich hochgeladen werden können.

 

## Stationsaufbau

Die Wetterstation wurde draußen an einem Geländer befestigt, dieses Geländer zeigt Richtung Südosten. Um die Stabilität zu gewährleisten,  wurde die Box mit Kabelbindern befestigt und auch das LAN-Kabel wurde an einer der Streben befestigt, das allerdings nur aus ästhetischen Gründen. Der genaue Standort ist auf der [OpensenseMap](http://opensensemap.org/#/explore/57065ef945fd40c81974c696) einzusehen.

<img src="https://github.com/Invisible619/Sensebox-H.T.U.N./blob/master/IMG-20160417-WA0002.jpg" width="400"/>
