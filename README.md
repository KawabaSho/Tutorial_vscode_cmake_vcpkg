# VS Code ではじめる C/C++ 環境

VS Code で `CMake` と `vcpkg` を使って C++ 開発環境を作るための最小サンプルです。 `CMake` はビルド管理システムとして使用し、 `vcpkg` は外部のC/C++パッケージを管理するライブラリです。 
このプロジェクトでは `fmt` を `vcpkg` の manifest mode で導入しています。なお、Githubでの設定は https://github.com/KawabaSho/Tutorial_vscode_python_uv を参考にしてください。

## 構成

- ビルドシステム: `CMake`
- パッケージ管理: `vcpkg`
- エディタ: `Visual Studio Code`
- コンパイラ: `MSVC`
- 依存ライブラリ: `fmt`

## 前提条件

以下をインストールしてください。

- `Git`
- `Visual Studio 2026` の C++ 開発環境
- `CMake`
- `Visual Studio Code`

Visual Studio には少なくとも次を入れておくと進めやすいです。

- `Desktop development with C++`
- Windows SDK
- MSVC toolchain

VS Code には次の拡張機能を入れてください。

- `C/C++`
- `CMake Tools`

## vcpkg のインストール

`vcpkg`をGithubからインストールする手順を説明します。まず、C直下でvcpkgをインストールする方法は以下のコマンドを PowerShell で実行します。

```powershell
git clone https://github.com/microsoft/vcpkg C:\vcpkg
C:\vcpkg\bootstrap-vcpkg.bat
```

次に環境変数 `VCPKG_ROOT` を設定します。  
このプロジェクトの `CMakePresets.json` では `VCPKG_ROOT` を参照しているので、システム環境変数で `VCPKG_ROOT` を定義して、内容は `C:\vcpkg` としてください。

コマンドで永続的に設定する場合:

```powershell
[System.Environment]::SetEnvironmentVariable("VCPKG_ROOT", "C:\vcpkg", "User")
```

## プロジェクトの作成

プロジェクトを作りたいフォルダに移動して、右クリックで PowerShell を開いて以下を実行します。

```powershell
git clone https://github.com/KawabaSho/Tutorial_vscode_cmake_vcpkg.git
```

実行すると、以下のようなファイルが生成されます。

```powershell
C:.
├─.vscode/c_cpp_properties.json
├─.gitignore
├─CMakeLists.txt
├─CMakePresets.json
├─main.cpp
└─vcpkg.json
```

次に VS Code で編集するために、VS Code を以下のコマンドで起動します。
```powershell
code .
```

## c_cpp_properties.json を確認する

CMakeでビルド管理しているため、VS Code で警告が表示されていても正常に動きますが、精神的ストレスを与えてくるので、 `includePath` と `compilerPath` に適切な情報を登録していきます。なお、`"env":{...}` は `c_cpp_properties.json` 内で変数を定義する場合の書き方です。自身のコンパイラーのバージョンにあったパスに書き直してください。ただし、本来は `c_cpp_properties.json` を直接書き換えることは推奨されていないため、`F1 + "C/C++: Edit Configurations (UI)"` でUIを開き、編集してください。

## CMakePreset.json を確認する
### コンパイラ構成
このプロジェクトは `CMakePresets.json` の `Default` preset を使ってビルド環境を作っていきます。
現在の設定では generator が `Visual Studio 18 2026` になっています。これは、Visual StudioのMSVC を使ってビルドすることを意味します。

```json
"generator": "Visual Studio 18 2026"
```

使っている Visual Studio のバージョンが異なる場合は、環境に合わせて `CMakePresets.json` の generator を変更してください。分からない場合はターミナルで `cmake --help` を実行してください。Generators の一覧が確認できます。

### パッケージ管理
外部ライブラリをvcpkgで管理しているため、その情報の登録を `toolchainFile` で行っています。また、vcpkgはPC本体のパッケージを汚染しないために、pythonの仮想環境のような設定を行います。vcpkgではマニフェストを有効化させます。

```json
"cacheVariables": {
    "VCPKG_MANIFEST_MODE": true,
    "VCPKG_MANIFEST_DIR": "${sourceDir}"
}
```

マニフェストモードで新しいパッケージを追加する場合は VS Code のターミナルで以下を実行してください。`build` フォルダ内でパッケージがインストールされます。

```powershell
 vcpkg add port <パッケージ名>
```

削除したい場合は、`vcpkg.json` で直接パッケージ名を削除してください。

※ PC本体にインストールするときは、 `vcpkg install <パッケージ名>` 、アンインストールするときは `vcpkg remove <パッケージ名>` になります（クラシックモードといいます）。


## CMakeLists.txt を確認する

例えば、`fmt` のパッケージを追加したら、`CMakeLists.txt` 内で以下のようにライブラリの追加を行います。

```m
find_package(fmt CONFIG REQUIRED)
target_link_libraries(my_program PRIVATE fmt::fmt)
```

## main.cpp を確認する

`fmt` ライブラリを使った文字列の出力プログラムになります。追加のライブラリを簡単にインクルードして使えるようになります。

## プロジェクトをビルドする

VS Code でこのフォルダを開いたら、次の順で進めます。

1. `F1`を押して、`CMake: Select Configure Preset` で `Default Config` を選ぶ
2. `F1`を押して、`CMake: Configure` を実行する
3. `F1`を押して、`CMake: Build` を実行する

