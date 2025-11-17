# NerconeOS
A homemade OS from the kernel level

## for non-Japanese users:
Sorry, this project is still under development and only supports Japanese, which is Nercone's native language. As the project progresses, it may support English and other languages ​​at some point.

## 現在の状態
現在は実際のコードが全くない状態で、計画段階です。

## 独自用語
- `NerconeOS`: このプロジェクトの名前。
- `NerKernel`: NerconeOSのカーネルのこと。
- `NerconeFS`: NerconeOS独自のパーティションのフォーマット。
- `NKMod`: NerKernelの内蔵/標準モジュール。(基本的には`nerconeos-base`以外のモジュールを指す)
- `NKExt`: NerKernelの拡張機能。NeXTSTEP/macOSの`kext`(Kernel EXTension)の名前を参考にしました。
- `KPart`: カーネルパーティションのこと。
- `MPart`: メインパーティションのこと。
- `ドットファイル`: `.`から始まるファイル。 (独自ではない気がする)

## 現時点の予定
- カーネルレベルから自作する
- 実用性のある、安定したOSにする
- 一部はPOSIXやLinuxを真似する
- amd64(x86_64)とaarch64(arm64)に対応する
    - まずはamd64に対応し、ある程度開発が進んでからaarch64に対応する予定。
- ソフトウェアの形式はELF
    - もしかしたら独自バイナリに対応するかも
- RustがUEFIブートに対応していると聞いたのでRustで書く
    - PythonとSwiftが得意だけどどちらも無理そうなのでRust
    - Rustはいつか勉強しようと思ってたのでちょうどいい
- 描画先はUEFIが渡してくれるFramebufferを利用する。
    - ただしFramebufferは高解像度化やマルチモニタ化などはできないため、通常はモニタ対応は`nerconeos-driver`の一部実装が担当する。
- できるだけAIに頼らない
    - Rustは(ほぼ)初めてなので、標準の機能とかで「こんな機能ある？」とかは聞くかもしれない
    - コード全体をAIに作らせたりリファクタリングさせたりするようなことはしない
    - AIに書かせるよりは自分で書いたほうがRustの勉強になりそうなのが理由の一つ
- NerconeFSを実装してからExt4に対応する。
    - Ext4はすでに存在するけれど、ジャーナリングとかのいろんな機能に対応するのは面倒なため、NerconeFS、特にNerconeFS Liteから先に実装する。
- ちゃんとモジュール分けする
    - `nerconeos-base`: ベース (起動してNKModやNKExtをロードする)
    - `nerconeos-test`: 開発中に使用するモジュール。動作しているかなどの確認や実験に使用する。
    - `nerconeos-shell`: シェル (自作シェル、ネルコンシェル/ネルシェル/ネルシュ`nerconesh` `nersh`)
    - `nerconeos-tui`: テキストの描画と、ある場合はシェルを実行して表示する
    - `nerconeos-tui-font`: `nerconeos-tui` で使う内蔵フォントパック
    - `nerconeos-gui`: GUI(Linuxでいうウィンドウマネージャーとデスクトップ環境)　最初は最低限実装して、後から本格的なUIにする
    - `nerconeos-fs`: NerconeFSと一般的に使われているLinuxファイルシステムの一部のものを実装。
    - `nerconeos-driver`: 一般的にPCについているハードウェア向けの内蔵ドライバー。モニターやUSB、GPUなど。後付けのドライバーを優先して、利用できない場合は`nerconeos-driver`の内部実装、対応していない場合はそのデバイスを「非対応」とする。
- `.`から始まるファイル(ドットファイル)は隠しファイルとして扱う

### NerconeFSの特徴や機能

#### NerconeFS Lite
- ある程度の利用に対応できるファイルシステム。
- 実装を簡単にするための初期バージョンで、開発やテスト時に使用するバージョン。
- 10GiBの固定長ファイルシステム
- ファイルサイズは最大500MiB
- ファイル名は最大10文字、文字セットはUTF-8

#### NerconeFS
- 一般的な利用に対応できるファイルシステム。
- 最大1TiBの可変長ファイルシステム。
- ファイルサイズは最大200GiB
- ファイル名は最大50文字、文字セットはUTF-8
- ハッシュ(SHA-256など)による全体/ファイル別の破損確認に対応。

