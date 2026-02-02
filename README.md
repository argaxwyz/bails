/**
 * Modified Whatsapp-API
 * Bail By Rixzz – FIXED VERSION (2026 Update)
 * Repo: https://github.com/ArgaXwyz/7Six
 */

const {
  default: makeWASocket,
  fetchLatestBaileysVersion,
  useMultiFileAuthState,
  DisconnectReason,
  Browsers
} = require('@whiskeysockets/baileys')

const pino = require('pino')

/* =======================
   CONNECT (QR / PAIRING) + Auto Reconnect
======================= */
async function connectWA({ pairing = false, number = null }) {
  const { state, saveCreds } = await useMultiFileAuthState('./session')
  const { version } = await fetchLatestBaileysVersion()

  const client = makeWASocket({
    version,
    auth: state,
    logger: pino({ level: 'silent' }),
    printQRInTerminal: !pairing,
    browser: Browsers.ubuntu('Chrome')
  })

  if (pairing && number && !client.authState.creds.registered) {
    const code = await client.requestPairingCode(number.trim())
    console.log('Your pairing code:', code)
  }

  client.ev.on('creds.update', saveCreds)

  client.ev.on('connection.update', ({ connection, lastDisconnect }) => {
    if (connection === 'close') {
      const reconnect =
        lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut
      if (reconnect) {
        setTimeout(() => connectWA({ pairing, number }), 3000)
      }
    }
  })

  return client
}

/* =======================
   SEND ORDER MESSAGE (NO THUMBNAIL)
======================= */
async function sendOrderMessage(client, m) {
  await client.sendMessage(
    m.chat,
    {
      orderMessage: {
        itemCount: 1,
        status: 1,
        surface: 1,
        message: "Gotta get a grip",
        orderTitle: "PongKangColi",
        sellerJid: client.user.id,
        totalAmount1000: 72502,
        totalCurrencyCode: "IDR"
      }
    },
    { quoted: m }
  )
}

/* =======================
   SEND POLL RESULT SNAPSHOT (FIX)
======================= */
async function sendPollResult(client, m) {
  await client.relayMessage(
    m.chat,
    {
      pollResultSnapshotMessage: {
        name: "n",
        options: [
          { optionName: "poll 1", voteCount: 10 },
          { optionName: "poll 2", voteCount: 5 }
        ],
        totalVotes: 15
      }
    },
    {}
  )
}

/* =======================
   SEND PRODUCT MESSAGE (NO IMAGE, NO BUTTON)
======================= */
async function sendProductMessage(client, m) {
  await client.relayMessage(
    m.chat,
    {
      productMessage: {
        product: {
          title: "DewaBail",
          description: "zZZ...",
          priceAmount1000: 72502,
          currencyCode: "IDR",
          retailerId: "EXAMPLE_RETAILER_ID",
          productId: "EXAMPLE_TOKEN",
          url: "https://t.me/RixzzNotDev"
        },
        businessOwnerJid: client.user.id
      }
    },
    {}
  )
}

/* =======================
   EXPORT
======================= */
module.exports = {
  connectWA,
  sendOrderMessage,
  sendPollResult,
  sendProductMessage
}
