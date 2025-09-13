#!/bin/bash

# 检查脚本是否以root权限运行
if [ "$EUID" -ne 0 ]; then
    echo "请使用sudo或以root权限运行此脚本"
    exit 1
fi

echo "开始强制停止并卸载占用53端口的软件..."

# 强制停止所有占用53端口的进程
echo "停止所有占用53端口的进程..."
pkill -9 -f ":53" 2>/dev/null

# 使用多种方法确保端口被释放
if command -v fuser &> /dev/null; then
    fuser -k -9 53/tcp 2>/dev/null
    fuser -k -9 53/udp 2>/dev/null
fi

if command -v ss &> /dev/null; then
    ss -tulpn | grep ":53" | awk '{print $7}' | cut -d'"' -f2 | cut -d',' -f1 | \
    while read pid; do
        kill -9 "$pid" 2>/dev/null
    done
fi

# 获取占用53端口的进程对应的软件包
PACKAGES_TO_REMOVE=""

# 尝试通过多种方式获取包名
for method in 1 2 3; do
    case $method in
        1)
            # 方法1: 通过lsof获取进程并查找包名
            if command -v lsof &> /dev/null; then
                lsof -i :53 -sTCP:LISTEN | awk 'NR>1 {print $2}' | while read pid; do
                    if [ -n "$pid" ]; then
                        # Debian/Ubuntu
                        if command -v dpkg &> /dev/null; then
                            dpkg -S $(readlink -f /proc/$pid/exe 2>/dev/null) 2>/dev/null | cut -d: -f1
                        # RedHat/CentOS
                        elif command -v rpm &> /dev/null; then
                            rpm -qf $(readlink -f /proc/$pid/exe 2>/dev/null) 2>/dev/null
                        fi
                    fi
                done | sort -u >> /tmp/port53_packages.txt
            fi
            ;;
        2)
            # 方法2: 检查常见的使用53端口的服务
            common_services="dnsmasq bind9 named unbound systemd-resolved"
            for service in $common_services; do
                if systemctl is-active --quiet $service 2>/dev/null || \
                   ps aux | grep -v grep | grep -q $service; then
                    echo $service >> /tmp/port53_packages.txt
                fi
            done
            ;;
        3)
            # 方法3: 检查监听53端口的服务
            netstat -tulpn 2>/dev/null | grep ":53" | awk '{print $7}' | \
            while read proc; do
                if [[ "$proc" == *"/"* ]]; then
                    proc_name=$(echo $proc | cut -d'/' -f2)
                    echo $proc_name >> /tmp/port53_packages.txt
                fi
            done
            ;;
    esac
done

# 收集所有可能的包名
if [ -f /tmp/port53_packages.txt ]; then
    PACKAGES_TO_REMOVE=$(cat /tmp/port53_packages.txt | sort -u | tr '\n' ' ')
    rm -f /tmp/port53_packages.txt
fi

# 添加一些常见的使用53端口的软件
COMMON_DNS_SOFTWARE="dnsmasq bind9 unbound resolvconf systemd-resolved"

# 卸载软件包
echo "开始卸载软件包..."

if [ ! -z "$PACKAGES_TO_REMOVE" ]; then
    echo "检测到以下软件包需要卸载: $PACKAGES_TO_REMOVE"
fi

# 根据包管理器类型执行卸载
if command -v apt-get > /dev/null; then
    # Debian/Ubuntu系统
    apt-get remove --purge -y $PACKAGES_TO_REMOVE $COMMON_DNS_SOFTWARE 2>/dev/null
    apt-get autoremove -y 2>/dev/null
elif command -v yum > /dev/null; then
    # RedHat/CentOS系统
    yum remove -y $PACKAGES_TO_REMOVE bind unbound 2>/dev/null
elif command -v dnf > /dev/null; then
    # Fedora系统
    dnf remove -y $PACKAGES_TO_REMOVE bind unbound 2>/dev/null
else
    echo "不支持的包管理器，尝试直接杀死进程"
fi

# 确保服务被禁用
for service in $COMMON_DNS_SOFTWARE; do
    if systemctl is-active --quiet $service 2>/dev/null; then
        systemctl stop $service 2>/dev/null
        systemctl disable $service 2>/dev/null
    fi
done

# 最后再次确保没有进程占用53端口
echo "最终检查并清理..."
pkill -9 -f ":53" 2>/dev/null

if command -v fuser &> /dev/null; then
    fuser -k -9 53/tcp 2>/dev/null
    fuser -k -9 53/udp 2>/dev/null
fi

# 验证端口是否已释放
if lsof -i :53 >/dev/null 2>&1 || ss -tulpn | grep -q ":53"; then
    echo "警告: 仍有进程监听53端口，尝试识别并手动处理..."
    # 显示仍在监听53端口的进程
    echo "当前监听53端口的进程:"
    lsof -i :53 2>/dev/null || ss -tulpn | grep ":53" 2>/dev/null
else
    echo "成功: 53端口已完全释放"
fi

echo "强制卸载完成!"
echo "注意: 系统DNS功能可能已受影响，请确保有替代的DNS配置"
