const express = require("express");
const http = require("http");
const path = require("path");
const fs = require("fs");
const crypto = require("crypto");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);

const io = new Server(server, {
  cors: { origin: "*", methods: ["GET", "POST"] },
  maxHttpBufferSize: 6 * 1024 * 1024
});

// ==============================================
// OBSŁUGA OBRAZÓW I GIF-ÓW Z GBOARD
// ==============================================

const uploadsDir = path.join(__dirname, "uploads");
fs.mkdirSync(uploadsDir, { recursive: true });

app.use("/uploads", express.static(uploadsDir, {
  maxAge: "7d"
}));

app.post(
  "/upload-image",
  express.raw({
    type: "image/*",
    limit: "5mb"
  }),
  (req, res) => {
    const mime = String(
      req.headers["content-type"] || ""
    )
      .split(";")[0]
      .trim()
      .toLowerCase();

    const extensions = {
      "image/gif": "gif",
      "image/png": "png",
      "image/jpeg": "jpg",
      "image/webp": "webp",
      "image/avif": "avif"
    };

    const extension = extensions[mime];

    if (!extension) {
      return res.status(415).json({
        error: "Nieobsługiwany format obrazu.",
        contentType: mime
      });
    }

    if (
      !Buffer.isBuffer(req.body) ||
      req.body.length === 0
    ) {
      return res.status(400).json({
        error: "Nie otrzymano danych obrazu."
      });
    }

    const filename =
      crypto.randomBytes(16).toString("hex") +
      "." +
      extension;

    fs.writeFile(
      path.join(uploadsDir, filename),
      req.body,
      error => {
        if (error) {
          console.error(
            "BŁĄD ZAPISU OBRAZU:",
            error
          );

          return res.status(500).json({
            error: "Nie udało się zapisać obrazu."
          });
        }

        console.log(
          "OBRAZ ZAPISANY:",
          filename,
          req.body.length,
          "bajtów"
        );

        return res.json({
          success: true,
          url: "/uploads/" + filename
        });
      }
    );
  }
);

// ==============================================
// STRONA GŁÓWNA CHATSKI
// ==============================================

// index.html znajduje się w głównym folderze projektu.
app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "index.html"));
});

// Dodatkowe pliki strony z folderu public.
app.use(express.static(path.join(__dirname, "public")));

const PORT = process.env.PORT || 3000;

const waitingUsers = new Map();

// ==============================================
// POMOCNICZE
// ==============================================

function normalizeGender(value) {
  const v = String(value || "")
    .trim()
    .toLowerCase();

  if (
    v === "male" ||
    v === "mężczyzna" ||
    v === "mezczyzna"
  ) {
    return "male";
  }

  if (
    v === "female" ||
    v === "kobieta"
  ) {
    return "female";
  }

  return null;
}

function normalizeSeeking(value) {
  const v = String(value || "")
    .trim()
    .toLowerCase();

  if (
    v === "male" ||
    v === "mężczyzna" ||
    v === "mezczyzna"
  ) {
    return "male";
  }

  if (
    v === "female" ||
    v === "kobieta"
  ) {
    return "female";
  }

  if (
    v === "random" ||
    v === "losowo" ||
    v === "any" ||
    v === "all"
  ) {
    return "random";
  }

  return "random";
}

function number(value, fallback) {
  const n = Number(value);

  return Number.isFinite(n) ? n : fallback;
}

function getOnlineCount() {
  return io.sockets.sockets.size;
}

function getSearchingCount() {
  let count = 0;

  for (const socket of io.sockets.sockets.values()) {
    if (
      socket.data.state === "searching" ||
      socket.data.state === "waiting"
    ) {
      count++;
    }
  }

  return count;
}

function updateStats() {
  const online = getOnlineCount();
  const searching = getSearchingCount();

  io.emit("online", online);
  io.emit("searching-count", searching);
  io.emit("searching", searching);
}

function removeFromQueue(socket) {
  if (!socket) return;

  waitingUsers.delete(socket.id);
}

function isConnected(socket) {
  return (
    !!socket &&
    io.sockets.sockets.has(socket.id)
  );
}

// ==============================================
// MATCHMAKING
// ==============================================

