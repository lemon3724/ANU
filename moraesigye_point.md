```
#include <stdio.h>

//빈공간을 채워주는 함수
void space(int s)
{
  while (s--)
  {
    printf(" ");
  }
}

//포인트(*)를 찍는 함수
void point(int i)
{
  for(int j = 0; j < i; j++)
  {
    printf("*");
  }
}


int main(void)
{
  int n, s = 0;
  scanf_s("%d", &n);

  //입력받은 값이 0또는 짝수라면 종료
  if(n % 2 == 0)
  {
    printf("0또는 짝수가 입력되었습니다.");
    return 0;
  }

  //모래시계의 상단부분
  for(int i=n; i>0; i-=2)
  {
    space(s);
    point(i);
    space(s);
    s++;
    printf("\n");
  }

  //모래시계의 하단부분
  s--;
  for(int i=3; i<=n; i+=2)
  {
    s--;
    space(s);
    point(i);
    space(s);
    printf("\n");
  }

  return 0;
}


/*
소감

C언어에 손을 놓은 지 조금 됐지만, 이번 기회에 다시 기억을 되살리면서 해보니깐 예전 기억이 새록새록 나면서 자연스럽게 작성하게 되었다.
*/
```
