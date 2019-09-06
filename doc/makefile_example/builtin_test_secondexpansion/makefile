run:first second 
	@echo target run is comming.

arg=hello

.SECONDEXPANSION:

first:$(arg)
	@echo target $@ is comming by $^
second:$$(arg)
	@echo target $@ is comming by $^

arg = world
.DEFAULT:
	@echo $@ is not exist.

