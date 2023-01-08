export TARGET=i686-elf
# export PATH="$HOME/opt/cross/bin:$PATH"

# $@ = target file
# $< = first dependency
# $^ = all dependencies

CPPSOURCES = $(wildcard kernel/*.cc libk/*.cc)
HEADERS = $(wildcard kernel/*.hh libk/*.hh)
OBJ = ${CPPSOURCES:.cc=.o}

CC = i686-elf-g++
CFLAGS = -g -std=c++17 -ffreestanding -nostdlib -lgcc -Wall -O2 -fno-threadsafe-statics
GDB = gdb

CRTI = boot/crti.o
CRTN_OBJ = boot/crtn.o
CRT0_OBJ = boot/crt0.o
CRTBEGIN_OBJ:=$(shell $(CC) $(CFLAGS) -print-file-name=crtbegin.o)
CRTEND_OBJ:=$(shell $(CC) $(CFLAGS) -print-file-name=crtend.o)
OBJ_LINKS:= $(CRT0_OBJ) $(CRTI_OBJ) $(CRTBEGIN_OBJ) $(OBJ) $(CRTEND_OBJ) $(CRTN_OBJ)

all: run

goosyos.bin: kernel.bin
	cat $^ > $@

kernel.bin: ${OBJ_LINKS}
	i686-elf-ld -use-linker-plugin -o $@ --script=linker.ld $^ --oformat binary

kernel.elf: ${OBJ_LINKS}
	i686-elf-ld -use-linker-plugin -o $@ --script=linker.ld $^

dump: kernel.elf
	objdump -d kernel.elf > kerneldump.txt

run: goosyos.bin
	qemu-system-i386 -hda $<

run-debug: goosyos.bin
	qemu-system-i386 -hda $< -d int -no-reboot -no-shutdown

run-console: goosyos.bin
	qemu-system-i386 -hda $< -display curses

gdb: goosyos.bin kernel.elf
	qemu-system-i386 -hda $< -S -s -d int -no-reboot -no-shutdown & \
	${GDB}

%.o: %.cc ${HEADERS}
	${CC} ${CFLAGS} -c $< -o $@

%.o: %.s
	nasm $< -f elf -o $@

%.bin: %.s
	nasm $< -f bin -o $@

clean:
	rm -rf *.bin *.dis *.o goosyos.bin *.elf
	rm -rf *.o boot/*.bin boot/*.o klib/*.o klib/*/*.o kernel/*.o obj/*