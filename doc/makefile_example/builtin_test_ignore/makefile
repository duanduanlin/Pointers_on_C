main:main.o foo.o bar.o
	gcc -0 main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -0 main.o

foo.o:foo.c
	-gcc -c foo.c -0 foo.o

bar.o:bar.c
	gcc -c bar.c -0 bar.o

.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm *.o

.IGNORE:main.o
