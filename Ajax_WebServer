/*
  Ajax Web  Server
 
 A simple web server that shows the value of the analog input pins.
 using an Arduino Wiznet Ethernet shield. 
 
 Circuit:
 * Ethernet shield attached to pins 10, 11, 12, 13
 * Analog inputs attached to pins A0 through A5 (optional)
 
 created 18 Dec 2009
 by David A. Mellis
 modified 4 Sep 2010
 by Tom Igoe

 - - - - - - - - - - - - - - - - - - - - - - - - 
 modified 16 Sep 2011 // Ajax version.
 by TETRASTYLE 
 for Arduino IDE 0022

 get Digital & Analog port status ( JSON(P) formatted data)
  <-- http://192.168.1.177/?=

  return value
  --> cb({"D":[D0,D1,D2,D3,D4,D5,D6,D7],"A":[A0,A1,A2,A3,A4,A5]});

 toggle Digital Port (D0 - D7) 
  <-- http://192.168.1.177/?btn=0
  <-- http://192.168.1.177/?btn=1
      ...
  <-- http://192.168.1.177/?btn=7

*/

#include <SPI.h>
#include <Ethernet.h>

boolean gflag=false;
String parm;
int dout[8]={0,0,0,0,0,0,0,0};

// Enter a MAC address and IP address for your controller below.
// The IP address will be dependent on your local network:
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
byte ip[] = { 192, 168, 1, 177 };

// Initialize the Ethernet server library
// with the IP address and port you want to use 
// (port 80 is default for HTTP):
Server server(80);

void setup()
{
  for (int i=0;i<8;i++){
    pinMode(i,OUTPUT);
    digitalWrite(i,dout[i]);
  }
  
  // start the Ethernet connection and the server:
  Ethernet.begin(mac, ip);
  server.begin();
  //Serial.begin(19200);
}

void loop()
{
  // listen for incoming clients
  Client client = server.available();
  if (client) {
    // an http request ends with a blank line
    boolean currentLineIsBlank = true;
    
    while (client.connected()) {
      if (client.available()) {
        char c = client.read();

        // serch parameter from "HTTP GET"
        if(gflag){
          if(c != ' '){
              parm += c;
          }else{
              gflag = false;
          }
        }
        if(c == '?' && parm == ""){
          gflag = true;
        }
        //Serial.print(c);


        // if you've gotten to the end of the line (received a newline
        // character) and the line is blank, the http request has ended,
        // so you can send a reply

        if (c == '\n' && currentLineIsBlank) {
          // send a standard http response header

         if(parm == ""){ // normal HTTP access
            client.println("HTTP/1.1 200 OK");
            client.println("Content-Type: text/html");
            client.println();

            client.println("<html><head><title>Hello Arduino</title>");
            client.println("<meta name='viewport' content='width=240px' />");
            
            client.println("<script type='text/javascript'>");

            client.println("function createXMLHttpRequest(cbFunc){");
            client.println("  var XObj = new XMLHttpRequest();");
            client.println("  if(XObj) XObj.onreadystatechange = cbFunc;return XObj;}");

            client.println("function setData(val){htObj = createXMLHttpRequest(displayData);");
            client.println("  if(htObj){htObj.open('GET','/?btn='+val,true);htObj.send(null);}}");

            client.println("function getData(){htObj = createXMLHttpRequest(displayData);");
            client.println("  if(htObj){htObj.open('GET','/?=',true);htObj.send(null);}}");

            client.println("function displayData(){ if((htObj.readyState == 4) && (htObj.status == 200)){");
            client.println("  document.getElementById('result').innerHTML =  htObj.responseText;}}");

            client.println("function strT(){ getData(); timerID=setTimeout('strT()',");
            client.println("  document.getElementById('tf1').value);}"); 

            client.println("function clrT(){clearTimeout(timerID);}");
            
            client.println("</script></head><body onLoad='getData()'><form action='/' method='GET'><br />");

            client.println("<input id='btn0' type='button' value=' D0 ' onClick='setData(0)'>");
            client.println("<input id='btn1' type='button' value=' D1 ' onClick='setData(1)'>");
            client.println("<input id='btn2' type='button' value=' D2 ' onClick='setData(2)'>");
            client.println("<input id='btn3' type='button' value=' D3 ' onClick='setData(3)'><br /><br />");
            client.println("<input id='btn4' type='button' value=' D4 ' onClick='setData(4)'>");
            client.println("<input id='btn5' type='button' value=' D5 ' onClick='setData(5)'>");
            client.println("<input id='btn6' type='button' value=' D6 ' onClick='setData(6)'>");
            client.println("<input id='btn7' type='button' value=' D7 ' onClick='setData(7)'><br /><br />");

            client.println("</form><form><input id='btn100' type='button' value=' START ' onClick='strT()'>");
            client.println("<input id='btn200' type='button' value=' STOP ' onClick='clrT()'>");
            client.println("<input id='tf1' type='text' size='6' value='1000'>");
            client.println("</form><br /><div id='result'></div>");

            client.println("<br /></body></html>");

         }else{ // using XMLhttpObject access
  
            int check = parm.indexOf('=');
            if(check != -1){
              //Set Digital Port
              int check2 = parm.indexOf('btn');
              if(check2 != -1){
                int port = (parm.substring(check+1)).toInt();
                dout[port] = !dout[port];
                digitalWrite(port, dout[port]);
              }

             //Write JSONP Data (Digital & Analog Ports Status, Callback function name is 'cb')
               client.print("cb({\"D\":[");
                 for(int i=0; i<8; i++ ){
                   client.print(dout[i]);
                   if(i<7){
                     client.print(",");
                   }
                 }
               client.print("],\"A\":[");
                 for(int i=0; i<6; i++ ){
                   client.print(analogRead(i));
                   if(i<5){
                     client.print(",");
                   }
                 } 
               client.print("]});");
              
            }
            parm = "";
          }


          break;
        }


        if (c == '\n') {
          // you're starting a new line
          currentLineIsBlank = true;
        } 
        else if (c != '\r') {
          // you've gotten a character on the current line
          currentLineIsBlank = false;
        }
      }
    }
    // give the web browser time to receive the data
    delay(1);

    // close the connection:
    client.stop();
  }
}

