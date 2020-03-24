LEPTON_NAME = leptonica-1.79.0
CURL_NAME	= curl-7.69.1
PNG_NAME    = libpng-1.6.34
JPEG_NAME   = jpeg-9d
TIFF_NAME   = tiff-4.1.0

SDK_IPHONEOS_PATH=$(shell xcrun --sdk iphoneos --show-sdk-path)
SDK_IPHONESIMULATOR_PATH=$(shell xcrun --sdk iphonesimulator --show-sdk-path)
XCODE_DEVELOPER_PATH=/Applications/Xcode.app/Contents/Developer
XCODETOOLCHAIN_PATH=$(XCODE_DEVELOPER_PATH)/Toolchains/XcodeDefault.xctoolchain
IOS_DEPLOY_TGT="8.0"

TESSERACT_SRC = $(shell pwd)/tesseract-ocr
LEPTON_SRC = $(shell pwd)/$(LEPTON_NAME)
CURL_SRC = $(shell pwd)/$(CURL_NAME)
IMAGE_SRC = $(shell pwd)/libtiff-ios
PNG_SRC   = $(IMAGE_SRC)/$(PNG_NAME)
JPEG_SRC = $(IMAGE_SRC)/$(JPEG_NAME)
TIFF_SRC = $(IMAGE_SRC)/$(TIFF_NAME)

IMAGE_LIB_DIR = $(IMAGE_SRC)/dependencies/lib/
IMAGE_INC_DIR = $(IMAGE_SRC)/dependencies/include/
INCLUDE_DIR   = $(shell pwd)/include
LEPT_INC_DIR  = $(INCLUDE_DIR)
TESS_INC_DIR  = $(INCLUDE_DIR)
CURL_INC_DIR  = $(INCLUDE_DIR)
LIB_FAT_DIR   = $(shell pwd)/lib

libtessfiles = libtesseract.a
libleptfiles = liblept.a
libcurlfiles = libcurl.a
libpngfiles = libpng.a
libjpegfiles = libjpeg.a
libtifffiles = libtiff.a

sdks = $(SDK_IPHONEOS_PATH) $(SDK_IPHONEOS_PATH) $(SDK_IPHONEOS_PATH) $(SDK_IPHONESIMULATOR_PATH) $(SDK_IPHONESIMULATOR_PATH)
archs_all = armv7 armv7s arm64 i386 x86_64
arch_names_all = arm-apple-darwin7 arm-apple-darwin7s arm-apple-darwin64 i386-apple-darwin x86_64-apple-darwin
arch_names = $(foreach arch, $(ARCHS), $(call swap, $(arch), $(archs_all), $(arch_names_all) ) )
ARCHS ?= $(archs_all)

libleptfolders = $(foreach arch, $(arch_names), $(LEPTON_SRC)/$(arch)/)
libtessfolders = $(foreach arch, $(arch_names), $(TESSERACT_SRC)/$(arch)/)
libcurlfolders = $(foreach arch, $(arch_names), $(CURL_SRC)/$(arch)/)
libleptfolders_all = $(foreach arch, $(arch_names_all), $(LEPTON_SRC)/$(arch)/)
libtessfolders_all = $(foreach arch, $(arch_names_all), $(TESSERACT_SRC)/$(arch)/)
libcurlfolders_all = $(foreach arch, $(arch_names_all), $(CURL_SRC)/$(arch)/)

libleptmakefile = $(foreach folder, $(libleptfolders), $(addprefix $(folder), Makefile) )
libtessmakefile = $(foreach folder, $(libtessfolders), $(addprefix $(folder), Makefile) )
libcurlmakefile = $(foreach folder, $(libcurlfolders), $(addprefix $(folder), Makefile) )
imagesmakefile  = $(addprefix $(IMAGE_SRC)/, Makefile)

libleptfat = $(LIB_FAT_DIR)/$(libleptfiles)
libtessfat = $(LIB_FAT_DIR)/$(libtessfiles)
libcurlfat = $(LIB_FAT_DIR)/$(libcurlfiles)
imagesfat  = $(libpngfat) $(libjpegfat) $(libtifffat)
libpngfat  = $(LIB_FAT_DIR)/$(libpngfiles)
libjpegfat = $(LIB_FAT_DIR)/$(libjpegfiles)
libtifffat = $(LIB_FAT_DIR)/$(libtifffiles)

libtess    = $(foreach folder, $(libtessfolders), $(addprefix $(folder)/lib/, $(libtessfiles)) )
liblept    = $(foreach folder, $(libleptfolders), $(addprefix $(folder)/lib/, $(libleptfiles)) )
libcurl    = $(foreach folder, $(libcurlfolders), $(addprefix $(folder)/lib/, $(libcurlfiles)) )
images     = $(libpng) $(libjpeg) $(libtiff)
libpng     = $(addprefix $(IMAGE_LIB_DIR), $(libpngfiles))
libjpeg    = $(addprefix $(IMAGE_LIB_DIR), $(libjpegfiles))
libtiff    = $(addprefix $(IMAGE_LIB_DIR), $(libtifffiles))

