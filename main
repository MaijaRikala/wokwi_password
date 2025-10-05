//include libraries
#include <LiquidCrystal_I2C.h>
#include <Keypad.h>

//LCD
#define I2C_address 0x27
#define LCD_columns 20
#define LCD_rows 4

LiquidCrystal_I2C lcd(I2C_address, LCD_columns, LCD_rows);


//keypad
const uint8_t kp_rows = 4;
const uint8_t kp_columns = 4;
char keys[kp_rows][kp_columns] = {
  { '1', '2', '3', 'A' },
  { '4', '5', '6', 'B' },
  { '7', '8', '9', 'C' },
  { '*', '0', '#', 'D' }
};
uint8_t rowPins[kp_rows] = { 9, 8, 7, 6 }; //connect rows to pins
uint8_t columnPins[kp_columns] = { 5, 4, 3, 2 }; //connect columns to pins
Keypad keypad(makeKeymap(keys), rowPins, columnPins, kp_rows, kp_columns);


//declare variables to aid in execution -------------------

//user input character
char input_key;
//at first 3, because this needs to be passed to systemBoot, which
//at first asks for number 1 or 2 so input_key is something else

//track whether user is a new or existing one
int user = 0;
//0 = undetermined, 1 = new, 2 = existing

//keep track whether the user has a password
//(either an existing user or a user that has just entered their
//password correctly)
int password_state = 0;
// 0 = password not entered (completely)
// 1 = second password not entered (completely)
// 2 = password set

//store set password
char password_1[4];
int n_1 = 0; //how many numbers already entered

//store the password that is entered the second time
char password_2[4];
int n_2 = 0; //how many numbers already entered

//password that the user finally enters after setting it (if not existing)
char final_password[4];

//how many password attempts are left
int attempts = 3;
int n_f = 0; //how many numbers already entered

//track if access is granted
int access = 0;
//0 = no access, 1 = access granted

// --------------------------------------------------------



//functions for different displays on the LCD -------------

//function to call on system boot (or after too many incorrect password attempts)
void systemBoot() {
  // empty whatever is displayed
  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("1. New User");
  lcd.setCursor(0, 1);
  lcd.print("2. Existing User");
}

//function to call, when user enters '1' (selects new user)
void newUser() {
  // empty whatever is displayed
  lcd.clear();

  if (password_state == 0) {
    //display message for setting password
    lcd.setCursor(1, 0);
    lcd.print("Set password");
  }
  else {
    //display message for re-entering password
    lcd.setCursor(1, 0);
    lcd.print("Re-enter");
    lcd.setCursor(1, 1);
    lcd.print("password");
  }
}

//function to call, when user enters the wrong password, while
//setting the password
void mismatch() {
  // empty whatever is displayed
  lcd.clear();

  lcd.setCursor(1, 0);
  lcd.print("Mismatch");
  lcd.setCursor(1, 1);
  lcd.print("Try again");
}

//function to call, when user enters '2' or has set password
void lockMode() {
  // empty whatever is displayed
  lcd.clear();

  if (attempts > 0 && access == 0) {
    lcd.setCursor(1, 0);
    lcd.print("Enter password");
  }
  if (attempts > 0 && access == 1) {
    lcd.setCursor(1, 0);
    lcd.print("Access granted!");
  }
  if (attempts == 0) {
    lcd.setCursor(1, 0);
    lcd.print("Out of");
    lcd.setCursor(1, 1);
    lcd.print("attempts");
  }
}

//function to call, when user enters the wrong password
//(not when setting the password)
void wrongPassword() {
  // empty whatever is displayed
  lcd.clear();

  lcd.setCursor(1, 0);
  lcd.print("Incorrect!");
  lcd.setCursor(1, 1);
  lcd.print("Try again");
}

// -------------------------------------------------------



void setup() {
  // put your setup code here, to run once:
  
  Serial.begin(9600);

  //setup LCD
  lcd.init();
  lcd.backlight();

  //display systemBoot screen
  systemBoot();
}

void loop() {
  // put your main code here, to run repeatedly:
  
  //read user input
  input_key = keypad.getKey();

  //check if user is new or existing
  //only executed, when this is undetermined so not after the user
  //has been asked this
  if (user == 0) {
    user = input_key;
    if (input_key == '1') {
      password_state = 0; //start process over for new user
      input_key = NO_KEY; // to avoid including this input in the password
      //for user clarity, the next step is slightly delayed
      delay(500);
      //update display
      newUser();
    }
    if (input_key == '2') {
      password_state = 2; //skip to password entering for existing user
      input_key = NO_KEY; // to avoid including this input in the password
      //for user clarity, the next step is slightly delayed
      delay(500);
      //update display
      lockMode();
    }
    //no error handling for incorrect input
  }

  
  //check that key has been input and if the user input something, use it
  
  if (password_state == 0 && input_key != NO_KEY) {
    //new user, who doesn't have a set password

    //save the entered password
    password_1[n_1] = input_key;
    n_1 += 1;

    if (n_1 == 4) {
      // enough numbers entered
      password_state = 1;
      input_key = NO_KEY; //this is so that the latest input
      // isn't stored into any passwords
      
      /*
      //print for checking
      for (int i; i<n_1; i++) {
        Serial.print(password_1[i]);
      }
      Serial.print("\n");
      */
      
      //again for clarity slight delay
      delay(500);
      //update display
      newUser();
    }
  }
  
  if (password_state == 1 && input_key != NO_KEY) {
    //new user who has entered the password but not confirmed it

    //save the second entered password
    password_2[n_2] = input_key;
    n_2 += 1;

    if (n_2 == 4) { // enough numbers entered
      /*
      //print for checking
      for (int i; i<n_2; i++) {
        Serial.print(password_2[i]);
      }
      Serial.print("");
      */

      // compare the passwords to make sure they match
      if (strcmp(password_1, password_2) == 0) {
        //if they match, move to the next stage
        password_state = 2;
        input_key = NO_KEY;
        //again for clarity slight delay
        delay(500);
        lockMode();
      }
      else {
        //otherwise go back to beginning
        //re-set everything
        password_state = 0;
        n_1 = 0;
        n_2 = 0;
        input_key = NO_KEY;
        
        //update display
        mismatch();
        delay(5000);
        
        //update display
        newUser();
      }
    }
  }

  if (password_state == 2 && input_key != NO_KEY) {
    //existing user and new user who has just entered their password

    //check that there are still attempts left
    if (attempts > 0) {
      //if there are, let enter the password
      final_password[n_f] = input_key;
      n_f += 1;

      if (n_f == 4) { // enough numbers entered
        //print for checking
        /*
        for (int i; i<n_f; i++) {
          Serial.print(final_password[i]);
        }
        Serial.println("");
        */

        //compare to the correct password already entered
        if ((strcmp(final_password, password_1)) == 0) {
          //password was correct -> access is granted
          access = 1;
          delay(500);

          //update display
          lockMode();
        }
        else {
          //password was incorrect -> access was not granted
          attempts -= 1; //reduce the amount of attempts left
          if (attempts > 0) {
            //this is so that can enter the next attempt
            input_key = NO_KEY;
            delay(100);

            //update display
            wrongPassword();
            n_f = 0;
            delay(2000);

            //update display
            lockMode();
          }
          else {
            //ran out of attempts

            delay(500);

            //update display
            lockMode();

            //re-set everything

            input_key = NO_KEY;
            user = 0;
            n_1 = 0;
            n_2 = 0;
            n_f = 0;
            password_state = 0;
            attempts = 3;
            
            delay(5000);

            //update display
            systemBoot();
          }
        }
      }
    }
  }
}