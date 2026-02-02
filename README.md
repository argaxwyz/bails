/**
 * Modified Whatsapp-API
 * Bail By Rixzz – FIXED VERSION (2026 Update)
 * Repo: github:ArgaXwyz7Six/Xwyz7ap
 */

const {
  default: makeWASocket,
  fetchLatestBaileysVersion,  // Lebih akurat daripada fetchLatestWAWebVersion
  useMultiFileAuthState,
  DisconnectReason,
  Browsers  // Optional: buat browser config lebih clean
} = require('@whiskeysockets/baileys');

const fs = require('fs');
const pino = require('pino');

/* =======================
   CONNECT (QR / PAIRING) + Auto Reconnect
======================= */
async function connectWA({ pairing = false, number = null }) {
  const { state, saveCreds } = await useMultiFileAuthState('./session');
  
  const { version } = await fetchLatestBaileysVersion();  // Fetch version terbaru otomatis

  const client = makeWASocket({
    version,
    auth: state,
    logger: pino({ level: 'silent' }),  // Silent logger
    printQRInTerminal: !pairing,
    browser: Browsers.ubuntu('Chrome'),  // Lebih clean & recommended daripada array manual
    // syncFullHistory: false,  // Uncomment kalau ga mau sync history full (hemat data)
  });

  if (pairing && number && !client.authState.creds.registered) {
    const code = await client.requestPairingCode(number.trim());
    console.log('Your pairing code:', code);
  }

  client.ev.on('creds.update', saveCreds);

  client.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect } = update;
    if (connection === 'close') {
      const shouldReconnect = lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut;
      console.log('Connection closed due to', lastDisconnect?.error, ', reconnecting?', shouldReconnect);
      if (shouldReconnect) {
        connectWA({ pairing, number });  // Reconnect recursive
      }
    } else if (connection === 'open') {
      console.log('WhatsApp Connected Successfully!');
    }
  });

  return client;
}

/* =======================
   SEND ORDER MESSAGE (Fixed Structure)
======================= */
async function sendOrderMessage(client, m) {
  const thumb = fs.readFileSync('./ZeppImage.jpg');  // Pastikan file ada & path benar

  await client.sendMessage(m.chat, {
    orderMessage: {
      itemCount: 1,
      status: 'INQUIRY',  // Atau 'ACCEPTED' / 'DECLINED' tergantung status
      surface: 'CATALOG',
      message: "Gotta get a grip",
      orderTitle: "PongKangColi",
      sellerJid: m.chat,  // Seller = chat sendiri kalau self-order
      thumbnail: thumb,
      totalAmount1000: 72502,
      totalCurrencyCode: "IDR"
    }
  }, { quoted: m });
}

/* =======================
   SEND POLL RESULT SNAPSHOT (Updated Structure)
======================= */
async function sendPollResult(client, m) {
  await client.sendMessage(m.chat, {
    pollResultSnapshotMessage: {
      name: "n",
      options: [
        { optionName: "poll 1", voteCount: 10 },
        { optionName: "poll 2", voteCount: 5 }
      ],
      totalVotes: 15  // Tambah ini biar lebih akurat (total vote count)
    }
  });
}

/* =======================
   SEND PRODUCT MESSAGE (Fixed with relayMessage structure)
======================= */
async function sendProductMessage(client, m) {
  const thumb = fs.readFileSync('./ZeppImage.jpg');

  await client.relayMessage(m.chat, {
    productMessage: {
      product: {
        productImage: {
          mimetype: "image/jpeg",
          jpegThumbnail: thumb
        },
        title: "DewaBail",
        description: "zZZ...",
        priceAmount1000: 72502,
        currencyCode: "IDR",
        retailerId: "EXAMPLE_RETAILER_ID",
        productId: "EXAMPLE_TOKEN",
        url: "https://t.me/RixzzNotDev"
      },
      businessOwnerJid: m.chat,
      body: "Nak Tido",
      footer: "Footer",
      buttons: [
        {
          buttonId: "cta_url",
          buttonText: { displayText: "7eppeli-Pdf" },
          type: 1
        }
      ]
    }
  }, {});
}

/* =======================
   EXPORT
======================= */
module.exports = {
  connectWA,
  sendOrderMessage,
  sendPollResult,
  sendProductMessage
};
