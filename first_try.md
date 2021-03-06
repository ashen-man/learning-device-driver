Linuxデバイスドライバの作り方

# はじめに

- 平田 豊さんのLinuxデバイスドライバプログラミング（以下、デバドラ本）に沿って進めていきます。
- ソースコードも基本的に上記の本のサポートページからDLしたものを使います。
- 上記のコードをTeraTermでラズパイにSCP転送して、そこでビルドするというスタイルです。

# 開発環境

- 開発環境はRaspberry Pi Zero W上で起動している下記のOSです。

```sh
uname -a
Linux raspberrypi 5.10.17+ #1403 Mon Feb 22 11:26:13 GMT 2021 armv6l GNU/Linux
```

- 最初はimagerを使ってsdカードにインストールしたRaspberry Pi OSで進めるつもりでした。
- ただ、そのOSには、/lib/modules/$(shell uname -r)/build/Makefileがなく、カーネルモジュールをビルドできませんでした。
- apt full-upgradeでカーネルをアップグレードすると（正しいやり方かは分らない）、上のMakefileも取得でき、ビルドができるようになりました。

# 最初のデバイスドライバ

- 下はデバドラ本記載のソースコードです。
- カーネルが起動するときか、モジュールがカーネルにインサートされたときに呼び出されるのがmodule_init。
- カーネルが取り除かれたときに呼び出されるのがmodule_exit。

```c
#include <linux/module.h>
#include <linux/init.h>

MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void)
{
	printk(KERN_ALERT "driver loaded\n");
		
	return 0;
}

static void hello_exit(void)
{
	printk(KERN_ALERT "driver unloaded\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

# ビルド

- 下はビルドのやり方を書いておくMakefileです。


```
obj-m := hello.o

all:
		make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
		make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

- このレシピを使い、下のコマンドでビルドします。

```sh
make
```

- ソースファイルなども含まれていますが、下のような感じでファイルが生成されました。

```sh
-rw-r--r-- 1 pi pi  316 Feb 20 14:07 hello.c
-rw-r--r-- 1 pi pi 3392 Mar  6 13:27 hello.ko
-rw-r--r-- 1 pi pi   55 Mar  6 13:27 hello.mod
-rw-r--r-- 1 pi pi  774 Mar  6 13:27 hello.mod.c
-rw-r--r-- 1 pi pi 2304 Mar  6 13:27 hello.mod.o
-rw-r--r-- 1 pi pi 2300 Mar  6 13:27 hello.o
-rw-r--r-- 1 pi pi  158 Feb 27 23:32 Makefile
-rw-r--r-- 1 pi pi   55 Mar  6 13:27 modules.order
-rw-r--r-- 1 pi pi    0 Mar  6 13:27 Module.symvers
```

# ロード・アンロード

- カーネルモジュールをロードして、アンロードします。

```sh
sudo insmod hello.ko
sudo rmmod hello.ko
```

- 下のコマンドでカーネルが出力するメッセージを確認します。
- たしかに、hello.koがロード・アンロードされたことが分かります。

```
dmesg

<略>
[  437.900769] driver loaded
[  447.160249] driver unloaded
<略>
```

# おわりに
- 今日はこんなところです。
