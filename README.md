# EX. NO:2 IMPLEMENTATION OF PLAYFAIR CIPHER

## AIM:
To write a C program to implement the Playfair Substitution technique.

## DESCRIPTION:
The Playfair cipher starts with creating a key table. The key table is a 5×5 grid of letters that will act as the key for encrypting your plaintext. Each of the 25 letters must be unique and one letter of the alphabet is omitted from the table (as there are 25 spots and 26 letters in the alphabet).

To encrypt a message, one would break the message into digrams (groups of 2 letters) such that, for example, "HelloWorld" becomes "HE LL OW OR LD", and map them out on the key table. The two letters of the diagram are considered as the opposite corners of a rectangle in the key table. Note the relative position of the corners of this rectangle. Then apply the following 4 rules, in order, to each pair of letters in the plaintext:

1. If both letters are the same (or only one letter is left), add an "X" after the first letter
2. If the letters appear on the same row of your table, replace them with the letters to their immediate right respectively
3. If the letters appear on the same column of your table, replace them with the letters immediately below respectively
4. If the letters are not on the same row or column, replace them with the letters on the same row respectively but at the other pair of corners of the rectangle defined by the original pair.

## EXAMPLE :-

<img width="558" height="380" alt="image" src="https://github.com/user-attachments/assets/d7dd0e55-c464-4fab-a87b-0d878a39f9d1" />

## ALGORITHM:

STEP-1: Read the plain text from the user. 
STEP-2: Read the keyword from the user.
STEP-3: Arrange the keyword without duplicates in a 5*5 matrix in the row order and fill the remaining cells with missed out letters in alphabetical order. Note that ‘i’ and ‘j’ takes the same cell. 
STEP-4: Group the plain text in pairs and match the corresponding corner letters by forming a rectangular grid. 
STEP-5: Display the obtained cipher text.

## PROGRAM :

```
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define MAX_TEXT 4096

char keyTable[5][5];

/* Convert to lowercase, keep only a-z, replace j with i */
void formatText(const char *in, char *out) {
    int k = 0;
    for (int i = 0; in[i] != '\0'; i++) {
        if (isalpha((unsigned char)in[i])) {
            char c = (char)tolower((unsigned char)in[i]);
            if (c == 'j') c = 'i';
            out[k++] = c;
        }
    }
    out[k] = '\0';
}

/* Generate the 5x5 key table */
void generateKeyTable(const char *key) {
    int used[26] = {0};

    char fkey[MAX_TEXT];
    formatText(key, fkey);

    int row = 0, col = 0;

    // Fill with key characters
    for (int i = 0; fkey[i] != '\0'; i++) {
        char c = fkey[i];
        int idx = c - 'a';
        if (!used[idx]) {
            keyTable[row][col++] = c;
            used[idx] = 1;
            if (col == 5) {
                row++;
                col = 0;
            }
        }
    }

    // Fill remaining alphabet (skip j)
    for (char c = 'a'; c <= 'z'; c++) {
        if (c == 'j') continue;
        int idx = c - 'a';
        if (!used[idx]) {
            keyTable[row][col++] = c;
            used[idx] = 1;
            if (col == 5) {
                row++;
                col = 0;
            }
        }
    }
}

/* Prepare plaintext into digraphs (insert x, pad x) */
void preparePlainText(const char *text, char *out) {
    char ftext[MAX_TEXT];
    formatText(text, ftext);

    int n = (int)strlen(ftext);
    int i = 0, k = 0;

    while (i < n) {
        char a = ftext[i];
        char b = (i + 1 < n) ? ftext[i + 1] : '\0';

        out[k++] = a;

        if (b == '\0') {
            out[k++] = 'x';
            i += 1;
        } else if (a == b) {
            out[k++] = 'x';
            i += 1; // only consume 'a', reprocess b next loop
        } else {
            out[k++] = b;
            i += 2;
        }
    }

    if (k % 2 != 0) out[k++] = 'x';
    out[k] = '\0';
}

/* Find character position in key table */
void findPosition(char c, int *r, int *cidx) {
    for (int i = 0; i < 5; i++) {
        for (int j = 0; j < 5; j++) {
            if (keyTable[i][j] == c) {
                *r = i;
                *cidx = j;
                return;
            }
        }
    }
    *r = -1;
    *cidx = -1;
}

/* Encrypt */
void encrypt(const char *plaintext, const char *key, char *cipher) {
    generateKeyTable(key);

    char prepared[MAX_TEXT];
    preparePlainText(plaintext, prepared);

    int n = (int)strlen(prepared);
    int k = 0;

    for (int i = 0; i < n; i += 2) {
        char a = prepared[i];
        char b = prepared[i + 1];

        int r1, c1, r2, c2;
        findPosition(a, &r1, &c1);
        findPosition(b, &r2, &c2);

        if (r1 == r2) {
            cipher[k++] = keyTable[r1][(c1 + 1) % 5];
            cipher[k++] = keyTable[r2][(c2 + 1) % 5];
        } else if (c1 == c2) {
            cipher[k++] = keyTable[(r1 + 1) % 5][c1];
            cipher[k++] = keyTable[(r2 + 1) % 5][c2];
        } else {
            cipher[k++] = keyTable[r1][c2];
            cipher[k++] = keyTable[r2][c1];
        }
    }

    cipher[k] = '\0';
}

/* Decrypt */
void decrypt(const char *ciphertext, const char *key, char *plain) {
    generateKeyTable(key);

    char fct[MAX_TEXT];
    formatText(ciphertext, fct);

    int n = (int)strlen(fct);
    int k = 0;

    for (int i = 0; i < n; i += 2) {
        char a = fct[i];
        char b = fct[i + 1];

        int r1, c1, r2, c2;
        findPosition(a, &r1, &c1);
        findPosition(b, &r2, &c2);

        if (r1 == r2) {
            plain[k++] = keyTable[r1][(c1 + 4) % 5];
            plain[k++] = keyTable[r2][(c2 + 4) % 5];
        } else if (c1 == c2) {
            plain[k++] = keyTable[(r1 + 4) % 5][c1];
            plain[k++] = keyTable[(r2 + 4) % 5][c2];
        } else {
            plain[k++] = keyTable[r1][c2];
            plain[k++] = keyTable[r2][c1];
        }
    }

    plain[k] = '\0';
}

/* Display key table */
void displayKeyTable(void) {
    printf("Key Table:\n");
    for (int i = 0; i < 5; i++) {
        for (int j = 0; j < 5; j++) {
            printf("%c ", keyTable[i][j]);
        }
        printf("\n");
    }
}

int main(void) {
    char key[MAX_TEXT], plaintext[MAX_TEXT];
    char cipher[MAX_TEXT], decrypted[MAX_TEXT];

    if (!fgets(key, sizeof(key), stdin)) return 1;
    key[strcspn(key, "\n")] = '\0';

 
    if (!fgets(plaintext, sizeof(plaintext), stdin)) return 1;
    plaintext[strcspn(plaintext, "\n")] = '\0';

    printf("************\n");
    printf("Key Text    : %s\n", key);
    printf("Plain Text  : %s\n", plaintext);

    encrypt(plaintext, key, cipher);
    printf("Cipher Text : %s\n", cipher);

    decrypt(cipher, key, decrypted);
    printf("Decrypted Text : %s\n", decrypted);

    return 0;
}
```

## OUTPUT :-

<img width="1899" height="779" alt="EXP2CRYPTo" src="https://github.com/user-attachments/assets/9c3ce82c-be47-4733-a3c2-7b2e87fe2e96" />

## RESULT :-

Thus the implementation of ceasar cipher had been executed successfully.
