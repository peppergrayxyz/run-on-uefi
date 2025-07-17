.DEFAULT_GOAL=all

TARGET 		?= example
TARGET_CMD	?= "Hello, world!"
SOURCE 		?= $(TARGET).c
RUN_ON_UEFI	?= ../run_on_uefi
UEFI_ARCH 	?= $(shell $(RUN_ON_UEFI) arch)
ARCH 		:= $(shell [ "$(UEFI_ARCH)" = "arm" ]   && echo "arm"   || echo "$(ARCH)" )
ARCH 		:= $(shell [ "$(UEFI_ARCH)" = "ia32" ]  && echo "i386"  || echo "$(ARCH)" )
LIBS_ARCH	?= $(shell [ "$(UEFI_ARCH)" = "ia32" ]  && echo "ia32"  || echo "$(ARCH)" )
OUT_PATH	?= build
OUT_FILES	?= $(OUT_PATH)/$(TARGET)$(UEFI_ARCH)

HEADERS		?= /usr/include/efi
LIBS		?= /usr/lib

CC			?= gcc
CC_LD		?= ld
OBJCOPY		?= objcopy

# https://wiki.osdev.org/UEFI
$(OUT_FILES).efi: $(SOURCE)
	mkdir -p $(OUT_PATH)
	
	$(CC) $(SOURCE)                    \
	    -c                             \
	    -fno-stack-protector           \
	    -fpic                          \
	    -fshort-wchar                  \
	    -ffreestanding                 \
	    -I $(HEADERS)                  \
	    -I $(HEADERS)/$(ARCH)          \
	    -DEFI_FUNCTION_WRAPPER         \
	    -o $(OUT_FILES).o

	$(CC_LD) $(OUT_FILES).o            \
	    $(LIBS)/crt0-efi-$(LIBS_ARCH).o     \
	    -nostdlib                      \
	    -znocombreloc                  \
	    -T $(LIBS)/elf_$(LIBS_ARCH)_efi.lds \
	    -shared                        \
	    -Bsymbolic                     \
	    -L $(LIBS)                     \
	    -l:libgnuefi.a                 \
	    -l:libefi.a                    \
	    -o $(OUT_FILES).so

	$(OBJCOPY) -j .text               \
	    -j .sdata                     \
	    -j .data                      \
	    -j .dynamic                   \
	    -j .rodata                    \
	    -j .dynsym                    \
	    -j .rel                       \
	    -j .rela                      \
	    -j .reloc                     \
	    --target=efi-app-$(ARCH)      \
	    $(OUT_FILES).so               \
	    $(OUT_FILES).efi

.PHONY=all
install:
	NOCONFIRM=true $(RUN_ON_UEFI) install

.PHONY=all
build: $(OUT_FILES).efi

.PHONY=all
run:
	$(RUN_ON_UEFI) run "$(TARGET)"

.PHONY=validate
validate: 
	@printf "Hello, world!\r\n" | $(RUN_ON_UEFI) validate "$(TARGET)"

.PHONY=all
all: build run validate

.PHONY=clean
clean:
	@rm -rvf build

