# Convallax Bot

**Versi:** 1.1.0 · **Harga:** Rp 100.000 (~$6) · **Kontak pembelian:** [@rmndkyl](https://t.me/rmndkyl)

Volume farming otomatis untuk [Convallax](https://www.convallax.com/predict) — options exchange di prediction market (Polymarket YES outcomes), settle on-chain di Polygon Amoy.

## Kebutuhan

| Item | Detail |
|---|---|
| Runtime | Node.js 18+ |
| Wallet | EVM / sesuai chain proyek (kamu sendiri) |
| Proxy | tidak perlu |
| Captcha | tidak perlu |
| Akses | akun proyek yang kamu kelola sendiri |

## Pembayaran

E-Wallet: DANA / GoPay / ShopeePay · Bank: BCA / Seabank · Crypto: contact for wallet address.
Pembeli menghubungi [@rmndkyl](https://t.me/rmndkyl), kirim bukti transfer, lalu terima license key.

---

Bot beli/tulis call & put lewat RFQ (request-for-quote) market maker, lalu settle satu transaksi `fill()`.

## Yang berubah di v1.1 (14 Nov 2026)

- **API v3**: host pindah ke `api.convallax.com` (`/v1/markets`, `/v1/series`, `/quote-requests`, `/faucet`).
- **Fix "No valid quotes available"** — dua penyebab nyata:
  1. bot memilih series yang sudah **kedaluwarsa** (117 dari 253 series `settled=false` ternyata expiry-nya lewat) → sekarang filter `expiry > now + 48 jam`;
  2. payload quote **tidak mengirim `market.yesPrice`** → market maker tidak pernah menjawab. Setelah `yesPrice` dikirim, quote masuk di attempt pertama (uji 32 request: 31 dapat quote).
- **Humanization** (anti-sybil): ukuran tidak bulat, jeda tidak tetap, market/expiry/strike bervariasi, sesekali skip, kuota harian per wallet.
- **Kontrak on-chain diperbarui** sesuai docs (alamat lama di README v1.0 sudah tidak dipakai).
- `/positions` dan `/trades/confirm` dihapus — endpoint itu tidak ada di API (404). Posisi dibaca dari ERC-1155 on-chain.
- `--status` (read-only), `--once`, `--side buy|sell`, `--dry-run`, `--trades N`, `--fast` (khusus uji).

## Bukti run (bukan mockup)

`node index.js --once` → quote PUT 80% @0.6964, 11 kontrak ($7.66), fill:

`--daily --trades 2` (2 wallet, humanize aktif) → 4 fill tambahan:

Sell side (`--side sell`) → premium diterima, collateral terkunci:

Cek di [amoy.polygonscan.com](https://amoy.polygonscan.com).

## Chain & kontrak

| Item | Nilai |
|---|---|
| Network | Polygon Amoy (chain ID `80002`) |
| RPC default | `https://polygon-amoy-bor-rpc.publicnode.com` |
| Explorer | https://amoy.polygonscan.com |

Alamat di atas dicek ulang 14 Nov 2026: bytecode ada di chain, dan docs.convallax.com/guides/testnet memakai alamat yang sama.

## Requirement

- Node.js 18+ (bot diuji di Node 22)
- Wallet EVM dengan POL Amoy untuk gas (≈0.02 POL cukup untuk beberapa fill; 1 fill ≈ 203k gas)
- Test USDC dari faucet Convallax (1000 USDC / wallet / 24 jam)
- Tidak butuh API key — taker authorized saat `fill()` on-chain

## Setup

```bash
npm install
```

2. Sesuaikan `config.json` bila perlu (budget, jeda, kuota). Default sudah aman untuk testnet.
3. Cek dulu tanpa transaksi:

```bash
node index.js --status
```

## Perintah

```bash
node index.js                 # menu interaktif
node index.js --status        # read-only: faucet, series aktif, saldo, kuota hari ini
node index.js --faucet        # klaim 1000 USDC semua wallet (cooldown 24 jam)
node index.js --once          # 1 trade wallet pertama (proof run)
node index.js --daily         # rutin harian semua wallet (faucet + trading, humanize)
node index.js --daily --trades 6
node index.js --side sell     # paksa sisi sell (menulis opsi, collateral USDC terkunci)
node index.js --dry-run       # sampai quote saja, tidak commit/fill
node index.js --fast          # jeda pendek, hanya untuk uji
```

## Humanization (anti-sybil)

| Perilaku | Nilai default |
|---|---|
| Budget beli | acak $3–$12, 2 desimal, hindari angka bulat |
| Size jual | 10–40 kontrak (acak, tidak seragam) |
| Sisi trade | 85% buy, 15% sell |
| Jeda antar trade | 45–210 detik |
| Jeda antar wallet | 2–7 menit |
| Jitter mulai sesi | 0–15 menit |
| Skip (ragu) | 8% per trade |
| Kuota harian | 12 trade/wallet (state di `data/state.json`) |
| Urutan wallet | diacak tiap run |
| Pilihan market | disebar merata (bukan menumpuk di 1 market) |

Semua bisa diubah di blok `humanize` dan `trade` pada `config.json`.

## Alur trade

1. `GET /v1/series?settled=false` → filter expiry > 48 jam dan punya harga YES.
2. `GET /api/markets` (proxy frontend) → `yesPrice` per conditionId.
3. `POST /quote-requests` (market + option + trade) → `requestId`, `makersConnected`.
4. Poll `GET /quote-requests/:id/quotes` tiap ~1.5s sampai `bestQuote` atau request ditutup.
5. `POST /quote-requests/:id/commit` → order EIP-712 + signature maker.
6. `approve()` USDC ke spender dari `takerApproval` (kalau allowance kurang).
7. `fill(order, makerSignature)` → premium dibayar, collateral terkunci, opsi di-mint (ERC-1155), satu tx.
8. Catat tx ke `data/state.json`.

Kalau maker tidak menjawab dalam ~12 detik, bot pindah kandidat (maks 4 attempt, beda market).

## Troubleshooting

| Masalah | Penyebab / solusi |
|---|---|
| `Tidak ada quote dari market maker` | Series jarang di-quote. Bot otomatis pindah kandidat. Pastikan `yesPrice` terisi (bot ambil dari `/api/markets`). |
| `HTTP 503` saat faucet | Relay faucet belum dikonfigurasi di sisi Convallax — coba lagi nanti, atau pakai wallet yang sudah punya USDC. |
| `429` saat faucet | Cooldown 24 jam per wallet. Bukan error. |
| `Low POL` | Isi POL Amoy (faucet Polygon). 1 fill ≈ 0.002–0.004 POL. |
| `fill reverted` | Order `validUntil` sudah lewat (signature kedaluwarsa) — jalankan ulang; bot commit baru. |
| RPC error / timeout | Ganti `rpcUrl` di `config.json` (mis. `https://polygon-amoy.drpc.org`). |

## Struktur

```
index.js            # CLI + alur trade
src/api.js          # klien RFQ API v3 + harga YES dari proxy
src/trader.js       # ethers: approve, fill, baca series/posisi
src/humanize.js     # ukuran/jeda acak + state harian per wallet
config.json         # parameter trade & humanize
data/state.json     # kuota + riwayat tx hari ini (tidak di-commit)
```

## Harga (LYID)

| Lisensi | Harga | Fitur |
|---|---|---|
| Single | Rp 100.000 | 1 device, lifetime |
| Bundle (3 bot) | Rp 100.000 | pilih 3 bot |
| Pro (semua bot) | Rp 100.000 | semua bot + update |

Beli: [@OG_LYID_bot](https://t.me/OG_LYID_bot) → `/buy` · Bayar: DANA/GoPay/ShopeePay/BCA/Seabank atau crypto.

## Catatan status project (14 Nov 2026)

- Testnet masih jalan dan masih **belum ada pengumuman token/TGE/airdrop resmi**.
- Backing: Alliance (ALL17). Audit kontrak: Sherlock. Docs: https://docs.convallax.com.
- Quest Zealy lama ($2.500 prize pool) sudah tidak ditemukan di `zealy.io/cw/convallax` — jangan jadikan patokan.
- Halaman pihak ketiga (Bitget dll) yang menyebut TGE "late 2024/early 2025" itu data lama/tidak valid.

## Contact

@rmndkyl · [@layerairdrop](https://t.me/layerairdrop) · [Discord LYID](https://discord.gg/h4mrYDdq9)
