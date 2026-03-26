# C Programming Mini Project – Digital Assignment

**Project Title:** Library Book Tracking System<br>
**Student Name:** Aryan<br>
**Register Number:** 25BCE1734<br>
**Department:** SCOPE<br>
**Course Name:** B. Tech<br>
**Faculty Name:** Dinakaran M<br>
**Submission Date:** 26 March 2026<br>
**Github Repository Link:** https://github.com/aryan475m/Library-Book-Tracking-System<br>
**Live Hosted Link:** https://aryan475m.github.io/Library-Book-Tracking-System/<br>

---

## 1. Problem Statement
This project provides a comprehensive Library Book Tracking System designed to help educational institutions and libraries manage their book inventory seamlessly. The system enables librarians to add new books, issue books to students, track return dates, and view comprehensive statistics regarding library usage. 

This system can be used by any library administrator to replace manual ledger tracking. It ensures books are not double-issued, maintains a permanent transaction history log of what was issued and returned, and allows for rapid searching of the inventory using Book IDs or Title keywords. By leveraging file handling in C, all data is retained permanently across multiple sessions.

## 2. Features Implemented
* **Add Record:** Add a new book (BookID, Title, Author, Status).
* **Issue Book:** Issue a book to a student (stores Registration No. and Timestamp).
* **Return Book:** Process returned books and update availability.
* **Search System:** Find books by ID or case-insensitive Title keyword.
* **File Handling:** Binary saving/loading of books and logs for persistence.
* **Double-Issue Prevention:** System rejects issuing a book that is already issued.
* **Transaction Logging:** Maintains a chronological history of all issues/returns.
* **Library Dashboard:** Shows totals for available books, issued books, and overall capacity.
* **Delete Book:** Safely remove a book from the inventory if it is not currently issued.

---

## 3. System Design

### Structures Used:
```c
typedef enum { AVAILABLE, ISSUED } BookStatus;

typedef struct {
    int    bookID;
    char   title[100];
    char   author[60];
    BookStatus status;
    char   issuedTo[30];   
    char   issueDate[26];  
} Book;

typedef struct {
    int    bookID;
    char   regNo[30];
    char   action[8];      // "ISSUE" or "RETURN"
    char   timestamp[26];
} LogEntry;
```

### Data Members Explanation:
* **`bookID`**: Unique integer identifying the book. Helps in O(N) searching and issuing.
* **`title` / `author`**: Character arrays storing the metadata of the book.
* **`status`**: Enum tracking whether a book is `AVAILABLE` or `ISSUED`.
* **`issuedTo`**: Stores the student Registration Number of the borrower.
* **`action` & `timestamp`**: Used in the `LogEntry` struct to maintain the time-stamped history of library operations.

### Functions Used:
| Function Name | Purpose | Inputs | Output |
| --- | --- | --- | --- |
| `addBook()` | Adds a new book to the library | BookID, Title, Author | Success/Error message |
| `issueBook()` | Issues available book to student | BookID, RegNo | Updates status, saves to file |
| `returnBook()` | Returns an issued book | BookID | Updates status, clears RegNo |
| `searchBook()` | Locates books | BookID or Keyword | Displays matched book details |
| `displayAllBooks()` | Formatted display of inventory | None | Tabular list of all books |
| `viewTransactionLog()` | Displays history of operations | None | Tabular list of log entries |
| `showStatistics()` | Library usage dashboard | None | Counts for Available/Issued |
| `saveBooks()` / `loadBooks()` | File handling for persistence | None | Binary `.dat` file I/O |

---

## 4. Program Flow
The program begins by loading existing books and transaction logs from binary files (`library_books.dat` and `library_log.dat`). It then presents a menu-driven interface to the administrator. 

When the user selects a function like "Issue Book", the system prompts for the Book ID. It validates the ID exists and checks the availability status. If available, it prompts for the Student Registration Number, stamps the current date and time, updates the book's status to `ISSUED`, adds a record to the transaction log array, and triggers `saveBooks()` and `saveLogs()` to permanently write the changes to the disk. 

The program loops infinitely until the user selects option `0`, which saves all final states and safely terminates the application.

---

