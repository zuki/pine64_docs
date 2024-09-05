# bootd コマンド

## 書式

```bash
bootd
```

## 説明

bootdコマンドは環境変数`bootcmd`に格納されているコマンドを実行します。
すなわち`run bootmcd`と同じことを行います。

## 使用例

```bash
=> setenv bootcmd 'echo Hello World'
=> bootd
Hello World
=> setenv bootcmd true
=> bootd; echo $?
0
=> setenv bootcmd false
=> bootd; echo $?
1
```

## 返り値

bootdコマンドの返り値`$?`は環境変数`bootcmd`にあるコマンドの返り値です。
