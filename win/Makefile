WINEARCH          ?= win32
FFMPEG_VERSION    ?= 3.4
FFMPEG_CACHE_DIR  ?= ${HOME}/.cache/ci/r128gain/ffmpeg-${WINEARCH}

# versions
PYTHON_VERSION       := 3.6.8
PYTHON_VERSION_MAJOR := $(word 1,$(subst ., ,${PYTHON_VERSION})).$(word 2,$(subst ., ,${PYTHON_VERSION}))
PYTHON_VERSION_SHORT := $(subst .,,${PYTHON_VERSION_MAJOR})
CXFREEZE_VERSION     := 5.1.1
SOX_VERSION          := 14.4.2

# installers
DOWNLOAD_DIR            ?= dl
PYTHON_INSTALLER-win32  := ${DOWNLOAD_DIR}/python-${PYTHON_VERSION}.exe
PYTHON_INSTALLER-win64  := ${DOWNLOAD_DIR}/python-${PYTHON_VERSION}-amd64.exe
PYTHON_INSTALLER        := ${PYTHON_INSTALLER-${WINEARCH}}
CXFREEZE_WHEEL-win32    := ${DOWNLOAD_DIR}/cx_Freeze-${CXFREEZE_VERSION}-cp${PYTHON_VERSION_SHORT}-cp${PYTHON_VERSION_SHORT}m-${WINEARCH}.whl
CXFREEZE_WHEEL-win64    := ${DOWNLOAD_DIR}/cx_Freeze-${CXFREEZE_VERSION}-cp${PYTHON_VERSION_SHORT}-cp${PYTHON_VERSION_SHORT}m-win_amd64.whl
CXFREEZE_WHEEL          := ${CXFREEZE_WHEEL-${WINEARCH}}
7ZIP_INSTALLER          := ${DOWNLOAD_DIR}/7z938.msi
SOX_ZIP                 := ${DOWNLOAD_DIR}/sox-${SOX_VERSION}-win32.zip

# env
CURL     := curl -s -f -L -m 600 --connect-timeout 30 --retry 10 --retry-max-time 5
WINE_DIR := /tmp/$(subst /,_,${CURDIR})/wine-${WINEARCH}
WINE     := mkdir -p $(dir ${WINE_DIR}) && WINEPREFIX=${WINE_DIR} WINEARCH=${WINEARCH} WINEDLLOVERRIDES="mscoree,mshtml=" WINEDEBUG=-all wine
UNZIP    := unzip -o -q -DD

# installed files
PYTHON_INSTALLED     := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/python.exe
CXFREEZE_INSTALLED   := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/Scripts/cxfreeze
PIP_INSTALLED        := ${WINE_DIR}/drive_c/Python${PYTHON_VERSION_SHORT}/Scripts/pip.exe
7ZIP_INSTALLED       := ${WINE_DIR}/drive_c/7-Zip/7z.sfx
SOX_INSTALLED        := ${WINE_DIR}/drive_c/windows/system32/sox.exe
FFMPEG_INSTALLED     := ${WINE_DIR}/drive_c/windows/system32/ffmpeg.exe
ifeq ($(shell lsb_release -si),Ubuntu)
WINPTHREAD_DLL_win32 := /usr/i686-w64-mingw32/lib/libwinpthread-1.dll
WINPTHREAD_DLL_win64 := /usr/x86_64-w64-mingw32/lib/libwinpthread-1.dll
else
WINPTHREAD_DLL_win32 := /usr/i686-w64-mingw32/bin/libwinpthread-1.dll
WINPTHREAD_DLL_win64 := /usr/x86_64-w64-mingw32/bin/libwinpthread-1.dll
endif
WINPTHREAD_DLL       := ${WINPTHREAD_DLL_${WINEARCH}}

# temp dirs
OUTPUT_DIR   := r128gain-${WINEARCH}


.PHONY: all test test_ut test_freeze fclean clean


all:: r128gain_${WINEARCH}.7z r128gain_${WINEARCH}.zip r128gain_${WINEARCH}.exe

test:: test_ut test_freeze

print-%:
	@echo $*=$($*)


#
# End targets
#

r128gain_${WINEARCH}.7z: ${OUTPUT_DIR}/r128gain.exe ${OUTPUT_DIR}/LICENSE ${OUTPUT_DIR}/readme.txt ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/ffmpeg.exe ${OUTPUT_DIR}/libwinpthread-1.dll
	7z a -t7z -mx9 $@ ${OUTPUT_DIR} > /dev/null

r128gain_${WINEARCH}.zip: ${OUTPUT_DIR}/r128gain.exe ${OUTPUT_DIR}/LICENSE ${OUTPUT_DIR}/readme.txt ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/ffmpeg.exe
	7z a -tzip -mx9 $@ ${OUTPUT_DIR} > /dev/null

r128gain_${WINEARCH}.exe: ${7ZIP_INSTALLED} r128gain_${WINEARCH}.7z
	cat $^ > $@


#
# Intermediate targets
#

${OUTPUT_DIR}/r128gain.exe: ${CXFREEZE_INSTALLED} ${LXML_INSTALLED} .requirements
	mkdir -p $(dir $@)
	cd .. && ${WINE} ${PYTHON_INSTALLED} freeze.py build_exe -b win/${OUTPUT_DIR} 2> /dev/null > /dev/null
	touch $@

