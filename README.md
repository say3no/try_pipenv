# try_pipenv

2019/01/11の時点で、僕はpythonの依存関係の整理に`requirements.txt`と`requirements.dev.tx`のみを書いている人間で、`setup.py`のことはちーともわからない人間なのだが、[Pythonのパッケージ周りのベストプラクティス（2019年現在） - エムスリーテックブログ](https://www.m3tech.blog/entry/python-packaging)というブログによれば`pipenv`がいいらしいので、早速触ってみた。

触ってみた今でも`setup.py`のことはわかっていないので、このブログで言うベストプラクティスとしての`setup.py`および`setup.cfg`はよくわかっていないのだが、virtualenv,pyenv,requirements.txtとrequriements.dev.txtの面倒くささを解決してくれることはわかったので、今後は`pipenv`を使っていこうと思います。

```sh
pyenv global 3.7.0
pyenv rehash
python -V
brew install pipenv
```

`pipenv -h`して感じを掴む。とりあえず`pipenv install`が`yarn init`とかに該当すると思ったので試す。

インタラクティブなインターフェースを期待していたのだが、現在試用しているpythonを種とした`venv`を作成し、続いて噂の`Pipfile`, `Pipfile.lock`を生成した。

`Pipfile`の中身は、こんな感じだ。前情報の通り、`requirements.txt`と`requirements.dev.txt`によって開発環境と商用環境（とでも言おうか）の依存関係を別々に管理するといった手間がなさそうで、よい。

```ini
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]

[dev-packages]

[requires]
python_version = "3.7"
```

`pipenv install` をすると、標準出力の最後に次の二行が出力された。

> To activate this project's virtualenv, run `pipenv shell` .
>
> Alternatively, run a command inside the virtualenv with `pipenv run.`

書いてある通りなのだが、`pipenv shell`で`venv/bin/activate`が実行される。その実態はリポジトリ配下ではなく、具体的に言うと`. /Users/say3no/.local/share/virtualenvs/try_pipenv-2OzW682t/bin/activate` で実行されているようだ。

ここで、`pipenv shell`のような形で実装されているのならば、他のパッケージ管理ソフトと同じように、`Pipfile`に任意のコマンドを定義して、`pipenv 任意のコマンド`みたいなことが出来るのではないかと予感させた。（たぶんできる）

`Pipfile`と`Pipfile.lock`をバージョン管理に含めるのは、phpやらjsやらと同じらしい。

## 実際に運用してみる

`aiohttp`を使ったしょーもないアプリを使ってみる。

ここで、`pipenv install aiohttp`なのか、`pipenv shell`してから`pip install aiohttp`なのか迷ったのだが、`composerとかbundlerのいいとこ取りをして作られたのがpipenvだ`と公式に書いてあったので、`pipenv install aiohttp`を試したところ、やはり間違っていなかった。

なお、[`pip==18.0`じゃないとうまく動かないとのこと。](https://github.com/pypa/pipenv/issues/2871)

しかし、aiohttpが`[packages]`と[dev-packages]`に追加されて欲しかったのだが、`[packages]`にしか追加されなかったのは悲しかった。両方に追加するオプションが必ずあるはずだと思い、`pipenv install -h` してみる。

```sh
$ pipenv install -h
Usage: pipenv install [OPTIONS] [PACKAGE_NAME] [MORE_PACKAGES]...

Options:
  -d, --dev                Install package(s) in [dev-packages].
  --three / --two          Use Python 3/2 when creating virtualenv.
  --python TEXT            Specify which version of Python virtualenv should
                           use.
  --pypi-mirror TEXT       Specify a PyPI mirror.
  --system                 System pip management.
  -r, --requirements TEXT  Import a requirements.txt file.
  -c, --code TEXT          Import from codebase.
  -v, --verbose            Verbose mode.
  --ignore-pipfile         Ignore Pipfile when installing, using the
                           Pipfile.lock.
  --sequential             Install dependencies one-at-a-time, instead of
                           concurrently.
  --skip-lock              Ignore locking mechanisms when installing—use the
                           Pipfile, instead.
  --deploy                 Abort if the Pipfile.lock is out-of-date, or Python
                           version is wrong.
  --pre                    Allow pre-releases.
  --keep-outdated          Keep out-dated dependencies from being updated in
                           Pipfile.lock.
  --selective-upgrade      Update specified packages.
  -h, --help               Show this message and exit.
```

ないやんけ！！！

しょうがないので、`pipenv install aiohttp -d`を実行した。一見不親切とも思ったのだが、実際の運用ではdev∋prodの関係が成立するとは限らない…のかもしれない。俺が未熟なだけで。

ほかにも、`pytest`,`pytest-watch`,`pytest-pythonpath`,`pytest-aiohttp`,`autopep8`,`pylint`は開発環境に必須だと思うので、`-d`で入れておく。

### アップデートの流れ

依存関係にあるパッケージのいずれかがアップデート可能かどうかは`pipenv update --outdated`で確認できる。このコマンドでアップデートをしたければ`pipenv update`で、どれか一つだけアップデートしたいのであれば`pipenv update <pkg>`だ。

## 斜め読みして便利だと思った機能たち

このリポジトリは[公式](https://pipenv-ja.readthedocs.io/ja/translate-ja/index.html)を雑に読み勧めて適当に触っているだけのログなのだが、便利そうだなと思った機能も備忘録として書いておく。

- `pipenv check`

>セキュリティの脆弱性とPipfileにあるPEP 508マーカーをチェックします。

- `pipenv graph`

パッケージの依存関係を可視化

- `pipenv lock -r`

`Pipfile`から`requirements.txt`を生成。

- `pipenv lock -dr`

`Pipfile`から`requirements.dev.txt`を生成。


## `.bashrc`に以下の二行を追加

```sh
export PIPENV_VENV_IN_PROJECT=true`
eval "$(pipenv --completion)"
```

環境変数`export PIPENV_VENV_IN_PROJECT=true`でプロジェクト直下に`venv`を作成してくれる。