function canMatch(a, b) {
  if (!a || !b) {
    return false;
  }

  const aGender = normalizeGender(a.gender);
  const bGender = normalizeGender(b.gender);

  const aSeeking = normalizeSeeking(a.seeking);
  const bSeeking = normalizeSeeking(b.seeking);

  // RANDOM łączy się wyłącznie z RANDOM
  if (
    aSeeking === "random" ||
    bSeeking === "random"
  ) {
    if (
      aSeeking !== "random" ||
      bSeeking !== "random"
    ) {
      return false;
    }
  } else {
    // Przy konkretnej płci obie strony
    // muszą wzajemnie spełniać swoje kryteria.
    if (!aGender || !bGender) {
      return false;
    }

    if (aSeeking !== bGender) {
      return false;
    }

    if (bSeeking !== aGender) {
      return false;
    }
  }

  // Zakresy wieku
  const aAge = number(a.age, NaN);
  const bAge = number(b.age, NaN);

  const aMin = number(a.minAge, NaN);
  const aMax = number(a.maxAge, NaN);

  const bMin = number(b.minAge, NaN);
  const bMax = number(b.maxAge, NaN);

  if (
    ![
      aAge,
      bAge,
      aMin,
      aMax,
      bMin,
      bMax
    ].every(Number.isFinite)
  ) {
    return false;
  }

  const aAcceptsB =
    bAge >= aMin &&
    bAge <= aMax;

  const bAcceptsA =
    aAge >= bMin &&
    aAge <= bMax;

  if (!aAcceptsB || !bAcceptsA) {
    return false;
  }

  return true;
}

// ==============================================
// GIF PROFILOWY
// ==============================================

function profileGif(value) {
  if (
    typeof value !== "string" ||
    !value ||
    value.length > 1000
  ) {
    return null;
  }

  try {
    const url = new URL(value);

    if (url.protocol !== "https:") {
      return null;
    }

    if (!url.hostname.includes("giphy.com")) {
      return null;
    }

    return value;
  } catch {
    return null;
  }
}

// ==============================================
// SZUKANIE PARTNERA
// ==============================================

function findPartner(socket) {
  // Usuwamy nieaktualne osoby z kolejki.
  for (const [id, candidate] of waitingUsers) {
    if (
      !isConnected(candidate) ||
      candidate.data.state !== "waiting"
    ) {
      waitingUsers.delete(id);
    }
  }

  for (const [id, partner] of waitingUsers) {
    if (id === socket.id) {
      continue;
    }

    if (!isConnected(partner)) {
      continue;
    }

    if (partner.data.state !== "waiting") {
      continue;
    }

    if (canMatch(socket.data, partner.data)) {
      return partner;
    }
  }

  return null;
}

// ==============================================
// TWORZENIE ROZMOWY
// ==============================================

function createMatch(a, b) {
  const room = `room_${Date.now()}_${a.id}`;

  removeFromQueue(a);
  removeFromQueue(b);

  a.join(room);
  b.join(room);

  a.data.room = room;
  b.data.room = room;

  a.data.state = "matched";
  b.data.state = "matched";

  a.emit("matched", {
    partnerId: b.id,
    profileGif: b.data.profileGif || null
  });

  b.emit("matched", {
    partnerId: a.id,
    profileGif: a.data.profileGif || null
  });

  a.emit("partner-profile", {
    partnerId: b.id,
    profileGif: b.data.profileGif || null
  });

  b.emit("partner-profile", {
    partnerId: a.id,
    profileGif: a.data.profileGif || null
  });

  a.emit("connected");
  b.emit("connected");

  console.log(
    `MATCH: ${a.id} <-> ${b.id} | ${room}`
  );

  updateStats();
}

// ==============================================
// ROZPOCZĘCIE SZUKANIA
// ==============================================

function startSearching(socket) {
  if (socket.data.state === "matched") {
    return;
  }

  removeFromQueue(socket);

  socket.data.state = "searching";

  updateStats();

  const partner = findPartner(socket);

  if (partner) {
    createMatch(socket, partner);
    return;
  }

  socket.data.state = "waiting";

  waitingUsers.set(socket.id, socket);

  socket.emit("waiting");

  console.log(
    `WAITING: ${socket.id} | kolejka=${waitingUsers.size}`
  );

  updateStats();
}

// ==============================================
// ROZŁĄCZENIE Z CZATU
// ==============================================

function disconnectFromChat(socket, notifySelf) {
  const room = socket.data.room;

  removeFromQueue(socket);

  socket.data.room = null;

  if (!room) {
    socket.data.state = "idle";

    if (notifySelf) {
      socket.emit("disconnected");
    }

    updateStats();
    return;
  }

  const partner = [
    ...io.sockets.sockets.values()
  ].find(
    other =>
      other.id !== socket.id &&
      other.data.room === room
  );

  socket.leave(room);

  socket.data.state = "idle";

  if (partner) {
    partner.leave(room);

    partner.data.room = null;
    partner.data.state = "idle";

    removeFromQueue(partner);

    partner.emit("partner-left");
  }

  if (notifySelf) {
    socket.emit("disconnected");
  }

  updateStats();
}

