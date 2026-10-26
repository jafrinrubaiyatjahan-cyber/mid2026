#include <stdio.h>
#include <sqlite3.h>

// 데이터 검색 결과를 출력하기 위한 콜백 함수
static int callback(void* NotUsed, int argc, char** argv, char** azColName) {
    for (int i = 0; i < argc; i++) {
        printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
    }
    printf("\n");
    return 0;
}

int main() {
    sqlite3* db;
    char* err_msg = 0;

    // 1. 데이터베이스 연결 (school.db 파일 생성)
    int rc = sqlite3_open("school.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "데이터베이스 오픈 실패: %s\n", sqlite3_errmsg(db));
        return rc;
    }

    // 2. 테이블 생성
    const char* sql_create = "CREATE TABLE IF NOT EXISTS students ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT, "
        "name TEXT NOT NULL, "
        "grade INTEGER, "
        "major TEXT);";

    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL 오류 (테이블 생성): %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return rc;
    }

    // 3. 3명 이상의 데이터 삽입
    const char* sql_insert = "DELETE FROM students; "
        "INSERT INTO students (name, grade, major) VALUES ('김철수', 2, '컴퓨터공학'); "
        "INSERT INTO students (name, grade, major) VALUES ('이영희', 3, '경영학'); "
        "INSERT INTO students (name, grade, major) VALUES ('박민수', 1, '전자공학');";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL 오류 (데이터 삽입): %s\n", err_msg);
        sqlite3_free(err_msg);
    }
    else {
        printf("데이터 삽입 완료!\n");
    }

    // 4. 데이터 검색 및 출력
    printf("\n[검색 결과 출력]\n");
    const char* sql_select = "SELECT * FROM students;";
    rc = sqlite3_exec(db, sql_select, callback, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL 오류 (검색): %s\n", err_msg);
        sqlite3_free(err_msg);
    }

    sqlite3_close(db);
    return 0;
}
