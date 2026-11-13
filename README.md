# Convallax Testnet Bot v1.1

Options volume farming untuk [Convallax](https://www.convallax.com/predict) — options exchange di prediction market (Polymarket YES outcomes), settle on-chain di Polygon Amoy.

Bot membuka RFQ ke market maker, commit ke quote terbaik, lalu mengeksekusi `fill()` satu transaksi: premium dibayar, collateral terkunci, option token (ERC-1155) di-mint.

> Source code private (produk LYID). Repo ini hanya dokumentasi & bukti run.

## Bukti run on-chain (14 Nov 2026)

Fill nyata dari bot, bukan mockup:

- `0xd60e895c269482085c409ca850a7402eecfceb38118ed3f40ef9cddd113eac80` — PUT 80% @0.6964, 11 kontrak ($7.66), gas 203.243
- `0x841d26c2cb38363eb0ebf84511b908dd6fab7e655d72818537aca00c175241dd` — PUT 20%, 67 kontrak ($11.61)
- `0x86fb44da0d898f709ba0e590148b300afad2b42d0e7caa6696c472e62666cb8c` — PUT 40%, 18 kontrak ($6.27)
- `0x853123aed63912218fb98b5b4aa06bffb69318d446abf191fb72f96eba3047f1` — PUT 40%, 15 kontrak ($4.84)
- `0x0f478ef76926180aa5d7483a418b52cee2f073b053ea61f0237fcd78d8b51026` — buy di "Stripe acquire Paypal"
- `0x2a212afda7ef0c93b7cf81120b9744392d298774e47874a1b59f720f8c80f814` — sisi **sell** 16 kontrak, premium diterima

Cek di [amoy.polygonscan.com](https://amoy.polygonscan.com).

## Highlights v1.1

- **API v3** (`api.convallax.com`): `/v1/markets`, `/v1/series`, `/quote-requests`, `/faucet`.
- **Fix quote kosong**: filter series kedaluwarsa (expiry > 48 jam) + kirim `market.yesPrice`. Sebelum fix: 0/14 request dapat quote. Sesudah: 31/32 (uji matriks).
- **Humanization anti-sybil**: budget acak non-bulat ($3–12), jeda 45–210s antar trade, 2–7 menit antar wallet, jitter sesi 0–15 menit, skip 8%, kuota 12 trade/wallet/hari, market disebar merata, 85% buy / 15% sell.
- **Multi-wallet** dari satu `privateKey.txt`.
- **Sell side** (menulis opsi, collateral USDC terkunci) dan **buy side**.
- `--status` read-only, `--once`, `--daily`, `--faucet`, `--side`, `--dry-run`.

## Fitur

| Fitur | Keterangan |
|---|---|
| Auto faucet | 1000 test USDC / wallet / 24 jam |
| Quote RFQ | buka request, poll quote, commit otomatis |
| On-chain fill | approve + `fill()` atomik |
| Multi-wallet | urutan diacak, jeda manusiawi |
| State harian | `data/state.json` (kuota + riwayat tx) |
| Retry pintar | pindah market/series kalau maker tidak menjawab |

## Chain & kontrak

| Item | Nilai |
|---|---|
| Network | Polygon Amoy (chain ID 80002) |
| RPC | `https://polygon-amoy-bor-rpc.publicnode.com` |
| Core | `0x76c41a03e0993F0261bB3B1949320cA693f76faE` |
| RFQ Settlement | `0x3E583BC44157c91F8c534372A3e91c371841107A` |
| Option Token (ERC-1155) | `0x68a45fFc7d5282665ffa01Ec04ff133a4384F7e1` |
| MockUSDC | `0xCc544B615B244Cf59A7cB6Cb7d7119ef73A485E5` |

## Status project (14 Nov 2026)

- Testnet masih live. **Belum ada pengumuman token/TGE/airdrop resmi.**
- Backing: Alliance (ALL17). Audit: Sherlock.
- Quest Zealy lama sudah tidak ditemukan di `zealy.io/cw/convallax`.
- Halaman pihak ketiga yang menyebut TGE 2024/2025 = data lama, tidak valid.

## Dapatkan bot

Bot tersedia sebagai produk LYID (Rp 100.000, lifetime, 1 device).

Chat [@OG_LYID_bot](https://t.me/OG_LYID_bot) → `/buy` · Info: [@layerairdrop](https://t.me/layerairdrop)

## Contact

@rmndkyl · [@layerairdrop](https://t.me/layerairdrop) · [Discord LYID](https://discord.gg/h4mrYDdq9)
