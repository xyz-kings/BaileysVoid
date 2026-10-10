# BaileysVoid

Fork [Baileys](https://github.com/WhiskeySockets/Baileys) (WhatsApp Web API, multi-device) yang sudah dibersihkan dan siap dipakai untuk bot.

> Library ini tidak berafiliasi dengan WhatsApp / Meta. Gunakan dengan bijak dan risiko ditanggung sendiri.

## Perubahan dari fork sebelumnya

- Dihapus: request tersembunyi di `requestPairingCode` yang mengirim nomor WhatsApp ke server pihak ketiga.
- Pairing code default: `XYZVOIDS` (bisa diganti lewat parameter, lihat di bawah).
- Fix syntax error di `lib/Socket/messages-send.js` yang membuat library gagal di-load.
- Dibersihkan: `devDependencies` berat (`sharp`, `eslint`, `jest`, dll) dan script `prepare: tsc`, jadi `npm install` aman di Termux.
- `jimp` jadi dependency (pure JS) sebagai pengganti `sharp` untuk thumbnail.

## Install

Butuh Node.js >= 18 dan `git` (dependency `libsignal` diambil dari GitHub).

```bash
npm install github:xyz-kings/BaileysVoid
```

atau di `package.json`:

```json
"dependencies": {
  "baileys-void": "github:xyz-kings/BaileysVoid"
}
```

## Contoh singkat

```js
const {
  default: makeWASocket,
  useMultiFileAuthState,
  fetchLatestBaileysVersion,
  Browsers
} = require('baileys-void')
const pino = require('pino')

async function start () {
  const { state, saveCreds } = await useMultiFileAuthState('./session')
  const { version } = await fetchLatestBaileysVersion()

  const sock = makeWASocket({
    version,
    auth: state,
    logger: pino({ level: 'silent' }),
    printQRInTerminal: false,
    browser: Browsers.ubuntu('Chrome')
  })

  sock.ev.on('creds.update', saveCreds)

  if (!sock.authState.creds.registered) {
    await new Promise((r) => setTimeout(r, 3000))
    const code = await sock.requestPairingCode('628123456789')           // kode default: XYZVOIDS
    // const code = await sock.requestPairingCode('628123456789', 'ABCD1234') // kode custom
    console.log('Kode pairing:', code)
  }

  sock.ev.on('messages.upsert', ({ messages }) => {
    for (const m of messages) console.log(m.key.remoteJid, m.message)
  })
}

start()
```

## Pairing code

```js
requestPairingCode(phoneNumber, customPairingCode?)
```

- `phoneNumber`: format internasional tanpa `+` (contoh `628123456789`).
- `customPairingCode`: opsional, tepat 8 karakter `A-Z` / `0-9`. Kalau tidak diisi, dipakai kode default `XYZVOIDS`.

Kode yang tetap hanya berlaku saat kamu sedang melakukan request pairing. Kalau bot dipakai banyak orang, sebaiknya kirim kode random lewat parameter.

## Lisensi

MIT, mengikuti proyek Baileys asal. Lihat file [LICENSE](./LICENSE).
