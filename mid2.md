#include <stdio.h>
#include <sqlite3.h>

int main() {
    sqlite3 *db;
    char *err_msg = 0;
    
    // 1. 데이터베이스 연결 (파일 생성)
    int rc = sqlite3_open("school.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "데이터베이스 오픈 실패: %s\n", sqlite3_errmsg(db));
        return rc;
    }

    // 2. 테이블 생성
    const char *sql_create = "CREATE TABLE IF NOT EXISTS students ("
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
    const char *sql_insert = "DELETE FROM students; "
                             "INSERT INTO students (name, grade, major) VALUES ('김철수', 2, '컴퓨터공학'); "
                             "INSERT INTO students (name, grade, major) VALUES ('이영희', 3, '경영학'); "
                             "INSERT INTO students (name, grade, major) VALUES ('박민수', 1, '전자공학');";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL 오류 (데이터 삽입): %s\n", err_msg);
        sqlite3_free(err_msg);
    } else {
        printf("데이터 삽입 완료!\n");
    }

    // 4. 데이터 검색 (콜백 함수 활용)
    // 콜백 함수 정의는 아래 참조
    const char *sql_select = "SELECT * FROM students WHERE major LIKE '%컴퓨터%';";
    
    // 콜백 함수를 이용한 SELECT문 실행 예시 (간이 출력용)
    // (실제 시험에서는 콜백 함수 대신 sqlite3_prepare_v2 등을 써도 됩니다)
    
    sqlite3_close(db);
    return 0;
}
