#!/bin/bash
clear
#--------------------------
# SCRIPT SSHPLUS PRO
# DEV: @Adenilson_VPN
# CANAL TELEGRAM: @AD-NETWORK
#--------------------------

# - Cores
RED='\033[1;31m'
YELLOW='\033[1;33m'
SCOLOR='\033[0m'

# - Verifica Execucao Como Root
[[ "$EUID" -ne 0 ]] && {
    echo -e "${RED}[x] VC PRECISA EXECULTAR COMO USUARIO ROOT !${SCOLOR}"
    exit 1
}

# - Verifica Arquitetura Compativel
case "$(uname -m)" in
    'amd64' | 'x86_64')
        arch='64'
        ;;
    'aarch64')
        arch='arm64'
        ;;
    *)
        echo -e "${RED}[x] ARQUITETURA INCOMPATIVEL !${SCOLOR}"
        exit 1
        ;;
esac

# - Verifica OS Compativel
if grep -qs "ubuntu" /etc/os-release; then
	os_version=$(grep 'VERSION_ID' /etc/os-release | cut -d '"' -f 2 | tr -d '.')
    [[ "$os_version" -lt 1804 ]] && {
        echo -e "${RED}[x] VERSAO DO UBUNTU INCOMPATIVEL !\n${YELLOW}[!] REQUER UBUNTU 18.04 OU SUPERIOR !${SCOLOR}"
        exit 1
    }
elif [[ -e /etc/debian_version ]]; then
	os_version=$(grep -oE '[0-9]+' /etc/debian_version | head -1)
	[[ "$os_version" -lt 9 ]] && {
        echo -e "${RED}[x] VERSAO DO DEBIAN INCOMPATIVEL !\n${YELLOW}[!] REQUER DEBIAN 9 OU SUPERIOR !${SCOLOR}"
        exit 1
    }
    [[ "$os_version" == 9 ]] && {
        echo -e "${RED}[!] ATENCAO O ${SCOLOR}DEBIAN 9 STRETCH${RED} CHEGOU\nOFICIALMENTE AO SEU FIM DE VIDA UTIL ! ${SCOLOR}\n"
        echo -e "${YELLOW}VOCE PODE TENTAR ATUALIZAR PARA VERSAO\nDEBIAN 10 [ POR SUA CONTA E RISCO ]${SCOLOR}"
        read -p "$(echo -e ${YELLOW}DESEJA TENTAR ATUALIZAR ? [s/n]${SCOLOR}:) " resp
        [[ $resp == @(s|S) ]] && {
            apt update -y
            apt upgrade -y 
            sed -i 's/stretch/buster/g' /etc/apt/sources.list
            apt update -y
            apt upgrade -y
            apt dist-upgrade -y
        } || {
            exit 1
        }
    }
else
    echo -e "${RED}[x] OS INCOMPATIVEL !\n${YELLOW}[!] REQUER DISTROS BASE DEBIAN/UBUNTU !${SCOLOR}"
    exit 1
fi

# - Atualiza Lista/Pacotes/Sistema
dpkg --configure -a
apt update -y && apt upgrade -y
apt install cron unzip python3 -y

# - Ajusta o sysctl

sysctl -w net.ipv6.conf.all.disable_ipv6=1 && echo 'net.ipv6.conf.all.disable_ipv6 = 1' > /etc/sysctl.d/70-disable-ipv6.conf && sysctl -p -f /etc/sysctl.d/70-disable-ipv6.conf

echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf && sysctl -p

# - Execulta instalador
[[ -e install-sshplus ]] && rm install-sshplus
wget /scripts/${arch}/install-sshplus
chmod +x install-sshplus
[[ $(systemctl | grep -ic fuse) != '0' ]] && ./install-sshplus || ./install-sshplus --appimage-extract-and-run
rm install-sshplus > /dev/null 2>&1
