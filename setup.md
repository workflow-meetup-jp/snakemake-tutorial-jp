この記事は[Workflow Advent Calendar 2020](https://adventar.org/calendars/5339)の3日目の記事です。
4日に分けて[Snakemake Tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html)を和訳します。
これはその4日のうちの第1日目です。

# Setup
## Requirements
To go through this tutorial, you need the following software installed:

このチュートリアルを実行するには、次のソフトウェアがインストールされている必要があります:

- Python ≥3.5
- Snakemake ≥5.24.1
- BWA 0.7
- SAMtools 1.9
- Pysam 0.15
- BCFtools 1.9
- Graphviz 2.42
- Jinja2 2.11
- NetworkX 2.5
- Matplotlib 3.3

The easiest way to setup these prerequisites is to use the Miniconda Python 3 distribution.
The tutorial assumes that you are using either Linux or MacOS X.
Both Snakemake and Miniconda work also under Windows, but the Windows shell is too different to be able to provide generic examples.

これらの必要なソフトをセットアップする最も簡単な方法は、Miniconda Python 3 ディストリビューションを使用することです。
このチュートリアルでは、LinuxまたはMacOS Xのいずれかを使用していることを前提としています。
SnakemakeとMinicondaはどちらもWindowsでも動作しますが、Windowsのシェルは(訳注:Linuxとは)かなり異なるため、一般的な例を提供できません。

## Setup a Linux VM with Vagrant under Windows
If you already use Linux or MacOS X, go on with Step 1. If you use Windows, you can setup a Linux virtual machine (VM) with Vagrant.
First, install Vagrant following the installation instructions in the Vagrant Documentation.
Then, create a reasonable new directory you want to share with your Linux VM, e.g., create a folder vagrant-linux somewhere.
Open a command line prompt, and change into that directory.
Here, you create a 64-bit Ubuntu Linux environment with

すでにLinuxまたはMacOSXを使用している場合は、手順1に進みます。
Windowsを使用している場合は、Vagrantを使用してLinux仮想マシン(VM)をセットアップできます。
まず、Vagrantドキュメントのインストール手順に従ってVagrantをインストールします。
次に、Linux VMと共有する適切な新しいディレクトリを作成します。
たとえば、vagrant-linuxフォルダーをどこかに作成します。
コマンドラインプロンプトを開き、そのディレクトリに移動します。
ここでは、64ビットのUbuntu Linux環境を作成します

```
> vagrant init hashicorp/precise64
> vagrant up
```

## Step 1: Installing Miniconda 3

First, please open a terminal or make sure you are logged into your Vagrant Linux VM.
Assuming that you have a 64-bit system, on Linux, download and install Miniconda 3 with

まず、ターミナルを開くか、VagrantのLinux VMにログインしていることを確認してください。
あなたはLinuxの64ビットシステムを使用していると仮定しています。
下記で Miniconda 3 をダウンロード、インストールしてください。

```
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ bash Miniconda3-latest-Linux-x86_64.sh
```

On MacOS X, download and install with

MacOS Xであれば、下記でダウンロード、インストールしてください。

```
$ curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh -o Miniconda3-latest-MacOSX-x86_64.sh
$ bash Miniconda3-latest-MacOSX-x86_64.sh
```

For a 32-bit system, URLs and file names are analogous but without the `_64`.
When you are asked the question

32ビットシステムの場合、URLとファイル名は似ていますが、 `_64`はありません。
下記のように聞かれたら

```
Do you wish the installer to prepend the Miniconda3 install location to PATH ...? [yes|no]
```

answer with **yes**.
Along with a minimal Python 3 environment, Miniconda contains the package manager [Conda](https://conda.pydata.org/).
After opening a **new terminal**, you can use the new `conda` command to install software packages and create isolated environments to, e.g., use different versions of the same package.
We will later use [Conda](https://conda.pydata.org/) to create an isolated environment with all required software for this tutorial.

**yes**で答えてください。
Minicondaには、最小限のPython 3 環境に加えて、パッケージマネージャーの[Conda](https://conda.pydata.org/)が含まれています。
**新しいターミナル**を開いた後、`conda` コマンドを使用してソフトウェアパッケージをインストールしたり、分離された環境(例えば同じパッケージの異なるバージョンを使用するために)を作成できます。
後で[Conda](https://conda.pydata.org/)を使用して、このチュートリアルに必要なすべてのソフトウェアを備えた分離された環境を作成します。

## Step 2: Preparing a working directory

First, create a new directory snakemake-tutorial at a reasonable place and change into that directory in your terminal:

```
$ mkdir snakemake-tutorial
$ cd snakemake-tutorial
```

If you use a Vagrant Linux VM from Windows as described above, create that directory under /vagrant/, so that the contents are shared with your host system (you can then edit all files from within Windows with an editor that supports Unix line breaks). Then, change to the newly created directory. In this directory, we will later create an example workflow that illustrates the Snakemake syntax and execution environment. First, we download some example data on which the workflow shall be executed:

```
$ wget https://github.com/snakemake/snakemake-tutorial-data/archive/v5.24.1.tar.gz
$ tar --wildcards -xf v5.24.1.tar.gz --strip 1 "*/data" "*/environment.yaml"
```

This will create a folder data and a file environment.yaml in the working directory.

## Step 3: Creating an environment with the required software

The `environment.yaml` file can be used to install all required software into an isolated Conda environment with the name `snakemake-tutorial` via

下記で `environment.yaml` ファイルを使用して、必要なすべてのソフトウェアを `snakemake-tutorial` という名前の分離されたConda環境にインストールできます。

```
$ conda env create --name snakemake-tutorial --file environment.yaml
```

## Step 4: Activating the environment
To activate the `snakemake-tutorial` environment, execute

`snakemake-tutorial` 環境をアクティブにするには、下記を実行します

```
$ conda activate snakemake-tutorial
```

Now you can use the installed tools. Execute

これで、インストールされたツールを使用できます。 次に下記を実行します。

```
$ snakemake --help
```

to test this and get information about the command-line interface of Snakemake.
To exit the environment, you can execute

すると、Snakemakeのコマンドラインインターフェイスに関する情報を取得します。
環境から抜けるのは、下記でできます

```
$ conda deactivate
```

but don’t do that now, since we finally want to start working with Snakemake :-).

しかし、私たちはこれからいよいよSnakemakeを使い始ようとするわけなので、今はそうしないでください :-)。
