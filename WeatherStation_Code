/**********************************************************************************
*   Projet: Station Météo (Arduino)                                               *
*   Auteur: Adrien Desloires                                                      *
*   Version: 1.0                                                                  *
*   Date: 13/11/2016                                                              *
**********************************************************************************/

#include <DHT.h> //Importe la lib de la dht
#include <LiquidCrystal.h> //Importe la lib de l'ecran lcd
#include <math.h> //Importe la lib maths pour log et puissances

#define DHTPIN 9     // Pin de la dht
#define DHTTYPE DHT11 // On definit la dht comme etant une dht-11

DHT dht(DHTPIN, DHTTYPE); // Initialise la dht.
LiquidCrystal lcd(12, 11, 5, 4, 3, 2); //Initialisation des pins du lcd

//Déclaration des données lors d'un Reset
//Temperature Interieur
float valeurMiniTin = 50;
float valeurMaxTin = 0;
float valeurMiniPrecedenteTin = 0;
float valeurMaxPrecedenteTin = 0;
//Humidité Interieur
float valeurMiniHin = 100;
float valeurMaxHin = 0;
float valeurMiniPrecedenteHin = 0;
float valeurMaxPrecedenteHin = 0;
//Temperature Exterieur
float valeurMiniTex = 50;
float valeurMaxTex = 0;
float valeurMiniPrecedenteTex = 0;
float valeurMaxPrecedenteTex = 0;
//Luminosité
int valeurMiniLum = 1000;
int valeurMaxLum = 0;
int valeurMiniPrecedenteLum = 0;
int valeurMaxPrecedenteLum = 0;

// Initialisation
void setup() {
  // initialize serial communication at 9600 bits per second:
  Serial.begin(9600);
  lcd.begin(16,2); //On initialise le lcd avec ses dimensions (16 caracteres par ligne, sur 2 lignes)
}

// Boucle d'éxecution:
void loop() {
  //Récupère la temperature interieur avec la dht
  float tempInterieur = dht.readTemperature();
  //Récupère l'humidité interieur avec la dht
  float humiInterieur = dht.readHumidity();
  
  //Verification de sécurité si lecture de la dht bien déroulée
  if (isnan(humiInterieur) || isnan(tempInterieur)) //NaN = Not a Number
  {
    Serial.println("Erreur de lecture DHT."); //msg erreur
    return; //on sort de l'iteration courante de la boucle
  }
  
  //Recupere temperature exterieur avec la thermistance
  float tensionThermistance = analogRead(A0); //Valeur analogique de la tension aux bornes de la thermistance
  //Conversions
  float tempExterieur = convertirThermistance(tensionThermistance);
    
  //Récupère luminosité
  int voltagePhoto = analogRead(A4); //Plus la tension augmente et plus la luminosité augmente
  
  //Activer le ventilo selon la temperature interieur (22° = 1% - 25° = 100%)
  calculPuissanceMoteur(tempInterieur);
  
  //Reset
  //Mise à jour incrémentale des valeurs min et max
  miseAJourValeurMinMax(tempInterieur, humiInterieur, tempExterieur, voltagePhoto);
  //Reset si appuis sur bouton poussoir
  if (analogRead(A2) > 10)
  {
    enregistrementValeurMinMax(); //Enregistrement des valeurs min et max
    Serial.println("Reset!"); //Msg de confirmation
  }
  
  //Affiche les données capteur sur ecran lcd
  String ligne1 = genererLigne1LCD(tempInterieur, humiInterieur);
  String ligne2 = genererLigne2LCD(tempExterieur, voltagePhoto);
  afficherEcran(ligne1, ligne2);
  
  //Affiche les données MinMax précédentes sur moniteur série
  afficherMoniteur();

  delay(3000);        // Delai de 3 secondes pour la stabilité du système et récupérer des données valides sur la DHT
}

                                 
                                 // FONCTIONS \\

/**********************************************************************************
* Nom: afficherEcran                                                              *
* Fonction: Affiche les données capteur sur l'écran LCD                           *
* Valeur de retour: Aucune                                                        *
**********************************************************************************/
void afficherEcran(String ligne1, String ligne2)
{
  lcd.setCursor(0,0); //Ligne 1
  lcd.print(ligne1); //Affiche la premiere ligne sur l'ecran
  
  lcd.setCursor(0,1); //Ligne 2
  lcd.print(ligne2); //Affiche la seconde ligne sur l'ecran
}


/**********************************************************************************
* Nom: genererLigne1LCD                                                           *
* Fonction: Génère la première ligne d'affichage de l'écran LCD                   *
* Valeur de retour: Première ligne de l'écran formatée                            *
**********************************************************************************/
String genererLigne1LCD(float tempInterieur, float humiInterieur)
{
  String temperature = String(tempInterieur, 1); //xx.x
  String humidite = String(humiInterieur, 0); //xx
  
  return String("Tin:"+temperature+" "+"Hum:"+humidite);
}


/**********************************************************************************
* Nom: genererLigne2LCD                                                           *
* Fonction: Génère la seconde ligne d'affichage de l'écran LCD                    *
* Valeur de retour: Seconde ligne de l'écran formatée                             *
**********************************************************************************/
String genererLigne2LCD(float tempExterieur, int voltagePhoto)
{
  String temperature = String(tempExterieur, 1); //xx.x
  String luminosite = String(voltagePhoto); //xx
  
  return String("Tex:"+temperature+" "+"Lum:"+luminosite);
}


