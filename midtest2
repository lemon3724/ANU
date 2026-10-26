#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h>

// SELECT 결과를 출력하기 위한 콜백 함수
static int callback(void *NotUsed, int argc, char **argv, char **azColName) {
    for (int i = 0; i < argc; i++) {
        printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
    }
    printf("-------------------------\n");
    return 0;
}

int main(void) {
    sqlite3 *db;
    char *err_msg = 0;
    int rc;

    // 1. 데이터베이스 연결 (파일이 없으면 자동 생성)
    rc = sqlite3_open("school.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "데이터베이스를 열 수 없습니다: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }
    printf("데이터베이스(school.db) 연결 성공!\n\n");

    // 2. 테이블 생성 (students 테이블)
    const char *sql_create = "CREATE TABLE IF NOT EXISTS students ("
                             "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                             "name TEXT NOT NULL, "
                             "age INTEGER, "
                             "major TEXT);";
    
    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "테이블 생성 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    // 3. 데이터 삽입 (3개 이상의 데이터 입력)
    // 중복 실행 방지를 위해 기존 데이터를 지우고 새로 넣거나, INSERT OR IGNORE 등을 쓸 수 있으나 단순화를 위해 직접 삽입 쿼리 실행
    const char *sql_insert = "INSERT INTO students (name, age, major) VALUES "
                             "('김철수', 21, '컴퓨터공학'),"
                             "('이영희', 22, '전자공학'),"
                             "('박민수', 20, '인공지능학'),"
                             "('정수진', 23, '컴퓨터공학');";
    
    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        // 이미 데이터가 존재하여 UNIQUE 제약 등이 걸릴 경우를 대비한 메시지 처리
        fprintf(stderr, "데이터 삽입 메시지: %s (이미 존재할 수 있음)\n", err_msg);
        sqlite3_free(err_msg);
    } else {
        printf("3개 이상의 학생 데이터가 성공적으로 추가되었습니다.\n\n");
    }

    // 4. 전체 데이터 조회 (SELECT)
    printf("=== [1] 전체 학생 목록 조회 ===\n");
    const char *sql_select_all = "SELECT * FROM students;";
    rc = sqlite3_exec(db, sql_select_all, callback, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "조회 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
    }

    // 5. 조건 검색 (컴퓨터공학 전공인 학생 검색 - Prepared Statement 활용)
    printf("\n=== [2] 조건 검색 (전공: 컴퓨터공학) ===\n");
    const char *sql_query = "SELECT id, name, age, major FROM students WHERE major = ?;";
    sqlite3_stmt *stmt;
    
    rc = sqlite3_prepare_v2(db, sql_query, -1, &stmt, 0);
    if (rc == SQLITE_OK) {
        // 바인딩: 첫 번째 '?' 자리에 "컴퓨터공학" 문자열 대입 (SQL Injection 방지)
        sqlite3_bind_text(stmt, 1, "컴퓨터공학", -1, SQLITE_STATIC);

        while (sqlite3_step(stmt) == SQLITE_ROW) {
            int id = sqlite3_column_int(stmt, 0);
            const unsigned char *name = sqlite3_column_text(stmt, 1);
            int age = sqlite3_column_int(stmt, 2);
            const unsigned char *major = sqlite3_column_text(stmt, 3);

            printf("ID: %d | 이름: %s | 나이: %d | 전공: %s\n", id, name, age, major);
        }
    } else {
        fprintf(stderr, "검색 쿼리 준비 실패\n");
    }

    // 메모리 정리 및 연결 종료
    sqlite3_finalize(stmt);
    sqlite3_close(db);
    printf("\n데이터베이스 연결을 안전하게 종료했습니다.\n");

    return 0;
}
