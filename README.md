[Untitled3 (1).ipynb](https://github.com/user-attachments/files/26380589/Untitled3.1.ipynb)
{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": []
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    }
  },
  "cells": [
    {
      "cell_type": "code",
      "execution_count": 5,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "6_t9TgCGNojX",
        "outputId": "74856111-52f5-4191-e4a8-53370ca181c8"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "1에서 40000까지의 합은 800020000입니다.\n"
          ]
        }
      ],
      "source": [
        "%%bash\n",
        "gcc -pthread -o thread_sum thread_sum.c\n",
        "./thread_sum"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "new_cell_id_1",
        "outputId": "e0ef6e07-05fc-431c-90a3-ebd8a4ef0b85"
      },
      "source": [
        "%%writefile thread_sum.c\n",
        "#include <pthread.h>\n",
        "#include <stdio.h>\n",
        "#include <stdlib.h>\n",
        "\n",
        "// 전역 변수 설정\n",
        "int sum = 0; // 모든 스레드가 공유할 합계 변수\n",
        "pthread_mutex_t mutex; // 공유 변수 보호를 위한 뮤텍스 객체\n",
        "\n",
        "// 스레드가 실행할 함수 (조건 3)\n",
        "void *runner(void *param) {\n",
        "    int id = *(int *)param;\n",
        "    int start = id * 10000 + 1; // 스레드별 시작 값 (1, 10001, 20001, 30001)\n",
        "    int end = (id + 1) * 10000;  // 스레드별 끝 값 (10000, 20000, 30000, 40000)\n",
        "\n",
        "    for (int i = start; i <= end; i++) {\n",
        "        // [임계 구역 보호 시작]\n",
        "        // PPT 64-65에서 설명하는 공유 자원 충돌을 방지하기 위해 lock을 겁니다.\n",
        "        pthread_mutex_lock(&mutex);\n",
        "\n",
        "        sum += i; // 공유 변수 sum에 접근\n",
        "\n",
        "        // [임계 구역 보호 종료]\n",
        "        pthread_mutex_unlock(&mutex);\n",
        "    }\n",
        "\n",
        "    free(param); // 할당된 메모리 해제\n",
        "    pthread_exit(0);\n",
        "}\n",
        "\n",
        "int main() {\n",
        "    pthread_t tid[4];\n",
        "\n",
        "    // 뮤텍스 초기화\n",
        "    pthread_mutex_init(&mutex, NULL);\n",
        "\n",
        "    // 조건 4: 4개의 스레드를 연속적으로 생성하여 동시에 실행\n",
        "    for (int i = 0; i < 4; i++) {\n",
        "        int *id = malloc(sizeof(int));\n",
        "        *id = i;\n",
        "        if (pthread_create(&tid[i], NULL, runner, id) != 0) {\n",
        "            fprintf(stderr, \"스레드 생성 실패\\n\");\n",
        "            return 1;\n",
        "        }\n",
        "    }\n",
        "\n",
        "    // 조건 5: 4개의 스레드가 모두 종료하기를 기다림 (Join)\n",
        "    for (int i = 0; i < 4; i++) {\n",
        "        pthread_join(tid[i], NULL);\n",
        "    }\n",
        "\n",
        "    // 결과 출력\n",
        "    printf(\"1에서 40000까지의 합은 %d입니다.\\n\", sum);\n",
        "\n",
        "    // 뮤텍스 제거\n",
        "    pthread_mutex_destroy(&mutex);\n",
        "\n",
        "    return 0;\n",
        "}"
      ],
      "execution_count": 6,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Overwriting thread_sum.c\n"
          ]
        }
      ]
    }
  ]
}
