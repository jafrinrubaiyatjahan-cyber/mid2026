
# 2번 문항

```

#include <stdio.h>
#include "sqlite3.h" // Use double quotes to look for the header file in the current directory

int main() {
    sqlite3* db;
    char* err_msg = 0;

    // 1. Connect to the database (creates the file if it doesn't exist)
    int rc = sqlite3_open("school.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Cannot open database: %s\n", sqlite3_errmsg(db));
        return rc;
    }

    // 2. Create table
    const char* sql_create = "CREATE TABLE IF NOT EXISTS students ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT, "
        "name TEXT NOT NULL, "
        "grade INTEGER, "
        "major TEXT);";

    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL error (Table creation): %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return rc;
    }

    // 3. Insert data (more than 3 rows)
    const char* sql_insert = "DELETE FROM students; "
        "INSERT INTO students (name, grade, major) VALUES ('Kim Chul-soo', 2, 'Computer Science'); "
        "INSERT INTO students (name, grade, major) VALUES ('Lee Young-hee', 3, 'Business Administration'); "
        "INSERT INTO students (name, grade, major) VALUES ('Park Min-soo', 1, 'Electronic Engineering');";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL error (Data insertion): %s\n", err_msg);
        sqlite3_free(err_msg);
    }
    else {
        printf("Data inserted successfully!\n");
    }

    // 4. Data search (SELECT query placeholder)
    const char* sql_select = "SELECT * FROM students WHERE major LIKE '%Computer%';";
    // (To output SELECT results to the screen, you can connect a callback function or use sqlite3_prepare_v2)

    sqlite3_close(db);
    return 0;
}
```

<img width="1105" height="390" alt="image" src="https://github.com/user-attachments/assets/eeb13bf5-cdd6-4864-97a6-4cacd01ea968" />
