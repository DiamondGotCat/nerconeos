# NerconeOS
A homemade OS from the kernel level

## for non-Japanese users:
Sorry, this project is still under development and only supports Japanese, which is Nercone's native language. As the project progresses, it may support English and other languages ​​at some point.

## 現時点の予定
- カーネルレベルから自作する
- amd64(x86_64)とaarch64(arm64)に対応する
- RustがUEFIブートに対応していると聞いたのでRustで書く
    - PythonとSwiftが得意だけどどちらも無理そうなのでRust。
    - Rustはいつか勉強しようと思ってたのでちょうどいい
- 描画先はUEFIが渡してくれる(らしい)のでそれを利用
- できるだけAIに頼らない
    - Rustは(ほぼ)初めてなので、標準の機能とかで「こんな機能ある？」とかは聞くかもしれない
    - コード全体をAIに作らせたりリファクタリングさせたりするようなことはしない
    - AIに書かせるよりは自分で書いたほうがRustの勉強になりそうなのが理由の一つ
- ちゃんとモジュール分けする
    - `nerconeos-base`: ベース (起動して別のモジュールをロードする)
    - `nerconeos-shell`: シェル (自作シェル、ネルコンシェル/ネルシェル/ネルシュ`nerconesh` `nersh`)
    - `nerconeos-tui`: テキストの描画と、ある場合はシェルを実行して表示する
    - `nerconeos-tui-font`: `nerconeos-tui` で使う内蔵フォントパック
    - `nerconeos-gui`: GUI(Linuxでいうウィンドウマネージャーとデスクトップ環境)　最初は最低限実装して、後から本格的なUIにする
    - `nerconeos-fs`: 独自ファイルシステムと一般的に使われているLinuxファイルシステムの一部のものを実装。
    - `nerconeos-driver`: 一般的にPCについているハードウェア向けの内蔵ドライバー。一応、後付けのドライバーに切り替えられるようにする予定。
