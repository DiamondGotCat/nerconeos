# NerconeOS
A homemade OS from the kernel level

## for non-Japanese users:
Sorry, this project is still under development and only supports Japanese, which is Nercone's native language. As the project progresses, it may support English and other languages ​​at some point.

## 独自用語
- `NerconeOS`: このプロジェクトの名前。
- `NerKernel`: NerconeOSのカーネルのこと。
- `NerconeFS`: NerconeOS独自のパーティションのフォーマット。
- `NKMod`: NerKernelの内蔵/標準モジュール。(基本的には`nerconeos-base`以外のモジュールを指す)
- `NKExt`: NerKernelの拡張機能。NeXTSTEP/macOSの`kext`(Kernel EXTension)の名前を参考にしました。
- `ドットファイル`: `.`から始まるファイル。 (独自ではない気がする)

## 現時点の予定
- カーネルレベルから自作する
- 実用性のある、安定したOSにする
- 一部はPOSIXやLinuxを真似する
- amd64(x86_64)とaarch64(arm64)に対応する
- ソフトウェアの形式はELF
    - もしかしたら独自バイナリに対応するかも
- RustがUEFIブートに対応していると聞いたのでRustで書く
    - PythonとSwiftが得意だけどどちらも無理そうなのでRust
    - Rustはいつか勉強しようと思ってたのでちょうどいい
- 描画先はUEFIが渡してくれる(らしい)のでそれを利用
- できるだけAIに頼らない
    - Rustは(ほぼ)初めてなので、標準の機能とかで「こんな機能ある？」とかは聞くかもしれない
    - コード全体をAIに作らせたりリファクタリングさせたりするようなことはしない
    - AIに書かせるよりは自分で書いたほうがRustの勉強になりそうなのが理由の一つ
- とりあえずExt4などを実装してからNerconeFSを実装する
    - 理由: 面倒
- ちゃんとモジュール分けする
    - `nerconeos-base`: ベース (起動してNKModやNKExtをロードする)
    - `nerconeos-shell`: シェル (自作シェル、ネルコンシェル/ネルシェル/ネルシュ`nerconesh` `nersh`)
    - `nerconeos-tui`: テキストの描画と、ある場合はシェルを実行して表示する
    - `nerconeos-tui-font`: `nerconeos-tui` で使う内蔵フォントパック
    - `nerconeos-gui`: GUI(Linuxでいうウィンドウマネージャーとデスクトップ環境)　最初は最低限実装して、後から本格的なUIにする
    - `nerconeos-fs`: NerconeFSと一般的に使われているLinuxファイルシステムの一部のものを実装。
    - `nerconeos-driver`: 一般的にPCについているハードウェア向けの内蔵ドライバー。一応、後付けのドライバーに切り替えられるようにする予定。
- `.`から始まるファイル(ドットファイル)は隠しファイルとして扱う

### NerconeOSのEFIシステムパーティションの基本的な構造
ESP (EFI System Partition)
- `EFI`
    - `BOOT`
        - `BOOTIA32.EFI`: x86 32bit (i386) 向けブートローダー (いつか追加するかも)
        - `BOOTX64.EFI`: x86 64bit (amd64/x86_64) 向けブートローダー (メイン)
        - `BOOTARM.EFI`: AArch 32bit (aarch32/armhf) 向けブートローダー (いつか追加するかも)
        - `BOOTAA64.EFI`: AArch 64bit (aarch64/arm64) 向けブートローダー (追加予定)
        - `BOOTRISCV64.EFI`: RISC-V 64bit (riscv64) 向けブートローダー (いつか追加するかも)

### NerconeOSのメインパーティション内の基本的な構造
- `mod`: NKModの保管場所。全て`*-unknown-none`でビルド済みのELFファイル。アクセスにはスーパーユーザー権限が必要。
    - `base`: `nerconeos-base` (これがブートローダーから呼ばれる)
    - `shell`: `nerconeos-shell`
    - `tui`: `nerconeos-tui`
    - `tui-font`: `nerconeos-tui-font`
    - `gui`: `nerconeos-gui`
    - `fs`: `nerconeos-fs`
    - `driver`: `nerconeos-driver`
- `ext`: NKExtの保管場所。アクセスにはスーパーユーザー権限が必要。
- `app`: NerconeOS向けアプリの保管場所。アクセスにはスーパーユーザー権限が必要。
    - `usr`: スーパーユーザー権限なしでアクセスが可能なディレクトリ。
- `bin`: Linuxの`/bin`と同じようなもの。ELFやバイナリの保管場所。アクセスにはスーパーユーザー権限が必要。
    - `usr`: スーパーユーザー権限なしでアクセスが可能なディレクトリ。
- `dev`: Linuxの`/dev`と同じようなもの。アクセスにはスーパーユーザー権限が必要。
- `etc`: Linuxの`/etc`と同じようなもの。アクセスにはスーパーユーザー権限が必要。
    - `usr`: スーパーユーザー権限なしでアクセスが可能なディレクトリ。
- `tmp`: Linuxの`/etc`と同じようなもの。一時ファイルを保管しておく場所。
- `usr`: ユーザーディレクトリの保管場所。アクセスにはスーパーユーザー権限が必要。
    - `{ユーザー名}` ユーザーディレクトリ。そのユーザーはアクセス可能。
