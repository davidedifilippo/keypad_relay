# keypad_relay

![This is an image](https://github.com/davidedifilippo/keypad_relay/blob/main/keypad_relay.png)

## Tastierino

Per gestire il tastierino bisogna includere la libreria di gestione 

    #include <Keypad.h>
    
Dobbiamo inserire il codice di configurazione prima della fase di setup:

    const byte ROWS = 4;
    const byte COLS = 4;

    char Keys[ROWS][COLS] = {
                              {'1', '2', '3', 'A'},
                              {'4', '5', '6', 'B'},
                              {'7', '8', '9', 'C'},
                              {'*', '0', '#', 'D'}
                            };

    byte rowPins[ROWS] = {9, 8, 7, 6};
    byte colPins[COLS] = {5, 4, 3, 2};

    Keypad myKeypad = Keypad(makeKeymap(Keys), rowPins, colPins, ROWS, COLS);
    
in questo modo la variabile myKeypad sarà visibile anche nel loop.

## Piedino di attivazione/disattivazione relè

        const int controlPin = 12;

## Gestione delle password

    #define lunghezza_password 8

Memorizzo la password per lo sblocco del relè:

    char Master[lunghezza_password] = "1234567";   //password di accesso

La lunghezza della password all'apparenza sembrerebbe 7. In realtà viene memorizzata la stringa '1' '2' '3' '4' '5' '6' '7' e il fine stringa '\0' che è composta di 8 caratteri.
All'inizio del programma dovrà chiaramente essere dichiarato un vettore di caratteri in cui memorizzare i caratteri immessi dall'utente:

    char password[lunghezza_password];

Mi serve una variabile che conti i caratteri immessi:

     int conta_caratteri = 0;

## Setup

Nella fase di setup impostiamo il piedino di controllo come uscita:

    void setup(){
      Serial.begin(9600);
      pinMode(controlPin, OUTPUT);
      digitalWrite(controlPin, LOW); //disattivo il relè
      Serial.println("Inserire la password...");
    }

Inoltre blocchiamo l'alimentazione della bobina del relay ponendo bassa la tensione sul piedino controlPin. In questo modo il transistor di controllo della bobina viene spento.


## Fase di loop 

Ad ogni ciclo si controlla se è stato premuto un tasto. Se è cosi si memorizza il caratere nella variabile KeyPressed:

        KeyPressed = myKeypad.getKey();
        
        if (KeyPressed){
            password[conta_caratteri] = KeyPressed; //memorizzo il carattere 
            Serial.print(password[conta_caratteri]); //eco del carattere sul monitor seriale
            conta_caratteri++; //vado avanti di un carattere 
    }
    

Come si vede deve essere dichiarata una variabile di conteggio che conti il numero di caratteri immessi. Inizialmente il suo valore deve essere posto a zero. La variabile è da dichiarare all'inizio del programma, in modo che sia visibile nel loop.

Ad ogni ciclo controllo se tutti i caratteri della password sono stati inseriti completamente:     
     
     if(conta_caratteri == lunghezza_password-1){

In caso affermativo:

          if(!strcmp(password, Master))
              { //restituisce zero se sono uguali
                  Serial.print("\nPassword valida: ");
                  Serial.println(" sblocco");
                  digitalWrite(controlPin, HIGH);
                  delay(10000); //tempo di sblocco
                  digitalWrite(controlPin, LOW);
                  Serial.println("Tempo scaduto: blocco");
                    }
          else
              {
                  Serial.println("\nPassword errata... ");
                    }
        
In entrambi i casi cancello la password e svuoto la memoria dei caratteri apppena immessi:

      cancellaPassword();
      Serial.println("Inserire la password...");
      } //Ho controllato se la password è valida e ho azzerato tutto. Si ricomincia.....

In caso i caratteri non siano tutti non faccio nulla e chiudo il loop.

## La funzione cancellaPassword
 
La funzione può essere inserita ovunque fuori dal setup e dal loop:


    void cancellaPassword()
    {
      while(count !=0){
      password[count--] = 0; 
    }
    return;
}