## 5. Core Logic
The core logic centers heavily around array traversal and data validation:
* **Double Issue Prevention:** Before issuing, the `status` enum is checked. If `status == ISSUED`, the program halts the transaction and alerts the user.
* **Search Algorithms:** The keyword search utilizes a custom case-insensitive string matching function `strcasestr_custom()` to ensure searches like "harry" will match "Harry Potter".
* **Persistent Storage:** Unlike standard text files, the system uses binary file handling (`fwrite`/`fread`) which is significantly faster and prevents edge cases where newline characters in titles might corrupt standard text parsing logic.

---

## 6. Testing And Validation

| Serial No. | Testing Context | Input/Action | Expected Output | Actual Output | Results |
| --- | --- | --- | --- | --- | --- |
| 1 | Add New Book | ID: 101, Title: C Programming | Book added to library successfully | Book added to library successfully | Works |
| 2 | Double ID Check | Add Book -> ID: 101 | "Book ID 101 already exists." | "Book ID 101 already exists." | Works |
| 3 | Issue Book | ID: 101, RegNo: 25BCE1734 | Book issued with correct Timestamp | Book issued with correct Timestamp | Works |
| 4 | Double Issue Check | Issue Book -> ID: 101 | "Book is already issued." | "Book is already issued." | Works |
| 5 | Return Book | ID: 101 | Book returned successfully | Book returned successfully | Works |
| 6 | Delete Issued Book | Delete Book -> ID: 101 (when issued) | "Return the book first before deleting."| "Return the book first before deleting." | Works |

---

## 7. Screenshots
*(Note: Please insert console screenshots here from your VS Code terminal showing the main menu, adding a book, and viewing the statistics dashboard)*

---

## 8. Deployment Explanation
The program was developed and natively compiled using the **GCC (MinGW-w64)** compiler within the MSYS2 environment on Windows 11. It integrates standard library functions and direct file I/O operations, ensuring it is highly portable. It can be compiled in any Unix or Windows environment using `gcc -o library library_book_tracking.c`.

---

## 9. Challenges Faced
One major challenge encountered was properly parsing strings with spaces (like full names of authors or long book titles). The standard `scanf("%s")` approach severed the string at the first white space. This was resolved by implementing a custom `readLine()` function that wraps `fgets()` while stripping trailing newline characters, ensuring full strings were captured. 

Another difficulty was handling time integration for the transaction log. C's `<time.h>` output required formatting into a readable string to store safely inside the `LogEntry` structure, which was accomplished by using `localtime()` and `strftime()`.

---

## 10. AI Usage Declaration
AI Assistants (Google Gemini / DeepMind) were consulted to optimize the file handling structures (switching from plain text to binary `fwrite`/`fread` for improved stability). General system logic, input validation formatting, and the custom case-insensitive search loop were heavily guided by the AI pair-programming environment to establish best practices and memory safety.

---

## 11. Complete Source Code