初回の configure 時に `vcpkg.json` を読み取り、`fmt` が自動でインストールされます。

### UIでビルドする場合

CMake tools を拡張機能でインストールしていると、左下に `歯車のbuild` ボタンがあると思われます。それを押し、`Default Config` を選択すれば、ビルドが行われます。 `歯車のbuild` の右側にある再生ボタンを押せば、プロジェクトの実行結果を確認できます。出ない場合は `歯車のbuild` をもう一度押すか、`F1`を押して、`CMake: Delete Cache and Reconfigure` を実行してください。それでも直らない場合は、`build` フォルダを丸ごと消して、`F1`を押して`Developer: Reload Window`を選択してください。それからまた `歯車のbuild` を押すところから始めてください。


### コマンドラインでビルドする場合

VS Code を使わなくてもビルドできます。

```powershell
cmake --preset default
cmake --build build
```

## 実行する

ビルド後、生成された実行ファイルを起動します。実行ファイルは `build\Debug\my_program.exe` にあります。`歯車のbuild` の右側にある再生ボタンを押せば、プロジェクトの実行結果を確認できます。または、ターミナルで

```powershell
.\build\Debug\my_program.exe
```

を実行してください。`.\<実行ファイル名.exe>` で実行できます。成功すると次のメッセージが表示されます。

```text
Hello, vcpkg with CMake!
```

## このプロジェクトで使っているファイル

- `CMakeLists.txt`: 実行ファイルの定義と `fmt` のリンク
- `CMakePresets.json`: generator、build ディレクトリ、vcpkg toolchain 設定
- `vcpkg.json`: 依存パッケージ定義
- `main.cpp`: 動作確認用の最小コード

## 番外編 Githubでの管理

詳しくは https://github.com/KawabaSho/Tutorial_vscode_python_uv を参考にしてください。Github の拡張機能を使うことができれば、UI で Github に push も可能です(UIの方が楽です)。補足ですが、Github で管理不要なものは `.gitignore` で設定しています。

```powershell
git init
git config --local user.name "ユーザー名"
git config --local user.email "Github上のメールアドレス"
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin git@github.com:ユーザー名/リポジトリ名.git
git push -u origin main
```

※ UIの場合は Github の拡張機能内の `Changes` の横の `+` ボタンを押して `Staged Changes` に変更したファイルを移動させたら、`Commit` の上の Message に内容を書いて `Commit` を押します。最終的に `Sync Changes` を押して Push 完了になります。

## メモ
### `vcpkg.json` のバージョン管理例

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "builtin-baseline": "3df739ef13451c2f90117b864a7c865fae3e4822",
  "dependencies": [
    {
      "name": "imgui",
      "version>=": "1.90.4"
    }
  ]
}
```

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "builtin-baseline": "3df739ef13451c2f90117b864a7c865fae3e4822",
  "dependencies": [
    "imgui"
  ],
  "overrides": [
    {
      "name": "imgui",
      "version": "1.89.9"
    }
  ]
}
```

vcpkg はライブラリのバージョン管理の手法（セマンティックバージョニングか、日付ベースかなど）によって、json に書くプロパティ名が変わります。ImGui のような通常のライブラリは version ですが、他ライブラリでは以下のように書き分ける必要があります。

- `version`: 通常のルール（1.90.4 など）
- `version-date`: 日付ベースのライブラリ（2026-01-15 など）
- `version-string`: バージョン規則が特殊なもの（v1.0-beta などの文字列）

### ランタイム設定

MSVCのランタイム(CRT)は既定では、動的リンク`MD`/`MDd`であるので、他のライブラリが混ざっても安全です。スタンドアロンの実行ファイルかつCRTの依存を減らしたい場合は、`CMakePresets.json` で `"cacheVariables"` の中身を以下のように変更してください。

```json
{
  "cacheVariables": {
      "CMAKE_CXX_STANDARD": "20",
      "CMAKE_EXPORT_COMPILE_COMMANDS": "ON",
      "VCPKG_MANIFEST_MODE": true,
      "VCPKG_MANIFEST_DIR": "${sourceDir}",
      "VCPKG_TARGET_TRIPLET":"x64-windows-static",
      "CMAKE_MSVC_RUNTIME_LIBRARY": "MultiThreaded$<$<CONFIG:Debug>:Debug>"
  }
}
```

DebugモードでCRTを静的リンクに変更しています。また、この場合、依存パッケージも静的にする必要があるので `"VCPKG_TARGET_TRIPLET"`:`"x64-windows-static"` としています(動作確認はしていないです)。

### 依存ライブラリの動的リンク設定

依存ライブラリを `.dll` ファイルとして生成したい場合は、`CMakePresets.json` で以下の指示に変更してください。CRTは動的リンクしつつ、かつ、`build\Debug` 内で `fmt.dll` などが生成されます。

```json
{
"VCPKG_TARGET_TRIPLET":"x64-windows"
}
```

ちなみに、このコマンドはCRTは動的リンクしつつ、依存関係は1つの実行ファイルの中に埋め込む場合は`"x64-windows-static-md"` となります。また、CRTが静的で依存ライブラリが動的の場合は基本的に出来ないようです。理論上はできるようですが、

- `DLL` と `EXE` で `new/delete` を共有しない
- `FILE*` や `std::string`、`std::vector` など CRT 管理リソースをまたがない
- CRT 間で状態を共有しない

などの制約があるみたいです(エラーが確実に起こりそうですね)。
