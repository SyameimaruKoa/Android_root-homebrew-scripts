# Android-root_termux_script

Android（主に Termux や root 環境）で使う、シンプルなシェルスクリプト集です。ネットワーク監視やシステム確認、Tailscale の疎通チェックなど、日常運用でよく使う小物をまとめています。

この README では、収録スクリプトの目的・使い方・依存関係を短くまとめています。なお、bmon/btop/Tailscale など su（root）で実行するコマンドは、Magisk のモジュールとしてシステムに直接導入している前提です。

> 重要: このリポジトリは「完全に自分用に保存しているスクリプト」をパブリックにしているだけです。一般向けのドキュメント整備やサポートは基本的に行いません。内容は予告なく変更・破棄される可能性があります。

## 目次

- 準備（実行権限・インストール）
- スクリプト一覧
  - bmon.sh（ネットワーク帯域モニタ）
  - btop.sh（プロセス/リソースモニタ）
  - ip.sh（IP/IF 情報の確認）
  - tailscale_ping_loop.sh（Tailscale Ping ループ監視）
  - tailscale_status_loop.sh（Tailscale ステータス監視）
  - tailscale-device.sh（Tailscale デバイス情報）
- 注意事項
- 貢献・ライセンス

---

## 準備（実行権限・インストール）

- 実行権限を付与してください。
  - 例: `chmod +x ./bmon.sh`（他の `.sh` も同様）
- すべてのスクリプトは `su` コマンドを使用してルート権限で実行します。
- bmon / btop / Tailscale は Magisk モジュールでシステムに直接インストールしている想定です。
  - そのため、これらについては Termux パッケージの導入は基本的に不要です。
  - **重要**: Termux の `pkg install` でインストールしたパッケージは、`su -c` では実行できません。Termux パッケージは Termux 環境内でのみ動作します。
  - Termux のみで完結させたい場合は、スクリプトから `su -c` を削除し、Termux 環境内で直接実行するように修正してください。
- ネットワーク系の補助ツールが必要な場合のみ、Termux で追加パッケージを導入してください。
  - 例: `pkg install iproute2`（`ip` コマンド用）

---

## スクリプト一覧

### bmon.sh（ネットワーク帯域モニタ）

- 目的: ネットワーク帯域のリアルタイム監視ツール bmon を手軽に起動するためのラッパー。
- 使い方:
  - `./bmon.sh`
- 依存関係: `bmon`（Magisk モジュールで導入済みを想定）
  - 注意: `pkg install bmon` でインストールした場合は `su -c` では動作しません。
- 実行内容: `su -c 'bmon -b'`

### btop.sh（プロセス/リソースモニタ）

- 目的: CPU/メモリ/プロセス/ネットワーク等を見やすく表示する btop を起動。
- 使い方:
  - `./btop.sh`
- 依存関係: `btop`（Magisk モジュールで導入済みを想定）
  - 注意: `pkg install btop` でインストールした場合は `su -c` では動作しません。
- 実行内容: `su -c 'btop'`

### ip.sh（IP/IF 情報の確認）

- 目的: ネットワークインターフェースや IPv4 情報を簡単に確認。
- 使い方:
  - `./ip.sh`
- 依存関係: `ip`（iproute2）
- 実行内容: `su -c 'ip -4 a'`

### tailscale_ping_loop.sh（Tailscale Ping ループ監視）

- 目的: Tailscale デバイス一覧を表示し、指定したデバイスに対して1秒おきに ping を実行し続け、疎通を監視。
- 使い方:
  1. `./tailscale_ping_loop.sh`
  2. デバイス一覧が表示されるので、Ping を送りたいデバイスの IP アドレスを入力
  3. Ctrl+C で停止
- 依存関係: `tailscale`（Magisk モジュールで導入済みを想定）
- 特徴:
  - デバイス一覧を自動表示
  - 1秒間隔で `tailscale ping` を実行
  - 時刻付きで pong/timeout を表示
  - `su` で一度だけルート権限を取得し、ループ全体を実行
- 注意:
  - 長時間実行でログが肥大化したりバッテリー消費が増える可能性があります。

### tailscale_status_loop.sh（Tailscale ステータス監視）

- 目的: Tailscale のステータスを2秒おきに更新表示し、ネットワークの状態を監視。
- 使い方:
  - `./tailscale_status_loop.sh`
  - Ctrl+C で停止
- 依存関係: `tailscale`（Magisk モジュールで導入済みを想定）
- 特徴:
  - ちらつき対策: カーソル位置制御（`tput cup 0 0`）により画面をクリアせず上書き表示
  - 2秒間隔で `tailscale status` を自動更新
  - 時刻付きで表示
  - `su` で一度だけルート権限を取得し、ループ全体を実行

### tailscale-device.sh（Tailscale デバイス情報）

- 目的: Tailscale のデバイス一覧や状態確認。
- 使い方:
  - `./tailscale-device.sh`
  - Enter キーを押すと終了
- 依存関係: `tailscale`（Magisk モジュールで導入済みを想定）
- 実行内容: `su -c 'tailscale status'` を実行後、入力待ち

---

## 注意事項

- このリポジトリは個人利用のために保存しているものを公開しているだけです。互換性や安定性、サポートは保証しません。
- お使いの端末・ROM・カーネル・Termux バージョンにより挙動が異なります。
- すべてのスクリプトはルート権限（`su`）で実行されます。取り扱いに注意してください。自己責任でご利用ください。
- ネットワーク関連のツールは権限や SELinux 設定の影響を受ける場合があります。

## 貢献・ライセンス

- バグ報告や改善提案は Issue / Pull Request にて歓迎します。(やる気があれば)
