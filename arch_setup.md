# archlinux セットアップメモ

## 環境
* PC: Katana-GF66-12UGS-043JP
  - https://kakaku.com/item/K0001417923/spec/#tab
* インストローラ: archlinux-2023.07.01-x86_64.iso
  - Macでフラッシュメモリ(usb)に焼いて使った



## 手順

---

### インストローラの準備
archwikiのインストールページにに載ってる`https://www.archlinux.jp/download/`から.iosファイルをダウンロードする．

Macに焼く対象のフラッシュメモリを差す前後で`$ diskutil list`を実行してフラッシュメモリの位置を特定する．
(大体`/dev/disk[0-9]`にある気がする)

フラッシュメモリを`MS-DOS`形式でフォーマットする．
`UNTITLED`はフラッシュメモリの名前になるのでなんでもいいです．
```
$ diskutil eraseDisk MS-DOS UNTITLED /dev/disk.
```

#### ISOイメージの書き込み
対象のフラッシュメモリを一旦アンマウントして作業をする．
```
$ diskutil unmountDisk /dev/disk.
```
アンマウントしたらddでフラッシュメモリに焼く
```
$ sudo dd if=./archlinux-2023.07.01-x86_64.iso of=/dev/disk2 bs=1m
```
オプションについては
> ##### ofオプションについて
> 
> /dev/rdisk. と指定しています。
> 両者の違いは、
> disk … 通常のランダムアクセス
> rdisk … シーケンシャル(順次)アクセス
> ということです。ddでランダムアクセスをしてしまうと激遅になります…
> ##### bsオプションについて
> 
> 転送バイト数を指定しますが、デフォルトは512バイトです。
> 効率が悪いはずなので、ある程度上げます。
> USB2.0 の実効上の最大転送速度が 40MB/s くらいなので、その半分弱程度…ということで適当に 16m としています。
> どうしても遅ければ環境に合わせた検証が必要なはずですが……
> 引用元: https://qiita.com/kapibarasensei/items/9fb81f0102f9ccc31559

最後にフラッシュメモリを取り出して終了．
```
$ diskutil eject /dev/disk.
```

---

### archlinuxのインストール
PCに焼いたフラッシュメモリを差して暗転明けからF12を押してUEFIを使っていく．
UEFIの場合`Arch Linux install medium (x86_64, UEFI)`みたいなやつを選ぶ

作業記録を残すために`script`コマンドでシェルに入力した内容を保存しておくと便利．
```
# script
Script started, file is typescript
```
終了するときは`exit`コマンドを実行する．
すると`typescript`という名前で履歴ファイルが保存される．

#### キーボードを設定する(ここではJISキーボード)
```
# loadkeys jp106
```

#### システムクロックの設定をする
```
# timedatectl set-ntp true
```
正しい時刻かどうかは`# timedatectl status`で確認する．

#### インターネットが接続があるかの確認をする
```
# ping archlinux.jp
```

> #### インターネット(wi-fiで接続する場合)
> pingコマンドで、インターネット接続を確認します。 デスクトップパソコン等で、イーサネットケーブルで有線接続している場合はインターネットに繋がっているはずです。
> ```
> ping archlinux.org
> ```
> しかし、ノートパソコン等で、有線ではなくwifi接続の場合は、 名前解決が出来ない等のエラーが表示されて、ネットワークに接続出来ていないはずです。 そこで、次からの手順でwifiを使ってネットワークに接続しましょう。
> 
> wifiの接続用SSIDと接続用パスワードを準備しておく。 wifiルーター側で行うMACアドレス制限等をクリアしておく。 パソコン側のMACアドレスが必要ならば「ip a」で確認できる。
> 
> 「iwctl」コマンドを使います。（iwを必要に応じて参照）
> ```
> iwctl
> ```
> 
> コンソールが起動するので、まず、デバイス名を確認。入力は補完が効くので適宜タブ。
> ```
> [iwd]# device list
> ```
> 
> 上記で確認したwifiデバイス名がwlan0として、 次に接続ポイントを確認します。
> ```
> [iwd]# station wlan0 scan
> [iwd]# station wlan0 get-networks
> ```
> 
> 接続ポイントが確認できたら、次の入力( SSIDの部分は、自分のところのSSIDに入れ替える事を忘れずに )を行い、 出てきたパスワードプロンプトにパスワードを渡します。
> ```
> [iwd]# station wlan0 connect SSID
> ```
> 
> 数秒待って特にエラーが出なければ、 exitコマンドでiwdコンソールを終了します。
> ```
> [iwd]# exit
> ```
> http://neko-mac.blogspot.com/2021/05/arch-linux.html

