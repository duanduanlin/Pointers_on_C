
.SUFFIXES:.a .b

target:file a.b b.b c.b
	touch target
.a.b:
	touch $(patsubst %.a,%.b,$^)

file:a.a b.a c.a

%.a:
	touch $@

.PHONY:cleanall cleanb cleana

cleanall:cleanb cleana
	-rm target

cleanb:
	-rm *.b

cleana:
	-rm *.a

