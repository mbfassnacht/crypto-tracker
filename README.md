# crypto-tracker
Crypto tracker created with a Particle Photon

## Preview

![crypto-tracker](https://github.com/mbfassnacht/assets/raw/master/images/crypto-tracker/welcome.png)

## Motivation
My motivation to create this was to have fun with my Photon that was in his box for a while.
I'm also into the crypto world, and I decided to create a tool that helps me without being the whole time accessing to exchanges or to stocks dashboards.
Another good reason is that I had always the problem that I could not see all the cryptocurrencies that I wanted in different currency values (EUR, BTC, USD, etc) at once.

## How it works
This crypto tracker is connected to the internet and works with the particle.io infraestructure, using the Webhooks functionality as well as the remote console for deploying the code in your Photon.
The data is fetched from an external free [API](https://www.coingecko.com/en/api#explore-api)

## What do you need

1. [1 Photon](https://store.particle.io/products/photon)
2. [1 Protoboard and 20 Cables](https://www.amazon.com/ZYAMY-Solderless-Breadboard-Protoboard-Self-Adhesive/dp/B074YZCNSM/ref=sr_1_20?dchild=1&keywords=protoboard&qid=1612603074&sr=8-20)
3. [1 Red and 1 Green led](https://www.amazon.com/YETAIDA-Assortment-Universal-Science-Experience/dp/B07Q567MNP/ref=sr_1_10?crid=GXWE5W4AKP6N&dchild=1&keywords=arduino+red+led&qid=1612603251&sprefix=arduino+red%2Caps%2C252&sr=8-10)
4. [1 LCD 16X2](https://www.amazon.com/Interface-Characters-Display-Monitor-Compatible/dp/B07PFBDDYB/ref=sr_1_2_sspa?dchild=1&keywords=LCD+16X2&qid=1612603212&sr=8-2-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFKOFA5Q1M4VjlSQjUmZW5jcnlwdGVkSWQ9QTAxNzkwOTUxTkFOOUdCNTU4OE5KJmVuY3J5cHRlZEFkSWQ9QTA0MTQyMTMyQUxRQlJCRlVYWVAzJndpZGdldE5hbWU9c3BfYXRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==)
5. [1 Potentiometer](https://www.amazon.com/Tegg-Linear-Taper-Rotary-Potentiometer/dp/B07PLXFRLK/ref=sr_1_4?dchild=1&keywords=1+Potentiometer&qid=1612603374&sr=8-4)
6. [2 Resistors](https://www.amazon.com/AUSTOR-Resistors-Assortment-Resistor-Experiments/dp/B07BKRS4QZ/ref=sr_1_13?crid=OSHHKW2V9JNR&dchild=1&keywords=arduino+resistors&qid=1612603436&sprefix=arduino+resi%2Caps%2C269&sr=8-13)


## Build the circuit

## Add Particle Webhooks

## Write your code

### Go into Particle Web IDE
https://build.particle.io/build/new

If you are new here, you will need to setup an account.

### Create a new App
Choose the tiltle that you want.

### Add the libraries to the app
To do this you will need to go to the Libraries Menu that is in the sidebar and search for the libraries, then you will be able to choose to which project you want to add them, choose the app that you just created.
#### LiquidCrystal
We use LiquidCrystal to make the handling of the LCD 16X2 easier.

#### JsonParserGeneratorRK
We use JsonParserGeneratorRK to make the parsing of a JSON response easier.

### Write the code for your app
#### Define PINS and LCD
Start by defining the PINs of the PHOTON that you are going to use, as well as which PINS will use the LCD 16X2.

As an example I just added 5 coins here, you can change here the names and use the coins that you are interested. Then you will need to change them in the following steps

```C
// This #include statement was automatically added by the Particle IDE.
#include <JsonParserGeneratorRK.h>
#include "application.h"
#include <LiquidCrystal.h>

LiquidCrystal lcd(D0, D1, D2, D3, D4, D5);

int RED_LED = A2;
int GREEN_LED = A5;
int BITCOIN_VALUE = 0;
int ETHEREUM_VALUE = 0;
double ALGORAND_VALUE = 0;
double THE_GRAPH_VALUE = 0;
double STELLAR_VALUE = 0;

void setup() {

}

void loop() {

}
```
#### Write our Setup
Now that we alredy defined the PINS and the coins that we want we can proceed to write our setup and loop. This methods are important in the lifecycle of your app and are part of the specifications of Particle Apps.

For the setup we will subscribe to the webhooks that we defined before as well as the callbacks that we want to use for each hook.
We can also define the message that we want in our LCD to be displayed at the begining of the app.

This code will be run only one time at the initalization of our app.

```C
void setup() {
    Particle.subscribe("hook-response/GET_VALUES", printValues, MY_DEVICES);
    Particle.subscribe("hook-response/GET_MARKET_STATUS", changeLights, MY_DEVICES);

    pinMode(RED_LED, OUTPUT);
    pinMode(GREEN_LED, OUTPUT);
    // set up the LCD's number of columns and rows: 
    lcd.begin(16,2);
    // Print a message to the LCD.
    lcd.print("Welcome to");
    lcd.setCursor(0, 1);
    lcd.print("CRYPTO TRACKER");
    delay(3000);
    lcd.clear();
}
```
#### Write our Loop
Define our loop. Here we are saying that every 18 seconds we want to trigger our Webhooks.
```C
void loop() {
    Particle.publish("GET_VALUES", "FETCH PRICES", PRIVATE);
    Particle.publish("GET_MARKET_STATUS", "FETCH BITCOIN MARKET", PRIVATE);

    delay(18000);
}
```

#### Print the values
For printing the values we will need this auxiliar functions, so we keep changing our led every 4 seconds, so we can see all the coins that we configured.
```C
void printIntCurrency(const int newPrice, const int oldPrice, const char *cryptoCurrency, const char *crypyoLabel ) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print(crypyoLabel);
    
    lcd.setCursor(0, 1);
    if(newPrice > oldPrice) {
        lcd.print(String(newPrice) + " " + cryptoCurrency + " ++" );
    } else {
        if(newPrice < oldPrice) {
            lcd.print(String(newPrice) + " " + cryptoCurrency + " --" );

        } else {
            lcd.print(String(newPrice) + " " + cryptoCurrency);
        }
    }
}


void printDoubleCurrency(const double newPrice, const double oldPrice, const char *cryptoCurrency, const char *crypyoLabel ) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print(crypyoLabel);
    
    lcd.setCursor(0, 1);
    if(newPrice > oldPrice) {
        lcd.print(String(newPrice) + " " + cryptoCurrency + " ++" );
    } else {
        if(newPrice < oldPrice) {
            lcd.print(String(newPrice) + " " + cryptoCurrency + " --" );

        } else {
            lcd.print(String(newPrice) + " " + cryptoCurrency);
        }
    }
}

void printValues(const char *event, const char *data) {
    JsonParser parser;


    int responseIndex = 0;

	const char *slashOffset = strrchr(event, '/');
	if (slashOffset) {
		responseIndex = atoi(slashOffset + 1);
	}

	if (responseIndex == 0) {
		parser.clear();
	}
	parser.addString(data);

    parser.parse();
    
    int newBitcoinValue = parser.getReference().key("bitcoin").key("usd").valueInt();
    int newEthereumValue = parser.getReference().key("ethereum").key(currencies[CURRENT_CURRENCY]).valueInt();
    double newAlgorandValue = parser.getReference().key("algorand").key(currencies[CURRENT_CURRENCY]).valueDouble();
    double newGraphValue = parser.getReference().key("the-graph").key(currencies[CURRENT_CURRENCY]).valueDouble();
    double newStellarValue = parser.getReference().key("stellar").key(currencies[CURRENT_CURRENCY]).valueDouble();

    printIntCurrency(newBitcoinValue, BITCOIN_VALUE, "USD", "Bitcoin");
    delay(4000);
    BITCOIN_VALUE = newBitcoinValue;
    
    printIntCurrency(newEthereumValue, ETHEREUM_VALUE, "EUR", "Ethereum");
    delay(4000);
    ETHEREUM_VALUE = newEthereumValue;
    
    printDoubleCurrency(newAlgorandValue,ALGORAND_VALUE , "EUR", "Algorand");
    delay(4000);
    ALGORAND_VALUE = newAlgorandValue;
    
    printDoubleCurrency(newGraphValue, THE_GRAPH_VALUE, "EUR", "The Graph");
    delay(4000);
    THE_GRAPH_VALUE = newGraphValue;
    
    printDoubleCurrency(newStellarValue, STELLAR_VALUE, "EUR", "Stellar Lumens");
    delay(4000);
    STELLAR_VALUE = newStellarValue;
}
```

#### All Source Code
```C
// This #include statement was automatically added by the Particle IDE.
#include <JsonParserGeneratorRK.h>
#include "application.h"
#include <LiquidCrystal.h>

LiquidCrystal lcd(D0, D1, D2, D3, D4, D5);

int RED_LED = A2;
int GREEN_LED = A5;
int BITCOIN_VALUE = 0;
int ETHEREUM_VALUE = 0;
double ALGORAND_VALUE = 0;
double THE_GRAPH_VALUE = 0;
double STELLAR_VALUE = 0;


const char *currencies[] = {"eur","usd"};
int CURRENT_CURRENCY = 0;

JsonParserStatic<2048, 100> marketValuesJsonParser;


void printIntCurrency(const int newPrice, const int oldPrice, const char *cryptoCurrency, const char *crypyoLabel ) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print(crypyoLabel);
    
    lcd.setCursor(0, 1);
    if(newPrice > oldPrice) {
        lcd.print(String(newPrice) + " " + cryptoCurrency + " ++" );
    } else {
        if(newPrice < oldPrice) {
            lcd.print(String(newPrice) + " " + cryptoCurrency + " --" );

        } else {
            lcd.print(String(newPrice) + " " + cryptoCurrency);
        }
    }
}


void printDoubleCurrency(const double newPrice, const double oldPrice, const char *cryptoCurrency, const char *crypyoLabel ) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print(crypyoLabel);
    
    lcd.setCursor(0, 1);
    if(newPrice > oldPrice) {
        lcd.print(String(newPrice) + " " + cryptoCurrency + " ++" );
    } else {
        if(newPrice < oldPrice) {
            lcd.print(String(newPrice) + " " + cryptoCurrency + " --" );

        } else {
            lcd.print(String(newPrice) + " " + cryptoCurrency);
        }
    }
}

void printValues(const char *event, const char *data) {
    JsonParser parser;


    int responseIndex = 0;

	const char *slashOffset = strrchr(event, '/');
	if (slashOffset) {
		responseIndex = atoi(slashOffset + 1);
	}

	if (responseIndex == 0) {
		parser.clear();
	}
	parser.addString(data);

    parser.parse();
    
    int newBitcoinValue = parser.getReference().key("bitcoin").key("usd").valueInt();
    int newEthereumValue = parser.getReference().key("ethereum").key(currencies[CURRENT_CURRENCY]).valueInt();
    double newAlgorandValue = parser.getReference().key("algorand").key(currencies[CURRENT_CURRENCY]).valueDouble();
    double newGraphValue = parser.getReference().key("the-graph").key(currencies[CURRENT_CURRENCY]).valueDouble();
    double newStellarValue = parser.getReference().key("stellar").key(currencies[CURRENT_CURRENCY]).valueDouble();

    printIntCurrency(newBitcoinValue, BITCOIN_VALUE, "USD", "Bitcoin");
    delay(4000);
    BITCOIN_VALUE = newBitcoinValue;
    
    printIntCurrency(newEthereumValue, ETHEREUM_VALUE, "EUR", "Ethereum");
    delay(4000);
    ETHEREUM_VALUE = newEthereumValue;
    
    printDoubleCurrency(newAlgorandValue,ALGORAND_VALUE , "EUR", "Algorand");
    delay(4000);
    ALGORAND_VALUE = newAlgorandValue;
    
    printDoubleCurrency(newGraphValue, THE_GRAPH_VALUE, "EUR", "The Graph");
    delay(4000);
    THE_GRAPH_VALUE = newGraphValue;
    
    printDoubleCurrency(newStellarValue, STELLAR_VALUE, "EUR", "Stellar Lumens");
    delay(4000);
    STELLAR_VALUE = newStellarValue;
}

void changeLights(const char *event, const char *data) {
    int responseIndex = 0;


	marketValuesJsonParser.addChunkedData(event, data);
	
	if (marketValuesJsonParser.parse()) {
        double bitcoinStatus = marketValuesJsonParser.getReference().index(0).key("price_change_percentage_1h_in_currency").valueDouble();
    
        Particle.publish("DEBUG", "BITCOIN VALUE IS: " + String(bitcoinStatus), PRIVATE);

        if (bitcoinStatus > 0.0) {
            Particle.publish("DEBUG", "BITCOIN IS UP", PRIVATE);
            digitalWrite(GREEN_LED, HIGH);
            digitalWrite(RED_LED, LOW);
        } else {
            Particle.publish("DEBUG", "BITCOIN IS DOWN", PRIVATE);
            digitalWrite(GREEN_LED, LOW);
            digitalWrite(RED_LED, HIGH);
        }
        
        marketValuesJsonParser.clear();

    }
}

       
void setup() {
    Particle.subscribe("hook-response/GET_VALUES", printValues, MY_DEVICES);
    Particle.subscribe("hook-response/GET_MARKET_STATUS", changeLights, MY_DEVICES);

    pinMode(RED_LED, OUTPUT);
    pinMode(GREEN_LED, OUTPUT);
    // set up the LCD's number of columns and rows: 
    lcd.begin(16,2);
    // Print a message to the LCD.
    lcd.print("Welcome to");
    lcd.setCursor(0, 1);
    lcd.print("CRYPTO TRACKER");
    delay(3000);
    lcd.clear();
}

void loop() {
    Particle.publish("GET_VALUES", "FETCH PRICES", PRIVATE);
    Particle.publish("GET_MARKET_STATUS", "FETCH BITCOIN MARKET", PRIVATE);

    delay(18000);
}
      

```
## All setup, run it!

![crypto-tracker](https://github.com/mbfassnacht/assets/raw/master/images/crypto-tracker/final.png)


## Disclaimer
Any data that you see in the crypto-tracker can have delays and can be not 100% accurate. This information should not be a reason for investing.
