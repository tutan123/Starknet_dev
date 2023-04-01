# 将你的第一个 Cairo 1 合约部署到 Starknet

随着[0.11 版](https://starkware.medium.com/starknet-alpha-v0-11-0-the-transition-to-cairo-1-0-begins-30442d494515)在 Starknet 上的推出，您现在可以在 Starknet 上部署您的第一个 Cairo 1 合约！极好的！

... 但是怎么办？

在我们整理文档之前，这是一个非常粗略的指南

## 流量

您将需要：

* 在特定高度克隆 Cairo 存储库（编译 Cairo 1 合约的工具）
* 安装最新版本的 Cairo-lang（与 Starknet 交互的工具）
* 使用 cairo 将你的合约从 Cairo 编译到 Sierra
* 使用 cairo-lang 声明你的 Sierra 合约
* 使用 cairo-lang 部署你的合约
* 使用 Twitter 向您的朋友吹嘘

## 克隆这个仓库

```shell
git clone https://github.com/starknet-edu/deploy-cairo1-demo
cd deploy-cairo1-demo
```

## 安装 Cairo one 编译器

如果您是第一次安装 Cairo：

```shell
git clone https://github.com/starkware-libs/cairo/
cd cairo
git checkout 9c190561ce1e8323665857f1a77082925c817b4c
cargo build --all --release
```

如果您已经安装了 Cairo：

```shell
cd cairo
git fetch && git pull
git checkout 9c190561ce1e8323665857f1a77082925c817b4c
cargo build --all --release
```

此时，您已在此存储库中安装了 Cairo。

## 安装 Cairo-lang

返回回购的根文件夹

```shell
cd ..
```

搭建python虚拟环境

```shell
python3.9 -m venv ~/cairo_venv_v11
source ~/cairo_venv_v11/bin/activate
```

如果您之前安装了 cairo-lang，请将其卸载

```shell
pip3 uninstall cairo-lang
```

安装开罗语言

```shell
pip3 install ecdsa fastecdsa sympy
pip3 install cairo-lang
```

检查您是否已正确安装

```shell
starknet --version
```

## 使用 Cairo 编译你的合约

您可以使用[这个小的演示合约](https://github.com/starknet-edu/deploy-cairo1-demo/blob/master/hello_starknet.cairo)来完成这个测试。这是一个非常基本的事件记录器。

演示合约代码如下：

```python
#[contract]
mod HelloStarknet {
    use starknet::get_caller_address;
    use starknet::ContractAddress;


    #[event]
    fn Hello(from: ContractAddress, value: felt252) {}


    #[external]
    fn Say_Hello(message: felt252) {
        let caller = get_caller_address();
        Hello(caller, message);
    }

}
```

从您的终端，在您安装 Cairo 的文件夹中

```shell
cd cairo
cargo run --bin starknet-compile -- ../hello_starknet.cairo ../hello_starknet.json --replace-ids
```

恭喜，您已经编制了从开罗到塞拉利昂的合同！

## 使用 Cairo-lang 声明你的合同

### 设置环境变量

```shell
export STARKNET_NETWORK=alpha-goerli
export STARKNET_WALLET=starkware.starknet.wallets.open_zeppelin.OpenZeppelinAccount
```

### 为教程设置一个新帐户

您需要确保您的 starknet CLI 设置了适当的账户合约和资金。为了安全起见，我将为本教程创建一个新的。

```shell
starknet new_account --account version_11
```

* 你应该得到你期望的合约地址
* 发送 0.1 ETH 给它
* 监控转账交易。一旦通过“pending”，继续

```shell
starknet deploy_account --account version_11
```

监控部署事务。一旦通过“pending”，继续

### 声明合同

返回教程的根文件夹，然后尝试声明

```shell
cd ..
starknet declare --contract hello_starknet.json --account version_11
```

您应该收到新声明的类哈希！

### 故障排除

好的，所以我遇到了问题。使用 declare 发回：

```shell
Error: OSError: [Errno 8] Exec format error: '~/cairo_venv_11/lib/python3.9/site-packages/starkware/starknet/compiler/v1/bin/starknet-sierra-compile'
```

原因是 cairo-lang 需要编译你的 sierra 代码，它为此使用了导入的 rust 二进制文件“starknet-sierra-compile”。但就我而言，导入的二进制文件不是为我的架构构建的:-(。

所以，很简单：我从本地构建的 cairo 存储库中获取了“starknet-sierra-compile”，并将其替换为 python 包。这是一个应该起作用的命令：

```shell
cp cairo/target/release/starknet-sierra-compile ~/cairo_venv_v11/lib/python3.9/site-packages/starkware/starknet/compiler/v1/bin/starknet-sierra-compile
```

然后再次尝试声明。它应该工作！

## 使用 Cairo-lang 部署你的合约

此时，您又回到了习惯的流程中。使用您之前收到的类哈希：

```shell
starknet deploy --class_hash <class_hash> --account version_11
```

监控您的交易。如果由于费用估算而失败，请重试。一旦您的交易被 accepted_on_l2.... 恭喜！你已经部署了你的第一个 Cairo 1 合约！

相关官方视频教程：https://www.youtube.com/watch?v=JB53QUFJGvA
