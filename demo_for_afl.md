```c
#include <stdio.h> 
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <signal.h>


int vuln(char *Data) {
	//printf("Data Size is :%d\n", Size);
	int num = rand() % 100 + 1;
	//printf("num is %d\n", num); 
	printf("Data is generated, num is %d\n", num);
	if(Data[0] == 'C' && num == 25)
	{
		raise(SIGSEGV);
	}
	else if(Data[0] == 'F' && num == 90)
	{
		raise(SIGSEGV);
	}
	else{
		printf("it is good!\n");	
	}
	return 0;
}

int main(int argc, char *argv[])
{
	char buf[40]={0};
	FILE *input = NULL;
	input = fopen(argv[1], "r");
	if(input != 0)
	{
		fscanf(input, "%s", &buf);
		printf("buf is %s\n", buf);
		vuln(buf);
		fclose(input);
	}
	else
	{
		printf("bad file!");
	}
	return 0;
}

```