#### NerconeFS Plus
- 小〜中規模のサーバー利用に対応できるファイルシステム。
- 最大100TiBの可変長ファイルシステム。
- ファイルサイズは最大5TiB
- ファイル名は最大200文字、文字セットはUTF-8
- ハッシュ(SHA-256など)による全体/ファイル別の破損確認に対応。
- スナップショット機能に対応。

#### NerconeFS Full
- 大規模のサーバー利用に対応できるファイルシステム。
- 最大10PiBの可変長ファイルシステム。
- ファイルサイズは最大50TiB
- ファイル名は最大1000文字、文字セットはUTF-8
- ハッシュ(SHA-256など)による全体/ファイル別の破損確認に対応。
- スナップショット機能に対応。
- 小規模のデータ破損の修復に対応。(破損する前まで戻すことができる)

### NerconeOSのEFIシステムパーティションの基本的な構造
ESP (EFI System Partition)
- `EFI`
    - `BOOT`: UEFIで実行されるブートローダーの保管場所。全て`*-unknown-uefi`でビルド済みのELFファイル。
        - `BOOTIA32.EFI`: x86 32bit (i386) 向けブートローダー (いつか追加するかも)
        - `BOOTX64.EFI`: x86 64bit (amd64/x86_64) 向けブートローダー (メイン)
        - `BOOTARM.EFI`: AArch 32bit (aarch32/armhf) 向けブートローダー (いつか追加するかも)
        - `BOOTAA64.EFI`: AArch 64bit (aarch64/arm64) 向けブートローダー (追加予定)
        - `BOOTRISCV64.EFI`: RISC-V 64bit (riscv64) 向けブートローダー (いつか追加するかも)

### KPartの基本的な構造
FAT16/FAT32
- `mod`: NKModの保管場所。全て`*-unknown-none`でビルド済みのELFファイル。
    - `base`: `nerconeos-base` (これがブートローダーから呼ばれる)
    - `test`: `nerconeos-test`
    - `shell`: `nerconeos-shell`
    - `tui`: `nerconeos-tui`
    - `tui-font`: `nerconeos-tui-font`
    - `display`: `nerconeos-display`
    - `gui`: `nerconeos-gui`
    - `fs`: `nerconeos-fs`
    - `driver`: `nerconeos-driver`
- `ext`: NKExtの保管場所。
- `cfg`: カーネルの設定ファイルの保管場所。
    - `nkmod.cfg`: NKMod関連の設定。モジュールの有効化/無効化などが可能。
    - `nkmod`: それぞれのNKModの設定を保管する場所。
        - `{NKModのID}.cfg`: NKModの設定ファイル。[^1]
    - `nkext.cfg`: NKExt関連の設定。カーネル拡張の有効化/無効化などが可能。
    - `nkext`: それぞれのNKExtの設定を保管する場所。
        - `{NKExtのID}.cfg`: NKExtの設定ファイル。[^2]

[^1]: NKModのIDには、`kpart/mod/*`のそれぞれのファイル名が使用されます。
[^2]: NKExtのIDには、`kpart/ext/*`のそれぞれのファイル名が使用されます。

### MPartの基本的な構造
NerconeFS/Ext4/etc
- `app`: NerconeOS向けアプリの保管場所。アクセスにはスーパーユーザー権限が必要。
    - `usr`: スーパーユーザー権限なしでアクセスが可能なディレクトリ。
- `bin`: Linuxの`/bin`と同じようなもの。ELFやバイナリの保管場所。アクセスにはスーパーユーザー権限が必要。
    - `usr`: スーパーユーザー権限なしでアクセスが可能なディレクトリ。
- `dev`: Linuxの`/dev`と同じようなもの。アクセスにはスーパーユーザー権限が必要。
- `etc`: Linuxの`/etc`と同じようなもの。アクセスにはスーパーユーザー権限が必要。
    - `usr`: スーパーユーザー権限なしでアクセスが可能なディレクトリ。
- `tmp`: Linuxの`/tmp`と同じようなもの。一時ファイルを保管しておく場所。
- `mnt`: Linuxの`/mnt`と同じようなもの。MPart以外のパーティションのマウントポイント。
    - `kpart`: KPartのマウントポイント。
- `usr`: ユーザーディレクトリの保管場所。アクセスにはスーパーユーザー権限が必要。
    - `{ユーザー名}` ユーザーディレクトリ。そのユーザーはアクセス可能。