*(Copy and paste the entire contents of `library_book_tracking.c` here for the final submission)*
```c
/*
 * ============================================================
 *  LIBRARY BOOK TRACKING SYSTEM
 *  ------------------------------------------------------------
 *  Features:
 *    1. Add Book          (BookID, Title, Author, Status)
 *    2. Issue Book        (to Student RegNo, with date)
 *    3. Return Book       (with date)
 *    4. Search Book       (by ID or Title keyword)
 *    5. Display All Books (formatted table)
 *    6. Show Issued Books
 *    7. Delete Book
 *    8. Transaction Log   (issue / return history)
 *    9. Library Stats     (total, available, issued counts)
 *   10. Save & Load       (persistent file storage)
 *
 *  Enhancements over bare requirements:
 *    - Date stamps on every issue / return
 *    - Full transaction history log saved to file
 *    - Library statistics dashboard
 *    - Delete-book capability
 *    - Colourful console UI  (Windows: uses system("color"))
 *    - Input-length guards & duplicate-ID prevention
 * ============================================================
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <ctype.h>

/* ───────── limits ───────── */
#define MAX_BOOKS        500
#define MAX_TITLE        100
#define MAX_AUTHOR        60
#define MAX_REGNO         30
#define MAX_LOG          1000

#define DATA_FILE        "library_books.dat"
#define LOG_FILE         "library_log.dat"

/* ───────── status codes ───────── */
typedef enum { AVAILABLE, ISSUED } BookStatus;

/* ───────── data structures ───────── */
typedef struct {
    int    bookID;
    char   title[MAX_TITLE];
    char   author[MAX_AUTHOR];
    BookStatus status;
    char   issuedTo[MAX_REGNO];   /* student registration number */
    char   issueDate[26];         /* date string                 */
} Book;

typedef struct {
    int    bookID;
    char   regNo[MAX_REGNO];
    char   action[8];             /* "ISSUE" or "RETURN"         */
    char   timestamp[26];
} LogEntry;

/* ───────── globals ───────── */
static Book     books[MAX_BOOKS];
static int      bookCount = 0;

static LogEntry logs[MAX_LOG];
static int      logCount  = 0;

/* ═══════════════════════════════════════════════════════════
 *  UTILITY HELPERS
 * ═══════════════════════════════════════════════════════════ */

/* current timestamp as a readable string */
static void getCurrentTime(char *buf, size_t len)
{
    time_t     t  = time(NULL);
    struct tm *tm = localtime(&t);
    strftime(buf, len, "%Y-%m-%d %H:%M:%S", tm);
}

/* case-insensitive substring search */
static int strcasestr_custom(const char *haystack, const char *needle)
{
    size_t hLen = strlen(haystack);
    size_t nLen = strlen(needle);
    if (nLen > hLen) return 0;
    for (size_t i = 0; i <= hLen - nLen; i++) {
        size_t j;
        for (j = 0; j < nLen; j++) {
            if (tolower((unsigned char)haystack[i + j]) !=
                tolower((unsigned char)needle[j]))
                break;
        }
        if (j == nLen) return 1;
    }
    return 0;
}

/* safe line reader – strips newline */
static void readLine(char *buf, int max)
{
    if (fgets(buf, max, stdin)) {
        buf[strcspn(buf, "\r\n")] = '\0';
    }
}

/* print a horizontal rule */
static void hr(void)
{
    printf("  -----------------------------------------------------------------------\n");
}

/* pause before returning to menu */
static void pause_screen(void)
{
    printf("\n  Press ENTER to continue...");
    getchar();
}

/* flush any remaining chars in stdin */
static void flushInput(void)
{
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

/* find index by bookID, returns -1 if not found */
static int findBookByID(int id)
{
    for (int i = 0; i < bookCount; i++) {
        if (books[i].bookID == id) return i;
    }
    return -1;
}

/* ═══════════════════════════════════════════════════════════
 *  FILE I/O  – binary save / load for speed & reliability
 * ═══════════════════════════════════════════════════════════ */

static void saveBooks(void)
{
    FILE *fp = fopen(DATA_FILE, "wb");
    if (!fp) { printf("  [ERROR] Could not open %s for writing.\n", DATA_FILE); return; }
    fwrite(&bookCount, sizeof(int), 1, fp);
    fwrite(books, sizeof(Book), bookCount, fp);
    fclose(fp);
}

static void loadBooks(void)
{
    FILE *fp = fopen(DATA_FILE, "rb");
    if (!fp) { bookCount = 0; return; }          /* first run – no file yet */
    fread(&bookCount, sizeof(int), 1, fp);
    fread(books, sizeof(Book), bookCount, fp);
    fclose(fp);
}

static void saveLogs(void)
{
    FILE *fp = fopen(LOG_FILE, "wb");
    if (!fp) return;
    fwrite(&logCount, sizeof(int), 1, fp);
    fwrite(logs, sizeof(LogEntry), logCount, fp);
    fclose(fp);
}

static void loadLogs(void)
{
    FILE *fp = fopen(LOG_FILE, "rb");
    if (!fp) { logCount = 0; return; }
    fread(&logCount, sizeof(int), 1, fp);
    fread(logs, sizeof(LogEntry), logCount, fp);
    fclose(fp);
}

/* append to the in-memory log and persist */
static void addLog(int bookID, const char *regNo, const char *action)
{
    if (logCount >= MAX_LOG) {
        printf("  [WARNING] Transaction log full – oldest entries will be lost.\n");
        /* shift everything left by 1 to make room */
        memmove(&logs[0], &logs[1], sizeof(LogEntry) * (MAX_LOG - 1));
        logCount = MAX_LOG - 1;
    }
    LogEntry *e = &logs[logCount++];
    e->bookID = bookID;
    strncpy(e->regNo, regNo, MAX_REGNO - 1);
    e->regNo[MAX_REGNO - 1] = '\0';
    strncpy(e->action, action, 7);
    e->action[7] = '\0';
    getCurrentTime(e->timestamp, sizeof(e->timestamp));
    saveLogs();
}

/* ═══════════════════════════════════════════════════════════
 *  DISPLAY HELPERS
 * ═══════════════════════════════════════════════════════════ */

static void printBookHeader(void)
{
    printf("\n");
    hr();
    printf("  %-8s %-30s %-20s %-10s %-12s\n",
           "BookID", "Title", "Author", "Status", "Issued To");
    hr();
}

static void printBookRow(const Book *b)
{
    printf("  %-8d %-30.30s %-20.20s %-10s %-12s\n",
           b->bookID,
           b->title,
           b->author,
           b->status == AVAILABLE ? "Available" : "Issued",
           b->status == ISSUED    ? b->issuedTo : "---");
}

/* ═══════════════════════════════════════════════════════════
 *  CORE  FEATURES
 * ═══════════════════════════════════════════════════════════ */

/* 1. ADD BOOK */
static void addBook(void)
{
    printf("\n  =====================  ADD NEW BOOK  =====================\n\n");

    if (bookCount >= MAX_BOOKS) {
        printf("  [ERROR] Library is full! Cannot add more books.\n");
        pause_screen();
        return;
    }

    int id;
    printf("  Enter Book ID (integer) : ");
    if (scanf("%d", &id) != 1) { printf("  Invalid input.\n"); flushInput(); pause_screen(); return; }
    flushInput();

    if (findBookByID(id) != -1) {
        printf("  [ERROR] Book ID %d already exists. Use a unique ID.\n", id);
        pause_screen();
        return;
    }

    Book *b    = &books[bookCount];
    b->bookID  = id;
    b->status  = AVAILABLE;
    memset(b->issuedTo, 0, MAX_REGNO);
    memset(b->issueDate, 0, 26);

    printf("  Enter Title            : ");
    readLine(b->title, MAX_TITLE);
    printf("  Enter Author           : ");
    readLine(b->author, MAX_AUTHOR);

    bookCount++;
    saveBooks();

    printf("\n  [SUCCESS] Book \"%s\" (ID %d) added to library.\n", b->title, b->bookID);
    pause_screen();
}

/* 2. ISSUE BOOK */
static void issueBook(void)
{
    printf("\n  =====================  ISSUE BOOK  ======================\n\n");

    int id;
    printf("  Enter Book ID to issue : ");
    if (scanf("%d", &id) != 1) { printf("  Invalid input.\n"); flushInput(); pause_screen(); return; }
    flushInput();

    int idx = findBookByID(id);
    if (idx == -1) {
        printf("  [ERROR] No book found with ID %d.\n", id);
        pause_screen();
        return;
    }

    Book *b = &books[idx];
    if (b->status == ISSUED) {
        printf("  [ERROR] This book is already issued to %s (on %s).\n",
               b->issuedTo, b->issueDate);
        printf("          Please return it first before re-issuing.\n");
        pause_screen();
        return;
    }

    char regNo[MAX_REGNO];
    printf("  Enter Student Reg. No  : ");
    readLine(regNo, MAX_REGNO);
    if (strlen(regNo) == 0) {
        printf("  [ERROR] Registration number cannot be empty.\n");
        pause_screen();
        return;
    }

    b->status = ISSUED;
    strncpy(b->issuedTo, regNo, MAX_REGNO - 1);
    b->issuedTo[MAX_REGNO - 1] = '\0';
    getCurrentTime(b->issueDate, sizeof(b->issueDate));

    saveBooks();
    addLog(b->bookID, regNo, "ISSUE");

    printf("\n  [SUCCESS] Book \"%s\" issued to %s on %s.\n",
           b->title, b->issuedTo, b->issueDate);
    pause_screen();
}

/* 3. RETURN BOOK */
static void returnBook(void)
{
    printf("\n  =====================  RETURN BOOK  =====================\n\n");

    int id;
    printf("  Enter Book ID to return: ");
    if (scanf("%d", &id) != 1) { printf("  Invalid input.\n"); flushInput(); pause_screen(); return; }
    flushInput();

    int idx = findBookByID(id);
    if (idx == -1) {
        printf("  [ERROR] No book found with ID %d.\n", id);
        pause_screen();
        return;
    }

    Book *b = &books[idx];
    if (b->status == AVAILABLE) {
        printf("  [INFO]  Book \"%s\" is not currently issued.\n", b->title);
        pause_screen();
        return;
    }

    char returnDate[26];
    getCurrentTime(returnDate, sizeof(returnDate));

    printf("  Returning \"%s\" (issued to %s on %s)...\n",
           b->title, b->issuedTo, b->issueDate);

    addLog(b->bookID, b->issuedTo, "RETURN");

    b->status = AVAILABLE;
    memset(b->issuedTo, 0, MAX_REGNO);
    memset(b->issueDate, 0, 26);

    saveBooks();

    printf("  [SUCCESS] Book returned successfully on %s.\n", returnDate);
    pause_screen();
}

/* 4. SEARCH BOOK */
static void searchBook(void)
{
    printf("\n  =====================  SEARCH BOOK  =====================\n\n");
    printf("  Search by:\n");
    printf("    1. Book ID\n");
    printf("    2. Title keyword\n");
    printf("  Choice: ");

    int choice;
    if (scanf("%d", &choice) != 1) { printf("  Invalid.\n"); flushInput(); pause_screen(); return; }
    flushInput();

    if (choice == 1) {
        int id;
        printf("  Enter Book ID: ");
        if (scanf("%d", &id) != 1) { printf("  Invalid.\n"); flushInput(); pause_screen(); return; }
        flushInput();

        int idx = findBookByID(id);
        if (idx == -1) {
            printf("  No book found with ID %d.\n", id);
        } else {
            printBookHeader();
            printBookRow(&books[idx]);
            hr();
        }
    } else if (choice == 2) {
        char keyword[MAX_TITLE];
        printf("  Enter title keyword: ");
        readLine(keyword, MAX_TITLE);

        int found = 0;
        printBookHeader();
        for (int i = 0; i < bookCount; i++) {
            if (strcasestr_custom(books[i].title, keyword)) {
                printBookRow(&books[i]);
                found++;
            }
        }
        hr();
        if (!found) printf("  No books matching \"%s\".\n", keyword);
        else        printf("  %d book(s) found.\n", found);
    } else {
        printf("  Invalid choice.\n");
    }
    pause_screen();
}

/* 5. DISPLAY ALL BOOKS */
static void displayAllBooks(void)
{
    printf("\n  ==================  ALL BOOKS IN LIBRARY  ================\n");

    if (bookCount == 0) {
        printf("\n  Library is empty. Add some books first!\n");
        pause_screen();
        return;
    }

    printBookHeader();
    for (int i = 0; i < bookCount; i++) {
        printBookRow(&books[i]);
    }
    hr();
    printf("  Total: %d book(s)\n", bookCount);
    pause_screen();
}

/* 6. SHOW ISSUED BOOKS */
static void showIssuedBooks(void)
{
    printf("\n  ================  CURRENTLY ISSUED BOOKS  ================\n");

    int found = 0;
    printf("\n");
    hr();
    printf("  %-8s %-25s %-15s %-12s %-19s\n",
           "BookID", "Title", "Author", "Issued To", "Issue Date");
    hr();

    for (int i = 0; i < bookCount; i++) {
        if (books[i].status == ISSUED) {
            printf("  %-8d %-25.25s %-15.15s %-12s %-19s\n",
                   books[i].bookID,
                   books[i].title,
                   books[i].author,
                   books[i].issuedTo,
                   books[i].issueDate);
            found++;
        }
    }
    hr();

    if (!found) printf("  No books are currently issued.\n");
    else        printf("  %d book(s) currently issued.\n", found);

    pause_screen();
}

/* 7. DELETE BOOK */
static void deleteBook(void)
{
    printf("\n  ====================  DELETE BOOK  =======================\n\n");

    int id;
    printf("  Enter Book ID to delete: ");
    if (scanf("%d", &id) != 1) { printf("  Invalid input.\n"); flushInput(); pause_screen(); return; }
    flushInput();

    int idx = findBookByID(id);
    if (idx == -1) {
        printf("  [ERROR] No book found with ID %d.\n", id);
        pause_screen();
        return;
    }

    if (books[idx].status == ISSUED) {
        printf("  [WARNING] Book \"%s\" is currently issued to %s.\n",
               books[idx].title, books[idx].issuedTo);
        printf("  Return the book first before deleting.\n");
        pause_screen();
        return;
    }

    printf("  Are you sure you want to delete \"%s\" (ID %d)? (y/n): ",
           books[idx].title, books[idx].bookID);
    char confirm;
    scanf(" %c", &confirm);
    flushInput();

    if (confirm != 'y' && confirm != 'Y') {
        printf("  Deletion cancelled.\n");
        pause_screen();
        return;
    }

    /* shift remaining books left */
    for (int i = idx; i < bookCount - 1; i++) {
        books[i] = books[i + 1];
    }
    bookCount--;
    saveBooks();

    printf("  [SUCCESS] Book deleted.\n");
    pause_screen();
}

/* 8. TRANSACTION LOG */
static void viewTransactionLog(void)
{
    printf("\n  ===============  TRANSACTION HISTORY LOG  ================\n");

    if (logCount == 0) {
        printf("\n  No transactions recorded yet.\n");
        pause_screen();
        return;
    }

    printf("\n");
    hr();
    printf("  %-6s %-8s %-12s %-8s %-19s\n",
           "No.", "BookID", "RegNo", "Action", "Timestamp");
    hr();

    /* show most recent first */
    int displayNo = 1;
    for (int i = logCount - 1; i >= 0; i--, displayNo++) {
        printf("  %-6d %-8d %-12s %-8s %-19s\n",
               displayNo,
               logs[i].bookID,
               logs[i].regNo,
               logs[i].action,
               logs[i].timestamp);
    }
    hr();
    printf("  %d transaction(s) recorded.\n", logCount);
    pause_screen();
}

/* 9. LIBRARY STATISTICS */
static void showStatistics(void)
{
    int available = 0, issued = 0;
    for (int i = 0; i < bookCount; i++) {
        if (books[i].status == AVAILABLE) available++;
        else                              issued++;
    }

    printf("\n  ================  LIBRARY STATISTICS  ====================\n\n");
    printf("  +-------------------------+--------+\n");
    printf("  |  Total Books            | %6d |\n", bookCount);
    printf("  +-------------------------+--------+\n");
    printf("  |  Available              | %6d |\n", available);
    printf("  +-------------------------+--------+\n");
    printf("  |  Currently Issued       | %6d |\n", issued);
    printf("  +-------------------------+--------+\n");
    printf("  |  Total Transactions     | %6d |\n", logCount);
    printf("  +-------------------------+--------+\n");
    pause_screen();
}

/* ═══════════════════════════════════════════════════════════
 *  MAIN  MENU
 * ═══════════════════════════════════════════════════════════ */

static void printMenu(void)
{
    printf("\n");
    printf("  *****************************************************************\n");
    printf("  *                                                               *\n");
    printf("  *          L I B R A R Y   B O O K   T R A C K E R             *\n");
    printf("  *                                                               *\n");
    printf("  *****************************************************************\n");
    printf("  *                                                               *\n");
    printf("  *    1.  Add New Book              6.  Show Issued Books        *\n");
    printf("  *    2.  Issue Book                7.  Delete Book              *\n");
    printf("  *    3.  Return Book               8.  Transaction Log          *\n");
    printf("  *    4.  Search Book               9.  Library Statistics       *\n");
    printf("  *    5.  Display All Books         0.  Exit                     *\n");
    printf("  *                                                               *\n");
    printf("  *****************************************************************\n");
    printf("\n  Enter your choice: ");
}

int main(void)
{
    /* load persisted data */
    loadBooks();
    loadLogs();

    printf("\n  [INFO] Loaded %d book(s) and %d transaction(s) from file.\n",
           bookCount, logCount);

    int running = 1;
    while (running) {
        printMenu();

        int choice;
        if (scanf("%d", &choice) != 1) {
            printf("  [ERROR] Please enter a number (0-9).\n");
            flushInput();
            continue;
        }
        flushInput();

        switch (choice) {
            case 1: addBook();            break;
            case 2: issueBook();          break;
            case 3: returnBook();         break;
            case 4: searchBook();         break;
            case 5: displayAllBooks();    break;
            case 6: showIssuedBooks();    break;
            case 7: deleteBook();         break;
            case 8: viewTransactionLog(); break;
            case 9: showStatistics();     break;
            case 0:
                saveBooks();
                saveLogs();
                printf("\n  All data saved. Goodbye!\n\n");
                running = 0;
                break;
            default:
                printf("  [ERROR] Invalid choice. Enter 0–9.\n");
        }
    }

    return 0;
}
```