// ==============================================
// SOCKET.IO
// ==============================================

io.on("connection", socket => {
  // Profil domyślny
  socket.data = {
    gender: null,
    seeking: "random",
    age: 25,
    minAge: 18,
    maxAge: 80,
    profileGif: null,
    room: null,
    state: "idle"
  };

  console.log(
    `CONNECT: ${socket.id} | online=${getOnlineCount()}`
  );

  socket.emit("online", getOnlineCount());
  socket.emit("searching-count", getSearchingCount());
  socket.emit("searching", getSearchingCount());

  updateStats();

  // START SZUKANIA
  socket.on("searching-start", () => {
    startSearching(socket);
  });

  // STOP SZUKANIA
  socket.on("searching-stop", () => {
    removeFromQueue(socket);

    if (
      socket.data.state === "searching" ||
      socket.data.state === "waiting"
    ) {
      socket.data.state = "idle";
    }

    updateStats();
  });

  // USTAWIANIE PROFILU
  socket.on("setProfile", data => {
    if (!data || typeof data !== "object") {
      return;
    }

    socket.data.gender =
      normalizeGender(data.gender);

    socket.data.seeking =
      normalizeSeeking(data.seeking);

    socket.data.age = Math.min(
      80,
      Math.max(18, number(data.age, 25))
    );

    socket.data.minAge = Math.min(
      80,
      Math.max(18, number(data.minAge, 18))
    );

    socket.data.maxAge = Math.min(
      80,
      Math.max(18, number(data.maxAge, 80))
    );

    if (socket.data.minAge > socket.data.maxAge) {
      [
        socket.data.minAge,
        socket.data.maxAge
      ] = [
        socket.data.maxAge,
        socket.data.minAge
      ];
    }

    socket.data.profileGif =
      profileGif(data.profileGif);

    console.log(
      `PROFILE ${socket.id}: ` +
      `gender=${socket.data.gender}, ` +
      `seeking=${socket.data.seeking}, ` +
      `age=${socket.data.age}, ` +
      `range=${socket.data.minAge}-${socket.data.maxAge}`
    );

    updateStats();
  });

  // FIND
  socket.on("find", () => {
    console.log(
      `FIND: ${socket.id} ` +
      `gender=${socket.data.gender} ` +
      `seeking=${socket.data.seeking}`
    );

    startSearching(socket);
  });

  // STATUS AKTYWNOŚCI — wysyłany wyłącznie do drugiej osoby.
  // Klient zgłasza AFK po 10 sekundach bez aktywności.
  socket.on("activity-status", data => {
    const room = socket.data.room;

    if (
      !room ||
      socket.data.state !== "matched" ||
      !data ||
      typeof data.afk !== "boolean"
    ) {
      return;
    }

    socket.to(room).emit("activity-status", {
      afk: data.afk
    });
  });

  // WIADOMOŚCI TEKSTOWE I OBRAZY
  socket.on("message", (data, ack) => {
    const handlerStartedAt = Date.now();
    const room = socket.data.room;

    if (
      !room ||
      socket.data.state !== "matched"
    ) {
      return;
    }

    // Nowa ścieżka dla GIF-ów Gboard.
    // Obraz przesyłany binarnie przez Socket.IO.
    if (
      data &&
      typeof data === "object" &&
      data.image !== undefined
    ) {
      const mimeType = String(
        data.mimeType || "image/gif"
      )
        .split(";")[0]
        .trim()
        .toLowerCase();

      const allowedMimeTypes = new Set([
        "image/gif",
        "image/png",
        "image/jpeg",
        "image/webp",
        "image/avif"
      ]);

      if (!allowedMimeTypes.has(mimeType)) {
        console.warn(
          "[GIF POMIAR] Odrzucono nieobsługiwany MIME:",
          mimeType
        );
        return;
      }

      let imageBuffer = null;

      if (Buffer.isBuffer(data.image)) {
        imageBuffer = data.image;
      } else if (
        data.image instanceof ArrayBuffer
      ) {
        imageBuffer = Buffer.from(data.image);
      } else if (
        ArrayBuffer.isView(data.image)
      ) {
        imageBuffer = Buffer.from(
          data.image.buffer,
          data.image.byteOffset,
          data.image.byteLength
        );
      } else if (
        data.image &&
        data.image.type === "Buffer" &&
        Array.isArray(data.image.data)
      ) {
        imageBuffer = Buffer.from(data.image.data);
      }

      if (
        !imageBuffer ||
        imageBuffer.length === 0 ||
        imageBuffer.length > 5 * 1024 * 1024
      ) {
        console.warn(
          "[GIF POMIAR] Odrzucono obraz: brak danych albo przekroczony limit."
        );
        return;
      }

      console.log(
        "[GIF POMIAR] Serwer odebrał GIF:",
        imageBuffer.length,
        "bajtów; czas obsługi:",
        Date.now() - handlerStartedAt,
        "ms"
      );

      // Przekazujemy GIF wyłącznie rozmówcy.
      socket.to(room).emit("message", {
        image: imageBuffer,
        mimeType,
        sender: socket.id,
        serverSentAt: Date.now()
      });

      // Potwierdzenie odbioru przez serwer.
      if (typeof ack === "function") {
        ack({
          ok: true,
          bytes: imageBuffer.length
        });
      }

      console.log(
        "[GIF POMIAR] Serwer przekazał GIF do pokoju."
      );

      return;
    }

    // Dotychczasowa obsługa tekstu i linków.
    let text = "";

    if (typeof data === "string") {
      text = data;
    } else if (
      data &&
      typeof data.text === "string"
    ) {
      text = data.text;
    }

    text = text.trim();

    if (!text) {
      return;
    }

    const isImage =
      text.startsWith("data:image/") ||
      text.startsWith("https://");

    const maxLength = isImage
      ? 5 * 1024 * 1024
      : 5000;

    if (text.length > maxLength) {
      return;
    }

    io.to(room).emit("message", {
      text: text,
      sender: socket.id
    });
  });

  // PISANIE
  socket.on("typing", () => {
    if (socket.data.room) {
      socket
        .to(socket.data.room)
        .emit("typing");
    }
  });

  // INFORMACJA O NAGRYWANIU GŁOSÓWKI
  socket.on("voice-recording", isRecording => {
    if (!socket.data.room) {
      return;
    }

    if (typeof isRecording !== "boolean") {
      return;
    }

    socket
      .to(socket.data.room)
      .emit("voice-recording", isRecording);
  });

  // GŁOSÓWKI
  socket.on("voice-message", data => {
    const room = socket.data.room;

    if (
      !room ||
      socket.data.state !== "matched"
    ) {
      return;
    }

    if (!data || typeof data !== "object") {
      return;
    }

    const audio = data.audio;

    let buffer = null;

    if (Buffer.isBuffer(audio)) {
      buffer = audio;
    } else if (audio instanceof Uint8Array) {
      buffer = Buffer.from(audio);
    } else if (audio instanceof ArrayBuffer) {
      buffer = Buffer.from(
        new Uint8Array(audio)
      );
    } else if (
      audio &&
      audio.type === "Buffer" &&
      Array.isArray(audio.data)
    ) {
      buffer = Buffer.from(audio.data);
    }

    if (!buffer || buffer.length === 0) {
      return;
    }

    if (buffer.length > 5 * 1024 * 1024) {
      return;
    }

    const mimeType =
      typeof data.mimeType === "string" &&
      data.mimeType.startsWith("audio/")
        ? data.mimeType
        : "audio/mp4";

    const duration = Number.isFinite(
      Number(data.duration)
    )
      ? Math.max(0, Number(data.duration))
      : 0;

    socket
      .to(room)
      .emit("voice-message", {
        audio: buffer,
        sender: socket.id,
        mimeType: mimeType,
        duration: duration
      });
  });

  // RĘCZNE ROZŁĄCZENIE
  socket.on("manual-disconnect", () => {
    console.log(
      `MANUAL DISCONNECT: ${socket.id}`
    );

    disconnectFromChat(socket, true);
  });

  // ROZŁĄCZENIE SOCKET.IO
  socket.on("disconnect", reason => {
    console.log(
      `DISCONNECT: ${socket.id} | ${reason}`
    );

    const room = socket.data.room;

    removeFromQueue(socket);

    if (room) {
      const partner = [
        ...io.sockets.sockets.values()
      ].find(
        other =>
          other.id !== socket.id &&
          other.data.room === room
      );

      if (partner) {
        partner.data.room = null;
        partner.data.state = "idle";

        removeFromQueue(partner);

        partner.emit("partner-left");
      }
    }

    updateStats();
  });
});

// ==============================================
// ODŚWIEŻANIE STATYSTYK
// ==============================================

setInterval(updateStats, 500);

// ==============================================
// START SERWERA
// ==============================================

server.listen(PORT, "0.0.0.0", () => {
  console.log("====================================");
  console.log(
    `CHATSKI SERVER DZIAŁA NA PORCIE ${PORT}`
  );
  console.log(`http://localhost:${PORT}`);
  console.log("====================================");
});