/**********************************************************************************
* Nom: calculPuissanceMoteur                                                      *
* Fonction: Calcul la puissance nécessaire au moteur et l'assigne                 *
* Valeur de retour: Aucune                                                        *
**********************************************************************************/
void calculPuissanceMoteur(float tempInterieur)
{
  int envoiPuissance = 0;
  int niveauPuissance = 0;
  
  niveauPuissance = map(tempInterieur, 21.9, 25, 0, 100); //Calcul du pourcentage de puissance necessaire
  envoiPuissance = map(niveauPuissance, 0, 100, 0, 255); //Conversion pourcentage en analogique
      
  analogWrite(6, envoiPuissance); //Envoi de la puissance necessaire à la broche
}


/********************************************************************************************************
* Nom: miseAJourValeurMinMax                                                                            *
* Fonction: Met à jour les valeurs mini et maxi des capteurs à chaque incrément de la boucle principale *
* Valeur de retour: Aucune                                                                              *
********************************************************************************************************/
void miseAJourValeurMinMax(float tempInterieur, float humiInterieur, float tempExterieur, int voltagePhoto)
{
  //Temperature Interieur
  if (tempInterieur < valeurMiniTin) //Si plus petit que le Min
  {
    valeurMiniTin = tempInterieur;
  }
  if (tempInterieur > valeurMaxTin) //Si superieur que le Max
  {
    valeurMaxTin = tempInterieur;
  }
  
  //Humidite Interieur
  if (humiInterieur < valeurMiniHin) //Si plus petit que le Min
  {
    valeurMiniHin = humiInterieur;
  }
  if (humiInterieur > valeurMaxHin) //Si superieur que le Max
  {
    valeurMaxHin = humiInterieur;
  }
  
  //Temperature Exterieur
  if (tempExterieur < valeurMiniTex) //Si plus petit que le Min
  {
    valeurMiniTex = tempExterieur;
  }
  if (tempExterieur > valeurMaxTex) //Si superieur que le Max
  {
    valeurMaxTex = tempExterieur;
  }
  
  //Luminosité
  if (voltagePhoto < valeurMiniLum) //Si plus petit que le Min
  {
    valeurMiniLum = voltagePhoto;
  }
  if (voltagePhoto > valeurMaxLum) //Si superieur que le Max
  {
    valeurMaxLum = voltagePhoto;
  }
}


/**********************************************************************************
* Nom: enregistrementValeurMinMax                                                 *
* Fonction: Enregistre les valeurs Mini et Maxi lors de l'appui du bouton reset   *
* Valeur de retour: Aucune                                                        *
**********************************************************************************/
void enregistrementValeurMinMax()
{
  //Temperature Interieur
  valeurMiniPrecedenteTin = valeurMiniTin;
  valeurMaxPrecedenteTin = valeurMaxTin;
  
  //Humidité Interieur
  valeurMiniPrecedenteHin = valeurMiniHin;
  valeurMaxPrecedenteHin = valeurMaxHin;
  
  //Temperature Exterieur
  valeurMiniPrecedenteTex = valeurMiniTex;
  valeurMaxPrecedenteTex = valeurMaxTex;
  
  //Luminosité
  valeurMiniPrecedenteLum = valeurMiniLum;
  valeurMaxPrecedenteLum = valeurMaxLum;
}


/**********************************************************************************
* Nom: afficherMoniteur                                                           *
* Fonction: Affiche les valeurs Mini et Maxi enregistrées sur le moniteur série   *
* Valeur de retour: Aucune                                                        *
**********************************************************************************/
void afficherMoniteur()
{
  //Temperature Interieur
  //Min
  Serial.println(String("Minimum Tin: "+String(valeurMiniPrecedenteTin, 1)));
  //Max
  Serial.println(String("Maximum Tin: "+String(valeurMaxPrecedenteTin, 1)));
  
  //Humidite Interieur
  //Min
  Serial.println(String("Minimum Hin: "+String(valeurMiniPrecedenteHin, 1)));
  //Max
  Serial.println(String("Maximum Hin: "+String(valeurMaxPrecedenteHin, 1)));
  
  //Temperature Exterieur
  //Min
  Serial.println(String("Minimum Tex: "+String(valeurMiniPrecedenteTex, 1)));
  //Max
  Serial.println(String("Maximum Tex: "+String(valeurMaxPrecedenteTex, 1)));
  
  //Luminosité
  //Min
  Serial.println(String("Minimum Lum: "+String(valeurMiniPrecedenteLum)));
  //Max
  Serial.println(String("Maximum Lum: "+String(valeurMaxPrecedenteLum)));
}


/**********************************************************************************
* Nom: convertirThermistance                                                      *
* Fonction: Converti la tension de la thermistance en température Celcius         *
* Valeur de retour: Temperature Exterieur en Celcius                              *
**********************************************************************************/
float convertirThermistance(float tensionThermistance)
{
  float retourTemperature = 0;
  
  float tensionThermistanceConvertie = (tensionThermistance * 5)/1024; //Conversion analogique en volt
  
  float Rthermi = (tensionThermistanceConvertie/((5-tensionThermistanceConvertie)/10000)); //Conversion volt vers ohms
  
  double tempKelvin = 1/(0.00086+0.0002718*log(Rthermi)-0.0000000598*(pow(log(Rthermi), 3))); //Steinhart Hart
  
  retourTemperature = tempKelvin-273.15; //kelvin->celcius
  
  return retourTemperature;
}