libtessautogen = $(TESSERACT_SRC)/autogen.sh
libtessconfig = $(TESSERACT_SRC)/configure
libleptconfig = $(LEPTON_SRC)/configure
libcurlconfig = $(CURL_SRC)/configure

index = $(words $(shell a="$(2)";echo $${a/$(1)*/$(1)} ))
swap  = $(word $(call index,$(1),$(2)),$(3))

dependant_libs = $(libtessfat) $(libleptfat) $(libpngfat) $(libjpegfat) $(libtifffat) $(libcurlfat)

common_cflags = -L$(LIB_FAT_DIR) -Qunused-arguments -arch $(call swap, $*, $(arch_names_all), $(archs_all)) -pipe -no-cpp-precomp -isysroot $$SDKROOT -miphoneos-version-min=$(IOS_DEPLOY_TGT) -O2

.PHONY : all
all : $(dependant_libs)

#######################
# TESSERACT-OCR
#######################
$(libtessfat) : $(libtess)
	mkdir -p $(LIB_FAT_DIR)
	xcrun lipo $(realpath $(addsuffix lib/$(@F), $(libtessfolders_all)) ) -create -output $@
	mkdir -p $(TESS_INC_DIR)
	cp -rvf $(firstword $(libtessfolders))/include/tesseract $(TESS_INC_DIR)

$(libtess) : $(libtessmakefile)
	cd $(abspath $(@D)/..) && $(MAKE) -sj8 && $(MAKE) install

$(TESSERACT_SRC)/%/Makefile : $(libtessconfig) $(libleptfat) $(libcurlfat)
	export LIBS="-lz -lpng -ljpeg -ltiff" ; \
	export SDKROOT="$(call swap, $*, $(arch_names_all), $(sdks))" ; \
	export CFLAGS="-I$(TESSERACT_SRC)/$*/ $(common_cflags) -fembed-bitcode" ; \
	export CPPFLAGS=$$CFLAGS ; \
	export CXXFLAGS="-I$(TESSERACT_SRC)/$*/ $(common_cflags) -Wno-deprecated-register"; \
	export LDFLAGS="-L$$SDKROOT/usr/lib/ -L$(LEPTON_SRC)/$*/src/.libs" ; \
	export LEPTONICA_CFLAGS="-I$(TESSERACT_SRC)/$*/ $(common_cflags) -I$(LEPT_INC_DIR)/leptonica" ; \
	export LEPTONICA_LIBS="-llept" ; \
	export libcurl_CFLAGS="-I$(INCLUDE_DIR)"; \
	export libcurl_LIBS"-L$(LIB_FAT_DIR) -lcurl"; \
	mkdir -p $(@D) ; \
	cd $(@D) ; \
	ln -s $(LEPTON_SRC)/src/ leptonica ; \
	../configure --host=$* --prefix=`pwd` --enable-shared=no --disable-graphics

$(libtessconfig) : $(libtessautogen)
	cd $(@D) && ./autogen.sh 2> /dev/null
	cp tesseract_configure.ac tesseract-ocr/configure.ac

#######################
# LEPTONLIB
#######################
$(libleptfat) : $(liblept)
	mkdir -p $(LIB_FAT_DIR)
	xcrun lipo $(realpath $(addsuffix lib/$(@F), $(libleptfolders_all)) ) -create -output $@
	mkdir -p $(LEPT_INC_DIR)
	cp -rvf $(firstword $(libleptfolders))/include/leptonica $(LEPT_INC_DIR)

$(liblept) : $(libleptmakefile)
	cd $(abspath $(@D)/..) ; \
	$(MAKE) -sj8 && $(MAKE) install

$(LEPTON_SRC)/%/Makefile : $(imagesfat) $(libleptconfig)
	export LIBS="-lz -lpng -ljpeg -ltiff" ; \
	export SDKROOT="$(call swap, $*, $(arch_names_all), $(sdks))" ; \
	export CFLAGS="-I$(INCLUDE_DIR) $(common_cflags) -fembed-bitcode" ; \
	export CPPFLAGS=$$CFLAGS ; \
	export CXXFLAGS="-I$(INCLUDE_DIR) $(common_cflags) -Wno-deprecated-register"; \
    export LDFLAGS="-L$$SDKROOT/usr/lib/ -L$(LIB_FAT_DIR)" ; \
	export JPEG_CFLAGS="-I$(INCLUDE_DIR)"; \
	export JPEG_LIBS="-L$(LIB_FAT_DIR) -ljpeg"; \
	export LIBPNG_CFLAGS="-I$(INCLUDE_DIR)"; \
	export LIBPNG_LIBS="-L$(LIB_FAT_DIR) -lpng -lz"; \
	export LIBTIFF_CFLAGS="-I$(INCLUDE_DIR)"; \
	export LIBTIFF_LIBS="-L$(LIB_FAT_DIR) -ltiff"; \
	mkdir -p $(@D) ; \
	cd $(@D) ; \
    ../configure --host=$* --prefix=`pwd` --enable-shared=no --disable-programs --with-zlib --with-libpng --with-jpeg --without-giflib --with-libtiff --without-libwebp --without-libopenjpeg

