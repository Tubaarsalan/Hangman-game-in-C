# Hangman-game-in-C
#include <ctype.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_WORD_LENGTH 50
#define MAX_TRIES 6
#define MAX_PLAYERS 100

// Player stats
struct Player {
    char name[50];
    int gamesPlayed;
    int wins;
    int losses;
};

// Word and hint
struct WordWithHint {
    char word[MAX_WORD_LENGTH];
    char hint[MAX_WORD_LENGTH];
};

// Display functions
void displayWord(const char word[], const bool guessed[]);
void drawHangman(int tries);
void loadPlayerData(struct Player players[], int *totalPlayers, int *totalWins, int *totalLosses);
void saveAllData(struct Player players[], int totalPlayers, int totalWins, int totalLosses);

// Data I/O


int main() {
    struct Player players[MAX_PLAYERS];
    int totalPlayers = 0, totalWins = 0, totalLosses = 0;
    char playerName[50];
    int currentPlayerIndex = -1;
    char playAgain;

    loadPlayerData(players, &totalPlayers, &totalWins, &totalLosses);

    // Ask for player name
    printf("Enter player name: ");
    fgets(playerName, sizeof(playerName), stdin);
    playerName[strcspn(playerName, "\n")] = '\0';

    // Find or create player
    for (int i = 0; i < totalPlayers; i++) {
        if (strcmp(players[i].name, playerName) == 0) {
            currentPlayerIndex = i;
            break;
        }
    }
    if (currentPlayerIndex == -1 && totalPlayers < MAX_PLAYERS) {
        currentPlayerIndex = totalPlayers++;
        strcpy(players[currentPlayerIndex].name, playerName);
        players[currentPlayerIndex].gamesPlayed = 0;
        players[currentPlayerIndex].wins = 0;
        players[currentPlayerIndex].losses = 0;
    }

    struct WordWithHint wordList[] = {
        { "c programming", "Computer coding" },
        { "elephant", "A large mammal with a trunk" },
        { "biryani", "A popular Karachi dish" },
        { "beach", "Sandy shore by the sea" },
        { "sunflower", "A tall yellow plant that follows the sun" },
        { "giraffe", "The tallest land animal with a long neck" },
        { "volcano", "A mountain that erupts with lava" },
        { "library", "A quiet place full of books" },
        { "keyboard", "Used to type letters into a computer" },
        { "rainbow", "Appears in the sky after rain and has many colors" },
        { "pyramid", "A triangular structure found in Egypt" },
        { "Markhor", "Pakistan National Animal" },
        { "oxygen", "The gas we need to breathe" },
        { "satellite", "Orbits planets and helps in communication" },
        { "chocolate", "A sweet treat made from cocoa" },
        { "penguin", "A flightless bird that swims in cold water" },
        { "moonlight", "Light from the moon at night" },
        { "notebook", "Used to take notes in school" },
        { "backpack", "Used to carry books on your back" },
        { "robot", "A machine that can perform tasks" },
        { "desert", "A very dry place with lots of sand" },
        { "forest", "A large area filled with trees" },
        { "calendar", "Shows days, months, and years" },
        { "glasses", "Worn to help you see clearly" },
        { "hammer", "A tool used to drive nails" }
    };
    int wordCount = sizeof(wordList) / sizeof(wordList[0]);

    srand((unsigned)time(NULL));
    do {
        int idx = rand() % wordCount;
        const char *secret = wordList[idx].word;
        const char *hint = wordList[idx].hint;
        int len = strlen(secret);
        bool guessed[26] = { false };
        char reveal[MAX_WORD_LENGTH] = { 0 };
        for (int i = 0; i < len; ++i)
            reveal[i] = isalpha(secret[i]) ? '_' : secret[i];
        reveal[len] = '\0';

        int tries = 0;
        printf("\nHint: %s\n", hint);

        while (tries < MAX_TRIES) {
            displayWord(reveal, guessed);
            drawHangman(tries);
            printf("Enter a letter: ");
            char guess;
            scanf(" %c", &guess);
            guess = tolower(guess);

            if (!isalpha(guess) || guessed[guess - 'a']) {
                printf("Invalid or repeated guess.\n");
                continue;
            }
            guessed[guess - 'a'] = true;

            bool hit = false;
            for (int i = 0; i < len; i++) {
                if (tolower(secret[i]) == guess) {
                    reveal[i] = secret[i];
                    hit = true;
                }
            }
            if (!hit) ++tries;

            if (strcmp(secret, reveal) == 0) {
                printf("\nYou won! Word: %s\n", secret);
                players[currentPlayerIndex].wins++;
                totalWins++;
                break;
            }
        }

        if (tries >= MAX_TRIES) {
            printf("\nYou lost. Word was: %s\n", secret);
            players[currentPlayerIndex].losses++;
            totalLosses++;
        }

        players[currentPlayerIndex].gamesPlayed++;
        printf("Play again? (y/n): ");
        scanf(" %c", &playAgain);
        getchar();
    } while (playAgain == 'y' || playAgain == 'Y');

    saveAllData(players, totalPlayers, totalWins, totalLosses);
    printf("\n=== Final Summary ===\n");
    printf("Total players: %d\n", totalPlayers);
    printf("Total wins: %d\n", totalWins);
    printf("Total losses: %d\n", totalLosses);
    printf("\nStats saved to C:\\hangman_stats.txt\n");
    return 0;
}

void displayWord(const char word[], const bool guessed[]) {
    printf("Word: ");
    for (int i = 0; word[i]; i++) {
        if (word[i] == ' ' || !isalpha(word[i])) {
            printf("%c ", word[i]);
        } else if (guessed[tolower(word[i]) - 'a']) {
            printf("%c ", word[i]);
        } else {
            printf("_ ");
        }
    }
    printf("\n");
}

void drawHangman(int tries) {
    const char *hangmanParts[] = {
        "     _________",
        "    |         |",
        "    |         O",
        "    |        /|\\",
        "    |        / \\",
        "    |"
    };
    printf("\n");
    for (int i = 0; i <= tries && i < 6; i++) {
        printf("%s\n", hangmanParts[i]);
    }
}
void loadPlayerData(struct Player players[], int *totalPlayers, int *totalWins, int *totalLosses) {
    FILE *fp = fopen("C:\\my files\\hangman_stats.txt", "r");
    if (!fp) return;
    struct Player p;
    while (fscanf(fp,
        "Player Name: %[^\n]\n"
        "Games Played: %d\n"
        "Wins: %d\n"
        "Losses: %d\n\n",
        p.name, &p.gamesPlayed, &p.wins, &p.losses) == 4) {
        if (*totalPlayers < MAX_PLAYERS) {
            players[(*totalPlayers)++] = p;
            *totalWins += p.wins;
            *totalLosses += p.losses;
        }
    }
    fclose(fp);
}

void saveAllData(struct Player players[], int totalPlayers, int totalWins, int totalLosses) {
    FILE *fp = fopen("C:\\my files\\hangman_stats.txt", "w");
    if (!fp) {
        printf("Failed to write stats.\n");
        return;
    }
    for (int i = 0; i < totalPlayers; i++) {
        fprintf(fp,
            "Player Name: %s\n"
            "Games Played: %d\n"
            "Wins: %d\n"
            "Losses: %d\n\n",
            players[i].name,
            players[i].gamesPlayed,
            players[i].wins,
            players[i].losses
        );
    }
    fprintf(fp,
        "Total Players: %d\n"
        "Total Wins: %d\n"
        "Total Losses: %d\n",
        totalPlayers, totalWins, totalLosses
    );
    fclose(fp);
}
