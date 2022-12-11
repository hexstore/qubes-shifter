# Qubes Shifter

这是一个在Qubes OS中安装公共代理的方案，旨在帮助Qubes OS用户增强匿名环境下的网络可用性，它可被用做多重代理。

注意：**这是一个实验性项目，有关公共代理的潜在风险需要自行衡量！**

## 工作原理

它运行在一个网络服务盒子，它有两个定时器，**shifter-fetch**和**shifter-watch**分别作用是抓取可用的公共代理和守护代理服务，**shifter-watch**兼具管理**sing-box**的作用，即自动切换可用代理主机。

## 使用场景

它可以工作在这些场景下，也许您有自己的方案！

- sys-net <- sys-firewall <- **sys-shifter** <- AppVM(s)
- sys-net <- **sys-shifter** <- sys-firewall <- AppVM(s)

## 前提条件

- Qubes OS

## 安装

这里创建一个代理盒子，它被命名为**sys-shifter**，然后从GitHub下载[sing-box](https://github.com/SagerNet/sing-box/releases)的二进制文件和代理抓取工具[proxy-scraper-checker](https://github.com/monosans/proxy-scraper-checker)。

```bash
[user@dom0 ~]$ qvm-create sys-shifter --class AppVM --label blue
[user@dom0 ~]$ qvm-prefs sys-shifter provides_network true
[user@dom0 ~]$ qvm-prefs sys-shifter autostart true
[user@dom0 ~]$ qvm-prefs sys-shifter memory 500
[user@dom0 ~]$ qvm-prefs sys-shifter maxmem 500
```

接下来将安装sing-box到`/rw/usrlocal/bin`目录，配置文件`sing-box.json`被安装到`/rw/bind-dirs/etc/sing-box`目录，
守护运行配置文件`sing-box.service`和`shifter-*`被安装到`/rw/bind-dirs/etc/systemd/system`目录，**proxy-scraper-checker**将被安装到`/home/shifter/.local/src`目录。

```bash
[user@dom0 ~]$ qvm-start sys-shifter
[user@dom0 ~]$ qrexec-client -W -d sys-shifter user:'curl --proto "=https" -tlsv1.2 -SfL https://git.sr.ht/~qubes/shifter/blob/main/shifter | sh -s -- --install'
```

来到这一步，将要重启**sys-shifter**盒子。

```bash
[user@dom0 ~]$ qvm-shutdown --wait sys-shifter
[user@dom0 ~]$ qvm-start sys-shifter
```

确认**sys-shifter**盒子确认代理服务的运行状态。

```bash
[user@dom0 ~]$ qrexec-client -W -d sys-shifter root:'journalctl -f'
```

## 贡献

您在使用这个项目的过程中发现任何问题或疑问，可以随时[创建ticket](https://todo.sr.ht/~qubes/shifter)，我们将会尽快解答。另外，您有任何改进方案，欢迎提交一个[patch](https://git.sr.ht/~qubes/shifter/send-email)。

## 其它

请注意：为避免无法预测的异常，_apt和shifter用户的流量不经过代理，您在使用时请留意相关的安全问题。

守护任务`shifter-fetch`和`shifter-watch`的运行时间分别是15min和30s，您可以根据自己的需要更改`/rw/bind-dirs/etc/systemd/system/shifter-*.timer`的时间设置。

首次重启需要等待shifter-fetch服务执行完成，如果需要加快它的处理速度，您可以在安装完成时更改`/home/shifter/.local/src/proxy-scraper-checker/config.ini`，禁用不必要的项目。

## 相关

- [Qubes OS](https://www.qubes-os.org/)
- [sing-box](https://sing-box.sagernet.org/)
- [sing-box GitHub](https://github.com/SagerNet/sing-box)
- [proxy-scraper-checker](https://github.com/monosans/proxy-scraper-checker)
