SRCS:=$(wildcard *.c)
OBJS:=$(patsubst %.c,%.o,$(SRCS))
ELF:=a.out
CC:=gcc
CFLAGS:=-g -Wall 
$(ELF):$(OBJS)
	gcc -o $@ $^ -pthread -lcrypto 
clean:
	rm -rf $(OBJS) $(ELF)