#### インストールするディスクのセッティング
まずPCに繋がっているディスクを`# fdisk -l`で確認する．

パーティションを設定する必要があるが昔に作ったパーティションで問題ないため省略．
> 昔にやったパーティションはgdiskを使った気がする
> 参考: https://qiita.com/j8takagi/items/235e4ae484e8c587ca92#パーティションの作成
> 　　: https://aznote.jakou.com/archlinux/install2.html

今回作るレイアウト
| マウント先| パーティションタイプ | 容量 |
| :--- | :--- | :---- |
| /mnt/boot | EFIシステムパーティション | 512MB |
| [swap]    | Linux swap            | 1GB |
| /mnt      | Linux x86-64 root     | 残り全部 |

パーティションができたら各パーティションごとにフォーマットしていく.
```
# mkfs.fat -F32 <EFIシステムパーティション>
# mkfs.ext4 <Linux x86-64 root>
# mkswap <Linux swap>
```
フォーマットができたら/mntにマウントしていく
```
# mount <Linux x86-64 root> /mnt
# mkdir /mnt/boot
# mount <EFIシステムパーティション> /mnt/boot
# swapon <Linux swap>
```

#### インストール
必須のパッケージをインストールする
```
# pacstrap /mnt base linux linux-firmware
```
あと，必須ではないが便利そうなものも少しインストールしておく
```
# pacstrap /mnt vim zsh
```

#### システムの設定
fstabを作成する(UUIDを使うため-Uオプションを添加)
```
# genfstab -U /mnt >> /mnt/etc/fstab
```

次に新しくインストールしたシステムにchrootする．
```
# arch-chroot /mnt
```

タイムゾーンの設定を行う．
```
# ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
# hwclock --systohc
```

次にローカリゼーションを行う．
`/etc/locale.gen`を編集して`en_US.UTF-8 UTF-8`と`ja_JP.UTF-8 UTF-8`の行をアンコメントして保存する．
編集後に以下のコマンドを実行してlocaleを作成する．
```
# locale-gen
```
`locale-gen`の実行が終わってlocaleができたら言語を設定する．
```
# echo "LANG=en_US.UTF-8" > /etc/locale.conf
```


キーマップの設定を永続化するため`/etc/vconsole.conf`を以下の様にする．
```
# vim /etc/cconsole.conf
#KEYMAP=us
KEYMAP=jp106
#FONT=
```

ホスト名を設定するため以下のコマンドを実行する．(設定するホスト名は任意でつける)
```
echo <hostname> > /etc/hostname
```
ついでに`/etc/hosts`も編集する
```
# vim /etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	<hostname>.localdomain	<hostname>
```

最後にrootのパスワードを設定する．
```
# passwd
```

> 参考: https://aznote.jakou.com/archlinux/install4.html

#### ブートローダの用意
今回はGRUBを使った．(systemd-bootでも良かったかも?)
> インストールしたLinuxなどのOSをブートするためには、ブートローダーの設定が必要です。Linuxでのブートローダーの設定にはGRUBが用いられることが一般的でした。
> しかし最近は、ブートモードがUEFIの場合にはsystemdに含まれるsystemd-bootも多く用いられています。
> ここでは、systemd-bootを用いてブートローダーを設定します。
> 引用: https://qiita.com/j8takagi/items/235e4ae484e8c587ca92#ブートローダーの設定

GRUBに必要なパッケージをインストールする．
```
# pacman -S grub efibootmgr dosfstools os-prober mtools
```
GURBを以下のコマンドでインストールする．
```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB_UEFI
```
GRUBをインストールできたらGURBの設定ファイルを作成する．
**今後GRUBの設定を変更する度に忘れずに以下のコマンドを実行して`gurb.cfg`ファイルを作成する**
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

