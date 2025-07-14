# 🚀 Baileys WhatsApp API - By HAMZAxLEGEND1

![GitHub stars](https://img.shields.io/github/stars/HAMZAxLEGEND1/baileys?style=social)
![Node.js](https://img.shields.io/badge/node-%3E%3D14.0-green)

Baileys WhatsApp API is a Node.js-based library for communicating with WhatsApp Web without the need for additional WebSockets. It is a modification of Whiskey Baileys to improve stability and support more message types. Developed with high performance in mind for bots, message automation, and integration with other WhatsApp applications. 

## 📌 About This Project
This repository is developed and maintained by **HAMZAxLEGEND1** and other open-source contributors. Community support and contributions are greatly appreciated! 💖

---

## ✨ Key Features

✅ **QR-free authentication using session authentication**
✅ **WhatsApp's latest Multi-Device (MD) support** 
✅ **Send and receive messages in various formats**
✅ **Manage groups (create groups, add/remove members, set descriptions, etc.)**
✅ **Event integration such as group entry/exit, message received, message read  **
---

## 📦 Installation

 Make sure **Node.js ≥ 14.0++** is installed. Then run the following command in the terminal:
 
```sh
npm install @HAMZAxLEGEND1/baileys
```

Atau dengan **Yarn**:

```sh
yarn add @HAMZAxLEGEND1/baileys
```

---

## 🚀 Basic Use of Pairing Code

```javascript
const { useMultiFileAuthState, makeWASocket } = require('@HAMZAxLEGEND1/baileys');
const readline = require('readline');

const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
const question = (q) => new Promise(res => rl.question(q, res));

async function start() {
  const { state, saveCreds } = await useMultiFileAuthState('./session');
  const usePairingCode = true;

  const conn = makeWASocket({
    auth: state,
    printQRInTerminal: !usePairingCode,
    keepAliveIntervalMs: 50000,
  });

  conn.ev.on('creds.update', saveCreds);

  if (usePairingCode && !conn.authState.creds.registered) {
    const phone = (await question('Enter Your Number Phone:\n')).replace(/\D/g, '');
    rl.close();
    try {
      const code = await conn.requestPairingCode(phone);
      console.log('Code Whatsapp:', code.match(/.{1,4}/g).join('-'));
    } catch (e) {
      console.log('Failed to get pairing code:', e.message);
    }
  }

  conn.ev.on('connection.update', ({ connection, lastDisconnect }) => {
    if (connection === 'close') {
      const reason = lastDisconnect?.error?.output?.statusCode || 'Unknown';
      console.log('Connection closed:', reason);
      if ([401, 405].includes(reason)) {
        console.log('Logged out, delete session folder to relogin');
      } else if ([515, 428].includes(reason)) {
        console.log('Restarting...');
        start();
      }
    } else if (connection === 'open') {
      console.log('Whatsapp connected');
    }
  });

  conn.ev.on('messages.upsert', async (m) => {
    if (m.type === 'notify') {
      const msg = m.messages[0];
      if (!msg.key.fromMe && msg.message) {
        await conn.sendMessage(msg.key.remoteJid, { text: 'Hello there!' });
      }
    }
  });
}

start();
```

---

## 📜 API Documentation

| Features.       | Description |
|---------------------|----------| | 
`sendMessage()` |
 Send text messages,
  images, videos, etc. | 
 | `updateProfile()` |
  Change profile photo and username |
   | `getChats()` |
    Get a list of user chats |
 | `groupParticipantsUpdate()` 
 | Add/remove group members |
  📖 **Full documentation:** [Baileys API Docs](https://github.com/adiwajshing/Baileys/wiki)

---

### 🪀 Sending All Interactive Msg 
```ts

await sock.sendMessage(m.chat, {
  text: "halo",
  title: "tes",
  footer: "© hamzaxlegend - 2025",
  interactiveButtons: [{
               "name": "single_select",
               "buttonParamsJson": "{\"title\":\"title\",\"sections\":[{\".menu\":\".play dj webito\",\"highlight_label\":\"label\",\"rows\":[{\"header\":\"header\",\"title\":\"title\",\"description\":\"description\",\"id\":\"id\"},{\"header\":\"header\",\"title\":\"title\",\"description\":\"description\",\"id\":\"id\"}]}]}"
             },
             {
               "name": "cta_reply",
               "buttonParamsJson": "{\"display_text\":\"quick_reply\",\"id\":\"message\"}"
             },
               {
                "name": "cta_url",
                "buttonParamsJson": "{\"display_text\":\"url\",\"url\":\"https://www.google.com\",\"merchant_url\":\"https://www.google.com\"}"
             },
             {
                "name": "cta_call",
                "buttonParamsJson": "{\"display_text\":\"call\",\"id\":\"message\"}"
             },
             {
                "name": "cta_copy",
                "buttonParamsJson": "{\"display_text\":\"copy\",\"id\":\"123456789\",\"copy_code\":\"message\"}"
             },
             {
                "name": "cta_reminder",
                "buttonParamsJson": "{\"display_text\":\"Recordatorio\",\"id\":\"message\"}"
             },
             {
                "name": "cta_cancel_reminder",
                "buttonParamsJson": "{\"display_text\":\"cta_cancel_reminder\",\"id\":\"message\"}"
             },
             {
                "name": "address_message",
                "buttonParamsJson": "{\"display_text\":\"address_message\",\"id\":\"message\"}"
             },
             {
                "name": "send_location",
                "buttonParamsJson": ""
             }]
}, { quoted: m });
```


### 🏓 Send Message Button 
```ts
sock.sendMessage(m.chat, {
     text: "Hello World !",
     footer: "Justhamzaxlegend",
     buttons: [ 
         { buttonId: `.play`,
          buttonText: {
              displayText: 'ini button'
          }, type: 1 }
     ],
     headerType: 1,
     viewOnce: true
 },{ quoted: null })
```

### 📢 Send Store Messages  
```ts
sock.sendMessage(msg.key.remoteJid, {
    text: "Isi Pesan",
    title: "Judul",
    subtitle: "Subjudul",
    footer: "Footer",
    viewOnce: true,
    shop: 3,
    id: "199872865193",
}, { quoted: m })
```

### 📊 Poll Results from Newsletter  
```ts
await sock.sendMessage(msg.key.remoteJid, {
    pollResult: {
        name: "Judul Polling",
        votes: [
            ["Opsi 1", 10], 
            ["Opsi 2", 10]
        ],
    }
}, { quoted: m })
```

### 🏷️ Mention in Status  
```ts
await sock.StatusMentions({ text: "Halo!" }, [
    "123456789123456789@g.us",
    "123456789@s.whatsapp.net",
])
```

### 🃏 Order by Card 
```ts
await sock.sendMessage(msg.key.remoteJid, {
    text: "Halo!",
    footer: "Pesan Footer",
    cards: [
        {
            image: { url: 'https://example.jpg' }, 
            title: 'Judul Kartu',
            caption: 'Keterangan Kartu',
            footer: 'Footer Kartu',
            buttons: [
                { name: "quick_reply", buttonParamsJson: JSON.stringify({ display_text: "Tombol Cepat", id: "ID" }) },
                { name: "cta_url", buttonParamsJson: JSON.stringify({ display_text: "Buka Link", url: "https://www.example.com" }) }
            ]
        }
    ]
}, { quoted: m })
```

### 📷 Order Albums  
```ts
await sock.sendAlbumMessage(msg.key.remoteJid, [
    { image: { url: "https://example.jpg" }, caption: "Halo Dunia" },
    { video: { url: "https://example.mp4" }, caption: "Halo Dunia" }
], { quoted: m, delay: 2000 })
```

### 📌 Save & Pin Messages  
```ts
await sock.sendMessage(msg.key.remoteJid, { keep: message.key, type: 1, time: 86400 })
await sock.sendMessage(msg.key.remoteJid, { pin: message.key, type: 1, time: 86400 })
```

### 📨 Group Invite Message  
```ts
await sock.sendMessage(msg.key.remoteJid, { 
    groupInvite: { 
        subject: "Nama Grup",
        jid: "1234@g.us",
        text: "Undangan Grup",
        inviteCode: "KODE",
        inviteExpiration: 86400 * 3,
    } 
}, { quoted: m })
```
**And Many More New Message Codes⚡**
---

## 🤝 Contributions
 We welcome contributions from everyone! If you'd like to help: 
 1. **Fork** this repository 
 2. **Create a new branch** 
 3. **Create a Pull Request (PR)** 
💡 Have a great idea? Please create an **Issue** at 
[repository ini](https://github.com/HAMZAxLEGEND1/baileys/issues).  

---

## 📬 Contact

📱**whatsapp** 9234567512069
📩 **Email**: hamzaxlegend19024@gmail.com  
🌍 **Website**: [Baileys API](https://github.com/HAMZAxLEGEND/baileys)  

---

🚀 **Hope you like it & have fun coding!**
