
first foo.o:foo.c
	gcc -c foo.c -o $@ 

main:main.o foo.o bar.o
	gcc -o main main.o foo.o bar.o

main.o:main.c
	gcc -c main.c -o main.o

bar.o:bar.c
	gcc -c bar.c -o bar.o
