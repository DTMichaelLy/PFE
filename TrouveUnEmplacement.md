# Code de localisation-Arduino 

#include <GSM.h>
#include <TinyGPS.h>

//creer les objects 
TinyGPS gps; 
GSM gsmAccess;
GSM_SMS sms;
GSMVoiceCall VC;

//si la carte sim as un mot de pass, entrer le dans PINNUMBER
#define PINNUMBER ""
//ajouter un numero si vous voulez envoyer les coordonnees a seulment un numero et commentaire la ligne en bas
char remoteNumber[20]=""; 

// creer les variables de latitude et de longitude 
String latitude, longitude;
float lat, lon; 

void setup(){
    Serial.begin(9600); // connect serial
    Serial.println("Bienvenue à notre projet!");
    Serial1.begin(9600); // connect GPS serial

  // connection state
  boolean notConnected = true;

    if (gsmAccess.begin(PINNUMBER) == GSM_READY) {
      notConnected = false;
      Serial.println("Le GSM est initialisé! Faire un appel pour recevoir les coordonnés!");
    } else {
      Serial.println("Le GSM n'est pas connecte...Verifie les fils");
      delay(1000);
    }
    }

//un boucle qui si repete plusieurs fois pour afficher la position et de verifier si on a une appel
void loop(){
     GetGPS(); //appel le fonction getGPS
     CheckCall();} //appel le fonction CheckCall

//si on a un appel, enregistre le numero et assigne le numero à remoteNumber  
//raccrocher l'appel et envoyer un texto
void CheckCall(){
  if (VC.getvoiceCallStatus()==RECEIVINGCALL){ 
      Serial.println('\n');
      Serial.println("Envoyer les coordonnés à ce numéro:");
      VC.retrieveCallingNumber(remoteNumber, 20);  
      Serial.println(remoteNumber);
      VC.hangCall();
      SendText();}
}

//fonction qui affiche la lat et la lon
void GetGPS(){
    if (Serial1.available()){ 
    if(gps.encode( Serial1.read()))
     gps.f_get_position(&lat,&lon);
}
}

//fonction qui envoye les coordonnes par SMS
void SendText(){
      char text[128];

      strcpy_P( text, (const char *) F("\nLatitude: ") );
      char *ptr = &text[ strlen(text) ];
      
      dtostrf( lat, 4, 5, ptr ); // ajouter le latitude
      char   *lat    = ptr;                
      size_t  latLen = strlen(ptr);        
      ptr = &ptr[ latLen ];

      strcpy_P( ptr, (const char *) F("\nlongitude: ") ); // append le string
      ptr = &ptr[ strlen(ptr) ];

      dtostrf( lon, 4, 5, ptr ); // ajouter la longitude
      char   *lon    = ptr;                 
      size_t  lonLen = strlen(ptr);
      ptr = &ptr[ lonLen ];

      strcpy_P( ptr, (const char *) F("\n\nhttps://maps.google.com/maps/@") );
      ptr = &ptr[ strlen(ptr) ];

      strncpy( ptr, lat, latLen );  // copie la latitude
      ptr = &ptr[ latLen ];

      *ptr++ = ',';  // ajouter un character

      strncpy( ptr, lon, lonLen );  // copie la longitude 
      ptr = &ptr[ lonLen ];


     
      size_t textLen = strlen(text);
      if (textLen >= sizeof(text)) {
        Serial.print( F("Tableau texte trop petit") );
        Serial.print( textLen + 1 );
        Serial.println( F("] !") );
      }

      Serial.println(text);
       //  envoyer le message
      sms.beginSMS(remoteNumber);
      sms.print(text);
      sms.endSMS();}
