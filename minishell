#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
#define MAX_LINE 512
#define MAX_ARGS 64
#define MAX_HISTORY 100

char history[MAX_HISTORY][MAX_LINE];
int  history_count = 0;

void save_history(char *command)			// 명령어 기록 저장
{
    int i;
    if (history_count == MAX_HISTORY) {
        for (i = 0; i < MAX_HISTORY - 1; i++) {
            strcpy(history[i], history[i + 1]);
        }
        history_count--;
    }
    strcpy(history[history_count], command);
    history_count++;
}

int split_command(char *command, char *args[])	// 명령어를 공백 기준으로 분리
{
    int count = 0;
    char *token;
    token = strtok(command, " ");
    while (token != NULL && count < MAX_ARGS - 1) {
        args[count] = token;
        count++;
        token = strtok(NULL, " ");
    }
    args[count] = NULL;				// execvp()는 NULL로 끝나는 배열 필요
    return count;
}

int run_builtin(int argc, char *args[])			// 내장 명령 처리
{
    char current_dir[MAX_LINE];
    int i;
    if (strcmp(args[0], "cd") == 0) {			// cd
        if (argc < 2) {
            chdir(getenv("HOME"));
        } else {
            if (chdir(args[1]) != 0) {
                perror("cd");
            }
        }
        return 1;
    }

    if (strcmp(args[0], "pwd") == 0) {			// pwd
        if (getcwd(current_dir, sizeof(current_dir)) != NULL) {
            printf("%s\n", current_dir);
        } else {
            perror("pwd");
        }
        return 1;
    }

    if (strcmp(args[0], "history") == 0) {			// history
        for (i = 0; i < history_count; i++) {
            printf("%d  %s\n", i + 1, history[i]);
        }
        return 1;
    }

    return 0;
}

int main(void)
{
    char  command[MAX_LINE];
    char  copy[MAX_LINE];
    char *args[MAX_ARGS];
    int   argc;
    int   length;
    pid_t pid;
    int   status;

    while (1) {
        printf("miniShell> ");					// 프롬프트 출력
        fflush(stdout);

        if (fgets(command, MAX_LINE, stdin) == NULL) {	//명령어 입력
            printf("\n");
            break;
        }

        length = strlen(command);
        if (length > 0 && command[length - 1] == '\n') {
            command[length - 1] = '\0';			// 개행 문자 제거
        }

        strcpy(copy, command);
        argc = split_command(copy, args);
        if (argc == 0) {
            continue;
        }

        save_history(command);

        if (strcmp(args[0], "exit") == 0) {				// exit 시 종료
            break;
        }

        if (run_builtin(argc, args) == 1) {			// 내장 명령은 fork 없이 처리
            continue;
        }
        pid = fork();						// 자식 프로세스 생성

        if (pid < 0) {
            perror("fork");
        }
        else if (pid == 0) {
            execvp(args[0], args);				// 자식: 명령 실행
            perror(args[0]);					// execvp() 실패 시에만 도달
            exit(1);
        }
        else {
            wait(&status);					// 부모: 자식 종료까지 대기
        }
    }
    printf("miniShell 종료\n");
    return 0;
}
