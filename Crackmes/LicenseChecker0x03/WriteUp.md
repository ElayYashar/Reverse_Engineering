***From: `https://crackmes.one/crackme/62072dd633c5d46c8bcbfd9`***  
***Difficulty: `1.8`***

# Solution

Code after cleaning it up:

    void main(int argc,undefined8 *argv)
    {
    int currentCharAsDec;
    size_t lengthOfInput;
    long offset;
    char currentCharInInput;
    int unknownInt1;
    int inputCharIndex;
    undefined8 local_10;
    
    local_10 = *(undefined8 *)(offset + 0x28);
    if (argc != 2) {
        printf("Usage : %s <license pass code here [numbers only]>\n",*argv);
                        /* WARNING: Subroutine does not return */
        exit(0);
    }
    unknownInt1 = 0;
    inputCharIndex = 0;
    while( true ) {
                        /* Gets the length of the second argument entered, the license */
        lengthOfInput = strlen((char *)argv[1]);
                        /* If reached the end of the string, stop */
        if ((int)lengthOfInput <= inputCharIndex) break;
        currentCharInInput = *(char *)((long)inputCharIndex + argv[1]);
                        /* Converts the char value into an int */
        currentCharAsDec = atoi(&currentCharInInput);
        unknownInt1 = unknownInt1 + currentCharAsDec;
        inputCharIndex = inputCharIndex + 1;
    }
    if (unknownInt1 == 0x32) {
        puts("Premium access has been activated !");
                        /* WARNING: Subroutine does not return */
        exit(0);
    }
    puts("Wrong license code");
                        /* WARNING: Subroutine does not return */
    exit(0);
    }

In order to get the right license code, we need to enter numbers that add up to the hexadecimal ***0x32***, which in decimal is ***50***. 

    root@kali ~/D/C/C/L/source# ./license_checker_3 5555555555                                      
    Premium access has been activated ! 

# Crack

We can just delete all the checks

    #include <stdio.h>                                                                              
                                                                                                    
    void main(int argc, char **argv)                                                                 
    {                                                                                               
        if (argc != 2) {                                                                              
            printf("Usage : %s <license pass code here [numbers only]>\n",*argv);                       
                            /* WARNING: Subroutine does not return */                                   
            exit(0);                                                                                    
        }
        puts("Premium access has been activated !");
        exit(0);
    }