#######################
# CURLLIB
#######################
$(libcurlfat) : $(libcurl)
	mkdir -p $(LIB_FAT_DIR)
	xcrun lipo $(realpath $(addsuffix lib/$(@F), $(libcurlfolders_all)) ) -create -output $@
	mkdir -p $(CURL_INC_DIR)
	cp -rvf $(firstword $(libcurlfolders))/include/curl $(CURL_INC_DIR)

$(libcurl) : $(libcurlmakefile)
	cd $(abspath $(@D)/..) ; \
	$(MAKE) -sj8 && $(MAKE) install

$(CURL_SRC)/%/Makefile : $(libcurlconfig)
	export LIBS="-lz" ; \
	export SDKROOT="$(call swap, $*, $(arch_names_all), $(sdks))" ; \
	export CFLAGS="-I$(INCLUDE_DIR) $(common_cflags) -fembed-bitcode" ; \
	export CPPFLAGS=$$CFLAGS ; \
	export CXXFLAGS="-I$(INCLUDE_DIR) $(common_cflags) -Wno-deprecated-register"; \
    export LDFLAGS="-L$$SDKROOT/usr/lib/ -L$(LIB_FAT_DIR)" ; \
	mkdir -p $(@D) ; \
	cd $(@D) ; \
	../buildconf ; \
    ../configure --host=$* --prefix=`pwd` --disable-debug --disable-dependency-tracking --disable-silent-rules --with-secure-transport --without-ca-bundle --without-ca-path

#######################
# Build libtiff and all of it's dependencies
#######################
$(imagesfat) : $(images)
	mkdir -p $(@D)
	cp -rvf $? $(@D)
	mkdir -p $(INCLUDE_DIR)
	cp -rvf $(IMAGE_INC_DIR) $(INCLUDE_DIR)

$(images) : $(imagesmakefile) FORCE
	cd $(IMAGE_SRC) ; \
	until `$(MAKE) -s`; do sleep 5; done

#######################
# Download dependencies
#######################
$(imagesmakefile) $(libtessautogen) :
	git submodule init
	git submodule update

$(libleptconfig) :
	curl http://leptonica.org/source/$(LEPTON_NAME).tar.gz | tar -xpf-
	cd $(LEPTON_NAME) ; \
	patch -p1 < ../leptonica.patch

$(libcurlconfig) :
	curl https://curl.haxx.se/download/$(CURL_NAME).tar.bz2 | tar -xpf-

#######################
# Clean
#######################
.PHONY : clean
clean : cleanimages cleanlept cleantess

.PHONY : distclean
distclean : distcleanimages distcleanlept distcleantess

.PHONY : mostlyclean
mostlyclean : mostlycleanimages mostlycleanlept mostlycleantess

.PHONY : cleanimages
cleanimages :
	cd $(IMAGE_SRC) ; \
	$(MAKE) clean

.PHONY : cleanlept
cleanlept :
	for folder in $(realpath $(libleptfolders_all) ); do \
        cd $$folder; \
        $(MAKE) clean; \
	done ;

.PHONY : cleantess
cleantess :
	for folder in $(realpath $(libtessfolders_all) ); do \
        cd $$folder; \
        $(MAKE) clean; \
    done ;

.PHONY : mostlycleanimages
mostlycleanimages :

.PHONY : mostlycleanlept
mostlycleanlept :
	for folder in $(realpath $(libleptfolders) ); do \
        cd $$folder; \
        $(MAKE) mostlyclean; \
    done ;

.PHONY : mostlycleantess
mostlycleantess :
	for folder in $(realpath $(libtessfolders_all) ); do \
        cd $$folder; \
        $(MAKE) mostlyclean; \
    done ;

.PHONY : distcleanimages
distcleanimages :
	-rm -rf $(IMAGE_SRC)

PHONY : distcleanlept
distcleanlept :
	-rm -rf $(LEPT_INC_DIR)/leptonica
	-rm -rf $(libleptfat)
	-rm -rf $(LEPTON_SRC)

.PHONY : distcleantess
distcleantess :
	-rm -rf $(TESS_INC_DIR)/tesseract
	-rm -rf $(libtessfat)
	-rm -rf $(TESSERACT_SRC)

.PHONY : FORCE
FORCE :
