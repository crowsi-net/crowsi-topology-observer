# Crowsi Topology Observer

ZixcelのローカルTopologyを、Crowsi認可とレスポンス真正性の両方が成立した
場合だけCoelaへ渡すpurpose-specific Rust boundaryです。汎用HTTP proxy、
Workload自己申告、同一UIDの鍵所有を認証として扱いません。

## Observation chain

1. `serve-one`がowner-only Unix socketを作り、1接続だけ待ちます。
2. current-only deployment v3がpin留めしたPAとは別のiHAT status鍵・issuer・固定
   audience `crowsi://topology-observer/current-status`・service `service:crowsi`と、
   pairwise Subject・Device・proof key・service-scoped opaque `session_ref`へ一致する30秒以下のowner-only
   `CurrentDeviceStatusV1`をdurableに適用します。未配備・replay・別端末はsocket公開前に拒否します。
3. clientとserverは`SO_PEERCRED`、PID start time、双方の実行ファイルSHA-256を
   確認します。
4. Local Control BridgeがV2 PA署名、`service:crowsi` pairwise subject、端末固有
   proof key、posture revision、subject/service/device/session revocation epoch、
   sender proof、SPIFFE workload、user verification、Action、Resource、Purpose、
   operation body SHAと一回利用reservationを同一stream上で検証・durable消費します。
5. operation bodyに含まれる32-byte nonceを使い、固定されたZixcel loopback URLへ
   GETを行います。
6. Ed25519 response proofのURL、nonce、status、body SHA、security domain、
   deployment、trust revision、期限を確認し、response nonceを別のprivate SQLite
   でdurable消費します。
7. Crowsi evidenceとZixcel response-authenticity evidenceを分離したclosed receipt
   だけを同じUnix streamへ返します。

## Commands

```bash
cargo run --offline -- sample-readiness
cargo run --offline -- serve-one /absolute/private/deployment.json
cargo run --offline -- observe /absolute/private/observer.sock \
  /absolute/private/one-use-observation-bundle.json
```

Coelaは前二つのruntime commandを短命processとして組み合わせます。bundleはPAが
発行した60秒以下・one-useのauthorizationを含み、Webやbrowserへ渡しません。
`serve-one`は1回の成功または失敗で終了し、socket inodeも回収します。
library/CLIはowner-only local policyを開発検証用に読めますが、Coela native
adapterはroot所有policyとroot所有binaryのdigest一致を必須とします。したがって
同一UIDが任意のpolicyとbinaryを置くだけではconnectedへ昇格できません。

## Verification

```bash
# WONDERLAND_ROOT is the workspace checkout root.
"$WONDERLAND_ROOT/bin/verify-repositories" --rust --tier standard
node ${WONDERLAND_ROOT}/tools/check-source-layout.mjs .
```

socketを使うintegration testは実行環境でUnix/loopback socket作成権限が必要です。

## Deployment boundary

repositoryは検証経路を実装しますが、PAのper-use発行、専用非root UID、
root管理の親配下へprovisionされたservice専用0700 socket directory、
署名済みbinary配布、TPM/HSM-backed trust、
rollback-resistant trusted clockをprovisionしません。これらがない環境を
production readyとは判定せず、`sample-readiness`は常に`unavailable`です。

## Response audience

The owner-provisioned response trust document supplies the exact application
audience. The observer validates it as a bounded identifier, sends it in the
request header, and requires the signed response to match. Request callers
cannot override that audience. Existing trust documents retain their configured
audience; this change neither generates keys nor changes a deployed service.