${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll: ${PYTHON_INSTALLED}
	mkdir -p $(dir $@)
	cp $(dir ${PYTHON_INSTALLED})$(notdir $@) $@

${OUTPUT_DIR}/LICENSE:
	mkdir -p $(dir $@)
	cp ../LICENSE $@

${OUTPUT_DIR}/readme.txt:
	mkdir -p $(dir $@)
	echo 'Get source, help and latest version at: https://github.com/desbma/r128gain' > $@

${OUTPUT_DIR}/ffmpeg.exe: ${FFMPEG_INSTALLED}
	mkdir -p $(dir $@)
	cp $< $@

${OUTPUT_DIR}/libwinpthread-1.dll: ${WINPTHREAD_DLL}
	mkdir -p $(dir $@)
	cp $< $@


#
# Tests
#

test_ut: ${LXML_INSTALLED} .requirements ${FFMPEG_INSTALLED} ${SOX_INSTALLED}
	cd .. && ${WINE} ${PYTHON_INSTALLED} setup.py test

test_freeze: ${OUTPUT_DIR}/r128gain.exe ${OUTPUT_DIR}/python${PYTHON_VERSION_SHORT}.dll ${OUTPUT_DIR}/ffmpeg.exe ${OUTPUT_DIR}/libwinpthread-1.dll sample.ogg
	${WINE} $< -v debug sample.ogg

sample.ogg: ${DOWNLOAD_DIR}/sample.ogg
	cp $< $@

#
# Installations
#

.requirements: ${PIP_INSTALLED}
	cd .. && ${WINE} ${PYTHON_INSTALLED} -m pip -qqq install -r requirements.txt
	touch $@

${PYTHON_INSTALLED}: ${PYTHON_INSTALLER}
	${WINE} $< /quiet InstallAllUsers=1 TargetDir=$(dir ${PYTHON_INSTALLED})
	touch $@

${CXFREEZE_INSTALLED}: ${CXFREEZE_WHEEL} ${PIP_INSTALLED} ${PYTHON_INSTALLED}
	${WINE} ${PYTHON_INSTALLED} -m pip -qqq install $<
	touch $@

${PIP_INSTALLED}: ${PYTHON_INSTALLED}
	${WINE} ${PYTHON_INSTALLED} -m pip -qqq install -U pip
	touch $@

${7ZIP_INSTALLED}: ${7ZIP_INSTALLER}
	${WINE} msiexec /i $< /qb INSTALLDIR='c:\7-Zip'
	touch $@

${FFMPEG_INSTALLED}:
	mkdir -p $(dir $@)
	./build-minimal-ffpmeg ${FFMPEG_VERSION} ${WINEARCH} ${FFMPEG_CACHE_DIR} $@

${SOX_INSTALLED}: ${SOX_ZIP}
	mkdir -p ${WINE_DIR}/drive_c
	${UNZIP} $< -d ${WINE_DIR}/drive_c
	cp ${WINE_DIR}/drive_c/sox-${SOX_VERSION}/*.dll $(dir $@)
	cp ${WINE_DIR}/drive_c/sox-${SOX_VERSION}/sox.exe $@


#
# Downloads
#

${PYTHON_INSTALLER}:
	mkdir -p $(dir $@)
	${CURL} https://www.python.org/ftp/python/${PYTHON_VERSION}/$(notdir $@) > $@

${CXFREEZE_WHEEL-win32}:
	mkdir -p $(dir $@)
	${CURL} https://pypi.python.org/packages/3b/04/d68567b8f3265d3936e37c1ee8b3378dc189ec66b4d43650affbefcc69c5/$(notdir $@) > $@

${CXFREEZE_WHEEL-win64}:
	mkdir -p $(dir $@)
	${CURL} https://pypi.python.org/packages/49/2b/07fc4cfff671ea348b310400e4f13a596e02079afc6bd1c33150b2cf30de/$(notdir $@) > $@

${7ZIP_INSTALLER}:
	mkdir -p $(dir $@)
	${CURL} http://www.7-zip.org/a/$(notdir $@) > $@.unckecked
	echo "673377a03f91ff0f7f384794934b767c3389e50d7dc308f7c73f4633121518b3e4af8d33cf282398a710577e12fbb963152591c3e29b1092c79530bb2d8aa882  $@.unckecked" | sha512sum --status -c - && mv $@.unckecked $@

${SOX_ZIP}:
	mkdir -p $(dir $@)
	${CURL} https://sourceforge.net/projects/sox/files/sox/${SOX_VERSION}/$(notdir $@) > $@

${DOWNLOAD_DIR}/sample.ogg:
	mkdir -p $(dir $@)
	${CURL} https://upload.wikimedia.org/wikipedia/en/0/09/Opeth_-_Deliverance.ogg > $@


#
# Clean
#

fclean:
	-rm -Rf ${WINE_DIR}
	-rm -Rf ${OUTPUT_DIR} .requirements
	-rm -f r128gain_${WINEARCH}.7z r128gain_${WINEARCH}.zip r128gain_${WINEARCH}.exe sample.ogg

clean: fclean
	-rm -Rf ${DOWNLOAD_DIR}
