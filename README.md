#!/bin/bash

# Patcher script for legacy NVIDIA drivers 470.xxx

DRIVERVERSION='470.256.02'

# add here patch URLs or FULL PATH to local patch files
declare -a PATCHURLSORFILES=(
	'https://aur.archlinux.org/cgit/aur.git/plain/0001-Fix-conftest-to-ignore-implicit-function-declaration.patch?h=nvidia-470xx-utils&id=df0426ab325cb0ad8909a3058d66336ce1f872ce'
	'https://aur.archlinux.org/cgit/aur.git/plain/0002-Fix-conftest-to-use-a-short-wchar_t.patch?h=nvidia-470xx-utils&id=df0426ab325cb0ad8909a3058d66336ce1f872ce'
	'https://aur.archlinux.org/cgit/aur.git/plain/0003-Fix-conftest-to-use-nv_drm_gem_vmap-which-has-the-se.patch?h=nvidia-470xx-utils&id=df0426ab325cb0ad8909a3058d66336ce1f872ce'
	'https://aur.archlinux.org/cgit/aur.git/plain/kernel-6.10.patch?h=nvidia-470xx-utils&id=df0426ab325cb0ad8909a3058d66336ce1f872ce'
	# joanbm's Tentative fix for NVIDIA 470.256.02 driver for Linux 6.12-rc1
	'https://gist.githubusercontent.com/joanbm/a6d3f7f873a60dec0aa4a734c0f1d64e/raw/6bae5606c033b6c6c08233523091992370e357b7/nvidia-470xx-fix-linux-6.12.patch'
	# joanbm's Tentative fix for NVIDIA 470.256.02 driver for Linux 6.13-rc1
	'https://gist.githubusercontent.com/joanbm/d1f89391a4b20f4b56ba931ef6ca62da/raw/8458c7c58249a0dceb5ab1b5aada7e705a88b4ff/nvidia-470xx-fix-linux-6.13.patch'
)

#### nothing to change below

DRIVERNAME="NVIDIA-Linux-x86_64-${DRIVERVERSION}"
DRIVERFILE="${DRIVERNAME}.run"
DRIVERDIR="${DRIVERNAME}"

# fetch the NVIDIA driver from NVIDIA's site if not exist
if [ ! -f NVIDIA-Linux-x86_64-470.256.02.run ]; then
	read -p "Do you want to download the NVIDIA driver ${DRIVERVERSION} ? (Y/n) " answ
	if ! [[ "${answ}" =~ ^[yY].*$ ]]; then echo "can not continue"; exit 1; fi
	CMD="curl 'https://us.download.nvidia.com/XFree86/Linux-x86_64/${DRIVERVERSION}/${DRIVERFILE}' -o '${DRIVERFILE}'"
	eval ${CMD}; if [ $? -ne 0 ]; then echo "$0 : command has failed: ${CMD}"; exit 1; fi
	echo "$0 : done, downloaded NVIDIA driver."
fi

# erase the old NVIDIA driver dir if exists
if [ -d "${DRIVERDIR}" ]; then
	read -p "Do you want to erase old NVIDIA driver dir (${DRIVERDIR})? (Y/n) " answ
	if ! [[ "${answ}" =~ ^[yY].*$ ]]; then echo "can not continue"; exit 1; fi
	CMD="rm -rf '${DRIVERDIR}'"
	eval ${CMD}; if [ $? -ne 0 ]; then echo "$0 : command has failed: ${CMD}"; exit 1; fi
	echo "$0 : done, erased old NVIDIA driver dir (${DRIVERDIR})."
fi

# extract the NVIDIA driver archive (*.run) to a dir
CMD="sh '${DRIVERFILE}' --extract-only"
eval ${CMD}; if [ $? -ne 0 ]; then echo "$0 : command has failed: ${CMD}"; exit 1; fi
echo "$0 : done, extracted files from the NVIDIA driver archive (${DRIVERFILE}) into '${DRIVERDIR}'."

# go into the NVIDIA driver dir
CMD="cd '${DRIVERDIR}'"
eval ${CMD}; if [ $? -ne 0 ]; then echo "$0 : command has failed: ${CMD}"; exit 1; fi

# apply the patches from inside the NVIDIA driver dir
for ap in "${PATCHURLSORFILES[@]}"; do
	if [[ "${ap}" =~ ^http ]]; then
		CMD="curl '${ap}' | patch -Np1 -d kernel"
	else
		CMD="patch -Np1 -d kernel < '${ap}'"
	fi
	echo "$0 : patching with this command : ${CMD}"
	eval ${CMD}; if [ $? -ne 0 ]; then echo "$0 : command has failed: ${CMD}"; exit 1; fi
done
