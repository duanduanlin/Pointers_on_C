main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

foo.o:foo.c
	gcc -c foo.c -o foo.o

bar.o:bar.c
	gcc -c bar.c -o bar.o

.INTERMEDIATE:main.o foo.o bar.o

.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm *.o