プロセッサ用のマイクロコードを用意する．(今回はintelのプロセッサだからintel-ucodeを使う)
(amdの場合はamd-ucodeをインストールする)
```
# pacman -S intel-ucode
# grub-mkconfig -o /boot/grub/grub.cfg
```

> 参考: http://neko-mac.blogspot.com/2021/05/arch-linux.html

#### ネットワークの準備
**この項目はおそらくwi-fiでインターネット接続しているときに必須であると思われる．**
1行目で必要なものをインストールする．
2行目ではトラブル時に便利らしい?ものとインストールする．
最後にsystemctlで，NetworkManageが自動で立ち上がる登録する．
```
# pacman -S networkmanager wpa_supplicant dialog
# pacman -S iw iwd dhcpcd netctl
# systemctl enable NetworkManger
```
詳しい設定は再起動後に行う．

#### 再起動と作業記録の移動
arch-chrootを終了させる
```
# exit
```
さらにscriptコマンドの記録をここで終了させたのち，
作業記録をマウント後のディレクトリにコピーする．
```
# exit
# cp typescript /mnt/root/arch_linux_install.log
```

最後にPCを再起動させる
```
# reboot
```


---

### インストール後の作業
ログインプロンプトで`root`でログインする．

#### ネットワークの設定(wi-fi)
pingコマンドでインターネット接続を確認する．
```
# ping archlinux.org
```
接続されていない場合は以下のコマンドでNetworkManager設定メニューを表示する(gui)．
```
# nmtui
```
するとGUIの設定メニューが表示されるため`Activate a connection`を選択して，
wifi等の設定を行う．

> nmtuiで一度、設定しておけば、次回システム起動時は適当に繋がります。 ノート等で複数wifiポイントを利用する場合でも、各ポイントで初回に設定しておけば、 各ポイント毎に自動で繋がってくれます。 接続設定は一般的に「プロファイル」と呼ばれ、プロファイルもnmtuiの「edit a connection」から編集出来ます。
> 
> nmtuiの使い方は、まず初めに、「Activate a connection」。
> 
> もしその設定を変更する必要があれば、「edit a connection」を使います（普段は使わないということ）。
> http://neko-mac.blogspot.com/2021/05/arch-linux.html

#### システムの設定
システムのアップデートをする．
```
# pacman -Syu
```
pacmanの表示を見やすくするために以下の項目をアンコメントする．
(ILoveCandyは追記しないといけない)
- Color : 出力をカラー化
- ILoveCandy : ダウンロードがパックマンになる(お遊びなので要らなければコメントアウトするかそもそも書かなくて良い)
- ParallelDownloads = 5 : リポジトリからの複数のパッケージを同時に ダウンロードしてくれるようになる(らしいけど内容は不明)
```
# vim /etc/pacman.conf

..略..
# Misc options
..略..
Color
ILoveCandy
..略..
ParallelDownloads = 5
..略..
```

#### アカウントの準備とsudo
一般アカウントをwheelに所属させる．
そして新しく作ったアカウントのパスワードを設定する．
```
# useradd -m -G wheel <new account>
# passwd <new account>
```
ついでにsudoコマンドも設定する．
sudoをインストールして，適当なエディタでvisudoコマンドを実行する．
以下のようなwheelについての記述をアンコメントして保存する．
```
# pacman -S sudo
# EDITOR=vim visudo

..略..
%wheel ALL=(ALL:ALL) ALL
..略..
```

忘れてたzshのアドオン?みたいなやつをいれておく．
```
# pacman -S zsh-completions grml-zsh-config
```

#### 一般アカウントの設定
whichコマンドを追加して，デフォルトshellをzshに設定する．
```
$ sudo pacman -S which
$ chsh -s $(which zsh)
```
ついでにバッテーリー表示ができるようにする．
```
$ vim ~/.zshrc.pre
GRML_DISPLAY_BATTERY=1
```

時計の設定を行う．
```
$ sudo timedatectl set-ntp true
```


