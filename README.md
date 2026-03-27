STM32 Nucleo-H743ZI2ボードを使用し、FreeRTOS + LwIP上でmicro-ROS（UDP通信）を動作させるプロジェクトです。

### 構成
- **Board:** NUCLEO-H743ZI2
- **Framework:** STM32CubeIDE / FreeRTOS / LwIP
- **ROS 2 Version:** Humble
- **Transport:** UDP (Ethernet)

### システム構成図
`[STM32 Nucleo-H743ZI2 (Client)] <--- LAN Cable ---> [Ubuntu PC (Agent)]`

- **STM32 IP:** `192.168.1.10`
- **Ubuntu IP:** `192.168.1.100` (設定により適宜変更)
- **Port:** `8888`

### セットアップ手順

#### 1. 環境
- Ubuntu 22.04 lts (ROS 2 Humble インストール済み)
- STM32CubeMX
- STM32CubeIDE
- Docker (micro-ROS静的ライブラリビルドおよびAgent用)

#### 2. リポジトリの準備
```bash
git clone https://github.com/kai-1208/micro-ros_h743zi2_omni-wheel.git
cd micro-ros_h743zi2_omni-wheel
```

#### 3. micro-ROS 静的ライブラリの生成
本プロジェクトは外部ユーティリティ [micro_ros_stm32cubemx_utils](https://github.com/micro-ROS/micro_ros_stm32cubemx_utils) を使用しています。以下のコマンドでライブラリ（`libmicroros.a`）を手動生成してください。

```bash
cd micro_ros_stm32cubemx_utils
sudo docker run -it --rm -v $(pwd)/..:/project --env MICROROS_LIBRARY_FOLDER=micro_ros_stm32cubemx_utils/microros_static_library_ide microros/micro_ros_static_library_builder:humble
```
※完了後、`microros_static_library_ide` フォルダ内にライブラリが生成されます。

#### 4. STM32CubeIDE の設定確認
プロジェクトを開き、以下の設定が反映されていることを確認してください。

- **Linker Script (`STM32H743ZITX_FLASH.ld`):**
  - イーサネットDMAバッファを SRAM3 (`0x30040000`) に配置する設定が末尾に記述されていること。
- **Library Path:**
  - `MCU GCC Linker -> Libraries` に `microros` が追加されていること。
  - `Library search path` に上記ライブラリフォルダが指定されていること。

### 実行方法

#### 1. micro-ROS Agent の起動 (Ubuntu側)
Dockerを使用してAgentを立ち上げ、マイコンからの接続を待ち受けます。

```bash
sudo docker run -it --rm --net=host microros/micro-ros-agent:humble udp4 --port 8888 -v6
```

#### 2. プログラムの書き込みと実行
STM32CubeIDEからマイコンにプログラムをビルド・書き込みます。
起動後、Nucleoのリセットボタンを押すとAgentとのセッションが確立されます。

#### 3. トピックの確認
新しいターミナルで以下を実行します。
```bash
ros2 topic list
ros2 topic echo /cubemx_publisher
```

### STM32H7 特有の重要設定 (Troubleshooting)
STM32H7でEthernet通信を安定させるため、以下の対策を適用済みです。

1. **D-Cache の無効化:** DMA通信とキャッシュの整合性問題を避けるため、CPUのデータキャッシュを無効化しています。
2. **NVIC 優先度:** Ethernet割り込みの優先度を `5` に設定（FreeRTOSのAPIを割り込み内で使用するため）。
3. **SRAM3 の利用:** Linker Scriptを修正し、Ethernet記述子およびバッファをDMAアクセス可能なSRAM3領域に強制配置しています。
4. **Stack Size:** micro-ROSタスクのスタックサイズを `3000 words (12KB)` 以上に確保しています。

### micro-ROSの[README](https://github.com/micro-ROS/micro_ros_stm32cubemx_utils/tree/humble?tab=readme-ov-file#Transport-configuration)みてえ
### stmのサンプルコードの[README](https://github.com/stm32-hotspot/STM32H7-LwIP-Examples/tree/main)もみてえ

### ライセンス
[Apache License 2.0](LICENSE)
