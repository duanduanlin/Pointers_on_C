main:obj/main.o obj/foo.o obj/bar.o
	gcc -o main obj/main.o obj/foo.o obj/bar.o

obj/main.o:main.c
	gcc -c main.c -o obj/main.o

obj/foo.o:foo.c
	gcc -c foo.c -o obj/foo.o

obj/bar.o:bar.c
	gcc -c bar.c -o obj/bar.o

.PHONY:cleanall cleanobj

cleanall:cleanobj
	-rm main

cleanobj:
	-rm obj/*.o
