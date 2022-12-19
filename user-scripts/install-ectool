#!/usr/bin/env python3
import os
import sys
import argparse

from functions import *


# parse arguments from the cli.
def process_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("-v", "--verbose", action="store_true", dest="verbose", default=False, help="Print more output")
    return parser.parse_args()


# This script will update the postinstall scripts on an existing installation.
if __name__ == "__main__":
    if os.geteuid() == 0 and not path_exists("/tmp/.root_ok"):
        print_error("Please start the script as non-root/without sudo")
        exit(1)

    args = process_args()

    # Restart script as root
    if not os.geteuid() == 0:
        # create empty file to confirm script was started as non-root
        with open("/tmp/.root_ok", "w") as file:
            file.write("")
        sudo_args = ['sudo', sys.executable] + sys.argv + [os.environ]
        os.execlpe('sudo', *sudo_args)

    # delete file to confirm script was started as root
    rmfile("/tmp/.root_ok")

    print_error("Not ready for use yet")
    exit(1)

    # delete old scripts tmp dir
    rmdir("/tmp/eupnea-ectool")

    # install dependencies
    print_status("Installing dependencies")

    if path_exists("/usr/bin/apt"):  # Ubuntu + debian
        bash("apt-get install -y libftdi-dev libusb-dev libncurses-dev pkgconf")
    elif path_exists("/usr/bin/pacman"):  # Arch
        bash("pacman -S --noconfirm libftdi libusb ncurses pkgconf")
    elif path_exists("/usr/bin/dnf"):  # Fedora
        bash("dnf install -y libftdi libusb ncurses pkgconf")  # cgpt is included in vboot-utils on fedora
    elif path_exists("/usr/bin/zypper"):  # openSUSE
        bash("sudo zypper --non-interactive install libftdi libusb ncurses pkgconf")

    # download ectool source from chromium git repo
    print_status("Downloading ectool source from chromium git repo")
    start_progress()  # start fake download progress
    bash("git clone --depth=1 https://chromium.googlesource.com/chromiumos/platform/ec /tmp/eupnea-ectool/postinstall")
    stop_progress()  # stop fake download progress

    # build ectool
    # Doesn't seem to work
    # make BOARD=$BOARD CROSS_COMPILE= HOST_CROSS_COMPILE= build/$BOARD/util/ectool

    cpfile("/tmp/ec/build/$BOARD/util/ectool", "/usr/bin/ectool")
    # chmod?