#### その他
- AURヘルパー(Pacmanラッパー) : yay, etc...(https://wiki.archlinux.jp/index.php/AUR_ヘルパー)
- CLIファイルマネージャー : ranger

#### AURヘルパーのインストール(yay)
yayのgitリポジトリからインストールする
```
$ pacman -S --needed git base-devel
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```


> https://github.com/Jguer/yay
> http://neko-mac.blogspot.com/2021/05/arch-linux.html

#### 参考URL
* https://wiki.archlinux.jp/index.php/インストールガイド
* https://qiita.com/j8takagi/items/235e4ae484e8c587ca92
* http://neko-mac.blogspot.com/2021/05/arch-linux.html
* https://zenn.dev/ytjvdcm/articles/0efb9112468de3


---

### GUIインストール(i3)

#### Xウィンドウシステム
Xウィンドウシステム
- xorg-server : 
- xorg-xinit : ウィンドウマネージャなしでGUIを起動するのに必要(startx, /etc/X11/xinit/xinitrc, ~/.xinitrc)
- alacritty : ターミナルエミュレータ
- xterm : ターミナルエミュレータ
- rxvt-unicode : ターミナルエミュレータ

ディスプレイマネージャ
- lightdm : GUIのディスプレイマネージャ
- lightdm-gtk-greeter : lightDMでGUIのログイン情報をユーザに求めるために必要

フォント
- ttf-ricty : プログラミング用フォント(日本語対応?)
- noto-fonts : ラテン文字系フォント

その他
- dmenu : i3のアプリケーションランチャー(コマンドを実行できるやつ)
- rofi : ファイルランチャー(dmenuの代替品)

```
$ sudo pacman -S xorg-server xorg-xinit dmenu
$ sudo pacman -S lightdm lightdm-gtk-greeter
$ yay -S ttf-ricty noto-fonts
$ sudo pacman -S dmenu
```
```
$ sudo systemctl enable lightdm
```

#### ビデオドライバ(NVIDIA)
- xf86-vido-intel : IntelのCPU内蔵グラフィックを使う場合のドライバ?
- nvidia : nvidiaのGPUを使うためのドライバ
- nvidia-prime : GPUとCPU内蔵グラフィックを両方使うときに必要らしい？(https://wiki.archlinux.jp/index.php/PRIME#PRIME_.E3.83.AC.E3.83.B3.E3.83.80.E3.83.BC.E3.82.AA.E3.83.95.E3.83.AD.E3.83.BC.E3.83.89)
```
$ sudo pacman -S xf86-video-intel nvidia
```

以下試行錯誤したもの(よくわからなくて保留)
```
$ xrandr -q
表示できるディスプレイ一覧を表示

$ sudo nvidia-xconfig
自動で/etc/X11/xorg.confに設定を書き込む(バックアップも同じディレクトリに.backupで作成してくれる)
ただ，この設定だとHDMI出力がデフォルトになってlaptopの画面が使えなくなる

$ vim /etc/X11/xorg.conf
Section "Monitor"
    Identifier  "VGA1"
    Option      "Primary" "true"
EndSection

Section "Monitor"
    Identifier  "HDMI1"
    Option      "LeftOf" "VGA1"
EndSection

以上のことの記載したパターンも試したがHDMI出力が自動で行われなかった

$ xrandr --output eDP1 --auto  --output HDMI1-0-0 --auto --right-of eDP1
このコマンドでマニュアル的に複数ディスプレイの実現ができる

```

加えてHDMI出力した時のみ音声出力ができた(alsomixerとかでみてもnvidaのメディアしか見つからないのが関係してそう??)

> > グラッフィックをGPUオンリーにする場合
> KMS (Kernel Mode Settings) を有効化し、ブートプロセスの早い段階で GPU をセットアップ (Late KMS start) するよう設定します。 NVIDIA 以外のドライバなら以下の設定は必要ありません。
> まずはブートローダーエントリ /boot/loader/entries/arch.conf を編集し、カーネルパラメータ nvidia-drm.modeset=1 を追加します。
> ```
> $ sudoedit /boot/loader/entries/arch.conf
> ```
> ```/boot/loader/entries/arch.conf
> -options root=PARTUUID=********-****-****-****-************ rw
> +options root=PARTUUID=********-****-****-****-************ nvidia-drm.modeset=1 rw
> ```
> /etc/mkinitcpio.conf を編集し、 initramfs にロードするモジュールを追加します。
> ```
> $ sudoedit /etc/mkinitcpio.conf
> ```
> ```/etc/mkinitcpio.conf
> -MODULES=()
> +MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
> ```
> mkinitcpio コマンドで initramfs を再生成します。
> ```
> $ sudo mkinitcpio -P
> ```
> 最後に /etc/pacman.d/hooks/nvidia.hook を作成し、 nvidia パッケージのアップグレード時に自動的に initramfs を再生成するよう pacman フックを設定します。
> ```
> $ sudo mkdir /etc/pacman.d/hooks
> $ sudoedit /etc/pacman.d/hooks/nvidia.hook
> ```
> ```/etc/pacman.d/hooks/nvidia.hook
> [Trigger]
> Operation=Install
> Operation=Upgrade
> Operation=Remove
> Type=Package
> Target=nvidia
> 
> [Action]
> Depends=mkinitcpio
> When=PostTransaction
> Exec=/usr/bin/mkinitcpio -P
> ```
> https://blog.livewing.net/install-arch-linux-2021

#### ウィンドウマネージャ(i3)

- i3 : i3に必要なものが四つ入ってるらしい
  - i3-wm
  - i3blocks
  - i3lock
  - i3status
```
$ sudo pacman -S i3
```

ここまできたら再起動
```
$ systemctl reboot
```

再起動後，GUIのログイン画面が出現するため，それでログイン

#### i3とかの設定
xorgでのキーボードレイアウトをjpに変える
```
$ sudo localectl set-x11-keymap jp
$ sudoedit /etc/locale.conf
-LANG=en_US.UTF-8
+LANG=ja_JP.UTF-8

$ systemctl reboot
```

CapsLockとCtrlを入れ替える
ついでにリピート入力の設定を`xset`でする
```
$ vim ~/.Xmodmap
clear lock
clear control
keycode 66 = Control_L
add control = Control_L Control_R

$ xmodmap ~/.Xmodmap
$ nano ~/.xprofile
#!/bin/sh

xset r rate 300 25
xmodmap ~/.Xmodmap
```

### その他
- ブラウザ : vivaldi , chromium, firefox
- 音声管理 : pulseaudio, alsa-utils
- 背景設定 : fehとか
- i3の操作コマンド変更 : ~/.config/i3/config
- 日本語化 : Mozc(fcitx fcitx-mozc fcitx-im fcitx-configtool)
- 開発環境 : vscode(code, visual-studio-code-bin) gcc
- ファイルマネージャ : ranger

```
$ sudo pacman -S vivaldi vivaldi-ffmpeg-codecs feh
$ sudo pacman -Rs vivaldi vivaldi-ffmpeg-codecs
$ sudo pacman -S firefox
$ sudo pacman -S code gcc
$ sudo pacman -S pulseaudio alsa-utils
```
#### ファイルマネージャ(ranger)
インストールしたのちデフォルトの設定をファイルに起こすコマンドを実行する．
```
$ sudo pacman -S ranger
$ ranger --copy-config=all
```
設定ファイルは`~/.config/ranger/rc.conf commands.py rifle.conf`にある

```
$ vim ~/.config/ranger/rc.conf
- set show_hidden false
+ set show_hidden true
```
隠しファイルを常時表示するようにする(デフォルトは`zh`で表示をトグルできる)

##### rangerの基本的な操作
基本的にvimライクな操作
- hjkl : 階層などの移動
- E : エディタ(`$EDITOR`)で開く(多分vim)
- i : ページャで開く(多分less)
- r : アプリを指定して開く(codeとかで開ける)

> http://malkalech.com/ranger_filer
> https://wiki.archlinux.jp/index.php/Ranger

#### 背景設定(feh)
解像度確認用のコマンドをインストールして，画面の解像度をしらべる
```
$ sudo pacman -S xorg-randr
$ randr
```

今回は1980*1080だったのでそれに合うような画像を適当に見繕う．
(今回はリムーバブルメディア経由で他PCで編集した画像を`~/Picture/`に持ってきた)
画像を確認する時は`$ feh <画像ファイル>`でプレビューが見れる．

背景に設定する時は以下のコマンドを実行する．
一番上のコマンドを実行すると`~/.fehbg`というシェルスクリプトが生成されるため，再起動時に壁紙を設定するためにコンフィグファイルに追記する．
```
$ feh --bg-center <画像ファイル>
$ vim ~/.config/i3/config

+ exec --no-startup-id ~/.fehbg
```

> http://neko-mac.blogspot.com/2022/04/archlinux.html
> https://ja.linux-console.net/?p=9740#gsc.tab=0
> http://malkalech.com/i3_wm_3_shortcut_keys

#### 日本語化(Mozc)
Mozc(fcitx5)をインストールする．
- fcitx5-im : fcitx5, fcitx5-configtool, fcitx5-gtk(firefoxとかで日本語化が使えるようになる??), fcitx5-qt
```
$ sudo pacman -S fcitx5-im fcitx5-mozc
```
環境変数や起動設定を追加する．
```
$ vim ~/.xprofile
+ export GTK_IM_MODULE=fcitx
+ export QT_IM_MODULE=fcitx
+ export XMODIFIERS=@im=fcitx

$ vim ~/.config/i3/config
+ exec --no-startup-id fcitx5
$ reboot
```

> https://wiki.archlinux.jp/index.php/Mozc
> https://wiki.archlinux.jp/index.php/Fcitx5
> https://slacknotebook.com/arch-linux-how-to-setup-fcitx5-2023/
> https://riq0h.jp/2023/02/25/211717/
> https://blog.livewing.net/install-arch-linux-2021
> https://opensquare.jp/2021/09/18/fcitx5の設定/

#### urxvtの設定
デフォルトが白すぎてみづらいので透明化とか色々設定する．
```
$ vim ~/.Xresources
--別掲予定--

$ xrdb -merge ~/.Xresources
```

> https://retrotecture.jp/freebsd/urxvt.html
> https://blog.desdelinux.net/ja/personalizando-urxvt-rxvt-unicode-esa-magnifica-consola/
> http://malkalech.com/urxvt_terminal_emulator
> https://wiki.archlinux.jp/index.php/X_resources
> https://wiki.archlinux.jp/index.php/Rxvt-unicode
> https://man.archlinux.org/man/urxvt.1

#### 音声管理


#### i3の設定(~/.config/i3/config)
一旦保留

> https://riq0h.jp/2023/02/25/211717/

#### i3blocksの設定(~/.config/i3blocks/config)

一旦保留

---

> #### usbのマウントについて
> usb-TypeAのポートは左に2つ，右に1つある．
> どのポートにリムーバブルメディアを挿しても`$ lsblk -fp`や`$ sudo fdisk -l`で認識が確認できない．
> ただ`$ sudo dmesg`で挿されたことは確認できた．
> 
> 挿した状態で起動するとリムーバブルメディアを`$ lsblk -fp`や`$ sudo fdisk -l`で確認できるため．
> 以下のコマンドでマウントするとできる．
> ```
> マウント手順
> $ sudo mkdir /mnt/usb #初めてするときだけこのディレクトリを作成する必要がある
> $ sudo mount /dev/sda1 /mnt/usb
> 
> アンマウント手順
> $ sudo umount /mnt/usb
> ```
> 
> ただ，`fstab`とかを導入して自動マウント(fstabとか)を設定したら解決するかも(とりあえず保留)
> 
> 追伸 : リムーバブルメディアを挿したまま再起動したら抜き差ししても認識するようになった(自動マウントではない)．


#### 参考URL
* https://wiki.archlinux.jp/index.php/Xinit
* https://wiki.archlinux.jp/index.php/ディスプレイマネージャ
* https://trap.jp/post/425/
* https://riq0h.jp/2023/02/25/211717/
* https://qiita.com/Tonooo/items/256fcaffd7113b1ed018
* https://blog.livewing.net/install-arch-linux-2021
* http://malkalech.com/i3_window_manager